Landing Page
============

``landing_page.dart`` is the first screen that a new user sees when they load the
website, so it acts as the main first impression for the app.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'package:unisphere_app/widgets/app_footer.dart';
   import 'package:unisphere_app/widgets/header.dart';
   import 'create_event_screen.dart';
   import 'discover_event_screen.dart';
   import 'package:flutter_map/flutter_map.dart';
   import 'package:latlong2/latlong.dart';

Imports material.dart as it acts as the framework for flutter. Import standardised widgets, 
``app_footer.dart`` and ``header.dart``, for easier navigation and cleaner workspace. 
The screen imports are for navigation through ``MaterialPageRoute``, found in classes ``LandingPage()``,
``_HeroText()``, ``_HeroVisualState()``, and ``_CTASection()``. The ``flutter_map`` and 
``latlong.dart`` import are for the live interactive maps in the class ``_HeroVisual()``,
which also calculates the latitude of a given location.




