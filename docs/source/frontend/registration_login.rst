=====================================================================
Registration & log in pages - register_screen.dart, login_screen.dart, forgot_password_screen.dart
=====================================================================
These sections will cover the implentation of unisphere's log in, registration and forgot password features. 
These screens allow for users to log in, register and change their password if they have forgotten it. 

register_screen.dart
--------------------

``register_screen.dart`` is the account registration screen for new UniSphere users.
It gathers identity details, validates the form, checks that the user agrees to the terms,
and creates the account through the backend before routing the user into the logged-in
experience.

Imports
^^^^^^^

.. code-block:: dart

   import 'package:flutter/material.dart';
   import 'package:flutter/gestures.dart';
   import '../utils/validators.dart';
   import '../widgets/auth_text_field.dart';
   import '../utils/unis.dart';
   import '../services/sqlite_backend.dart';

``material.dart`` provides the Flutter UI framework for the registration form.
The validator, auth text field, university list, and backend utilities support field
validation, dropdown data, and account creation. The backend import is important because
this screen now uses the SQLite-backed registration flow instead of the mock backend.

Main Register Widget
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    class RegisterScreen extends StatefulWidget {
       const RegisterScreen({super.key});

       @override
       State<RegisterScreen> createState() => _RegisterScreenState();
    }

`RegisterScreen` is the top-level `StatefulWidget` for the registration page.
It creates the state object so the form can manage controllers, password visibility,
consent handling, and async submission feedback. The widget itself stays lightweight,
but it is the entry point for the full sign-up experience.

Register Screen State
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    class _RegisterScreenState extends State<RegisterScreen> {
    final _formKey = GlobalKey<FormState>();
    final _firstNameController = TextEditingController();
    final _lastNameController = TextEditingController();
    final _emailController = TextEditingController();
    final TextEditingController _universityFieldController =
       TextEditingController();
    final TextEditingController _universitySearchController =
       TextEditingController();
    final _passwordController = TextEditingController();
    final _confirmPasswordController = TextEditingController();

    bool _obscurePassword = true;
    bool _obscureConfirm = true;
    bool _agreeToTerms = false;
    bool _isLoading = false;
    }

`_RegisterScreenState` is the state holder for the registration workflow.
It stores the form key, all text controllers, the password visibility flags, the consent
checkbox state, and the loading state used during submission. The university field
controllers are especially important because they keep the autocomplete input and the
stored institution value aligned while the user types or selects a suggestion.

Dispose Method
^^^^^^^^^^^^^^

.. code-block:: dart

    @override
    void dispose() {
       _universitySearchController.dispose();
       _universityFieldController.dispose();
       _firstNameController.dispose();
       _lastNameController.dispose();
       _emailController.dispose();
       _passwordController.dispose();
       _confirmPasswordController.dispose();
       super.dispose();
    }

The `dispose` method releases all text controllers when the registration screen is removed
from the widget tree. This prevents controller leaks and keeps the state lifecycle clean for
a form that uses several text inputs and autocomplete widgets.

Registration Handler
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Future<void> _handleRegister() async {
       if (!_formKey.currentState!.validate()) return;
       if (!_agreeToTerms) {
          ScaffoldMessenger.of(context).showSnackBar(...);
          return;
       }

       setState(() => _isLoading = true);
       final firstName = _firstNameController.text.trim();
       final lastName = _lastNameController.text.trim();
       final email = _emailController.text.trim().toLowerCase();
       final password = _passwordController.text;
       final university = _universityFieldController.text.trim();
       final success = await MockBackend().register(...);
       setState(() => _isLoading = false);

       if (!mounted) return;

       if (success) {
          ScaffoldMessenger.of(context).showSnackBar(...);
          Navigator.pushNamedAndRemoveUntil(context, '/logged-in', (r) => false);
       } else {
          ScaffoldMessenger.of(context).showSnackBar(...);
       }
    }

`_handleRegister` is the asynchronous method that drives account creation.
It validates the form, enforces the terms agreement, normalises the submitted values, and
sends the registration payload to `SqliteBackend().register`. The method also controls the
loading state and the feedback shown to the user after the backend responds.

