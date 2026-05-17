================
api_backend.dart
================

``api_backend.dart`` acts as the central data repository and network layer for 
the application. Although the class is historically named ``SqliteBackend`` 
(and exported via a stub ``sqlite_backend.dart`` file), it actually implements 
a robust HTTP client that communicates with a live REST API. It manages local 
application state using the ``ChangeNotifier`` pattern, ensuring the UI remains 
synchronized with the server.


Imports
-------

.. code-block:: dart

   import 'dart:convert';
   import 'dart:typed_data';

   import 'package:flutter/foundation.dart';
   import 'package:http/http.dart' as http;
   import 'package:shared_preferences/shared_preferences.dart';

   import '../models/database_models.dart';

This block handles the core network, encoding, and persistence dependencies 
required to communicate with the external API and store data locally.


Singleton Setup
---------------

.. code-block:: dart

   class SqliteBackend extends ChangeNotifier {
     SqliteBackend._internal();

     static final SqliteBackend _instance = SqliteBackend._internal();
     factory SqliteBackend() => _instance;
     factory SqliteBackend.getInstance() => _instance;
     
     // ...
   }

The ``SqliteBackend`` class utilizes the Singleton design pattern via the 
``_internal()`` constructor and factory methods. This guarantees that every 
widget in the Flutter application interacts with the exact same instance of 
the backend, preventing data desynchronization and redundant network calls.


State & Network Initialization
------------------------------

.. code-block:: dart

   static const String _defaultBaseUrl = String.fromEnvironment(
     'API_BASE_URL',
     defaultValue: 'http://127.0.0.1:8000',
   );

   final http.Client _client = http.Client();

   DbUser? _currentUser;
   final List<DbPurchasedTicket> _purchasedTickets = [];
   final List<Map<String, dynamic>> _cachedEvents = [];
   DbPurchasedTicket? _pendingPurchase;
   Map<String, dynamic>? _pendingEvent;

   String _baseUrl = _defaultBaseUrl;

   static const String _kCurrentUserKey = 'current_user';

   DbUser? get currentUser => _currentUser;
   List<DbPurchasedTicket> get purchasedTickets =>
       List.unmodifiable(_purchasedTickets);
   List<Map<String, dynamic>> get events => List.unmodifiable(_cachedEvents);

This segment defines the core memory of the application. It establishes the 
``_baseUrl`` (which can be overridden via compile-time environment variables) 
and spins up a persistent ``http.Client``. 

Crucially, it holds the master copies of the user session, ticket inventory, 
and event feed. To protect this state from being accidentally modified by the UI, 
it exposes these lists exclusively through ``List.unmodifiable()`` getters, forcing 
components to use the designated backend methods for any data mutation. It also sets 
up ``_pendingPurchase`` and ``_pendingEvent`` to temporarily hold actions from 
unauthenticated guest users.


Local Persistence (SharedPreferences)
-------------------------------------

To provide a seamless user experience, the backend caches the authenticated user 
session locally. This prevents the user from having to log in every time they 
completely close and reopen the application.

_saveCurrentUserToPrefs & _loadSavedUser & _clearSavedUserFromPrefs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<void> _saveCurrentUserToPrefs() async {
     try {
       final prefs = await SharedPreferences.getInstance();
       if (_currentUser == null) {
         await prefs.remove(_kCurrentUserKey);
         return;
       }
       await prefs.setString(
         _kCurrentUserKey,
         jsonEncode(_currentUser!.toMap()),
       );
     } catch (e) {
       print('Error saving current user: $e');
     }
   }

   Future<void> _loadSavedUser() async {
     try {
       final prefs = await SharedPreferences.getInstance();
       final raw = prefs.getString(_kCurrentUserKey);
       if (raw == null || raw.isEmpty) return;
       _currentUser = DbUser.fromMap(jsonDecode(raw) as Map<String, dynamic>);
     } catch (e) {
       print('Error loading saved user: $e');
     }
   }

   Future<void> _clearSavedUserFromPrefs() async {
     try {
       final prefs = await SharedPreferences.getInstance();
       await prefs.remove(_kCurrentUserKey);
     } catch (e) {
       print('Error clearing saved user: $e');
     }
   }

