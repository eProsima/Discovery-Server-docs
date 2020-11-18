# eProsima Discovery Server Documentation

<a href="http://www.eprosima.com">
    <img src="https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcSd0PDlVz1U_7MgdTe0FRIWD0Jc9_YH-gGi0ZpLkr-qgCI6ZEoJZ5GBqQ" align="left" hspace="8" vspace="2" width="100" height="100" >
</a>

The [RTPS standard](http://www.omg.org/spec/DDSI-RTPS/2.3) specifies in section 8.5 a non-centralized, distributed
simple discovery mechanism. This mechanism was devised to allow interoperability among independent
vendor-specific implementations but it is not expected to be optimal in every environment.
There are several scenarios were the simple discovery mechanism is unsuitable or plainly cannot be
applied:
a) a high number of endpoint entities are continuously entering and exiting a large network.
b) networks without multicasting capabilities.
c) WiFi-based communication networks.

In order to cope with the aforementioned issues, the *eProsima Fast DDS* discovery mechanisms are extended with a new
Discovery Server discovery paradigm.
This is based on a client-server architecture, i.e. the metatraffic (message exchange among
DDS DomainParticipants to identify each other) is managed by one or several server DomainParticipants (right figure),
as opposed to simple discovery (left figure), where metatraffic is exchanged using a message broadcast mechanism like
an IP multicast protocol.
Please, refer to
[Fast DDS documentation](https://fast-dds.docs.eprosima.com/en/latest/fastdds/discovery/server_client.html) for
further information about the Discovery Server discovery mechanism.

![discovery diagrams][diagrams]

[diagrams]: docs/01-figures/ds_uml.png

The Discovery Server tool is developed to ease Discovery Server discovery mechanism setup and testing.

You can find all the application's source code on our [GitHub repository](https://github.com/eProsima/Discovery-Server).

The documentation is built using [Sphinx](https://www.sphinx-doc.org), and it is hosted at
[Read the Docs](https://readthedocs.org).
The online documentation generated with this project can be found in
[Discovery Server documentation](https://eprosima-discovery-server.readthedocs.io).

1. [Installation Guide](#installation-guide)
1. [Getting Started](#getting-started)
1. [Generating documentation in other formats](#generating-documentation-in-other-formats)
1. [Running documentation tests](#running-documentation-tests)
1. [Contributing](#contributing)

## Project structure

The project is structured as follows:

1. The root directory contains global scope files, such as this one.
1. The [docs directory](#docs-directory) contains all documentation source code.

## Installation Guide

1. In order to build and test the documentation, some dependencies must be installed beforehand:

    ```bash
    sudo apt update
    sudo apt install -y \
        git \
        gcc \
        g++ \
        cmake \
        curl \
        wget \
        libasio-dev \
        libtinyxml2-dev \
        doxygen \
        python3 \
        python3-pip \
        python3-venv \
        python3-sphinxcontrib.spelling \
        imagemagick
    ```

1. Clone the repository

    ```bash
    cd ~
    git clone https://github.com/eProsima/Discovery-Server-docs discovery-server-docs
    ```

1. Create a virtual environment and install python3 dependencies.

    ```bash
    cd ~/discovery-server-docs
    python3 -m venv discovery-server-docs-venv
    source discovery-server-docs-venv/bin/activate
    pip3 install -r docs/requirements.txt
    cd discovery-server-docs-venv/lib/<python-version>/site-packages
    curl https://patch-diff.githubusercontent.com/raw/sphinx-doc/sphinx/pull/7851.diff | git apply
    cd -
    ```

    The version of python3 used in the virtual environment can be seen by running the following command within the
    virtual environment:

    ```bash
    python3 -V
    ```

## Getting Started

To generate the documentation in a HTML format for a specific branch of Fast DDS run:

```bash
cd ~/discovery-server-docs
source discovery-server-docs-venv/bin/activate
make html
```

## Generating documentation in other formats

The documentation can be generated in several formats such as HTML, PDF, LaTex, etc. For a complete list of targets run:

```bash
cd ~/discovery-server-docs
make help
```

Once you have selected a format, generate the documentation with:

```bash
cd ~/discovery-server-docs
source discovery-server-docs-venv/bin/activate
FASTDDS_BRANCH=<branch> make <output_format>
```

## Running documentation tests

DISCLAIMER: In order to run documentation tests, access to eProsima's intranet is required.

This repository provides a set of tests that verify that:

1. The RST follows the style guidelines
1. The HTML is built correctly
1.There are no spelling errors

Run the tests by:

```bash
cd ~/discovery-server-docs
source discovery-server-docs-venv/bin/activate
FASTDDS_BRANCH=<branch> make test
```

## Contributing

If you are interested in making some contributions, either in the form of an issue or a pull request, please refer to
our [Contribution Guidelines](https://github.com/eProsima/all-docs/blob/master/CONTRIBUTING.md).

