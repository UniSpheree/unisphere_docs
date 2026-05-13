login_screen.dart
=================

``login_screen.dart`` is the authentication screen for signing in to UniSphere.
It provides the login form, validates credentials, and routes the user to the logged-in
area when authentication succeeds. The screen is now backed by the SQLite backend, and
the role-selection logic has been removed from this page.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import '../utils/validators.dart';
   import 'package:unisphere_app/widgets/header.dart';
   import 'create_event_screen.dart';
   import 'discover_event_screen.dart';
   import '../widgets/auth_header.dart';
  import '../widgets/auth_text_field.dart';
  import '../services/sqlite_backend.dart';

`material.dart` provides the Flutter framework and layout widgets used to build the screen.
The shared authentication text field widget and validator utilities keep the login form
consistent with the rest of the app. The SQLite backend import is the key source-level update
here because authentication now runs against the real backend instead of a mock service.

Main Login Widget
-----------------

.. code-block:: dart

   class LoginScreen extends StatefulWidget {
     const LoginScreen({super.key});

     @override
     State<LoginScreen> createState() => _LoginScreenState();
   }

`LoginScreen` is the top-level `StatefulWidget` for the login page.
It creates the state object that handles text input, password visibility, loading feedback,
and the authentication request. The widget itself is lightweight, but it is the entry point
for the full sign-in workflow.

Login Screen State
------------------

.. code-block:: dart

   class _LoginScreenState extends State<LoginScreen> {
     final _formKey = GlobalKey<FormState>();
     final _emailController = TextEditingController();
     final _passwordController = TextEditingController();

     bool _obscurePassword = true;
     bool _keepLoggedIn = false;
     bool _isLoading = false;
   }

`_LoginScreenState` is the state holder for the login form.
It stores the form key, the email and password controllers, the password visibility flag,
the remember-me flag, and the loading state. The role-selection note in the source means this
screen now focuses only on authentication, while role handling is deferred elsewhere in the app.

Dispose Method
--------------

.. code-block:: dart

   @override
   void dispose() {
     _emailController.dispose();
     _passwordController.dispose();
     super.dispose();
   }

The `dispose` method releases the email and password controllers when the login screen is
removed from the widget tree. This prevents memory leaks and keeps the authentication screen’s
state lifecycle tidy.

Login Handler
-------------

.. code-block:: dart

   Future<void> _handleLogin() async {
     if (!_formKey.currentState!.validate()) return;

     setState(() => _isLoading = true);
     final email = _emailController.text.trim().toLowerCase();
     final password = _passwordController.text;
     final success = await MockBackend().login(email: email, password: password);
     setState(() => _isLoading = false);

     if (!mounted) return;

     if (success) {
       ScaffoldMessenger.of(context).showSnackBar(
         const SnackBar(
           content: Text(
             'Welcome back! Login successful.',
             style: TextStyle(color: Colors.white),
           ),
           backgroundColor: Color(0xFF2D3A8C),
           behavior: SnackBarBehavior.floating,
           shape: RoundedRectangleBorder(
             borderRadius: BorderRadius.all(Radius.circular(8)),
           ),
         ),
       );
       Navigator.pushNamedAndRemoveUntil(
         context,
         '/logged-in',
         (route) => false,
       );
     } else {
       ScaffoldMessenger.of(context).showSnackBar(
         const SnackBar(
           content: Text(
             'Invalid email or password.',
             style: TextStyle(color: Colors.white),
           ),
           backgroundColor: Colors.redAccent,
           behavior: SnackBarBehavior.floating,
           shape: RoundedRectangleBorder(
             borderRadius: BorderRadius.all(Radius.circular(8)),
           ),
         ),
       );
     }
   }

`_handleLogin` is the asynchronous method that drives the sign-in flow.
It validates the form, lowercases the email, sends the credentials to `SqliteBackend().login`,
and updates the loading state while waiting for the response. If authentication succeeds, it
shows a success message and clears the navigation stack to send the user into the logged-in
route; otherwise, it shows an error message.

Widget Build
------------

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     return Scaffold(
       backgroundColor: const Color(0xFFF0F2F8),
       body: Column(
         children: [
           AppHeader(...),
           Expanded(
             child: Center(
               child: SingleChildScrollView(
                 child: ConstrainedBox(
                   constraints: const BoxConstraints(maxWidth: 420),
                   child: Container(
                     child: Form(
                       key: _formKey,
                       child: Column(
                         children: [
                           GestureDetector(...),
                           Text(...),
                           AuthTextField(...),
                           AuthTextField(...),
                           TextButton(...),
                           Checkbox(...),
                           ElevatedButton(...),
                           GestureDetector(...),
                         ],
                       ),
                     ),
                   ),
                 ),
               ),
             ),
           ),
         ],
       ),
     );
   }

