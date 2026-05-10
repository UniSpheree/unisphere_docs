===========================
Header.dart
===========================

'Header.dart' is the navigation widget which is used throughout the Unisphere app to provide a consistent and responsive navigation 
experience for users, containing links to key screens within the application. The header adapts to different screen sizes, 
making it suitable for both mobile and desktop platforms.


Imports
--------

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:unisphere_app/screens/create_event_screen.dart';
    import 'package:unisphere_app/services/sqlite_backend.dart';

material.dart provides the Flutter UI framework and widgets used throughout the header. 
The create_event_screen import allows the header's "Create Event" button to navigate to the event creation page. 
The sqlite_backend import is to provide authentication state and user information for the conditional rendering of navigation options 
depending on whether a user is currently logged into the app.

Header Widget
---------

.. code-block:: dart

   class AppHeader extends StatelessWidget

The ``AppHeader`` widget is the primary widget for the header component. It provides responsive navigation, authentication-aware UI behaviour, 
and route-based navigation across the UniSphere application.

Constructor
-------
.. code-block:: dart

     const AppHeader({
    super.key,
    this.onFindEventsTap,
    this.onCreateEventsTap,
    this.onMyTicketsTap,
    this.onAboutTap,
    this.onSignInTap,
    this.onHostEventTap,
    this.onRegisterTap,
    this.showProfile = true,
    this.showBackButton = false,
  });

The constructor defines the configurable navigation callbacks and UI options for the header widget, allowing different screens to customise the 
navigation and control certain visible elements like the profile icon and back button. 