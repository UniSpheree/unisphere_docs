=================
profile_page.dart
=================

``profile_page.dart`` acts as the central hub and dashboard for the user's account. 
It allows users to manage their personal information, seamlessly toggle their 
system role between "Attendee" and "Organiser," and access deep links to 
role-specific tools (like the event creator and calendar). The layout is highly 
responsive, utilizing a modular card-based design that adapts gracefully to 
different screen sizes.

Imports
-------

.. code-block:: dart

   import 'package:flutter/material.dart';

   import '../services/sqlite_backend.dart';
   import '../models/database_models.dart';
   import '../widgets/app_footer.dart';
   import '../widgets/header.dart';
   import 'calendar_page.dart';
   import 'create_event_screen.dart';
   import 'discover_event_screen.dart';
   import 'my_events_page.dart';

This block imports standard Flutter material components alongside the application's 
local SQLite database service and user models. It also imports the shared global 
header/footer widgets, as well as the target screen widgets required for the 
dashboard's quick-action navigation tiles.


State Initialization & Dynamic Getters
--------------------------------------

_ProfilePageState
^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class ProfilePage extends StatefulWidget {
     final ImageProvider? image;

     const ProfilePage({super.key, this.image});

     @override
     State<ProfilePage> createState() => _ProfilePageState();
   }

   class _ProfilePageState extends State<ProfilePage> {
     DbUser? _user;
     late final TextEditingController _nameController;
     late final TextEditingController _descriptionController;

     @override
     void initState() {
       super.initState();
       _user = SqliteBackend().currentUser;
       _nameController = TextEditingController(text: _displayName);
       _descriptionController = TextEditingController(text: _displayDescription);
     }

     @override
     void dispose() {
       _nameController.dispose();
       _descriptionController.dispose();
       super.dispose();
     }
     
     // ...
   }

The state class fetches and stores the active ``_user`` object from the local 
backend upon initialization. It spins up two ``TextEditingController`` instances 
required for the profile editing modal, intentionally pre-populating them using the 
dynamic getter methods to ensure the user isn't presented with blank fields.

Dynamic Getters
^^^^^^^^^^^^^^^

.. code-block:: dart

   String get _displayName {
     final user = _user;
     if (user == null || user.fullName.trim().isEmpty) return 'Guest User';
     return user.fullName;
   }

   String get _displayDescription {
     final description = _user?.description.trim() ?? '';
     return description.isNotEmpty
         ? description
         : 'Add a short bio to introduce yourself to the community.';
   }

   String get _email => _user?.email ?? 'alex@university.edu';
   String get _university => _user?.university ?? 'University of Example';
   String get _role => _user?.role ?? 'Attendee';
   bool get _isOrganiser => _role.toLowerCase() == 'organiser';

To keep the UI code clean and robust against null or incomplete database records, 
the file utilizes a series of Dart getter methods. These getters act as a safety 
net, instantly falling back to default placeholder text (e.g., "Guest User") if the 
backend returns missing or empty string data.


Profile Logic Handlers
----------------------

_saveProfile
^^^^^^^^^^^^

.. code-block:: dart

   Future<void> _saveProfile() async {
     final updatedUser = await SqliteBackend().updateCurrentUserProfile(
       name: _nameController.text,
       description: _descriptionController.text,
     );

     if (!mounted || updatedUser == null) return;
     setState(() {
       _user = updatedUser;
       _nameController.text = updatedUser.fullName;
       _descriptionController.text = updatedUser.description;
     });

     ScaffoldMessenger.of(context).showSnackBar(
       SnackBar(
         content: Text('Profile updated for ${updatedUser.fullName}.'),
         behavior: SnackBarBehavior.floating,
       ),
     );
   }

This asynchronous method extracts the text from the edit modal's controllers and 
pushes the changes to the ``SqliteBackend``. Upon a successful database update, it 
triggers a local state rebuild with the fresh ``updatedUser`` object and displays a 
confirmation snackbar.

_setRole
^^^^^^^^

