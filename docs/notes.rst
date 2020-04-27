Version 1.2.0
*************

* xml schema and sources has been updated to DDS API (1.10)

Version 1.1.0
#############

* `CMakeLists.txt` has been modified to be able to:

    - load tinyxml2 library in whatever form it's present: this library may be provided with CMake export or only
      sources. CMake export may include namespace for target names or not.
    - allow handling a larger amount of tests.

* Now the database keeps timestamps for each discovery callback (PDP and EDP) in order to measure discovery mechanism
  performance.
* Publisher liveliness callbacks are recorded in the database. A new test is added to check Publisher liveliness works
  well under client-server discovery.
* Discovery server (schema and binaries) has been extended to allow create simple discovery participants. This eases
  performance comparison between simple and client-server mechanism. This led to solve and ABBA deadlock on startup that
  remain unnoticed. A new simple discovery test has been added.
* When running discovery-server on several computers to run a simulation:

    - the snapshot output can be merged into one single snapshot (besides the validation). This simplifies statistical
      analysis of the tests.
    - a delay to shutdown after the last snapshot has been introduced to account for startup delays in the different computers. Startup
      synchronization in our test-bench is done externally setting an active wait for a given time to launch the discovery-server
      binary.

* Lease duration announcement test has been updated to use a more reliable mechanism that parses the xml snapshot
  files.
* A new backup test has been added.

Version 1.0.0
#############

First release.

* server creation and setup capabilities.
* discovery testing capabilities.
* single process suite of tests added.
* multiprocess-machine test capability added (generation and validation of snapshots serialized as XML).
* Hello World example over UDP and TCP.
* extended schema that simplifies setup for server creation and testing purposes.
