===========================
landing_page_logged_in.dart
===========================

``landing_page_logged_in.dart`` serves as the primary dashboard for authenticated users 
within UniSphere. It dynamically presents tailored event discovery tools, personalized 
ticketing information, and organizer metrics depending on the active user's role.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'dart:typed_data';
   import 'package:unisphere_app/widgets/app_footer.dart';
   import 'package:unisphere_app/widgets/header.dart';
   import 'package:unisphere_app/widgets/pagination_controls.dart';
   import 'event_details_screen.dart';
   import 'package:unisphere_app/services/sqlite_backend.dart';
   import 'package:unisphere_app/utils/event_categories.dart';
   import 'package:unisphere_app/utils/event_date_filters.dart';
   import 'package:unisphere_app/utils/pagination.dart';
   import 'create_event_screen.dart';
   import 'my_tickets_screen.dart';
   import 'my_events_page.dart';

`material.dart` provides the Flutter widgets and layout primitives used across
the file. The typed data import supports banner image handling, while the shared
header, footer, and pagination widgets keep the screen consistent with the rest
of the app. The screen imports are used for navigation from the dashboard and
discover cards, while the SQLite backend and event utility imports provide the
live data, category chips, date filters, and page navigation helpers used by the
logged-in landing page.

PersonalizedLandingPage
-------------------------

.. code-block:: dart

   class PersonalizedLandingPage extends StatefulWidget {
     final String userName;
     final String role;

     const PersonalizedLandingPage({
       super.key,
       this.userName = 'Alex',
       this.role = 'Attendee',
     });
   }

   static const Color background = Color(0xFFF5F7FB);
   static const Color surface = Colors.white;
   static const Color primary = Color(0xFF4F46E5);
   static const Color primaryDark = Color(0xFF3730A3);
   static const Color text = Color(0xFF111827);
   static const Color muted = Color(0xFF6B7280);
   static const Color border = Color(0xFFE5E7EB);
   static const Color softBlue = Color(0xFFEEF2FF);

`PersonalizedLandingPage` is the main entry widget for the logged-in home
screen. It stores the current user's display name and role so the page can
adjust its messaging, dashboard actions, and event summaries. The widget also
holds the page-level visual tokens used by the helper widgets. The colors constants 
define the color palette used throughout the screen.

For testing purposes, the PersonaliszedLandingPage is initialised with the default
values. 'Alex' for `userName` and 'Attendee' for `role`. 


_PersonalizedLandingPageState
-----------------------------

