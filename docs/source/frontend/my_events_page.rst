===================
My events - my_events_page.dart
===================

``my_events_page.dart`` serves as the central dashboard for event organizers. It 
acts as a strict, role-gated environment where authenticated organizers can view, 
edit, and delete the events they have created.

Imports
-------

.. code-block:: dart

   import 'dart:typed_data';
   import 'package:flutter/material.dart';
   import '../services/sqlite_backend.dart';
   import 'create_event_screen.dart';
   import 'event_details_screen.dart';
   import '../utils/pagination.dart';
   import '../widgets/header.dart';
   import '../widgets/app_footer.dart';
   import '../widgets/pagination_controls.dart';

These imports provide the foundational building blocks for the dashboard. They 
pull in Flutter's material library, image data handling, the SQLite backend for 
fetching user and event data, the pagination utilities, and the routing targets 
for creating or viewing events.

State Management & Navigation Logic
-----------------------------------

.. code-block:: dart

   class MyEventsPage extends StatefulWidget {
     const MyEventsPage({super.key});

     @override
     State<MyEventsPage> createState() => _MyEventsPageState();
   }

   class _MyEventsPageState extends State<MyEventsPage> {
     int _currentPage = 0;

     void _changePage(int nextPage) {
       setState(() {
         _currentPage = nextPage;
       });
     }

     @override
     Widget build(BuildContext context) {
       final user = SqliteBackend().currentUser;
       final isOrganiser = user?.isOrganiser ?? false;

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
                         AppHeader(...),
                         if (!isOrganiser)
                           _buildLockedState(context)
                         else
                           _buildEventsList(context, constraints),
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
     }
     
     // ... child builder methods ...
   }

The state class manages the current pagination index (``_currentPage``) for the 
events list. The core logic of the screen begins immediately inside the main ``build`` 
method, where it fetches the currently logged-in user from the database. It evaluates
the ``isOrganiser`` boolean flag, which acts as a strict access gate. The UI 
dynamically branches: standard attendees are routed to the locked state, while 
organizers are permitted to view the events list.

The Locked State
----------------

.. code-block:: dart

   Widget _buildLockedState(BuildContext context) {
     return Padding(
       padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 60),
       child: Center(
         child: ConstrainedBox(
           constraints: const BoxConstraints(maxWidth: 560),
           child: Container(
             // ... container styling and shadows ...
             child: Column(
               mainAxisSize: MainAxisSize.min,
               crossAxisAlignment: CrossAxisAlignment.start,
               children: [
                 Container(
                   // ... lock icon container ...
                   child: const Icon(Icons.lock_outline, color: Color(0xFF4F46E5), size: 32),
                 ),
                 const SizedBox(height: 24),
                 const Text('Organiser only access', ...),
                 const SizedBox(height: 12),
                 Text('Switch your profile to Organiser to manage your created events and see their performance.', ...),
                 const SizedBox(height: 32),
                 SizedBox(
                   width: double.infinity,
                   height: 52,
                   child: FilledButton(
                     onPressed: () => Navigator.pushNamed(context, '/profile'),
                     child: const Text('Open profile'),
                   ),
                 ),
               ],
             ),
           ),
         ),
       ),
     );
   }

If the user does not possess the organizer role, this method renders a clean, 
informative access-denied UI. It displays a prominent lock icon alongside an 
explanatory message. Rather than leaving the user stranded, it provides a primary 
call-to-action button that routes them directly to their profile settings, 
encouraging them to upgrade their account type to unlock event management 
capabilities.

Events List & Reactivity
------------------------

.. code-block:: dart

   Widget _buildEventsList(BuildContext context, BoxConstraints constraints) {
     final isMobile = constraints.maxWidth < 800;

     return AnimatedBuilder(
       animation: SqliteBackend(),
       builder: (context, _) {
         final email = SqliteBackend().currentUser?.email;
         
         // Filter database for this specific organizer's events
         final myEvents = SqliteBackend().events
             .where((event) =>
                 event['organizerEmail']?.toString() == email &&
                 event['title']?.toString().toLowerCase() != 'demo event')
             .toList();
             
         final itemsPerPage = eventsPerPageForWidth(constraints.maxWidth);
         final totalPages = totalPagesForLength(myEvents.length, itemsPerPage);
         final pageIndex = clampPageIndex(_currentPage, totalPages);
         final pageEvents = paginateItems(myEvents, pageIndex, itemsPerPage);

         // ... safety state update for pagination bounds ...

         return Padding(
           padding: EdgeInsets.symmetric(horizontal: isMobile ? 16 : 32, vertical: 32),
           child: Center(
             child: ConstrainedBox(
               constraints: const BoxConstraints(maxWidth: 1000),
               child: Column(
                 crossAxisAlignment: CrossAxisAlignment.start,
                 children: [
                   // ... Breadcrumb Navigation ...
                   
                   Row(
                     mainAxisAlignment: MainAxisAlignment.spaceBetween,
                     children: [
                       const Text('My Events', ...),
                       if (!isMobile)
                         FilledButton.icon(
                           onPressed: () => Navigator.push(
                             context,
                             MaterialPageRoute(builder: (_) => const CreateEventScreen()),
                           ),
                           icon: const Icon(Icons.add, size: 20),
                           label: const Text('Create Event'),
                         ),
                     ],
                   ),
                   const SizedBox(height: 32),
                   
                   if (myEvents.isEmpty)
                     _buildEmptyState(context)
                   else
                     Column(
                       children: [
                         ListView.separated(
                           shrinkWrap: true,
                           physics: const NeverScrollableScrollPhysics(),
                           itemCount: pageEvents.length,
                           separatorBuilder: (context, index) => const SizedBox(height: 20),
                           itemBuilder: (context, index) {
                             return _buildEventCard(context, pageEvents[index]);
                           },
                         ),
                         PaginationControls(
                           currentPage: pageIndex,
                           totalPages: totalPages,
                           onPrevious: () => _changePage(_currentPage - 1),
                           onNext: () => _changePage(_currentPage + 1),
                         ),
                       ],
                     ),
                 ],
               ),
             ),
           ),
         );
       },
     );
   }

