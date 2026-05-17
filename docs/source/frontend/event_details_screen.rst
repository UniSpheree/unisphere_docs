=========================
Event details page - event_details_screen.dart
=========================

``event_details_screen.dart`` is responsible for rendering the comprehensive, 
detailed view of a single event. It handles local data fetching, conditional UI 
rendering based on the user's authentication status and role (attendee vs. organizer)
, and manages the complete ticket purchasing workflow.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'dart:typed_data';
   import 'package:unisphere_app/services/sqlite_backend.dart';
   import 'package:unisphere_app/models/database_models.dart';
   import '../widgets/header.dart';
   import '../widgets/app_footer.dart';

These imports provide the essential dependencies for the screen. They pull in the 
core Flutter framework for UI construction, ``dart:typed_data`` for handling raw 
image byte arrays (such as custom event banners), the local SQLite database service 
and models for real-time state synchronization, and the shared global header and 
footer widgets to maintain structural consistency across the application.

EventDetailsScreen Class
------------------------

.. code-block:: dart

   class EventDetailsScreen extends StatefulWidget {
     final Map<String, dynamic> event;
     final bool allowPurchase;

     const EventDetailsScreen({
       super.key,
       required this.event,
       this.allowPurchase = true,
     });

     @override
     State<EventDetailsScreen> createState() => _EventDetailsScreenState();
   }

The ``EventDetailsScreen`` is a stateful widget that acts as the entry point for 
viewing an event. It requires an ``event`` map containing the event's raw data 
payload. It also accepts an optional ``allowPurchase`` boolean (defaulting to true),
which allows parent routing components to disable ticketing UI if the event is being
previewed by an organizer or is already past.

Lifecycle & Database Listener
-----------------------------

.. code-block:: dart

   class _EventDetailsScreenState extends State<EventDetailsScreen> {
     void _onBackendChanged() {
       if (!mounted) return;
       setState(() {});
     }

     @override
     void initState() {
       super.initState();
       SqliteBackend().addListener(_onBackendChanged);
     }

     @override
     void dispose() {
       SqliteBackend().removeListener(_onBackendChanged);
       super.dispose();
     }
     
     // ... build method ...
   }

This state class manages the dynamic reactivity of the details screen. During 
``initState``, it subscribes to the ``SqliteBackend``. If any changes occur in the 
database (e.g., the event is updated, deleted, or a ticket is bought elsewhere), 
the ``_onBackendChanged`` callback safely triggers a UI rebuild. The ``dispose`` 
method ensures this listener is cleanly removed when the user navigates away to 
prevent memory leaks.

Deleted Event State
-------------------

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     final currentEventId = widget.event['id']?.toString();
     final latestEvent = currentEventId == null
         ? widget.event
         : SqliteBackend().events.firstWhere(
             (e) => e['id']?.toString() == currentEventId,
             orElse: () => widget.event,
           );
     final isDeleted = currentEventId != null &&
         !SqliteBackend().events.any((e) => e['id']?.toString() == currentEventId);

     if (isDeleted) {
       return Scaffold(
         backgroundColor: const Color(0xFFF0F2F8),
         body: Center(
           child: ConstrainedBox(
             constraints: const BoxConstraints(maxWidth: 560),
             child: Container(
               // ... styling ...
               child: Column(
                 mainAxisSize: MainAxisSize.min,
                 children: [
                   const Icon(Icons.event_busy, size: 56, color: Colors.redAccent),
                   const Text('This event was deleted', ...),
                   Text('The event is no longer available.', ...),
                   FilledButton(
                     onPressed: () => Navigator.maybePop(context),
                     child: const Text('Go back'),
                   ),
                 ],
               ),
             ),
           ),
         ),
       );
     }
     
     // ... main render logic ...
   }

At the very beginning of the ``build`` method, the screen performs a safety check 
against the local database to see if the event has been deleted since the user 
opened the screen. By comparing the passed ``widget.event`` ID against the current 
database state, it determines if the data is still valid. If ``isDeleted`` evaluates 
to true, the UI intercepts the normal rendering flow and immediately returns an error 
state ``Scaffold``, presenting a friendly error message and a button allowing the 
user to safely navigate back to the previous screen.

Field extraction and helpers
----------------------------

.. code-block:: dart

    final color = event['color'] as Color? ?? const Color(0xFF4F46E5);
    final description = event['description'] as String? ?? 'No extra description provided for this event.';
    final organizer = event['organizer'] as String? ?? 'Organizer not specified';
    final capacity = event['capacity'] != null ? '${event['capacity']}' : null;
    final tags = (event['tags'] as List<dynamic>?)?.cast<String>() ?? [];
    final price = (event['price'] as String?)?.trim();

The widget reads event metadata and provides sensible defaults when fields
are missing. The `color` token is used to theme badges and the purchase CTA.

Data Parsing & Role Resolution
------------------------------

.. code-block:: dart

   final event = latestEvent;
   final allowPurchase = widget.allowPurchase;
   final color = event['color'] as Color? ?? const Color(0xFF4F46E5);
   final description = event['description'] as String? ?? 'No extra description provided for this event.';
   final bannerImageData = event['bannerImageData'];
   final Uint8List? bannerBytes = bannerImageData is Uint8List ? bannerImageData : null;

   final organizer = event['organizer'] as String? ?? 'Organizer not specified';
   final capacity = event['capacity'] != null ? '${event['capacity']}' : null;
   final tags = (event['tags'] as List<dynamic>?)?.cast<String>() ?? [];
   final price = (event['price'] as String?)?.trim();

   final eventId = int.tryParse(event['id']?.toString() ?? '');
   final canonicalEvent = SqliteBackend().events.cast<Map<String, dynamic>?>().firstWhere(
     (e) => e != null && int.tryParse(e['id']?.toString() ?? '') == eventId,
     orElse: () => null,
   );
   final organizerEmail = (canonicalEvent?['organizerEmail'] ?? event['organizerEmail'])?.toString() ?? '';

   final isOrganizerViewing = SqliteBackend().currentUser?.email == organizerEmail;

