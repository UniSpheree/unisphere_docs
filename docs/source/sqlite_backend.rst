===================
sqlite_backend.dart
===================

``sqlite_backend.dart`` serves as a legacy compatibility layer (or proxy export) 
for the application's backend architecture.

.. code-block:: dart

   export 'api_backend.dart';

During the development lifecycle, the application migrated from a local SQLite 
database to a live REST API. Rather than updating the ``import`` statements across 
dozens of frontend UI screens, this file was retained to simply re-export the new 
``api_backend.dart`` file. 

As a result, any screen importing ``sqlite_backend.dart`` is seamlessly routed to 
the active HTTP client.

For the actual backend implementation, network logic, and state management, please 
refer to the :doc:`api_backend` documentation.