.. code-block:: dart

   Future<void> _setRole(bool organiser) async {
     final nextRole = organiser ? 'Organiser' : 'Attendee';
     final updatedUser = await SqliteBackend().updateCurrentUserRole(nextRole);
     if (!mounted || updatedUser == null) return;

     setState(() => _user = updatedUser);
     ScaffoldMessenger.of(context).showSnackBar(
       SnackBar(
         content: Text('Role changed to ${updatedUser.role}. Permissions updated.'),
         behavior: SnackBarBehavior.floating,
       ),
     );
   }

This method handles the core permission toggle of the application. Driven by a 
boolean input from the UI's role chips, it commits the string value of the desired 
role to the backend, immediately updating the UI state to instantly lock or unlock 
organizer-specific features on the dashboard.

_openEditDialog
^^^^^^^^^^^^^^^

.. code-block:: dart

   Future<void> _openEditDialog() async {
     _nameController.text = _displayName;
     _descriptionController.text = _user?.description ?? '';

     await showDialog<void>(
       context: context,
       builder: (dialogContext) {
         return AlertDialog(
           shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(24)),
           title: const Text('Edit profile'),
           content: SizedBox(
             width: 520,
             child: SingleChildScrollView(
               child: Column(
                 mainAxisSize: MainAxisSize.min,
                 crossAxisAlignment: CrossAxisAlignment.start,
                 children: [
                   Text('Only name and description can be changed here.', ...),
                   const SizedBox(height: 18),
                   TextFormField(
                     controller: _nameController,
                     decoration: const InputDecoration(labelText: 'Full name', ...),
                   ),
                   const SizedBox(height: 16),
                   TextFormField(
                     controller: _descriptionController,
                     minLines: 4,
                     maxLines: 6,
                     decoration: const InputDecoration(labelText: 'Description', ...),
                   ),
                   const SizedBox(height: 18),
                   _ReadOnlyField(label: 'Email', value: _email),
                   const SizedBox(height: 12),
                   _ReadOnlyField(label: 'University', value: _university),
                 ],
               ),
             ),
           ),
           actions: [
             TextButton(
               onPressed: () => Navigator.pop(dialogContext),
               child: const Text('Cancel'),
             ),
             FilledButton(
               onPressed: () {
                 Navigator.pop(dialogContext);
                 _saveProfile();
               },
               child: const Text('Save changes'),
             ),
           ],
         );
       },
     );
   }

Rather than forcing the user to navigate to a completely separate "Edit Profile" 
screen, this method overlays a sleek ``AlertDialog``. It resets the text controllers 
to their latest values upon opening to ensure the user is editing their current data. 
It renders the editable fields (name and description) alongside the non-editable data 
(rendered safely via the ``_ReadOnlyField`` utility widget) before culminating in an 
action row that either dismisses the modal or triggers ``_saveProfile()``.


Main Build & Responsive Layout
------------------------------

The primary ``build`` method utilizes nested ``LayoutBuilder`` widgets to create 
a highly responsive dashboard that smoothly transitions from a wide-screen, 
multi-column layout down to a vertically stacked mobile view.

build & Breadcrumbs
^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   @override
   Widget build(BuildContext context) {
     return Scaffold(
       backgroundColor: const Color(0xFFF5F7FB),
       body: LayoutBuilder(
         builder: (context, constraints) {
           return SingleChildScrollView(
             child: ConstrainedBox(
               constraints: BoxConstraints(minHeight: constraints.maxHeight),
               child: Column(
                 mainAxisAlignment: MainAxisAlignment.spaceBetween,
                 children: [
                   Column(
                     children: [
                       AppHeader(...),
                       Padding(
                         padding: const EdgeInsets.fromLTRB(20, 24, 20, 0),
                         child: Center(
                           child: ConstrainedBox(
                             constraints: const BoxConstraints(maxWidth: 1120),
                             child: Column(
                               crossAxisAlignment: CrossAxisAlignment.start,
                               children: [
                                 Row(
                                   // ... Breadcrumb navigation (Home / Profile) ...
                                 ),
                                 const SizedBox(height: 16),
                                 
                                 // ... Hero Banner ...
                                 // ... Dashboard Cards ...
                                 // ... Session Management ...
                                 
                               ],
                             ),
                           ),
                         ),
                       ),
                     ],
                   ),
                   const AppFooter(),
                 ],
               ),
             ),
           );
         },
       ),
     );
   }

