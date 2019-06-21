# Insecure registry setter for SAP Data Hub Pipeline Modeler

In SAP Data Hub 2.5.x releases, it is possible to configure insecure
registry for Pipeline Modeler (aka vflow pod) neither via installer nor
in the UI.

The insecure registry needs to be set if the container registry listens
on insecure port (HTTP) or the communication is encrypted using a
self-signed certificate.

Without the insecure registry set, kaniko builder cannot push built
images into the configured registry for the Pipeline Modeler (see
"Container Registry for Pipeline Modeler" Input Parameter at
https://help.sap.com/viewer/e66c399612e84a83a8abe97c0eeb443a/2.5.latest/en-US/abfa9c73f7704de2907ea7ff65e7a20a.html)

The registry to mark as insecure will be determined from the
installer-config secret located in the `SDH_NAMESPACE`. If another registry
shall be marked as insecure, it can be specified with an additional
`REGISTRY` parameter.

## Usage

If running the observer in the same namespace as Data Hub, instantiate the
template as is in the desired namespace:

    oc project $SDH_NAMESPACE
    oc process insecure-registry-for-vflow | oc create -f -

If running in a different/new namespace/project, instantiate the
template with parameters `SDH_NAMESPACE` and `NAMESPACE`, e.g.:

    oc new-project sapdatahub-admin
    oc process insecure-registry-for-vflow \
        SDH_NAMESPACE=$SDH_NAMESPACE \
        NAMESPACE=sapdatahub-admin | oc create -f -

