==================
calendar page - calendar_page.dart
==================

``calendar_page.dart`` provides a weekly visual planner exclusively for event 
organizers. It allows authenticated organizers to view their scheduled events 
mapped out on a 24-hour weekly grid, helping them manage their time and event 
overlaps effectively.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import '../services/sqlite_backend.dart';
   import '../widgets/header.dart';
   import '../widgets/app_footer.dart';

These imports pull in the core Flutter framework, the local SQLite database for 
fetching the active user and their associated events, and the shared global header 
and footer widgets to maintain layout consistency.

State & Date Navigation
-----------------------

.. code-block:: dart

   class CalendarPage extends StatefulWidget {
     const CalendarPage({super.key});

     @override
     State<CalendarPage> createState() => _CalendarPageState();
   }

   class _CalendarPageState extends State<CalendarPage> {
     DateTime _currentWeekStart = _getStartOfWeek(DateTime(2026, 5, 17));

     static DateTime _getStartOfWeek(DateTime date) {
       return date.subtract(Duration(days: date.weekday % 7));
     }

     void _goToPreviousWeek() {
       setState(() {
         _currentWeekStart = _currentWeekStart.subtract(const Duration(days: 7));
       });
     }

     void _goToNextWeek() {
       setState(() {
         _currentWeekStart = _currentWeekStart.add(const Duration(days: 7));
       });
     }
     
     // ... build methods ...
   }

The state class manages the temporal context of the calendar. It initializes the 
``_currentWeekStart`` to a hardcoded baseline date (May 17, 2026) for demonstration 
purposes, automatically snapping it to the nearest Sunday using the ``_getStartOfWeek`` 
math. The two helper functions, ``_goToPreviousWeek`` and ``_goToNextWeek``, trigger 
a state rebuild by safely subtracting or adding exactly 7 days to the current week's 
start date, allowing the user to paginate through time.

Main Build Method
-----------------

.. code-block:: dart

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
                       // ... AppHeader Navigation ...
                       if (!isOrganiser)
                         _buildLockedState(context)
                       else
                         _buildCalendarView(context, constraints),
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

The primary ``build`` method establishes the responsive layout and enforces 
role-based access. It immediately fetches the ``currentUser`` and evaluates the 
``isOrganiser`` flag. If the user is a standard attendee, it routes them to the 
locked state UI; otherwise, it permits the rendering of the full calendar view.

AppHeader Navigation
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   AppHeader(
     onHostEventTap: () => Navigator.pushNamed(context, '/create-event'),
     onFindEventsTap: () => Navigator.pushNamed(context, '/discover'),
     onCreateEventsTap: () => Navigator.pushNamed(context, '/create-event'),
     onMyTicketsTap: () => Navigator.pushNamed(context, '/my-tickets'),
     onAboutTap: () => Navigator.pushNamed(context, '/about'),
     onSignInTap: () => Navigator.pushNamed(context, '/login'),
   )