.. code-block:: dart

   class _PersonalizedLandingPageState extends State<PersonalizedLandingPage> {
   // State variables
   late String _selectedFilter;
   late String _searchQuery;
   late TextEditingController _searchController;
   late bool _showFiltersDropdown;
   late Map<String, bool> _dateFilters;

   @override
   void initState() {
      super.initState();
      _selectedFilter = 'All';
      _searchQuery = '';
      _searchController = TextEditingController();
      _showFiltersDropdown = false;
      _dateFilters = {
         'today': false,
         'tomorrow': false,
         'this week': false,
         'next week': false,
         'this month': false,
         'next month': false,
      };
      SqliteBackend().addListener(_onBackendChanged);
   }

The _PersonalizedLandingPageState class manages the interactive parts of the landing page: filter
selection, search text, the date filter dropdown, the controller attached to the
search input, and the derived event lists shown in the two main columns.

- ``_selectedFilter`` stores the active category chip.
- ``_searchQuery`` stores the current search string.
- ``_searchController`` keeps the text field and state synchronized.
- ``_showFiltersDropdown`` controls whether the compact date filter panel is open.
- ``_dateFilters`` tracks toggled date buckets such as today, tomorrow, and this month.

`initState` prepares the screen before the first build. It sets the default
filter values, creates the search controller, initializes the dropdown state,
and subscribes to `SqliteBackend` so the page can refresh when backend data
changes.


User interaction helpers
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   void _setFilter(String filter) {
     setState(() {
       _selectedFilter = filter;
     });
   }

   void _setSearchQuery(String query) {
     setState(() {
       _searchQuery = query;
     });
   }

   void _toggleFiltersDropdown() {
     setState(() {
       _showFiltersDropdown = !_showFiltersDropdown;
     });
   }

   void _setDateFilter(String filter, bool value) {
     setState(() {
       _dateFilters[filter] = value;
     });
   }

These private methods encapsulate the reactive logic of the landing page, 
handling everything from user input to database synchronization:  

- ``_onBackendChanged``: Triggers a UI rebuild whenever the local SQLite database updates, ensuring the displayed events stay fresh.  
- ``dispose``: Prevents memory leaks by explicitly removing the database listener and destroying the _searchController before the widget is permanently removed from the tree.  
- ``_setFilter``: Updates the active category filter state and forces a UI refresh.  
- ``_setSearchQuery``: Captures user text input to filter the event list dynamically as they type.  
- ``_toggleFiltersDropdown``: Acts as a simple boolean toggle to show or hide the advanced date filtering panel.  
- ``_setDateFilter``: Updates specific active/inactive flags within the multi-select date filter map.  
- ``_tryParseDate``: A safe utility function that attempts to convert dynamic or raw string inputs into DateTime objects, catching formatting errors to prevent app crashes.

Event Data Getters and Filters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   DateTime? _tryParseDate(dynamic value) {
    if (value == null) return null;
    final raw = value.toString().trim();
    if (raw.isEmpty) return null;
    try {
      return DateTime.parse(raw);
    } catch (_) {
      return null;
    }
   }

   List<Map<String, dynamic>> _sortUpcomingEvents(
    List<Map<String, dynamic>> events,
   ) {
    final now = DateTime.now();
    final limit = now.add(const Duration(days: 30));

    final upcoming = <Map<String, dynamic>>[];
    for (final event in events) {
      final eventDate = _tryParseDate(event['date']);
      if (eventDate == null) continue;
      if (eventDate.isBefore(now) || eventDate.isAfter(limit)) continue;
      final copy = Map<String, dynamic>.from(event);
      copy['eventDate'] = eventDate;
      upcoming.add(copy);
    }

    upcoming.sort((a, b) {
      final aDate = a['eventDate'] as DateTime?;
      final bDate = b['eventDate'] as DateTime?;
      if (aDate == null && bDate == null) return 0;
      if (aDate == null) return 1;
      if (bDate == null) return -1;
      return aDate.compareTo(bDate);
    });

    return upcoming;
   }

   List<Map<String, dynamic>> get _organiserLiveEvents {
    final email = SqliteBackend().currentUser?.email;
    if (email == null) return [];

    return SqliteBackend().events
        .where(
          (event) =>
              event['organizerEmail']?.toString() == email &&
              event['title']?.toString().toLowerCase() != 'demo event',
        )
        .map(_toDashboardEvent)
        .toList();
   }

   List<Map<String, dynamic>> get _upcomingEventsWithinNextMonth {
    final isOrganiser =
        (SqliteBackend().currentUser?.isOrganiser ?? false) ||
        widget.role.toLowerCase() == 'organiser';

    if (isOrganiser) {
      return _applyDateFiltersToEvents(
        _sortUpcomingEvents(_organiserLiveEvents),
      );
    }

    final upcoming = <Map<String, dynamic>>[];
    for (final ticket in SqliteBackend().purchasedTickets) {
      if (ticket.title.toLowerCase() == 'demo event') continue;

      final matchingEvent = SqliteBackend().events.firstWhere((event) {
        final eventId = int.tryParse(event['id']?.toString() ?? '');
        if (ticket.eventId != null && eventId == ticket.eventId) {
          return true;
        }
        return event['title']?.toString() == ticket.title &&
            event['date']?.toString() == ticket.date &&
            event['location']?.toString() == ticket.location;
      }, orElse: () => {});

      final eventDate = _tryParseDate(matchingEvent['date'] ?? ticket.date);
      if (eventDate == null) continue;

      upcoming.add({
        'title': ticket.title,
        'date': matchingEvent.isNotEmpty
            ? matchingEvent['date']?.toString() ?? ticket.date
            : ticket.date,
        'eventDate': eventDate,
        'location': ticket.location,
        'category': ticket.category,
        'icon': Icons.confirmation_number_outlined,
        'color': const Color(0xFF4F46E5),
        'bannerImageData': matchingEvent.isNotEmpty
            ? matchingEvent['bannerImageData']
            : null,
        'organizer': matchingEvent.isNotEmpty
            ? (matchingEvent['organizer']?.toString() ?? 'UniSphere')
            : 'UniSphere',
      });
    }

    return _applyDateFiltersToEvents(_sortUpcomingEvents(upcoming));
   }

   List<Map<String, dynamic>> _applyDateFiltersToEvents(
    List<Map<String, dynamic>> events,
   ) {
    if (!hasActiveDateFilters(_dateFilters)) return events;
    return events
        .where((event) => matchesDateFilters(event['date'], _dateFilters))
        .toList();
   }

   Map<String, dynamic> _toDashboardEvent(Map<String, dynamic> event) {
    final category = event['category']?.toString() ?? 'Other';
    final color = _dashboardColorForCategory(category);
    final dateText = event['date']?.toString() ?? '';

    return {
      'title': event['title']?.toString() ?? 'Untitled Event',
      'date': dateText,
      'location': event['location']?.toString() ?? 'TBA',
      'category': category,
      'color': color,
      'icon': _dashboardIconForCategory(category),
      'bannerImageData': event['bannerImageData'],
      'organizer': event['organizer']?.toString() ?? 'UniSphere',
    };
   }

   Color _dashboardColorForCategory(String category) {
    switch (category.toLowerCase()) {
      case 'social':
        return const Color(0xFF0F766E);
      case 'tech':
      case 'technology':
        return const Color(0xFF4F46E5);
      case 'career':
        return const Color(0xFFEA580C);
      case 'sports':
        return const Color(0xFF16A34A);
      case 'music':
        return const Color(0xFFDB2777);
      default:
        return const Color(0xFF4F46E5);
    }
   }

  IconData _dashboardIconForCategory(String category) {
    switch (category.toLowerCase()) {
      case 'social':
        return Icons.groups_rounded;
      case 'tech':
      case 'technology':
        return Icons.code_rounded;
      case 'career':
        return Icons.work_outline_rounded;
      case 'sports':
        return Icons.sports_soccer_rounded;
      case 'music':
        return Icons.music_note_rounded;
      default:
        return Icons.event_rounded;
    }
   }

  List<Map<String, dynamic>> get _filteredEvents {
    final backendEvents = SqliteBackend().events
        .where(
          (e) =>
              e['title']?.toString().toLowerCase() != 'demo event' &&
              e['visibility']?.toString() != 'Private',
        )
        .map(
          (event) => {
            'id': event['id'],
            'title': event['title'] ?? 'Untitled Event',
            'date': event['date'] ?? '',
            'location': event['location'] ?? 'TBA',
            'category': event['category'] ?? 'Other',
            'description': event['description'] ?? '',
            'imageColor': const Color(0xFFE0E7FF),
            'icon': Icons.event_rounded,
            'bannerImageData': event['bannerImageData'],
            'organizer': event['organizer'] ?? 'UniSphere',
            'organizerEmail': event['organizerEmail'],
          },
        )
        .toList();

    final baseEvents = _selectedFilter == 'All'
        ? backendEvents
        : backendEvents
              .where((event) => event['category'] == _selectedFilter)
              .toList();

    if (_searchQuery.isEmpty) {
      return _applyDateFiltersToEvents(baseEvents);
    }

    final query = _searchQuery.toLowerCase();
    final searched = baseEvents.where((event) {
      final category = (event['category'] as String).toLowerCase();
      final title = (event['title'] as String).toLowerCase();
      final titleWords = title.split(' ');
      final location = (event['location'] as String).toLowerCase();

      return category.startsWith(query) ||
          titleWords.any((word) => word.contains(query)) ||
          location.contains(query);
    }).toList();

    return _applyDateFiltersToEvents(searched);
  }


This section acts as the data transformation layer for the landing page, 
handling the retrieval, sorting, and formatting of event data before it reaches 
the UI:  

- ``_sortUpcomingEvents``: Filters a raw list of events to only include those happening within the next 30 days, then sorts them chronologically.
- ``_organiserLiveEvents``: A getter that retrieves all events owned by the currently logged-in organizer, explicitly filtering out placeholder "demo" events.
- ``_upcomingEventsWithinNextMonth``: Adapts to the user's role; it either fetches an organizer's upcoming hosted events or matches an attendee's purchased tickets with database records to build their personal itinerary. 
- ``_applyDateFiltersToEvents``: Cross-references a provided list of events against the active _dateFilters map, ensuring only events falling within selected timeframes (e.g., "today", "this week") are returned.  
- ``_toDashboardEvent``: A data normalization utility that converts raw database maps into guaranteed, non-null objects formatted specifically for the dashboard cards.  
- ``_dashboardColorForCategory`` & ``_dashboardIconForCategory``: Helper functions containing switch statements that assign consistent visual branding (colors and icons) based on an event's category.  
- ``_filteredEvents``: The master getter for the discovery feed. It fetches public events, applies the active category chip, performs text-based searches across titles and locations, and passes the result through the date filters.

Widget build
^^^^^^^^^^^^

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
    final isOrganiser =
        SqliteBackend().currentUser?.isOrganiser ??
        widget.role.toLowerCase() == 'organiser';
    final upcomingEvents = _upcomingEventsWithinNextMonth;

    return Scaffold(
      backgroundColor: PersonalizedLandingPage.background,
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
                      AppHeader(
                        onHostEventTap: () {
                          Navigator.pushNamed(context, '/create-event');
                        },
                        onFindEventsTap: () {
                          Navigator.pushNamed(context, '/discover');
                        },
                        onCreateEventsTap: () {
                          Navigator.pushNamed(context, '/create-event');
                        },
                        onMyTicketsTap: () {
                          Navigator.pushNamed(context, '/my-tickets');
                        },
                        onAboutTap: () {
                          Navigator.pushNamed(context, '/about');
                        },
                        onSignInTap: () {
                          Navigator.pushNamed(context, '/login');
                        },
                      ),
                      LayoutBuilder(
                        builder: (context, constraints) {
                          final isMobile = constraints.maxWidth < 1100;

                          if (isMobile) {
                            return Padding(
                              padding: const EdgeInsets.all(20),
                              child: Column(
                                children: [
                                  _DiscoverSection(
                                    userName: widget.userName,
                                    isOrganiser: isOrganiser,
                                    selectedFilter: _selectedFilter,
                                    filteredEvents: _filteredEvents,
                                    onFilterChanged: _setFilter,
                                    searchController: _searchController,
                                    onSearchChanged: _setSearchQuery,
                                    showFiltersDropdown: _showFiltersDropdown,
                                    dateFilters: _dateFilters,
                                    onToggleFiltersDropdown:
                                        _toggleFiltersDropdown,
                                    onDateFilterChanged: _setDateFilter,
                                  ),
                                  const SizedBox(height: 20),
                                  _DashboardPanel(
                                    userName: widget.userName,
                                    isOrganiser: isOrganiser,
                                    upcomingEvents: upcomingEvents,
                                  ),
                                ],
                              ),
                            );
                          } else {
                            return Center(
                              child: ConstrainedBox(
                                constraints: const BoxConstraints(
                                  maxWidth: 1400,
                                ),
                                child: Row(
                                  crossAxisAlignment: CrossAxisAlignment.start,
                                  children: [
                                    Expanded(
                                      flex: 7,
                                      child: Padding(
                                        padding: const EdgeInsets.fromLTRB(
                                          28,
                                          28,
                                          20,
                                          28,
                                        ),
                                        child: _DiscoverSection(
                                          userName: widget.userName,
                                          isOrganiser: isOrganiser,
                                          selectedFilter: _selectedFilter,
                                          filteredEvents: _filteredEvents,
                                          onFilterChanged: _setFilter,
                                          searchController: _searchController,
                                          onSearchChanged: _setSearchQuery,
                                          showFiltersDropdown:
                                              _showFiltersDropdown,
                                          dateFilters: _dateFilters,
                                          onToggleFiltersDropdown:
                                              _toggleFiltersDropdown,
                                          onDateFilterChanged: _setDateFilter,
                                        ),
                                      ),
                                    ),
                                    Container(
                                      width: 430,
                                      decoration: const BoxDecoration(
                                        color: Color(0xFFF8FAFC),
                                        border: Border(
                                          left: BorderSide(
                                            color:
                                                PersonalizedLandingPage.border,
                                          ),
                                        ),
                                      ),
                                      child: Padding(
                                        padding: const EdgeInsets.all(24),
                                        child: _DashboardPanel(
                                          userName: widget.userName,
                                          isOrganiser: isOrganiser,
                                          upcomingEvents: upcomingEvents,
                                        ),
                                      ),
                                    ),
                                  ],
                                ),
                              ),
                            );
                          }
                        },
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
  }

The build method constructs the primary visual scaffolding for the authenticated user's
dashboard. It intelligently handles state resolution, responsiveness, and component 
composition:  

- User Role Resolution: Resolves whether the active user is an organizer or an attendee before rendering, ensuring the correct data is passed to child widgets.  
- Responsive Layout Strategy: Utilizes a nested LayoutBuilder to detect screen width. For screens under 1100px, it stacks the _DiscoverSection and _DashboardPanel vertically. For wider desktop screens, it displays them side-by-side.  
- Structural Constraints: Wraps the entire scrollable content inside a ConstrainedBox with a minimum height constraint, which guarantees that the AppFooter always anchors neatly to the bottom of the viewport even if the main content is brief.  
- Component Composition: Assembles the final page by combining the shared global AppHeader, the dynamic middle content areas, and the global AppFooter.

Discover section widget
-----------------------

.. code-block:: dart

   class _DiscoverSection extends StatelessWidget {
     final String userName;
     final bool isOrganiser;
     final String selectedFilter;
     final List<Map<String, dynamic>> filteredEvents;
     final Function(String) onFilterChanged;
     final TextEditingController searchController;
     final ValueChanged<String> onSearchChanged;
     final bool showFiltersDropdown;
     final Map<String, bool> dateFilters;
     final VoidCallback onToggleFiltersDropdown;
     final Function(String, bool) onDateFilterChanged;

     const _DiscoverSection({...});
   
   @override
     Widget build(BuildContext context) {
       return Column(
         crossAxisAlignment: CrossAxisAlignment.start,
         children: [
           _TopWelcomeStrip(userName: userName, isOrganiser: isOrganiser),
           const SizedBox(height: 24),
           _SearchAndFilters(...),
           const SizedBox(height: 28),
           // ... Header text ...
           _CategoryChips(...),
           const SizedBox(height: 24),
           _PaginatedEventGrid(...),
         ],
       );
     }
   }

The _DiscoverSection acts as the primary layout coordinator for the main content area. 
It is a StatelessWidget that receives all necessary state variables and callback 
functions from the parent page and passes them down to its specialized child components: 
the welcome strip, the search bar, the category chips, and the final event grid.

_TopWelcomeStrip
^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _TopWelcomeStrip extends StatelessWidget {
     final String userName;
     final bool isOrganiser;

     const _TopWelcomeStrip({required this.userName, required this.isOrganiser});

     @override
     Widget build(BuildContext context) {
       return Container(
         // ... styling and gradient ...
         child: Row(
            // ... icon ...
             child: Column(
               crossAxisAlignment: CrossAxisAlignment.start,
               children: [
                  Text('Welcome back, $userName', ...),
                  Text(
                  isOrganiser
                        ? 'Manage events, review performance, and keep your listings active.'
                        : 'Jump back into discovery and see what’s happening next.',
                  // ...
                  ),
               ],
            ),
            ),
            ElevatedButton(
            onPressed: () {
               Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => const CreateEventScreen()),
               );
            },
            child: const Text('Create Events'),
            ),
         ],
         ),
      );
   }

