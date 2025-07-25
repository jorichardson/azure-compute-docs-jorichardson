---
title: Understanding periodic backup configuration
description: Use Service Fabric's periodic backup and restore feature to configure periodic backup of your Reliable stateful services or Reliable Actors.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 04/17/2025
# Customer intent: "As a cloud administrator, I want to configure periodic backups for my stateful services, so that I can ensure data integrity and recovery in case of data loss."
---

# Understanding periodic backup configuration in Azure Service Fabric

Configuring periodic backup of your Reliable stateful services or Reliable Actors consists of the following steps:

1. **Creation of backup policies**: In this step, one or more backup policies are created depending on requirements.

2. **Enabling backup**: In this step, you associate backup policies created in **Step 1** to the required entities, _Application_, _Service_, or a _Partition_.

## Create Backup Policy

A backup policy consists of the following configurations:

* **Auto restore on data loss**: Specifies whether to trigger restore automatically using the latest available backup in case the partition experiences a data loss event.
> [!NOTE]
> It is recommended to NOT set Auto Restore in production clusters
>

* **Max incremental backups**: Defines the maximum number of incremental backups to be taken between two full backups. Max incremental backups specify the upper limit. A full backup may be taken before specified number of incremental backups are completed in one of the following conditions

    1. The replica has never taken a full backup since it has become primary.

    2. Some of the log records since the last backup have been truncated.

    3. Replica passed the MaxAccumulatedBackupLogSizeInMB limit.

* **Backup schedule**: The time or frequency at which to take periodic backups. One can schedule backups to be recurring at specified interval or at a fixed time daily/ weekly.

    1. **Frequency-based backup schedule**: This schedule type should be used if the need is to take data backup at fixed intervals. Desired time interval between two consecutive backups is defined using ISO8601 format. Frequency-based backup schedule supports interval resolution to the minute.
        ```json
        {
            "ScheduleKind": "FrequencyBased",
            "Interval": "PT10M"
        }
        ```

    2. **Time-based backup schedule**: This schedule type should be used if the need is to take data backup at specific times of the day or week. Schedule frequency type can either be daily or weekly.
        1. **_Daily_ Time-based backup schedule**: This schedule type should be used if the need is to take data backup at specific times of the day. To specify this, set `ScheduleFrequencyType` to _Daily_; and set `RunTimes` to list of desired time during the day in ISO8601 format, date specified along with time will be ignored. For example, `0001-01-01T18:00:00` represents _6:00 PM_ everyday, ignoring date part _0001-01-01_. Below example illustrates the configuration to trigger daily backup at _9:00 AM_ and _6:00 PM_ everyday.

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Daily",
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

        2. **_Weekly_ Time-based backup schedule**: This schedule type should be used if the need is to take data backup at specific times of the day. To specify this, set `ScheduleFrequencyType` to _Weekly_; set `RunDays` to list of days in a week when backup needs to be triggered and set `RunTimes` to list of desired time during the day in ISO8601 format, date specified along with time will be ignored. List of days of a week when to trigger the periodic backup. Below example illustrates the configuration to trigger daily backup at _9:00 AM_ and _6:00 PM_ during Monday to Friday.

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Weekly",
                "RunDays": [
                   "Monday",
                   "Tuesday",
                   "Wednesday",
                   "Thursday",
                   "Friday"
                ],
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