If registration succeeds, it shows a success SnackBar and clears the navigation stack by
routing to `/logged-in`. If registration fails, it shows an error SnackBar indicating an
existing account.


.. code-block:: dart

    if (!_agreeToTerms) {
       ScaffoldMessenger.of(context).showSnackBar(...);
       return;
    }

This is the terms guard that blocks registration until the user has agreed to the Terms of Service and Privacy
Policy. It is a critical validation step because the account should not be created if legal
consent has not been given.

Backend Registration Call
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    final success = await MockBackend().register(
       email: email,
       password: password,
       firstName: firstName,
       lastName: lastName,
       role: 'Attendee',
       university: university,
       isApproved: true,
    );

This backend call submits the cleaned registration data to the SQLite backend. It sends the
email, password, first name, last name, role, university, and approval status in one place so
the backend can create the account. The call returns a boolean success result, which determines
whether the screen shows a success message and redirects the user or displays an error about an
existing account.

Widget Build
^^^^^^^^^^^^^

.. code-block:: dart

    @override
    Widget build(BuildContext context) {
       return Scaffold(
          backgroundColor: const Color(0xFFF0F2F8),
          body: Center(
             child: SingleChildScrollView(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 32),
                child: ConstrainedBox(
                   constraints: const BoxConstraints(maxWidth: 480),
                   child: Container(
                      padding: const EdgeInsets.all(36),
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(16),
                        boxShadow: [
                          BoxShadow(
                            color: Colors.black.withOpacity(0.07),
                            blurRadius: 24,
                            offset: const Offset(0, 6),
                          ),
                        ],
                      ),
                      child: Form(
                        key: _formKey,
                        child: Column(
                          mainAxisSize: MainAxisSize.min,
                          crossAxisAlignment: CrossAxisAlignment.stretch,
                          children: [
                            GestureDetector(...),
                            Text(...),
                            Row(...),
                            AuthTextField(...),
                            Autocomplete<String>(...),
                            AuthTextField(...),
                            AuthTextField(...),
                            Row(...),
                            ElevatedButton(...),
                            Row(...),
                          ],
                        ),
                      ),
                   ),
                ),
             ),
          ),
       );
    }

The `build` method constructs the full registration form UI. It returns a centered scrollable
card so the page remains readable on both smaller screens and desktop widths. The layout is
intentionally wrapped in a constrained container because the form contains many stacked inputs
and needs a stable, narrow reading column.



Registration Card Layout
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Container(
       padding: const EdgeInsets.all(36),
       decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(16),
          boxShadow: [
             BoxShadow(
                color: Colors.black.withOpacity(0.07),
                blurRadius: 24,
                offset: const Offset(0, 6),
             ),
          ],
       ),
       child: Form(
          key: _formKey,
          child: Column(
             mainAxisSize: MainAxisSize.min,
             crossAxisAlignment: CrossAxisAlignment.stretch,
             children: [...],
          ),
       ),
    )

This container is the main visual wrapper for the registration form. It separates the form
from the page background with rounded corners and shadow, which keeps the focus on the account
creation flow. Its role is structural rather than functional, but it provides the visual
framing that makes the authentication screen feel complete.


.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushNamed(context, '/'),
       child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
    )

The logo is a tappable brand element that routes the user back to the public landing page.
It gives the registration screen a clear escape route and keeps the app identity visible at the
top of the form. The gesture wrapper matters because the image itself acts as navigation rather
than static decoration.



.. code-block:: dart

    const Text('Create your Account')
    const Text('Join UniSphere and connect with your university community.')

These two text widgets are the title and intro text that introduce the registration flow and explain the purpose
of the screen.

Authentication fields
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Row(
       children: [
          Expanded(
             child: AuthTextField(
                label: 'First Name',
                controller: _firstNameController,
                validator: (v) => (v == null || v.trim().isEmpty) ? 'Required' : null,
             ),
          ),
          Expanded(
             child: AuthTextField(
                label: 'Last Name',
                controller: _lastNameController,
                validator: (v) => (v == null || v.trim().isEmpty) ? 'Required' : null,
             ),
          ),
       ],
    )

