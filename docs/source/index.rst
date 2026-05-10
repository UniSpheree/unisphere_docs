Welcome to Unisphere's documentation!
=====================================

**Unisphere** is a event-discovery website for university students,
aimed towards students who are looking for a smoother experience when booking events.

Check out the :doc:`usage` section for further information, including
how to :ref:`installation` the project.

.. note::

   This project is under active development.

Dependencies
------------
These are the expected dependencies listed within ``pubspec.yaml``:

.. code-block:: yaml

   environment:
     sdk: ^3.9.2

   dependencies:
     flutter:
       sdk: flutter
     cupertino_icons: ^1.0.8
     image_picker: ^1.0.7
     flutter_map: ^7.0.2
     latlong2: ^0.9.1


Contents
--------
.. toctree::
  :maxdepth: 1

  installation

.. toctree::
  :caption: Front End
  :maxdepth: 1

   landing_page.dart <landing_page>
   landing_page_logged_in.dart <landing_page_logged_in>
   create_event_screen.dart <create_event_screen>
   discover_event_screen.dart <discover_event_screen>
   event_details_screen.dart <event_details_screen>
   profile_page.dart <profile_page>
   my_tickets_screen.dart <my_tickets_screen>
   my_events_page.dart <my_events_page>
   calendar_page.dart <calendar_page>
   register_screen.dart <register_screen>
   login_screen.dart <login_screen>
   forgot_password_screen.dart <forgot_password_screen>

.. toctree::
  :caption: Back End
  :maxdepth: 1

   main.dart <main>
   api_backend.dart <api_backend>
   sqlite_backend.dart <sqlite_backend>






