=========
main.dart
=========

``main.dart`` serves as the central entry point for the UniSphere application. 
It handles the crucial initialisation of the backend database before rendering 
the root widget, configures the global Material Design theme, and defines the 
complete routing table for navigation between screens.


Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'screens/create_event_screen.dart';
   import 'screens/profile_page.dart';
   import 'screens/landing_page.dart';
   import 'screens/landing_page_logged_in.dart';
   import 'screens/forgot_password_screen.dart';
   import 'screens/login_screen.dart';
   import 'screens/register_screen.dart';
   import 'screens/discover_event_screen.dart';
   import 'screens/about_us_page.dart';
   import 'screens/terms_page.dart';
   import 'screens/privacy_page.dart';
   import 'screens/my_tickets_screen.dart';
   import 'screens/my_events_page.dart';
   import 'screens/calendar_page.dart';
   import 'services/sqlite_backend.dart';

This block imports the core Flutter Material package alongside every screen 
used within the application's routing map. It also imports the primary backend 
service to ensure data is loaded before the UI attempts to render.


Initialization (main)
---------------------

.. code-block:: dart

   void main() async {
     WidgetsFlutterBinding.ensureInitialized();

     // Initialize backend
     final backend = SqliteBackend();
     await backend.initializeDatabase();

     runApp(const MyApp());
   }

The asynchronous ``main`` function is the first code executed when the app 
launches. ``WidgetsFlutterBinding.ensureInitialized()`` is required because the 
app performs asynchronous operations before calling ``runApp()``. It instantiates 
the backend singleton and pauses execution to ``await backend.initializeDatabase()``, 
ensuring user sessions, events, and network health are verified before the first 
pixel is drawn on the screen.


MyApp Configuration & Routing
-----------------------------

.. code-block:: dart

  class MyApp extends StatelessWidget {
    const MyApp({super.key});

    @override
    Widget build(BuildContext context) {
      return MaterialApp(
        debugShowCheckedModeBanner: false,
        title: 'UniSphere',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF2D3A8C)),
          useMaterial3: true,
          scaffoldBackgroundColor: const Color(0xFFF0F2F8),
          fontFamily: 'Roboto',
        ),
        // ── Initial route (landing page as the app entry point)
        initialRoute: '/',
        routes: {
          '/': (_) => const LandingPage(),
          '/logged-in': (_) {
            final user = MockBackend().currentUser;
            return PersonalizedLandingPage(
              userName: user?.fullName ?? 'Guest',
              role: user?.role ?? 'Attendee',
            );
          },
          '/login': (_) => const LoginScreen(),
          '/register': (_) => const RegisterScreen(),
          '/forgot-password': (_) => const ForgotPasswordScreen(),
          '/profile': (_) => const ProfilePage(),
          '/create-event': (_) => const CreateEventScreen(),
        },
        onGenerateRoute: (settings) {
          if (settings.name == '/dashboard') {
            final role = (settings.arguments as String?) ?? 'Attendee';
            return MaterialPageRoute(
              settings: settings,
              builder: (_) => DashboardScreen(role: role),
            );
          }
          return null;
        },
      );
    }
  }

MyApp creates the MaterialApp configuration, defining the app title (UniSphere), 
global theme with seed color 0xFF2D3A8C and Material3 design. Navigation is 
defined here with seven named routes and dynamic route generation for dashboard 
pages that require a role parameter.
   class MyApp extends StatelessWidget {
     const MyApp({super.key});

     @override
     Widget build(BuildContext context) {
       final startRoute = SqliteBackend().currentUser != null ? '/logged-in' : '/';

       return MaterialApp(
         debugShowCheckedModeBanner: false,
         title: 'UniSphere',
         theme: ThemeData(
           colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF2D3A8C)),
           useMaterial3: true,
           scaffoldBackgroundColor: const Color(0xFFF0F2F8),
           fontFamily: 'Roboto',
         ),
         // ── Initial route
         initialRoute: startRoute,
         routes: {
           '/': (_) => const LandingPage(),
           '/logged-in': (_) {
             final user = SqliteBackend().currentUser;
             return PersonalizedLandingPage(
               userName: user?.fullName ?? 'Guest',
               role: user?.role ?? 'Attendee',
             );
           },
           // ... static route definitions for Login, Profile, Create Event, etc ...
         },
       );
     }
   }

The ``MyApp`` root widget configures the overarching application container via 
``MaterialApp``. 

It defines the global ``ThemeData``, enforcing the primary seed color 
(``#2D3A8C``), enabling Material 3 design paradigms, setting a standardized 
background color, and declaring `Roboto` as the default font family.

Crucially, it implements dynamic initial routing. By checking 
``SqliteBackend().currentUser`` during the build process, it intelligently 
assigns ``startRoute``. If a valid user session is found in memory, the app 
immediately drops them onto the ``/logged-in`` dashboard; otherwise, it presents 
the public ``/`` landing page, seamlessly handling persistent authentication.