For authenticated organizers, this method wraps the main content in an 
``AnimatedBuilder`` attached to the ``SqliteBackend``. This ensures real-time 
UI updates whenever events are added, edited, or deleted elsewhere in the app. 
It explicitly filters the global database for events matching the organizer's email, 
calculates the required pagination slices based on screen width, and constructs the main dashboard layout, complete with a prominent "Create Event" call-to-action.

Empty State
-----------

.. code-block:: dart

   Widget _buildEmptyState(BuildContext context) {
     return Center(
       child: Column(
         children: [
           const SizedBox(height: 60),
           Icon(Icons.event_note_outlined, size: 80, color: Colors.grey.shade300),
           const SizedBox(height: 24),
           const Text('No events created yet', ...),
           const SizedBox(height: 12),
           Text('Start your journey as an organiser by launching your first event.', ...),
           const SizedBox(height: 32),
           FilledButton(
             onPressed: () => Navigator.push(
               context,
               MaterialPageRoute(builder: (_) => const CreateEventScreen()),
             ),
             child: const Text('Create your first event'),
           ),
         ],
       ),
     );
   }

If the database query returns zero events for the active organizer, the screen safely 
falls back to this placeholder UI. It displays a calendar icon and an encouraging 
call-to-action button that routes the user directly to the ``CreateEventScreen`` 
to begin their onboarding flow.

Event Card & Action Handlers
----------------------------

.. code-block:: dart

   Widget _buildEventCard(BuildContext context, Map<String, dynamic> event) {
     final String dateStr = event['date']?.toString() ?? '';
     final String endDateStr = event['endDate']?.toString() ?? '';
     // ... extract title, location, category, and bannerBytes ...

     return Container(
       // ... card styling and shadows ...
       child: Material(
         color: Colors.transparent,
         child: InkWell(
           onTap: () {
             Navigator.push(
               context,
               MaterialPageRoute(
                 builder: (context) => EventDetailsScreen(event: event, allowPurchase: false),
               ),
             );
           },
           borderRadius: BorderRadius.circular(20),
           child: Padding(
             padding: const EdgeInsets.all(20),
             child: Row(
               children: [
                 // Banner thumbnail or category fallback icon
                 ClipRRect(
                   borderRadius: BorderRadius.circular(16),
                   child: Container(
                     width: 92, height: 72,
                     color: const Color(0xFFF3F4F6),
                     child: bannerBytes != null
                         ? Image.memory(bannerBytes, fit: BoxFit.cover)
                         : Icon(_getCategoryIcon(category), color: const Color(0xFF4F46E5)),
                   ),
                 ),
                 const SizedBox(width: 20),
                 
                 // Main Event Info
                 Expanded(
                   child: Column(
                     crossAxisAlignment: CrossAxisAlignment.start,
                     children: [
                       Row(
                         children: [
                           Container(
                             child: Text(category.toUpperCase(), ...),
                           ),
                           const SizedBox(width: 8),
                           Text(_formatDate(dateStr, endDateStr), ...),
                         ],
                       ),
                       Text(title, ...),
                       Row(
                         children: [
                           Icon(Icons.location_on_outlined, ...),
                           Expanded(child: Text(location, ...)),
                         ],
                       ),
                     ],
                   ),
                 ),
                 
                 // Action Buttons (Edit & Delete)
                 Row(
                   mainAxisSize: MainAxisSize.min,
                   children: [
                     IconButton(
                       onPressed: () {
                         Navigator.push(
                           context,
                           MaterialPageRoute(
                             builder: (context) => CreateEventScreen(existingEvent: event),
                           ),
                         );
                       },
                       icon: const Icon(Icons.edit_outlined),
                       tooltip: 'Edit Event',
                     ),
                     const SizedBox(width: 8),
                     IconButton(
                       onPressed: () => _showDeleteConfirmation(context, event),
                       icon: const Icon(Icons.delete_outline_rounded),
                       tooltip: 'Delete Event',
                     ),
                   ],
                 ),
               ],
             ),
           ),
         ),
       ),
     );
   }

   void _showDeleteConfirmation(BuildContext context, Map<String, dynamic> event) {
     // ... displays AlertDialog and calls SqliteBackend().deleteEvent() ...
   }

   IconData _getCategoryIcon(String category) {
     // ... switch statement returning specific icons based on category string ...
   }

   String _formatDate(String isoString, String endIsoString) {
     // ... safely parses ISO strings into readable "Day Month, Year • HH:MM" formats ...
   }

The ``_buildEventCard`` method constructs a rich, interactive list item for each 
created event. It extracts the event's details, handling custom banners (falling 
back to ``_getCategoryIcon`` if no image exists) and formatted timestamps (via 
``_formatDate``). 

Crucially, it provides three distinct user interactions:
1. **Preview:** Tapping the card routes the user to a read-only view of the ``EventDetailsScreen``.
2. **Edit:** Tapping the edit icon routes the user to the ``CreateEventScreen``, passing the current event payload forward to pre-populate the form fields.
3. **Delete:** Tapping the delete icon triggers ``_showDeleteConfirmation``, an asynchronous dialog that safely removes the event from the database upon user confirmation.