* **Backup storage**: Specifies the location to upload backups. Storage can be either Azure blob store or file share.
    1. **Azure blob store with managed identity**: This storage type should be selected when the need is to store generated backups in Azure. Both _standalone_ and _Cloud-based_ clusters can use this storage type. Description for this storage type requires BlobServiceUri and name of the container where backups need to be uploaded. If the container with the specified name isn't available, then it gets created during upload of a backup. Replace `account-name` with your storage account name.

        ```json
        {
            "StorageKind": "ManagedIdentityAzureBlobStore",
            "FriendlyName": "AzureMI_storagesample",
            "BlobServiceUri": "https://<account-name>.blob.core.windows.net",
            "ContainerName": "backup-container",
            "ManagedIdentityType": "VMSS",
            "ManagedIdentityClientId": "<Client-Id of User-Assigned MI>" 
        }
        ```
        
        > [NOTE]
        > Use Optional parameter `ManagedIdentityClientId` with Client-Id of User-Assigned Managed Identity in case of multiple User-Assigned Managed Identities assigned to your resource or both SAMI & UAMI assigned, and we need to use UAMI as the default, else no need of this paramter.

        follow steps for managed-identity assignment on Azure resource:

        1. Enable system assigned or User assigned managed identity in the VMSS [Configure managed identities on virtual machine scale set](/entra/identity/managed-identities-azure-resources/how-to-configure-managed-identities-scale-sets)

        2. Assign role to the VMSS managed identity to storage account by following the instructions starting in step two of [Assign Azure roles using the Azure portal - Azure RBAC](/azure/role-based-access-control/role-assignments-portal)
            * Storage Account Contributor, Storage Blob Data Contributor, and Storage Table Data Contributor Role at minimum

        [For more information on Managed Identity](/entra/identity/managed-identities-azure-resources/overview)
    
    2. **Azure blob store with ConnectionString**: This storage type should be selected when the need is to store generated backups in Azure. Both _standalone_ and _Cloud-based_ clusters can use this storage type. Description for this storage type requires connection string and name of the container where backups need to be uploaded. If the container with the specified name is not available, then it gets created during upload of a backup.

        ```json
        {
            "StorageKind": "AzureBlobStore",
            "FriendlyName": "Azure_storagesample",
            "ConnectionString": "<Put your Azure blob store connection string here>",
            "ContainerName": "backup-container"
        }
        ```

        > [!NOTE]
        > Backup restore Service does not work with v1 Azure storage
        > ConnectionString is not recommended in production as direct accessing to resource without user authentication

    3. **File share**: This storage type should be selected for _standalone_ clusters when the need is to store data backup on-premises. Description for this storage type requires file share path where backups need to be uploaded. Access to the file share can be configured using one of the following options
        1. _Integrated Windows Authentication_, where access to file share is provided to all computers belonging to the Service Fabric cluster. In this case, set following fields to configure _file-share_ based backup storage.

            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore"
            }
            ```

        2. _Protecting file share using username and password_, where access to file share is provided to specific users. File share storage specification also provides capability to specify secondary username and secondary password to provide fall-back credentials in case authentication fails with primary username and primary password. In this case, set following fields to configure _file-share_ based backup storage.

            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore",
                "PrimaryUserName": "backupaccount",
                "PrimaryPassword": "<Password for backupaccount>",
                "SecondaryUserName": "backupaccount2",
                "SecondaryPassword": "<Password for backupaccount2>"
            }
            ```

> [!NOTE]
> Ensure that the storage reliability meets or exceeds reliability requirements of backup data.
>

* **Retention Policy**: Specifies the policy to retain backups in the configured storage. Only Basic Retention Policy is supported.
    1. **Basic Retention Policy**: This retention policy allows to ensure optimal storage utilization by removing backup files, which are no longer required. `RetentionDuration` can be specified to set the time span for which backups are required to be retained in the storage. `MinimumNumberOfBackups` is an optional parameter that can be specified to make sure that the specified number of backups are always retained irrespective of the `RetentionDuration`. Below example illustrates the configuration to retain backups for _10_ days and doesn't allow number of backups to go below _20_.

        ```json
        {
            "RetentionPolicyType": "Basic",
            "RetentionDuration": "P10D",
            "MinimumNumberOfBackups": 20
        }
        ```

## Enable periodic backup
After defining backup policy to fulfill data backup requirements, the backup policy should be appropriately associated either with an _application_, or _service_, or a _partition_.

> [!NOTE]
> Ensure that there are no application upgrades in progress before enabling backup
>

### Hierarchical propagation of backup policy
In Service Fabric, the relation between application, service, and partitions is hierarchical as explained in [Application model](./service-fabric-application-model.md). Backup policy can be associated either with an _application_, _service_, or a _partition_ in the hierarchy. Backup policy propagates hierarchically to the next level. Assuming there's only one backup policy created and associated with an _application_, all stateful partitions belonging to all _Reliable stateful services_ and _Reliable Actors_ of the _application_ are backed-up using the backup policy. Or if the backup policy is associated with a _Reliable stateful service_, all its partitions are backed-up using the backup policy.

### Overriding backup policy
There may be a scenario where data backup with same backup schedule is required for all services of the application except for specific services where the need is to have data backup using higher frequency schedule or taking backup to a different storage account or fileshare. To address such scenarios, backup restore service provides facility to override propagated policy at service and partition scope. When the backup policy is associated at _service_ or _partition_, it overrides propagated backup policy, if any.

### Example

This example uses setup with two applications, _MyApp_A_ and _MyApp_B_. Application _MyApp_A_ contains two Reliable Stateful services, _SvcA1_ & _SvcA3_, and one Reliable Actor service, _ActorA2_. _SvcA1_ contains three partitions while _ActorA2_ and _SvcA3_ contain two partitions each.  Application _MyApp_B_ contains three Reliable Stateful services, _SvcB1_, _SvcB2_, and _SvcB3_. _SvcB1 and SvcB2 contain two partitions each while _SvcB3_ contains three partitions.

Assume that these applications' data backup requirements are as follows