This component provides a personalized greeting at the top of the feed. It dynamically
adjusts its subtitle text based on whether the isOrganiser flag is true, and includes
a prominent primary button that routes users directly to the ``CreateEventScreen``.

_SearchAndFilters
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _SearchAndFilters extends StatelessWidget {
     final TextEditingController searchController;
     final ValueChanged<String> onSearchChanged;
     final bool showFiltersDropdown;
     final Map<String, bool> dateFilters;
     final VoidCallback onToggleFiltersDropdown;
     final Function(String, bool) onDateFilterChanged;

     const _SearchAndFilters({ ... });

      @override
      Widget build(BuildContext context) {
         return Column(
           children: [
             Row(
               children: [
                 Expanded(
                   child: TextField(
                     controller: searchController,
                     onChanged: onSearchChanged,
                     decoration: const InputDecoration(
                       hintText: 'Search events, societies, categories, places...',
                     ),
                   ),
                 ),
                 GestureDetector(
                   onTap: onToggleFiltersDropdown,
                   child: Icon(Icons.tune_rounded), // Filter toggle button
                 ),
               ],
             ),
             if (showFiltersDropdown) ...[
               // ... Expandable Date Filter Panel ...
               Wrap(
                 children: kEventDateFilters.map((filter) {
                   final isSelected = dateFilters[filter] ?? false;
                   return GestureDetector(
                     onTap: () => onDateFilterChanged(filter, !isSelected),
                     // ... Custom checkbox UI ...
                   );
                 }).toList(),
               ),
             ],
           ],
         );
      }
   }

