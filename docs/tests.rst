Testing
#######

**Table of Contents**

* :ref:`test_01_PDP_UDP.xml`
* :ref:`test_02_PDP_TCP.xml`
* :ref:`test_03_PDP_UDP.xml`
* :ref:`test_04_PDP_UDP.xml`
* :ref:`test_05_EDP_UDP.xml & test_05_EDP_TCP.xml`
* :ref:`test_06_EDP_UDP.xml`
* :ref:`test_07_PDP_UDP.xml`
* :ref:`test_08_lease_client.xml & test_08_lease_server.xml`
* :ref:`test_09_WLP_client.xml & test_09_WLP_server.xml`
* :ref:`test_10_simple_PDP_EDP.xml`
* :ref:`test_11_BACKUP_client.xml & test_11_BACKUP_server.xml`

Discovery testing is done resorting to discover-server configuration files. Each test uses a specific XML file that
tests a single or several discovery features. To automatically launch the tests using colcon, please check section
`installation <installation.html#installation-steps>`_. To manually launch a test, use the following procedure:

Linux:

.. code-block:: bash

	[BUILD]/install/discovery-server/bin$ . ../../local_setup.bash
	[BUILD]/install/discovery-server/bin$ ./discovery-server-X.Y.Z(d)
	[SOURCES]/discovery-server/resources/xml/test_XXX.xml

Windows:

.. code-block:: bat

	[BUILD]\install\discovery-server\bin>..\..\local_setup.bat
	[BUILD]\install\discovery-server\bin>discovery-server-X.Y.Z(d) [SOURCES]\discovery-server\resources\xml\test_XXX.xml

To view the full discovery information messages and snapshots in debug configuration, run colcon with the additional
flag `-DLOG_LEVEL_INFO=ON`.

A brief description of each test is given below. Note that a detailed explanation of the XML syntax is given in section
`configuration files <xml_schemas.html>`_.

test_01_PDP_UDP.xml
*******************

This is the most simple scenario: a single server manages the discovery information of four clients. The server prefix
and listening ports are given in the profiles **UDP server** and **UDP client**. A snapshot is taken after 3 seconds
to assess that all clients are aware of the existence of each other.

.. literalinclude:: ../tests/test_01_PDP_UDP.xml
	:language: xml
	:start-after:	<!-- This test merely assess the clients can locate a UDP server -->
	:end-before:	<profiles>

The snapshot information output would be something like:

.. literalinclude:: ../tests/test_01_snapshot.log

We'll only get this with the debug binary. On Release mode, we can resort to provide a filename to the **snapshots** tag.
Then an XML file will be generated with the same info (note that generating an XML automatically disables validation).

.. literalinclude:: ../tests/test_01_snapshot.xml
	:language: xml
	:start-after:	<DS_Snapshots
	:end-before:	</DS_Snapshots>

Here we see how all participants reported the discovery of all the others. Note that, because there is no Fast-RTPS
discovery callback from a participant to report its own discovery, participants do not report themselves. This must
be taken into account when a snapshot is checked. Note, however, that participants do discover themselves when they
create a publisher or subscriber because there are callbacks associated with those cases.

test_02_PDP_TCP.xml
*******************

Resembles the previous scenario but uses TCP transport instead of the default UDP one. A single server manages the
discovery info of four clients. The server prefix and listening ports are given in the profiles **TCP server** and
**TCP client**. A snapshot is taken after 3 seconds to assess that all clients are aware of the existence of each
other.

Specific transport descriptor must be created for server and clients:

.. literalinclude:: ../tests/test_02_PDP_TCP.xml
	:language: xml
	:start-after:	<transport_descriptors>
	:end-before:	</transport_descriptors>

Client and server participant profiles must reference this transport and discard builtin ones.

.. code-block:: xml

	<participant profile_name="TCP client" >
	  <rtps>
		<userTransports>
		  <transport_id>TCPv4_CLIENT</transport_id>
		</userTransports>
		<useBuiltinTransports>false</useBuiltinTransports>
		...
	  </rtps>
	</participant>

	<participant profile_name="TCP server" >
	  <rtps>
		....
		<userTransports>
		  <transport_id>TCPv4_SERVER</transport_id>
		</userTransports>
		<useBuiltinTransports>false</useBuiltinTransports>
		...
	  </rtps>
	</participant>

test_03_PDP_UDP.xml
*******************

Here we test the discovery capacity of handling late joiners. A single server is created, which manages the discovery
information of four clients with different lifespans. The server prefix and listening ports are given in the profiles
**UDP server** and **UDP client**. A snapshot is taken whenever there is a participant creation or removal to assess
all entities are aware of it.

.. literalinclude:: ../tests/test_03_PDP_UDP.xml
	:language: xml
	:start-after:	<!-- This test merely assess the server capability to handle client changes -->
	:end-before:	<profiles>


test_04_PDP_UDP.xml
*******************

Here we test the capability of one server to exchange information with another one. Two servers are created and each
one has different associated clients. We take a snapshot to assess all clients are aware of the other server's clients
existence. Note that we don't need to modify the previous tests profiles, as we can rely on the *server* and *client* tag
attributes to avoid create redundant boilerplate profiles:

    - *server* **prefix** attribute is used to supersede the profile specified one and uniquely identifies each server.
    - *server* **ListeningPorts** and **ServerList** tags allow us to link servers between them without creating
      specific server profiles.
    - *client* **server** attribute is used to link a client with its server without using a new profile or a
      **ServerList**.

