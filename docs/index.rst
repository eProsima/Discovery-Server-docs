.. eProsima Discovery Server documentation master file

**********************************************
eProsima Discovery Server Documentation
**********************************************

.. image:: /01-figures/logo.png
    :height: 100px
    :width: 100px
    :align: left
    :alt: eProsima
    :target: http://www.eprosima.com/

The `RTPS standard <http://www.omg.org/spec/DDSI-RTPS/2.2>`__ specifies in section 8.5 a non-centralized, distributed
simple discovery mechanism. This mechanism was devised to allow interoperability among independent
vendor-specific implementations but is not expected to be optimal in every environment.
There are several scenarios were the simple discovery mechanism is unsuitable or plainly cannot be
applied: a) a high number of endpoint entities are continuously entering and leaving the communication, b) wide
communication systems deployment, and c) networks without multicasting capabilities.

In order to cope with the above issues, the *eProsima Fast DDS* discovery mechanism was extended with a new
Discovery Server discovery mechanism.
This mechanism is based on a client-server discovery paradigm, i.e. the metatraffic (message exchange among
DDS DomainParticipants to identify each other) is managed by one or several server DomainParticipants (left figure), as
opposed to simple discovery (right figure), where metatraffic is exchanged using a message broadcast mechanism like an
IP multicast protocol.
Please, refer to
`Fast DDS documentation <https://fast-dds.docs.eprosima.com/en/latest/fastdds/discovery/server_client.html>`__ for
further information about the Discovery Server discovery mechanism.

This documentation is organized into the following sections:

* :ref:`installation`
* :ref:`user`
* :ref:`examples`
* :ref:`tests`
* :ref:`notes`

.. _installation:

.. toctree::
	:caption: Installation

	installation

.. _user:

.. toctree::
	:caption: User Manual

	command_line
	xml_schemas

.. _examples:

.. toctree::
	:caption: Examples

	HelloWorldExample

.. _tests:

.. toctree::
	:caption: Tests

	tests

.. _notes:

.. toctree::
    :caption: Release Notes

    notes