The first and last name fields that collect the user’s first and last name side by side. They reduce vertical space
while capturing identity details early in the form, which matches the rest of the compact
registration layout. Each field uses a simple required validator so the form can reject
incomplete submissions immediately.


.. code-block:: dart

    AuthTextField(
       label: 'University Email Address',
       controller: _emailController,
       keyboardType: TextInputType.emailAddress,
       validator: validateUniversityEmail,
    )

The University email field that collects the user’s university email address. It uses the shared university email
validator to make sure the address matches the expected institutional format before registration
can continue. The email is later normalised to lowercase in the submission handler, so the stored
value is consistent.


.. code-block:: dart

    Autocomplete<String>(
       optionsBuilder: (TextEditingValue textEditingValue) { ... },
       fieldViewBuilder: (context, controller, focusNode, onFieldSubmitted) { ... },
       onSelected: (String selection) { ... },
    )

The autocomplete field that lets the user search for and select their institution from the known
university list. It supports both typing and suggestion filtering, which makes it easier to find a
valid institution while still keeping the input constrained to supported values. The selected
value is mirrored into the visible form controller, so the field and the stored institution stay
in sync.



.. code-block:: dart

    validator: (v) {
       if (v == null || v.isEmpty) {
          return 'Institution is required';
       }
       if (!ukUniversities.contains(v)) {
          return 'Institution does not exist.';
       }
       ...
       if (lowerV.contains(uniFromDomain)) {
          return 'You cannot enter your own university.';
       }
       return null;
    }

This is the validator that checks that an institution was provided, that the value exists in the supported
university list, and that the institution does not match the user’s own email-domain university.
It protects the registration flow from invalid or contradictory account data. The domain
comparison is an important subtlety because it prevents users from registering with their own
university as a disallowed case in this flow.


.. code-block:: dart

    AuthTextField(
       label: 'Password',
       obscureText: _obscurePassword,
       controller: _passwordController,
       validator: validatePassword,
    )

The password field that captures the inputted account password. It uses the shared password validator and hides the
input while the obscured state is enabled. The field is part of the shared authentication pattern,
so it behaves consistently with the login screen.


.. code-block:: dart

    IconButton(
       icon: Icon(
          _obscurePassword
                ? Icons.visibility_outlined
                : Icons.visibility_off_outlined,
       ),
       onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
    )

This password visibility toggle button that toggles the password field between hidden and visible text. It updates the
password visibility state so the icon and field behaviour stay aligned. The toggle matters because
it gives users control over checking the password they entered without leaving the form.


.. code-block:: dart

    AuthTextField(
       label: 'Confirm Password',
       obscureText: _obscureConfirm,
       controller: _confirmPasswordController,
       validator: (v) {
          final password = _passwordController.text;
          if (v == null || v.isEmpty) {
             return 'Please confirm your password';
          }
          if (v != password) {
             return 'Passwords do not match';
          }
          return null;
       },
    )

Password confirmation field where the user must repeat the password entry to confirm their password before submitting the
form. It validates that the confirmation is not empty and that it matches the primary password
field. This extra check reduces accidental mismatches before the backend request is made.



.. code-block:: dart

    IconButton(
       icon: Icon(
          _obscureConfirm
                ? Icons.visibility_outlined
                : Icons.visibility_off_outlined,
       ),
       onPressed: () => setState(() => _obscureConfirm = !_obscureConfirm),
    )

This icon button toggles visibility for the confirmation field independently of the main
password field. It keeps the confirmation workflow consistent with the main password input while
still allowing separate control. The independence matters because users may want to inspect the
confirmation entry without changing the original password visibility state.

Terms and Privacy Consent
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

      Row(
          children: [
               Checkbox(
                   value: _agreeToTerms,
                   onChanged: (v) => setState(() => _agreeToTerms = v ?? false),
               ),
               Expanded(
                  child: RichText(
                     text: TextSpan(
                        style: const TextStyle(fontSize: 12, color: Colors.grey),
                        children: [
                           const TextSpan(text: 'I agree to the '),
                           TextSpan(
                              text: 'Terms of Service',
                              style: const TextStyle(
                                 color: Color(0xFF2D3A8C),
                                 fontWeight: FontWeight.w600,
                                 decoration: TextDecoration.underline,
                              ),
                              recognizer: TapGestureRecognizer()
                                 ..onTap = () {
                                    Navigator.pushNamed(context, '/terms');
                                 },
                           ),
                           const TextSpan(text: ' and '),
                           TextSpan(
                              text: 'Privacy Policy',
                              style: const TextStyle(
                                 color: Color(0xFF2D3A8C),
                                 fontWeight: FontWeight.w600,
                                 decoration: TextDecoration.underline,
                              ),
                              recognizer: TapGestureRecognizer()
                                 ..onTap = () {
                                    Navigator.pushNamed(context, '/privacy');
                                 },
                           ),
                        ],
                     ),
                  ),
               ),
          ],
      )

