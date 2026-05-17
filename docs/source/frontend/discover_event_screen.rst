=================================================
Event discovery page - discover_event_screen.dart
=================================================

``discover_event_screen.dart`` is the event discovery screen. It provides the
search bar, category chips, date filters, a paginated event grid, and the
event cards used to browse available events before opening the full details
page. The implementation is backed by `SqliteBackend()` so the UI listens for
live updates to the event store.

Imports
-------

.. code-block:: dart

    import 'dart:typed_data';
    import 'package:flutter/material.dart';
    import '../services/sqlite_backend.dart';
    import '../utils/event_categories.dart';
    import '../utils/event_date_filters.dart';
    import '../utils/pagination.dart';
    import '../widgets/app_footer.dart';
    import '../widgets/header.dart';
    import '../widgets/pagination_controls.dart';
    import 'event_details_screen.dart';

This import block brings in Flutter widgets, typed data support, the SQLite
backend service, and the helper modules that provide category chips, date
filter logic, pagination, and shared page chrome. It is the foundation for the
rest of the screen because the page needs both the backend data and the helper
widgets to render the discovery flow.

The `SqliteBackend` import is the important source-level change here because
the screen now listens to the real backend instead of a mock service. The other
imports support the widgets that follow, especially the header, footer,
pagination controls, and the event-details route.

App header
----------

.. code-block:: dart

        AppHeader(
            onHostEventTap: () => Navigator.pushNamed(context, '/create-event'),
            onRegisterTap: () => Navigator.pushNamed(context, '/register'),
            onFindEventsTap: () {},
            onCreateEventsTap: () => Navigator.pushNamed(context, '/create-event'),
            onMyTicketsTap: () => Navigator.pushNamed(context, '/my-tickets'),
            onAboutTap: () => Navigator.pushNamed(context, '/about'),
            onSignInTap: () => Navigator.pushNamed(context, '/login'),
            showProfile: true,
        )

`AppHeader` is the shared navigation bar at the top of the page. It gives the
screen access to the other primary routes and keeps the discovery view aligned
with the rest of the app.

This header is reused across pages, so the discovery screen only supplies the
navigation callbacks it needs. The find-events action is intentionally left
empty because the user is already on the discovery page.

Main Discover Widget
--------------------

.. code-block:: dart

    class DiscoverEventScreen extends StatefulWidget {
       final String? initialSearchQuery;
       const DiscoverEventScreen({super.key, this.initialSearchQuery});
    }

`DiscoverEventScreen` is the top-level `StatefulWidget` for the discovery page.
It accepts an optional `initialSearchQuery` so other screens can open the page
with a prefilled search term, then it creates the state object that manages the
filtering, search, and backend-driven refresh logic.

If `initialSearchQuery` is supplied, the state copies it into the search
controller and submitted query during initialization. That lets the page start
already filtered instead of requiring the user to type the same search again.

Discover Event State
--------------------

.. code-block:: dart

    class _DiscoverEventScreenState extends State<DiscoverEventScreen> {
       bool _showFiltersDropdown = false;
       String _selectedFilter = 'All';
       late final TextEditingController _searchController;
       String _submittedSearchQuery = '';
       late Map<String, bool> _dateFilters;
    }

`_DiscoverEventScreenState` stores the page state for the discovery flow. It
tracks whether the advanced filters are visible, which category chip is
selected, the search controller, the submitted search text, and the active date
filter map.

These fields are the inputs for the computed getters later in the file. The
filter chips, search bar, and pagination widgets all depend on this state so
the grid can rebuild when the user changes the visible subset of events.

The date filter map is initialised from `kEventDateFilters` and the search
controller is created in `initState`, so both must be cleaned up correctly when
the page is removed.

Lifecycle methods (initState / dispose)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    @override
    void initState() {
       super.initState();
       _searchController = TextEditingController();
       _dateFilters = {for (final item in kEventDateFilters) item: false};
       SqliteBackend().addListener(_onBackendChanged);
       if (widget.initialSearchQuery != null) { ... }
    }

`initState` is where the screen prepares itself for use. It creates the search
controller, builds the default date filter map, and registers `_onBackendChanged`
with `SqliteBackend()` so the grid updates when the backend data changes.

`dispose` performs the matching cleanup by removing the backend listener and
disposing the controller. Those steps matter because this screen depends on a
live listener and a text controller that should not survive after the widget is
removed.

Backend listener
^^^^^^^^^^^^^^^^

.. code-block:: dart

    void _onBackendChanged() {
       if (!mounted) return;
       setState(() {});
    }

