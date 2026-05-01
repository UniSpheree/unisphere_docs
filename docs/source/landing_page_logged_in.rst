landing_page_logged_in.dart
===========================

``landing_page_logged_in.dart`` is the personalized landing page shown after
sign-in. It combines event discovery, account context, and a dashboard-style
summary into one responsive screen. The file is structured around one stateful
page widget and a set of small helper widgets that build the discover feed and
dashboard areas.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'package:unisphere_app/widgets/app_footer.dart';
   import 'package:unisphere_app/widgets/header.dart';
   import 'event_details_screen.dart';
   import 'package:unisphere_app/utils/mock_backend.dart';
   import 'create_event_screen.dart';
   import 'my_tickets_screen.dart';
   import 'my_events_page.dart';

`material.dart` provides the Flutter widgets and layout primitives used across
the file. The shared header and footer widgets keep the landing page consistent
with the rest of the app. The screen imports are used for navigation from the
dashboard and discover cards, while `MockBackend` provides sample data and user
state.

Personalized Landing Page
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

The `PersonalizedLandingPage` class is the main entry widget for the logged-in
home screen. It stores the current user's display name and role so the page can
adjust its messaging, dashboard actions, and event summaries.

Visual tokens
^^^^^^^^^^^^^

.. code-block:: dart

   static const Color background = Color(0xFFF5F7FB);
   static const Color surface = Colors.white;
   static const Color primary = Color(0xFF4F46E5);
   static const Color primaryDark = Color(0xFF3730A3);
   static const Color text = Color(0xFF111827);
   static const Color muted = Color(0xFF6B7280);
   static const Color border = Color(0xFFE5E7EB);
   static const Color softBlue = Color(0xFFEEF2FF);

These constants define the color palette used throughout the screen. Keeping
them on the widget class makes the page styling easy to reuse inside helper
widgets without threading values through every constructor.

Sample event data
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   static const List<Map<String, dynamic>> discoverEvents = [ ... ];
   static final List<Map<String, dynamic>> upcomingEvents = [ ... ];

The page includes sample discover cards and upcoming dashboard events. The
discover list demonstrates the card layout used in the browse column, while the
upcoming events list powers the dashboard preview shown to logged-in users.

State class
-----------

.. code-block:: dart

   class _PersonalizedLandingPageState extends State<PersonalizedLandingPage> {
     late String _selectedFilter;
     late String _searchQuery;
     late TextEditingController _searchController;
     late bool _showFiltersDropdown;
     late Map<String, bool> _dateFilters;
   }

The state class manages the interactive parts of the landing page: filter
selection, search text, the date filter dropdown, and the controller attached
to the search input.

State fields
^^^^^^^^^^^^

- `_selectedFilter` stores the active category chip.
- `_searchQuery` stores the current search string.
- `_searchController` keeps the text field and state synchronized.
- `_showFiltersDropdown` controls whether the compact date filter panel is open.
- `_dateFilters` tracks toggled date buckets such as today, tomorrow, and this month.

Lifecycle methods
^^^^^^^^^^^^^^^^^

.. code-block:: dart

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
     MockBackend().addListener(_onBackendChanged);
   }

`initState` prepares the screen before the first build. It sets the default
filter values, creates the search controller, initializes the dropdown state,
and subscribes to the mock backend so the page can refresh when backend data
changes.

.. code-block:: dart

   @override
   void dispose() {
     MockBackend().removeListener(_onBackendChanged);
     _searchController.dispose();
     super.dispose();
   }

`dispose` removes the backend listener and cleans up the text controller. This
keeps the widget lifecycle tidy and avoids retaining resources after the page
is removed from the tree.

Backend listener
^^^^^^^^^^^^^^^^

.. code-block:: dart

   void _onBackendChanged() {
     if (!mounted) return;
     setState(() {});
   }

This listener causes the page to rebuild when `MockBackend` notifies changes.
It first checks `mounted` so it only updates while the widget is still active.

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

These methods isolate state mutations behind named callbacks. The child widgets
use them for category changes, search updates, and date-filter toggles, which
keeps the lower-level widgets stateless and easier to reason about.

Date helper
^^^^^^^^^^^

.. code-block:: dart

   static bool _isWithinNext30Days(DateTime eventDate) {
     final now = DateTime.now();
     final today = DateTime(now.year, now.month, now.day);
     final limit = today.add(const Duration(days: 30));
     return !eventDate.isBefore(today) && eventDate.isBefore(limit);
   }

This helper checks whether an event falls within the next 30 days. The landing
page uses that logic to decide which events belong in the upcoming dashboard
area.

Event transformation helpers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   List<Map<String, dynamic>> get _organiserLiveEvents { ... }
   List<Map<String, dynamic>> get _upcomingEventsWithinNextMonth { ... }
   Map<String, dynamic> _toDashboardEvent(Map<String, dynamic> event) { ... }

`_organiserLiveEvents` filters backend events to those owned by the current
user and maps them into dashboard-friendly data. `_upcomingEventsWithinNextMonth`
returns organiser events for organisers or purchased tickets for attendees.
`_toDashboardEvent` normalizes a raw event record into the shape expected by
the dashboard cards.

Category styling helpers
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Color _dashboardColorForCategory(String category) { ... }
   IconData _dashboardIconForCategory(String category) { ... }

These helpers map category names to a consistent color and icon. They make the
dashboard cards feel coherent by keeping category styling logic in one place.

