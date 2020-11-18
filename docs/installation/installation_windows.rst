.. _windows_installation:

Windows installation
####################

The instructions for installing the Discovery Server tool in a Windows environment are provided in this page.
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

.. _windows_requirements:

Requirements
************

The installation of *eProsima Fast DDS* in a Windows environment from sources requires the following tools to be
installed in the system:

* :ref:`visual_studio_sw`
* :ref:`chocolatey_sw`
* :ref:`windows_cmake_pip3_wget_git_sw`
* :ref:`windows_python_modules` [optional]

.. _visual_studio_sw:

Visual Studio
=============

`Visual Studio <https://visualstudio.microsoft.com/>`_ is required to
have a C++ compiler in the system. For this purpose, make sure to check the
:code:`Desktop development with C++` option during the Visual Studio installation process.

If Visual Studio is already installed but the Visual C++ Redistributable packages are not,
open Visual Studio and go to :code:`Tools` -> :code:`Get Tools and Features` and in the :code:`Workloads` tab enable
:code:`Desktop development with C++`. Finally, click :code:`Modify` at the bottom right.

.. _chocolatey_sw:

Chocolatey
==========

Chocolatey is a Windows package manager. It is needed to install some of *eProsima Fast DDS*'s dependencies.
Download and install it directly from the `website <https://chocolatey.org/>`_.

.. _windows_cmake_pip3_wget_git_sw:

CMake, pip3, wget and git
==========================

These packages provide the tools required to install the Discovery Server tool, *eProsima Fast DDS* and its
dependencies from command line.
Download and install CMake_, pip3_, wget_ and git_ by following the instructions detailed in the respective websites.
Once installed, add the path to the executables to the :code:`PATH` from the
*Edit the system environment variables* control panel.

.. _windows_python_modules:

Python3 modules
===============

To execute the tests that verify the proper operation of the Discovery Server discovery mechanism, it is necessary
to install some Python3 modules. These can be installed using `pip`.

.. code-block:: bash

    > pip3 install jsondiff==1.2.0 xmltodict==0.12.0

Dependencies
************

*eProsima Fast RTPS* has the following dependencies, when installed from sources in a Windows environment:

* :ref:`asiotinyxml2_sw`
* :ref:`openssl_sw`

.. _asiotinyxml2_sw:

Asio and TinyXML2 libraries
===========================

Asio is a cross-platform C++ library for network and low-level I/O programming, which provides a consistent
asynchronous model.
TinyXML2 is a simple, small and efficient C++ XML parser.
They can be downloaded directly from the links below:

* `Asio <https://github.com/ros2/choco-packages/releases/download/2020-02-24/asio.1.12.1.nupkg>`_
* `TinyXML2 <https://github.com/ros2/choco-packages/releases/download/2020-02-24/tinyxml2.6.0.0.nupkg>`_

After downloading these packages, open an administrative shell with *PowerShell* and execute the following command:

.. code-block:: bash

    > choco install -y -s <PATH_TO_DOWNLOADS> asio tinyxml2

where :code:`<PATH_TO_DOWNLOADS>` is the folder into which the packages have been downloaded.

.. _openssl_sw:

OpenSSL
=======

OpenSSL is a robust toolkit for the TLS and SSL protocols and a general-purpose cryptography library.
Download and install the latest OpenSSL version for Windows at this
`link <https://slproweb.com/products/Win32OpenSSL.html>`_.
After installing, add the environment variable :code:`OPENSSL_ROOT_DIR` pointing to the installation root directory.

For example:

.. code-block:: bash

   > OPENSSL_ROOT_DIR=C:\Program Files\OpenSSL-Win64


.. _colcon_installation_windows:

Installation steps
******************

colcon_ is a command line tool based on CMake_ aimed at building sets of software packages.
This section explains how to use it to compile the Discovery Server tool and its dependencies.

.. important::

    Run colcon within a Visual Studio prompt. To do so, launch a *Developer Command Prompt* from the
    search engine.

#. Install the ROS 2 development tools (colcon_ and vcstool_) by executing the following command:

   .. code-block:: bash

       > pip3 install -U colcon-common-extensions vcstool

   and add the path to the :code:`vcs` executable to the :code:`PATH` from the
   *Edit the system environment variables* control panel.

   .. note::

       If this fails due to an Environment Error, add the :code:`--user` flag to the :code:`pip3` installation command.

#.  Create a Discovery Server workspace and download the repos file that will be used to install the Discovery Server
    tool and its dependencies:

    .. code-block:: bash

        > mkdir discovery-server-ws
        > cd discovery-server-ws
        > mkdir src
        > wget https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos
        > vcs import src < discovery-server.repos

    A
    `discovery-server.repos <https://raw.githubusercontent.com/eProsima/Discovery-Server/master/discovery-server.repos>`__
    file is available in order to profit from `vcstool <https://github.com/dirk-thomas/vcstool>`__
    capabilities to download the needed repositories.

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

        > colcon build --base-paths src \
                --packages-up-to discovery-server \
                --cmake-args -DTHIRDPARTY=ON -DLOG_LEVEL_INFO=ON \
                        -DCOMPILE_EXAMPLES=ON -DINTERNALDEBUG=ON -DCMAKE_BUILD_TYPE=Debug
        > colcon build --base-paths src \
                --packages-up-to discovery-server \
                --cmake-args -DTHIRDPARTY=ON -DCOMPILE_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Release

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

        > HelloWorldExampleDS --help


    to display the example usage instructions.

    In order to test the ``HelloWorldExampleDS`` open three consoles and run the above command.
    Then run the following command in each console:

    -   Console 1:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS publisher

    -   Console 2:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS subscriber

    -   Console 3:

        .. code-block:: bash

            > cd <path/to/discovery-server-ws>/discovery-server-ws/install/discovery-server/examples/HelloWorldExampleDS
            > HelloWorldExampleDS server

.. External links

.. _colcon: https://colcon.readthedocs.io/en/released/
.. _CMake: https://cmake.org
.. _pip3: https://docs.python.org/3/installing/index.html
.. _wget: https://www.gnu.org/software/wget/
.. _git: https://git-scm.com/
.. _OpenSSL: https://www.openssl.org/
.. _Gtest: https://github.com/google/googletest
.. _vcstool: https://pypi.org/project/vcstool/
