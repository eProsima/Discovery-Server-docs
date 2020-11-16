Installation
############

The instructions for installing the Discovery Server tool are provided in this page.

-   :ref:`Dependencies`
-   :ref:`Installation steps`

    -   :ref:`Linux`
    -   :ref:`Windows`

Dependencies
************

In order to use the Discovery Server tool, its necessary to have a compatible version of
`eProsima Fast DDS <https://eprosima-fast-rtps.readthedocs.io/en/latest/>`__ installed (over release 2.0.2).

*Fast DDS* dependencies as tinyxml must be accessible, either because *Fast DDS* was build-installed defining
THIRDPARTY=ON or because those libraries have been specifically installed.
The well known cross-platform tool `colcon <https://colcon.readthedocs.io/en/released/>`__ was chosen to simplify the
installation of the several mutually dependent `CMake <https://cmake.org/cmake/help/latest/>`__ projects.
In order to use colcon, `Python3 <https://www.python.org/>`__ and `CMake <https://cmake.org/cmake/help/latest/>`__
must be first installed.

A `discovery-server.repos <https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos>`__
file is available in order to profit from `vcstool <https://github.com/dirk-thomas/vcstool>`__ capabilities to download
the needed repositories.

.. note::

    In order to avoid using vcstool the following repositories should be downloaded from github into
    the `discovery-server-ws/src` directory:

+------------------------------------+---------------------------------------------------------+-------------+
| PACKAGE                            | URL                                                     | BRANCH      |
+====================================+=========================================================+=============+
| eProsima/Fast-CDR                  | https://github.com/eProsima/Fast-CDR.git                | master      |
+------------------------------------+---------------------------------------------------------+-------------+
| eProsima/Fast-RTPS                 | https://github.com/eProsima/Fast-RTPS.git               | master      |
+------------------------------------+---------------------------------------------------------+-------------+
| eProsima/Discovery-Server          | https://github.com/eProsima/Discovery-Server.git        | master      |
+------------------------------------+---------------------------------------------------------+-------------+
| eProsima/foonathan_memory_vendor   | https://github.com/eProsima/foonathan_memory_vendor.git | master      |
+------------------------------------+---------------------------------------------------------+-------------+
| leethomason/tinyxml2               | https://github.com/leethomason/tinyxml2.git             | master      |
+------------------------------------+---------------------------------------------------------+-------------+

Installation steps
******************

Linux
-----

#.  Create a Fast-DDS directory and download the repos file that will be used to install eProsima Fast DDS and its
    dependencies:

    .. code-block:: bash

        $ mkdir -p discovery-server-ws/src && cd discovery-server-ws
        $ wget https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos
        $ vcs import src < discovery-server.repos

#.  Finally, use colcon to compile all software.
    Choose the build configuration by declaring ``CMAKE_BUILD_TYPE`` as Debug or Release.
    For this example, the Debug option has been choosen, which would be the choice of advanced users for debugging
    purposes.

    .. code-block:: bash

        $ colcon build --base-paths src
                \ --packages-up-to discovery-server
                \ --cmake-args -DTHIRDPARTY=ON -DLOG_LEVEL_INFO=ON -DCOMPILE_EXAMPLES=ON -DINTERNALDEBUG=ON -DCMAKE_BUILD_TYPE=Debug

#.  If you installed the Discovery Server tool following the steps outlined above, you can try the
    ``HelloWorldExampleDS``.
    To run the example navigate to the following directory

    ``<path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS``

    and run

    .. code-block:: bash

        $ ./HelloWorldExampleDS --help


    to display the example usage instructions.

    In order to test the ``HelloWorldExampleDS`` open three terminals and run the above command.
    Then run the following command in each terminal:

    -   Terminal 1:

        .. code-block:: bash

            $ cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            $ ./HelloWorldExampleDS publisher

    -   Terminal 2:

        .. code-block:: bash

            $ cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            $ ./HelloWorldExampleDS subscriber

    -   Terminal 3:

        .. code-block:: bash

            $ cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            $ ./HelloWorldExampleDS server


Windows
-------

#.  Create a *Fast DDS* directory and download the repos file that will be used to install *eProsima Fast DDS* and its
    dependencies:

    .. code-block:: bash

        > mkdir discovery-server-ws
        > cd discovery-server-ws
        > mkdir src
        > wget https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos
        > vcs import src < discovery-server.repos


#.  If the generator (compiler) of choice is Visual Studio, launch colcon from a visual studio console.
    Any console can be setup into a visual studio one by executing a batch file.
    For example, in VS2017 is usually
    :code:`C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat`.

#.  Finally, use colcon to compile all software.
    Choose the build configuration by declaring ``CMAKE_BUILD_TYPE`` as Debug or Release.
    For this example, the Debug option has been choosen, which would be the choice of advanced users for debugging
    purposes.
    If using a multi-configuration generator like Visual Studio we recommend to build both in debug and release modes

    .. code-block:: bash

        > colcon build --base-paths src
                \ --packages-up-to discovery-server
                \ --cmake-args -DTHIRDPARTY=ON -DLOG_LEVEL_INFO=ON -DCOMPILE_EXAMPLES=ON -DINTERNALDEBUG=ON -DCMAKE_BUILD_TYPE=Debug
        > colcon build --base-paths src
                \ --packages-up-to discovery-server
                \ --cmake-args -DTHIRDPARTY=ON -DCOMPILE_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Release

#.  If you installed the Discovery Server tool following the steps outlined above, you can try the `HelloWorldExampleDS`.
    To run the example navigate to the following directory

    ``<path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS``

    and run

    .. code-block:: bash

        > HelloWorldExampleDS --help


    to display the example usage instructions.

    In order to test the ``HelloWorldExampleDS`` open three terminals and run the above command.
    Then run the following command in each terminal:

    -   Terminal 1:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS publisher

    -   Terminal 2:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS subscriber

    -   Terminal 3:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS server