`_onBackendChanged` is the callback that the backend listener uses to refresh
the page. It simply calls `setState` so the filtered list, cards, and counts can
rebuild when the stored events change.

The `mounted` check is important because the callback may fire after the page
has already been popped. Without that guard, the screen could try to update a
state object that no longer exists.

Breadcrumb row
--------------

.. code-block:: dart

        Row(
            children: [
                InkWell(
                    onTap: () => Navigator.pop(context),
                    child: const Icon(Icons.arrow_back),
                ),
                const SizedBox(width: 8),
                GestureDetector(
                    onTap: () => Navigator.pop(context),
                    child: const Text('Landing Page'),
                ),
                const Text('  /  '),
                const Flexible(child: Text('Discover Events')),
            ],
        )

This breadcrumb row gives the page a back navigation point and shows the
current section title. It sits above the main content so users can quickly
return to the landing page if they reached discovery from elsewhere.

The row uses both an `InkWell` and a tappable text label so the back action is
easy to reach. That keeps the navigation affordance obvious while still fitting
within the compact page header area.

Searching & Filtering
---------------------
The code for the search and filtering functions of the event discovery page

Search bar
^^^^^^^^^^

.. code-block:: dart

        TextField(
            controller: _searchController,
            onChanged: (value) {
                setState(() {
                    _submittedSearchQuery = value;
                });
            },
            onSubmitted: (value) {
                setState(() {
                    _submittedSearchQuery = value;
                });
            },
        )

The search field captures the user's query and stores it in the submitted
search state. It is used to narrow the event list before the paginated grid is
rendered.

The controller is created in `initState` so the field can also be prefilled
from `initialSearchQuery`. That makes it possible for other screens to open the
discovery page with a search term already applied.

Filter toggle
^^^^^^^^^^^^^

.. code-block:: dart

        IconButton(
            onPressed: () {
                setState(() {
                    _showFiltersDropdown = !_showFiltersDropdown;
                });
            },
            icon: const Icon(Icons.tune_rounded),
            tooltip: 'Filter events',
        )

This button toggles the advanced filter panel open and closed. It gives the
page a compact way to expose the date filter controls without taking space away
from the main search bar.

The filter panel state lives in `_showFiltersDropdown`, so the button only needs
to flip that flag. The actual filter values are stored separately in
`_dateFilters`.

Date filters
^^^^^^^^^^^^

.. code-block:: dart

        Wrap(
            spacing: 12,
            runSpacing: 10,
            children: kEventDateFilters.map((filter) { ... }).toList(),
        )

The date filter chips let the user narrow the discovery list by time buckets.
They are used later by `_filteredEvents` when the page decides which events are
visible.

The filter map is created from `kEventDateFilters` in `initState`, so each chip
has a matching state entry from the start. That keeps the UI and the filtering
logic in sync.

Category chips
^^^^^^^^^^^^^^

.. code-block:: dart

        Wrap(
            spacing: 10,
            runSpacing: 10,
            children: _filterChips.map((chip) { ... }).toList(),
        )

The category chips give the discovery page its main event category filter.
They update `_selectedFilter`, which is then used by `_filteredEvents` before
the grid is paginated.

The chip list comes from `kEventFilterCategories`, so the categories stay
consistent with the rest of the app. The selected chip changes style to make
the active filter easy to see.

Search summary
^^^^^^^^^^^^^^

.. code-block:: dart

        if (_submittedSearchQuery.isNotEmpty)
            Padding(
                padding: const EdgeInsets.only(top: 16),
                child: Text('Showing results for "$_submittedSearchQuery"'),
            )

This text confirms which search term is currently being applied. It appears
only after the user has entered or submitted a query so the page can show the
active filter state clearly.

The summary sits directly above the paginated grid, which helps the user see
why the results have changed. It is tied to `_submittedSearchQuery`, not the raw
text field content, so it reflects the committed search state.



.. code-block:: dart

    void _setFilter(String filter) { ... }
    void _setDateFilter(String filter, bool value) { ... }

These are the filter helper methods that update the selected category and date filter values. They
are called from the chips and other filter controls so the UI can stay simple
while the state object owns the actual filter values.

The helpers only wrap `setState`; they do not perform the filtering themselves.
That work happens in the computed getters later in the file so the screen keeps
the update logic in one place.



.. code-block:: dart

    List<Map<String, dynamic>> get _discoverEvents {
      return SqliteBackend().events
          .where((e) => e['title']?.toString().toLowerCase() != 'demo event' &&
                        e['visibility']?.toString() != 'Private')
          .map((event) => { 'id': ..., 'bannerImageData': event['bannerImageData'], ... })
          .toList();
    }
