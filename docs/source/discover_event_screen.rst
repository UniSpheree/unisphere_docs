discover_event_screen.dart
==========================

``discover_event_screen.dart`` is the event discovery screen. It provides the
search, category filtering, date filtering, and event card grid used to browse
available events before opening the full details page.

Imports
-------

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:unisphere_app/widgets/app_footer.dart';
    import 'package:unisphere_app/widgets/header.dart';
    import 'package:unisphere_app/utils/mock_backend.dart';
    import 'event_details_screen.dart';
    import 'create_event_screen.dart';

`material.dart` supplies the Flutter widgets and layout primitives used by the
screen. The shared header and footer keep the discovery page visually aligned
with the rest of the site. `MockBackend` provides the event list, while the
screen imports support navigation to event details and event creation.

Main Discover Widget
--------------------

.. code-block:: dart

    class DiscoverEventScreen extends StatefulWidget {
       final String? initialSearchQuery;

       const DiscoverEventScreen({super.key, this.initialSearchQuery});
    }

The `DiscoverEventScreen` class is a `StatefulWidget`. It accepts an optional
`initialSearchQuery` so the page can open with a prefilled search term and then
creates the state object that manages the filters, search input, and filtered
event grid.

Discover Event State
--------------------

.. code-block:: dart

    class _DiscoverEventScreenState extends State<DiscoverEventScreen> {
       bool _showFiltersDropdown = false;
       final Map<String, bool> _dateFilters = { ... };
       final Map<String, bool> _priceFilters = {'free': false, 'paid': false};
       String _selectedFilter = 'All';
       late TextEditingController _searchController;
       String _submittedSearchQuery = '';
    }

The `_DiscoverEventScreenState` class stores the active filter state, the text
controller for the search bar, and the search query that is actually applied to
the results. It also keeps the dropdown open state and two filter maps for date
and price controls.

State fields
^^^^^^^^^^^^

- `_showFiltersDropdown` controls whether the filter panel is visible.
- `_dateFilters` tracks which date buckets are selected.
- `_priceFilters` tracks the free and paid toggles.
- `_selectedFilter` stores the current category chip.
- `_searchController` keeps the search field text in sync with the state.
- `_submittedSearchQuery` stores the query that is applied to the grid.

Lifecycle methods
^^^^^^^^^^^^^^^^^

.. code-block:: dart

    @override
    void initState() {
       super.initState();
       _searchController = TextEditingController();
       MockBackend().addListener(_onBackendChanged);
       if (widget.initialSearchQuery != null &&
             widget.initialSearchQuery!.isNotEmpty) {
          _submittedSearchQuery = widget.initialSearchQuery!;
          _searchController.text = widget.initialSearchQuery!;
       }
    }

`initState` creates the search controller, subscribes to the mock backend so
the grid refreshes when the event list changes, and copies the optional initial
search query into both the controller and the submitted query value.

.. code-block:: dart

    @override
    void dispose() {
       MockBackend().removeListener(_onBackendChanged);
       _searchController.dispose();
       super.dispose();
    }

`dispose` removes the backend listener and disposes the search controller.
This keeps the state lifecycle clean and avoids holding resources after the
screen is removed.

Backend listener
^^^^^^^^^^^^^^^^

.. code-block:: dart

    void _onBackendChanged() {
       if (!mounted) return;
       setState(() {});
    }

This listener forces a refresh when the mock backend changes. The mounted check
prevents updates after the widget has already been removed from the tree.

Filter helpers
^^^^^^^^^^^^^^

.. code-block:: dart

    void _toggleFiltersDropdown() {
       setState(() {
          _showFiltersDropdown = !_showFiltersDropdown;
       });
    }

    void _setDateFilter(String key, bool value) {
       setState(() {
          _dateFilters[key] = value;
       });
    }

    void _setPriceFilter(String key, bool value) {
       setState(() {
          _priceFilters[key] = value;
       });
    }

    void _setFilter(String filter) {
       setState(() {
          _selectedFilter = filter;
       });
    }

These methods centralize the page's interactive state updates. They are called
by the filter toggle, the date chips, the price chips, and the category chips
so the UI can stay stateless at the widget level.

Event data getter
^^^^^^^^^^^^^^^^^