These asynchronous helpers interface with the device's native storage via the 
``shared_preferences`` plugin. When a user authenticates, their ``DbUser`` object 
is serialized into a JSON string and saved under the ``_kCurrentUserKey``. Upon app 
startup, ``_loadSavedUser`` attempts to read and decode this string back into the 
active ``_currentUser`` memory slot. The ``_clearSavedUserFromPrefs`` method safely 
purges this data during the logout or account deletion flows.


Core HTTP Helpers
-----------------

.. code-block:: dart

   Uri _uri(String path) => Uri.parse('$_baseUrl$path');

   Future<http.Response> _get(String path) => _client.get(_uri(path));
   
   Future<http.Response> _delete(String path) => _client.delete(_uri(path));
   
   Future<http.Response> _post(String path, Map<String, dynamic> body) =>
       _client.post(
         _uri(path),
         headers: const {'Content-Type': 'application/json'},
         body: jsonEncode(body),
       );
       
   Future<http.Response> _put(String path, Map<String, dynamic> body) =>
       _client.put(
         _uri(path),
         headers: const {'Content-Type': 'application/json'},
         body: jsonEncode(body),
       );

To keep the rest of the backend code clean and readable, all external network calls 
are routed through these private helper methods. They automatically construct the 
full URI using the environment's base URL. For ``POST`` and ``PUT`` requests, they 
automatically inject the required ``application/json`` headers and serialize the 
Dart map payloads into JSON strings, completely abstracting away the boilerplate 
of the ``http`` package.


Data Mapping & Initialization
-----------------------------

This section handles the application's boot sequence and the critical task of 
translating raw JSON data from the server into strongly-typed Dart models.

initializeDatabase & _loadEventsFromApi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<void> initializeDatabase() async {
     try {
       await _loadSavedUser();
       final response = await _get('/health');
       if (response.statusCode != 200) {
         throw Exception('Backend not available');
       }
       await _loadEventsFromApi();
       print('✓ API backend initialized successfully');
     } catch (e) {
       print('✗ Error initializing API backend: $e');
     }
   }

   Future<void> _loadEventsFromApi() async {
     try {
       final response = await _get('/events');
       if (response.statusCode != 200) throw Exception('Failed to load events');

       final list = jsonDecode(response.body) as List<dynamic>;
       final loaded = <Map<String, dynamic>>[];

       for (final item in list) {
         final raw = Map<String, dynamic>.from(item as Map);
         final eventMap = await _mapEventFromApi(raw);
         loaded.add(eventMap);
       }

       _cachedEvents
         ..clear()
         ..addAll(loaded);

       _pruneStalePurchasedTickets();
       notifyListeners();
     } catch (e) {
       print('Error loading events: $e');
     }
   }

``initializeDatabase`` is the bootloader for the app. It first loads the 
persistent user session, pings the server's ``/health`` endpoint to ensure the 
API is reachable, and then triggers the event fetcher. 

``_loadEventsFromApi`` queries the primary event feed. It parses the JSON array, 
maps each item, updates the global ``_cachedEvents`` list, triggers a cleanup of 
any tickets for events that no longer exist, and finally calls ``notifyListeners()`` 
to force the UI to render the newly fetched data.

_mapEventFromApi
^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<Map<String, dynamic>> _mapEventFromApi(Map<String, dynamic> raw) async {
     final organizerEmail = raw['organizerEmail']?.toString() ?? '';
     final map = <String, dynamic>{
       // ... standard string and date mappings ...
       'organizerEmail': organizerEmail,
       'bannerImageUrl': raw['bannerImageUrl']?.toString(),
     };

     // Fetch organizer name
     if (organizerEmail.isNotEmpty) {
       try {
         final response = await _get('/profiles/$organizerEmail');
         if (response.statusCode == 200) {
           final userData = jsonDecode(response.body) as Map<String, dynamic>;
           map['organizer'] = '${userData['firstName']} ${userData['lastName']}'.trim();
         } else {
           map['organizer'] = 'UniSphere';
         }
       } catch (e) {
         map['organizer'] = 'UniSphere';
       }
     } else {
       map['organizer'] = 'UniSphere';
     }

     // Fetch banner image bytes
     final bannerUrl = map['bannerImageUrl'] as String?;
     if (bannerUrl != null && bannerUrl.isNotEmpty) {
       try {
         final bytes = await _client.get(Uri.parse(bannerUrl));
         if (bytes.statusCode == 200) {
           map['bannerImageData'] = bytes.bodyBytes;
         }
       } catch (e) {
         print('Error loading banner image: $e');
       }
     }

     return map;
   }

