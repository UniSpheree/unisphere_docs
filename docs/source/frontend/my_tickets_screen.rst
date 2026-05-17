======================
My tickets page - my_tickets_screen.dart
======================

``my_tickets_screen.dart`` manages the user's digital ticket wallet. It acts as
a personalized inventory, integrating local SQLite storage with live event data 
to ensure attendees always have access to their up-to-date purchased tickets and 
event information.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'dart:typed_data';
   import 'package:unisphere_app/screens/event_details_screen.dart';
   import 'package:unisphere_app/services/sqlite_backend.dart';
   import 'package:unisphere_app/models/database_models.dart';
   import 'package:unisphere_app/utils/pagination.dart';
   import 'package:unisphere_app/widgets/header.dart';
   import 'package:unisphere_app/widgets/app_footer.dart';
   import 'package:unisphere_app/widgets/pagination_controls.dart';

These imports gather the necessary tools to build the ticket wallet. They include 
Flutter's material design library, typed data for image handling, the local 
database models and services for fetching tickets, pagination utilities for 
handling large lists, and the shared global widgets (header, footer, pagination 
controls) to maintain consistent app navigation.

State Management & Helper Methods
---------------------------------

.. code-block:: dart

   class MyTicketsScreen extends StatefulWidget {
     const MyTicketsScreen({super.key});

     @override
     State<MyTicketsScreen> createState() => _MyTicketsScreenState();
   }

   class _MyTicketsScreenState extends State<MyTicketsScreen> {
     final TextEditingController _searchController = TextEditingController();
     String _query = '';
     int _currentPage = 0;

     Map<String, dynamic>? _findMatchingEvent(DbPurchasedTicket ticket) {
       // ... cross-references ticket ID or details with live events ...
     }

     bool _hasExistingEvent(DbPurchasedTicket ticket) {
       return _findMatchingEvent(ticket) != null;
     }

     @override
     void dispose() {
       _searchController.dispose();
       super.dispose();
     }

     Map<String, dynamic> _ticketToEvent(DbPurchasedTicket ticket) {
       // ... maps a flat ticket record into a rich event data map ...
     }

     bool _matchesQuery(DbPurchasedTicket ticket) {
       // ... performs case-insensitive search across ticket fields ...
     }
     
     // ... build method ...
   }

The state class for the tickets screen primarily manages the search input 
tracking (``_query`` and ``_searchController``) and the current pagination index 
(``_currentPage``). Because the database stores lightweight ticket receipts that 
may lack full event details (like updated banners or organizer info), the widget 
utilizes helper methods like ``_findMatchingEvent`` and ``_ticketToEvent`` to 
dynamically cross-reference ticket records against the live ``events`` table. 
This ensures the user's ticket wallet always displays the most current information. 
Additionally, ``_matchesQuery`` provides robust text filtering across all ticket 
attributes.

