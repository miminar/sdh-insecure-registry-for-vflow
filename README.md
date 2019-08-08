# Insecure registry setter for SAP Data Hub Pipeline Modeler

**NOTE**: obsoleted by [miminar/sdh-helpers](https://github.com/miminar/sdh-helpers)

In SAP Data Hub 2.5.x releases, it is possible to configure insecure
registry for Pipeline Modeler (aka vflow pod) neither via installer nor
in the UI.

The insecure registry needs to be set if the container registry listens
on insecure port (HTTP) or the communication is encrypted using a
self-signed certificate.

Without the insecure registry set, kaniko builder cannot push built images into
the configured registry for the Pipeline Modeler (see "Container Registry for
Pipeline Modeler" Input Parameter at [the official SAP Data Hub documentation](
https://help.sap.com/viewer/e66c399612e84a83a8abe97c0eeb443a/2.5.latest/en-US/abfa9c73f7704de2907ea7ff65e7a20a.html).

The registry to mark as insecure will be determined from the
installer-config secret located in the `SDH_NAMESPACE`. If another registry
shall be marked as insecure, it can be specified with an additional
`REGISTRY` parameter.

Please refer to the [SAP Data Hub 2.X on OpenShift Container Platform knowledge
base article](https://access.redhat.com/articles/3630111) for more information.

## How it works

Once deployed, the `vflow-observer` pod will modify all the existing pipeline
modeler deployments and pods in the `SDH_NAMESPACE` project in a way that the
vflow registry is marked as insecure. This will terminate existing pipeline
modeler pods and spawn new ones. Once the modified pods are running, all the
subsequents pushes to the insecure registry will succeed as long as there are
no other issues.

The `vflow-observer` will then continue to watch `SDH_NAMESPACE` project and
will modify any newly created pipeline modeler deployments and pods in the same
way.

## Usage

The `insecure-registry-for-vflow-template.yaml` template can be deployed
before, during or after SAP Data Hub's installation either in the same namespace/project
or in a different one.

### Deploying in the Data Hub project

If running the observer in the same namespace/project as Data Hub, instantiate the
template as is in the desired namespace:

    oc project $SDH_NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-insecure-registry-for-vflow/master/insecure-registry-for-vflow-template.yaml \
       | oc create -f -

### Deploying in a different project

If running in a different/new namespace/project, instantiate the
template with parameters `SDH_NAMESPACE` and `NAMESPACE`, e.g.:

    SDH_NAMESPACE=sdh25
    NAMESPACE=sapdatahub-admin
    oc new-project $NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-insecure-registry-for-vflow/master/insecure-registry-for-vflow-template.yaml \
        SDH_NAMESPACE=$SDH_NAMESPACE NAMESPACE=$NAMESPACE | oc create -f -
