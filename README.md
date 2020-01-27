# Example setup the IoT simulator

## Getting EnMasse

Download EnMasse:

    curl -sL https://github.com/EnMasseProject/enmasse/releases/download/0.30.2/enmasse-0.30.2.tgz -o enmasse.tar.gz

Unpack the archive:

    mkdir enmasse
    cd enmasse
    tar --strip-components=1 -xzf ../enmasse.tar.gz
    cd ..

## Using CRC

When using CRC, create a new machine using the following command:

    crc start

## Prepare your OpenShift cluster

Create a new project:

    # Log in as admin
    oc new-project enmasse-infra

## Installing EnMasse

    oc apply -f enmasse/install/bundles/enmasse
    oc apply -f enmasse/install/preview-bundles/iot
    oc apply -f enmasse/install/components/example-authservices/standard-authservice.yaml
    oc apply -f enmasse/install/components/example-roles
    oc apply -f enmasse/install/components/example-plans

## Enable IoT feature

Enable the feature:

    oc apply -f deploy/010-Infrastructure

And wait for the configuration to be ready:

    oc get iotconfig

The output should show `Ready` in the column `PHASE`:

    NAME      PHASE
    default   Ready

## Create a new IoT project

Create the new project:

    oc new-project myapp
    oc apply -f deploy/020-Project

And wait for the project to be ready:

    oc get iotproject

The output should show `Ready` in the column `PHASE`:

    NAME   IOT TENANT   DOWNSTREAM HOST                          DOWNSTREAM PORT   TLS    PHASE
    iot    myapp.iot    messaging-vv2rt4cho5.enmasse-infra.svc   5671              true   Ready

## Deploy the simulator

    oc new-project iot-simulator
    oc apply -f deploy/030-SimulatorOperator

## Create a new simulator instance

Get the necessary information:

    export MESSAGING_HOST=$(oc -n myapp get addressspace iot -o 'jsonpath={ .status.endpointStatuses[?(@.name=="messaging")].serviceHost }')
    export CA_CERT=$(oc -n myapp get addressspace iot -o 'jsonpath={ .status.caCert }')
    export MGMT_TOKEN=$(oc whoami -t)

And then apply the configuration:

    cat deploy/040-Simulator/010-Simulator.yaml.in | envsubst | oc apply -f -

## Deploy the simulator workload

    oc apply -f deploy/050-Workload