Before constructing the main UI, the ``build`` method safely extracts all necessary 
event attributes (falling back to sensible defaults for missing data). Crucially, 
it queries the ``SqliteBackend`` for the event's "canonical" record to retrieve the 
true ``organizerEmail``. It then compares this email against the currently logged-in 
user to evaluate ``isOrganizerViewing``, a boolean flag that heavily dictates the 
UI states shown later on the screen.

Main Layout & Header Details
----------------------------

.. code-block:: dart

   return Scaffold(
     backgroundColor: const Color(0xFFF0F2F8),
     body: LayoutBuilder(
       builder: (context, constraints) {
         return SingleChildScrollView(
           child: ConstrainedBox(
             constraints: BoxConstraints(minHeight: constraints.maxHeight),
             child: Column(
               mainAxisAlignment: MainAxisAlignment.spaceBetween,
               children: [
                 Column(
                   children: [
                     const AppHeader(),
                     Padding(
                       // ... Breadcrumbs (Back / Event Title) ...
                       
                       // Large header card
                       Container(
                         // ... styling ...
                         child: Column(
                           crossAxisAlignment: CrossAxisAlignment.start,
                           children: [
                             if (bannerBytes != null) ...[
                               ClipRRect(...), // Hero Banner Image
                             ],
                             Text(event['title'] as String, ...),
                             Row(
                               children: [
                                 Icon(Icons.calendar_today_outlined, ...),
                                 Text(event['date'] as String, ...),
                                 Icon(Icons.location_on_outlined, ...),
                                 Text(event['location'] as String, ...),
                               ],
                             ),
                             // ... Category and Tags Wrap ...
                             // ... Organizer and Capacity Row ...
                           ],
                         ),
                       ),
                       // ... remaining components ...
                     ),
                   ],
                 ),
                 const AppFooter(),
               ],
             ),
           ),
         );
       },
     ),
   );

The primary structure utilizes a responsive ``LayoutBuilder`` and 
``SingleChildScrollView`` to ensure the content scrolls smoothly while keeping the 
``AppFooter`` anchored to the bottom. The top of the page features breadcrumb navigation 
for easy backward routing. Below that sits the main hero card, which displays the 
event's banner image, title, date, location, tags, and organizer information in a 
clean, hierarchical format.

Event Description Section
-------------------------

.. code-block:: dart

   Container(
     width: double.infinity,
     padding: const EdgeInsets.all(32),
     decoration: BoxDecoration(
       color: Colors.white,
       borderRadius: BorderRadius.circular(24),
       // ... shadows ...
     ),
     child: Column(
       crossAxisAlignment: CrossAxisAlignment.start,
       children: [
         const Text('About this event', style: TextStyle(fontSize: 22, fontWeight: FontWeight.w800)),
         const SizedBox(height: 16),
         Text(
           description,
           style: TextStyle(fontSize: 16, height: 1.6, color: Colors.grey[800]),
         ),
       ],
     ),
   )

This straightforward section acts as the content body of the page. It wraps the 
raw ``description`` string parsed earlier in a stylized card, providing the user 
with the organizer's full context about the event's schedule or requirements.

Action Bar & Purchase Flow
--------------------------

.. code-block:: dart

   if (allowPurchase && !isOrganizerViewing)
     Container(
       // ... "Buy Ticket" UI ...
       child: Row(
         children: [
           // ... Price Display ...
           FilledButton(
             onPressed: () {
               if (SqliteBackend().currentUser == null) {
                 // Guest Flow: Save pending purchase and redirect to login
                 final pending = DbPurchasedTicket(...);
                 SqliteBackend().setPendingPurchase(pending);
                 Navigator.pushNamed(context, '/register');
                 return;
               }

               // Authenticated Flow: Complete purchase instantly
               SqliteBackend().purchaseTicket(DbPurchasedTicket(...));
               Navigator.pushNamed(context, '/my-tickets');
             },
             child: const Text('Buy ticket now'),
           ),
         ],
       ),
     )
   else if (isOrganizerViewing)
     Container(
       // ... Organizer Warning UI ...
       child: Text('This is your event. You can manage it from the My Events page.'),
     )
   else
     Container(
       // ... Already Purchased UI ...
       child: Text('You already have a ticket for this event.'),
     )

The action bar at the bottom of the content area handles the business logic of the 
page, rendering one of three distinct states:

1. **Purchase State:** If ticketing is allowed and the user is an attendee, it shows the price and a "Buy ticket now" button. Clicking this button dynamically checks the user's authentication state. Unauthenticated guests have their ticket payload saved as a "pending purchase" before being routed to the registration screen. Authenticated users purchase the ticket instantly and are routed to their ticket wallet.
2. **Organizer State:** If the ``isOrganizerViewing`` flag is true, purchasing is disabled to prevent organizers from buying their own tickets. They are shown a message redirecting them to their dashboard.
3. **Purchased/Disabled State:** If the event does not permit purchases (e.g., they already own a ticket), a success banner is shown confirming their registration.