.. code-block:: dart

    List<Map<String, dynamic>> get _discoverEvents {

The `_discoverEvents` getter converts `MockBackend().events` into the smaller
map shape used by the discovery cards. It normalizes missing values and assigns
default icon and color metadata for the grid.

Filtering helper
^^^^^^^^^^^^^^^^

.. code-block:: dart

    List<Map<String, dynamic>> get _filteredEvents {

The `_filteredEvents` getter applies the selected category filter and the
submitted search query to the event list. It first narrows the events by
category, then performs a case-insensitive match against the title and
category text.

Widget build
^^^^^^^^^^^^

.. code-block:: dart

    @override
    Widget build(BuildContext context) {

The `build` method assembles the full discovery page. It returns a scaffold
with a shared header, a scrollable content area, a responsive search and filter
section, a category chip row, and a grid of event cards. The page also keeps
the shared footer visible at the bottom of the layout.

App header
----------

.. code-block:: dart

    AppHeader(
       onHostEventTap: () { ... },
       onRegisterTap: () { ... },
       onFindEventsTap: () { ... },
       onCreateEventsTap: () { ... },
       onMyTicketsTap: () {},
       onAboutTap: () {},
       onSignInTap: () { ... },
       showProfile: true,
    )

The shared `AppHeader` gives the user access to the primary site navigation.
On this page, the find-events action is intentionally inactive because the
user is already on the discovery screen.

Breadcrumb and title row
------------------------

.. code-block:: dart

    Row(
       children: [
          InkWell(...),
          GestureDetector(...),
          Text('  /  '),
          Flexible(child: Text('Discover Events')),
       ],
    )

This row acts as a breadcrumb and back-navigation control. It lets the user
return to the landing page while also showing the current screen name.

Search bar
----------

.. code-block:: dart

    TextField(
       controller: _searchController,
       onChanged: (value) {
          setState(() {});
       },
       onSubmitted: (value) {
          setState(() {
             _submittedSearchQuery = value;
          });
       },
    )

The search field captures the user's query and only applies it when the input
is submitted or the arrow button is pressed. This lets the page preview typing
without immediately filtering the grid.

Filter toggle
-------------

.. code-block:: dart

    IconButton(
       onPressed: () {
          _toggleFiltersDropdown();
       },
       icon: const Icon(Icons.tune_rounded),
    )

This button expands or collapses the advanced filter panel. It gives the page a
compact filter entry point without taking space away from the search bar.

Date filters
------------

.. code-block:: dart

    Wrap(
       children: _dateFilters.keys.map((filter) { ... }).toList(),
    )

The date filter section renders a set of selectable chips for time buckets such
as today, tomorrow, and next month. Each chip updates the `_dateFilters` map
through `_setDateFilter`.

Price filters
-------------

.. code-block:: dart

    Wrap(
       children: _priceFilters.keys.map((filter) { ... }).toList(),
    )

The price filter section mirrors the date filters and lets the user toggle free
and paid event categories. The state is stored in `_priceFilters` and updated
through `_setPriceFilter`.

Category chips
--------------

.. code-block:: dart

    Wrap(
       spacing: 10,
       runSpacing: 10,
       children: _filterChips.map((chip) { ... }).toList(),
    )

These chips provide the main category filter for the event grid. The selected
chip is highlighted and updates `_selectedFilter` when tapped.

Search summary
--------------

.. code-block:: dart

    Text('Showing results for \'$_submittedSearchQuery\'')

This text only appears after a query has been submitted. It confirms which
search term is currently being applied to the visible event cards.

Event grid
----------

.. code-block:: dart

    GridView.count(
       crossAxisCount: crossAxisCount,
       crossAxisSpacing: 16,
       mainAxisSpacing: 16,
       shrinkWrap: true,
       physics: const NeverScrollableScrollPhysics(),
       children: _filteredEvents.map((event) { ... }).toList(),
    )

The grid renders the filtered event cards in one, two, or three columns based
on available width. It uses the filtered event list so the search and category
controls directly shape the visible results.

Event card
----------

.. code-block:: dart

    Container(
       child: Column(
          children: [
             Expanded(...),
             Padding(...),
          ],
       ),
    )

Each card shows a colored media area, event title, date, location, category
badge, and a `View details` button. The button opens `EventDetailsScreen` with
the selected event map.

Footer
------

.. code-block:: dart

    const AppFooter(),

The shared footer closes the page and keeps the discovery screen consistent
with the rest of the site layout.

Summary
-------

This file is built around one stateful discovery page and a set of responsive
sections for search, filters, and results. The state class controls the active
filters and search query, while the UI widgets handle the breadcrumb, the
search controls, the advanced filter panel, and the event card grid.