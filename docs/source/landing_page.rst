=================
landing_page.dart
=================

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

This class centralises the landing page color palette into reusable constants.

AppSpacing
----------
.. code-block:: dart

  class AppSpacing {
    static const double sectionY = 90;
    static const double sectionX = 24;
    static const double maxWidth = 1200;
  }

This class defines shared layout values for section padding and content width.

AppTextStyles
-------------
.. code-block:: dart

  class AppTextStyles {
    static const TextStyle heroTitle = TextStyle(
      fontSize: 52,
      fontWeight: FontWeight.w800,
      color: AppColors.text,
      height: 1.1,
    );

    static const TextStyle sectionTitle = TextStyle(
      fontSize: 36,
      fontWeight: FontWeight.w700,
      color: AppColors.text,
      height: 1.2,
    );

    static const TextStyle sectionSubtitle = TextStyle(
      fontSize: 17,
      color: AppColors.muted,
      height: 1.6,
    );

    static const TextStyle cardTitle = TextStyle(
      fontSize: 18,
      fontWeight: FontWeight.w700,
      color: AppColors.text,
    );

    static const TextStyle body = TextStyle(
      fontSize: 15,
      color: AppColors.muted,
      height: 1.6,
    );
  }

This class stores reusable typography styles for headings, section text, 
cards, and body content.

_SectionContainer
-----------------
.. code-block:: dart

  class _SectionContainer extends StatelessWidget {
    final Widget child;
    final EdgeInsetsGeometry? padding;
    final Color? color;

    const _SectionContainer({required this.child, this.padding, this.color});

    @override
    Widget build(BuildContext context) {
      return Container(
        width: double.infinity,
        color: color,
        padding:
            padding ??
            const EdgeInsets.symmetric(
              horizontal: AppSpacing.sectionX,
              vertical: AppSpacing.sectionY,
            ),
        child: Center(
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: AppSpacing.maxWidth),
            child: child,
          ),
        ),
      );
    }
  }

This wrapper takes a required ``child`` widget plus optional ``padding`` and ``color`` values.
It renders the child inside a full-width container with shared spacing defaults and a constrained
max width for consistent section alignment.


_HeroSection
------------
.. code-block:: dart

  class _HeroSection extends StatelessWidget {
    const _HeroSection();

    @override
    Widget build(BuildContext context) {
      final width = MediaQuery.of(context).size.width;
      final isMobile = width < 920;

      return _SectionContainer(
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 80),
        child: isMobile
            ? Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: const [
                  _HeroText(),
                  SizedBox(height: 40),
                  _HeroVisual(),
                ],
              )
            : const Row(
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  Expanded(flex: 11, child: _HeroText()),
                  SizedBox(width: 40),
                  Expanded(flex: 10, child: _HeroVisual()),
                ],
              ),
      );
    }
  }

This class is a stateless widget dynamically displays the ``_HeroText()``, 
and ``_HeroVisual()`` widgets within a wrapper. The attributes defined are to check
the user's screen and dynamically change the format to a smaller non-expanding column
if true.

_HeroText
---------
.. code-block:: dart

  class _HeroText extends StatelessWidget {
    const _HeroText();

    @override
    Widget build(BuildContext context) {
      return Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 8),
            decoration: BoxDecoration(
              color: AppColors.accent,
              borderRadius: BorderRadius.circular(999),
            ),
            child: const Text(
              'Event discovery & organiser tools in one platform',
              style: TextStyle(
                fontSize: 13,
                fontWeight: FontWeight.w600,
                color: AppColors.primaryDark,
              ),
            ),
          ),
          const SizedBox(height: 24),
          const Text(
            'Discover local events.\nHost unforgettable ones.',
            style: AppTextStyles.heroTitle,
          ),
          const SizedBox(height: 22),
          const Text(
            'UniSphere helps attendees explore nearby events through a live map, '
            'smart filters, and social discovery tools — while giving organisers '
            'everything they need to create, promote, manage, and grow events.',
            style: AppTextStyles.sectionSubtitle,
          ),
          const SizedBox(height: 28),
          const Wrap(
            spacing: 18,
            runSpacing: 12,
            children: [
              _HeroBullet(text: 'Live event map'),
              _HeroBullet(text: 'Social sharing'),
              _HeroBullet(text: 'Ticket management'),
              _HeroBullet(text: 'Organiser dashboard'),
            ],
          ),
          const SizedBox(height: 34),
          Wrap(
            spacing: 14,
            runSpacing: 14,
            children: [
              ElevatedButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const DiscoverEventScreen(),
                    ),
                  );
                },
                style: ButtonStyle(
                  backgroundColor: MaterialStatePropertyAll(AppColors.primary),
                  foregroundColor: MaterialStatePropertyAll(Colors.white),
                  elevation: MaterialStatePropertyAll(0),
                  padding: MaterialStatePropertyAll(
                    EdgeInsets.symmetric(horizontal: 24, vertical: 18),
                  ),
                  shape: MaterialStatePropertyAll(
                    RoundedRectangleBorder(
                      borderRadius: BorderRadius.all(Radius.circular(14)),
                    ),
                  ),
                ),
                child: Text('Discover Events'),
              ),
              OutlinedButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const CreateEventScreen(),
                    ),
                  );
                },
                style: ButtonStyle(
                  foregroundColor: MaterialStatePropertyAll(AppColors.primary),
                  side: MaterialStatePropertyAll(
                    BorderSide(color: AppColors.border),
                  ),
                  padding: MaterialStatePropertyAll(
                    EdgeInsets.symmetric(horizontal: 24, vertical: 18),
                  ),
                  shape: MaterialStatePropertyAll(
                    RoundedRectangleBorder(
                      borderRadius: BorderRadius.all(Radius.circular(14)),
                    ),
                  ),
                ),
                child: const Text('Host an Event'),
              ),
            ],
          ),
        ],
      );
    }
  }

