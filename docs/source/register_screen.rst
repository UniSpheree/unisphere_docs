register_screen.dart
====================

``register_screen.dart`` is the account registration screen that allows a new
user to create a UniSphere account. It collects identity details, validates
form input, enforces consent to terms, and routes to the logged-in landing
page when registration succeeds.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';
   import '../utils/validators.dart';
   import '../widgets/header.dart';
   import 'create_event_screen.dart';
   import 'discover_event_screen.dart';
   import '../widgets/auth_text_field.dart';
   import '../utils/unis.dart';
   import '../utils/mock_backend.dart';

``material.dart`` provides the Flutter UI framework for the registration form. 
The validator, auth text field, university list, and backend utilities support field 
validation, dropdown data, and account creation. The navigation imports keep the 
page connected to the rest of the app.

Main Register Widget
--------------------

.. code-block:: dart

    class RegisterScreen extends StatefulWidget {
       const RegisterScreen({super.key});

       @override
       State<RegisterScreen> createState() => _RegisterScreenState();
    }

The RegisterScreen class is a StatefulWidget. It takes no additional attributes
and returns an instance of ``_RegisterScreenState`` so the page can manage text
controllers, UI toggles, and async registration feedback.

Register Screen State
---------------------

.. code-block:: dart

    class _RegisterScreenState extends State<RegisterScreen> {
       final _formKey = GlobalKey<FormState>();
       final _firstNameController = TextEditingController();
       final _lastNameController = TextEditingController();
       final _emailController = TextEditingController();
       String? _selectedUniversity;
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

Here we define the state for the RegisterScreen workflow. This state class
stores the form key, all text controllers, visibility flags for password
fields, the terms agreement toggle, and the loading state used while
submitting.

The ``_formKey`` validates all fields before registration. The controller
attributes store first name, last name, email, institution, and password
inputs. ``_selectedUniversity`` tracks the selected autocomplete option.
``_obscurePassword`` and ``_obscureConfirm`` switch secure text visibility,
``_agreeToTerms`` enforces consent, and ``_isLoading`` disables submission
during backend requests.

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

The dispose method releases all text controllers when the page is removed from
the widget tree. This prevents memory leaks and keeps the registration screen
state lifecycle clean.

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

The ``_handleRegister`` method controls account creation. It takes no
parameters and returns ``Future<void>`` because registration is asynchronous.
The method validates the form, enforces terms acceptance, normalizes submitted
values, and calls ``MockBackend().register``.

If registration succeeds, it shows a success SnackBar and clears the navigation
stack by routing to ``/logged-in``. If registration fails, it shows an error
SnackBar indicating an existing account.

Terms Guard
^^^^^^^^^^^

.. code-block:: dart

    if (!_agreeToTerms) {
       ScaffoldMessenger.of(context).showSnackBar(...);
       return;
    }

This guard ensures registration cannot continue unless the user agrees to the
Terms of Service and Privacy Policy.

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

This backend call submits the prepared registration payload. The method returns
a boolean success result that determines which feedback and navigation branch is
executed.

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
                            constraints: const BoxConstraints(maxWidth: 480),
                            child: Container(
                               child: Form(
                                  key: _formKey,
                                  child: Column(
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
                ),
             ],
          ),
       );
    }

The build method returns the main registration interface. It uses a scrollable,
constrained card layout so the form remains readable across smaller screens and
desktop widths.

App Header
^^^^^^^^^^

.. code-block:: dart

    AppHeader(
            onHostEventTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => const CreateEventScreen(),
                ),
              );
            },
            onRegisterTap: () {},
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
              Navigator.pushReplacementNamed(context, '/login');
            },
            showProfile: false,
          ),

The shared header provides fast navigation to discovery and event creation.
``onRegisterTap`` is intentionally inactive because this is already the
registration screen. ``onSignInTap`` routes users to the login page.

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

The registration form is wrapped in a centered card with rounded corners and
shadow. This separates the form visually from the background and keeps the
authentication content focused.

Logo
^^^^

.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushNamed(context, '/'),
       child: Image.asset('assets/image.png', height: 64, fit: BoxFit.contain),
    )

The logo is a tappable brand element that routes users back to the public
landing page.

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

The first and last name inputs are displayed side by side to reduce vertical
space and collect identity details early in the form. Each field uses a simple
required validator.

University Email Field
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    AuthTextField(
       label: 'University Email Address',
       controller: _emailController,
       keyboardType: TextInputType.emailAddress,
       validator: validateUniversityEmail,
    )

This field collects the user's university email and validates it using
``validateUniversityEmail`` from the shared validators utility.

Institution Autocomplete
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Autocomplete<String>(
       optionsBuilder: (TextEditingValue textEditingValue) { ... },
       fieldViewBuilder: (context, controller, focusNode, onFieldSubmitted) { ... },
       onSelected: (String selection) { ... },
    )

The institution picker combines free typing with a filtered suggestion list from
``ukUniversities``. It supports quick matching while still validating that the
selected institution exists in the known dataset.

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

This validator enforces three checks: a value must be provided, it must exist
in the supported university list, and it cannot match the user's own email
domain institution.

Password Field
^^^^^^^^^^^^^^

.. code-block:: dart

    AuthTextField(
       label: 'Password',
       obscureText: _obscurePassword,
       controller: _passwordController,
       validator: validatePassword,
    )

The password field captures the account password, validates strength using
``validatePassword``, and hides characters while ``_obscurePassword`` is true.

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

This toggle switches password visibility and keeps the icon state synchronized
with the field behavior.

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

The confirm password field ensures the repeated password matches the primary
password value before registration can continue.

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

This toggle controls visibility for the confirm password field independently of
the main password field.

Terms and Privacy Consent
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Row(
       children: [
          Checkbox(
             value: _agreeToTerms,
             onChanged: (v) => setState(() => _agreeToTerms = v ?? false),
          ),
          RichText(
             text: TextSpan(
                children: [
                   TextSpan(text: 'Terms of Service'),
                   TextSpan(text: 'Privacy Policy'),
                ],
             ),
          ),
       ],
    )

This consent block stores whether the user agrees to legal terms before account
creation. The highlighted text visually emphasizes the Terms of Service and
Privacy Policy references.

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

The primary submit button triggers registration through ``_handleRegister``.
When ``_isLoading`` is true, it disables interaction and shows a progress
indicator until the backend call completes.

Sign In Link
^^^^^^^^^^^^

.. code-block:: dart

    GestureDetector(
       onTap: () => Navigator.pushReplacementNamed(context, '/login'),
       child: const Text('Sign In'),
    )

This footer link routes existing users to the login screen using replacement
navigation so they do not stack the registration page in history.
The _RegisterScreenState class stores the text controllers, selected university value, password visibility flags, and submission state for the sign-up form. It returns the full registration experience.