C++ example application
#######################

The *eProsima Fast DDS* *HelloWorldExample* has been updated to illustrate the Discovery Server functionality.
Its installation details are explained in the `installation section <installation.html>`_.
Basically, the DDS DomainParticipants are now *Clients* and can only discover each other when a *Server* participant
is created.

As usual, publishers and subscribers are launched by running the *HelloWorldExampleDS* executable with the
corresponding *publisher* or *subscriber* argument.
Each publisher and subscriber is launched within its own participant, but now the
:func:`HelloWorldPublisher::init()` and :func:`HelloWorldSubscriber::init()` functions are modified to create clients
and add the server address specified by command line (see :ref:`lan_testing_helloworldexampleds`).

HelloWorldExample command line syntax
**************************************

The environmental variables must be appropriately set up as explained in the
:ref:`linux_installation` and :ref:`windows_installation` by employing a colcon generated script file.
For colcon builds the relative path to the script from the example directory would be:

*   **Linux**

    .. code-block:: bash

        $ . ../../../../../local_setup.bash

*   **Windows**

    .. code-block:: bash

        > ..\..\..\..\..\local_setup.bat

Otherwise, modify the console ``PATH`` or the ``LIB_PATH_DIR`` environmental variables to allow the example binary to
locate *Fast DDS* shared libraries.

The command-line syntax is the usual one for the *HelloWorldExample*, although a new flag ``-t`` or ``--tcp`` is
introduced to enforce the use of TCP transport:

*   **Linux**

    .. code-block:: bash

        $ ./HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>]  | -l ip[:port] ]

*   **Windows**

    .. code-block:: bat

        > HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>] | -l ip[:port] ]

+--------------------+----------------------+--------------------------------------------------------------------------+
| SHORTCUT           |       FLAG           |                         MEANING                                          |
+====================+======================+==========================================================================+
| ``-h``             | ``--help``           | Produce help message                                                     |
+--------------------+----------------------+--------------------------------------------------------------------------+
| ``-t``             | ``--tcp``            | Use TCP transport instead of the default UDP one                         |
+--------------------+----------------------+--------------------------------------------------------------------------+
| ``-c <num>``       | ``--count=<num>``    | Number of datagrams to send (0=infinite) defaults to 10                  |
+--------------------+----------------------+--------------------------------------------------------------------------+
| ``-i <num>``       | ``--Interval=<num>`` | Time between samples in milliseconds defaults to 100                     |
+--------------------+----------------------+--------------------------------------------------------------------------+
| ``-l <ip[:port]>`` | ``--ip=<ip[:port]>`` | Server address and physical port                                         |
+--------------------+----------------------+--------------------------------------------------------------------------+

Additionally to the Publisher and Subscriber instances, a Server participant must be launched in order to allow
publishers and subscribers to discover each other.
A simple test would be as follows:

*   **Linux**

    *   Terminal 1:

        .. code-block:: bash

            $ ./HelloWorldExampleDS publisher

    *   Terminal 2:

        .. code-block:: bash

            $ ./HelloWorldExampleDS subscriber

    *   Terminal 3:

        .. code-block:: bash

            $ ./HelloWorldExampleDS server

*   **Windows**

    -   Console 1:

        .. code-block:: bash

            > HelloWorldExampleDS publisher

    -   Console 2:

        .. code-block:: bash

            > HelloWorldExampleDS subscriber

    -   Console 3:

        .. code-block:: bash

            > HelloWorldExampleDS server

The HelloWorldExampleDS Server instance can be replaced by a Discovery Server instance that creates a suitable Server.
Thus instead of running :code:`HelloWorldExampleDS server`, it can be done running the following commands:

*   **Linux**

    .. code-block:: bash

        $ ./discovery-server-X.X.X(d) config-file.xml

*   **Windows**

    .. code-block:: bat

        > discovery-server-X.X.X(d).exe config-file.xml

being the ``config-file.xml``,

.. literalinclude:: ../02-resources/examples/xml/config_file_server.xml
    :linenos:
    :language: xml
    :start-after:	<profiles>
    :end-before:	</profiles>

.. _lan_testing_helloworldexampleds:

LAN testing using HelloWorldExampleDS
-------------------------------------

First, the Server network address and its physical port must be known.
In this example it would be `192.168.1.113:64863`.
The UDP protocol is used as the default transport protocol, but it is possible to change it to the TCP protocol by
adding the `--tcp` flag to the following commands:

*   **Linux**

    -   Terminal 1:

        .. code-block:: bash

            $ . ../../../../../local_setup.bash
            $ ./HelloWorldExampleDS publisher --count=0 --ip=192.168.1.113:64863

        by specifying ``--count=0`` the publisher keeps publishing samples forever.

    -   Terminal 2:

        .. code-block:: bash

            $ . ../../../../../local_setup.bash
            $ ./HelloWorldExampleDS subscriber --ip=192.168.1.113:64863

    -   Terminal 3:

        .. code-block:: bash

            $ . ../../../../../local_setup.bash
            $ ./HelloWorldExampleDS server --ip=0.0.0.0:64863

*   **Windows**

    -   Console 1:

        .. code-block:: bat

            > ..\..\..\..\..\local_setup.bat
            > HelloWorldExampleDS publisher --count=0 --ip=192.168.1.113:64863

        by specifying ``--count=0`` the publisher keeps publishing samples forever.

    -   Console 2:

        .. code-block:: bat

            > ..\..\..\..\..\local_setup.bat
            > HelloWorldExampleDS subscriber --ip=192.168.1.113:64863

    -   Console 3:

        .. code-block:: bat

            > ..\..\..\..\..\local_setup.bat
            > HelloWorldExampleDS server --ip=0.0.0.0:64863

Note that by using ``0.0.0.0`` as IP address, the server is forced to publish its metatraffic information
through all the local interfaces (``192.168.1.133`` would be one of then in this example).
The clients, once received the server metadata, would choose the fastest interface among the server's interfaces.
Of course, the server can be configured to use single interface by doing
``--ip=192.168.1.133:64863``.
Finally, specifying the localhost network address as the interface (``--ip=127.0.0.1:64863``) only local clients
will be able to reach the server.

.. note::

    If no port number is provided a default one will used (``11811``).