The _HeroText class is a StatelessWidget. It builds the main hero content for the
landing page, including the badge, heading, supporting text, feature bullets, and 
action buttons. It returns a Column widget that lays out the text and navigation 
controls used in the hero section.

_HeroBullet
-----------
.. code-block:: dart

  class _HeroBullet extends StatelessWidget {
    final String text;

    const _HeroBullet({required this.text});

    @override
    Widget build(BuildContext context) {
      return Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          const Icon(
            Icons.check_circle_rounded,
            color: AppColors.primary,
            size: 18,
          ),
          const SizedBox(width: 8),
          Text(
            text,
            style: const TextStyle(
              color: AppColors.text,
              fontWeight: FontWeight.w500,
            ),
          ),
        ],
      );
    }
  }

The _HeroBullet class is a StatelessWidget. It takes a required text value as
a parameter and uses it to build a compact feature bullet with a check icon. It 
returns a Row widget that displays the icon and label side by side.

_HeroVisual
-----------
.. code-block:: dart

  class _HeroVisual extends StatefulWidget {
    const _HeroVisual();

    static final List<_EventMarkerData> events = [
      _EventMarkerData(
        title: 'Tech Meetup',
        subtitle: 'Today • 6:30 PM',
        location: LatLng(50.8198, -1.0880), // Portsmouth
        count: 9,
        color: Color(0xff9bd36a),
      ),
      _EventMarkerData(
        title: 'Live Music Night',
        subtitle: 'Fri • 8:00 PM',
        location: LatLng(51.5072, -0.1276), // London
        count: 19,
        color: Color(0xffe8c75f),
      ),
      _EventMarkerData(
        title: 'Food Festival',
        subtitle: 'Sat • 1:00 PM',
        location: LatLng(52.4862, -1.8904), // Birmingham
        count: 12,
        color: Color(0xff9bd36a),
      ),
      _EventMarkerData(
        title: 'Startup Talks',
        subtitle: 'Sun • 5:00 PM',
        location: LatLng(53.4808, -2.2426), // Manchester
        count: 7,
        color: Color(0xff9bd36a),
      ),
    ];

    @override
    State<_HeroVisual> createState() => _HeroVisualState();
  }

The _HeroVisual class is a StatefulWidget. It stores the shared event marker data 
used by the landing page map and returns an instance of the _HeroVisualState class. 
The widget is used to display the interactive map preview in the hero section.

_HeroVisualState
----------------
.. code-block:: dart

  class _HeroVisualState extends State<_HeroVisual> {
    late TextEditingController _searchController;

    @override
    void initState() {
      super.initState();
      _searchController = TextEditingController();
    }

    @override
    void dispose() {
      _searchController.dispose();
      super.dispose();
    }

    @override
    Widget build(BuildContext context) {
      return Container(
        height: 430,
        decoration: BoxDecoration(
          gradient: const LinearGradient(
            colors: [Color(0xffeef2ff), Color(0xffffffff)],
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
          ),
          borderRadius: BorderRadius.circular(28),
          border: Border.all(color: AppColors.border),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.06),
              blurRadius: 30,
              offset: const Offset(0, 16),
            ),
          ],
        ),
        child: Stack(
          children: [
            Positioned(
              top: 24,
              left: 24,
              right: 24,
              child: Container(
                height: 72,
                padding: const EdgeInsets.symmetric(horizontal: 18),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(18),
                  border: Border.all(color: AppColors.border),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.search_rounded, color: AppColors.muted),
                    const SizedBox(width: 12),
                    Expanded(
                      child: TextField(
                        controller: _searchController,
                        onSubmitted: (value) {
                          if (value.isNotEmpty) {
                            Navigator.push(
                              context,
                              MaterialPageRoute(
                                builder: (context) => DiscoverEventScreen(
                                  initialSearchQuery: value,
                                ),
                              ),
                            );
                          }
                        },
                        decoration: const InputDecoration(
                          hintText: 'Search events, categories, places...',
                          hintStyle: TextStyle(color: AppColors.muted),
                          border: InputBorder.none,
                          contentPadding: EdgeInsets.zero,
                        ),
                        style: const TextStyle(color: AppColors.text),
                      ),
                    ),
                    const Icon(Icons.tune_rounded, color: AppColors.primary),
                  ],
                ),
              ),
            ),
            Positioned(
              left: 24,
              right: 24,
              top: 120,
              bottom: 24,
              child: ClipRRect(
                borderRadius: BorderRadius.circular(22),
                child: FlutterMap(
                  options: MapOptions(
                    initialCenter: LatLng(51.0, -0.8),
                    initialZoom: 5.5,
                  ),
                  children: [
                    TileLayer(
                      urlTemplate:
                          'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
                      userAgentPackageName: 'com.example.unisphere_app',
                    ),
                    MarkerLayer(
                      markers: _HeroVisual.events.map((event) {
                        return Marker(
                          point: event.location,
                          width: 60,
                          height: 60,
                          child: _EventBubble(
                            count: event.count,
                            color: event.color,
                          ),
                        );
                      }).toList(),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      );
    }
  }

The _HeroVisualState class is the state class for _HeroVisual. It manages the 
search text controller, builds the map preview container, and displays the 
interactive FlutterMap with event markers. It also handles search submission 
by navigating to DiscoverEventScreen with the entered query.
