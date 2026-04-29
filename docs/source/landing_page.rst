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

Main Landing Page Widget
------------------------
.. code-block:: dart

  class LandingPage extends StatelessWidget {
    const LandingPage({super.key});

    @override
    Widget build(BuildContext context) {
      return Scaffold(
        backgroundColor: AppColors.background,
        body: SingleChildScrollView(
          child: Column(
            children: [
              AppHeader(
                onHostEventTap: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const CreateEventScreen(),
                    ),
                  );
                },
                onRegisterTap: () {
                  Navigator.pushNamed(context, '/register');
                },
                onFindEventsTap: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const DiscoverEventScreen(),
                    ),
                  );
                },
                onCreateEventsTap: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const CreateEventScreen(),
                    ),
                  );
                },
                onMyTicketsTap: () {},
                onAboutTap: () {},
                onSignInTap: () {
                  Navigator.pushNamed(context, '/login');
                },
                showProfile: false,
              ),
              _HeroSection(),
              _StatsSection(),
              _AudienceSection(),
              _HowItWorksSection(),
              _CTASection(),
              const AppFooter(),
            ],
          ),
        ),
      );
    }
  }
The ``LandingPage`` class builds a scrollable ``Scaffold`` that presents the ``AppHeader`` first,
then stacks the main sections (hero, stats, audience, how-it-works, and CTA) before the footer.
Header actions provide immediate navigation to register, sign in, discover events, or create an event.

AppColors
---------
.. code-block:: dart

  class AppColors {
    static const background = Color(0xfff5f7fb);
    static const surface = Colors.white;
    static const primary = Color(0xff4f46e5);
    static const primaryDark = Color(0xff3730a3);
    static const accent = Color(0xffeef2ff);
    static const text = Color(0xff111827);
    static const muted = Color(0xff6b7280);
    static const border = Color(0xffe5e7eb);
  }

This function centralises the landing page color palette into reusable constants.

AppSpacing
----------
.. code-block:: dart

  class AppSpacing {
    static const double sectionY = 90;
    static const double sectionX = 24;
    static const double maxWidth = 1200;
  }

This function defines shared layout values for section padding and content width.