This consent block stores whether the user agrees to the legal terms before registration proceeds.
It combines a checkbox with tappable legal links so the user can review the Terms of Service and
Privacy Policy directly from the form. The legal links matter because the consent state is not
just visual; it gates the actual account creation flow.

Authentication links
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    ElevatedButton(
       onPressed: _isLoading ? null : _handleRegister,
       child: _isLoading
             ? const CircularProgressIndicator(...)
             : const Row(
                   children: [Text('Create Account'), Icon(Icons.arrow_forward)],
                ),
    )

This is the create account button that triggers the registration handler. It disables itself and shows a loading indicator
while the backend request is running so users do not submit the form twice. The loading state is
important because registration is asynchronous and the UI needs to reflect that the app is waiting
for a backend response.


.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushReplacementNamed(context, '/login'),
       child: const Text('Sign In'),
    )

This footer link is the sign in link that routes existing users to the login screen. It uses replacement navigation so the
registration page does not remain in the back stack after the user moves to sign in. That
navigation choice keeps the auth flow clean and avoids sending the user back to an unnecessary
form.


login_screen.dart
------------------

``login_screen.dart`` is the authentication screen for signing in to UniSphere.
It provides the login form, validates credentials, and routes the user to the logged-in
area when authentication succeeds. The screen is now backed by the SQLite backend, and
the role-selection logic has been removed from this page.

Imports
^^^^^^^

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

Login Screen widget
^^^^^^^^^^^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^^^^^^

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

State methods
^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^

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

Layout section
^^^^^^^^^^^^^^^

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

.. code-block:: dart

   Container(
     decoration: BoxDecoration(
       color: Colors.white,
       borderRadius: BorderRadius.circular(16),
       boxShadow: [
         BoxShadow(
           color: Colors.black.withOpacity(0.07),
           blurRadius: 24,
           offset: const Offset(0, 6),
         ),
       ],
     ),
   )

This container is the main visual wrapper for the login form. It isolates the form from the page
background with a white card, rounded corners, and shadow so the authentication area feels focused.
The card is especially important in this layout because the rest of the screen is intentionally
minimal.


.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.maybePop(context),
     child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
   )

The Unisphere logo is used tappable brand image that lets the user return to the previous page if possible.
It acts as both navigation and identity, giving the login screen a consistent top anchor. The use
of `Navigator.maybePop` is a subtle detail because it avoids forcing a route change if there is no
previous page to return to.

.. code-block:: dart

   const Text('Welcome to UniSphere'),
   const Text('Please select your role and enter your credentials.'),

These text widgets introduce the login form and tell the user what credentials are needed.
They provide orientation before the input fields begin, which keeps the screen straightforward and
easy to scan. The copy still references the role flow in the source, but the code comments make
clear that role selection is no longer handled here.

Authentication form
^^^^^^^^^^^^^^^^^^^
Contains all input fields and controls required for user login.

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


.. code-block:: dart

   Checkbox(
     value: _keepLoggedIn,
     onChanged: (v) => setState(() => _keepLoggedIn = v ?? false),
   )

This checkbox stores whether the user wants the session to persist. The value is tracked in
`_keepLoggedIn`, even though the actual persistence behaviour would be handled elsewhere in the
authentication flow. It gives the login form a standard “remember me” control without complicating
the visible layout.

Authentication Links
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   TextButton(
     onPressed: () => Navigator.pushNamed(context, '/forgot-password'),
     child: const Text('Forgot Password?'),
   )