This is the most complex data mapping function in the backend. Because the 
API returns a normalized event object containing only the organizer's email 
and an image URL, this function performs secondary network requests to enrich 
the data. It calls the ``/profiles/`` endpoint to fetch the organizer's actual 
first and last name for display, and directly downloads the image bytes from the 
``bannerImageUrl`` so the UI can render the image instantly from memory without 
additional network latency.

_userFromApi & _ticketFromApi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   DbUser _userFromApi(Map<String, dynamic> json) {
     return DbUser(
       id: json['id'] as int?,
       email: json['email']?.toString() ?? '',
       password: '', // Password never stored in state
       firstName: json['firstName']?.toString() ?? '',
       lastName: json['lastName']?.toString() ?? '',
       role: json['role']?.toString() ?? 'Attendee',
       university: json['university']?.toString() ?? '',
       description: json['description']?.toString() ?? '',
       isApproved: json['isApproved'] == true,
       createdAt: DateTime.parse(json['createdAt']?.toString() ?? DateTime.now().toIso8601String()),
     );
   }

   DbPurchasedTicket _ticketFromApi(Map<String, dynamic> json) {
    return DbPurchasedTicket(
      id: json['id'] as int?,
      userEmail: json['userEmail']?.toString() ?? '',
      title: json['title']?.toString() ?? '',
      date: json['date']?.toString() ?? '',
      location: json['location']?.toString() ?? '',
      category: json['category']?.toString() ?? '',
      price: json['price']?.toString() ?? '',
      purchasedAt: DateTime.parse(
        json['purchasedAt']?.toString() ?? DateTime.now().toIso8601String(),
      ),
      eventId: json['eventId'] as int?,
    );
  }

These are standard factory methods used to safely construct strongly-typed 
``DbUser`` and ``DbPurchasedTicket`` model classes from raw API JSON. They rely 
heavily on null-aware operators (``??``) to provide safe default values, ensuring 
the app does not crash if the backend returns an unexpected null field.

Public Event Getters
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<List<Map<String, dynamic>>> getEvents() async {
     await _loadEventsFromApi();
     return events;
   }

   Future<Map<String, dynamic>?> getEventById(String id) async {
     try {
       final response = await _get('/events/$id');
       if (response.statusCode != 200) return null;
       return await _mapEventFromApi(
         jsonDecode(response.body) as Map<String, dynamic>,
       );
     } catch (e) {
       print('Error getting event: $e');
       return null;
     }
   }

These functions act as the primary read-only interface for the UI. ``getEvents`` 
forces a fresh network load before returning the unmodifiable event list. 
``getEventById`` makes a targeted network call to retrieve a single event's details, 
passing the raw response through the ``_mapEventFromApi`` parser to ensure 
consistency.


Authentication Handlers
-----------------------

This section encapsulates all network requests related to user identity, including 
account creation, session authentication, password management, and profile 
modifications.

register & login
^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<bool> register({
     required String email, required String password, required String firstName,
     required String lastName, required String role, required String university,
     required bool isApproved,
   }) async {
     try {
       final response = await _post('/auth/register', {
         'email': email, 'password': password, 'firstName': firstName,
         'lastName': lastName, 'role': role, 'university': university,
         'description': '', 'isApproved': isApproved,
       });

       if (response.statusCode != 200 && response.statusCode != 201) return false;

       _currentUser = _userFromApi(jsonDecode(response.body) as Map<String, dynamic>);
       await _saveCurrentUserToPrefs();
       await _loadEventsFromApi();
       await _completePendingPurchaseInternal();
       await _completePendingEventInternal();
       notifyListeners();
       return true;
     } catch (e) {
       return false;
     }
   }

   Future<bool> login({required String email, required String password}) async {
     try {
       final response = await _post('/auth/login', {'email': email, 'password': password});
       if (response.statusCode != 200) return false;

       _currentUser = _userFromApi(jsonDecode(response.body) as Map<String, dynamic>);
       await _saveCurrentUserToPrefs();
       
       await _loadEventsFromApi();
       final ticketsResponse = await _get('/tickets/${Uri.encodeComponent(_currentUser!.email)}');
       _purchasedTickets
         ..clear()
         ..addAll((jsonDecode(ticketsResponse.body) as List<dynamic>).map(
             (item) => _ticketFromApi(Map<String, dynamic>.from(item as Map))));
             
       _pruneStalePurchasedTickets();
       await _completePendingPurchaseInternal();
       await _completePendingEventInternal();
       notifyListeners();
       return true;
     } catch (e) {
       return false;
     }
   }

