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