This text button includes the link that routes the user to the password recovery page. It gives users a direct escape
path when they cannot remember their login details. The compact styling matters because the link
should be available without dominating the form.



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


.. code-block:: dart

   GestureDetector(
     onTap: () => Navigator.pushNamed(context, '/register'),
     child: const Text('Register here'),
   )

This footer link reroutes new users to the registration screen. It uses direct navigation so users
who do not have an account can move into the sign-up flow immediately. The link keeps the login page
connected to the wider auth experience without crowding the main form.


forgot_password_screen.dart
---------------------------

``forgot_password_screen.dart`` handles the complete password reset workflow.
Rather than routing the user across multiple separate pages, it operates as a 
seamless, 3-step "Wizard" form controlled by local state. It manages email 
verification, mock code validation, and secure password updates before routing 
the user back to the login screen.

Imports
^^^^^^^

.. code-block:: dart

   import 'package:flutter/material.dart';
   import '../utils/validators.dart';
   import '../widgets/auth_header.dart';
   import '../widgets/auth_text_field.dart';
   import '../services/sqlite_backend.dart';

This block imports standard Flutter material UI, custom form field widgets, generic 
input validators, and the SQLite backend service required to query and update user 
credentials.

Controller State
^^^^^^^^^^^^^^^^

State Initialisation

.. code-block:: dart

   class ForgotPasswordScreen extends StatefulWidget {
     const ForgotPasswordScreen({super.key});

     @override
     State<ForgotPasswordScreen> createState() => _ForgotPasswordScreenState();
   }

   class _ForgotPasswordScreenState extends State<ForgotPasswordScreen> {
     // Shared state
     int _step = 1; // 1 | 2 | 3
     bool _isLoading = false;

     // Step 1
     final _emailFormKey = GlobalKey<FormState>();
     final _emailController = TextEditingController();

     // Step 2
     final _codeFormKey = GlobalKey<FormState>();
     final _codeController = TextEditingController();
     static const _fakeCode = '123456';

     // Step 3
     final _pwFormKey = GlobalKey<FormState>();
     final _newPasswordController = TextEditingController();
     final _confirmPasswordController = TextEditingController();
     bool _obscureNew = true;
     bool _obscureConfirm = true;

     @override
     void dispose() {
       _emailController.dispose();
       _codeController.dispose();
       _newPasswordController.dispose();
       _confirmPasswordController.dispose();
       super.dispose();
     }
   }

The state class tracks the active wizard page via the ``_step`` integer and manages 
a global ``_isLoading`` boolean to prevent duplicate network calls. Crucially, it 
isolates each step by assigning them distinct ``GlobalKey<FormState>`` and 
``TextEditingController`` instances. This isolation ensures that validation errors 
on Step 1 do not accidentally trigger or interfere with inputs on Step 3. All 
controllers are cleanly destroyed in the ``dispose`` method to prevent memory leaks.


Authentication Handlers
^^^^^^^^^^^^^^^^^^^^^^^

These methods encapsulate the asynchronous business logic that validates user 
inputs and dictates the progression of the wizard steps.


.. code-block:: dart

   Future<void> _submitEmail() async {
     if (!_emailFormKey.currentState!.validate()) return;
     setState(() => _isLoading = true);
     final email = _emailController.text.trim().toLowerCase();
     final exists = await SqliteBackend().forgotPassword(email);
     setState(() => _isLoading = false);
     if (!mounted) return;
     
     if (exists) {
       setState(() => _step = 2);
     } else {
       ScaffoldMessenger.of(context).showSnackBar(
         SnackBar(
           content: const Text('No account found with this email.', style: TextStyle(color: Colors.white)),
           backgroundColor: Colors.redAccent,
           // ... styling ...
         ),
       );
     }
   }

The _submitEmail function drives Step 1. It validates the email input against the standard 
university regex and queries the ``SqliteBackend`` to check if an active account 
matches the address. If an account is found, it progresses the wizard to Step 2; 
otherwise, it surfaces a prominent red error ``SnackBar`` to inform the user that 
the account does not exist.