These functions handle session initialization. Upon a successful ``POST`` request 
to the respective ``/auth`` endpoints, both methods immediately deserialize the 
response into the ``_currentUser`` state and lock it into local storage via 
``_saveCurrentUserToPrefs``. 

Crucially, both flows invoke ``_completePendingPurchaseInternal()`` and 
``_completePendingEventInternal()``. This creates a seamless experience for guest 
users: if a user creates an event or buys a ticket *before* logging in, the app 
temporarily holds that payload in memory and automatically flushes it to the database 
the moment their authentication succeeds.

forgotPassword & resetPassword
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<bool> forgotPassword(String email) async {
     try {
       final response = await _post('/auth/forgot-password', {'email': email});
       if (response.statusCode != 200) return false;
       final data = jsonDecode(response.body) as Map<String, dynamic>;
       return data['exists'] == true;
     } catch (e) {
       return false;
     }
   }

   Future<bool> resetPassword(String email, String newPassword) async {
     try {
       final response = await _post('/auth/reset-password', {
         'email': email,
         'newPassword': newPassword,
       });
       return response.statusCode == 200;
     } catch (e) {
       return false;
     }
   }

These utility methods support the 3-step password recovery flow found in the UI. ``forgotPassword`` acts as a verification ping to confirm an account exists for the provided email, while ``resetPassword`` commits the new credential to the backend.

updateCurrentUserProfile & updateCurrentUserRole
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<DbUser?> updateCurrentUserProfile({required String name, required String description}) async {
     final current = _currentUser;
     if (current == null) return null;

     final parts = name.trim().split(RegExp(r'\s+')).where((p) => p.isNotEmpty).toList();
     final firstName = parts.isNotEmpty ? parts.first : current.firstName;
     final lastName = parts.length > 1 ? parts.sublist(1).join(' ') : current.lastName;

     try {
       final response = await _put('/profiles/${Uri.encodeComponent(current.email)}', {
         'firstName': firstName,
         'lastName': lastName,
         'description': description.trim(),
       });
       if (response.statusCode != 200) return null;
       
       _currentUser = _userFromApi(jsonDecode(response.body) as Map<String, dynamic>);
       await _saveCurrentUserToPrefs();
       notifyListeners();
       return _currentUser;
     } catch (e) {
       return null;
     }
   }

   // updateCurrentUserRole implementation follows identical PUT request pattern...

Triggered primarily from the ``ProfilePage`` dashboard, these methods issue ``PUT`` 
requests to modify specific user attributes. Notably, ``updateCurrentUserProfile`` 
intercepts a single "Full Name" string from the UI and safely splits it back into 
``firstName`` and ``lastName`` fields using a regex space-delimiter before sending 
the payload.

logout & deleteAccount
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   void logout() {
     _currentUser = null;
     _purchasedTickets.clear();
     _pendingPurchase = null;
     _pendingEvent = null;
     _clearSavedUserFromPrefs();
     notifyListeners();
   }

   Future<bool> deleteAccount() async {
     final current = _currentUser;
     if (current == null) return false;
     try {
       final response = await _delete('/auth/users/${Uri.encodeComponent(current.email)}');
       if (response.statusCode == 200) {
         _currentUser = null;
         _purchasedTickets.clear();
         _cachedEvents.clear();
         await _clearSavedUserFromPrefs();
         notifyListeners();
         return true;
       }
       return false;
     } catch (e) {
       return false;
     }
   }

