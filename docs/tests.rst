Testing
#######

**Table of Contents**

* :ref:`test_1_PDP_UDP.xml`
* :ref:`test_2_PDP_TCP.xml`
* :ref:`test_3_PDP_UDP.xml`
* :ref:`test_4_PDP_UDP.xml`
* :ref:`test_5_EDP_UDP.xml & test_5_EDP_TCP.xml`
* :ref:`test_6_EDP_UDP.xml`
* :ref:`test_7_PDP_UDP.xml`
* :ref:`test_8_lease_client.xml & test_8_lease_server.xml`

Discovery testing is done resorting to discover-server config files. Each test uses a specific XML file that tests a
single or several discovery features. To automatically launch the tests using colcon, please check section
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
flag `-DLOG_LEVEL_INFO=1`.

A brief description of each test is given below. Note that a detailed explanation of the XML syntax is given in section
`configuration files <xml_schemas.html>`_.

test_1_PDP_UDP.xml
******************

This is the most simple scenario: a single server manages the discovery information of four clients. The server prefix
and listening ports are given in the profiles **UDP server** and **UDP client**. A snapshot is taken after 3 seconds
to assess that all clients are aware of the existence of each other.

.. literalinclude:: ../tests/test_1_PDP_UDP.xml
	:language: xml
	:start-after:	<!-- This test merely assess the clients can locate a UDP server -->
	:end-before:	<profiles>
	
The snapshot information output would be something like:

.. code-block:: bat

	2019-04-24 12:58:36.936 [DISCOVERY_SERVER Info] Snapshot taken at 2019-04-24 12:58:36 description: Check all clients
	met the server and know each other
	5 participants report the following discovery info:
	Participant 1.f.1.30.ac.12.0.0.2.0.0.0|0.0.1.c1 discovered:
			 Participant client2 1.f.1.30.ac.12.0.0.3.0.0.0|0.0.1.c1
			 Participant client3 1.f.1.30.ac.12.0.0.4.0.0.0|0.0.1.c1
			 Participant client4 1.f.1.30.ac.12.0.0.5.0.0.0|0.0.1.c1
			 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1

	Participant 1.f.1.30.ac.12.0.0.3.0.0.0|0.0.1.c1 discovered:
			 Participant client1 1.f.1.30.ac.12.0.0.2.0.0.0|0.0.1.c1
			 Participant client3 1.f.1.30.ac.12.0.0.4.0.0.0|0.0.1.c1
			 Participant client4 1.f.1.30.ac.12.0.0.5.0.0.0|0.0.1.c1
			 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1

	Participant 1.f.1.30.ac.12.0.0.4.0.0.0|0.0.1.c1 discovered:
			 Participant client1 1.f.1.30.ac.12.0.0.2.0.0.0|0.0.1.c1
			 Participant client2 1.f.1.30.ac.12.0.0.3.0.0.0|0.0.1.c1
			 Participant client4 1.f.1.30.ac.12.0.0.5.0.0.0|0.0.1.c1
			 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1

	Participant 1.f.1.30.ac.12.0.0.5.0.0.0|0.0.1.c1 discovered:
			 Participant client1 1.f.1.30.ac.12.0.0.2.0.0.0|0.0.1.c1
			 Participant client2 1.f.1.30.ac.12.0.0.3.0.0.0|0.0.1.c1
			 Participant client3 1.f.1.30.ac.12.0.0.4.0.0.0|0.0.1.c1
			 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1

	Participant 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1 discovered:
			 Participant client1 1.f.1.30.ac.12.0.0.2.0.0.0|0.0.1.c1
			 Participant client2 1.f.1.30.ac.12.0.0.3.0.0.0|0.0.1.c1
			 Participant client3 1.f.1.30.ac.12.0.0.4.0.0.0|0.0.1.c1
			 Participant client4 1.f.1.30.ac.12.0.0.5.0.0.0|0.0.1.c1
			 
We'll only get this with the debug binary. On Release mode we can resort to provide a filename to the **snapshots** tag.
Then an xml file will be generated with the same info (note that generating an xml automatically disables validation
thus it cannot be used in singleton tests).