The top-level structure wraps the content in a ``SingleChildScrollView`` and a 
``ConstrainedBox`` set to a maximum width of 1120px, ensuring the UI doesn't stretch 
awkwardly on ultrawide monitors. It includes a standard breadcrumb row at the top 
left, providing quick "Back to Home" navigation without requiring browser-level 
controls.

Hero Banner
^^^^^^^^^^^

.. code-block:: dart

   Container(
     width: double.infinity,
     padding: const EdgeInsets.all(24),
     decoration: BoxDecoration(
       gradient: const LinearGradient(
         colors: [Color(0xFF111827), Color(0xFF4F46E5)],
         begin: Alignment.topLeft,
         end: Alignment.bottomRight,
       ),
       borderRadius: BorderRadius.circular(28),
       // ... shadows ...
     ),
     child: LayoutBuilder(
       builder: (context, constraints) {
         final stacked = constraints.maxWidth < 820;
         final avatar = CircleAvatar(...);
         final heroText = Expanded(
           child: Column(
             crossAxisAlignment: CrossAxisAlignment.start,
             children: [
               Wrap(
                 // ... Name and Role Pill ...
               ),
               Text(_displayDescription, ...),
               Wrap(
                 // ... Email and University MetaChips ...
               ),
               Wrap(
                 // ... Edit Profile and Browse Events buttons ...
               ),
             ],
           ),
         );

         if (stacked) {
           return Column(
             crossAxisAlignment: CrossAxisAlignment.start,
             children: [avatar, const SizedBox(height: 18), heroText],
           );
         }
         return Row(
           crossAxisAlignment: CrossAxisAlignment.center,
           children: [avatar, const SizedBox(width: 22), heroText],
         );
       },
     ),
   )

The Hero Banner serves as the visual anchor of the profile. It features a bold 
gradient background and utilizes its own internal ``LayoutBuilder``. If the 
available width drops below 820 pixels, it flips the layout of the user's avatar 
and textual information from a side-by-side ``Row`` into a vertical ``Column`` to 
prevent the text from clipping on smaller screens.

Dashboard Sections (Cards)
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   LayoutBuilder(
     builder: (context, constraints) {
       final narrow = constraints.maxWidth < 860;

       final accountCard = _SectionCard(
         title: 'Account details',
         // ... email, university, and role info rows ...
       );

       final roleCard = _SectionCard(
         title: 'Role & permissions',
         child: Column(
           children: [
             Row(
               children: [
                 ChoiceChip(
                   label: const Text('Attendee'),
                   selected: !_isOrganiser,
                   onSelected: (_) => _setRole(false),
                 ),
                 ChoiceChip(
                   label: const Text('Organiser'),
                   selected: _isOrganiser,
                   onSelected: (_) => _setRole(true),
                 ),
               ],
             ),
             // ... dynamic helper text based on `_isOrganiser` ...
           ],
         ),
       );

       final toolsCard = _SectionCard(
         title: 'Useful pages',
         child: Column(
           children: [
             _ActionTile(
               icon: Icons.add_circle_outline,
               title: 'Create event',
               subtitle: _isOrganiser ? '...' : 'Locked for attendees...',
               enabled: _isOrganiser,
               onTap: _isOrganiser 
                   ? () => Navigator.pushNamed(context, '/create-event')
                   : () => _showLockedMessage('Create event'),
             ),
             // ... Calendar and My Events tiles ...
           ],
         ),
       );

       return narrow
           ? Column(
               children: [accountCard, const SizedBox(height: 18), roleCard, const SizedBox(height: 18), toolsCard],
             )
           : Row(
               crossAxisAlignment: CrossAxisAlignment.start,
               children: [
                 Expanded(child: accountCard),
                 const SizedBox(width: 18),
                 Expanded(child: roleCard),
                 const SizedBox(width: 18),
                 Expanded(child: toolsCard),
               ],
             );
     },
   )