1. MyApp_A
    1. Create daily backup of data for all partitions of all _Reliable Stateful services_ and _Reliable Actors_ belonging to the application. Upload backup data to location _BackupStore1_.

    2. One of the services, _SvcA3_, requires data backup every hour.

    3. Data size in partition _SvcA1_P2_ is more than expected and its backup data should be stored to different storage location _BackupStore2_.

2. MyApp_B
    1. Create backup of data every Sunday at 8:00 AM for all partitions of _SvcB1_ service. Upload backup data to location _BackupStore1_.

    2. Create backup of data every day at 8:00 AM for partition _SvcB2_P1_. Upload backup data to location _BackupStore1_.

To address these data backup requirements, backup policies BP_1 to BP_5 is created, and backup is enabled as follows.
1. MyApp_A
    1. Create backup policy, _BP_1_, with frequency-based backup schedule where frequency is set to 24 Hrs. and backup storage configured to use storage location _BackupStore1_. Enable this policy for Application _MyApp_A_ using [Enable Application Backup](/rest/api/servicefabric/sfclient-api-enableapplicationbackup) API. This action enables data backup using backup policy _BP_1_ for all partitions of _Reliable Stateful services_ and _Reliable Actors_ belonging to application _MyApp_A_.

    2. Create backup policy, _BP_2_, with frequency-based backup schedule where frequency is set to 1 Hrs. and backup storage configured to use storage location _BackupStore1_. Enable this policy for service _SvcA3_ using [Enable Service Backup](/rest/api/servicefabric/sfclient-api-enableservicebackup) API. This action overrides propagated policy _BP_1_ by explicitly enabled backup policy _BP_2_ for all partitions of service _SvcA3_ leading to data backup using backup policy _BP_2_ for these partitions.

    3. Create backup policy, _BP_3_, with frequency-based backup schedule where frequency is set to 24 Hrs. and backup storage configured to use storage location _BackupStore2_. Enable this policy for partition _SvcA1_P2_ using [Enable Partition Backup](/rest/api/servicefabric/sfclient-api-enablepartitionbackup) API. This action overrides propagated policy _BP_1_ by explicitly enabled backup policy _BP_3_ for partition _SvcA1_P2_.

2. MyApp_B
    1. Create backup policy, _BP_4_, with time-based backup schedule where schedule frequency type is set to weekly, run days is set to Sunday, and run times is set to 8:00 AM. Backup storage configured to use storage location _BackupStore1_. Enable this policy for service _SvcB1_ using [Enable Service Backup](/rest/api/servicefabric/sfclient-api-enableservicebackup) API. This action enables data backup using backup policy _BP_4_ for all partitions of service _SvcB1_.

    2. Create backup policy, _BP_5_, with time-based backup schedule where schedule frequency type is set to daily and run times is set to 8:00 AM. Backup storage configured to use storage location _BackupStore1_. Enable this policy for partition _SvcB2_P1_ using [Enable Partition Backup](/rest/api/servicefabric/sfclient-api-enablepartitionbackup) API. This action enables data backup using backup policy _BP_5_ for partition _SvcB2_P1_.

The following diagram depicts explicitly enabled backup policies and propagated backup policies.

![Service Fabric Application Hierarchy][0]

## Disable backup
Backup policies can be disabled when there's no need to backup data. Backup policy enabled at an _application_ can only be disabled at the same _application_ using [Disable Application Backup](/rest/api/servicefabric/sfclient-api-disableapplicationbackup) API, Backup policy enabled at a _service_ can be disabled at the same _service_ using [Disable Service Backup](/rest/api/servicefabric/sfclient-api-disableservicebackup) API, and Backup policy enabled at a _partition_ can be disabled at the same _partition_ using [Disable Partition Backup](/rest/api/servicefabric/sfclient-api-disablepartitionbackup) API.

* Disabling backup policy for an _application_ stops all periodic data backups happening as a result of propagation of the backup policy to Reliable Stateful service partitions or Reliable Actor partitions.

* Disabling backup policy for a _service_ stops all periodic data backups happening as a result of propagation of this backup policy to the partitions of the _service_.

* Disabling backup policy for a _partition_ stops all periodic data backup happening due to the backup policy at the partition.

* While disabling backup for an entity(application/service/partition), `CleanBackup` can be set to _true_ to delete all the backups in configured storage.
    ```json
    {
        "CleanBackup": true 
    }
    ```
> [!NOTE]
> Ensure that there are no application upgrades in progress before disabling backup
>

## Suspend & resume backup
Certain situations may demand temporary suspension of periodic backup of data. In such situations, depending on the requirement, suspend backup API may be used at an _Application_, _Service_, or _Partition_. Periodic backup suspension is transitive over subtree of the application's hierarchy from the point it's applied. 

* When suspension is applied at an _Application_ using [Suspend Application Backup](/rest/api/servicefabric/sfclient-api-suspendapplicationbackup) API, then all the services and partitions under this application are suspended for periodic backup of data.