These functions handle destructive session actions. ``logout`` strictly modifies 
local state, wiping all cached user data, tickets, and pending actions from memory 
and ``SharedPreferences``. ``deleteAccount`` goes a step further by issuing a 
``DELETE`` request to the API to permanently erase the user record before clearing 
the local state.


Event Management (CRUD)
-----------------------

This section contains the core functions utilized by authenticated organizers 
to create, update, and remove events from the backend platform.

createEvent & _createEventInternal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<String> _createEventInternal(Map<String, dynamic> eventData) async {
     if (_currentUser == null) throw Exception('No user logged in');

     final payload = {
       'title': eventData['title']?.toString() ?? '',
       'date': eventData['date']?.toString() ?? '',
       'endDate': eventData['endDate']?.toString(),
       'location': eventData['location']?.toString() ?? '',
       'category': eventData['category']?.toString() ?? '',
       'description': eventData['description']?.toString() ?? '',
       'visibility': eventData['visibility']?.toString() ?? 'Public',
       'organizerEmail': _currentUser!.email,
       'bannerImageBase64': eventData['bannerImageData'] is Uint8List
           ? base64Encode(eventData['bannerImageData'] as Uint8List)
           : null,
     };

     final response = await _post('/events', payload);
     if (response.statusCode != 200 && response.statusCode != 201) {
       throw Exception(response.body.isNotEmpty ? response.body : 'Failed to create event');
     }

     final event = jsonDecode(response.body) as Map<String, dynamic>;
     await _loadEventsFromApi();
     notifyListeners();
     return event['id'].toString();
   }

   Future<String> createEvent(Map<String, dynamic> eventData) async {
     final id = await _createEventInternal(eventData);
     await _loadEventsFromApi();
     return id;
   }

These functions construct the final payload sent to the API. A critical 
feature of ``_createEventInternal`` is how it handles images: it checks if 
``bannerImageData`` contains raw bytes (``Uint8List``) and automatically encodes 
it into a Base64 string so it can be safely transmitted as JSON. Once the API 
confirms the creation, it triggers ``_loadEventsFromApi()`` to pull the refreshed 
event list (ensuring backend-generated fields like IDs and timestamps are 
synchronized) before updating the UI.

updateEvent & deleteEvent
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<bool> updateEvent(String id, Map<String, dynamic> updated) async {
     try {
       final payload = Map<String, dynamic>.from(updated);
       if (payload['bannerImageData'] is Uint8List) {
         payload['bannerImageBase64'] = base64Encode(payload['bannerImageData'] as Uint8List);
       }
       payload.remove('bannerImageData');
       final response = await _put('/events/$id', payload);
       if (response.statusCode != 200) return false;
       await _loadEventsFromApi();
       return true;
     } catch (e) {
       return false;
     }
   }

   Future<bool> deleteEvent(String id) async {
     try {
       final response = await _delete('/events/$id');
       if (response.statusCode != 200) return false;
       await _loadEventsFromApi();
       return true;
     } catch (e) {
       return false;
     }
   }

``updateEvent`` performs similar Base64 encoding logic for modified banner images, 
explicitly dropping the raw byte data from the payload before sending the ``PUT`` 
request. ``deleteEvent`` simply issues a ``DELETE`` request targeting the specific 
event ID. Both methods force a global refresh of the events list upon success to 
keep the dashboard accurate.

setPendingPurchase & setPendingEvent & _completePendingEventInternal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   void setPendingPurchase(DbPurchasedTicket ticket) {
     _pendingPurchase = ticket;
   }

   void setPendingEvent(Map<String, dynamic> eventData) {
     _pendingEvent = Map<String, dynamic>.from(eventData);
   }

   Future<void> _completePendingEventInternal() async {
     if (_currentUser == null || _pendingEvent == null) return;
     final payload = Map<String, dynamic>.from(_pendingEvent!);
     payload['organizerEmail'] = _currentUser!.email;
     await _createEventInternal(payload);
     _pendingEvent = null;
   }

These helper functions temporarily store user actions in memory when they interact 
with the app as an unauthenticated guest. ``_completePendingEventInternal`` is 
triggered automatically during the login and registration flows; it binds the newly 
authenticated user's email to the stashed event payload and commits it to the 
database.


Ticket Purchasing & Validation
------------------------------

