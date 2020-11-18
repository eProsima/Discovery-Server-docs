.. _linux_installation:

Linux installation
##################

The instructions for installing the Discovery Server tool in a Linux environment are provided in this page.
In order to use the Discovery Server tool, its necessary to have a compatible version of
`eProsima Fast DDS <https://eprosima-fast-rtps.readthedocs.io/en/latest/>`__ installed (over release 2.0.2).

*eProsima Fast DDS* dependencies as tinyxml must be accessible, either because *Fast DDS* was build-installed defining
THIRDPARTY=ON or because those libraries have been specifically installed.
The cross-platform tool `colcon <https://colcon.readthedocs.io/en/released/>`__ was chosen to simplify the
installation of the several mutually dependent `CMake <https://cmake.org/cmake/help/latest/>`__ projects.
In order to use colcon, `Python3 <https://www.python.org/>`__ and `CMake <https://cmake.org/cmake/help/latest/>`__
must be first installed.


.. contents::
    :local:
    :backlinks: none
    :depth: 2

.. _linux_requirements:

Requirements
************

The installation of the Discovery Server tool in a Linux environment from sources requires the following tools to be
installed in the system:

* :ref:`linux_cmake_gcc_pip3_wget_git_sl`
* :ref:`linux_python_modules` [optional]

.. _linux_cmake_gcc_pip3_wget_git_sl:

CMake, g++, pip3, wget and git
==============================

These packages provide the tools required to install the Discovery Server tool and its dependencies from command line.
Install CMake_, `g++ <https://gcc.gnu.org/>`_, pip3_, wget_ and git_ using the package manager of the appropriate
Linux distribution. For example, on Ubuntu use the command:

.. code-block:: bash

    sudo apt install cmake g++ python3-pip wget git

.. _linux_python_modules:

Python3 modules
===============

To execute the tests that verify the proper operation of the Discovery Server discovery mechanism, it is necessary
to install some Python3 modules. These can be installed using `pip`.

.. code-block:: bash

    pip3 install jsondiff==1.2.0 xmltodict==0.12.0

.. _dependencies_sl:

Dependencies
************

The Discovery Server tool and *eProsima Fast DDS* has the following dependencies, when installed from binaries in a
Linux environment:

* :ref:`asiotinyxml2_sl`
* :ref:`openssl_sl`

.. _asiotinyxml2_sl:

Asio and TinyXML2 libraries
===========================

Asio is a cross-platform C++ library for network and low-level I/O programming, which provides a consistent
asynchronous model.
TinyXML2 is a simple, small and efficient C++ XML parser.
Install these libraries using the package manager of the appropriate Linux distribution.
For example, on Ubuntu use the command:

.. code-block:: bash

    sudo apt install libasio-dev libtinyxml2-dev

.. _openssl_sl:

OpenSSL
=======

OpenSSL is a robust toolkit for the TLS and SSL protocols and a general-purpose cryptography library.
Install OpenSSL_ using the package manager of the appropriate Linux distribution.
For example, on Ubuntu use the command:

.. code-block:: bash

   sudo apt install libssl-dev


.. _colcon_installation_linux:

Installation steps
******************

colcon_ is a command line tool based on CMake_ aimed at building sets of software packages.
This section explains how to use it to compile the Discovery Server tool and its dependencies.

#. Install the ROS 2 development tools (colcon_ and vcstool_) by executing the following command:

   .. code-block:: bash

       pip3 install -U colcon-common-extensions vcstool

   .. note::

       If this fails due to an Environment Error, add the :code:`--user` flag to the :code:`pip3` installation command.

#.  Create a Discovery Server workspace and download the repos file that will be used to install the Discovery Server
    tool and its dependencies:

    .. code-block:: bash

        $ mkdir -p discovery-server-ws/src && cd discovery-server-ws
        $ wget https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos
        $ vcs import src < discovery-server.repos

    The
    `discovery-server.repos <https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos>`__
    file is provided in order to profit from `vcstool <https://github.com/dirk-thomas/vcstool>`__ capabilities
    to download the needed repositories.

    .. note::

        In order to avoid using vcstool the following repositories should be downloaded from Github into
        the ``discovery-server-ws/src`` directory:

        +------------------------------------+-----------------------------------------------------------+-------------+
        | PACKAGE                            | URL                                                       | BRANCH      |
        +====================================+===========================================================+=============+
        | eProsima/Fast-CDR                  | https://github.com/eProsima/Fast-CDR.git                  | master      |
        +------------------------------------+-----------------------------------------------------------+-------------+
        | eProsima/Fast-RTPS                 | https://github.com/eProsima/Fast-RTPS.git                 | master      |
        +------------------------------------+-----------------------------------------------------------+-------------+
        | eProsima/Discovery-Server          | https://github.com/eProsima/Discovery-Server.git          | master      |
        +------------------------------------+-----------------------------------------------------------+-------------+
        | eProsima/foonathan_memory_vendor   | https://github.com/eProsima/foonathan_memory_vendor.git   | master      |
        +------------------------------------+-----------------------------------------------------------+-------------+
        | leethomason/tinyxml2               | https://github.com/leethomason/tinyxml2.git               | master      |
        +------------------------------------+-----------------------------------------------------------+-------------+

#.  Finally, use colcon to compile all software.
    Choose the build configuration by declaring ``CMAKE_BUILD_TYPE`` as Debug or Release.
    For this example, the Debug option has been choosen, which would be the choice of advanced users for debugging
    purposes.

    .. code-block:: bash

        $ colcon build --base-paths src \
                --packages-up-to discovery-server \
                --cmake-args -DTHIRDPARTY=ON -DLOG_LEVEL_INFO=ON \
                        -DCOMPILE_EXAMPLES=ON -DINTERNALDEBUG=ON -DCMAKE_BUILD_TYPE=Debug

.. note::

    Being based on CMake_, it is possible to pass the CMake configuration options to the :code:`colcon build`
    command. For more information on the specific syntax, please refer to the
    `CMake specific arguments <https://colcon.readthedocs.io/en/released/reference/verb/build.html#cmake-specific-arguments>`_
    page of the colcon_ manual.


Run an application
******************

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

.. External links

.. _colcon: https://colcon.readthedocs.io/en/released/
.. _CMake: https://cmake.org
.. _pip3: https://docs.python.org/3/installing/index.html
.. _wget: https://www.gnu.org/software/wget/
.. _git: https://git-scm.com/
.. _OpenSSL: https://www.openssl.org/
.. _Gtest: https://github.com/google/googletest
.. _vcstool: https://pypi.org/project/vcstool/
