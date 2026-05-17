=================
landing_page.dart
=================

``landing_page.dart`` is the public-facing landing page for UniSphere.
It assembles the app's first impression and provides entry points for discovery,
event creation, registration, and sign-in.

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

These imports bring in the Flutter framework, the shared header and footer widgets,
the discovery and creation screens, and the map packages used by the hero preview.

Main landing page widget
------------------------

.. code-block:: dart

   class LandingPage extends StatelessWidget {
     const LandingPage({super.key});

     @override
     Widget build(BuildContext context) {
       return Scaffold(
         backgroundColor: AppColors.background,
         body: LayoutBuilder(
           builder: (context, constraints) {
             return SingleChildScrollView(
               child: ConstrainedBox(
                 constraints: BoxConstraints(minHeight: constraints.maxHeight),
                 child: Column(
                   mainAxisAlignment: MainAxisAlignment.spaceBetween,
                   children: [
                     Column(
                       children: [
                         AppHeader(...),
                         _HeroSection(),
                         _StatsSection(),
                         _AudienceSection(),
                         _HowItWorksSection(),
                         _CTASection(),
                       ],
                     ),
                     const AppFooter(),
                   ],
                 ),
               ),
             );
           },
         ),
       );
     }
   }

`LandingPage` is the top-level `StatelessWidget` for the screen.
It composes the full landing page by stacking the shared header, the hero, stats,
audience, how-it-works, and call-to-action sections before finishing with the footer.
The `LayoutBuilder` and `ConstrainedBox` keep the footer anchored on short pages while
still allowing the content to scroll naturally on smaller viewports.

Design constants
----------------

**AppColors**

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

`AppColors` is the shared colour palette for the landing page.
It defines the background, surface, brand, text, muted, and border values used by the
sections, cards, buttons, and map preview. Centralising these values keeps the page visually
consistent and makes later style adjustments straightforward.

**AppSpacing**

.. code-block:: dart

   class AppSpacing {
     static const double sectionY = 90;
     static const double sectionX = 24;
     static const double maxWidth = 1200;
   }

`AppSpacing` is the layout spacing constant set for the page.
It defines the shared section padding and maximum content width that the later widgets rely
on to stay aligned across wide and narrow screens. The max width is especially important
because it controls how the centered page content is constrained on desktop.

**AppTextStyles**

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

`AppTextStyles` is the shared typography set for the landing page.
It defines the text styles reused by the hero, section headings, subtitles, card titles, and
body copy so the page reads as one cohesive design system. Later widgets depend on these
styles to avoid duplicating font sizes, weights, and line heights.

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
         padding: padding ?? const EdgeInsets.symmetric(
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

`_SectionContainer` is a reusable layout wrapper for full-width landing page sections.
It applies the standard spacing, optional background colour, and centered max-width
constraint that most of the page blocks use. This wrapper is the structural foundation for
the hero, stats, audience, how-it-works, and CTA sections, so changes here affect multiple
parts of the page at once.

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

`_HeroSection` is the responsive hero layout for the landing page.
It decides whether the hero content should sit beside the map preview or stack vertically
depending on screen width, then delegates the actual content to the text and visual widgets.
This section is the first major content block on the page, so its breakpoint behaviour shapes
the user's first impression on both desktop and mobile.

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
             'smart filters, and social discovery tools while giving organisers '
             'the tools they need to create, promote, and manage events.',
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
                 onPressed: () => Navigator.push(...),
                 child: const Text('Discover Events'),
               ),
               OutlinedButton(
                 onPressed: () => Navigator.push(...),
                 child: const Text('Host an Event'),
               ),
             ],
           ),
         ],
       );
     }
   }

`_HeroText` is the headline and call-to-action column inside the hero section.
It presents the tagline badge, main heading, summary copy, feature bullets, and the primary
buttons that take users into discovery or event creation. This block is the landing page's
main conversion surface, so the buttons connect directly to the later app flows.

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
           const Icon(Icons.check_circle_rounded, color: AppColors.primary, size: 18),
           const SizedBox(width: 8),
           Text(text, style: const TextStyle(color: AppColors.text, fontWeight: FontWeight.w500)),
         ],
       );
     }
   }

