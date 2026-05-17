=======
Widgets - app_footer.dart, auth_header.dart etc.
=======

This section contains the documentation for all reusable, stateless, 
and stateful visual components found within the ``widgets`` directory. Abstracting 
these elements ensures design consistency across the application, enforces routing 
security, and significantly reduces boilerplate code in the main layout trees.


app_footer.dart
---------------

``app_footer.dart`` contains the global footer widget used across most screens 
in the application. It provides a standardized, responsive layout for essential 
navigation links, branding, and legal pages.

AppFooter
^^^^^^^^^

.. code-block:: dart

   class AppFooter extends StatelessWidget {
     final String brandName;
     final String tagline;
     final String copyrightText;

     const AppFooter({
       super.key,
       this.brandName = 'UniSphere',
       this.tagline = 'Discover, share, and manage events with confidence.',
       this.copyrightText = '© UniSphere — Event Discovery Platform',
     });

     @override
     Widget build(BuildContext context) {
       return Container(
         width: double.infinity,
         color: const Color(0xff111827),
         padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 60),
         child: Center(
           child: ConstrainedBox(
             constraints: const BoxConstraints(maxWidth: 1200),
             child: Column(
               // ... Grid of _FooterColumn widgets ...
               // ... LayoutBuilder for responsive copyright and legal links ...
             ),
           ),
         ),
       );
     }
   }

This main widget defines a dark, professional footer constrained to a maximum 
width of 1200px. It organizes navigation links into columns for larger screens 
and collapses the legal links (About, Privacy, Terms) appropriately on mobile views 
using an internal ``LayoutBuilder``.

_FooterColumn
^^^^^^^^^^^^^

.. code-block:: dart

   class _FooterColumn extends StatelessWidget {
     final String title;
     final List<Map<String, String?>> links;

     // ...

     @override
     Widget build(BuildContext context) {
       return SizedBox(
         width: 160,
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Text(title, ...),
             ...links.map(
               (linkData) => Padding(
                 padding: const EdgeInsets.only(bottom: 12),
                 child: InkWell(
                   onTap: () {
                     final route = linkData['route'];
                     if (route != null) {
                       Future.microtask(() {
                         if (context.mounted) {
                           final requiresAuth = ['/create-event', '/profile', '/calendar', '/my-events'].contains(route);
                           if (requiresAuth && SqliteBackend().currentUser == null) {
                             Navigator.pushNamed(context, '/register');
                           } else {
                             Navigator.pushNamed(context, route);
                           }
                         }
                       });
                     }
                   },
                   child: Text(linkData['label'] ?? '', ...),
                 ),
               ),
             ),
           ],
         ),
       );
     }
   }

A private helper widget that renders a vertical column of links. It handles 
internal routing and securely enforces authentication checks, seamlessly redirecting 
guest users to the register page if they click on protected routes like 
``/create-event`` or ``/profile``.


auth_header.dart
----------------

``auth_header.dart`` provides a simplified, non-interactive navigation bar 
specifically designed for the authentication flows (Login and Register screens).

AuthHeader
^^^^^^^^^^

.. code-block:: dart

   class AuthHeader extends StatelessWidget implements PreferredSizeWidget {
     const AuthHeader({super.key});

     @override
     Size get preferredSize => const Size.fromHeight(kToolbarHeight);

     @override
     Widget build(BuildContext context) {
       return AppBar(
         backgroundColor: Colors.white,
         elevation: 1,
         automaticallyImplyLeading: false,
         title: Padding(
           padding: const EdgeInsets.symmetric(horizontal: 24),
           child: Row(
             children: [
               // ... Brand Logo ...
               const Spacer(),
               // ... Help Center & Language Selectors ...
             ],
           ),
         ),
       );
     }
   }

Implementing ``PreferredSizeWidget``, this app bar strips away primary navigation 
links to prevent users from abandoning the authentication process. It anchors the 
UniSphere logo on the left alongside secondary, non-destructive actions 
(like a language selector) on the right.


auth_text_field.dart
--------------------

``auth_text_field.dart`` encapsulates the standardized input styling used in 
forms throughout the application.

AuthTextField
^^^^^^^^^^^^^

.. code-block:: dart

   class AuthTextField extends StatelessWidget {
     final String label;
     final String? hintText;
     final IconData prefixIcon;
     final Widget? suffixWidget;
     final bool obscureText;
     final TextEditingController? controller;
     final TextInputType keyboardType;
     final String? Function(String?)? validator;

     // ...

     @override
     Widget build(BuildContext context) {
       return Column(
         crossAxisAlignment: CrossAxisAlignment.start,
         children: [
           Text(label, ...),
           const SizedBox(height: 6),
           TextFormField(
             controller: controller,
             obscureText: obscureText,
             keyboardType: keyboardType,
             validator: validator,
             decoration: InputDecoration(
               // ... standardized borders, padding, and colors ...
             ),
           ),
         ],
       );
     }
   }

This reusable component enforces strict consistency for typography, border 
radius, error states, and focus colors. It accepts a wide range of parameters 
(icons, visibility toggles, validators) to drastically reduce boilerplate code 
in complex layouts like the password reset wizard.


header.dart
-----------

