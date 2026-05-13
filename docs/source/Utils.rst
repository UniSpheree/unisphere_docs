utils
=====


event_categories.dart
---------------------
'event_categories.dart' defines the list of the event categories which are used for event discovery and filtering on the unisphere app. 
The categories are used to categorise events and allow users to easily find events that match their interests.

event category constants
^^^^^^^^^^^^^^^^^^^^^^^^

..code-block:: dart
    const List<String> kEventCategories = [
      'Technology',
      'Music',
      'Entertainment',
      'Career',
      'Sports',
      'Workshops',
      'Academic',
      'Social',
      'Other',
    ];

    const List<String> kEventFilterCategories = ['All', ...kEventCategories];

Defines the list of event categories, and a second filter list that includes an"All" option for the event discovery page.

event_date_filters.dart
-----------------------
'event_date_filters.dart' defines the date filter options and helper functions used for event discovery and filtering in the Unisphere app.
This utility file enable events to be filtered based on their date ranges.

Event date filter constants
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    const List<String> kEventDateFilters = [
      'today',
      'tomorrow',
      'this week',
      'next week',
      'this month',
      'next month',
    ];

A list of the supported date filters that can be applied to events for 
filtering events based on the date range. 

Date filter state helpers
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    bool hasActiveDateFilters(Map<String, bool> filters) {
      return filters.values.any((value) => value);
    }

Checks whether any date filters are currently enabled.

Date filtering logic
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    bool matchesDateFilters(
      dynamic dateValue,
      Map<String, bool> filters, {
      DateTime? reference,
    }) {
      ...
    }

Determines whether an event date matches one or more selected date filters.

pagination.dart
----------------
'pagination.dart' defines the pagination logic and helper functions
which are used for splitting the event list into multiple pages for 
better performance and user experience when browsing events. 

Pagination helpers
^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    int eventsPerPageForWidth(double width)

Returns the number of events displayed per page based on 
screen width.

.. code-block:: dart

    int totalPagesForLength(int length, int itemsPerPage)

Calculates total number of pages needed to display all items
based on number of items and items per page.

.. code-block:: dart

    int clampPageIndex(int pageIndex, int totalPages)

Ensures current page index is within valid bounds.

.. code-block:: dart

    List<T> paginateItems<T>(
      List<T> items,
      int pageIndex,
      int itemsPerPage,
    )

Returns items for the current page based on index and items per page.

unis.dart
----------
'unis.dart' defines the list of universities used for university selection when
creating an account on the Unisphere app.

.. code-block:: dart

    const List<String> ukUniversities = [
      'University of Oxford',
      'University of Cambridge',
      'Imperial College London',
      ...
    ];

