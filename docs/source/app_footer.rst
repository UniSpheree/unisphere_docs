app_footer.dart
=================

``app_footer.dart`` is the reusable footer widget for the UniSphere application. 
The footer provides a consistent footer section across the app, which contains links
to further important information for users.

Imports
--------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'package:unisphere_app/services/sqlite_backend.dart';

'material.dart' provides the Flutter UI framework and widgets used by the footer.
'sqlite_backend.dart' is imported to access user authentication state and conditionally render certain links in the footer based on whether a user is logged in or not.

widget
------

.. code-block:: dart

    class AppFooter extends StatelessWidget {
      final String brandName;
      final String tagline;
      final String copyrightText;

The ``AppFooter`` widget is defined as a stateless widget used throughout the 
app to display a consistent footer section for branding and information. It 
contains configurable properties for the brand name, tagline and copyright 
information.

Responsive Layout
-----------------

.. code-block:: dart 

    final isMobile = constraints.maxWidth < 600;

The footer utilises a responsive layout which makes it adapt to different screen 
sizes.

Navigation Links
----------------

.. code-block:: dart

    Navigator.pushNamed(context, '/about');

The footer contains navigation links to important informational pages such as the
about page, privacy, terms and other pages.

Authentication-aware navigation
--------------------------------

.. code-block:: dart

    if (requiresAuth &&
        SqliteBackend().currentUser == null)
      Navigator.pushNamed(context, '/register');
       } else {
         Navigator.pushNamed(context, route);
       }

The footer uses authentication-aware navigation to conditionally allow users
to access certain pages based on their authentication state. Certain routes
require the user to be logged in before they can access them. 

The widget will check whether a selected route requires authentication and verifies
a user's logged in state using:

.. code-block:: dart

   SqliteBackend().currentUser

If a user is not logged in and tries to access an authenticated route, they are
then redirected to the registration page, otherwise navigation will proceed
as normal. 