.. literalinclude:: ../tests/test_1_snapshot.xml
	:language: xml
	:start-after:	<DS_Snapshots>
	:end-before:	</DS_Snapshots>
		 
Here we see how all participants reported the discovery of all the others. Note that, because there is no fast-RTPS
discovery callback from a participant to report its own discovery, participants do not report themselves. This must
be taken into account when a snapshot is checked. Note, however, that participants do discover themselves when they
create a publisher or subscriber, because there are callbacks associated for those cases.

test_2_PDP_TCP.xml
******************

Resembles the previous scenario but uses TCP transport instead of the default UDP one. A single server manages the
discovery info of four clients. The server prefix and listening ports are given in the profiles **TCP server** and
**TCP client**. A snapshot is taken after 3 seconds to assess that all clients are aware of the existence of each
other.
 
Specific transport descriptor must be created for server and clients:
  
.. literalinclude:: ../tests/test_2_PDP_TCP.xml
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
	
test_3_PDP_UDP.xml
******************

Here we test the discovery capacity of handling late joiners. A single server is created, which manages the discovery
information of four clients with different lifespans. The server prefix and listening ports are given in the profiles
**UDP server** and **UDP client**. A snapshot is taken whenever there is a participant creation or removal to assess
all entities are aware of it.

.. literalinclude:: ../tests/test_3_PDP_UDP.xml
	:language: xml
	:start-after:	<!-- This test merely assess the server capability to handle client changes -->
	:end-before:	<profiles>
	

test_4_PDP_UDP.xml
******************

Here we test the capability of one server to exchange information with another one. Two servers are created and each
one has different associated clients. We take a snapshot to assess all clients are aware of the other server's clients
existence. Note that we don't need to modify the previous tests profiles, as we can rely on *server* and *client* tag
attributes to avoid create redundant boilerplate profiles:

    - *server* **prefix** attribute is used to superseed the profile specified one, and uniquely identifies each server.
    - *server* **ListeningPorts** and **ServerList** tags allow us to link servers between them without creating
      specific server profiles.
    - *client* **server** attribute is used to link a client with its server without using a new profile or a
      **ServerList**.
	
.. literalinclude:: ../tests/test_4_PDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>
	
test_5_EDP_UDP.xml & test_5_EDP_TCP.xml
***************************************

These tests introduce dummy publishers and subscribers to assess proper EDP discovery operation over UDP and TCP
transport. A server and two clients are created, and each participant (server included) creates publishers and subscribers with different types and topics. At the end a snapshot is taken to verify all publishers and subscribers have been reported by all participants. Note that the tags *publisher* and *subscriber* have attributes to superseed topics specified in profiles.

.. literalinclude:: ../tests/test_5_EDP_TCP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>
	
Snapshots with EDP information are far more verbose:

.. code-block:: bat

	2019-04-24 14:52:44.300 [DISCOVERY_SERVER Info] Snapshot taken at 2019-04-24 14:52:44 description: Check all
	publishers and subscribers are properly discovered by everybody
	3 participants report the following discovery info:
	Participant 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.c1 discovered: 
		 Participant 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.c1 has:
			1 publishers:
				Publisher 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			3 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.2.4 TypeName: HelloWorld TopicName: HelloWorldTopic
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.3.4 TypeName: sample_type_2 TopicName: topic_2
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.4.4 TypeName: sample_type_1 TopicName: topic_1

		 Participant client2 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.c1 has:
			3 publishers:
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.3 TypeName: HelloWorld TopicName: HelloWorldTopic
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.2.3 TypeName: sample_type_1 TopicName: topic_1
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.3.3 TypeName: sample_type_1 TopicName: topic_1
			1 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.3.0.0.0|0.0.4.4 TypeName: sample_type_2 TopicName: topic_2

		 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1 has:
			1 publishers:
				Publisher 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			1 subscribers:
				Subscriber 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.2.4 TypeName: sample_type_1 TopicName: topic_1


	Participant 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.c1 discovered: 
		 Participant client1 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.c1 has:
			1 publishers:
				Publisher 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			3 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.2.4 TypeName: HelloWorld TopicName: HelloWorldTopic
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.3.4 TypeName: sample_type_2 TopicName: topic_2
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.4.4 TypeName: sample_type_1 TopicName: topic_1

		 Participant 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.c1 has:
			3 publishers:
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.3 TypeName: HelloWorld TopicName: HelloWorldTopic
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.2.3 TypeName: sample_type_1 TopicName: topic_1
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.3.3 TypeName: sample_type_1 TopicName: topic_1
			1 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.3.0.0.0|0.0.4.4 TypeName: sample_type_2 TopicName: topic_2

		 Participant server 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1 has:
			1 publishers:
				Publisher 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			1 subscribers:
				Subscriber 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.2.4 TypeName: sample_type_1 TopicName: topic_1


	Participant 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1 discovered: 
		 Participant client1 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.c1 has:
			1 publishers:
				Publisher 1.f.1.30.64.47.0.0.2.0.0.0|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			3 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.2.4 TypeName: HelloWorld TopicName: HelloWorldTopic
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.3.4 TypeName: sample_type_2 TopicName: topic_2
				Subscriber 1.f.1.30.64.47.0.0.2.0.0.0|0.0.4.4 TypeName: sample_type_1 TopicName: topic_1

		 Participant client2 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.c1 has:
			3 publishers:
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.1.3 TypeName: HelloWorld TopicName: HelloWorldTopic
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.2.3 TypeName: sample_type_1 TopicName: topic_1
				Publisher 1.f.1.30.64.47.0.0.3.0.0.0|0.0.3.3 TypeName: sample_type_1 TopicName: topic_1
			1 subscribers:
				Subscriber 1.f.1.30.64.47.0.0.3.0.0.0|0.0.4.4 TypeName: sample_type_2 TopicName: topic_2

		 Participant 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.c1 has:
			1 publishers:
				Publisher 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.1.3 TypeName: sample_type_2 TopicName: topic_2
			1 subscribers:
				Subscriber 4d.49.47.55.45.4c.5f.42.41.52.52.4f|0.0.2.4 TypeName: sample_type_1 TopicName: topic_1

test_6_EDP_UDP.xml
******************

Here we test how the discovery handles EDP late joiners. It's the same scenario with a server and two clients with
different lifespans. Each participant (server included) creates publishers and subscribers with different lifespans,
types and topics. Snapshots are taken whenever an enpoint is created or destroyed to assess every participant shares
the same discovery info.

.. literalinclude:: ../tests/test_6_EDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>
	
test_7_PDP_UDP.xml
******************

Here we test how the discovery handles server shutdown and reboot. This is a clean shutdown made from the fast RTPS API
:code:`Domain::removeParticipant`. Each time the server dies it notifies this fact to all its clients which
automatically begin pinging on the server again trying to reconnect when its rebooted. Snapshots check that clients
are aware of server absence after shutdown and presence after reboot.

.. literalinclude:: ../tests/test_7_PDP_UDP.xml
	:language: xml
	:start-after:	-->
	:end-before:	<profiles>
	
test_8_lease_client.xml & test_8_lease_server.xml
*************************************************

Standard lease duration mechanism no longer makes sense on the client-server architecture. Clients no longer multicast
:code:`DATA(p)` messages in order to make all other clients aware of its presence as in PDP standard mechanism, thus,
this periodical messages can no longer be used to assert participant liveliness. In the client-server architecture:

    - clients only track its server liveliness by sending periodical messages to them. If a server dies because of
      lease-duration its client must resume pinging on it in order to reconnect.
    - servers track clients and linked servers liveliness by sending preiodical messages to them. If a client dies
      the server must propagate a :code:`DATA(p[UD])` for that client over its PDP network. This way all server's clients
      have a shared lease duration capability.

In order to test this a python script is used to launch two discovery-servers instances:

    - A server with several clients. This instances will take an snapshot at the beginning and another at the end.

    - A client which references the server on the first instance. This process would be killed from python between the
      snapshots.

The first snapshot must show how all clients (remote one included) known each other. After killing process 2
(and its client) the server must kill its proxy by lease duration time out and report it to all other clients.
The second snapshot must show how all participants have removed the remote client from its discovery database.	