The `_SearchAndFilters` widget combines the search bar, the filter toggle, and
the optional date filter panel. It stays stateless and delegates all changes
back to the parent through callbacks. The dropdown is important because it
controls the date filter map used by the query pipeline.

_CategoryChips
^^^^^^^^^^^^^^

.. code-block:: dart

   class _CategoryChips extends StatelessWidget {
     final String selectedFilter;
     final Function(String) onFilterChanged;
   
     const _CategoryChips({
       required this.selectedFilter,
       required this.onFilterChanged,
     });
   
     @override
     Widget build(BuildContext context) {
       final chips = kEventFilterCategories;
   
       return Wrap(
         spacing: 10,
         runSpacing: 10,
         children: chips.map((chip) {
           final isSelected = chip == selectedFilter;
           return GestureDetector(
             onTap: () => onFilterChanged(chip),
             child: Container(
               // ... dynamic styling based on isSelected ...
               child: Text(chip),
               ),
             ),
           );
         }).toList(),
       );
     }
   }

This widget renders a responsive row of selectable categories (e.g., "Tech", "Social", "Sports")
mapped from the kEventFilterCategories constant. It applies dynamic styling to visually 
highlight the active filter based on the selectedFilter string passed from the parent
state.

_DiscoverEventCard
------------------

