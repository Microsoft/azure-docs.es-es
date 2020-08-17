---
title: Compatibilidad de etiquetas de los recursos
description: Muestra los tipos de recursos de Azure que admiten etiquetas. Proporciona detalles de todos los servicios de Azure.
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 2ebf55a15233c401904460d459a118fc46142ac3
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87338000"
---
# <a name="tag-support-for-azure-resources"></a>Compatibilidad de etiquetas de los recursos de Azure
En este artículo se describe si un tipo de recurso admite [etiquetas](tag-resources.md). La columna con la etiqueta **Admite etiquetas** indica si el tipo de recurso tiene una propiedad para la etiqueta. La columna con la etiqueta **Etiqueta en el informe de costos** indica si ese tipo de recurso pasa la etiqueta al informe de costos. Puede ver los costos por etiquetas en el [análisis de costos de Cost Management](../../cost-management-billing/costs/group-filter.md) y los [datos de uso diario y de facturación de Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md).

Para obtener los mismos datos como un archivo de valores separados por comas, descargue [tag-support.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/tag-support.csv).

Vaya a un espacio de nombres del proveedor de recursos:
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.Azure.Geneva](#microsoftazuregeneva)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ChangeAnalysis](#microsoftchangeanalysis)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft. ConnectedCache](#microsoftconnectedcache)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataLakeAnalytics](#microsoftdatalakeanalytics)
> - [Microsoft.DataLakeStore](#microsoftdatalakestore)
> - [Microsoft.DataMigration](#microsoftdatamigration)
> - [Microsoft.DataProtection](#microsoftdataprotection)
> - [Microsoft.DataShare](#microsoftdatashare)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.DeploymentManager](#microsoftdeploymentmanager)
> - [Microsoft.DesktopVirtualization](#microsoftdesktopvirtualization)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.DynamicsLcs](#microsoftdynamicslcs)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Gallery](#microsoftgallery)
> - [Microsoft.Genomics](#microsoftgenomics)
> - [Microsoft.GuestConfiguration](#microsoftguestconfiguration)
> - [Microsoft.HanaOnAzure](#microsofthanaonazure)
> - [Microsoft.HardwareSecurityModules](#microsofthardwaresecuritymodules)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.HealthcareApis](#microsofthealthcareapis)
> - [Microsoft.HybridCompute](#microsofthybridcompute)
> - [Microsoft.HybridData](#microsofthybriddata)
> - [Microsoft.HybridNetwork](#microsofthybridnetwork)
> - [Microsoft.Hydra](#microsofthydra)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [Microsoft.Insights](#microsoftinsights)
> - [Microsoft.Intune](#microsoftintune)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.Maintenance](#microsoftmaintenance)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.ManagedNetwork](#microsoftmanagednetwork)
> - [Microsoft.ManagedServices](#microsoftmanagedservices)
> - [Microsoft.Management](#microsoftmanagement)
> - [Microsoft.Maps](#microsoftmaps)
> - [Microsoft.Marketplace](#microsoftmarketplace)
> - [Microsoft.MarketplaceApps](#microsoftmarketplaceapps)
> - [Microsoft.MarketplaceOrdering](#microsoftmarketplaceordering)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Microservices4Spring](#microsoftmicroservices4spring)
> - [Microsoft.Migrate](#microsoftmigrate)
> - [Microsoft.MixedReality](#microsoftmixedreality)
> - [Microsoft.NetApp](#microsoftnetapp)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.Notebooks](#microsoftnotebooks)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.ObjectStore](#microsoftobjectstore)
> - [Microsoft.OffAzure](#microsoftoffazure)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Peering](#microsoftpeering)
> - [Microsoft.PolicyInsights](#microsoftpolicyinsights)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.ProjectBabylon](#microsoftprojectbabylon)
> - [Microsoft.ProviderHub](#microsoftproviderhub)
> - [Microsoft.Quantum](#microsoftquantum)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.RedHatOpenShift](#microsoftredhatopenshift)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.ResourceHealth](#microsoftresourcehealth)
> - [Microsoft.Resources](#microsoftresources)
> - [Microsoft.SaaS](#microsoftsaas)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityGraph](#microsoftsecuritygraph)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.SQL](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageReplication](#microsoftstoragereplication)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VMwareOnAzure](#microsoftvmwareonazure)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsDefenderATP](#microsoftwindowsdefenderatp)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WorkloadBuilder](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | DomainServices | Sí | Sí |
> | DomainServices/oucontainer | No | No |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | supportProviders | No | No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | aadsupportcases | No | No |
> | addsservices | No | No |
> | agents | No | No |
> | anonymousapiusers | No | No |
> | configuración | No | No |
> | logs | No | No |
> | reports | No | No |
> | servicehealthmetrics | No | No |
> | services | No | No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | configuraciones | No | No |
> | generateRecommendations | No | No |
> | metadata | No | No |
> | de películas | No | No |
> | suppressions | No | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | actionRules | Sí | Sí |
> | alerts | No | No |
> | alertsList | No | No |
> | alertsMetaData | No | No |
> | alertsSummary | No | No |
> | alertsSummaryList | No | No |
> | smartDetectorAlertRules | Sí | Sí |
> | smartGroups | No | No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | servers | Sí | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | reportFeedback | No | No |
> | service | Sí | Sí |
> | validateServiceName | No | No |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | configurationStores | Sí | Sí |
> | configurationStores/eventGridFilters | No | No |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Spring | Sí | Sí |
> | Spring/apps | No | No |
> | Spring/apps/deployments | No | No |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | attestationProviders | Sí | Sí |
> | defaultProviders | No | No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | classicAdministrators | No | No |
> | dataAliases | No | No |
> | denyAssignments | No | No |
> | elevateAccess | No | No |
> | findOrphanRoleAssignments | No | No |
> | locks | No | No |
> | permisos | No | No |
> | policyAssignments | No | No |
> | policyDefinitions | No | No |
> | policySetDefinitions | No | No |
> | privateLinkAssociations | No | No |
> | providerOperations | No | No |
> | resourceManagementPrivateLinks | No | No |
> | roleAssignments | No | No |
> | roleAssignmentsUsageMetrics | No | No |
> | roleDefinitions | No | No |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | automationAccounts | Sí | Sí |
> | automationAccounts/configurations | Sí | Sí |
> | automationAccounts/jobs | No | No |
> | automationAccounts / privateEndpointConnectionProxies | No | No |
> | automationAccounts / privateEndpointConnections | No | No |
> | automationAccounts / privateLinkResources | No | No |
> | automationAccounts/runbooks | Sí | Sí |
> | automationAccounts/softwareUpdateConfigurations | No | No |
> | automationAccounts/webhooks | No | No |

>[!NOTE]
>Azure Automation solo admite la creación de un máximo de 15 pares de nombre/valor de etiqueta para cada recurso de Automation.
> 

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | privateClouds | Sí | Sí |
> | privateClouds / authorizations | No | No |
> | privateClouds / clusters | No | No |
> | privateClouds / hcxEnterpriseSites | No | No |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | environments | No | No |
> | environments/accounts | No | No |
> | environments/accounts/namespaces | No | No |
> | environments/accounts/namespaces/configurations | No | No |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | b2cDirectories | Sí | No |
> | b2ctenants | No | No |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | dataControllers | Sí | Sí |
> | hybridDataManagers | Sí | Sí |
> | postgresInstances | Sí | Sí |
> | sqlInstances | Sí | Sí |
> | sqlManagedInstances | Sí | Sí |
> | sqlServerInstances | Sí | Sí |
> | sqlServerRegistrations | Sí | Sí |
> | sqlServerRegistrations/sqlServers | No | No |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | cloudManifestFiles | No | No |
> | edgeSubscriptions | Sí | Sí |
> | registrations | Sí | Sí |
> | registrations/customerSubscriptions | No | No |
> | registrations/products | No | No |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | batchAccounts | Sí | Sí |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | billingAccounts | No | No |
> | billingAccounts/agreements | No | No |
> | billingAccounts/billingPermissions | No | No |
> | billingAccounts/billingProfiles | No | No |
> | billingAccounts/billingProfiles/billingPermissions | No | No |
> | billingAccounts/billingProfiles/billingRoleAssignments | No | No |
> | billingAccounts/billingProfiles/billingRoleDefinitions | No | No |
> | billingAccounts/billingProfiles/billingSubscriptions | No | No |
> | billingAccounts/billingProfiles/createBillingRoleAssignment | No | No |
> | billingAccounts/billingProfiles/customers | No | No |
> | billingAccounts / billingProfiles / instructions | No | No |
> | billingAccounts/billingProfiles/invoices | No | No |
> | billingAccounts/billingProfiles/invoices/pricesheet | No | No |
> | billingAccounts/billingProfiles/invoices/transactions | No | No |
> | billingAccounts/billingProfiles/invoiceSections | No | No |
> | billingAccounts/billingProfiles/invoiceSections/billingPermissions | No | No |
> | billingAccounts/billingProfiles/invoiceSections/billingRoleAssignments | No | No |
> | billingAccounts/billingProfiles/invoiceSections/billingRoleDefinitions | No | No |
> | billingAccounts/billingProfiles/invoiceSections/billingSubscriptions | No | No |
> | billingAccounts/billingProfiles/invoiceSections/createBillingRoleAssignment | No | No |
> | billingAccounts/billingProfiles/invoiceSections/initiateTransfer | No | No |
> | billingAccounts/billingProfiles/invoiceSections/products | No | No |
> | billingAccounts/billingProfiles/invoiceSections/products/transfer | No | No |
> | billingAccounts/billingProfiles/invoiceSections/products/updateAutoRenew | No | No |
> | billingAccounts/billingProfiles/invoiceSections/transactions | No | No |
> | billingAccounts/billingProfiles/invoiceSections/transfers | No | No |
> | billingAccounts/BillingProfiles/patchOperations | No | No |
> | billingAccounts/billingProfiles/paymentMethods | No | No |
> | billingAccounts/billingProfiles/policies | No | No |
> | billingAccounts/billingProfiles/pricesheet | No | No |
> | billingAccounts/billingProfiles/pricesheetDownloadOperations | No | No |
> | billingAccounts/billingProfiles/products | No | No |
> | billingAccounts/billingProfiles/transactions | No | No |
> | billingAccounts/billingRoleAssignments | No | No |
> | billingAccounts/billingRoleDefinitions | No | No |
> | billingAccounts/billingSubscriptions | No | No |
> | billingAccounts/billingSubscriptions/invoices | No | No |
> | billingAccounts/createBillingRoleAssignment | No | No |
> | billingAccounts/createInvoiceSectionOperations | No | No |
> | billingAccounts/customers | No | No |
> | billingAccounts/customers/billingPermissions | No | No |
> | billingAccounts/customers/billingSubscriptions | No | No |
> | billingAccounts/customers/initiateTransfer | No | No |
> | billingAccounts/customers/policies | No | No |
> | billingAccounts/customers/products | No | No |
> | billingAccounts/customers/transactions | No | No |
> | billingAccounts/customers/transfers | No | No |
> | billingAccounts/departments | No | No |
> | billingAccounts / departments / billingPermissions | No | No |
> | billingAccounts / departments / billingRoleAssignments | No | No |
> | billingAccounts / departments / billingRoleDefinitions | No | No |
> | billingAccounts/enrollmentAccounts | No | No |
> | billingAccounts / enrollmentAccounts / billingPermissions | No | No |
> | billingAccounts / enrollmentAccounts / billingRoleAssignments | No | No |
> | billingAccounts / enrollmentAccounts / billingRoleDefinitions | No | No |
> | billingAccounts/invoices | No | No |
> | billingAccounts / invoices / transactions | No | No |
> | billingAccounts/invoiceSections | No | No |
> | billingAccounts/invoiceSections/billingSubscriptionMoveOperations | No | No |
> | billingAccounts/invoiceSections/billingSubscriptions | No | No |
> | billingAccounts/invoiceSections/billingSubscriptions/transfer | No | No |
> | billingAccounts/invoiceSections/elevate | No | No |
> | billingAccounts/invoiceSections/initiateTransfer | No | No |
> | billingAccounts/invoiceSections/patchOperations | No | No |
> | billingAccounts/invoiceSections/productMoveOperations | No | No |
> | billingAccounts/invoiceSections/products | No | No |
> | billingAccounts/invoiceSections/products/transfer | No | No |
> | billingAccounts/invoiceSections/products/updateAutoRenew | No | No |
> | billingAccounts/invoiceSections/transactions | No | No |
> | billingAccounts/invoiceSections/transfers | No | No |
> | billingAccounts/lineOfCredit | No | No |
> | billingAccounts/patchOperations | No | No |
> | billingAccounts/paymentMethods | No | No |
> | billingAccounts/products | No | No |
> | billingAccounts/transactions | No | No |
> | billingPeriods | No | No |
> | billingPermissions | No | No |
> | billingProperty | No | No |
> | billingRoleAssignments | No | No |
> | billingRoleDefinitions | No | No |
> | createBillingRoleAssignment | No | No |
> | departments | No | No |
> | enrollmentAccounts | No | No |
> | invoices | No | No |
> | transfers | No | No |
> | transfers/acceptTransfer | No | No |
> | transfers/declineTransfer | No | No |
> | transfers/operationStatus | No | No |
> | transfers/validateTransfer | No | No |
> | validateAddress | No | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | mapApis | Sí | Sí |
> | updateCommunicationPreference | No | No |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | blockchainMembers | Sí | Sí |
> | cordaMembers | Sí | Sí |
> | watchers | Sí | Sí |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | TokenServices | Sí | Sí |
> | TokenServices/BlockchainNetworks | No | No |
> | TokenServices/Groups | No | No |
> | TokenServices/Groups/Accounts | No | No |
> | TokenServices/TokenTemplates | No | No |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | blueprintAssignments | No | No |
> | blueprintAssignments/assignmentOperations | No | No |
> | blueprintAssignments/operations | No | No |
> | blueprints | No | No |
> | blueprints/artifacts | No | No |
> | blueprints/versions | No | No |
> | blueprints/versions/artifacts | No | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | botServices | Sí | Sí |
> | botServices/channels | No | No |
> | botServices/connections | No | No |
> | languages | No | No |
> | plantillas | No | No |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Redis | Sí | Sí |
> | Redis / EventGridFilters | No | No |
> | Redis / privateEndpointConnectionProxies | No | No |
> | Redis / privateEndpointConnectionProxies / validate | No | No |
> | Redis / privateEndpointConnections | No | No |
> | Redis / privateLinkResources | No | No |
> | redisEnterprise | Sí | Sí |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | appliedReservations | No | No |
> | autoQuotaIncrease | No | No |
> | calculateExchange | No | No |
> | calculatePrice | No | No |
> | calculatePurchasePrice | No | No |
> | catalogs | No | No |
> | commercialReservationOrders | No | No |
> | cambio | No | No |
> | placePurchaseOrder | No | No |
> | reservationOrders | No | No |
> | reservationOrders/calculateRefund | No | No |
> | reservationOrders/merge | No | No |
> | reservationOrders/reservations | No | No |
> | reservationOrders/reservations/revisions | No | No |
> | reservationOrders/return | No | No |
> | reservationOrders/split | No | No |
> | reservationOrders/swap | No | No |
> | reservations | No | No |
> | resourceProviders | No | No |
> | resources | No | No |
> | validateReservationOrder | No | No |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | CdnWebApplicationFirewallManagedRuleSets | No | No |
> | CdnWebApplicationFirewallPolicies | Sí | Sí |
> | edgenodes | No | No |
> | profiles | Sí | Sí |
> | profiles/endpoints | Sí | Sí |
> | profiles/endpoints/customdomains | No | No |
> | profiles/endpoints/origingroups | No | No |
> | profiles/endpoints/origins | No | No |
> | validateProbe | No | No |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | certificateOrders | Sí | Sí |
> | certificateOrders/certificates | No | No |
> | validateCertificateRegistrationInformation | No | No |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | perfile | No | No |
> | resourceChanges | No | No |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | capabilities | No | No |
> | domainNames | No | No |
> | domainNames/capabilities | No | No |
> | domainNames/internalLoadBalancers | No | No |
> | domainNames/serviceCertificates | No | No |
> | domainNames/slots | No | No |
> | domainNames/slots/roles | No | No |
> | domainNames/slots/roles/metricDefinitions | No | No |
> | domainNames/slots/roles/metrics | No | No |
> | moveSubscriptionResources | No | No |
> | operatingSystemFamilies | No | No |
> | operatingSystems | No | No |
> | quotas | No | No |
> | resourceTypes | No | No |
> | validateSubscriptionMoveAvailability | No | No |
> | virtualMachines | No | No |
> | virtualMachines/diagnosticSettings | No | No |
> | virtualMachines/metricDefinitions | No | No |
> | virtualMachines/metrics | No | No |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | classicInfrastructureResources | No | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | capabilities | No | No |
> | expressRouteCrossConnections | No | No |
> | expressRouteCrossConnections/peerings | No | No |
> | gatewaySupportedDevices | No | No |
> | networkSecurityGroups | No | No |
> | quotas | No | No |
> | reservedIps | No | No |
> | virtualNetworks | No | No |
> | virtualNetworks/remoteVirtualNetworkPeeringProxies | No | No |
> | virtualNetworks/virtualNetworkPeerings | No | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | capabilities | No | No |
> | disks | No | No |
> | images | No | No |
> | osImages | No | No |
> | osPlatformImages | No | No |
> | publicImages | No | No |
> | quotas | No | No |
> | storageAccounts | No | No |
> | storageAccounts/blobServices | No | No |
> | storageAccounts/fileServices | No | No |
> | storageAccounts/metricDefinitions | No | No |
> | storageAccounts/metrics | No | No |
> | storageAccounts/queueServices | No | No |
> | storageAccounts/services | No | No |
> | storageAccounts/services/diagnosticSettings | No | No |
> | storageAccounts/services/metricDefinitions | No | No |
> | storageAccounts/services/metrics | No | No |
> | storageAccounts/tableServices | No | No |
> | storageAccounts/vmImages | No | No |
> | vmImages | No | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | RateCard | No | No |
> | UsageAggregates | No | No |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | availabilitySets | Sí | Sí |
> | diskAccesses | Sí | Sí |
> | diskEncryptionSets | Sí | Sí |
> | disks | Sí | Sí |
> | galleries | Sí | Sí |
> | galleries/applications | No | No |
> | galleries/applications/versions | No | No |
> | galleries/images | No | No |
> | galleries/images/versions | No | No |
> | hostGroups | Sí | Sí |
> | hostGroups/hosts | Sí | Sí |
> | images | Sí | Sí |
> | proximityPlacementGroups | Sí | Sí |
> | restorePointCollections | Sí | Sí |
> | restorePointCollections/restorePoints | No | No |
> | sharedVMExtensions | Sí | Sí |
> | sharedVMExtensions/versions | No | No |
> | sharedVMImages | Sí | Sí |
> | sharedVMImages/versions | No | No |
> | snapshots | Sí | Sí |
> | sshPublicKeys | Sí | Sí |
> | virtualMachines | Sí | Sí |
> | virtualMachines/extensions | Sí | Sí |
> | virtualMachines/metricDefinitions | No | No |
> | virtualMachines / runCommands | Sí | Sí |
> | virtualMachineScaleSets | Sí | Sí |
> | virtualMachineScaleSets/extensions | No | No |
> | virtualMachineScaleSets/networkInterfaces | No | No |
> | virtualMachineScaleSets/publicIPAddresses | No | No |
> | virtualMachineScaleSets/virtualMachines | No | No |
> | virtualMachineScaleSets/virtualMachines/networkInterfaces | No | No |

> [!NOTE]
> No se puede aregar una etiqueta a una máquina virtual marcada como generalizada. Un máquina virtual se marca como generalizada con [Set-AzVm-Generalized](/powershell/module/Az.Compute/Set-AzVM) o [az vm generalize](/cli/azure/vm#az-vm-generalize).

## <a name="microsoftconnectedcache"></a>Microsoft.ConnectedCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | CacheNodes | Sí | Sí |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | AggregatedCost | No | No |
> | Saldos | No | No |
> | Presupuestos | No | No |
> | Charges | No | No |
> | CostTags | No | No |
> | credits | No | No |
> | events | No | No |
> | Previsiones | No | No |
> | lots | No | No |
> | Catálogos de soluciones | No | No |
> | Pricesheets | No | No |
> | products | No | No |
> | ReservationDetails | No | No |
> | ReservationRecommendationDetails | No | No |
> | ReservationRecommendations | No | No |
> | ReservationSummaries | No | No |
> | ReservationTransactions | No | No |
> | Etiquetas | No | No |
> | tenants | No | No |
> | Términos | No | No |
> | UsageDetails | No | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | containerGroups | Sí | Sí |
> | serviceAssociationLinks | No | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | registries | Sí | Sí |
> | registries/agentPools | Sí | Sí |
> | registries/builds | No | No |
> | registries/builds/cancel | No | No |
> | registries/builds/getLogLink | No | No |
> | registries/buildTasks | Sí | Sí |
> | registries/buildTasks/steps | No | No |
> | registries/eventGridFilters | No | No |
> | registries / exportPipelines | No | No |
> | registries/generateCredentials | No | No |
> | registries/getBuildSourceUploadUrl | No | No |
> | registries/GetCredentials | No | No |
> | registries/importImage | No | No |
> | registries / importPipelines | No | No |
> | registries / pipelineRuns | No | No |
> | registries/privateEndpointConnectionProxies | No | No |
> | registries/privateEndpointConnectionProxies/validate | No | No |
> | registries / privateEndpointConnections | No | No |
> | registries/privateLinkResources | No | No |
> | registries/queueBuild | No | No |
> | registries/regenerateCredential | No | No |
> | registries/regenerateCredentials | No | No |
> | registries/replications | Sí | Sí |
> | registries/runs | No | No |
> | registries/runs/cancel | No | No |
> | registries/scheduleRun | No | No |
> | registries/scopeMaps | No | No |
> | registries/taskRuns | No | No |
> | registries/tasks | Sí | Sí |
> | registries/tokens | No | No |
> | registries/updatePolicies | No | No |
> | registries/webhooks | Sí | Sí |
> | registries/webhooks/getCallbackConfig | No | No |
> | registries/webhooks/ping | No | No |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | containerServices | Sí | Sí |
> | managedClusters | Sí | Sí |
> | openShiftManagedClusters | Sí | Sí |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Alertas | No | No |
> | BillingAccounts | No | No |
> | Presupuestos | No | No |
> | CloudConnectors | No | No |
> | Conectores | Sí | Sí |
> | costAllocationRules | No | No |
> | Departments | No | No |
> | Dimensions | No | No |
> | EnrollmentAccounts | No | No |
> | Exports | No | No |
> | ExternalBillingAccounts | No | No |
> | ExternalBillingAccounts/Alerts | No | No |
> | ExternalBillingAccounts/Dimensions | No | No |
> | ExternalBillingAccounts/Forecast | No | No |
> | ExternalBillingAccounts/Query | No | No |
> | ExternalSubscriptions | No | No |
> | ExternalSubscriptions/Alerts | No | No |
> | ExternalSubscriptions/Dimensions | No | No |
> | ExternalSubscriptions/Forecast | No | No |
> | ExternalSubscriptions/Query | No | No |
> | Forecast | No | No |
> | Consultar | No | No |
> | registro | No | No |
> | Reportconfigs | No | No |
> | Informes | No | No |
> | Configuración | No | No |
> | showbackRules | No | No |
> | Vistas | No | No |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Solicitudes | No | No |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | associations | No | No |
> | resourceProviders | Sí | Sí |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | jobs | Sí | Sí |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | DataBoxEdgeDevices | Sí | Sí |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | workspaces | Sí | No |
> | workspaces / dbWorkspaces | No | No |
> | workspaces / storageEncryption | No | No |
> | workspaces/virtualNetworkPeerings | No | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | catalogs | Sí | Sí |
> | datacatalogs | Sí | Sí |
> | datacatalogs/datasources | No | No |
> | datacatalogs/datasources/scans | No | No |
> | datacatalogs/datasources/scans/datasets | No | No |
> | datacatalogs/datasources/scans/triggers | No | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | dataFactories | Sí | No |
> | dataFactories/diagnosticSettings | No | No |
> | dataFactories/metricDefinitions | No | No |
> | dataFactorySchema | No | No |
> | factories | Sí | No |
> | factories/integrationRuntimes | No | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |
> | accounts/dataLakeStoreAccounts | No | No |
> | accounts/storageAccounts | No | No |
> | accounts/storageAccounts/containers | No | No |
> | accounts/transferAnalyticsUnits | No | No |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |
> | accounts/eventGridFilters | No | No |
> | accounts/firewallRules | No | No |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | services | No | No |
> | services/projects | No | No |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | BackupVaults | Sí | Sí |
> | ResourceOperationGateKeepers | Sí | Sí |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |
> | accounts/shares | No | No |
> | accounts/shares/datasets | No | No |
> | accounts/shares/invitations | No | No |
> | accounts/shares/providersharesubscriptions | No | No |
> | accounts/shares/synchronizationSettings | No | No |
> | accounts/sharesubscriptions | No | No |
> | accounts/sharesubscriptions/consumerSourceDataSets | No | No |
> | accounts/sharesubscriptions/datasetmappings | No | No |
> | accounts/sharesubscriptions/triggers | No | No |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/keys | No | No |
> | servers/privateEndpointConnectionProxies | No | No |
> | servers/privateEndpointConnections | No | No |
> | servers/privateLinkResources | No | No |
> | servers/queryTexts | No | No |
> | servers/recoverableServers | No | No |
> | servers/topQueryStatistics | No | No |
> | servers/virtualNetworkRules | No | No |
> | servers/waitStatistics | No | No |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/keys | No | No |
> | servers/privateEndpointConnectionProxies | No | No |
> | servers/privateEndpointConnections | No | No |
> | servers/privateLinkResources | No | No |
> | servers/queryTexts | No | No |
> | servers/recoverableServers | No | No |
> | servers/topQueryStatistics | No | No |
> | servers/virtualNetworkRules | No | No |
> | servers/waitStatistics | No | No |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | serverGroups | Sí | Sí |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/keys | No | No |
> | servers/privateEndpointConnectionProxies | No | No |
> | servers/privateEndpointConnections | No | No |
> | servers/privateLinkResources | No | No |
> | servers/queryTexts | No | No |
> | servers/recoverableServers | No | No |
> | servers/topQueryStatistics | No | No |
> | servers/virtualNetworkRules | No | No |
> | servers/waitStatistics | No | No |
> | serversv2 | Sí | Sí |
> | singleServers | Sí | Sí |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | artifactSources | Sí | Sí |
> | rollouts | Sí | Sí |
> | serviceTopologies | Sí | Sí |
> | serviceTopologies/services | Sí | Sí |
> | serviceTopologies/services/serviceUnits | Sí | Sí |
> | steps | Sí | Sí |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applicationgroups | Sí | Sí |
> | applicationgroups/applications | No | No |
> | applicationgroups/desktops | No | No |
> | applicationgroups/startmenuitems | No | No |
> | hostpools | Sí | Sí |
> | hostpools/sessionhosts | No | No |
> | hostpools/sessionhosts/usersessions | No | No |
> | hostpools/usersessions | No | No |
> | workspaces | Sí | Sí |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | ElasticPools | Sí | Sí |
> | ElasticPools/IotHubTenants | Sí | Sí |
> | ElasticPools / IotHubTenants / securitySettings | No | No |
> | IotHubs | Sí | Sí |
> | IotHubs/eventGridFilters | No | No |
> | IotHubs / securitySettings | No | No |
> | ProvisioningServices | Sí | Sí |
> | usages | No | No |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | pipelines | Sí | Sí |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | controllers | Sí | Sí |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | labcenters | Sí | Sí |
> | labs | Sí | Sí |
> | labs/environments | Sí | Sí |
> | labs/serviceRunners | Sí | Sí |
> | labs/virtualMachines | Sí | Sí |
> | schedules | Sí | Sí |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | digitalTwinsInstances | Sí | Sí |
> | digitalTwinsInstances / endpoints | No | No |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | databaseAccountNames | No | No |
> | databaseAccounts | Sí | Sí |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | domains | Sí | Sí |
> | domains/domainOwnershipIdentifiers | No | No |
> | generateSsoRequest | No | No |
> | topLevelDomains | No | No |
> | validateDomainRegistrationInformation | No | No |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | lcsprojects | No | No |
> | lcsprojects/clouddeployments | No | No |
> | lcsprojects/connectors | No | No |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | services | Sí | Sí |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | domains | Sí | Sí |
> | domains/topics | No | No |
> | eventSubscriptions | No | No |
> | extensionTopics | No | No |
> | partnerNamespaces | Sí | Sí |
> | partnerNamespaces / eventChannels | No | No |
> | partnerRegistrations | Sí | Sí |
> | partnerTopics | Sí | Sí |
> | partnerTopics / eventSubscriptions | No | No |
> | systemTopics | Sí | Sí |
> | systemTopics / eventSubscriptions | No | No |
> | topics | Sí | Sí |
> | topicTypes | No | No |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/disasterrecoveryconfigs | No | No |
> | namespaces/eventhubs | No | No |
> | namespaces/eventhubs/authorizationrules | No | No |
> | namespaces/eventhubs/consumergroups | No | No |
> | namespaces/networkrulesets | No | No |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | experimentWorkspaces | Sí | Sí |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | espacios de nombres | Sí | Sí |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | featureProviders | No | No |
> | features | No | No |
> | providers | No | No |
> | subscriptionFeatureRegistrations | No | No |

## <a name="microsoftgallery"></a>Microsoft.Gallery

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | enroll | No | No |
> | galleryitems | No | No |
> | generateartifactaccessuri | No | No |
> | myareas | No | No |
> | myareas/areas | No | No |
> | myareas/areas/areas | No | No |
> | myareas/areas/areas/galleryitems | No | No |
> | myareas/areas/galleryitems | No | No |
> | myareas/galleryitems | No | No |
> | registro | No | No |
> | resources | No | No |
> | retrieveresourcesbyid | No | No |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | autoManagedAccounts | Sí | Sí |
> | autoManagedVmConfigurationProfiles | Sí | Sí |
> | configurationProfileAssignments | No | No |
> | guestConfigurationAssignments | No | No |
> | software | No | No |
> | softwareUpdateProfile | No | No |
> | softwareUpdates | No | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | hanaInstances | Sí | Sí |
> | sapMonitors | Sí | Sí |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | dedicatedHSMs | Sí | Sí |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |
> | clusters/applications | No | No |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | services | Sí | Sí |
> | services / iomtconnectors | No | No |
> | services / iomtconnectors / connections | No | No |
> | services / iomtconnectors / mappings | No | No |
> | services / privateEndpointConnectionProxies | No | No |
> | services / privateEndpointConnections | Sí | Sí |
> | services / privateLinkResources | Sí | Sí |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | machines | Sí | Sí |
> | machines/extensions | Sí | Sí |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | dataManagers | Sí | Sí |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | devices | Sí | Sí |
> | registeredSubscriptions | No | No |
> | vendors | No | No |
> | vendors / skus | No | No |
> | vendors/vnfs | No | No |
> | virtualNetworkFunctionSkus | No | No |
> | vnfs | Sí | Sí |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | components | Sí | Sí |
> | networkScopes | Sí | Sí |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | jobs | Sí | Sí |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | actionGroups | Sí | Sí |
> | activityLogAlerts | Sí | Sí |
> | alertrules | Sí | Sí |
> | autoscalesettings | Sí | Sí |
> | components | Sí | Sí |
> | components/linkedStorageAccounts | No | No |
> | components/ProactiveDetectionConfigs | No | No |
> | diagnosticSettings | No | No |
> | guestDiagnosticSettings | Sí | Sí |
> | guestDiagnosticSettingsAssociation | Sí | Sí |
> | logprofiles | Sí | Sí |
> | metricAlerts | Sí | Sí |
> | privateLinkScopes | Sí | Sí |
> | privateLinkScopes/privateEndpointConnections | No | No |
> | privateLinkScopes/scopedResources | No | No |
> | queryPacks | Sí | Sí |
> | queryPacks/queries | No | No |
> | scheduledQueryRules | Sí | Sí |
> | webtests | Sí | Sí |
> | workbooks | Sí | Sí |
> | workbooktemplates | Sí | Sí |

## <a name="microsoftintune"></a>Microsoft.Intune

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | No | No |
> | diagnosticSettingsCategories | No | No |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | appTemplates | No | No |
> | IoTApps | Sí | Sí |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Grafo | Sí | Sí |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | deletedVaults | No | No |
> | hsmPools | Sí | Sí |
> | managedHSMs | Sí | Sí |
> | vaults | Sí | Sí |
> | vaults/accessPolicies | No | No |
> | vaults/eventGridFilters | No | No |
> | vaults/secrets | No | No |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | connectedClusters | Sí | Sí |
> | registeredSubscriptions | No | No |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | sourceControlConfigurations | No | No |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |
> | clusters/attacheddatabaseconfigurations | No | No |
> | clusters/databases | No | No |
> | clusters/databases/dataconnections | No | No |
> | clusters/databases/eventhubconnections | No | No |
> | clusters / databases / principalassignments | No | No |
> | clusters/dataconnections | No | No |
> | clusters / principalassignments | No | No |
> | clusters/sharedidentities | No | No |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | labaccounts | Sí | Sí |
> | users | No | No |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | hostingEnvironments | Sí | Sí |
> | integrationAccounts | Sí | Sí |
> | integrationServiceEnvironments | Sí | Sí |
> | integrationServiceEnvironments/managedApis | No | No |
> | isolatedEnvironments | Sí | Sí |
> | workflows | Sí | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | commitmentPlans | Sí | Sí |
> | webServices | Sí | Sí |
> | Áreas de trabajo | Sí | Sí |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | workspaces | Sí | Sí |
> | workspaces/computes | No | No |
> | workspaces/eventGridFilters | No | No |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applyUpdates | No | No |
> | configurationAssignments | No | No |
> | maintenanceConfigurations | Sí | Sí |
> | updates | No | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Identities | No | No |
> | userAssignedIdentities | Sí | Sí |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | managedNetworks | Sí | Sí |
> | managedNetworks / managedNetworkGroups | Sí | Sí |
> | managedNetworks / managedNetworkPeeringPolicies | Sí | Sí |
> | notificación | Sí | Sí |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | marketplaceRegistrationDefinitions | No | No |
> | registrationAssignments | No | No |
> | registrationDefinitions | No | No |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | getEntities | No | No |
> | managementGroups | No | No |
> | managementGroups / settings | No | No |
> | resources | No | No |
> | startTenantBackfill | No | No |
> | tenantBackfillStatus | No | No |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |
> | accounts/eventGridFilters | No | No |
> | accounts/privateAtlases | Sí | Sí |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | offers | No | No |
> | offerTypes | No | No |
> | offerTypes/publishers | No | No |
> | offerTypes/publishers/offers | No | No |
> | offerTypes/publishers/offers/plans | No | No |
> | offerTypes/publishers/offers/plans/agreements | No | No |
> | offerTypes/publishers/offers/plans/configs | No | No |
> | offerTypes/publishers/offers/plans/configs/importImage | No | No |
> | privategalleryitems | No | No |
> | privateStoreClient | No | No |
> | privateStores | No | No |
> | privateStores/offers | No | No |
> | products | No | No |
> | publishers | No | No |
> | publishers/offers | No | No |
> | publishers/offers/amendments | No | No |
> | registro | No | No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | classicDevServices | Sí | Sí |
> | updateCommunicationPreference | No | No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | agreements | No | No |
> | offertypes | No | No |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | mediaservices | Sí | Sí |
> | mediaservices/accountFilters | No | No |
> | mediaservices/assets | No | No |
> | mediaservices/assets/assetFilters | No | No |
> | mediaservices/contentKeyPolicies | No | No |
> | mediaservices/eventGridFilters | No | No |
> | mediaservices/liveEventOperations | No | No |
> | mediaservices/liveEvents | Sí | Sí |
> | mediaservices/liveEvents/liveOutputs | No | No |
> | mediaservices / liveEvents / privateEndpointConnectionProxies | No | No |
> | mediaservices/liveOutputOperations | No | No |
> | mediaservices/mediaGraphs | No | No |
> | mediaservices/streamingEndpointOperations | No | No |
> | mediaservices/streamingEndpoints | Sí | Sí |
> | mediaservices / streamingEndpoints / privateEndpointConnectionProxies | No | No |
> | mediaservices/streamingLocators | No | No |
> | mediaservices/streamingPolicies | No | No |
> | mediaservices / streamingPrivateEndpointConnectionProxyOperations | No | No |
> | mediaservices/transforms | No | No |
> | mediaservices/transforms/jobs | No | No |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | appClusters | Sí | Sí |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | assessmentProjects | Sí | Sí |
> | migrateprojects | Sí | Sí |
> | moveCollections | Sí | Sí |
> | projects | Sí | Sí |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | holographicsBroadcastAccounts | Sí | Sí |
> | objectUnderstandingAccounts | Sí | Sí |
> | remoteRenderingAccounts | Sí | Sí |
> | spatialAnchorsAccounts | Sí | Sí |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | netAppAccounts | Sí | No |
> | netAppAccounts/accountBackups | No | No |
> | netAppAccounts/capacityPools | Sí | No |
> | netAppAccounts/capacityPools/volumes | Sí | No |
> | netAppAccounts/capacityPools/volumes/snapshots | No | No |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applicationGateways | Sí | Sí |
> | applicationGatewayWebApplicationFirewallPolicies | Sí | Sí |
> | applicationSecurityGroups | Sí | Sí |
> | azureFirewallFqdnTags | No | No |
> | azureFirewalls | Sí | No |
> | bastionHosts | Sí | No |
> | bgpServiceCommunities | No | No |
> | connections | Sí | Sí |
> | ddosCustomPolicies | Sí | Sí |
> | ddosProtectionPlans | Sí | Sí |
> | dnsOperationStatuses | No | Sin |
> | dnszones | Sí | Sí |
> | dnszones/A | No | Sin |
> | dnszones/AAAA | No | Sin |
> | dnszones/all | No | Sin |
> | dnszones/CAA | No | Sin |
> | dnszones/CNAME | No | Sin |
> | dnszones/MX | No | Sin |
> | dnszones/NS | No | Sin |
> | dnszones/PTR | No | Sin |
> | dnszones/recordsets | No | Sin |
> | dnszones/SOA | No | Sin |
> | dnszones/SRV | No | Sin |
> | dnszones/TXT | No | No |
> | expressRouteCircuits | Sí | Sí |
> | expressRouteCrossConnections | Sí | Sí |
> | expressRouteGateways | Sí | Sí |
> | expressRoutePorts | Sí | Sí |
> | expressRouteServiceProviders | No | Sin |
> | firewallPolicies | Sí | Sí |
> | frontdoors | Sí, pero con límites (consulte la [nota siguiente](#frontdoor)) | Sí |
> | frontdoorWebApplicationFirewallManagedRuleSets | Sí, pero con límites (consulte la [nota siguiente](#frontdoor)) | No |
> | frontdoorWebApplicationFirewallPolicies | Sí, pero con límites (consulte la [nota siguiente](#frontdoor)) | Sí |
> | getDnsResourceReference | No | Sin |
> | internalNotify | No | Sin |
> | loadBalancers | Sí | Sí |
> | localNetworkGateways | Sí | Sí |
> | natGateways | Sí | Sí |
> | networkIntentPolicies | Sí | Sí |
> | networkInterfaces | Sí | Sí |
> | networkProfiles | Sí | Sí |
> | networkSecurityGroups | Sí | Sí |
> | networkWatchers | Sí | Sí |
> | networkWatchers/connectionMonitors | Sí | No |
> | networkWatchers/flowLogs | No | Sin |
> | networkWatchers/lenses | Sí | No |
> | networkWatchers/pingMeshes | Sí | No |
> | p2sVpnGateways | Sí | Sí |
> | privateDnsOperationStatuses | No | Sin |
> | privateDnsZones | Sí | Sí |
> | privateDnsZones/A | No | Sin |
> | privateDnsZones/AAAA | No | Sin |
> | privateDnsZones/all | No | Sin |
> | privateDnsZones/CNAME | No | Sin |
> | privateDnsZones/MX | No | Sin |
> | privateDnsZones/PTR | No | Sin |
> | privateDnsZones/SOA | No | No |
> | privateDnsZones/SRV | No | No |
> | privateDnsZones/TXT | No | No |
> | privateDnsZones/virtualNetworkLinks | Sí | Sí |
> | privateEndpoints | Sí | Sí |
> | privateLinkServices | Sí | Sí |
> | publicIPAddresses | Sí | Sí |
> | publicIPPrefixes | Sí | Sí |
> | routeFilters | Sí | Sí |
> | routeTables | Sí | Sí |
> | serviceEndpointPolicies | Sí | Sí |
> | trafficManagerGeographicHierarchies | No | No |
> | trafficmanagerprofiles | Sí | Sí |
> | trafficmanagerprofiles/heatMaps | No | No |
> | trafficManagerUserMetricsKeys | No | No |
> | virtualHubs | Sí | Sí |
> | virtualNetworkGateways | Sí | Sí |
> | virtualNetworks | Sí | Sí |
> | virtualNetworks / subnets | No | No |
> | virtualNetworkTaps | Sí | Sí |
> | virtualWans | Sí | No |
> | vpnGateways | Sí | Sí |
> | vpnSites | Sí | Sí |
> | webApplicationFirewallPolicies | Sí | Sí |

<a id="frontdoor"></a>

> [!NOTE]
> Para Azure Front Door Service, puede aplicar etiquetas al crear el recurso, pero no se admite actualmente la actualización o la adición de etiquetas.


## <a name="microsoftnotebooks"></a>Microsoft.Notebooks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | NotebookProxies | No | No |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | espacios de nombres | Sí | No |
> | namespaces/notificationHubs | Sí | No |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | osNamespaces | Sí | Sí |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | HyperVSites | Sí | Sí |
> | ImportSites | Sí | Sí |
> | ServerSites | Sí | Sí |
> | VMwareSites | Sí | Sí |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |
> | deletedWorkspaces | No | No |
> | linkTargets | No | No |
> | storageInsightConfigs | No | No |
> | workspaces | Sí | Sí |
> | workspaces / dataExports | No | No |
> | workspaces/dataSources | No | No |
> | workspaces/linkedServices | No | No |
> | workspaces/linkedStorageAccounts | No | No |
> | workspaces/metadata | No | No |
> | workspaces/query | No | No |
> | workspaces/scopedPrivateLinkProxies | No | No |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | managementassociations | No | No |
> | managementconfigurations | Sí | Sí |
> | solutions | Sí | Sí |
> | views | Sí | Sí |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | legacyPeerings | No | No |
> | peerAsns | No | No |
> | peerings | Sí | Sí |
> | peeringServiceCountries | No | No |
> | peeringServiceProviders | No | No |
> | peeringServices | Sí | Sí |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | policyEvents | No | No |
> | policyMetadata | No | No |
> | policyStates | No | No |
> | policyTrackedResources | No | No |
> | remediations | No | No |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | consoles | No | No |
> | dashboards | Sí | Sí |
> | userSettings | No | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | privateLinkServicesForPowerBI | Sí | Sí |
> | tenants | Sí | Sí |
> | tenants / workspaces | No | No |
> | workspaceCollections | Sí | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | capacities | Sí | Sí |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | No | No |
> | providerRegistrations / defaultRollouts | No | No |
> | providerRegistrations/resourceTypeRegistrations | No | No |
> | rollouts | Sí | Sí |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Áreas de trabajo | Sí | Sí |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | backupProtectedItems | No | No |
> | vaults | Sí | Sí |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | OpenShiftClusters | Sí | Sí |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/hybridconnections | No | No |
> | namespaces/hybridconnections/authorizationrules | No | No |
> | namespaces / privateEndpointConnections | No | No |
> | namespaces/wcfrelays | No | No |
> | namespaces/wcfrelays/authorizationrules | No | No |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | Consultas | Sí | Sí |
> | resourceChangeDetails | No | No |
> | resourceChanges | No | No |
> | resources | No | No |
> | resourcesHistory | No | No |
> | subscriptionsStatus | No | No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | availabilityStatuses | No | No |
> | childAvailabilityStatuses | No | No |
> | childResources | No | No |
> | emergingissues | No | No |
> | events | No | No |
> | impactedResources | No | No |
> | metadata | No | No |
> | Notificaciones | No | No |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | calculateTemplateHash | No | No |
> | deployments | Sí | No |
> | deployments/operations | No | No |
> | deploymentScripts | Sí | Sí |
> | deploymentScripts/logs | No | No |
> | vínculos | No | No |
> | notifyResourceJobs | No | No |
> | providers | No | No |
> | resourceGroups | Sí | No |
> | subscriptions | Sí | No |
> | templateSpecs | Sí | Sí |
> | templateSpecs / versions | Sí | Sí |
> | tenants | No | No |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applications | Sí | Sí |
> | saasresources | No | No |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | resourceHealthMetadata | No | No |
> | searchServices | Sí | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | adaptiveNetworkHardenings | No | No |
> | advancedThreatProtectionSettings | No | No |
> | alerts | No | No |
> | alertsSuppressionRules | No | No |
> | allowedConnections | No | No |
> | applicationWhitelistings | No | No |
> | assessmentMetadata | No | No |
> | assessments | No | No |
> | autoDismissAlertsRules | No | No |
> | automations | Sí | Sí |
> | AutoProvisioningSettings | No | No |
> | Compliances | No | No |
> | dataCollectionAgents | No | No |
> | deviceSecurityGroups | No | No |
> | discoveredSecuritySolutions | No | No |
> | externalSecuritySolutions | No | No |
> | InformationProtectionPolicies | No | No |
> | iotSecuritySolutions | Sí | Sí |
> | iotSecuritySolutions/analyticsModels | No | No |
> | iotSecuritySolutions/analyticsModels/aggregatedAlerts | No | No |
> | iotSecuritySolutions/analyticsModels/aggregatedRecommendations | No | No |
> | iotSecuritySolutions / iotAlerts | No | No |
> | iotSecuritySolutions / iotAlertTypes | No | No |
> | jitNetworkAccessPolicies | No | No |
> | directivas | No | No |
> | pricings | No | No |
> | regulatoryComplianceStandards | No | No |
> | regulatoryComplianceStandards/regulatoryComplianceControls | No | No |
> | regulatoryComplianceStandards/regulatoryComplianceControls/regulatoryComplianceAssessments | No | No |
> | secureScoreControlDefinitions | No | No |
> | secureScoreControls | No | No |
> | secureScores | No | No |
> | secureScores/secureScoreControls | No | No |
> | securityContacts | No | No |
> | securitySolutions | No | No |
> | securitySolutionsReferenceData | No | No |
> | securityStatuses | No | No |
> | securityStatusesSummaries | No | No |
> | serverVulnerabilityAssessments | No | No |
> | configuración | No | No |
> | subAssessments | No | No |
> | tareas | No | No |
> | topologies | No | No |
> | workspaceSettings | No | No |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | No | No |
> | diagnosticSettingsCategories | No | No |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | aggregations | No | No |
> | alertRules | No | No |
> | alertRuleTemplates | No | No |
> | automationRules | No | No |
> | bookmarks | No | No |
> | cases | No | No |
> | dataConnectors | No | No |
> | dataConnectorsCheckRequirements | No | No |
> | entities | No | No |
> | entityQueries | No | No |
> | incidents | No | No |
> | officeConsents | No | No |
> | configuración | No | No |
> | threatIntelligence | No | No |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | consoleServices | No | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/disasterrecoveryconfigs | No | No |
> | namespaces/eventgridfilters | No | No |
> | namespaces/networkrulesets | No | No |
> | namespaces/queues | No | No |
> | namespaces/queues/authorizationrules | No | No |
> | namespaces/topics | No | No |
> | namespaces/topics/authorizationrules | No | No |
> | namespaces/topics/subscriptions | No | No |
> | namespaces/topics/subscriptions/rules | No | No |
> | premiumMessagingRegions | No | No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applications | Sí | Sí |
> | clusters | Sí | Sí |
> | clusters/applications | No | No |
> | containerGroups | Sí | Sí |
> | containerGroupSets | Sí | Sí |
> | edgeclusters | Sí | Sí |
> | edgeclusters/applications | No | No |
> | managedclusters | Sí | Sí |
> | managedclusters/nodetypes | No | No |
> | networks | Sí | Sí |
> | secretstores | Sí | Sí |
> | secretstores/certificates | No | No |
> | secretstores/secrets | No | No |
> | volumes | Sí | Sí |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applications | Sí | Sí |
> | containerGroups | Sí | Sí |
> | gateways | Sí | Sí |
> | networks | Sí | Sí |
> | secrets | Sí | Sí |
> | volumes | Sí | Sí |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | No | No |
> | providerRegistrations/resourceTypeRegistrations | No | No |
> | rollouts | Sí | Sí |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | SignalR | Sí | Sí |
> | SignalR/eventGridFilters | No | No |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | hybridUseBenefits | No | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | applicationDefinitions | Sí | Sí |
> | applications | Sí | Sí |
> | jitRequests | Sí | Sí |


## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | managedInstances | Sí | Sí |
> | managedInstances/databases | Yes (see [note below](#sqlnote)) | Sí |
> | managedInstances/databases/backupShortTermRetentionPolicies | No | No |
> | managedInstances/databases/schemas/tables/columns/sensitivityLabels | No | No |
> | managedInstances/databases/vulnerabilityAssessments | No | No |
> | managedInstances/databases/vulnerabilityAssessments/rules/baselines | No | No |
> | managedInstances/encryptionProtector | No | No |
> | managedInstances/keys | No | No |
> | managedInstances/restorableDroppedDatabases/backupShortTermRetentionPolicies | No | No |
> | managedInstances/vulnerabilityAssessments | No | No |
> | servers | Sí | Sí |
> | servers/administrators | No | No |
> | servers/communicationLinks | No | No |
> | servers/databases | Yes (see [note below](#sqlnote)) | Sí |
> | servers/encryptionProtector | No | No |
> | servers/firewallRules | No | No |
> | servers/keys | No | No |
> | servers/restorableDroppedDatabases | No | No |
> | servers/serviceobjectives | No | No |
> | servers/tdeCertificates | No | No |
> | virtualClusters | No | No |

<a id="sqlnote"></a>

> [!NOTE]
> La base de datos maestra no admite etiquetas, pero otras bases de datos, como las de Azure Synapse Analytics, sí lo hacen. Las bases de datos de Azure Synapse Analytics deben estar en estado activo (no en pausa).

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | SqlVirtualMachineGroups | Sí | Sí |
> | SqlVirtualMachineGroups/AvailabilityGroupListeners | No | No |
> | SqlVirtualMachines | Sí | Sí |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | storageAccounts | Sí | Sí |
> | storageAccounts/blobServices | No | No |
> | storageAccounts/fileServices | No | No |
> | storageAccounts/queueServices | No | No |
> | storageAccounts/services | No | No |
> | storageAccounts/services/metricDefinitions | No | No |
> | storageAccounts/tableServices | No | No |
> | usages | No | No |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | caches | Sí | Sí |
> | caches/storageTargets | No | No |
> | usageModels | No | No |

## <a name="microsoftstoragereplication"></a>Microsoft.StorageReplication

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | replicationGroups | No | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Sí | Sí |
> | storageSyncServices/registeredServers | No | No |
> | storageSyncServices/syncGroups | No | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No | No |
> | storageSyncServices/syncGroups/serverEndpoints | No | No |
> | storageSyncServices/workflows | No | No |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Sí | Sí |
> | storageSyncServices/registeredServers | No | No |
> | storageSyncServices/syncGroups | No | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No | No |
> | storageSyncServices/syncGroups/serverEndpoints | No | No |
> | storageSyncServices/workflows | No | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Sí | Sí |
> | storageSyncServices/registeredServers | No | No |
> | storageSyncServices/syncGroups | No | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No | No |
> | storageSyncServices/syncGroups/serverEndpoints | No | No |
> | storageSyncServices/workflows | No | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | managers | Sí | Sí |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | clusters | Sí | Sí |
> | streamingjobs | Sí (vea la nota siguiente) | Sí |

> [!NOTE]
> No se pueden agregar etiquetas cuando hay instancias de streamingjobs en ejecución. Detenga el recurso si desea agregar una etiqueta.

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | cancel | No | No |
> | CreateSubscription | No | No |
> | enable | No | No |
> | rename | No | No |
> | SubscriptionDefinitions | No | No |
> | SubscriptionOperations | No | No |
> | subscriptions | No | No |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | privateLinkHubs | Sí | Sí |
> | workspaces | Sí | Sí |
> | workspaces / bigDataPools | Sí | Sí |
> | workspaces / operationStatuses | No | No |
> | workspaces / sqlPools | Sí | Sí |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | environments | Sí | No |
> | environments/accessPolicies | No | No |
> | environments/eventsources | Sí | No |
> | environments/referenceDataSets | Sí | No |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | almacenes | Sí | Sí |
> | stores / accessPolicies | No | No |
> | stores/services | No | No |
> | stores/services/tokens | No | No |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | imageTemplates | Sí | Sí |
> | imageTemplates / runOutputs | No | No |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | ArcZones | Sí | Sí |
> | ResourcePools | Sí | Sí |
> | VCenters | Sí | Sí |
> | VirtualMachines | Sí | Sí |
> | VirtualMachineTemplates | Sí | Sí |
> | VirtualNetworks | Sí | Sí |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | dedicatedCloudNodes | Sí | Sí |
> | dedicatedCloudServices | Sí | Sí |
> | virtualMachines | Sí | Sí |

## <a name="microsoftvmwareonazure"></a>Microsoft.VMwareOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | privateClouds | Sí | Sí |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | devices | Sí | Sí |
> | registeredSubscriptions | No | No |
> | vendors | No | No |
> | vendors / skus | No | No |
> | vendors/vnfs | No | No |
> | virtualNetworkFunctionSkus | No | No |
> | vnfs | Sí | Sí |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | accounts | Sí | Sí |
> | plans | Sí | Sí |
> | registeredSubscriptions | No | No |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | apiManagementAccounts | No | No |
> | apiManagementAccounts/apiAcls | No | No |
> | apiManagementAccounts/apis | No | No |
> | apiManagementAccounts/apis/apiAcls | No | No |
> | apiManagementAccounts/apis/connectionAcls | No | No |
> | apiManagementAccounts/apis/connections | No | No |
> | apiManagementAccounts/apis/connections/connectionAcls | No | No |
> | apiManagementAccounts/apis/localizedDefinitions | No | No |
> | apiManagementAccounts/connectionAcls | No | No |
> | apiManagementAccounts/connections | No | No |
> | billingMeters | No | No |
> | certificates | Sí | Sí |
> | connectionGateways | Sí | Sí |
> | connections | Sí | Sí |
> | customApis | Sí | Sí |
> | deletedSites | No | No |
> | hostingEnvironments | Sí | Sí |
> | hostingEnvironments / eventGridFilters | No | No |
> | hostingEnvironments/multiRolePools | No | No |
> | hostingEnvironments/workerPools | No | No |
> | kubeEnvironments | Sí | Sí |
> | publishingUsers | No | No |
> | de películas | No | No |
> | resourceHealthMetadata | No | No |
> | runtimes | No | No |
> | serverFarms | Sí | Sí |
> | serverFarms/eventGridFilters | No | No |
> | sites | Sí | Sí |
> | sites/config  | No | No |
> | sites/eventGridFilters | No | No |
> | sites/hostNameBindings | No | No |
> | sites/networkConfig | No | No |
> | sites/premieraddons | Sí | Sí |
> | sites/slots | Sí | Sí |
> | sites/slots/eventGridFilters | No | No |
> | sites/slots/hostNameBindings | No | No |
> | sites/slots/networkConfig | No | No |
> | sourceControls | No | No |
> | staticSites | Sí | Sí |
> | validate | No | No |
> | verifyHostingEnvironmentVnet | No | No |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | No | No |
> | diagnosticSettingsCategories | No | No |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | multipleActivationKeys | Sí | Sí |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | DeviceServices | Sí | Sí |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | workloads | Sí | Sí |
> | workloads / instances | No | No |
> | workloads / versions | No | No |
> | workloads / versions / artifacts | No | No |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Compatible con las etiquetas | Etiqueta en el informe de costos |
> | ------------- | ----------- | ----------- |
> | components | No | No |
> | componentsSummary | No | No |
> | monitorInstances | No | No |
> | monitorInstancesSummary | No | No |
> | monitors | No | No |
> | notificationSettings | No | No |

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo aplicar etiquetas a recursos, consulte [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md).
