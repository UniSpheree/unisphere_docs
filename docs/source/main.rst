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

MyApp()
-------
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