.. code-block:: dart

   class _DiscoverEventCard extends StatelessWidget {
     final Map<String, dynamic> event;

     const _DiscoverEventCard({required this.event});

     @override
     Widget build(BuildContext context) {
       final color = event['imageColor'] as Color;
       final icon = event['icon'] as IconData;
       final bannerData = event['bannerImageData'];
       final Uint8List? bannerBytes = bannerData is Uint8List ? bannerData : null;

       return Container(
         decoration: BoxDecoration(
           color: Colors.white,
           borderRadius: BorderRadius.circular(22),
           // ... border and shadow styling ...
         ),
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Expanded(
               child: Container(
                 width: double.infinity,
                 decoration: BoxDecoration(
                   color: color,
                   borderRadius: const BorderRadius.vertical(top: Radius.circular(22)),
                 ),
                 child: _buildEventCardImage(bannerBytes, icon),
               ),
             ),
             Padding(
               padding: const EdgeInsets.all(18),
               child: Column(
                 crossAxisAlignment: CrossAxisAlignment.start,
                 children: [
                   Text(event['title'] as String, ...),
                   Text(event['date'] as String, ...),
                   Text(event['location'] as String, ...),
                   Text('By ${event['organizer']?.toString() ?? 'UniSphere'}', ...),
                   const SizedBox(height: 14),
                   Row(
                     children: [
                       Container(
                         // ... Category Badge styling ...
                         child: Text(event['category'] as String, ...),
                       ),
                       const Spacer(),
                       TextButton(
                         onPressed: () {
                           Navigator.push(
                             context,
                             MaterialPageRoute(
                               builder: (_) => EventDetailsScreen(event: event),
                             ),
                           );
                         },
                         child: const Text('View details'),
                       ),
                     ],
                   ),
                 ],
               ),
             ),
           ],
         ),
       );
     }

     Widget _buildEventCardImage(Uint8List? bannerBytes, IconData icon) {
       if (bannerBytes != null) {
         return ClipRRect(
           borderRadius: const BorderRadius.vertical(top: Radius.circular(22)),
           child: Image.memory(bannerBytes, fit: BoxFit.cover, ...),
         );
       } else {
         return Center(
           child: Icon(icon, size: 42, color: PersonalizedLandingPage.primary),
         );
       }
     }
   }