Directly below the banner sits the dashboard grid. This section manages three 
distinct ``_SectionCard`` components: Account Details, Role Permissions, and Useful 
Pages. 

The layout logic evaluates a ``narrow`` boolean (width < 860px). On wide displays, 
the three cards sit perfectly side-by-side inside a ``Row`` using ``Expanded`` 
widgets. On narrower displays, they stack vertically. Crucially, the ``toolsCard`` 
passes the ``_isOrganiser`` boolean into its ``_ActionTile`` components, visually 
disabling and locking routing for tools the user does not have permission to access.

Session & Account Deletion
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   _SectionCard(
     title: 'Session',
     subtitle: 'Sign out when you are finished using UniSphere.',
     child: Column(
       crossAxisAlignment: CrossAxisAlignment.stretch,
       children: [
         OutlinedButton.icon(
           onPressed: () {
             SqliteBackend().logout();
             Navigator.pushReplacementNamed(context, '/');
           },
           icon: const Icon(Icons.logout),
           label: const Text('Log out'),
           // ... styling ...
         ),
         OutlinedButton.icon(
           onPressed: () async {
             final confirm = await showDialog<bool>(
               context: context,
               builder: (dialogContext) {
                 return AlertDialog(
                   title: const Text('Delete account'),
                   content: const Text('Deleting your account will remove your profile, all events you created...'),
                   actions: [
                     TextButton(onPressed: () => Navigator.pop(dialogContext, false), child: const Text('Cancel')),
                     FilledButton(
                       onPressed: () => Navigator.pop(dialogContext, true),
                       child: const Text('Delete'),
                       // ... red styling ...
                     ),
                   ],
                 );
               },
             );

             if (confirm != true) return;
             final ok = await SqliteBackend().deleteAccount();
             // ... show success/failure snackbar and route to landing page ...
           },
           icon: const Icon(Icons.delete_outline),
           label: const Text('Delete account'),
           // ... red styling ...
         ),
       ],
     ),
   )

The final section handles destructive session actions. The "Log out" button 
clears the current session and routes the user back to the public landing page. 
The "Delete account" button triggers an asynchronous workflow: it first requires 
the user to explicitly accept a warning via an ``AlertDialog``, and if confirmed, 
issues a permanent deletion command to the SQLite database before terminating the 
session.


Custom UI Components
--------------------

The bottom half of the file is dedicated to reusable, stateless visual components. 
Abstracting these elements ensures design consistency across the dashboard and 
significantly reduces boilerplate code in the main layout tree.

_SectionCard & _InfoRow
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _SectionCard extends StatelessWidget {
     final String title;
     final String subtitle;
     final Widget child;

     const _SectionCard({required this.title, required this.subtitle, required this.child});

     @override
     Widget build(BuildContext context) {
       return Container(
         padding: const EdgeInsets.all(22),
         decoration: BoxDecoration(
           color: Colors.white,
           borderRadius: BorderRadius.circular(24),
           border: Border.all(color: const Color(0xFFE5E7EB)),
           boxShadow: [
             BoxShadow(color: Colors.black.withOpacity(0.05), blurRadius: 24, offset: const Offset(0, 10)),
           ],
         ),
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Text(title, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w800, color: Color(0xFF111827))),
             const SizedBox(height: 6),
             Text(subtitle, style: TextStyle(fontSize: 13, color: Colors.grey.shade600, height: 1.45)),
             const SizedBox(height: 18),
             child,
           ],
         ),
       );
     }
   }

   // _InfoRow implementation...

``_SectionCard`` is the foundational wrapper for the dashboard grid. It enforces 
a uniform white container with subtle drop shadows, rounded corners, and standardized 
title/subtitle typography. It accepts a generic ``child`` widget, allowing it to 
house anything from radio toggles to action lists. ``_InfoRow`` is a specific child 
element used within these cards to display key-value pairs (like Email or University) alongside a themed icon.

_ActionTile
^^^^^^^^^^^

