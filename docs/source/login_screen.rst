login_screen.dart
=================

``login_screen.dart`` is the authentication screen that allows a user to sign in to
UniSphere. It provides the login form, validates the user input, and routes the user
to the logged-in dashboard when authentication succeeds.

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
   import '../widgets/role_toggle.dart';
   import '../utils/mock_backend.dart';

`material.dart` provides the Flutter framework and layout widgets used to build the
screen. The shared header and authentication widgets are imported to keep the login
page consistent with the rest of the app. The validator utilities and mock backend are
used to validate credentials and simulate the login process. The role toggle widget is
imported for the wider authentication flow, although the current screen no longer shows
role selection.

Main Login Widget
-----------------

.. code-block:: dart

   class LoginScreen extends StatefulWidget {
     const LoginScreen({super.key});

     @override
     State<LoginScreen> createState() => _LoginScreenState();
   }

The LoginScreen class is a StatefulWidget. It creates and returns an instance of the
_LoginScreenState class so the login form can manage text input, password visibility,
loading state, and authentication feedback.

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

Here we define the state of the LoginScreen class. This section covers the form key,
the text controllers, and the boolean values that control password visibility, login
persistence, and loading feedback.

The _formKey attribute is used to validate the form before login begins. The
_emailController and _passwordController attributes store the user's input, while the
_obscurePassword, _keepLoggedIn, and _isLoading attributes track the screen's current
interaction state.

Dispose Method
--------------

.. code-block:: dart

   @override
   void dispose() {
     _emailController.dispose();
     _passwordController.dispose();
     super.dispose();
   }

The dispose method releases the email and password controllers when the screen is
removed from the widget tree. This prevents memory leaks and keeps the login screen's
state lifecycle clean.

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

The _handleLogin method handles the login action for the screen. It takes no input
parameters and returns a Future<void> because the login flow runs asynchronously. The
method validates the form, sends the credentials to the mock backend, and updates the
loading state while the request is running. If the login succeeds, it shows a success
SnackBar and navigates to the logged-in route while clearing the previous navigation
stack. If the login fails, it displays an error message using a SnackBar.

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

The build method is the main widget builder for the LoginScreen state. It returns a
Scaffold containing the shared app header and a centered login card. The form is wrapped
in a scroll view and constrained width so the layout stays readable on smaller screens.

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

The shared app header sits at the top of the screen and gives the user quick access to
discover, create, and register actions. On this page, the sign-in action is inactive
because the user is already on the login screen.

Login Card
^^^^^^^^^^

The login form is wrapped in a centered card with rounded corners and a shadow. This
keeps the authentication area visually separated from the page background and focuses
attention on the form.

Logo
^^^^

.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.maybePop(context),
     child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
   )

The logo section is a tappable image that allows the user to return to the previous page
if possible. It also acts as the visual identity for the login form.

Title and Intro Text
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   const Text('Welcome to UniSphere'),
   const Text('Please select your role and enter your credentials.'),

The title and supporting text introduce the login form and explain what the user needs
to do next.

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

The email field captures the user's university email address. It uses the shared
authentication text field widget and validates the input with the university email
validator before login continues.

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

The password field stores the user's password and hides the text while _obscurePassword
is true. It uses the shared password validator to check the password before submission.

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

This button switches the password field between hidden and visible text. It updates the
_obscurePassword state so the password icon and the field behavior stay in sync.

Forgot Password Link
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   TextButton(
     onPressed: () => Navigator.pushNamed(context, '/forgot-password'),
     child: const Text('Forgot Password?'),
   )

The forgot password link routes the user to the password recovery page when they need
to reset their login details.

Keep Me Logged In
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   Checkbox(
     value: _keepLoggedIn,
     onChanged: (v) => setState(() => _keepLoggedIn = v ?? false),
   )

This checkbox stores whether the user wants to stay logged in. The value is saved in
_keepLoggedIn so the screen can track the user's preference.

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

The login button triggers the authentication flow. When _isLoading is true, it is
disabled and replaced with a progress indicator while the login request is running.

Register Link
^^^^^^^^^^^^^

.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.pushNamed(context, '/register'),
     child: const Text('Register here'),
   )

The register link gives new users a direct route to the registration screen if they do
not already have an account.
