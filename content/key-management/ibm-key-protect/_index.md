---
title: IBM Key Protect
logo: /logos/ibm.png
weight: 1
keywords: Portworx, ibm, kp, containers, storage, encryption
description: Instructions on using IBM Key Protect with Portworx
disableprevnext: true
noicon: true
series: key-management
---


{{<info>}}
**NOTE:** Supported from Portworx Enterprise 1.7 onwards
{{</info>}}

Portworx integrates with IBM Key Protect to store your encryption keys/secrets and credentials. This guide will help configure Portworx with IBM Key Protect. IBM Key Protect can be used to store Portworx secrets for Volume Encryption and Cloud Credentials.

Portworx requires the following IBM Key Protect credentials to use its APIs

- **Service Instance ID [IBM_SERVICE_INSTANCE_ID]**

    The Instance ID of the IBM Key Protect service can be found by running the following command

    ```text
    ibmcloud resource service-instance <name of your key protect service>
    ```

    This should output something like. The ID from the below CRN is `0647c737-906d-blah-8a68-2c187e11b29b`
    ```
    crn:v1:bluemix:public:kms:us-south:a/fb474855a3e76c1ceblahf57e0f1a9f:0647c737-906d-blah-8a68-2c187e11b29b::
    ```

- **Service API Key [IBM_SERVICE_API_KEY]**

    Follow [this](https://cloud.ibm.com/docs/iam?topic=iam-serviceidapikeys#serviceidapikeys) IBM document to retrieve the API Key.

- **Customer Root Key [IBM_CUSTOMER_ROOT_KEY]**

    Follow [this](https://cloud.ibm.com/docs/services/key-protect?topic=key-protect-create-root-keys#create-root-keys) IBM document to create a Customer Root Key

- **Base URL [IBM_BASE_URL]**

    BaseURL specifies the URL where your Key Protect instance resides. It is region specific. Default value which will be used is: `https://keyprotect.us-south.bluemix.net`

- **Token URL [IBM_TOKEN_URL]**

    Default value which will be used is: `https://iam.bluemix.net/oidc/token`
    Based on your installation type use the following methods to provide these credentials to Portworx.

## For Kubernetes Users

### Step 1: Provide IBM Key Protect credentials to Portworx.

Portworx reads the IBM credentials required to authenticate with IBM Key Protect through a Kubernetes secret. Create a Kubernetes secret with the name `px-ibm` in the `portworx` namespace. Following is an example kubernetes secret spec

```text
apiVersion: v1
kind: Secret
metadata:
  name: px-ibm
  namespace: portworx
type: Opaque
data:
  IBM_SERVICE_API_KEY: RnVWZ1JiSndxVHl5VXblah1TWU5paEdHQzhRUVAwVDVMN1NSSHA5Z2VNa2k=
  IBM_INSTANCE_ID: NGZkNDA1Y2QtODBmZC00MGblahE0ZWQtZDdhOTFiMTdiZjEx
  IBM_CUSTOMER_ROOT_KEY: MGYxNGExNzMtODE2blahN2Q3LTg1ZGYtY2M3ZWI5YmYxNzRj
```

Portworx is going to look for this secret with name `px-ibm` under the `portworx` namespace. While installing Portworx it creates a kubernetes role binding which grants access to reading kubernetes secrets only from the `portworx` namespace.

### Step 2: Set up IBM Key Protect as the secrets provider for Portworx.

#### New installation

##### Using helm chart

While deploying Portworx using helm chart on an IKS cluster, by default Portworx is configured to use IBM Key Protect as a secrets provider. Follow [these instructions](https://github.com/portworx/helm/blob/master/charts/portworx/README.md) to install the helm chart.

In a non-IKS cluster, set the `secretType` as `ibm-kp` in the helm chart's values.yml configuration file.

##### Using Portworx Spec Generator
When generating the Portworx Kubernetes spec file on the Portworx Spec Generator page in [PX-Central](https://central.portworx.com), select `IBM Key Protect` from the "Secrets type" list.

#### Existing installation

For an existing Portworx cluster follow these steps to configure IBM Key Protect as the secrets provider

##### Step 2a: Add Permissions to access kubernetes secrets

Portworx needs permissions to access the `px-ibm` secret created in Step 1. The following Kubernetes spec grants portworx access to all the secrets defined under the `portworx` namespace

```text
cat <<EOF | kubectl apply -f -
# Namespace to store credentials
apiVersion: v1
kind: Namespace
metadata:
  name: portworx
---
# Role to access secrets under portworx namespace only
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role
  namespace: portworx
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "create", "update", "patch"]
---
# Allow portworx service account to access the secrets under the portworx namespace
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role-binding
  namespace: portworx
subjects:
- kind: ServiceAccount
  name: px-account
  namespace: kube-system
roleRef:
  kind: Role
  name: px-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

##### Step 2b: Edit the Portworx Daemonset

Edit the Portworx daemonset `secret_type` field to `ibm-kp`, so that all the new Portworx nodes will also start using IBM Key Protect.

```text
kubectl edit daemonset portworx -n kube-system
```

Add the `"-secret_type", "ibm-kp"` arguments to the `portworx` container in the daemonset. It should look something like this:
```text
containers:
  - args:
    - -c
    - testclusterid
    - -s
    - /dev/sdb
    - -x
    - kubernetes
    - -secret_type
    - ibm-kp
    name: portworx
```

Editing the daemonset will restart all the Portworx pods.

## Other users

### Step 1: Provide IBM Key Protect credentials to Portworx.

Provide the following IBM credentials (key value pairs) as environment variables to Portworx

- [Required] IBM_SERVICE_INSTANCE_ID=[service_instance_id]
- [Required] IBM_SERVICE_API_KEY=[service_api_key]
- [Required] IBM_CUSTOMER_ROOT_KEY=[customer_root_key]
- [Optional] IBM_BASE_URL=[base_url] → only required if the instance is in a different region
- [Optional] IBM_TOKEN_URL=[token_url] → only required if the different than the default token url

### Step 2: Set up IBM Key Protect as the secrets provider for Portworx.

#### New installation

While installing Portworx set the input argument `-secret_type` to `ibm-kp`.

#### Existing installation

Based on your installation method provide the `-secret_type ibm-kp` input argument and restart Portworx on all the nodes.


## Using IBM Key Protect with Portworx

{{<homelist series="ibm-key-protect-uses">}}