.. code-block:: dart

   Future<void> _submitCode() async {
     if (!_codeFormKey.currentState!.validate()) return;
     setState(() => _isLoading = true);
     await Future.delayed(const Duration(milliseconds: 700));
     setState(() => _isLoading = false);

     if (!mounted) return;
     if (_codeController.text.trim() != _fakeCode) {
       ScaffoldMessenger.of(context).showSnackBar(
         SnackBar(
           content: const Text('Invalid code. Please try again.', style: TextStyle(color: Colors.white)),
           backgroundColor: Colors.redAccent,
           // ... styling ...
         ),
       );
       return;
     }
     setState(() => _step = 3);
   }

The _submitCode function drives Step 2. It simulates network latency with a brief delay and 
compares the user's input against a hardcoded mock value (``123456``). If the code 
matches, the wizard progresses to the final step.


.. code-block:: dart

   Future<void> _submitNewPassword() async {
     if (!_pwFormKey.currentState!.validate()) return;
     setState(() => _isLoading = true);
     final email = _emailController.text.trim().toLowerCase();
     final newPassword = _newPasswordController.text;
     final success = await SqliteBackend().resetPassword(email, newPassword);
     setState(() => _isLoading = false);

     if (!mounted) return;
     if (success) {
       ScaffoldMessenger.of(context).showSnackBar(
         SnackBar(
           content: const Text('Password reset successfully! Please log in.', style: TextStyle(color: Colors.white)),
           backgroundColor: const Color(0xFF2D3A8C),
           // ... styling ...
         ),
       );
       Navigator.pushReplacementNamed(context, '/login');
     } else {
       ScaffoldMessenger.of(context).showSnackBar(
         SnackBar(
           content: const Text('Failed to reset password. Please try again.', style: TextStyle(color: Colors.white)),
           backgroundColor: Colors.redAccent,
           // ... styling ...
         ),
       );
     }
   }

The _submitNewPassword function drives the final Step 3. It utilizes the custom ``validatePassword`` 
function to ensure the new password meets security requirements (minimum length, 
special characters, etc.). It then extracts the verified email from Step 1 and 
passes both to the database's ``resetPassword`` method. Upon success, it 
automatically routes the user back to the login screen so they can authenticate 
with their new credentials.


Main Build & Wizard Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

build

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     return Scaffold(
       backgroundColor: const Color(0xFFF0F2F8),
       appBar: null,
       body: Center(
         child: SingleChildScrollView(
           padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 32),
           child: ConstrainedBox(
             constraints: const BoxConstraints(maxWidth: 420),
             child: Container(
               // ... container styling ...
               child: AnimatedSwitcher(
                 duration: const Duration(milliseconds: 300),
                 child: _step == 1
                     ? _StepEmail(
                         key: const ValueKey(1),
                         formKey: _emailFormKey,
                         controller: _emailController,
                         isLoading: _isLoading,
                         onSubmit: _submitEmail,
                       )
                     : _step == 2
                     ? _StepCode(
                         key: const ValueKey(2),
                         // ... code parameters ...
                       )
                     : _StepNewPassword(
                         key: const ValueKey(3),
                         // ... password parameters ...
                       ),
               ),
             ),
           ),
         ),
       ),
     );
   }

The primary ``build`` method constructs the centered, constrained card layout used 
throughout the authentication flow. Instead of relying on the Navigator to move 
between screens, it utilizes an ``AnimatedSwitcher``. This widget watches the 
``_step`` integer state variable; whenever it changes, the switcher gracefully 
fades out the current stateless UI component and fades in the next one. The 
``ValueKey`` assignments are critical here, as they force Flutter's rendering 
engine to recognize that the child widget has completely changed, triggering the 
animation.


Step UI Components
^^^^^^^^^^^^^^^^^^

To keep the main widget clean, the UI for each individual step of the wizard is 
extracted into its own private, stateless widget. They rely entirely on callbacks 
passed down from the parent state.


.. code-block:: dart

   class _StepEmail extends StatelessWidget {
     // ... constructor and variables ...

     @override
     Widget build(BuildContext context) {
       return Form(
         key: formKey,
         child: Column(
           mainAxisSize: MainAxisSize.min,
           crossAxisAlignment: CrossAxisAlignment.stretch,
           children: [
             // ... clickable logo ...
             const Text('Forgot Password?', ...),
             const Text('Enter your university email...', ...),
             
             AuthTextField(
               label: 'University Email Address',
               controller: controller,
               keyboardType: TextInputType.emailAddress,
               validator: validateUniversityEmail,
             ),
             
             // ... submit button tied to `onSubmit` callback ...
             _BackToLogin(),
           ],
         ),
       );
     }
   }

