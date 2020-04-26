.. eProsima Discovery Server documentation master file

***************************************
eProsima Discovery Server Documentation
***************************************

.. image:: logo.png
    :height: 80px
    :width: 80px
    :align: left
    :alt: eProsima
    :target: http://www.eprosima.com/

| The `Current DDS-RTPS standard`_ in its section 8.5 specifies a non-centralized, distributed simple discovery mechanism
| for RTPS. This mechanism was devised to allow interoperability among independent vendor-specific implementations
| but is not expected to be optimal in every environment.

There are several scenarios were the simple discovery mechanism is unsuitable or plainly cannot be applied:

+ a high number of endpoint entities are continuously entering and exiting a large network.
+ a network without multicasting capabilities.

In order to cope with the above issues, the Fast-RTPS discovery mechanism was extended with a client-server
functionality. Besides, to simplify the management and testing of this new functionality this discovery-server
application was devised.

.. _`Current DDS-RTPS standard`: https://www.omg.org/spec/DDSI-RTPS/2.3

This documentation is organized into the following sections:

* :ref:`installation`
* :ref:`user`
* :ref:`examples`
* :ref:`tests`
* :ref:`notes`

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
	:caption: Tests explanation

	tests

.. _notes:

.. toctree::
    :caption: Release Notes

    notes
