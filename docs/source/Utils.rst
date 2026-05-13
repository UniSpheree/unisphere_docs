utils
=====


event_categories.dart
---------------------
'event_categories.dart' defines the list of the event categories which are used for event discovery and filtering on the unisphere app. 
The categories are used to categorise events and allow users to easily find events that match their interests.

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
----------------------
