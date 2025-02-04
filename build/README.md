# OSCAL Ci/CD Build Tools

This subdirectory contains a set of build scripts used to create OSCAL-related artifacts (e.g., schemas, converters, documentation). Below are instructions for building using these scripts.

## Prerequisites

If using Docker:

- [Docker 19.03+](https://docs.docker.com/install/)

If not using Docker:

- macOS, Linux or Windows Subsystem for Linux (WSL) (the build scripts don't support Windows natively at this time)

## Executing the Build Environment with Docker

A Docker container configuration is provided that establishes the runtime environment needed to run the build scripts.

1. Install Docker and Docker Compose

    - Follow the Docker installation [instructions](https://docs.docker.com/install/) for your system.
    - Follow the Docker Compose installation [instructions](https://docs.docker.com/compose/install/) for your system. Note: Some docker installations also install docker compose. The installation instructions will tell you if this is the case.

2. Build the Docker container

    You can build the Docker conatiner for the build environment using Docker Compose as follows:

    ```
    docker-compose build
    ```

3. Run the Docker container

    Executing the container will launch an interactive shell that is preconfigured with all required tools and the needed environment pre-configured.

    You can run the Docker conatiner for the build environment using Docker Compose as follows:

    ```
    docker-compose run cli
    ```

    On Windows environments, you may need to execute in a pty that allows for using an interactive shell. In such a case you can run the Docker conatiner as follows:


    ```
    winpty docker-compose run cli
    ```

    This should launch an interactive shell.

## Manual Setup of the Build Environment

The following steps are known to work on Ubuntu [18.04 LTS](http://releases.ubuntu.com/18.04.4/).

1. Setup environment variables

    A few environment variables are used to configure the runtime environment.

    - SAXON_VERSION - Defines which version of Saxon-HE to use
    - SCHEMATRON_HOME - Defines the directory where the Schematron Skeleton will be checked out
    - OSCAL_TOOLS_DIR - Defines the directory where the OSCAL tools will be checked out. The OSCAL Java Validation API is one of the needed tools located in this repo.
    - JSON_CLI_VERSION - Defines the version of the OSCAL Java Validation API

    The following is an example of how to configure the environment. You will need to customize the `SCHEMATRON_HOME` and `OSCAL_TOOLS_DIR` variables for your own environment.

    ```bash
    export SAXON_VERSION="9.9.1-3"
    export SCHEMATRON_HOME=/home/user/git-schematron
    export OSCAL_TOOLS_DIR=/home/user/oscal-tools
    export JSON_CLI_VERSION=0.0.1-SNAPSHOT
    ```

    You may want to add these exports to your `~/.bashrc` to persist the configuration.

1. Install required packages

    To install the required Linux packages, run the following:

    ```bash
    apt-get update && apt-get install -y apt-utils libxml2-utils jq maven hugo nodejs npm build-essential python3-pip git && apt-get clean
    ```

1. Install Node.js modules

    To install the required Node.js modules, run the following:

    ```bash
    npm install -g prettyjson markdown-link-check json-diff
    ```

1. Install Python modules

    To install the required Python modules, run the following:

    ```bash
    pip3 install lxml
    ```

1. Install Schematron Skeleton

    The build environment uses the [ISO Schematron](http://schematron.com/) Skeleton to generate Schematron schemas used for some validation operations.

    To install the Schematron Skeleton, run the following:

    ```bash
    mkdir -p "${SCHEMATRON_HOME}"
    git clone --depth 1 --no-checkout https://github.com/Schematron/schematron.git ."${SCHEMATRON_HOME}"
    cd "${SCHEMATRON_HOME}"
    git checkout master -- trunk/schematron/code
    ```

1. Install and Build the Java JSON Validation CLI

    The build environment uses a custom JSON validation API command line tool to validate JSON content.

    To install the JSON validation API command line tool, run the following:

    ```bash
    mkdir -p "${OSCAL_TOOLS_DIR}"
    git clone --depth 1 https://github.com/usnistgov/oscal-tools.git "${OSCAL_TOOLS_DIR}"
    cd "${OSCAL_TOOLS_DIR}/json-cli"
    mvn install
    ```
1. Install Saxon-HE

    To install Saxon-HE, run the following:

    ```bash
    mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:get -DartifactId=Saxon-HE -DgroupId=net.sf.saxon -Dversion=${SAXON_VERSION}
    ```

Your environment should be setup.

## Running the Build Scripts

From the root directory of the repository, execute the following command to run all scripts:

```bash
./build/ci-cd/run-all.sh -v
```

This will execute all validations, generate all OSCAL artifacts, and execute all unit tests.

## Running the Scripts Individually

The following executions can be used to build various OSCAL artifacts.

### Building XML and JSON Schema for the OSCAL models

To build the XML and JSON Schema for the OSCAL models, run the following:

```
./build/ci-cd/generate-schema.sh
```

This will generate schemas for the Metaschema definitions defined in the metaschema [configuration file][metaschema-config].

### Building XML-to-JSON and JSON-to-XML Converters for the OSCAL models

To build the XML-to-JSON and JSON-to-XML Converters for the OSCAL models, run the following:

```
./build/ci-cd/generate-content-converters.sh
```

This will generate converters for the Metaschema definitions defined in the metaschema [configuration file][metaschema-config].

### Building Website Documentation

To generate OSCAL model documentation, which is used as part of the [website generation pipeline](../docs), execute the following:

```
./build/ci-cd/generate-model-documentation.sh
```

[metaschema-config]: ci-cd/config/metaschema