This segment configures the global application header for the Calendar page. 
It binds anonymous callback functions to each of the header's interaction points, 
utilizing Flutter's named route navigator to seamlessly push the user to the 
respective screens without requiring complex state management.

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
             // ... container styling ...
             child: Column(
               mainAxisSize: MainAxisSize.min,
               crossAxisAlignment: CrossAxisAlignment.start,
               children: [
                 Container(
                   // ... icon styling ...
                   child: const Icon(Icons.lock_outline, color: Color(0xFF4F46E5), size: 32),
                 ),
                 const SizedBox(height: 24),
                 const Text('Organiser calendar locked', ...),
                 const SizedBox(height: 12),
                 Text('Switch your profile to Organiser to view the calendar for events you create...', ...),
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

If the user is not an organizer, this method renders a friendly, access-denied UI. 
It clearly explains the restriction and provides a direct call-to-action button that 
routes the user to their profile settings, encouraging them to update their account 
role to unlock the calendar feature.

The Calendar View
-----------------

.. code-block:: dart

   Widget _buildCalendarView(BuildContext context, BoxConstraints constraints) {
     final daysOfWeek = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
     final weekDates = List.generate(
       7,
       (i) => _currentWeekStart.add(Duration(days: i)),
     );
     final hours = List.generate(24, (i) => i);
     final isMobile = constraints.maxWidth < 800;

     final email = SqliteBackend().currentUser?.email;
     final userEvents = SqliteBackend().events
         .where((e) => e['organizerEmail']?.toString() == email)
         .toList();

     return Padding(
       padding: EdgeInsets.symmetric(horizontal: isMobile ? 16 : 32, vertical: 32),
       child: Center(
         child: ConstrainedBox(
           constraints: const BoxConstraints(maxWidth: 1100),
           child: Column(
             crossAxisAlignment: CrossAxisAlignment.start,
             children: [
               // ... Breadcrumbs ...
               
               Container(
                 // ... Main Calendar Container Styling ...
                 child: Column(
                   children: [
                     // ... Header & Controls ...
                     // ... Days Row ...
                     // ... Hours Grid ...
                   ]
                 )
               )
             ]
           )
         )
       )
     );
   }

This method serves as the entry point for the authenticated organizer's dashboard. 
Before rendering any UI, it calculates the arrays needed to build the grid: the 
7 specific ``DateTime`` objects for the current week, and a 24-integer array 
representing the hours of the day. It also performs a local database query to 
fetch only the events belonging to the currently logged-in organizer.

Calendar Header & Controls
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Padding(
     padding: const EdgeInsets.fromLTRB(24, 24, 24, 16),
     child: Row(
       mainAxisAlignment: MainAxisAlignment.spaceBetween,
       children: [
         Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             const Text('Organiser Calendar', style: TextStyle(fontSize: 24, fontWeight: FontWeight.w800)),
             const SizedBox(height: 4),
             Text(
               '${_monthName(weekDates.first.month)} ${weekDates.first.year}',
               style: TextStyle(fontSize: 14, color: Colors.grey.shade600),
             ),
           ],
         ),
         Row(
           children: [
             IconButton(
               icon: const Icon(Icons.chevron_left_rounded),
               onPressed: _goToPreviousWeek,
               // ... styling ...
             ),
             const SizedBox(width: 8),
             IconButton(
               icon: const Icon(Icons.chevron_right_rounded),
               onPressed: _goToNextWeek,
               // ... styling ...
             ),
           ],
         ),
       ],
     ),
   )

Sitting at the top of the calendar container, this segment displays the current 
month and year based on the active ``_currentWeekStart``. It wires up the chevron 
icon buttons directly to the ``_goToPreviousWeek`` and ``_goToNextWeek`` state 
methods, allowing the user to navigate backward and forward in time.

Days Row & Today Highlight
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Container(
     color: const Color(0xFFF9FAFB),
     padding: const EdgeInsets.only(left: 60), // Offset for the hours column
     child: Row(
       children: List.generate(7, (i) {
         // Mock highlight logic
         final isToday = weekDates[i].day == 20 && weekDates[i].month == 5; 
         
         return Expanded(
           child: Container(
             padding: const EdgeInsets.symmetric(vertical: 16),
             // ... borders ...
             child: Column(
               children: [
                 Text(daysOfWeek[i], ...),
                 const SizedBox(height: 4),
                 Container(
                   width: 32, height: 32,
                   decoration: BoxDecoration(
                     color: isToday ? const Color(0xFF4F46E5) : Colors.transparent,
                     shape: BoxShape.circle,
                   ),
                   child: Center(
                     child: Text('${weekDates[i].day}', ...),
                   ),
                 ),
               ],
             ),
           ),
         );
       }),
     ),
   )

