event_details_screen.dart
=========================

``event_details_screen.dart`` is the detailed event view. It renders the
event header (title, date, location, tags), the descriptive content, and the
purchase/action area. The page adapts its call-to-action depending on whether
the current user is the organiser, a guest, or already owns the ticket.

Imports
-------

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:unisphere_app/utils/mock_backend.dart';
    import 'my_tickets_screen.dart';
    import 'register_screen.dart';

The file uses Flutter's material widgets, the `MockBackend` helper for demo
state, and navigation targets for tickets and registration flows.

Main widget
-----------

.. code-block:: dart

    class EventDetailsScreen extends StatelessWidget {
       final Map<String, dynamic> event;
       final bool allowPurchase;

       const EventDetailsScreen({
          super.key,
          required this.event,
          this.allowPurchase = true,
       });
    }

`EventDetailsScreen` is a `StatelessWidget` that accepts a required `event`
map and an optional `allowPurchase` flag. The `event` map is expected to
contain fields such as `title`, `date`, `location`, `category`, `tags`,
`organizerEmail`, `organizer`, `capacity`, `price`, and `color`.

Field extraction and helpers
----------------------------

.. code-block:: dart

    final color = event['color'] as Color? ?? const Color(0xFF4F46E5);
    final description = event['description'] as String? ?? 'No extra description provided for this event.';
    final organizer = event['organizer'] as String? ?? 'Organizer not specified';
    final capacity = event['capacity'] != null ? '${event['capacity']}' : null;
    final tags = (event['tags'] as List<dynamic>?)?.cast<String>() ?? [];
    final price = (event['price'] as String?)?.trim();

The widget reads event metadata and provides sensible defaults when fields
are missing. The `color` token is used to theme badges and the purchase CTA.

Determining viewer role
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    final isOrganizerViewing = MockBackend().currentUser?.email == (event['organizerEmail']?.toString() ?? '');

This boolean is used to branch the action area: organisers see a passive
informational bar, attendees see purchase controls, and guests are prompted to
register before completing a purchase.

Scaffold and app bar
--------------------

.. code-block:: dart

    return Scaffold(
       backgroundColor: const Color(0xFFF0F2F8),
       appBar: AppBar(
          backgroundColor: Colors.white,
          elevation: 0,
          leading: IconButton(onPressed: () => Navigator.pop(context), icon: const Icon(Icons.arrow_back_rounded)),
          title: Text(event['title'] as String),
       ),
       body: SafeArea(...),
    );

The page uses a white app bar with a back button and the event title as the
primary header.

Header card
-----------

.. code-block:: dart

    Container(
       padding: const EdgeInsets.all(24),
       decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(12)),
       child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
             Text(event['title'] as String, style: const TextStyle(fontSize: 26, fontWeight: FontWeight.w900)),
             Row(... date, location ...),
             Wrap(... category badge and tags ...),
             Row(... organizer and capacity ...),
          ],
       ),
    )

The header card shows the title, date, location, category badge, optional
tags, organiser name, and optionally the event capacity.

Description section
-------------------

.. code-block:: dart

    Container(
       padding: const EdgeInsets.all(20),
       decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(12)),
       child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
             const Text('About this event', style: TextStyle(fontSize: 18, fontWeight: FontWeight.w700)),
             const SizedBox(height: 8),
             Text(description),
          ],
       ),
    )

This section contains the long-form description. The widget substitutes a
fallback string if no description is available.

Action area and purchase flow
-----------------------------

.. code-block:: dart

    if (allowPurchase && !isOrganizerViewing) {
       Row(
          children: [
             if (price != null && price.isNotEmpty) Text(price, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.w900)),
             FilledButton(onPressed: () { ... purchase logic ... }, child: const Text('Buy ticket')),
          ],
       )
    } else if (isOrganizerViewing) {
       Container(... 'This is your event — buyers cannot purchase from here.' ...)
    } else {
       Container(... 'This ticket is already in your tickets list.' ...)
    }

When the purchase button is pressed the implementation does the following:

- If no user is signed in, it creates a `PurchasedTicket` object, sets it as
   a pending purchase on `MockBackend()`, shows a `SnackBar` informing the user
   the ticket was saved, and routes to `RegisterScreen`.
- If a user is signed in, it calls `MockBackend().purchaseTicket(...)`, shows
   a success `SnackBar`, and navigates to `MyTicketsScreen`.

Note: the documentation references the `PurchasedTicket` helper type used by
the mock backend to persist ticket objects.

Organizer and owned-ticket states
----------------------------------

.. code-block:: dart

    else if (isOrganizerViewing) => show organiser info container
    else => show owned-ticket container

Organisers see an informational container explaining they cannot buy from the
public view. If the ticket is already in the user's ticket list, the page
shows a confirmation container instead of the buy CTA.

Navigation targets
------------------

.. code-block:: dart

    Navigator.push(context, MaterialPageRoute(builder: (_) => const RegisterScreen()));
    Navigator.push(context, MaterialPageRoute(builder: (_) => const MyTicketsScreen()));

These routes are used by the purchase flow to either direct guests to register
or take purchasers to their ticket library after checkout.