* When suspension is applied at a _Service_ using [Suspend Service Backup](/rest/api/servicefabric/sfclient-api-suspendservicebackup) API, then all the partitions under this service are suspended for periodic backup of data.

* When suspension is applied at a _Partition_ using [Suspend Partition Backup](/rest/api/servicefabric/sfclient-api-suspendpartitionbackup) API, then it suspends partitions under this service are suspended for periodic backup of data.

Once the need for suspension is over, then the periodic data backup can be restored using respective resume backup API. Periodic backup must be resumed at same _application_, _service_, or _partition_ where it was suspended.

* If suspension was applied at an _Application_, then it should be resumed using [Resume Application Backup](/rest/api/servicefabric/sfclient-api-resumeapplicationbackup) API. 

* If suspension was applied at a _Service_, then it should be resumed using [Resume Service Backup](/rest/api/servicefabric/sfclient-api-resumeservicebackup) API.

* If suspension was applied at a _Partition_, then it should be resumed using [Resume Partition Backup](/rest/api/servicefabric/sfclient-api-resumepartitionbackup) API.

### Difference between Suspend and Disable backups
Disable backup should be used when backups are no longer required for a particular application, service, or partition. One can invoke disable backup request along with clean backups parameter to be true, which would mean all existing backups are deleted as well. However, suspend is to be used in scenarios where one wants to turn off backups temporarily like when local disk becomes full or uploading backup is failing due to known network issue etc. 

While disable can be invoked only at a level, which was earlier enabled for backup explicitly however suspension can be applied at any level, which is currently enabled for backup either directly or via inheritance/ hierarchy. For example, if backup is enabled at an application level, one can invoke disable only at the application level however suspend can be invoked at application, any service, or partition under that application. 

## Auto restore on data loss
The service partition may lose data due to unexpected failures. For example, the disk for two out of three replicas for a partition (including the primary replica) gets corrupted or wiped.

When Service Fabric detects that the partition is in data loss, it invokes `OnDataLossAsync` interface method on the partition and expects partition to take the required action to come out of data loss. In this situation, if the effective backup policy at the partition has `AutoRestoreOnDataLoss` flag set to `true` then the restore gets triggered automatically using latest available backup for this partition.

> [!NOTE]
> It is recommended to NOT set Auto Restore in production clusters
>

## Get backup configuration
Separate APIs are made available to get backup configuration information at an _application_, _service_, and _partition_ scope. [Get Application Backup Configuration Info](/rest/api/servicefabric/sfclient-api-getapplicationbackupconfigurationinfo), [Get Service Backup Configuration Info](/rest/api/servicefabric/sfclient-api-getservicebackupconfigurationinfo), and [Get Partition Backup Configuration Info](/rest/api/servicefabric/sfclient-api-getpartitionbackupconfigurationinfo) are these APIs respectively. Mainly, these APIs return the applicable backup policy, scope at which the backup policy is applied and backup suspension details. Following is brief description about returned results of these APIs.

- Application backup configuration info: provides the details of backup policy applied at application and all the overridden policies at services and partitions belonging to the application. It also includes the suspension information for the application and its services, and partitions.

- Service backup configuration info: provides the details of effective backup policy at service and the scope at which this policy was applied and all the overridden policies at its partitions. It also includes the suspension information for the service and its partitions.

- Partition backup configuration info: provides the details of effective backup policy at partition and the scope at which this policy was applied. It also includes the suspension information for the partitions.

## List available backups

Available backups can be listed using the Get Backup List API. The result of API call includes backup info items related to all the backups available at the backup storage, which is configured in the applicable backup policy. Different variants of this API are provided to list available backups belonging to an application, service, or partition. These APIs support getting the _latest_ available backup of all applicable partitions, or filtering of backups based on _start date_ and _end date_.

These APIs also support pagination of the results, when _MaxResults_ parameter is set to nonzero positive integer then the API returns maximum _MaxResults_ backup info items. In case, there are more backup info items available than the _MaxResults_ value, then a continuation token is returned. Valid continuation token parameter can be used to get the next set of results. When valid continuation token value is passed to next call of the API, the API returns next set of results. No continuation token is included in the response when all available results are returned.

Following is the brief information about supported variants.

- [Get Application Backup List](/rest/api/servicefabric/sfclient-api-getapplicationbackuplist): Returns a list of backups available for every partition belonging to given Service Fabric application.

- [Get Service Backup List](/rest/api/servicefabric/sfclient-api-getservicebackuplist): Returns a list of backups available for every partition belonging to given Service Fabric service.
 
- [Get Partition Backup List](/rest/api/servicefabric/sfclient-api-getpartitionbackuplist): Returns a list of backups available for the specified partition.

## Next steps
- [Backup restore REST API reference](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/backup-policy-association-example.png