This sub-component generates the 7 horizontal columns representing the days of the 
week. It utilizes an offset padding (``left: 60``) to align perfectly with the hours 
grid below it. For demonstration, it includes hardcoded logic to highlight "May 20th" 
as the active day, wrapping the date integer in a distinct, filled circle to visually 
orient the user.

Hours Grid & Event Matching
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   SizedBox(
     height: 600,
     child: ListView.builder(
       itemCount: hours.length,
       itemBuilder: (context, hourIdx) {
         final hour = hours[hourIdx];
         return SizedBox(
           height: 80,
           child: Row(
             children: [
               // Time Label (e.g., "09:00")
               SizedBox(
                 width: 60,
                 child: Align(
                   alignment: Alignment.topRight,
                   child: Text('${hour.toString().padLeft(2, '0')}:00', ...),
                 ),
               ),
               // 7 Day Slots for this Hour
               ...List.generate(7, (dayIdx) {
                 final dayDate = weekDates[dayIdx];

                 // Find events overlapping this specific hour block
                 final matchingEvents = userEvents.where((e) {
                   try {
                     final start = DateTime.parse(e['date'].toString());
                     final end = e['endDate'] != null
                         ? DateTime.parse(e['endDate'].toString())
                         : start.add(const Duration(hours: 1));

                     final slotStart = DateTime(dayDate.year, dayDate.month, dayDate.day, hour);
                     final slotEnd = slotStart.add(const Duration(hours: 1));

                     return (start.isBefore(slotEnd) && end.isAfter(slotStart));
                   } catch (_) {
                     return false;
                   }
                 }).toList();

                 return Expanded(
                   child: Container(
                     decoration: const BoxDecoration(...), // Cell borders
                     child: Stack(
                       children: matchingEvents.map((ev) {
                         return _buildEventBlock(ev, hour, dayDate);
                       }).toList(),
                     ),
                   ),
                 );
               }),
             ],
           ),
         );
       },
     ),
   )

This is the most complex logical segment of the screen. It builds a scrollable vertical 
list of 24 rows, one for each hour of the day. For every individual cell (an intersection 
of an hour and a day), it filters the ``userEvents`` list, parsing their ISO string 
start and end times. If an event's duration overlaps with that specific 1-hour slot, 
it is added to the ``matchingEvents`` list and rendered onto the grid using a 
``Stack``.

Event Block Component
---------------------

.. code-block:: dart

   Widget _buildEventBlock(Map<String, dynamic> event, int currentHour, DateTime currentDay) {
     return Padding(
       padding: const EdgeInsets.all(2.0),
       child: Container(
         width: double.infinity,
         decoration: BoxDecoration(
           color: const Color(0xFFEEF2FF).withOpacity(0.9),
           border: Border.all(color: const Color(0xFF818CF8)),
           borderRadius: BorderRadius.circular(8),
         ),
         padding: const EdgeInsets.all(6),
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Text(
               event['title']?.toString() ?? 'Event',
               maxLines: 1,
               overflow: TextOverflow.ellipsis,
               // ... typography ...
             ),
             if (event['location'] != null)
               Text(
                 event['location'].toString(),
                 maxLines: 1,
                 overflow: TextOverflow.ellipsis,
                 // ... typography ...
               ),
           ],
         ),
       ),
     );
   }

The ``_buildEventBlock`` is a dedicated method for rendering the visual representations 
of scheduled events. It outputs a distinctly colored bounding box containing the 
event's title and location. It utilizes strict padding and text-overflow parameters 
to ensure the block does not visually break out of its designated grid slot.

Month Name Helper
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   String _monthName(int month) {
     const months = [
       '', 
       'January', 
       'February', 
       'March', 
       'April', 
       'May', 
       'June',
       'July', 
       'August', 
       'September', 
       'October', 
       'November', 
       'December'
     ];
     return months[month];
   }

A simple data utility function that converts a numeric ``DateTime`` month integer 
(1-12) into its corresponding localized string name for use in the calendar header.