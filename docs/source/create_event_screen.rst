========================
create_event_screen.dart
========================

``create_event_screen.dart`` defines a comprehensive, dual-purpose form screen used 
for both creating new events and editing existing ones. It is strictly protected by 
role-based access to ensure only designated organizers can utilize it, and it 
features a robust state management system to handle complex inputs like validated 
text fields, dropdowns, custom date-time pickers, and image uploads.


Imports
-------

.. code-block:: dart

   import 'dart:io';
   import 'package:flutter/foundation.dart';
   import 'package:flutter/material.dart';
   import 'package:image_picker/image_picker.dart';
   import '../models/database_models.dart';
   import '../widgets/header.dart';
   import '../widgets/app_footer.dart';
   import '../services/sqlite_backend.dart';
   import '../utils/event_categories.dart';

This block imports all necessary dependencies, including Flutter's core material 
libraries, platform-specific I/O handling, the ``image_picker`` package for accessing 
the device's local media gallery, and the application's internal data models, 
backend services, and shared UI components.


State Variables & User Resolution
---------------------------------

CreateEventScreen
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class CreateEventScreen extends StatefulWidget {
     final Map<String, dynamic>? existingEvent;
     const CreateEventScreen({super.key, this.existingEvent});

     @override
     State<CreateEventScreen> createState() => _CreateEventScreenState();
   }

   class _CreateEventScreenState extends State<CreateEventScreen> {
     final _formKey = GlobalKey<FormState>();

     final TextEditingController eventNameController = TextEditingController();
     final TextEditingController descriptionController = TextEditingController();
     final TextEditingController venueController = TextEditingController();
     final TextEditingController maxAttendeesController = TextEditingController();

     DateTime? startDate;
     DateTime? endDate;
     String? eventCategory;
     String eventVisibility = 'Public';
     bool _bannerHovered = false;
     String? _dateError;
     bool _isSubmitting = false;
     int _formResetVersion = 0;
     XFile? _bannerImage;
     final ImagePicker _imagePicker = ImagePicker();
   }

The state class initializes all the controllers and tracking variables 
required to manage the form.

- There are individual ``TextEditingController`` instances for user input.
- Assigned ``DateTime`` objects for scheduling.
- ``ImagePicker`` instance for handling the banner upload.

_resolvedUser
-------------

.. code-block:: dart

      DbUser? _resolvedUser(BuildContext context) {
      final routeUser =
          (ModalRoute.of(context)?.settings.arguments as Map?)?['user'];
      if (routeUser is DbUser) return routeUser;
      return SqliteBackend().currentUser;
     }
     
``_resolvedUser`` helper method safely determines the active user by first checking 
if a user object was passed directly through route arguments before falling back to 
the global SQLite backend state.

initState
^^^^^^^^^

.. code-block:: dart

   @override
   void initState() {
     super.initState();
     final ev = widget.existingEvent;
     if (ev != null) {
       eventNameController.text = ev['title']?.toString() ?? '';
       descriptionController.text = ev['description']?.toString() ?? '';
       venueController.text = ev['location']?.toString() ?? '';
       maxAttendeesController.text = (ev['maxAttendees'] ?? '').toString();
       eventCategory = ev['category']?.toString();
       final dateStr = ev['date']?.toString();
       if (dateStr != null && dateStr.isNotEmpty) {
         try {
           startDate = DateTime.parse(dateStr);
           endDate = ev['endDate'] != null
               ? DateTime.parse(ev['endDate'].toString())
               : startDate?.add(const Duration(hours: 1));
         } catch (_) {}
       }
     }
   }

Because this screen serves dual purposes, the ``initState`` method checks if an 
``existingEvent`` map was passed into the widget at launch. If so, the screen 
operates in "Edit Mode" and immediately pre-populates all text controllers and 
parses the date strings so the organizer can modify their existing data rather 
than starting from scratch.


Form Options & Constants
------------------------