The ``_DiscoverEventCard`` is the primary visual component used to display individual events within the discovery grid.
It receives a single event mapping and constructs a visually distinct card. 

* **Dynamic Media:** The helper method ``_buildEventCardImage`` checks if the event contains custom ``bannerImageData`` (a ``Uint8List``). If it does, it renders the image; if not, it safely falls back to a categorized icon and background color.
* **Information Hierarchy:** Below the media block, it displays the event title, date, location, and organizer in a strict typographic hierarchy. 
* **Interaction:** The bottom row features a color-coded category badge and a text button that routes the user directly to the ``EventDetailsScreen``, passing the selected event's data payload along with it.

_DashboardPanel
---------------

.. code-block:: dart

   class _DashboardPanel extends StatelessWidget {
     final String userName;
     final bool isOrganiser;
     final List<Map<String, dynamic>> upcomingEvents;

     const _DashboardPanel({
       required this.userName,
       required this.isOrganiser,
       required this.upcomingEvents,
     });

     @override
     Widget build(BuildContext context) {
       return Column(
         crossAxisAlignment: CrossAxisAlignment.start,
         children: [
           const Text('Dashboard', ...), // Header text
           Container(
             // ... User Profile Card ...
             child: Row(
               children: [
                 CircleAvatar(...),
                 Expanded(
                   child: Column(
                     crossAxisAlignment: CrossAxisAlignment.start,
                     children: [
                       Text(userName, ...),
                       Text(isOrganiser ? 'Organiser account' : 'Attendee account', ...),
                     ],
                   ),
                 ),
                 IconButton(
                   onPressed: () {
                     if (SqliteBackend().currentUser != null) {
                       Navigator.pushNamed(context, '/profile');
                     } else {
                       Navigator.pushNamed(context, '/register');
                     }
                   },
                   icon: const Icon(Icons.settings_outlined),
                 ),
               ],
             ),
           ),
           const SizedBox(height: 18),
           Row(
             children: [
               Expanded(
                 child: GestureDetector(
                   onTap: isOrganiser ? () => Navigator.push(...) : null,
                   child: Builder(
                     builder: (context) {
                       final liveCount = isOrganiser
                           ? SqliteBackend().events.where(...).length
                           : SqliteBackend().purchasedTickets.length;
                       return _DashboardMetricCard(
                         title: isOrganiser ? 'Live Events' : 'Events Joined',
                         value: '$liveCount',
                         icon: isOrganiser ? Icons.event_note_rounded : Icons.event_available_outlined,
                         color: const Color(0xFF4F46E5),
                       );
                     },
                   ),
                 ),
               ),
               // ... Upcoming Metric Card ...
             ],
           ),
           const SizedBox(height: 12),
           ListenableBuilder(
             listenable: SqliteBackend(),
             builder: (context, _) {
               // ... Calculates Total Views or My Tickets ...
               return _DashboardMetricCard(...);
             },
           ),
           const SizedBox(height: 22),
           // ... Action Buttons (Create / My Tickets) ...
           const SizedBox(height: 28),
           const Text('Upcoming Events', ...),
           // ... Paginated Upcoming Events List or Empty State ...
         ],
       );
     }
   }