`_discoverEvents` is the event data getter that converts the raw backend rows into the smaller map structure
used by the widgets on this page. It pulls out the event title, date,
location, category, organizer, and banner image data so the UI does not have to
work with the backend representation directly.

This getter is the base list used by the filtering logic and the paginated grid.
The later widgets rely on this normalized shape, which is why it also preserves
fields such as `bannerImageData` so the cards can decide whether to show an
image or a fallback icon.

The getter skips demo events and private events. It also keeps the banner bytes
as `Uint8List` or `null` so the card widget can treat the image data safely.


.. code-block:: dart

    List<Map<String, dynamic>> get _filteredEvents {
      final baseEvents = _selectedFilter == 'All' ? _discoverEvents : ...
      final dateFiltered = baseEvents.where((e) => matchesDateFilters(e['date'], _dateFilters)).toList();
      if (_submittedSearchQuery.isEmpty) return dateFiltered;
      // case-insensitive title/category/location match
      return dateFiltered.where((...) => ...).toList();
    }

The filtering helper `_filteredEvents` is the computed list that applies the current category and
date filters, then narrows the result with the submitted search query. It is
the list that eventually gets passed to the paginated grid.

The getter works in stages so later widgets can rely on its output without
knowing about the filtering rules. `matchesDateFilters` from
`event_date_filters.dart` handles the date-bucket checks, and the text search is
applied only after the user submits the query.

That separation matters because the page allows preview typing in the search
field without filtering immediately. The actual search term is only committed
when the user submits it or presses the search action.

Pagination
----------

.. code-block:: dart

    class _PaginatedDiscoverGrid extends StatefulWidget { ... }

        class _PaginatedDiscoverGridState extends State<_PaginatedDiscoverGrid> {
            int _currentPage = 0;

            @override
            void didUpdateWidget(covariant _PaginatedDiscoverGrid oldWidget) {
                super.didUpdateWidget(oldWidget);
                if (oldWidget.events.length != widget.events.length ||
                        oldWidget.emptyMessage != widget.emptyMessage) {
                    _currentPage = 0;
                }
            }

            @override
            Widget build(BuildContext context) {
                return LayoutBuilder(
                    builder: (context, constraints) {
                        final availableWidth = constraints.maxWidth;
                        final itemsPerPage = eventsPerPageForWidth(availableWidth);
                        final totalPages = totalPagesForLength(
                            widget.events.length,
                            itemsPerPage,
                        );
                        final pageIndex = clampPageIndex(_currentPage, totalPages);
                        final pageEvents = paginateItems(
                            widget.events,
                            pageIndex,
                            itemsPerPage,
                        );

                        if (totalPages > 0 && pageIndex != _currentPage) {
                            WidgetsBinding.instance.addPostFrameCallback((_) {
                                if (!mounted) return;
                                setState(() => _currentPage = pageIndex);
                            });
                        }

                        return Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                                GridView.builder(
                                    itemCount: pageEvents.length,
                                    shrinkWrap: true,
                                    physics: const NeverScrollableScrollPhysics(),
                                    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                                        crossAxisCount: availableWidth > 800
                                                ? 3
                                                : (availableWidth > 400 ? 2 : 1),
                                        crossAxisSpacing: 16,
                                        mainAxisSpacing: 16,
                                        childAspectRatio: 0.72,
                                    ),
                                    itemBuilder: (context, index) =>
                                            _DiscoverEventCard(event: pageEvents[index]),
                                ),
                                PaginationControls(
                                    currentPage: pageIndex,
                                    totalPages: totalPages,
                                    onPrevious: () {
                                        setState(() {
                                            _currentPage -= 1;
                                        });
                                    },
                                    onNext: () {
                                        setState(() {
                                            _currentPage += 1;
                                        });
                                    },
                                ),
                            ],
                        );
                    },
                );
            }
        }

`_PaginatedDiscoverGrid` is the widget that wraps the event grid and pagination
controls. It exists so the discovery page can show many events without making
the whole page vertically huge.

It calculates how many items should appear on each page based on the available
width, slices the current page with the shared pagination helpers, and renders
the visible cards together with `PaginationControls` so the user can move
between pages.

The widget resets its page index whenever the event list or empty-state text
changes, and it uses `addPostFrameCallback` to recover cleanly if the layout
changes the number of pages after build. That keeps the current page valid when
the window size changes.

Event card
----------