.. code-block:: dart

   final List<String> eventCategories = kEventCategories;

   final List<Map<String, dynamic>> visibilityOptions = [
     {'value': 'Public', 'icon': Icons.public, 'desc': 'Any student'},
     {'value': 'Private', 'icon': Icons.lock_outline, 'desc': 'Invite only'},
   ];

These variables define the static options available for the form's selection inputs. 
``eventCategories`` pulls from a globally defined constant list (``kEventCategories``) 
to ensure category consistency across the entire application. ``visibilityOptions`` 
constructs a rich list of maps, pairing each privacy setting with its corresponding 
UI icon and subtext description to be rendered in the toggle buttons.

Interactive Helper Methods
--------------------------

This section contains the asynchronous functions responsible for handling complex user inputs that require external system dialogues, such as the device's native media gallery or system calendar.

_pickBannerImage
^^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<void> _pickBannerImage() async {
     try {
       final XFile? pickedFile = await _imagePicker.pickImage(
         source: ImageSource.gallery,
         maxWidth: 1200,
         maxHeight: 630,
         imageQuality: 85,
       );

       if (pickedFile != null) {
         setState(() {
           _bannerImage = pickedFile;
         });
       }
     } catch (e) {
       if (!mounted) return;
       ScaffoldMessenger.of(context).showSnackBar(
         SnackBar(
           backgroundColor: Colors.red.shade600,
           behavior: SnackBarBehavior.floating,
           content: Row(
             children: [
               const Icon(Icons.error_outline, color: Colors.white),
               const SizedBox(width: 10),
               Expanded(
                 child: Text('Failed to pick image: $e', style: const TextStyle(color: Colors.white)),
               ),
             ],
           ),
         ),
       );
     }
   }

The ``_pickBannerImage`` function triggers the device's native image gallery 
using the ``image_picker`` plugin. To optimize performance and storage, it 
automatically constraints the selected image to a maximum resolution of 1200x630 
pixels at 85% quality. If the user selects an image, it updates the ``_bannerImage`` 
state; if an error occurs (such as permission denial), it catches the exception and 
displays a styled error ``SnackBar``.

pickDateTime
^^^^^^^^^^^^

.. code-block:: dart

   Future<void> pickDateTime(bool isStart) async {
     final date = await showDatePicker(
       context: context,
       initialDate: DateTime.now(),
       firstDate: DateTime.now(),
       lastDate: DateTime(2100),
     );

     if (date == null) return;

     final time = await showTimePicker(
       context: context,
       initialTime: TimeOfDay.now(),
     );

     if (time == null) return;

     final selected = DateTime(
       date.year, date.month, date.day, time.hour, time.minute,
     );

     setState(() {
       if (isStart) {
         startDate = selected;
         if (endDate != null && selected.isAfter(endDate!)) {
           endDate = null;
           _dateError = 'End date was cleared because it was before the new start date.';
         } else {
           _dateError = null;
         }
       } else {
         if (startDate != null && !selected.isAfter(startDate!)) {
           _dateError = 'End date & time must be after start date & time.';
         } else {
           endDate = selected;
           _dateError = null;
         }
       }
     });
   }

Because Flutter requires separate dialogs for dates and times, ``pickDateTime`` 
chains ``showDatePicker`` and ``showTimePicker`` together sequentially. Once both 
are selected, it merges them into a single ``DateTime`` object. Crucially, this 
function acts as the primary chronological validator: it prevents end dates from 
being set before start dates, and will automatically clear the end date and surface 
a ``_dateError`` message if the user modifies the start date to overlap it.


Form Submission Logic
---------------------

The submission pipeline is the core operational hub of this screen. 
It handles everything from final chronological validation to intercepting 
unauthenticated users and directly committing data to the local SQLite database.

_submitForm
^^^^^^^^^^^

.. code-block:: dart

   Future<void> _submitForm() async {
     setState(() {
       if (startDate == null || endDate == null) {
         _dateError = 'Both start and end date & time are required.';
       } else if (!endDate!.isAfter(startDate!)) {
         _dateError = 'End date & time must be after start date & time.';
       } else {
         _dateError = null;
       }
     });

     if (!_formKey.currentState!.validate() || _dateError != null) return;
     
     // ...
   }

