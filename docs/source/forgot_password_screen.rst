===========================
forgot_password_screen.dart
===========================

``forgot_password_screen.dart`` handles the complete password reset workflow.
Rather than routing the user across multiple separate pages, it operates as a 
seamless, 3-step "Wizard" form controlled by local state. It manages email 
verification, mock code validation, and secure password updates before routing 
the user back to the login screen.

Imports
-------

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
----------------

State Initialisation
^^^^^^^^^^^^^^^^^^^^

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
-----------------------

These methods encapsulate the asynchronous business logic that validates user 
inputs and dictates the progression of the wizard steps.

_submitEmail
^^^^^^^^^^^^

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

This function drives Step 1. It validates the email input against the standard 
university regex and queries the ``SqliteBackend`` to check if an active account 
matches the address. If an account is found, it progresses the wizard to Step 2; 
otherwise, it surfaces a prominent red error ``SnackBar`` to inform the user that 
the account does not exist.

_submitCode
^^^^^^^^^^^

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

This function drives Step 2. It simulates network latency with a brief delay and 
compares the user's input against a hardcoded mock value (``123456``). If the code 
matches, the wizard progresses to the final step.

_submitNewPassword
^^^^^^^^^^^^^^^^^^

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

This function drives the final Step 3. It utilizes the custom ``validatePassword`` 
function to ensure the new password meets security requirements (minimum length, 
special characters, etc.). It then extracts the verified email from Step 1 and 
passes both to the database's ``resetPassword`` method. Upon success, it 
automatically routes the user back to the login screen so they can authenticate 
with their new credentials.


Main Build & Wizard Controller
------------------------------

build
^^^^^

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
------------------

To keep the main widget clean, the UI for each individual step of the wizard is 
extracted into its own private, stateless widget. They rely entirely on callbacks 
passed down from the parent state.

_StepEmail
^^^^^^^^^^

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

This component renders the initial view. It encapsulates the email input field, 
binding it directly to the strict ``validateUniversityEmail`` logic to ensure only 
valid academic addresses are submitted to the backend.

_StepCode
^^^^^^^^^

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

The second step verifies the user's identity. Because this is currently a 
frontend-driven demonstration, this component includes a highly visible, styled mock 
hint container that provides the hardcoded verification code directly to the user. 
The text field validator enforces a strict 6-character limit to mimic a standard OTP 
(One-Time Password) input.

_StepNewPassword
^^^^^^^^^^^^^^^^

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

The final step handles the creation of the new credential. It renders two password 
inputs that share synchronized validation logic (ensuring the confirmation field 
exactly matches the primary field). It also wires up the ``onToggleNew`` and 
``onToggleConfirm`` callbacks to the trailing eye icons, allowing the user to reveal 
or obscure their secure inputs.

_BackToLogin
^^^^^^^^^^^^

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

A shared, reusable footer component. Rather than duplicating this navigation row at the bottom of all three step widgets, it is abstracted into this single class, providing a consistent escape hatch that routes the user away from the wizard and back to the login screen.