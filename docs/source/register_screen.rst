register_screen.dart
====================

``register_screen.dart`` is the account registration screen for new UniSphere users.
It gathers identity details, validates the form, checks that the user agrees to the terms,
and creates the account through the backend before routing the user into the logged-in
experience.

Imports
-------

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
--------------------

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
---------------------

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
--------------

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
--------------------

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

Terms Guard
^^^^^^^^^^^

.. code-block:: dart

    if (!_agreeToTerms) {
       ScaffoldMessenger.of(context).showSnackBar(...);
       return;
    }

This guard blocks registration until the user has agreed to the Terms of Service and Privacy
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
------------

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



Registration Card
^^^^^^^^^^^^^^^^^

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

Logo
^^^^

.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushNamed(context, '/'),
       child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
    )

The logo is a tappable brand element that routes the user back to the public landing page.
It gives the registration screen a clear escape route and keeps the app identity visible at the
top of the form. The gesture wrapper matters because the image itself acts as navigation rather
than static decoration.

Title and Intro Text
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    const Text('Create your Account')
    const Text('Join UniSphere and connect with your university community.')

These two text widgets introduce the registration flow and explain the purpose
of the screen.

Name Fields
^^^^^^^^^^^

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

These fields collect the user’s first and last name side by side. They reduce vertical space
while capturing identity details early in the form, which matches the rest of the compact
registration layout. Each field uses a simple required validator so the form can reject
incomplete submissions immediately.

University Email Field
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    AuthTextField(
       label: 'University Email Address',
       controller: _emailController,
       keyboardType: TextInputType.emailAddress,
       validator: validateUniversityEmail,
    )

This field collects the user’s university email address. It uses the shared university email
validator to make sure the address matches the expected institutional format before registration
can continue. The email is later normalised to lowercase in the submission handler, so the stored
value is consistent.

Institution Autocomplete
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Autocomplete<String>(
       optionsBuilder: (TextEditingValue textEditingValue) { ... },
       fieldViewBuilder: (context, controller, focusNode, onFieldSubmitted) { ... },
       onSelected: (String selection) { ... },
    )

This autocomplete field lets the user search for and select their institution from the known
university list. It supports both typing and suggestion filtering, which makes it easier to find a
valid institution while still keeping the input constrained to supported values. The selected
value is mirrored into the visible form controller, so the field and the stored institution stay
in sync.

Institution Field Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

This validator checks that an institution was provided, that the value exists in the supported
university list, and that the institution does not match the user’s own email-domain university.
It protects the registration flow from invalid or contradictory account data. The domain
comparison is an important subtlety because it prevents users from registering with their own
university as a disallowed case in this flow.

Password Field
^^^^^^^^^^^^^^

.. code-block:: dart

    AuthTextField(
       label: 'Password',
       obscureText: _obscurePassword,
       controller: _passwordController,
       validator: validatePassword,
    )

This field captures the account password. It uses the shared password validator and hides the
input while the obscured state is enabled. The field is part of the shared authentication pattern,
so it behaves consistently with the login screen.

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

This icon button toggles the password field between hidden and visible text. It updates the
password visibility state so the icon and field behaviour stay aligned. The toggle matters because
it gives users control over checking the password they entered without leaving the form.

Confirm Password Field
^^^^^^^^^^^^^^^^^^^^^^

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

This field repeats the password entry so the user can confirm the value before submitting the
form. It validates that the confirmation is not empty and that it matches the primary password
field. This extra check reduces accidental mismatches before the backend request is made.

Confirm Password Toggle
^^^^^^^^^^^^^^^^^^^^^^^

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

Create Account Button
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

This button triggers the registration handler. It disables itself and shows a loading indicator
while the backend request is running so users do not submit the form twice. The loading state is
important because registration is asynchronous and the UI needs to reflect that the app is waiting
for a backend response.

Sign In Link
^^^^^^^^^^^^

.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushReplacementNamed(context, '/login'),
       child: const Text('Sign In'),
    )

This footer link routes existing users to the login screen. It uses replacement navigation so the
registration page does not remain in the back stack after the user moves to sign in. That
navigation choice keeps the auth flow clean and avoids sending the user back to an unnecessary
form.