.. literalinclude:: ../tests/test_04_PDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>

test_05_EDP_UDP.xml & test_05_EDP_TCP.xml
*****************************************

These tests introduce dummy publishers and subscribers to assess proper EDP discovery operation over UDP and TCP
transport. A server and two clients are created, and each participant (server included) creates publishers and
subscribers with different types and topics. In the end, a snapshot is taken to verify all publishers and subscribers
have been reported by all participants. Note that the tags *publisher* and *subscriber* have attributes to supersede
topics specified in profiles.

.. literalinclude:: ../tests/test_05_EDP_TCP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>

Snapshots with EDP information are far more verbose:

.. literalinclude:: ../tests/test_05_snapshot.log

test_06_EDP_UDP.xml
*******************

Here we test how the discovery handles EDP late joiners. It's the same scenario with a server and two clients with
different lifespans. Each participant (server included) creates publishers and subscribers with different lifespans,
types, and topics. Snapshots are taken whenever an endpoint is created or destroyed to assess every participant shares
the same discovery info.

.. literalinclude:: ../tests/test_06_EDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>

test_07_PDP_UDP.xml
*******************

Here we test how the discovery handles server shutdown and reboot. This is a clean shutdown made from the Fast RTPS API
:code:`Domain::removeParticipant`. Each time the server dies it notifies this fact to all its clients which
automatically begin pinging on the server again trying to reconnect when its rebooted. Snapshots check that clients
are aware of server absence after shutdown and presence after reboot.

.. literalinclude:: ../tests/test_07_PDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>

test_08_lease_client.xml & test_08_lease_server.xml
***************************************************

Standard lease duration mechanism no longer makes sense on the client-server architecture. Clients no longer multicast
:code:`DATA(p)` messages in order to make all other clients aware of its presence as in PDP standard mechanism, thus,
this periodical messages can no longer be used to assert participant liveliness. In the client-server architecture:

    - clients only track its server liveliness by sending periodical messages to them. If a server dies because of
      lease-duration its client must resume pinging on it in order to reconnect.
    - servers track clients and linked servers liveliness by sending periodical messages to them. If a client dies
      the server must propagate a :code:`DATA(p[UD])` for that client over its PDP network. This way all server's clients
      have a shared lease duration capability.

In order to test this a python script is used to launch two discovery-servers instances:

    - A server with several clients. These instances will take a snapshot at the beginning and another at the end.

    - A client who references the server on the first instance. This process would be killed from python between the
      snapshots.

The first snapshot must show how all clients (remote one included) known each other. After killing process 2
(and its client) the server must kill its proxy by lease duration time out and report it to all other clients.
The second snapshot must show how all participants have removed the remote client from its discovery database.

test_09_WLP_client.xml & test_09_WLP_server.xml
***********************************************

According to the RTPS standard the WLP (Writer Liveliness Protocol) defines the required information exchange between two
Participants in order to assert the liveliness of Writers contained by the Participants. This test assess this
mechanism proper operation under client-server discovery. Basically two discovery-servers instances are launched:

1 - A server with several clients that create subscribers of a common topic. This instances will take an snapshot at the beginning and another at the end.

2 - A client which references the server on the first instance and creates a publisher for the common topic. This
process would be killed from python between the snapshots. It's noteworthy that the lease-duration has been increased
int the profiles to prevent the server from noticing that the client has disappeared.

Both snapshots must show how all clients (remote one included) known each other and the user publishers and
subscribers. Subscribers though are notified by the WLP that the crashed publisher is not responsive. By parsing the
second snapshot we are able to verify that ``alive_count="0"`` there, that is, the subscribers were duly notified.

test_10_simple_PDP_EDP.xml
**************************

This test assess simple discovery proper operation by creating several `simple` participants with several endpoints each.
These participants and its endpoints are not created simultaneously and several snapshots are taken in order to validate
that all participants share the same info at the different test stages.

The syntax resembles the client-server participants. A `simples` collection of `simple` objects.

.. code-block:: xml

    <simples>
        <simple name="simple1" removal_time="10" profile_name="OtherDomain">
            <subscriber />
            ...
        </simple>
        <simple name="simple2" removal_time="10" profile_name="OtherDomain">
            <subscriber />
            ... 
        </simple>
        ...
    </simples>

It's noteworthy that:

- The builtin transport is disabled in order to add whitelisting. The whitelist allows to restrict traffic to the given
  interfaces. In our case to avoid polluting the network and restrict the UDP messages to *localhost*.

- Domain is modified to one randomly generated on CMake generator stage. This way interference with another tests is
  further downgraded.

The other tests only possible collision with each other was try to reserve the very same ports on the TCP and UDP
protocols. Those ports were also randomly generated on CMake generator stage over the IANA ephemeral range, thus,
collision is improbable.

.. literalinclude:: ../tests/test_10_simple_PDP_EDP.xml
	:language: xml
	:start-after:	<profiles>
	:end-before:	</profiles>

test_11_BACKUP_client.xml & test_11_BACKUP_server.xml
*****************************************************

This test verifies that the discovery info is properly serialized and deserialized by a backup server. The server is
created twice by running twice the test_11_BACKUP_server.xml file:

- The first time the discovery data is serialized to a backup file.
- Then this server crashes but the backup file prevails.
- The second time the discovery data is deserialized from the backup file because the leaseDuration doesn't allow the
  clients to detect server's demise.

By comparing the client and last server snapshot it can be verified that the server's discovery info was properly stored
and retrieved.

