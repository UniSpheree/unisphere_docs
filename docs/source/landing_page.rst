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

`material.dart` provides the core Flutter UI components. ``app_footer.dart`` and
``header.dart`` provide shared layout widgets used by the landing page.

Navigation using ``MaterialPageRoute`` appears in ``LandingPage``, ``_HeroText``,
``_HeroVisualState``, and ``_CTASection``. The map imports,
``package:flutter_map/flutter_map.dart`` and ``package:latlong2/latlong.dart``, are
used in ``_HeroVisualState`` to render ``FlutterMap`` and define event coordinates via
``LatLng``.

Page Composition
----------------

``LandingPage`` builds the screen in this order:

1. ``AppHeader``: top navigation and account actions.
2. ``_HeroSection``: headline, feature bullets, and map preview.
3. ``_StatsSection``: key metrics in a compact summary row.
4. ``_AudienceSection``: attendee and organiser value panels.
5. ``_HowItWorksSection``: three-step explainer cards.
6. ``_CTASection``: final call-to-action banner.
7. ``AppFooter``: page footer links and information.

Everything is wrapped in a ``SingleChildScrollView`` so the full landing page scrolls
as one continuous screen.