Main Layout & Reactivity
------------------------

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
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
                       SafeArea(
                         child: AnimatedBuilder(
                           animation: SqliteBackend(),
                           builder: (context, _) {
                             // 1. Fetch valid tickets
                             final tickets = SqliteBackend().purchasedTickets.where(_hasExistingEvent).toList();
                             
                             // 2. Apply search filter
                             final filteredTickets = tickets.where((t) => _matchesQuery(t) && t.title.toLowerCase() != 'demo event').toList();
                             
                             // 3. Calculate Pagination
                             final itemsPerPage = eventsPerPageForWidth(constraints.maxWidth);
                             final totalPages = totalPagesForLength(filteredTickets.length, itemsPerPage);
                             final pageIndex = clampPageIndex(_currentPage, totalPages);
                             final pageTickets = paginateItems(filteredTickets, pageIndex, itemsPerPage);

                             // ... safely update state if page bounds shifted ...

                             // ... build UI ...
                           }
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

The core layout utilizes a responsive ``LayoutBuilder`` to adapt to different 
screen sizes. Crucially, the main content area is wrapped in an ``AnimatedBuilder`` 
that listens directly to the ``SqliteBackend``. This reactive architecture ensures 
that whenever a ticket is bought or deleted elsewhere in the app, the ticket wallet 
updates instantly without requiring a manual page refresh. Inside the builder, the 
widget dynamically filters out invalid/demo tickets, applies the active search 
query, and calculates the precise mathematical slice of tickets to display based 
on the current screen width and page index.

Header Banner & Search Interface
--------------------------------

.. code-block:: dart

   // ... Breadcrumb Navigation ...

   Container(
     width: double.infinity,
     padding: const EdgeInsets.all(18),
     decoration: BoxDecoration(
       gradient: const LinearGradient(
         colors: [Color(0xFF4F46E5), Color(0xFF6D79FF)],
         begin: Alignment.topLeft,
         end: Alignment.bottomRight,
       ),
       borderRadius: BorderRadius.circular(18),
       // ... shadow styling ...
     ),
     child: Column(
       crossAxisAlignment: CrossAxisAlignment.start,
       children: [
         const Text('My Tickets', style: TextStyle(color: Colors.white, fontSize: 24, fontWeight: FontWeight.w800)),
         // ... conditional subtitle ...
         const SizedBox(height: 16),
         Container(
           // ... Search Bar Styling ...
           child: TextField(
             controller: _searchController,
             onChanged: (value) {
               setState(() {
                 _query = value;
                 _currentPage = 0; // Reset to first page on new search
               });
             },
             style: const TextStyle(color: Colors.white),
             decoration: InputDecoration(
               hintText: 'Search by event, date, venue, or category',
               prefixIcon: Icon(Icons.search_rounded, color: Colors.white.withOpacity(0.78)),
               suffixIcon: _query.isEmpty ? null : IconButton(
                 onPressed: () {
                   _searchController.clear();
                   setState(() => _query = '');
                 },
                 icon: Icon(Icons.close_rounded, color: Colors.white.withOpacity(0.78)),
               ),
               // ...
             ),
           ),
         ),
       ],
     ),
   )

The top section of the ticket wallet features a visually prominent, gradient-styled 
hero banner. This banner houses the primary search interface. The ``TextField`` is 
directly wired to update the ``_query`` state on every keystroke, which simultaneously 
forces the pagination back to the first page to prevent out-of-bounds errors. 
It also includes intuitive UI touches, such as a conditional "clear" button 
(an 'X' icon) that only appears when text is actively entered in the search bar.

Empty States & List Rendering
-----------------------------

.. code-block:: dart

   if (tickets.isEmpty) {
     children.add(
       Container(
         // ... Global Empty State UI ...
         child: Column(
           children: [
             Icon(Icons.confirmation_number_outlined, ...),
             const Text('No tickets saved yet', ...),
             const Text('Go back and browse events to buy one.', ...),
           ],
         ),
       ),
     );
   } else if (filteredTickets.isEmpty) {
     children.add(
       Container(
         // ... Search "No Matches" UI ...
         child: Column(
           children: [
             Icon(Icons.search_off_rounded, ...),
             const Text('No matching tickets', ...),
             const Text('Try a different keyword, venue, or category.', ...),
           ],
         ),
       ),
     );
   } else {
     for (final ticket in pageTickets) {
       children.add(
         Padding(
           padding: const EdgeInsets.only(bottom: 10),
           child: _SavedTicketCard(
             ticket: ticket,
             eventData: _ticketToEvent(ticket),
             onDelete: () async {
               final confirmed = await showDialog<bool>(...); // Show alert dialog
               if (confirmed != true) return;
               
               final ok = await SqliteBackend().deleteTicket(ticket.id?.toString() ?? '');
               // ... show success/failure snackbar ...
             },
             onTap: () {
               Navigator.push(
                 context,
                 MaterialPageRoute(
                   builder: (_) => EventDetailsScreen(
                     event: _ticketToEvent(ticket),
                     allowPurchase: false, // Read-only mode
                   ),
                 ),
               );
             },
           ),
         ),
       );
     }

     children.add(PaginationControls(...));
   }

To provide a clear user experience, the screen handles two distinct empty states: 
an overall empty state prompting the user to buy tickets if their wallet is 
completely empty, and a specific "no match" state if their active search query 
yields zero results. If valid tickets exist, the screen iterates through the 
calculated ``pageTickets`` slice, rendering individual ``_SavedTicketCard`` widgets. 
It also attaches the asynchronous logic required to delete a ticket (complete with 
a confirmation dialog) and handles routing to the ``EventDetailsScreen`` in a 
read-only state.

_SavedTicketCard
----------------

.. code-block:: dart

   class _SavedTicketCard extends StatelessWidget {
     final DbPurchasedTicket ticket;
     final Map<String, dynamic> eventData;
     final VoidCallback onDelete;
     final VoidCallback onTap;

     const _SavedTicketCard({
       required this.ticket,
       required this.eventData,
       required this.onDelete,
       required this.onTap,
     });

     @override
     Widget build(BuildContext context) {
       final bannerData = eventData['bannerImageData'];
       final Uint8List? bannerBytes = bannerData is Uint8List ? bannerData : null;
       final organizer = eventData['organizer']?.toString() ?? 'UniSphere';

       return InkWell(
         onTap: onTap,
         borderRadius: BorderRadius.circular(14),
         child: Container(
           // ... container styling and shadows ...
           child: Row(
             crossAxisAlignment: CrossAxisAlignment.start,
             children: [
               // Leading image or fallback icon
               bannerBytes != null
                   ? ClipRRect(...) 
                   : Container(...), 
               
               const SizedBox(width: 16),
               
               // Main Content
               Expanded(
                 child: Column(
                   crossAxisAlignment: CrossAxisAlignment.start,
                   children: [
                     Text(ticket.title, ...),
                     Text('${ticket.date} • ${ticket.location}', ...),
                     Text('By $organizer', ...),
                     Container(
                       // Category Badge
                       child: Text(ticket.category, ...),
                     ),
                   ],
                 ),
               ),
               
               const SizedBox(width: 12),
               
               // Trailing price and actions
               Column(
                 crossAxisAlignment: CrossAxisAlignment.end,
                 children: [
                   Text(ticket.price, ...),
                   const SizedBox(height: 12),
                   Row(
                     mainAxisSize: MainAxisSize.min,
                     children: [
                       IconButton(
                         onPressed: onDelete,
                         icon: const Icon(Icons.delete_outline_rounded),
                         color: const Color(0xFFDC2626),
                         // ... styling ...
                       ),
                       const SizedBox(width: 6),
                       const Icon(Icons.arrow_forward_ios_rounded, ...),
                     ],
                   ),
                 ],
               ),
             ],
           ),
         ),
       );
     }
   }

The ``_SavedTicketCard`` is a self-contained, stateless UI component responsible 
for displaying individual ticket data within the list. It features a flexible row 
layout that displays the event's banner image (or a fallback icon if missing), 
crucial event details, and a dynamic category badge. It surfaces the ticket price 
prominently on the right side, directly above the delete action button and navigation 
indicator, ensuring all critical information and actions are immediately accessible 
to the user.