The ``_DashboardPanel`` serves as the personalized sidebar (or stacked panel on mobile) providing a high-level overview of the user's account activity. 

* **Profile Overview:** Displays the user's name, their role, and a quick-access settings button that navigates to the profile or registration screen based on authentication status.
* **Dynamic Metric Cards:** Renders responsive stat cards (Live Events, Upcoming, Total Views/Tickets) that adapt their labels and calculations depending on whether the user is viewing the page as an Organizer or an Attendee. 
* **Real-time Updates:** Utilizes a ``ListenableBuilder`` attached to the ``SqliteBackend`` to ensure metrics like ticket counts or event views update instantly without requiring a full page reload.
* **Action Routing:** Provides contextual quick-action buttons (e.g., "Create" for organizers or "My Tickets" for attendees) and renders the user's immediate upcoming itinerary at the bottom of the panel.


_DashboardMetricCard
--------------------

.. code-block:: dart

   class _DashboardMetricCard extends StatelessWidget {
     final String title;
     final String value;
     final IconData icon;
     final Color color;
     final bool fullWidth;

     const _DashboardMetricCard({
       required this.title,
       required this.value,
       required this.icon,
       required this.color,
       this.fullWidth = false,
     });

     @override
     Widget build(BuildContext context) {
       return Container(
         width: fullWidth ? double.infinity : null,
         // ... styling ...
         child: Row(
           children: [
             Container(
               // ... Icon container ...
               child: Icon(icon, color: color, size: 20),
             ),
             const SizedBox(width: 12),
             Column(
               crossAxisAlignment: CrossAxisAlignment.start,
               children: [
                 Text(value, ...), // The metric number
                 Text(title, ...), // The metric label
               ],
             ),
           ],
         ),
       );
     }
   }

   String _formatCompactCount(int value) {
     if (value <= 0) return '0';
     if (value < 1000) return '$value';

     final thousands = value ~/ 1000;
     final remainder = value % 1000;
     if (remainder == 0) {
       return '${thousands}K';
     }

     return '${thousands}.${(remainder / 100).floor()}K';
   }

The ``_DashboardMetricCard`` is a reusable component designed to display key performance
 indicators (KPIs) or personal stats. It accepts a title, value, icon, and an accent color, 
 wrapping them in a standardized bordered container. 

Accompanying this widget is the ``_formatCompactCount`` utility function, which safely
converts large integers into easily scannable shorthand strings (e.g., converting 
``1500`` into ``"1.5K"``) to ensure the metric UI remains unbroken by excessively 
long numbers.