.. code-block:: dart

   class _ActionTile extends StatelessWidget {
     final IconData icon;
     final String title;
     final String subtitle;
     final bool enabled;
     final VoidCallback onTap;

     const _ActionTile({
       required this.icon, required this.title, required this.subtitle, required this.onTap, this.enabled = true,
     });

     @override
     Widget build(BuildContext context) {
       return InkWell(
         onTap: enabled ? onTap : null,
         borderRadius: BorderRadius.circular(18),
         child: Container(
           padding: const EdgeInsets.all(14),
           decoration: BoxDecoration(
             color: enabled ? const Color(0xFFF8FAFC) : const Color(0xFFFAFAFA),
             borderRadius: BorderRadius.circular(18),
             border: Border.all(color: enabled ? const Color(0xFFE5E7EB) : const Color(0xFFF0F0F0)),
           ),
           child: Row(
             // ... Icon, Title, and Subtitle rendering ...
             // ... Conditional Lock Icon rendering ...
           ),
         ),
       );
     }
   }

This interactive tile serves as the primary navigation button within the "Useful 
pages" card. It accepts an ``enabled`` boolean flag. If false, it completely 
disables the ``onTap`` ripple effect, "greys out" the border and background colors, 
and renders a small lock icon next to the title to clearly indicate that the user's 
current role restricts access to that route.

_MetaChip, _Pill, & _ReadOnlyField
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class _Pill extends StatelessWidget {
     final String label;
      final Color color;

      const _Pill({required this.label, required this.color});

     @override
     Widget build(BuildContext context) {
       return Container(
         padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
         decoration: BoxDecoration(
           color: color,
           borderRadius: BorderRadius.circular(999),
         ),
         child: Text(
           label,
           style: const TextStyle(
             color: Colors.white,
             fontSize: 12,
             fontWeight: FontWeight.w700,
           ),
         ),
       );
     }
   }

   class _MetaChip extends StatelessWidget {
     final IconData icon;
      final String label;

      const _MetaChip({required this.icon, required this.label});

     @override
     Widget build(BuildContext context) {
       return Container(
         padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 9),
         decoration: BoxDecoration(
           color: Colors.white.withOpacity(0.12),
           borderRadius: BorderRadius.circular(999),
           border: Border.all(color: Colors.white.withOpacity(0.18)),
         ),
         child: Row(
           mainAxisSize: MainAxisSize.min,
           children: [
             Icon(icon, size: 15, color: Colors.white),
             const SizedBox(width: 8),
             Flexible(
               child: Text(
                 label,
                 overflow: TextOverflow.ellipsis,
                 style: const TextStyle(
                   color: Colors.white,
                   fontSize: 13,
                   fontWeight: FontWeight.w600,
                 ),
               ),
             ),
           ],
         ),
       );
     }
   }

   class _ReadOnlyField extends StatelessWidget {
     final String label;
      final String value;

      const _ReadOnlyField({required this.label, required this.value});

     @override
     Widget build(BuildContext context) {
       return Container(
         width: double.infinity,
         padding: const EdgeInsets.all(14),
         decoration: BoxDecoration(
           color: const Color(0xFFF8FAFC),
           borderRadius: BorderRadius.circular(16),
           border: Border.all(color: const Color(0xFFE5E7EB)),
         ),
         child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             Text(
               label,
               style: TextStyle(fontSize: 12, color: Colors.grey.shade500),
             ),
             const SizedBox(height: 4),
             Text(
               value,
               style: const TextStyle(
                 fontSize: 14,
                 fontWeight: FontWeight.w600,
                 color: Color(0xFF111827),
               ),
             ),
           ],
         ),
       );
     }
   }
These are lightweight visual accents. ``_Pill`` and ``_MetaChip`` are designed specifically for the dark gradient Hero Banner, utilizing semi-transparent white backgrounds (``Colors.white.withOpacity(0.12)``) to create contrast without clashing with the underlying colors. ``_ReadOnlyField`` is used exclusively inside the ``_openEditDialog`` modal to safely display fixed account properties (like the user's university) using a distinct grey background, visually separating them from the editable text inputs.