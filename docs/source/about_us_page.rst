about_us_page.dart
===========================

``about_us_page.dart`` is a stateless page that displays information about the Unisphere project, such as it's purpose and goals. The page uses the 
reusable header and footer widgets, which are used accross the app to provide easier, consistent navigation as well as information for the user. 

File Location
-------------

::

   lib/pages/about_us_page.dart


Structure
---------

The screen is implemented using a ``StatelessWidget`` and includes:

- ``AppHeader`` for application navigation
- Informational content about UniSphere
- ``AppFooter`` for consistent page structure
- A responsive scrollable layout using ``SingleChildScrollView``