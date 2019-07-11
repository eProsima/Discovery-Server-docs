Installation
############

**Table of Contents**

* :ref:`Dependencies`
* :ref:`Installation steps`
* :ref:`Linux setup steps`
* :ref:`Windows setup steps`

Dependencies
************

In order to use discovery server, its necessary have a compatible version of `Fast RTPS`_ installed (over release 1.9.0).
Fast RTPS dependencies as tinyxml_ must be accessible, either because Fast RTPS was build-installed defining
THIRDPARTY=ON or because those libraries have been specifically installed.

.. in the future we may need to reference OpenSSH when security layer is implemented for PDPClient and PDPServer.

The well known cross-platform tool colcon_ was chosen to simplify the installation of the several mutually dependent
CMake_ projects. In order to use colcon_,  python_ and CMake_ must be already installed as detailed in the corresponding
hyperlinks.

A discovery-server.repos_ file is available in order to profit from vcstool capabilities to download the needed
repositories. Once vcstool python package is installed download the sources is as easy as download the
discovery-server.repos_ and call:

.. code-block:: bash

    [SOURCES]$ vcs import --input discovery-server.repos

on the **[SOURCES]** directory where the user wants to keep the repositories.

.. _discovery-server.repos: https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos

In order to avoid using vcstool the following repositories should be downloaded from github into **SOURCES**:

+----------------------------+--------------------------------------------------+
| eProsima/Fast-CDR:         | https://github.com/eProsima/Fast-CDR.git         |
+----------------------------+--------------------------------------------------+
| eProsima/Fast-RTPS:        | https://github.com/eProsima/Fast-RTPS.git        |
+----------------------------+--------------------------------------------------+
| eProsima/Discovery-Server: | https://github.com/eProsima/Discovery-Server.git |
+----------------------------+--------------------------------------------------+
| leethomason/tinyxml2:      | https://github.com/leethomason/tinyxml2.git      |
+----------------------------+--------------------------------------------------+


We also assume that the user wants to keep the build, log and installation files in a separate directory called
**[BUILD]**. If this is not the case, flag **--base-paths [SOURCES]** can be ignored in what follows.

.. _`Fast RTPS`: https://eprosima-Fast-RTPS.readthedocs.io/en/latest/
.. _colcon: https://colcon.readthedocs.io/en/released/
.. _CMake: https://cmake.org/cmake/help/latest/
.. _python: https://www.python.org/
.. _tinyxml: https://github.com/leethomason/tinyxml2.git

Installation steps
******************

Discovery Server supports the same platforms supported by Fast-RTPS: Windows, Linux and Mac. We proceed to detail each
platform specifics.

Linux setup steps
-----------------

Valid placeholders for the Linux example may be:

+---------------+----------------------------------------+
| PLACEHOLDER   |             EXAMPLE PATHS              |
+===============+========================================+
|[SOURCES]      | /home/username/Documents/colcon_sources|
+---------------+----------------------------------------+
|[BUILD]        |/home/username/Documents/colcon_build   |
+---------------+----------------------------------------+

1. Create directory **[BUILD]** where we want to keep the build, install and log compilation results.

2. Compile using the colcon tool. Choose the build configuration by declaring CMAKE_BUILD_TYPE as Debug or Release.
   In this example we have chosen Debug which would be the choice of advance users for debugging purposes:

.. code-block:: bash

    [BUILD]$ colcon build --base-paths [SOURCES] --packages-up-to discovery-server --cmake-args -DCOMPILE_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Debug

3. In order to run the tests use the following command:

.. code-block:: bash

    [BUILD]$ colcon test --base-paths [SOURCES] --packages-select discovery-server

note that only the test matching the build (step 2) configuration would run.

4. To run the example navigate to directory **[BUILD]**/install/discovery-server/examples/HelloWorldExampleDS/bin.
   The configuration bash file located in the install folder must be run first in order to set the required
   environment variables:

.. code-block:: bash

    [BUILD]/install/discovery-server/examples/C++/HelloWorldExampleDS/bin$ . ../../../../../local_setup.bash

In order to test the `example <HelloWorldExample.html#example-application>`_ open three terminals and run the above
command. Then launch the application with different arguments:

.. code-block:: bash

    [BUILD]/install/discovery-server/examples/HelloWorldExampleDS/bin$ ./HelloWorldExampleDS publisher
    [BUILD]/install/discovery-server/examples/HelloWorldExampleDS/bin$ ./HelloWorldExampleDS subscriber
    [BUILD]/install/discovery-server/examples/HelloWorldExampleDS/bin$ ./HelloWorldExampleDS server

Windows setup steps
-------------------

Valid placeholders for the windows example may be:

+---------------+------------------------------------------------+
| PLACEHOLDER   |             EXAMPLE PATHS                      |
+===============+================================================+
|\[SOURCES\]    |  C:\\Users\\username\\Documents\\colcon_sources|
+---------------+------------------------------------------------+
|\[BUILD\]      | C:\\Users\\username\\Documents\\colcon_build   |
+---------------+------------------------------------------------+

1. Create directory **[BUILD]** where you want to keep the build, install and log compilation results.

2. If the generator (compiler) of choice is Visual Studio, launch colcon from a visual studio console. Any console
   can be set up into a visual studio one by executing a batch file. For example in VS2017 is usually:

.. code-block:: text

   C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat

3. Compile using the colcon tool. If using a multi-configuration generator like Visual Studio we recommend to
   build both in debug and release modes:

.. code-block:: bat

    [BUILD]> colcon build --base-paths [SOURCES] --packages-up-to discovery-server --cmake-args -DCOMPILE_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Debug
    [BUILD]> colcon build --base-paths [SOURCES] --packages-up-to discovery-server --cmake-args -DCOMPILE_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Release

If using a single configuration tool just make the above call with your configuration of choice.

4. In order to run the tests in a multi-configuration generator like Visual Studio use the following command:

.. code-block:: bat

    [BUILD]> colcon test --base-paths [SOURCES] --packages-select discovery-server --ctest-args -C Debug

Here, --ctest-args allows specifying the configuration (Debug or Release) of interest (names are case sensitive).
If using a single configuration tool this flag has no effect, as only the test matching the build (step 3)
configuration would run.

5. In order to run the example, navigate to the directory
   **[BUILD]**\\install\\discovery-server\\examples\\HelloWorldExampleDS\\bin and run the executable,
   running first the configuration bat file located within the install folder in order to set required environment
   variables:

.. code-block:: bat

    [BUILD]\install\discovery-server\examples\C++\HelloWorldExampleDS\bin>..\..\..\..\..\local_setup.bat

To test the helloworld example_ open three consoles, run the above bat file and launch the application with different
arguments:

.. code-block:: bat

    [BUILD]\install\discovery-server\examples\C++\HelloWorldExampleDS\bin> HelloWorldExampleDS publisher
    [BUILD]\install\discovery-server\examples\C++\HelloWorldExampleDS\bin> HelloWorldExampleDS subscriber
    [BUILD]\install\discovery-server\examples\C++\HelloWorldExampleDS\bin> HelloWorldExampleDS server



