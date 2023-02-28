# Openshift Image Catalog Mirror

oc-mirror OpenShift CLI (oc) plugin allows customers to mirror images to a mirror registry in a fully or partially disconnected environments using a single tool.

It provides the following features:

* Provides a centralized method to mirror OpenShift Container Platform releases, Operators, helm charts, and other images.
* Maintains update paths for OpenShift Container Platform and Operators.
* Uses a declarative image set configuration file to include only the OpenShift Container Platform releases, Operators, and images that your cluster needs.
* Performs incremental mirroring, which reduces the size of future image sets.
* Prunes images from the target mirror registry that were excluded from the image set configuration since the previous execution.
* Optionally generates supporting artifacts for OpenShift Update Service (OSUS) usage.

It is important to bear in mind that it is required to run oc-mirror from a system with internet connectivity in order to download the required images from the official Red Hat registries.

## Prerequisites

* OpenShift Container Platform 4.9+
* Use the latest available version of the oc-mirror plugin regardless of which versions of OpenShift Container Platform you need to mirror.
* OpenShift Client (oc) mirror plugin 4.12.4+
* Mirror Registry for Red Hat OpenShift 1.3.0+ 

NOTE: It is possible to download the previous binaries from the (Red Hat Openshift Downloads Page)[https://console.redhat.com/openshift/downloads]

## Procedure

### Create an Internal Mirror Registry

First of all, it is required to deploy a *Mirror Registry* in order to have a image repository for installing/updating the Openshift environments and operators. Regarding requirements, it is important:

* A RHEL 8+ machine with Podman 3.3+
* RAM 6GB+
* Storage 1TB (*At least 500GB*)

In order to deploy this registry, it is required follow the next steps:

* Install de *Mirror Registry*

```
sudo ./mirror-registry install \
  --quayHostname <host_example_com> \
  --quayRoot <example_directory_name>

...
INFO[2023-02-21 16:32:32] Quay installed successfully, config data is stored in /tmp/quay 
INFO[2023-02-21 16:32:32] Quay is available at <host_example_com> with credentials (init, L73w0zi2QC849oNhf1xJRdWjVs5H6naF)
```

* Test login with browser 

* Test login with podman
```
podman login -u init -p <password> <host_example_com>:8443> --tls-verify=false 
```

NOTE: It is important to use a hostname that it is reacheable 

### Mirror Openshift Images

First of all, it is required to have a file named **pull-secret.txt** with an Openshift valid pull secret.

One the previous file has been create, it is time to execute the following procedure:

* Generate the credentials in base64 mode

```
echo -n 'init:<password>' | base64 -w0
```

* Generate the final pull secret file

```
## Create a local registry pull secret
cat << EOF > pull-secret-local.json
{
    "<host_example_com>:8443>": { 
      "auth": "<base64_credentials>", 
      "email": "test@test.com"
    }
}
EOF

## Prepare the pull secret
jq --argjson repo "$(<pull-secret-local.json)" '.auths += $repo' pull-secret.text > pull-secret.json

## Copy 
cp pull-secret.json $XDG_RUNTIME_DIR/containers/auth.json
```

* Modify the *ImageSetConfig* in order to add the correct imageURL to the mirror registry

```
vi ./imageset-config.yaml
```

* Mirror the different images

```
oc mirror --config=./imageset-config.yaml docker://<host_example_com>:8443>/ocp412 --dest-use-http
```

Once the oc mirror command is finish, it is possible to find all the images behind the respective ocp412 organization.

## Author

Asier Cidon @RedHat