Event filtering helper
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   List<Map<String, dynamic>> get _filteredEvents { ... }

This getter pulls events from `MockBackend`, normalizes them into a display
shape, applies the active category filter, and then narrows the result with a
case-insensitive search query.

Widget build
^^^^^^^^^^^^

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     final isOrganiser =
         MockBackend().currentUser?.isOrganiser ??
         widget.role.toLowerCase() == 'organiser';
     final upcomingEvents = _upcomingEventsWithinNextMonth;

     return Scaffold(
       backgroundColor: PersonalizedLandingPage.background,
       body: Column(
         children: [
           AppHeader(...),
           Expanded(
             child: LayoutBuilder(...),
           ),
         ],
       ),
     );
   }

The `build` method assembles the full page. It determines whether the user is
an organiser, prepares the upcoming event list, renders the shared app header,
and then uses `LayoutBuilder` to switch between a wide two-column layout and a
stacked mobile layout.

The header wires its callbacks to navigation targets such as `CreateEventScreen`.
Inside the scrollable body, the page places the discover section, the dashboard,
and the shared footer.

Discover section widget
-----------------------

.. code-block:: dart

   class _DiscoverSection extends StatelessWidget {
     const _DiscoverSection({
       required this.userName,
       required this.isOrganiser,
       required this.selectedFilter,
       required this.filteredEvents,
       required this.onFilterChanged,
       required this.searchController,
       required this.onSearchChanged,
       required this.showFiltersDropdown,
       required this.dateFilters,
       required this.onToggleFiltersDropdown,
       required this.onDateFilterChanged,
     });
   }

The `_DiscoverSection` widget builds the discovery column. It combines the
welcome strip, search and filter controls, category chips, and the event grid
into one cohesive section.

Its constructor receives the state it needs from the parent, which keeps the
widget itself free from business logic.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

Its build method lays out the discovery content vertically and uses a nested
`LayoutBuilder` to choose the event grid column count based on available width.

Top welcome strip
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _TopWelcomeStrip extends StatelessWidget {
     const _TopWelcomeStrip({required this.userName, required this.isOrganiser});
   }

This widget shows the greeting banner at the top of the discovery column. It
welcomes the current user, changes the supporting message based on organiser
status, and includes a primary action button for creating events.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

The build method renders a colored surface with a hand icon, greeting text, a
status-specific subtitle, and the `Create Events` button.

Search and filters widget
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _SearchAndFilters extends StatelessWidget {
     const _SearchAndFilters({
       required this.searchController,
       required this.onSearchChanged,
       required this.showFiltersDropdown,
       required this.dateFilters,
       required this.onToggleFiltersDropdown,
       required this.onDateFilterChanged,
     });
   }

The `_SearchAndFilters` widget combines the search bar, the filter toggle, and
the optional date filter panel. It stays stateless and delegates all changes
back to the parent through callbacks.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

Its build method wraps the search field and filter button in a shared container
and conditionally expands the date filter grid when the dropdown is open.

Category chips widget
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _CategoryChips extends StatelessWidget {
     const _CategoryChips({
       required this.selectedFilter,
       required this.onFilterChanged,
     });
   }

The `_CategoryChips` widget renders the horizontal set of category chips used
to narrow the discover feed. It highlights the active chip and notifies the
parent when the user selects a different category.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

The build method maps over the available categories, styles the selected chip,
and attaches tap handlers for filter changes.

Discover event card
^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _DiscoverEventCard extends StatelessWidget {
     const _DiscoverEventCard({required this.event});
   }

The `_DiscoverEventCard` widget displays a single event inside the discover
grid. It shows the event image area, title, date, location, category label, and
the `View details` action.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

Its build method uses the event map supplied by the parent and routes to
`EventDetailsScreen` when the user taps the details button.

Dashboard panel widget
----------------------

.. code-block:: dart

   class _DashboardPanel extends StatelessWidget {
     const _DashboardPanel({
       required this.userName,
       required this.isOrganiser,
       required this.upcomingEvents,
     });
   }

The `_DashboardPanel` widget builds the right-hand summary column. It provides
activity metrics, a primary action, and the list of upcoming events shown in
the logged-in dashboard view.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

The build method adds the dashboard title, a contextual description, user
summary card, metric tiles, a role-specific CTA button, and the upcoming events
list.

Dashboard metric card
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _DashboardMetricCard extends StatelessWidget {
     const _DashboardMetricCard({
       required this.title,
       required this.value,
       required this.icon,
       required this.color,
       this.fullWidth = false,
     });
   }

The `_DashboardMetricCard` widget renders a compact metric tile with an icon,
value, and label. It is reused for several dashboard statistics so the card
style stays consistent.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

The build method constructs a bordered surface with the icon badge on the left
and the metric text on the right. The optional `fullWidth` flag lets the card
fill the available width when needed.

Compact count formatter
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

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

This helper formats large numbers into compact strings such as `1.2K`. The
dashboard uses it for view counts and other large metrics so the labels remain
easy to scan.

Upcoming event card
^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _UpcomingEventCard extends StatelessWidget {
     const _UpcomingEventCard({required this.event});
   }

The `_UpcomingEventCard` widget renders the compact event row shown in the
dashboard's upcoming section. It displays the event icon, title, date,
location, and category badge in a single horizontal layout.

.. code-block:: dart

   @override
   Widget build(BuildContext context) { ... }

The build method reads the normalized event map passed in by the dashboard and
styles the card using the category color from that event.