This is the _StepEmail component that renders the initial view. It encapsulates the email input field, 
binding it directly to the strict ``validateUniversityEmail`` logic to ensure only 
valid academic addresses are submitted to the backend.



.. code-block:: dart

   class _StepCode extends StatelessWidget {
    final GlobalKey<FormState> formKey;
    final TextEditingController controller;
    final bool isLoading;
    final VoidCallback onSubmit;   
    const _StepEmail({
      super.key,
      required this.formKey,
      required this.controller,
      required this.isLoading,
      required this.onSubmit,
    });

     @override
     Widget build(BuildContext context) {
       return Form(
         key: formKey,
         child: Column(
           // ...
           children: [
             // ...
             Text('We sent a 6-digit code to $email', ...),
             
             // Mock Hint Container
             Container(
               // ... styling ...
               child: const Row(
                 children: [
                   Icon(Icons.info_outline, color: Color(0xFFF9A825)),
                   Text('Demo hint: use code 123456'),
                 ],
               ),
             ),
             
             AuthTextField(
               label: 'Verification Code',
               controller: controller,
               keyboardType: TextInputType.number,
               validator: (v) {
                 if (v == null || v.trim().isEmpty) return 'Code is required';
                 if (v.trim().length != 6) return 'Code must be 6 digits';
                 return null;
               },
             ),
             // ... resend and submit buttons ...
           ],
         ),
       );
     }
   }

The second step, _StepCode, verifies the user's identity. Because this is currently a 
frontend-driven demonstration, this component includes a highly visible, styled mock 
hint container that provides the hardcoded verification code directly to the user. 
The text field validator enforces a strict 6-character limit to mimic a standard OTP 
(One-Time Password) input.


.. code-block:: dart

   class _StepNewPassword extends StatelessWidget {
     // ... constructor and variables ...

     @override
     Widget build(BuildContext context) {
       return Form(
         key: formKey,
         child: Column(
           // ...
           children: [
             AuthTextField(
               label: 'New Password',
               obscureText: obscureNew,
               controller: newPasswordController,
               validator: validatePassword,
               suffixWidget: IconButton(
                 icon: Icon(obscureNew ? Icons.visibility_outlined : Icons.visibility_off_outlined),
                 onPressed: onToggleNew,
               ),
             ),
             
             AuthTextField(
               label: 'Confirm New Password',
               obscureText: obscureConfirm,
               controller: confirmPasswordController,
               validator: (v) {
                 if (v != newPasswordController.text) return 'Passwords do not match';
                 return null;
               },
               suffixWidget: IconButton(
                 icon: Icon(obscureConfirm ? Icons.visibility_outlined : Icons.visibility_off_outlined),
                 onPressed: onToggleConfirm,
               ),
             ),
             // ... submit button ...
           ],
         ),
       );
     }
   }

The final step, _StepNewPassword, handles the creation of the new credential. It renders two password 
inputs that share synchronized validation logic (ensuring the confirmation field 
exactly matches the primary field). It also wires up the ``onToggleNew`` and 
``onToggleConfirm`` callbacks to the trailing eye icons, allowing the user to reveal 
or obscure their secure inputs.

Back to Login
^^^^^^^^^^^^^

.. code-block:: dart

   class _BackToLogin extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return Row(
         mainAxisAlignment: MainAxisAlignment.center,
         children: [
           const Icon(Icons.arrow_back, size: 13, color: Colors.grey),
           const SizedBox(width: 4),
           GestureDetector(
             onTap: () => Navigator.pushReplacementNamed(context, '/login'),
             child: const Text('Back to Login', ...),
           ),
         ],
       );
     }
   }

_BackToLogin is a shared, reusable footer component. Rather than duplicating this navigation row at the bottom of all three step widgets, it is abstracted into this single class, providing a consistent escape hatch that routes the user away from the wizard and back to the login screen.