Before any network or database actions occur, the function performs a final 
sanity check. It manually verifies that both dates are present and chronologically 
valid, updating the local ``_dateError`` state if necessary. It then triggers the 
global ``_formKey`` validation (which checks all the text fields for emptiness or 
invalid characters). If any errors exist, the function aborts early.

Guest Draft Interception
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: dart

   final currentUser = _resolvedUser(context);
   final isLoggedIn = currentUser != null;

   if (!isLoggedIn) {
     if (widget.existingEvent == null) {
       final pendingPayload = <String, dynamic>{
         'title': eventNameController.text.trim(),
         'description': descriptionController.text.trim(),
         'location': venueController.text.trim(),
         'category': eventCategory ?? 'Other',
         'visibility': eventVisibility,
         'date': startDate?.toIso8601String() ?? '',
         'maxAttendees': int.tryParse(maxAttendeesController.text.trim()) ?? 0,
       };
       if (_bannerImage != null) {
         try {
           pendingPayload['bannerImageData'] = await _bannerImage!.readAsBytes();
         } catch (_) {}
       }
       SqliteBackend().setPendingEvent(pendingPayload);
     }

     ScaffoldMessenger.of(context).showSnackBar(
       const SnackBar(
         content: Text('Please register or sign in to create an event. Your event draft was saved.'),
       ),
     );

     Future.delayed(const Duration(milliseconds: 500), () {
       if (mounted) {
         Navigator.pushNamed(context, '/register');
       }
     });

     return;
   }

This segment provides a seamless user experience for unauthenticated users. 
If a guest fills out the entire form and clicks submit, they are not presented 
with a hard block. Instead, the screen packages their inputs into a ``pendingPayload`` 
and saves it temporarily in the ``SqliteBackend``. A snackbar informs them that their
draft is safe, and they are automatically routed to the registration screen to 
complete their account creation.

Database Commit
~~~~~~~~~~~~~~~

.. code-block:: dart

   setState(() => _isSubmitting = true);
   try {
     final payload = <String, dynamic>{
       'title': eventNameController.text.trim(),
       'description': descriptionController.text.trim(),
       'location': venueController.text.trim(),
       'category': eventCategory ?? 'Other',
       'visibility': eventVisibility,
       'date': startDate?.toIso8601String() ?? '',
       'endDate': endDate?.toIso8601String() ?? '',
       'maxAttendees': int.tryParse(maxAttendeesController.text.trim()) ?? 0,
       'organizerEmail': currentUser.email,
     };

     if ((payload['organizerEmail'] as String?)?.trim().isEmpty ?? true) {
       throw Exception('Organiser account not available');
     }

     if (_bannerImage != null) {
       payload['bannerImageData'] = await _bannerImage!.readAsBytes();
     }

     if (widget.existingEvent != null) {
       final ok = await SqliteBackend().updateEvent(
         widget.existingEvent!['id'].toString(),
         payload,
       );
       if (!ok) throw Exception('Failed to update event');
     } else {
       await SqliteBackend().createEvent(payload);
     }

     // ... show success snackbar, clear form, and pop route ...
     
   } catch (e) {
     // ... show error snackbar ...
   } finally {
     if (mounted) setState(() => _isSubmitting = false);
   }

For authenticated organizers, this block constructs the final data payload. It 
toggles the UI into a loading state (``_isSubmitting = true``) to prevent duplicate 
submissions. It checks the initial widget arguments: if an ``existingEvent`` is 
present, it issues an ``updateEvent`` command using the existing ID; otherwise, 
it issues a fresh ``createEvent`` command. It uses a ``try/catch/finally`` block 
to safely handle database failures and reset the loading state regardless of the 
outcome.

_clearForm
~~~~~~~~~~