`_HeroBullet` is a compact feature marker used in the hero section.
It pairs a check icon with short descriptive text to summarise the platform's headline
capabilities at a glance. The hero uses several of these bullets to reinforce the value
proposition without adding visual clutter.

_HeroVisual
-----------

.. code-block:: dart

   class _HeroVisual extends StatefulWidget {
     const _HeroVisual();

     static final List<_EventMarkerData> events = [
       _EventMarkerData(...),
       _EventMarkerData(...),
     ];

     @override
     State<_HeroVisual> createState() => _HeroVisualState();
   }

`_HeroVisual` is the stateful map preview shown beside the hero text.
It stores the sample event marker data that the preview uses and creates
`_HeroVisualState` to manage the interactive map area. This widget exists so the hero can
show believable event activity before the user performs any search.

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
         child: Stack(
           children: [
             Positioned(...search bar...),
             Positioned(...FlutterMap...),
           ],
         ),
       );
     }
   }

`_HeroVisualState` manages the hero map preview.
It initialises and disposes the search controller, builds the search field, and renders the
`FlutterMap` with bubble markers for the sample events. When a search is submitted, it can
send the user into the discovery flow with that query already provided.

_EventMarkerData
----------------

.. code-block:: dart

   class _EventMarkerData {
     final String title;
     final String subtitle;
     final LatLng location;
     final int count;
     final Color color;

     const _EventMarkerData({
       required this.title,
       required this.subtitle,
       required this.location,
       required this.count,
       required this.color,
     });
   }

`_EventMarkerData` is the simple data model for a sample event marker.
It stores the title, subtitle, location, count, and colour that the map preview needs in
order to render each event bubble. The hero visual depends on this structure being ready to
consume without additional transformation.

_EventBubble
------------

.. code-block:: dart

   class _EventBubble extends StatelessWidget {
     final int count;
     final Color color;

     const _EventBubble({required this.count, required this.color});

     @override
     Widget build(BuildContext context) {
       return Container(
         decoration: BoxDecoration(
           color: color.withOpacity(0.92),
           shape: BoxShape.circle,
           border: Border.all(color: Colors.white, width: 3),
           boxShadow: [BoxShadow(color: Colors.black.withOpacity(0.16), blurRadius: 12, offset: Offset(0, 4))],
         ),
         alignment: Alignment.center,
         child: Text('$count', style: const TextStyle(color: Colors.black87, fontWeight: FontWeight.w700, fontSize: 16)),
       );
     }
   }

`_EventBubble` is the marker badge used on the hero map.
It draws a circular bubble with the event count in the centre so the preview can communicate
how many events are associated with each location. The styling keeps the marker readable while
still fitting the page's visual language.

_StatsSection
-------------

.. code-block:: dart

   class _StatsSection extends StatelessWidget {
     const _StatsSection();

     @override
     Widget build(BuildContext context) {
       return _SectionContainer(
         color: Colors.white,
         padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 28),
         child: Wrap(
           alignment: WrapAlignment.spaceBetween,
           runSpacing: 20,
           spacing: 20,
           children: const [
             _StatItem(value: '10K+', label: 'Events discovered'),
             _StatItem(value: '2K+', label: 'Active organisers'),
             _StatItem(value: '25+', label: 'Event categories'),
             _StatItem(value: '99%', label: 'Mobile-friendly experience'),
           ],
         ),
       );
     }
   }

`_StatsSection` is the metric strip beneath the hero section.
It displays four headline numbers in a responsive wrap so the page can quickly communicate
scale and activity. The section sits between the hero and the audience comparison to build
trust before the user reaches the deeper product explanations.

_StatItem
---------

.. code-block:: dart

   class _StatItem extends StatelessWidget {
     final String value;
     final String label;

     const _StatItem({required this.value, required this.label});

     @override
     Widget build(BuildContext context) {
       return SizedBox(
         width: 240,
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Text(value, style: const TextStyle(fontSize: 30, fontWeight: FontWeight.w800, color: AppColors.primary)),
             const SizedBox(height: 6),
             Text(label, style: const TextStyle(color: AppColors.muted, fontSize: 15)),
           ],
         ),
       );
     }
   }

`_StatItem` is a single statistic card used by `_StatsSection`.
It pairs a value with a label in a compact vertical layout so the metrics can be scanned
quickly. Multiple instances of this widget are combined to form the full stats row.