``header.dart`` contains the primary global navigation bar used by both 
authenticated and guest users across the main application views.

AppHeader
^^^^^^^^^

.. code-block:: dart

   class AppHeader extends StatelessWidget {
     // ... optional callbacks for specific route overrides ...

     @override
     Widget build(BuildContext context) {
       final width = MediaQuery.of(context).size.width;
       final isMobile = width < 850;

       return Container(
         width: double.infinity,
         color: Colors.white.withOpacity(0.96),
         padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 20),
         child: Center(
           child: ConstrainedBox(
             constraints: const BoxConstraints(maxWidth: 1200),
             child: isMobile
                 ? Row(
                     mainAxisAlignment: MainAxisAlignment.spaceBetween,
                     children: [
                       // ... Mobile popup menu ...
                     ]
                   )
                 : Row(
                     mainAxisAlignment: MainAxisAlignment.spaceBetween,
                     children: [
                       // ... Desktop navigation row ...
                     ]
                   )
           ),
         ),
       );
     }
   }

The main header dynamically adapts its layout based on screen width. On wide 
screens, it lists explicit navigation buttons. On mobile devices, it condenses 
those options into a ``PopupMenuButton``. It reads the ``SqliteBackend().currentUser`` 
state to conditionally render options like "Sign In" vs. "My Profile".

_Brand & _NavItem
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _Brand extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return GestureDetector(
         onTap: () {
           final user = SqliteBackend().currentUser;
           Future.microtask(() {
             if (context.mounted) {
               Navigator.pushNamedAndRemoveUntil(
                 context,
                 user != null ? '/logged-in' : '/',
                 (route) => false,
               );
             }
           });
         },
         child: Row(
           // ... scaling overflow image and title ...
         ),
       );
     }
   }

   class _NavItem extends StatelessWidget {
     // ...
   }

These are lightweight structural utilities. ``_Brand`` houses the application 
logo and implements smart routing: clicking it returns an authenticated user to 
their dashboard, but returns a guest to the public landing page. ``_NavItem`` 
standardizes the padding and typography for desktop header links.


pagination_controls.dart
------------------------

``pagination_controls.dart`` provides a responsive UI component for navigating 
through paginated lists of data, such as the event discovery feed.

PaginationControls
^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class PaginationControls extends StatelessWidget {
     final int currentPage;
     final int totalPages;
     final VoidCallback onPrevious;
     final VoidCallback onNext;

     // ...

     @override
     Widget build(BuildContext context) {
       if (totalPages <= 1) return const SizedBox.shrink();

       return Container(
         // ... styling ...
         child: LayoutBuilder(
           builder: (context, constraints) {
             final compact = constraints.maxWidth < 420;
             final status = Row( /* ... Page X of Y ... */ );
             final controls = Row( /* ... Previous/Next buttons ... */ );

             if (compact) {
               return Column(children: [status, const SizedBox(height: 12), controls]);
             }
             return Row(children: [status, const Spacer(), controls]);
           },
         ),
       );
     }
   }

This widget acts as the logical wrapper for pagination. It automatically hides 
itself entirely if only one page of data exists. Utilizing a ``LayoutBuilder``, 
it gracefully adapts from a spaced-out desktop row to a vertically stacked column 
on compact mobile devices.

_PagerButton
^^^^^^^^^^^^

.. code-block:: dart

   class _PagerButton extends StatelessWidget {
     final String label;
     final IconData icon;
     final bool enabled;
     final VoidCallback onPressed;
     final bool inverse;
     
     class _PagerButton extends StatelessWidget {
  final String label;
  final IconData icon;
  final bool enabled;
  final VoidCallback onPressed;
  final bool inverse;

  const _PagerButton({
    required this.label,
    required this.icon,
    required this.enabled,
    required this.onPressed,
    required this.inverse,
  });

  @override
  Widget build(BuildContext context) {
    final background = enabled
        ? (inverse ? const Color(0xFF4F46E5) : const Color(0xFFF8FAFC))
        : const Color(0xFFF3F4F6);
    final foreground = enabled
        ? (inverse ? Colors.white : const Color(0xFF111827))
        : const Color(0xFF9CA3AF);

    return Material(
      color: Colors.transparent,
      child: InkWell(
        onTap: enabled ? onPressed : null,
        borderRadius: BorderRadius.circular(14),
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 160),
          padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
          decoration: BoxDecoration(
            color: background,
            borderRadius: BorderRadius.circular(14),
            border: Border.all(
              color: enabled
                  ? (inverse
                        ? const Color(0xFF4F46E5)
                        : const Color(0xFFD1D5DB))
                  : const Color(0xFFE5E7EB),
            ),
          ),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(icon, size: 18, color: foreground),
              const SizedBox(width: 6),
              Text(
                label,
                style: TextStyle(
                  fontSize: 13,
                  fontWeight: FontWeight.w700,
                  color: foreground,
                ),
              ),
            ],
          ),
        ),
      ),
    );
   }

A localized sub-component representing the "Previous" and "Next" actions. It safely 
handles disabled bounds (greying out colors and removing ripple effects on the 
first/last pages) and supports an ``inverse`` boolean parameter to visually 
highlight the forward/primary action button.