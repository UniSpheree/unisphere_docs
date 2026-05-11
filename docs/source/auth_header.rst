auth_header.dart
=================

'auth_header.dart' is a simplified version of the main header widget used on authentication pages such as login and registration.
It provides a cleaner, lighter navigation bar for authentication flows while preserving brand identity and useful utility actions.

Imports
--------

.. code-block:: dart

   import 'package:flutter/material.dart';

`material.dart` provides the Flutter UI framework and layout widgets used by the authentication header.

Widget
------

.. code-block:: dart

   class AuthHeader extends StatelessWidget implements PreferredSizeWidget

The ``AuthHeader`` widget is the primary component in this file. It implements 
``PreferredSizeWidget`` so it can be used directly as an app bar.

Constructor
-----------

.. code-block:: dart

   const AuthHeader({super.key});

This constructor creates a stateless authentication header with no additional 
configuration.

Preferred Size
--------------

.. code-block:: dart

   @override
   Size get preferredSize => const Size.fromHeight(kToolbarHeight);

The widget returns the standard toolbar height so it integrates cleanly with 
Flutter's app bar layout.

Appearance
----------

.. code-block:: dart

   AppBar(
     backgroundColor: Colors.white,
     elevation: 1,
     shadowColor: Colors.black12,
     automaticallyImplyLeading: false,
     titleSpacing: 0,
     title: Padding(
       padding: const EdgeInsets.symmetric(horizontal: 24),
       child: Row(
         children: [
           ...
         ],
       ),
     ),
   );

The authentication header renders an ``AppBar`` with a white background, subtle shadow, 
and no automatic leading button. This keeps the layout simple and focused.

Branding
--------

.. code-block:: dart

   Container(
     width: 32,
     height: 32,
     decoration: BoxDecoration(
       color: const Color(0xFF2D3A8C),
       borderRadius: BorderRadius.circular(8),
     ),
     child: const Icon(
       Icons.hub_outlined,
       color: Colors.white,
       size: 18,
     ),
   );

The header displays the UniSphere brand with a colored icon container and 
bold text label.


Actions
-------


.. code-block:: dart

   TextButton(
     onPressed: () {},
     child: Text('Help Center', ...),
   );

   OutlinedButton.icon(
     onPressed: () {},
     icon: const Text('🌐', style: TextStyle(fontSize: 13)),
     label: const Text('English', ...),
   );

The right side of the header contains utility actions for authentication pages 
with actions to keep support and language selection available without overwhelming 
the authentication flow.

Summary
-------