.. code-block:: dart

   void _clearForm() {
     FocusScope.of(context).unfocus();
     eventNameController.text = '';
     descriptionController.text = '';
     venueController.text = '';
     maxAttendeesController.text = '';
     setState(() {
       _formResetVersion++;
       startDate = null;
       endDate = null;
       eventCategory = null;
       eventVisibility = 'Public';
       _dateError = null;
       _bannerImage = null;
       _bannerHovered = false;
     });
   }

The _clearForm method is called after a successful submission. It dismisses the 
keyboard, clears all text controllers, resets local variables, and crucially 
increments ``_formResetVersion``. This version integer is bound to the keys of 
the form fields, ensuring they completely rebuild and clear any lingering validation
error UI.


Role Protection & Locked State
------------------------------

_buildLockedPage
^^^^^^^^^^^^^^^^

.. code-block:: dart

   Widget _buildLockedPage(BuildContext context) {
     return Scaffold(
       backgroundColor: const Color(0xfff5f7fb),
       body: Column(
         children: [
           const AppHeader(
             showProfile: false,
             onHostEventTap: null,
             onFindEventsTap: null,
             onAboutTap: null,
             onSignInTap: null,
           ),
           Expanded(
             child: Center(
               child: ConstrainedBox(
                 constraints: const BoxConstraints(maxWidth: 700),
                 child: Padding(
                   padding: const EdgeInsets.all(24),
                   child: Container(
                     // ... styling ...
                     child: Column(
                       mainAxisSize: MainAxisSize.min,
                       crossAxisAlignment: CrossAxisAlignment.start,
                       children: [
                         Container(
                           child: const Icon(Icons.lock_outline, color: Color(0xFF4F46E5)),
                         ),
                         const SizedBox(height: 18),
                         const Text('Create events is locked', style: TextStyle(fontSize: 24, fontWeight: FontWeight.w800)),
                         const SizedBox(height: 10),
                         Text('Your account is currently set to Attendee. Switch your profile role to Organiser to create events and access organiser calendars.'),
                         const SizedBox(height: 22),
                         Wrap(
                           spacing: 12, runSpacing: 12,
                           children: [
                             FilledButton(
                               onPressed: () => Navigator.pushNamed(context, '/profile'),
                               child: const Text('Open profile'),
                             ),
                             OutlinedButton(
                               onPressed: () => Navigator.pushNamed(context, '/logged-in'),
                               child: const Text('Back to dashboard'),
                             ),
                           ],
                         ),
                       ],
                     ),
                   ),
                 ),
               ),
             ),
           ),
         ],
       ),
     );
   }

This method renders a full-page, access-denied UI for users who do not possess the
"Organiser" role. It features a simplified, read-only ``AppHeader``, a lock icon, 
and explanatory text. It provides two clear escape hatches via routed buttons: one 
to upgrade their profile, and one to return to the main dashboard.


Main Build & Layout
-------------------

build
^^^^^

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     final user = _resolvedUser(context);
     if (user != null && !user.isOrganiser) {
       return _buildLockedPage(context);
     }
     final showApprovalBanner =
         user != null &&
         user.role.toLowerCase() == 'organiser' &&
         user.isApproved == false;

     return Scaffold(
       backgroundColor: const Color(0xfff5f7fb),
       body: LayoutBuilder(
         builder: (context, constraints) {
           final isMobile = constraints.maxWidth < 600;

           return SingleChildScrollView(
             child: ConstrainedBox(
               constraints: BoxConstraints(minHeight: constraints.maxHeight),
               child: Column(
                 mainAxisAlignment: MainAxisAlignment.spaceBetween,
                 children: [
                   Column(
                     children: [
                       // ... AppHeader ...
                       Center(
                         child: Column(
                           children: [
                             // ... Approval Banner ...
                             // ... Title & Subtitle ...
                             // ... Main Form ...
                           ]
                         )
                       )
                     ]
                   ),
                   const AppFooter(),
                 ]
               )
             )
           );
         }
       )
     );
   }

The primary ``build`` method handles the overarching screen layout and access 
control. It immediately evaluates the ``_resolvedUser``. If the user is an 
attendee, it short-circuits the build process and returns ``_buildLockedPage``. 
For authorized organizers, it wraps the interface in a responsive ``LayoutBuilder`` 
to dynamically adjust margins and layouts (e.g., stacking inputs vs. displaying them 
side-by-side) based on the ``isMobile`` boolean constraint.