.. code-block:: dart

    class _DiscoverEventCard extends StatelessWidget { ... }

        @override
        Widget build(BuildContext context) {
            final bannerData = event['bannerImageData'];
            final Uint8List? bannerBytes = bannerData is Uint8List ? bannerData : null;
            final icon = event['icon'] as IconData? ?? Icons.event_rounded;

            return Container(
                decoration: BoxDecoration(
                    color: Colors.white,
                    borderRadius: BorderRadius.circular(22),
                    border: Border.all(color: const Color(0xFFE5E7EB)),
                    boxShadow: [
                        BoxShadow(
                            color: Colors.black.withOpacity(0.04),
                            blurRadius: 16,
                            offset: const Offset(0, 8),
                        ),
                    ],
                ),
                child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                        Expanded(
                            flex: 2,
                            child: Container(
                                width: double.infinity,
                                decoration: BoxDecoration(
                                    color: event['imageColor'] as Color? ?? const Color(0xFFE0E7FF),
                                    borderRadius: const BorderRadius.vertical(
                                        top: Radius.circular(22),
                                    ),
                                ),
                                child: bannerBytes != null
                                        ? ClipRRect(
                                                borderRadius: const BorderRadius.vertical(
                                                    top: Radius.circular(22),
                                                ),
                                                child: Image.memory(
                                                    bannerBytes,
                                                    fit: BoxFit.cover,
                                                    errorBuilder: (context, error, stackTrace) => Center(
                                                        child: Icon(
                                                            icon,
                                                            size: 42,
                                                            color: const Color(0xFF4F46E5),
                                                        ),
                                                    ),
                                                ),
                                            )
                                        : Center(
                                                child: Icon(
                                                    icon,
                                                    size: 42,
                                                    color: const Color(0xFF4F46E5),
                                                ),
                                            ),
                            ),
                        ),
                        Padding(
                            padding: const EdgeInsets.all(18),
                            child: Column(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                children: [
                                    Text(
                                        event['title']?.toString() ?? 'Untitled Event',
                                        maxLines: 1,
                                        overflow: TextOverflow.ellipsis,
                                        style: const TextStyle(
                                            fontSize: 18,
                                            fontWeight: FontWeight.w700,
                                            color: Color(0xFF1A1F36),
                                        ),
                                    ),
                                    const SizedBox(height: 8),
                                    Text(
                                        event['date']?.toString() ?? '',
                                        style: const TextStyle(
                                            fontSize: 14,
                                            color: Color(0xFF6B7280),
                                        ),
                                    ),
                                    const SizedBox(height: 4),
                                    Text(
                                        event['location']?.toString() ?? 'TBA',
                                        maxLines: 1,
                                        overflow: TextOverflow.ellipsis,
                                        style: const TextStyle(
                                            fontSize: 14,
                                            color: Color(0xFF6B7280),
                                        ),
                                    ),
                                    const SizedBox(height: 4),
                                    Text(
                                        'By ${event['organizer']?.toString() ?? 'UniSphere'}',
                                        style: const TextStyle(
                                            fontSize: 12,
                                            color: Color(0xFF9CA3AF),
                                            fontStyle: FontStyle.italic,
                                        ),
                                    ),
                                    const SizedBox(height: 14),
                                    Row(
                                        children: [
                                            Container(
                                                padding: const EdgeInsets.symmetric(
                                                    horizontal: 10,
                                                    vertical: 6,
                                                ),
                                                decoration: BoxDecoration(
                                                    color: const Color(0xFFEEF2FF),
                                                    borderRadius: BorderRadius.circular(999),
                                                ),
                                                child: Text(
                                                    event['category']?.toString() ?? 'Other',
                                                    style: const TextStyle(
                                                        fontSize: 12,
                                                        fontWeight: FontWeight.w600,
                                                        color: Color(0xFF4F46E5),
                                                    ),
                                                ),
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
                                                child: const Text('Details'),
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

`_DiscoverEventCard` is the stateless card widget used for each event in the
grid. It shows the event banner area, the title and metadata, a category badge,
and the details button.

The card is used by the paginated grid, so it only needs a single event map and
can stay focused on presentation. It navigates to `EventDetailsScreen` when the
user taps the details button, which is how this discovery page hands off to the
full event view.

The banner image data may be `Uint8List` or `null`, so the card checks the type
before calling `Image.memory`. If the image cannot be decoded, the `errorBuilder`
falls back to a centered icon instead of breaking the layout.

Search summary & UI bits
------------------------

The remaining UI pieces are the page shell and footer layout around the
discovery controls. They compose the visible screen and call back into the
state helpers when the user changes filters or submits a search.

The search field keeps typing and submission separate: typing updates the text
controller, but the actual filtering only uses `_submittedSearchQuery` after
the user commits the search. That keeps the page responsive without filtering
on every keystroke.

