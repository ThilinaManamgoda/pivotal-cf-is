# BOSH release for WSO2 Identity Server deployment pattern 2

This directory contains the BOSH release implementation for WSO2 Identity Server 5.4.1
[deployment pattern 2](https://docs.wso2.com/display/IS541/Deployment+Patterns#DeploymentPatterns-Pattern2-HAclustereddeploymentofWSO2IdentityServerwithWSO2IdentityAnalytics).

![WSO2 Identity Server 5.4.1 deployment pattern 2](images/pattern-2.png)

The following sections provide general steps required for managing the WSO2 Identity Server 5.4.1 deployment pattern 2
BOSH release in a BOSH environment deployed in the desired IaaS.

For step-by-step guidelines to manage the BOSH release in specific environments, refer the following:
   - [In a local environment](bosh-lite.md) (using BOSH Lite)

## Contents

* [Prerequisites](#prerequisites)
* [Create Release](#create-release)
* [Deploy Release](#deploy-release)
* [Output](#output)
* [Delete Deployment](#delete-deployment)
* [BOSH Release Structure](#bosh-release-structure)
* [References](#references)

## Prerequisites

1. Install the following software:

    - [BOSH Command Line Interface (CLI) v2+](https://bosh.io/docs/cli-v2.html)
    - [WSO2 Update Manager (WUM)](http://wso2.com/wum)
    - [Git client](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
    - Software requirements specific to the IaaS
    
2. Obtain the following software distributions.

    - WSO2 Identity Server 5.4.1 WUM updated product distribution
    - WSO2 Identity Server Analytics 5.4.1 WUM updated product distribution
    - [Java Development Kit (JDK) 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
    - Relevant Java Database Connectivity (JDBC) connector (e.g. [MySQL JDBC driver](https://dev.mysql.com/downloads/connector/j/5.1.html)
    if the external database used is MySQL)
    
3. Clone this Git repository.

    ```
    git clone https://github.com/wso2/pivotal-cf-is
    ```
    
   **Note**: In the remaining sections, the project root directory has been referred to as, **pivotal-cf-is**.

## Create release

In order to create the BOSH release for deployment pattern 2, you must follow the standard steps for creating a release with BOSH.
 
1. Move to root directory of the deployment pattern 2 BOSH release.

    ```
    cd <pivotal-cf-is>/pattern-2/bosh-release/
    ```   
    
2. Create a BOSH environment and login to it.

    Please refer the [BOSH documentation](http://bosh.io/docs/init.html) for instructions on creating a BOSH environment in the desired IaaS.

3. Add the WSO2 Identity Server 5.4.1 and Identity Server Analytics 5.4.1 WUM updated product distributions, JDK distribution and MySQL JDBC driver
in the form of release blobs.

    Here, the **environment-alias** refers to the alias provided when saving the created environment, in step 2.

    ```
    bosh -e <environment-alias> add-blob <local_system_path_to_JDK_distribution> oraclejdk/jdk-8u<update-version>-linux-x64.tar.gz
    bosh -e <environment-alias> add-blob <local_system_path_to_MySQL_driver> mysqldriver/mysql-connector-java-<version>-bin.jar
    bosh -e <environment-alias> add-blob <local_system_path_to_WSO2_IS_distribution> wso2is/wso2is-<version>.zip
    bosh -e <environment-alias> add-blob <local_system_path_to_WSO2_IS_Analytics_distribution> wso2is_analytics/wso2is-analytics-<version>.zip
    ```
    
    **Note**: For evaluation purposes, the BOSH release implementation has created a release package for MySQL JDBC driver.
    This assumes that by default, the used external DBMS is MySQL. But if the external DBMS used is of another type, [create
    a BOSH release package](https://bosh.io/docs/packages.html) (similar to `<pivotal-cf-is>/pattern-2/bosh-release/packages/mysqldriver`) for the particular
    JDBC driver. Then, you have to upload the relevant JDBC driver in the form of a blob, as above.

4. **[Optional]** If the BOSH release is a final release, upload the blobs (added in step 4). Please refer
[BOSH documentation](https://bosh.io/docs/create-release.html#upload-blobs) for further details.

    ```
    bosh -e <environment-alias> -n upload-blobs
    ```

5. Create the BOSH release.

   - Dev release:
   ```
   bosh -e <environment-alias> create-release --force
   ```
   Please refer [BOSH Documentation](https://bosh.io/docs/create-release.html#dev-release) for detailed information on creating a dev release.
   
   - Final release:
   ```
   bosh -e <environment-alias> create-release
   ```
   Please refer [BOSH Documentation](https://bosh.io/docs/create-release.html#final-release) for detailed information on creating a final release.

## Deploy release

1. Setup and configure external product database(s).

    - Currently, it is expected that the external database holds the user management, registry, identity and workflow feature database tables
    for Identity Server and Identity Server Analytics.
    Please see WSO2 Identity Server [Documentation](https://docs.wso2.com/display/IS541/Setting+Up+Separate+Databases+for+Clustering)
    for further details.

    - Following table shows the external product database configurations, which have been set as properties under WSO2 Identity Server job specifications
    (**e.g.** see `properties` section under `<pivotal-cf-is>/pattern-2/bosh-release/jobs/wso2is_<job_number>/spec`).
    
   <br>
   
   Property | Description | Default
   -------- | ----------- | -------
   wso2is.user_ds.url | Connection URL of the user data source. | -
   wso2is.registry_ds.url | Connection URL of the registry data source. | -
   wso2is.identity_ds.url | Connection URL of the identity data source. | -
   wso2is.bps_ds.url | Connection URL of the BPS data source. | -
   wso2is.db.driver | Database driver class name of the data source. | -
   wso2is.db.username | Username of the WSO2 Identity Server product database user. | root
   wso2is.db.password | Password of the WSO2 Identity Server product database user. | root
   wso2is_analytics.user_ds.url | Connection URL of the user data source | -
   wso2is_analytics.registry_ds.url | Connection URL of the Registry data source | -
   wso2is_analytics.event_store_ds.url | Connection URL of the event store data source | -
   wso2is_analytics.processed_data_store_ds.url | Connection URL of the processed data store data source | -
   wso2is_analytics.db.driver | Database driver class name of the data source | -
   wso2is_analytics.db.username | Username of the WSO2 Identity Server Analytics product database user | root
   wso2is_analytics.db.password | Password of the WSO2 Identity Server Analytics product database user | root
   
   Among the above, the properties with no default values **must** be set in the deployment manifest,
   prior deployment of the release.
   If you intend to customize any of the properties with default values, you may set the customized values
   in the deployment manifest, prior to deployment of the release.
   
2. Move to the root directory of deployment pattern 2 BOSH release.

    ```
    cd <pivotal-cf-is>/pattern-2/bosh-release
    ```
    
3. Upload the deployment pattern 2 BOSH release.

    ```
    bosh -e <environment-alias> upload-release
    ```

4. Upload the desired stemcell directly to BOSH. [bosh.io](http://bosh.io/stemcells) provides a resource to find and download stemcells.

    ```
    bosh -e <environment-alias> upload-stemcell <URL/local_system_path_to_stemcell>
    ```
    
5. Upload the deployment manifest.

    ```
    bosh -e <environment-alias> -d wso2is-pattern-2 deploy manifests/<deployment-manifest>.yml
    ```
    
## Output

To find the IP addresses of created instances via the BOSH CLI and access the WSO2 Identity Server management console via a web browser,

1. List all the instances within a deployment.

    ```
    bosh -e <environment-alias> -d wso2is-pattern-2 vms
    ```

2. SSH into an instance.

    ```
    bosh -e <environment-alias> -d wso2is-pattern-2 ssh <instance_id>
    ```
    
3. Access the WSO2 Identity Server management console URL using the static IPs of the created instances.

    ```
    https://<IP_Address_of_IS_instance_1>:9443/carbon
    ```
    or
    ```
    https://<IP_Address_of_IS_instance_2>:9443/carbon
    ```
4. Access the WSO2 Identity Server Analytics management console URL using the static IPs of the created instances.

    ```
    https://<IP_Address_of_IS_Analytics_instance_1>:9444/carbon
    ```
    or
    ```
    https://<IP_Address_of_IS_Analytics_instance_2>:9444/carbon
    ```

## Delete deployment

1. Delete the deployment.

    ```
    bosh -e <environment-alias> -d wso2is-pattern-2 delete-deployment
    ```

2. **[Optional]** Cleanup the BOSH release, stemcell, disks and etc.

    ```
    bosh -e <environment-alias> clean-up --all
    ```

## BOSH release structure

Structure of the directories and files of the BOSH release is as follows:

```
└── bosh-release
    ├── config
    ├── dbscripts
    ├── images
    ├── jobs
    ├── manifests
        ├── wso2is-manifest.yml
    ├── packages
    └── README.md
```

## References

* [BOSH CLI v2 commands](https://bosh.io/docs/cli-v2.html)