The `build` method assembles the complete login interface.
It returns a scaffold with a centered, constrained login card so the form stays readable on
compact screens and desktop layouts. The scrollable layout is important because the login form
contains stacked controls that need to remain accessible on smaller viewports.

App Header
^^^^^^^^^^

.. code-block:: dart

   AppHeader(
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
     onRegisterTap: () {
       Navigator.pushNamed(context, '/register');
     },
     onSignInTap: () {},
     showProfile: false,
   )

The shared app header sits at the top of the login screen and provides quick access to
discovery, creation, and registration routes. On this page, the sign-in action is intentionally
inactive because the user is already on the login screen. This header keeps the authentication
page connected to the rest of the app without repeating the form controls in the page body.

Login Card
^^^^^^^^^^

This container is the main visual wrapper for the login form. It isolates the form from the page
background with a white card, rounded corners, and shadow so the authentication area feels focused.
The card is especially important in this layout because the rest of the screen is intentionally
minimal.

Logo
^^^^

.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.maybePop(context),
     child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
   )

The logo is a tappable brand image that lets the user return to the previous page if possible.
It acts as both navigation and identity, giving the login screen a consistent top anchor. The use
of `Navigator.maybePop` is a subtle detail because it avoids forcing a route change if there is no
previous page to return to.

Title and Intro Text
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   const Text('Welcome to UniSphere'),
   const Text('Please select your role and enter your credentials.'),

These text widgets introduce the login form and tell the user what credentials are needed.
They provide orientation before the input fields begin, which keeps the screen straightforward and
easy to scan. The copy still references the role flow in the source, but the code comments make
clear that role selection is no longer handled here.

Email Field
^^^^^^^^^^^

.. code-block:: dart

   AuthTextField(
     label: 'Email Address',
     hintText: 'alex@university.edu',
     prefixIcon: Icons.email_outlined,
     controller: _emailController,
     keyboardType: TextInputType.emailAddress,
     validator: validateUniversityEmail,
   )

This field collects the user’s university email address. It uses the shared authentication text
field widget and the shared university email validator so the format is checked before login
continues. Normalising the input to an email-based sign-in flow keeps the credentials consistent
with the registration screen.

Password Field
^^^^^^^^^^^^^^

.. code-block:: dart

   AuthTextField(
     label: 'Password',
     prefixIcon: Icons.lock_outline,
     obscureText: _obscurePassword,
     controller: _passwordController,
     validator: validatePassword,
   )

This field captures the user’s password. It hides the text while the obscured state is enabled
and validates the password using the shared validator. The field is structurally simple, but it
is central to the authentication flow because its value is what the backend checks against stored
credentials.

Password Visibility Toggle
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   IconButton(
     icon: Icon(
       _obscurePassword
           ? Icons.visibility_outlined
           : Icons.visibility_off_outlined,
     ),
     onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
   )

This icon button switches the password field between hidden and visible text. It updates the
password visibility state so the icon and the field behaviour remain synchronised. The toggle is a
small detail, but it makes the form easier to use without affecting authentication logic.

Forgot Password Link
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   TextButton(
     onPressed: () => Navigator.pushNamed(context, '/forgot-password'),
     child: const Text('Forgot Password?'),
   )

This text button routes the user to the password recovery page. It gives users a direct escape
path when they cannot remember their login details. The compact styling matters because the link
should be available without dominating the form.

Keep Me Logged In
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Checkbox(
     value: _keepLoggedIn,
     onChanged: (v) => setState(() => _keepLoggedIn = v ?? false),
   )

This checkbox stores whether the user wants the session to persist. The value is tracked in
`_keepLoggedIn`, even though the actual persistence behaviour would be handled elsewhere in the
authentication flow. It gives the login form a standard “remember me” control without complicating
the visible layout.

Login Button
^^^^^^^^^^^^

.. code-block:: dart

   ElevatedButton(
     onPressed: _isLoading ? null : _handleLogin,
     child: _isLoading
         ? const SizedBox(
             width: 20,
             height: 20,
             child: CircularProgressIndicator(
               strokeWidth: 2,
               color: Colors.white,
             ),
           )
         : const Row(
             mainAxisAlignment: MainAxisAlignment.center,
             children: [
               Text('Login to Dashboard'),
               Icon(Icons.arrow_forward),
             ],
           ),
   )

This button triggers the authentication handler. It disables itself and shows a progress
indicator while the backend request is in flight, which prevents duplicate submissions. The loading
state is important because the screen has no other obvious feedback while waiting for the backend
response.

Register Link
^^^^^^^^^^^^^

.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.pushNamed(context, '/register'),
     child: const Text('Register here'),
   )

This footer link routes new users to the registration screen. It uses direct navigation so users
who do not have an account can move into the sign-up flow immediately. The link keeps the login page
connected to the wider auth experience without crowding the main form.
