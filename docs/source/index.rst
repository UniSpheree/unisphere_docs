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
  :caption: Front end
  :maxdepth: 1

   landing_page.dart <landing_page>
   landing_page_logged_in.dart <landing_page_logged_in>
   discover_event_screen.dart <discover_event_screen>
   event_details_screen.dart <event_details_screen>
   register_screen.dart <register_screen>
   login_screen.dart <login_screen>
   main.dart <main>
   about,terms & policy <about_terms_policy>

.. toctree::
  :caption: About & Policies
  :maxdepth: 1

   about_us_page.dart <about_us_page>
   terms_page.dart <terms_page>
   privacy_page.dart <privacy_page>

.. toctree::
  :caption: Widgets
  :maxdepth: 1

   header.dart <Header>
   auth_header.dart <auth_header>
   app_footer.dart <app_footer>

