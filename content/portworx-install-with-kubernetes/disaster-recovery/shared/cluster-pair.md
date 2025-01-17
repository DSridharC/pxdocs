---
hidden: true
---

## Generate and Apply a ClusterPair Spec

In Kubernetes, you must define a trust object called **ClusterPair**. Portworx requires this object to communicate with the destination cluster. The ClusterPair object pairs the Portworx storage driver with the Kubernetes scheduler, allowing the volumes and resources to be migrated between clusters.

The ClusterPair is generated and used in the following way:

   * The **ClusterPair** spec is generated on the **destination** cluster.
   * The generated spec is then applied on the **source** cluster

Perform the following steps to create a cluster pair:

### Create object store credentials for cloud clusters

If you are running Kubernetes on-premises, you may skip this section. If your Kubernetes clusters are on the cloud, you must create object credentials on both the destination and source clusters before you can create a cluster pair.

The options you use to create your object store credentials differ based on which object store you use:

#### Create Amazon s3 credentials

Create the credentials by entering the `pxctl credentials create` command, specifying the following:

  * `--provider` as `s3`
  * `--s3-access-key` with your aws access key
  * `--s3-secret-key` with your aws secret key
  * `--s3-region` with your region
  * `--s3-endpoint` with `s3.amazonaws.com`
  * `clusterpair_` with the UUID of your destination cluster

```text
/opt/pwx/bin/pxctl credentials create --provider s3 --s3-access-key <aws_access_key> --s3-secret-key <aws_secret_key> --s3-region us-east-1  --s3-endpoint s3.amazonaws.com clusterpair_<UUID_of_destination_cluster>
```

#### Create Microsoft Azure credentials

Create the credentials by entering the `pxctl credentials create` command, specifying the following:

  * `--provider` as `azure`
  * `--azure-account-name` with the name of your Azure account
  * `--azure-account-key` with your Azure account key
  * `clusterpair_` with the UUID of your destination cluster appended

```text
/opt/pwx/bin/pxctl credentials create --provider azure --azure-account-name <your_azure_account_name> --azure-account-key <your_azure_account_key> clusterpair_<UUID_of_destination_cluster>
```

#### Create Google Cloud Platform credentials

Create the credentials by entering the `pxctl credentials create` command, specifying the following:

  * `--provider` as `google`
  * `--google-project-id` with the string of your Google project ID
  * `--google-json-key-file` with the filename of your GCP JSON key file
  * `clusterpair_` with the UUID of your destination cluster appended

```text
/opt/pwx/bin/pxctl credentials create --provider google --google-project-id <your_google_project_ID> --google-json-key-file <your_GCP_JSON_key_file> clusterpair_<UUID_of_destination_cluster>
```

### Generate a ClusterPair on the destination cluster

To generate the **ClusterPair** spec, run the following command on the **destination** cluster:

```text
storkctl generate clusterpair -n migrationnamespace remotecluster
```
Here, the name (remotecluster) is the Kubernetes object that will be created on the **source** cluster representing the pair relationship.

During the actual migration, you will reference this name to identify the destination of your migration.
