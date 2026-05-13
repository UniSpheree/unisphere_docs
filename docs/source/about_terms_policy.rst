About & Policies Pages
======================

This sections contains the about, terms and privacy screens of the Unisphere app. Each of the screens follows a very similar structure and design, 
using the reusable header and footer widgets, which are used across the app to provide easier, consistent navigation as well as information for the user.


about_us_page.dart
-------------

``about_us_page.dart`` is a stateless page that displays information about the Unisphere project, such as it's purpose and goals. The page uses the 
reusable header and footer widgets, which are used accross the app to provide easier, consistent navigation as well as information for the user. 

privacy_page.dart
-----------------

``privacy_page.dart`` is a stateless page for displaying the privacy policy for the UniSphere application. This page uses the reusable header and footer widgets,
which are used across the app to provide easier, consistent navigation as well as information for the user.


terms_page.dart
----------------

``terms_page.dart`` is a stateless page for displaying the terms of service for the UniSphere application. This page uses the reusable header and footer widgets, 
which are used across the app to provide easier, consistent navigation as well as information for the user.


Structure
---------

Each of the screens in this section are implemented using a ``StatelessWidget`` and includes:

- ``AppHeader`` for application navigation
- Informational content about UniSphere
- ``AppFooter`` for consistent page structure
- A responsive scrollable layout using ``SingleChildScrollView``