_AudienceSection
----------------

.. code-block:: dart

   class _AudienceSection extends StatelessWidget {
     const _AudienceSection();

     @override
     Widget build(BuildContext context) {
       return _SectionContainer(
         child: Column(
           children: [
             const Text('Built for both sides of the event experience', textAlign: TextAlign.center, style: AppTextStyles.sectionTitle),
             const SizedBox(height: 16),
             const SizedBox(
               width: 760,
               child: Text(
                 'UniSphere connects attendees looking for memorable local experiences with organisers who need simple, powerful tools to grow successful events.',
                 textAlign: TextAlign.center,
                 style: AppTextStyles.sectionSubtitle,
               ),
             ),
             const SizedBox(height: 48),
             LayoutBuilder(
               builder: (context, constraints) {
                 final isMobile = constraints.maxWidth < 900;
                 return isMobile ? const Column(children: [...]) : const Row(children: [...]);
               },
             ),
           ],
         ),
       );
     }
   }

`_AudienceSection` is the comparison section for the two main user groups.
It explains how UniSphere serves both attendees and organisers, then lays those experiences
out side by side or stacked depending on the available width. This section helps users see
that the product supports both event discovery and event creation.

_AudiencePanel
--------------

.. code-block:: dart

   class _AudiencePanel extends StatelessWidget {
     final String title;
     final String subtitle;
     final IconData icon;
     final List<String> items;

     const _AudiencePanel({required this.title, required this.subtitle, required this.icon, required this.items});
   }

`_AudiencePanel` is the card used for one audience group.
It displays a title, subtitle, icon, and checklist of features for either attendees or
organisers. The parent section reuses this widget twice so the page can compare the two
experiences in a consistent format.

_HowItWorksSection
------------------

.. code-block:: dart

   class _HowItWorksSection extends StatelessWidget {
     const _HowItWorksSection();

     @override
     Widget build(BuildContext context) {
       return _SectionContainer(
         color: Colors.white,
         child: Column(
           children: [
             const Text('How UniSphere works', textAlign: TextAlign.center, style: AppTextStyles.sectionTitle),
             const SizedBox(height: 14),
             const Text('A simple flow for discovering events or launching your own.', textAlign: TextAlign.center, style: AppTextStyles.sectionSubtitle),
             const SizedBox(height: 46),
             LayoutBuilder(
               builder: (context, constraints) {
                 final isMobile = constraints.maxWidth < 900;
                 return isMobile ? const Column(children: [...]) : const Row(children: [...]);
               },
             ),
           ],
         ),
       );
     }
   }

`_HowItWorksSection` is the workflow section for the landing page.
It explains the platform in three steps, moving from exploring events to choosing an
experience and then hosting one. This section translates the product into a simple process
that later widgets and navigation paths can support.

_StepCard
---------

.. code-block:: dart

   class _StepCard extends StatelessWidget {
     final String step;
     final String title;
     final String text;
     final IconData icon;

     const _StepCard({required this.step, required this.title, required this.text, required this.icon});
   }

`_StepCard` is one step in the how-it-works flow.
It combines the step number, icon, title, and explanatory text into a single card so the
process reads clearly in sequence. The parent section uses multiple copies of this widget to
present the full journey.

_CTASection
-----------

.. code-block:: dart

   class _CTASection extends StatelessWidget {
     const _CTASection();

     @override
     Widget build(BuildContext context) {
       return _SectionContainer(
         padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 90),
         child: Container(
           width: double.infinity,
           decoration: BoxDecoration(
             gradient: const LinearGradient(
               colors: [AppColors.primary, AppColors.primaryDark],
               begin: Alignment.topLeft,
               end: Alignment.bottomRight,
             ),
             borderRadius: BorderRadius.circular(28),
           ),
           child: LayoutBuilder(
             builder: (context, constraints) {
               final isMobile = constraints.maxWidth < 850;
               return isMobile ? Column(children: [...]) : Row(children: [...]);
             },
           ),
         ),
       );
     }
   }

`_CTASection` is the final call-to-action banner at the bottom of the page.
It gives users one last prompt to explore events or create an event, using the strongest
visual treatment in the landing page. The layout changes between mobile and desktop so the
buttons stay prominent while still fitting inside the gradient container.
