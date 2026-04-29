=========
main.dart
=========

``main.dart`` is the app entry file that initializes the Flutter application and 
defines the top-level navigation and configuration.

Imports
-------
.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'screens/create_event_screen.dart';
    import 'screens/profile_page.dart';
    import 'screens/landing_page.dart';
    import 'screens/landing_page_logged_in.dart';
    import 'screens/dashboard_screen.dart';
    import 'screens/forgot_password_screen.dart';
    import 'screens/login_screen.dart';
    import 'screens/profile_screen.dart';
    import 'screens/register_screen.dart';
    import 'utils/mock_backend.dart';

``material.dart`` provides the Flutter framework and Material Design widgets. 
The screen imports enable navigation across the app's seven routes. ``mock_backend`` 
is imported to retrieve the current user's name and role when navigating to the 
``/logged-in`` route, allowing ``PersonalizedLandingPage`` to display user-specific 
content.


main()
------
.. code-block:: dart

    void main() {
     runApp(const MyApp());
    }

This function calls ``runApp()`` with ``MyApp()`` as the root widget, which 
initializes the Flutter framework and renders the entire application. 