AppHeader
~~~~~~~~~

.. code-block:: dart

   AppHeader(
     showProfile: true,
     onFindEventsTap: () => Navigator.pushNamed(context, '/discover'),
     onCreateEventsTap: () {}, // Current page, no route needed
     onMyTicketsTap: () => Navigator.pushNamed(context, '/my-tickets'),
     onAboutTap: () => Navigator.pushNamed(context, '/about'),
     onSignInTap: () => Navigator.pushNamed(context, '/login'),
     onHostEventTap: () {}, 
   )

The active view utilizes the fully interactive ``AppHeader``. Notice that 
``onCreateEventsTap`` and ``onHostEventTap`` are assigned empty callbacks; 
because the user is already on the event creation screen, clicking these links 
acts as a safe no-op to prevent redundant routing.

Pending Approval Banner
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: dart

   if (showApprovalBanner)
     Container(
       width: double.infinity,
       color: Colors.orange.shade100,
       padding: const EdgeInsets.symmetric(vertical: 12, horizontal: 16),
       margin: const EdgeInsets.only(bottom: 12),
       child: Row(
         children: [
           const Icon(Icons.info_outline, color: Colors.orange),
           const SizedBox(width: 10),
           Expanded(
             child: Text(
               'Organizer approval pending',
               style: TextStyle(color: Colors.orange.shade900, fontWeight: FontWeight.w600),
             ),
           ),
         ],
       ),
     ),

This conditional UI block alerts the user if their newly created organizer account 
has not yet been approved by an administrator. This ensures transparency, as pending 
organizers can draft events, but those events might not be visible to the public 
until their account is fully verified.

UI Builder Utilities
--------------------

_label, _textField, & _datePicker
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Widget _label(String text) {
     return Padding(
       padding: const EdgeInsets.only(bottom: 8),
       child: Text(text, style: const TextStyle(fontWeight: FontWeight.w600)),
     );
   }

   Widget _textField({
     required TextEditingController controller,
     required String hint,
     String? label,
     int maxLines = 1,
     TextInputType keyboardType = TextInputType.text,
     String? Function(String?)? validator,
   }) {
     return Padding(
       padding: const EdgeInsets.only(bottom: 16),
       child: TextFormField(
         key: ValueKey('${controller.hashCode}_$_formResetVersion'),
         controller: controller,
         maxLines: maxLines,
         keyboardType: keyboardType,
         validator: validator,
         decoration: InputDecoration(
           // ... decoration styling ...
         ),
       ),
     );
   }

   Widget _datePicker({
    required String label,
    required DateTime? value,
    required VoidCallback onTap,
  }) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        _label(label),
        InkWell(
          onTap: onTap,
          child: Container(
            height: 56,
            padding: const EdgeInsets.symmetric(horizontal: 16),
            decoration: BoxDecoration(
              color: const Color(0xFFF7F9FC),
              borderRadius: BorderRadius.circular(12),
            ),
            alignment: Alignment.centerLeft,
            child: Text(
              value == null
                  ? 'dd/mm/yyyy --:--'
                  : '${value.day.toString().padLeft(2, '0')}/${value.month.toString().padLeft(2, '0')}/${value.year} ${value.hour.toString().padLeft(2, '0')}:${value.minute.toString().padLeft(2, '0')}',
              style: const TextStyle(color: Colors.black54),
            ),
          ),
        ),
      ],
    );
  }

To prevent the main ``build`` method from becoming unreadable, the screen abstracts 
repetitive form elements into distinct UI builder utilities. 

A critical feature here is the ``ValueKey`` applied to the ``TextFormField`` 
inside ``_textField``. By appending the ``_formResetVersion`` state integer to the 
controller's hashcode, the app forces Flutter to completely tear down and rebuild 
the text fields when the form is cleared, guaranteeing that any red validation error 
text is successfully wiped from the screen.