These methods manage the flow for attendees booking their spot at events, including 
handling edge cases where events are modified or deleted after a ticket is purchased.

purchaseTicket & _completePendingPurchaseInternal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   void purchaseTicket(DbPurchasedTicket ticket) {
     if (_currentUser == null) {
       _pendingPurchase = ticket;
       notifyListeners();
       return;
     }

     final effective = DbPurchasedTicket(
       userEmail: _currentUser!.email,
       title: ticket.title,
       date: ticket.date,
       location: ticket.location,
       category: ticket.category,
       price: ticket.price,
       purchasedAt: ticket.purchasedAt,
     );

     Future.microtask(() async {
       _pendingPurchase = effective;
       await _completePendingPurchaseInternal();
       await _loadEventsFromApi();
     });
   }

   Future<void> _completePendingPurchaseInternal() async {
     if (_currentUser == null || _pendingPurchase == null) return;

     final ticket = _pendingPurchase!;
     final payload = {
       'userEmail': _currentUser!.email,
       'title': ticket.title,
       // ... other ticket properties ...
       'eventId': ticket.eventId,
     };

     final response = await _post('/tickets', payload);
     if (response.statusCode == 200 || response.statusCode == 201) {
       final created = _ticketFromApi(jsonDecode(response.body) as Map<String, dynamic>);
       _purchasedTickets.add(created);
       _pendingPurchase = null;
       notifyListeners();
     } else {
       throw Exception('Failed to create ticket');
     }
   }

When a user clicks "Book Ticket", ``purchaseTicket`` evaluates their session. If 
they are a guest, the ticket is stored in ``_pendingPurchase`` (to be processed 
after registration). If they are logged in, it overrides the ticket's email with 
the authenticated user's email for security, queues up the network call using 
``Future.microtask``, and flushes it to the backend via 
``_completePendingPurchaseInternal``.

deleteTicket & _pruneStalePurchasedTickets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<bool> deleteTicket(String ticketId) async {
     final current = _currentUser;
     if (current == null) return false;

     try {
       final response = await _delete('/tickets/${Uri.encodeComponent(current.email)}/$ticketId');
       if (response.statusCode != 200) return false;

       _purchasedTickets.removeWhere((ticket) => ticket.id?.toString() == ticketId);
       _pruneStalePurchasedTickets();
       notifyListeners();
       return true;
     } catch (e) {
       return false;
     }
   }

   void _pruneStalePurchasedTickets() {
     if (_purchasedTickets.isEmpty) return;

     bool eventExistsForTicket(DbPurchasedTicket ticket) {
       for (final event in _cachedEvents) {
         final eventId = int.tryParse(event['id']?.toString() ?? '');
         if (ticket.eventId != null && eventId == ticket.eventId) return true;
       }
       // Fallback validation
       for (final event in _cachedEvents) {
         if (event['title'] == ticket.title && event['date'] == ticket.date && event['location'] == ticket.location) {
           return true;
         }
       }
       return false;
     }

     _purchasedTickets.removeWhere((ticket) => !eventExistsForTicket(ticket));
   }

``deleteTicket`` allows users to cancel their bookings by calling the API and 
dropping the ticket from local memory. 

``_pruneStalePurchasedTickets`` is a crucial localized garbage-collection method. 
Because organizers can delete events, attendees might be left holding tickets to 
events that no longer exist. This method iterates through all locally cached tickets 
and drops them if their parent event cannot be found in the active ``_cachedEvents`` 
list (verifying by ID, or falling back to a title/date/location match).


Debug & Diagnostics
-------------------

clear & getDiagnostics
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<void> clear() async {
     try {
       await _post('/debug/clear', {});
       logout();
       _cachedEvents.clear();
       notifyListeners();
     } catch (e) {
       print('❌ Error clearing data: $e');
     }
   }

   Future<String> getDiagnostics() async {
     try {
       final response = await _get('/debug/diagnostics');
       return response.body;
     } catch (e) {
       return 'Error getting diagnostics: $e';
     }
   }

Located at the very bottom of the file, these are developer-facing administrative 
tools. ``clear`` sends a destructive command to the API to wipe database state while 
simultaneously logging the user out and resetting the local cache. ``getDiagnostics`` 
fetches raw system health and variable states from the server.