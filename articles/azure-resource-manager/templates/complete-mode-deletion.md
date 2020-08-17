---
title: Eliminación del modo completo
description: Muestra cómo los tipos de recursos controlan la eliminación en modo completo en las plantillas de Azure Resource Manager.
ms.topic: conceptual
ms.date: 07/06/2020
ms.openlocfilehash: 48a7758ce706612607b97647bc88238957b118d3
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371352"
---
# <a name="deletion-of-azure-resources-for-complete-mode-deployments"></a>Eliminación de recursos de Azure en implementaciones en modo completo

En este artículo, se describe cómo se eliminan los tipos de recursos cuando no están en una plantilla implementada en modo completo.

Los tipos de recursos con la marca **Sí** se eliminan cuando el tipo no está en la plantilla que se implementó en modo completo.

Los tipos de recursos con la marca **No** no se eliminan automáticamente cuando no están en la plantilla; sin embargo, sí lo hacen cuando se elimina el recurso principal. Para ver una descripción completa de este comportamiento, consulte [Modos de implementación de Azure Resource Manager](deployment-modes.md).

Si implementa en [más de un grupo de recursos en una plantilla](cross-scope-deployment.md), se pueden eliminar los recursos del grupo de recursos especificado en la operación de implementación. Los recursos de los grupos de recursos secundarios no se eliminarán.

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
> - [Microsoft.ConnectedCache](#microsoftconnectedcache)
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
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | DomainServices | Sí |
> | DomainServices/oucontainer | No |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | supportProviders | No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | aadsupportcases | No |
> | addsservices | No |
> | agents | No |
> | anonymousapiusers | No |
> | configuración | No |
> | logs | No |
> | reports | No |
> | servicehealthmetrics | No |
> | services | No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | configuraciones | No |
> | generateRecommendations | No |
> | metadata | No |
> | de películas | No |
> | suppressions | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | actionRules | Sí |
> | alerts | No |
> | alertsList | No |
> | alertsMetaData | No |
> | alertsSummary | No |
> | alertsSummaryList | No |
> | smartDetectorAlertRules | Sí |
> | smartGroups | No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | servers | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | reportFeedback | No |
> | service | Sí |
> | validateServiceName | No |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | configurationStores | Sí |
> | configurationStores/eventGridFilters | No |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Spring | Sí |
> | Spring / apps | No |
> | Spring / apps / deployments | No |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | attestationProviders | Sí |
> | defaultProviders | No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | classicAdministrators | No |
> | dataAliases | No |
> | denyAssignments | No |
> | elevateAccess | No |
> | findOrphanRoleAssignments | No |
> | locks | No |
> | permisos | No |
> | policyAssignments | No |
> | policyDefinitions | No |
> | policySetDefinitions | No |
> | privateLinkAssociations | No |
> | providerOperations | No |
> | resourceManagementPrivateLinks | No |
> | roleAssignments | No |
> | roleAssignmentsUsageMetrics | No |
> | roleDefinitions | No |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | automationAccounts | Sí |
> | automationAccounts/configurations | Sí |
> | automationAccounts/jobs | No |
> | automationAccounts / privateEndpointConnectionProxies | No |
> | automationAccounts / privateEndpointConnections | No |
> | automationAccounts / privateLinkResources | No |
> | automationAccounts/runbooks | Sí |
> | automationAccounts/softwareUpdateConfigurations | No |
> | automationAccounts/webhooks | No |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | privateClouds | Sí |
> | privateClouds / authorizations | No |
> | privateClouds / clusters | No |
> | privateClouds / hcxEnterpriseSites | No |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | environments | No |
> | environments/accounts | No |
> | environments/accounts/namespaces | No |
> | environments/accounts/namespaces/configurations | No |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | b2cDirectories | Sí |
> | b2ctenants | No |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | dataControllers | Sí |
> | hybridDataManagers | Sí |
> | postgresInstances | Sí |
> | sqlInstances | Sí |
> | sqlManagedInstances | Sí |
> | sqlServerInstances | Sí |
> | sqlServerRegistrations | Sí |
> | sqlServerRegistrations/sqlServers | No |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | cloudManifestFiles | No |
> | edgeSubscriptions | Sí |
> | registrations | Sí |
> | registrations/customerSubscriptions | No |
> | registrations/products | No |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | batchAccounts | Sí |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | billingAccounts | No |
> | billingAccounts/agreements | No |
> | billingAccounts/billingPermissions | No |
> | billingAccounts/billingProfiles | No |
> | billingAccounts/billingProfiles/billingPermissions | No |
> | billingAccounts/billingProfiles/billingRoleAssignments | No |
> | billingAccounts/billingProfiles/billingRoleDefinitions | No |
> | billingAccounts/billingProfiles/billingSubscriptions | No |
> | billingAccounts/billingProfiles/createBillingRoleAssignment | No |
> | billingAccounts/billingProfiles/customers | No |
> | billingAccounts / billingProfiles / instructions | No |
> | billingAccounts/billingProfiles/invoices | No |
> | billingAccounts/billingProfiles/invoices/pricesheet | No |
> | billingAccounts/billingProfiles/invoices/transactions | No |
> | billingAccounts/billingProfiles/invoiceSections | No |
> | billingAccounts/billingProfiles/invoiceSections/billingPermissions | No |
> | billingAccounts/billingProfiles/invoiceSections/billingRoleAssignments | No |
> | billingAccounts/billingProfiles/invoiceSections/billingRoleDefinitions | No |
> | billingAccounts/billingProfiles/invoiceSections/billingSubscriptions | No |
> | billingAccounts/billingProfiles/invoiceSections/createBillingRoleAssignment | No |
> | billingAccounts/billingProfiles/invoiceSections/initiateTransfer | No |
> | billingAccounts/billingProfiles/invoiceSections/products | No |
> | billingAccounts/billingProfiles/invoiceSections/products/transfer | No |
> | billingAccounts/billingProfiles/invoiceSections/products/updateAutoRenew | No |
> | billingAccounts/billingProfiles/invoiceSections/transactions | No |
> | billingAccounts/billingProfiles/invoiceSections/transfers | No |
> | billingAccounts/BillingProfiles/patchOperations | No |
> | billingAccounts/billingProfiles/paymentMethods | No |
> | billingAccounts/billingProfiles/policies | No |
> | billingAccounts/billingProfiles/pricesheet | No |
> | billingAccounts/billingProfiles/pricesheetDownloadOperations | No |
> | billingAccounts/billingProfiles/products | No |
> | billingAccounts/billingProfiles/transactions | No |
> | billingAccounts/billingRoleAssignments | No |
> | billingAccounts/billingRoleDefinitions | No |
> | billingAccounts/billingSubscriptions | No |
> | billingAccounts/billingSubscriptions/invoices | No |
> | billingAccounts/createBillingRoleAssignment | No |
> | billingAccounts/createInvoiceSectionOperations | No |
> | billingAccounts/customers | No |
> | billingAccounts/customers/billingPermissions | No |
> | billingAccounts/customers/billingSubscriptions | No |
> | billingAccounts/customers/initiateTransfer | No |
> | billingAccounts/customers/policies | No |
> | billingAccounts/customers/products | No |
> | billingAccounts/customers/transactions | No |
> | billingAccounts/customers/transfers | No |
> | billingAccounts/departments | No |
> | billingAccounts / departments / billingPermissions | No |
> | billingAccounts / departments / billingRoleAssignments | No |
> | billingAccounts / departments / billingRoleDefinitions | No |
> | billingAccounts/enrollmentAccounts | No |
> | billingAccounts / enrollmentAccounts / billingPermissions | No |
> | billingAccounts / enrollmentAccounts / billingRoleAssignments | No |
> | billingAccounts / enrollmentAccounts / billingRoleDefinitions | No |
> | billingAccounts/invoices | No |
> | billingAccounts / invoices / transactions | No |
> | billingAccounts/invoiceSections | No |
> | billingAccounts/invoiceSections/billingSubscriptionMoveOperations | No |
> | billingAccounts/invoiceSections/billingSubscriptions | No |
> | billingAccounts/invoiceSections/billingSubscriptions/transfer | No |
> | billingAccounts/invoiceSections/elevate | No |
> | billingAccounts/invoiceSections/initiateTransfer | No |
> | billingAccounts/invoiceSections/patchOperations | No |
> | billingAccounts/invoiceSections/productMoveOperations | No |
> | billingAccounts/invoiceSections/products | No |
> | billingAccounts/invoiceSections/products/transfer | No |
> | billingAccounts/invoiceSections/products/updateAutoRenew | No |
> | billingAccounts/invoiceSections/transactions | No |
> | billingAccounts/invoiceSections/transfers | No |
> | billingAccounts/lineOfCredit | No |
> | billingAccounts/patchOperations | No |
> | billingAccounts/paymentMethods | No |
> | billingAccounts/products | No |
> | billingAccounts/transactions | No |
> | billingPeriods | No |
> | billingPermissions | No |
> | billingProperty | No |
> | billingRoleAssignments | No |
> | billingRoleDefinitions | No |
> | createBillingRoleAssignment | No |
> | departments | No |
> | enrollmentAccounts | No |
> | invoices | No |
> | transfers | No |
> | transfers/acceptTransfer | No |
> | transfers/declineTransfer | No |
> | transfers/operationStatus | No |
> | transfers/validateTransfer | No |
> | validateAddress | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | mapApis | Sí |
> | updateCommunicationPreference | No |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | blockchainMembers | Sí |
> | cordaMembers | Sí |
> | watchers | Sí |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | TokenServices | Sí |
> | TokenServices/BlockchainNetworks | No |
> | TokenServices/Groups | No |
> | TokenServices/Groups/Accounts | No |
> | TokenServices/TokenTemplates | No |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | blueprintAssignments | No |
> | blueprintAssignments/assignmentOperations | No |
> | blueprintAssignments/operations | No |
> | blueprints | No |
> | blueprints/artifacts | No |
> | blueprints/versions | No |
> | blueprints/versions/artifacts | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | botServices | Sí |
> | botServices/channels | No |
> | botServices/connections | No |
> | languages | No |
> | plantillas | No |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Redis | Sí |
> | Redis / EventGridFilters | No |
> | Redis / privateEndpointConnectionProxies | No |
> | Redis / privateEndpointConnectionProxies / validate | No |
> | Redis / privateEndpointConnections | No |
> | Redis / privateLinkResources | No |
> | redisEnterprise | Sí |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | appliedReservations | No |
> | autoQuotaIncrease | No |
> | calculateExchange | No |
> | calculatePrice | No |
> | calculatePurchasePrice | No |
> | catalogs | No |
> | commercialReservationOrders | No |
> | cambio | No |
> | placePurchaseOrder | No |
> | reservationOrders | No |
> | reservationOrders/calculateRefund | No |
> | reservationOrders/merge | No |
> | reservationOrders/reservations | No |
> | reservationOrders/reservations/revisions | No |
> | reservationOrders/return | No |
> | reservationOrders/split | No |
> | reservationOrders/swap | No |
> | reservations | No |
> | resourceProviders | No |
> | resources | No |
> | validateReservationOrder | No |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | CdnWebApplicationFirewallManagedRuleSets | No |
> | CdnWebApplicationFirewallPolicies | Sí |
> | edgenodes | No |
> | profiles | Sí |
> | profiles/endpoints | Sí |
> | profiles/endpoints/customdomains | No |
> | profiles/endpoints/origingroups | No |
> | profiles/endpoints/origins | No |
> | validateProbe | No |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | certificateOrders | Sí |
> | certificateOrders/certificates | No |
> | validateCertificateRegistrationInformation | No |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | perfile | No |
> | resourceChanges | No |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | capabilities | No |
> | domainNames | Sí |
> | domainNames/capabilities | No |
> | domainNames/internalLoadBalancers | No |
> | domainNames/serviceCertificates | No |
> | domainNames/slots | No |
> | domainNames/slots/roles | No |
> | domainNames/slots/roles/metricDefinitions | No |
> | domainNames/slots/roles/metrics | No |
> | moveSubscriptionResources | No |
> | operatingSystemFamilies | No |
> | operatingSystems | No |
> | quotas | No |
> | resourceTypes | No |
> | validateSubscriptionMoveAvailability | No |
> | virtualMachines | Sí |
> | virtualMachines/diagnosticSettings | No |
> | virtualMachines/metricDefinitions | No |
> | virtualMachines/metrics | No |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | classicInfrastructureResources | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | capabilities | No |
> | expressRouteCrossConnections | No |
> | expressRouteCrossConnections/peerings | No |
> | gatewaySupportedDevices | No |
> | networkSecurityGroups | Sí |
> | quotas | No |
> | reservedIps | Sí |
> | virtualNetworks | Sí |
> | virtualNetworks/remoteVirtualNetworkPeeringProxies | No |
> | virtualNetworks/virtualNetworkPeerings | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | capabilities | No |
> | disks | No |
> | images | No |
> | osImages | No |
> | osPlatformImages | No |
> | publicImages | No |
> | quotas | No |
> | storageAccounts | Sí |
> | storageAccounts/blobServices | No |
> | storageAccounts/fileServices | No |
> | storageAccounts/metricDefinitions | No |
> | storageAccounts/metrics | No |
> | storageAccounts/queueServices | No |
> | storageAccounts/services | No |
> | storageAccounts/services/diagnosticSettings | No |
> | storageAccounts/services/metricDefinitions | No |
> | storageAccounts/services/metrics | No |
> | storageAccounts/tableServices | No |
> | storageAccounts/vmImages | No |
> | vmImages | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | RateCard | No |
> | UsageAggregates | No |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | availabilitySets | Sí |
> | diskAccesses | Sí |
> | diskEncryptionSets | Sí |
> | disks | Sí |
> | galleries | Sí |
> | galleries/applications | No |
> | galleries/applications/versions | No |
> | galleries/images | No |
> | galleries/images/versions | No |
> | hostGroups | Sí |
> | hostGroups/hosts | Sí |
> | images | Sí |
> | proximityPlacementGroups | Sí |
> | restorePointCollections | Sí |
> | restorePointCollections/restorePoints | No |
> | sharedVMExtensions | Sí |
> | sharedVMExtensions/versions | No |
> | sharedVMImages | Sí |
> | sharedVMImages/versions | No |
> | snapshots | Sí |
> | sshPublicKeys | Sí |
> | virtualMachines | Sí |
> | virtualMachines/extensions | Sí |
> | virtualMachines/metricDefinitions | No |
> | virtualMachines / runCommands | Sí |
> | virtualMachineScaleSets | Sí |
> | virtualMachineScaleSets/extensions | No |
> | virtualMachineScaleSets/networkInterfaces | No |
> | virtualMachineScaleSets/publicIPAddresses | No |
> | virtualMachineScaleSets/virtualMachines | No |
> | virtualMachineScaleSets/virtualMachines/networkInterfaces | No |

## <a name="microsoftconnectedcache"></a>Microsoft.ConnectedCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | CacheNodes | Sí |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | AggregatedCost | No |
> | Saldos | No |
> | Presupuestos | No |
> | Charges | No |
> | CostTags | No |
> | credits | No |
> | events | No |
> | Previsiones | No |
> | lots | No |
> | Catálogos de soluciones | No |
> | Pricesheets | No |
> | products | No |
> | ReservationDetails | No |
> | ReservationRecommendationDetails | No |
> | ReservationRecommendations | No |
> | ReservationSummaries | No |
> | ReservationTransactions | No |
> | Etiquetas | No |
> | tenants | No |
> | Términos | No |
> | UsageDetails | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | containerGroups | Sí |
> | serviceAssociationLinks | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | registries | Sí |
> | registries/agentPools | Sí |
> | registries/builds | No |
> | registries/builds/cancel | No |
> | registries/builds/getLogLink | No |
> | registries/buildTasks | Sí |
> | registries/buildTasks/steps | No |
> | registries/eventGridFilters | No |
> | registries / exportPipelines | No |
> | registries/generateCredentials | No |
> | registries/getBuildSourceUploadUrl | No |
> | registries/GetCredentials | No |
> | registries/importImage | No |
> | registries / importPipelines | No |
> | registries / pipelineRuns | No |
> | registries/privateEndpointConnectionProxies | No |
> | registries/privateEndpointConnectionProxies/validate | No |
> | registries / privateEndpointConnections | No |
> | registries/privateLinkResources | No |
> | registries/queueBuild | No |
> | registries/regenerateCredential | No |
> | registries/regenerateCredentials | No |
> | registries/replications | Sí |
> | registries/runs | No |
> | registries/runs/cancel | No |
> | registries/scheduleRun | No |
> | registries/scopeMaps | No |
> | registries/taskRuns | No |
> | registries/tasks | Sí |
> | registries/tokens | No |
> | registries/updatePolicies | No |
> | registries/webhooks | Sí |
> | registries/webhooks/getCallbackConfig | No |
> | registries/webhooks/ping | No |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | containerServices | Sí |
> | managedClusters | Sí |
> | openShiftManagedClusters | Sí |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Alertas | No |
> | BillingAccounts | No |
> | Presupuestos | No |
> | CloudConnectors | No |
> | Conectores | Sí |
> | costAllocationRules | No |
> | Departments | No |
> | Dimensions | No |
> | EnrollmentAccounts | No |
> | Exports | No |
> | ExternalBillingAccounts | No |
> | ExternalBillingAccounts/Alerts | No |
> | ExternalBillingAccounts/Dimensions | No |
> | ExternalBillingAccounts/Forecast | No |
> | ExternalBillingAccounts/Query | No |
> | ExternalSubscriptions | No |
> | ExternalSubscriptions/Alerts | No |
> | ExternalSubscriptions/Dimensions | No |
> | ExternalSubscriptions/Forecast | No |
> | ExternalSubscriptions/Query | No |
> | Forecast | No |
> | Consultar | No |
> | registro | No |
> | Reportconfigs | No |
> | Informes | No |
> | Configuración | No |
> | showbackRules | No |
> | Vistas | No |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Solicitudes | No |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | associations | No |
> | resourceProviders | Sí |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | jobs | Sí |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | DataBoxEdgeDevices | Sí |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | workspaces | Sí |
> | workspaces / dbWorkspaces | No |
> | workspaces / storageEncryption | No |
> | workspaces/virtualNetworkPeerings | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | catalogs | Sí |
> | datacatalogs | Sí |
> | datacatalogs/datasources | No |
> | datacatalogs/datasources/scans | No |
> | datacatalogs/datasources/scans/datasets | No |
> | datacatalogs/datasources/scans/triggers | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | dataFactories | Sí |
> | dataFactories/diagnosticSettings | No |
> | dataFactories/metricDefinitions | No |
> | dataFactorySchema | No |
> | factories | Sí |
> | factories/integrationRuntimes | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |
> | accounts/dataLakeStoreAccounts | No |
> | accounts/storageAccounts | No |
> | accounts/storageAccounts/containers | No |
> | accounts/transferAnalyticsUnits | No |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |
> | accounts/eventGridFilters | No |
> | accounts/firewallRules | No |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | services | Sí |
> | services/projects | Sí |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | BackupVaults | Sí |
> | ResourceOperationGateKeepers | Sí |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |
> | accounts/shares | No |
> | accounts/shares/datasets | No |
> | accounts/shares/invitations | No |
> | accounts/shares/providersharesubscriptions | No |
> | accounts/shares/synchronizationSettings | No |
> | accounts/sharesubscriptions | No |
> | accounts/sharesubscriptions/consumerSourceDataSets | No |
> | accounts/sharesubscriptions/datasetmappings | No |
> | accounts/sharesubscriptions/triggers | No |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | servers | Sí |
> | servers/advisors | No |
> | servers/keys | No |
> | servers/privateEndpointConnectionProxies | No |
> | servers/privateEndpointConnections | No |
> | servers/privateLinkResources | No |
> | servers/queryTexts | No |
> | servers/recoverableServers | No |
> | servers/topQueryStatistics | No |
> | servers/virtualNetworkRules | No |
> | servers/waitStatistics | No |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | servers | Sí |
> | servers/advisors | No |
> | servers/keys | No |
> | servers/privateEndpointConnectionProxies | No |
> | servers/privateEndpointConnections | No |
> | servers/privateLinkResources | No |
> | servers/queryTexts | No |
> | servers/recoverableServers | No |
> | servers/topQueryStatistics | No |
> | servers/virtualNetworkRules | No |
> | servers/waitStatistics | No |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | serverGroups | Sí |
> | servers | Sí |
> | servers/advisors | No |
> | servers/keys | No |
> | servers/privateEndpointConnectionProxies | No |
> | servers/privateEndpointConnections | No |
> | servers/privateLinkResources | No |
> | servers/queryTexts | No |
> | servers/recoverableServers | No |
> | servers/topQueryStatistics | No |
> | servers/virtualNetworkRules | No |
> | servers/waitStatistics | No |
> | serversv2 | Sí |
> | singleServers | Sí |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | artifactSources | Sí |
> | rollouts | Sí |
> | serviceTopologies | Sí |
> | serviceTopologies/services | Sí |
> | serviceTopologies/services/serviceUnits | Sí |
> | steps | Sí |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applicationgroups | Sí |
> | applicationgroups/applications | No |
> | applicationgroups/desktops | No |
> | applicationgroups/startmenuitems | No |
> | hostpools | Sí |
> | hostpools/sessionhosts | No |
> | hostpools/sessionhosts/usersessions | No |
> | hostpools/usersessions | No |
> | workspaces | Sí |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | ElasticPools | Sí |
> | ElasticPools/IotHubTenants | Sí |
> | ElasticPools / IotHubTenants / securitySettings | No |
> | IotHubs | Sí |
> | IotHubs/eventGridFilters | No |
> | IotHubs / securitySettings | No |
> | ProvisioningServices | Sí |
> | usages | No |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | pipelines | Sí |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | controllers | Sí |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | labcenters | Sí |
> | labs | Sí |
> | labs/environments | Sí |
> | labs/serviceRunners | Sí |
> | labs/virtualMachines | Sí |
> | schedules | Sí |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | digitalTwinsInstances | Sí |
> | digitalTwinsInstances / endpoints | No |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | databaseAccountNames | No |
> | databaseAccounts | Sí |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | domains | Sí |
> | domains/domainOwnershipIdentifiers | No |
> | generateSsoRequest | No |
> | topLevelDomains | No |
> | validateDomainRegistrationInformation | No |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | lcsprojects | No |
> | lcsprojects/clouddeployments | No |
> | lcsprojects/connectors | No |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | services | Sí |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | domains | Sí |
> | domains/topics | No |
> | eventSubscriptions | No |
> | extensionTopics | No |
> | partnerNamespaces | Sí |
> | partnerNamespaces / eventChannels | No |
> | partnerRegistrations | Sí |
> | partnerTopics | Sí |
> | partnerTopics / eventSubscriptions | No |
> | systemTopics | Sí |
> | systemTopics / eventSubscriptions | No |
> | topics | Sí |
> | topicTypes | No |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |
> | espacios de nombres | Sí |
> | namespaces/authorizationrules | No |
> | namespaces/disasterrecoveryconfigs | No |
> | namespaces/eventhubs | No |
> | namespaces/eventhubs/authorizationrules | No |
> | namespaces/eventhubs/consumergroups | No |
> | namespaces/networkrulesets | No |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | experimentWorkspaces | Sí |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | espacios de nombres | Sí |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | featureProviders | No |
> | features | No |
> | providers | No |
> | subscriptionFeatureRegistrations | No |

## <a name="microsoftgallery"></a>Microsoft.Gallery

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | enroll | No |
> | galleryitems | No |
> | generateartifactaccessuri | No |
> | myareas | No |
> | myareas/areas | No |
> | myareas/areas/areas | No |
> | myareas/areas/areas/galleryitems | No |
> | myareas/areas/galleryitems | No |
> | myareas/galleryitems | No |
> | registro | No |
> | resources | No |
> | retrieveresourcesbyid | No |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | autoManagedAccounts | Sí |
> | autoManagedVmConfigurationProfiles | Sí |
> | configurationProfileAssignments | No |
> | guestConfigurationAssignments | No |
> | software | No |
> | softwareUpdateProfile | No |
> | softwareUpdates | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | hanaInstances | Sí |
> | sapMonitors | Sí |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | dedicatedHSMs | Sí |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |
> | clusters/applications | No |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | services | Sí |
> | services / iomtconnectors | No |
> | services / iomtconnectors / connections | No |
> | services / iomtconnectors / mappings | No |
> | services / privateEndpointConnectionProxies | No |
> | services / privateEndpointConnections | Sí |
> | services / privateLinkResources | Sí |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | machines | Sí |
> | machines/extensions | Sí |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | dataManagers | Sí |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | devices | Sí |
> | registeredSubscriptions | No |
> | vendors | No |
> | vendors / skus | No |
> | vendors/vnfs | No |
> | virtualNetworkFunctionSkus | No |
> | vnfs | Sí |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | components | Sí |
> | networkScopes | Sí |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | jobs | Sí |

## <a name="microsoftintune"></a>Microsoft.Intune

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | diagnosticSettings | No |
> | diagnosticSettingsCategories | No |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | appTemplates | No |
> | IoTApps | Sí |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Grafo | Sí |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | deletedVaults | No |
> | hsmPools | Sí |
> | managedHSMs | Sí |
> | vaults | Sí |
> | vaults/accessPolicies | No |
> | vaults/eventGridFilters | No |
> | vaults/secrets | No |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | connectedClusters | Sí |
> | registeredSubscriptions | No |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | sourceControlConfigurations | No |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |
> | clusters/attacheddatabaseconfigurations | No |
> | clusters/databases | No |
> | clusters/databases/dataconnections | No |
> | clusters/databases/eventhubconnections | No |
> | clusters / databases / principalassignments | No |
> | clusters/dataconnections | No |
> | clusters / principalassignments | No |
> | clusters/sharedidentities | No |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | labaccounts | Sí |
> | users | No |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | hostingEnvironments | Sí |
> | integrationAccounts | Sí |
> | integrationServiceEnvironments | Sí |
> | integrationServiceEnvironments/managedApis | Sí |
> | isolatedEnvironments | Sí |
> | workflows | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | commitmentPlans | Sí |
> | webServices | Sí |
> | Áreas de trabajo | Sí |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | workspaces | Sí |
> | workspaces/computes | No |
> | workspaces/eventGridFilters | No |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applyUpdates | No |
> | configurationAssignments | No |
> | maintenanceConfigurations | Sí |
> | updates | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Identities | No |
> | userAssignedIdentities | Sí |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | managedNetworks | Sí |
> | managedNetworks / managedNetworkGroups | Sí |
> | managedNetworks / managedNetworkPeeringPolicies | Sí |
> | notificación | Sí |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | marketplaceRegistrationDefinitions | No |
> | registrationAssignments | No |
> | registrationDefinitions | No |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | getEntities | No |
> | managementGroups | No |
> | managementGroups / settings | No |
> | resources | No |
> | startTenantBackfill | No |
> | tenantBackfillStatus | No |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |
> | accounts/eventGridFilters | No |
> | accounts/privateAtlases | Sí |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | offers | No |
> | offerTypes | No |
> | offerTypes/publishers | No |
> | offerTypes/publishers/offers | No |
> | offerTypes/publishers/offers/plans | No |
> | offerTypes/publishers/offers/plans/agreements | No |
> | offerTypes/publishers/offers/plans/configs | No |
> | offerTypes/publishers/offers/plans/configs/importImage | No |
> | privategalleryitems | No |
> | privateStoreClient | No |
> | privateStores | No |
> | privateStores/offers | No |
> | products | No |
> | publishers | No |
> | publishers/offers | No |
> | publishers/offers/amendments | No |
> | registro | No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | classicDevServices | Sí |
> | updateCommunicationPreference | No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | agreements | No |
> | offertypes | No |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | mediaservices | Sí |
> | mediaservices/accountFilters | No |
> | mediaservices/assets | No |
> | mediaservices/assets/assetFilters | No |
> | mediaservices/contentKeyPolicies | No |
> | mediaservices/eventGridFilters | No |
> | mediaservices/liveEventOperations | No |
> | mediaservices/liveEvents | Sí |
> | mediaservices/liveEvents/liveOutputs | No |
> | mediaservices / liveEvents / privateEndpointConnectionProxies | No |
> | mediaservices/liveOutputOperations | No |
> | mediaservices/mediaGraphs | No |
> | mediaservices/streamingEndpointOperations | No |
> | mediaservices/streamingEndpoints | Sí |
> | mediaservices / streamingEndpoints / privateEndpointConnectionProxies | No |
> | mediaservices/streamingLocators | No |
> | mediaservices/streamingPolicies | No |
> | mediaservices / streamingPrivateEndpointConnectionProxyOperations | No |
> | mediaservices/transforms | No |
> | mediaservices/transforms/jobs | No |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | appClusters | Sí |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | assessmentProjects | Sí |
> | migrateprojects | Sí |
> | moveCollections | Sí |
> | projects | Sí |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | holographicsBroadcastAccounts | Sí |
> | objectUnderstandingAccounts | Sí |
> | remoteRenderingAccounts | Sí |
> | spatialAnchorsAccounts | Sí |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | netAppAccounts | Sí |
> | netAppAccounts/accountBackups | No |
> | netAppAccounts/capacityPools | Sí |
> | netAppAccounts/capacityPools/volumes | Sí |
> | netAppAccounts/capacityPools/volumes/snapshots | Sí |
## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applicationGateways | Sí |
> | applicationGatewayWebApplicationFirewallPolicies | Sí |
> | applicationSecurityGroups | Sí |
> | azureFirewallFqdnTags | No |
> | azureFirewalls | Sí |
> | bastionHosts | Sí |
> | bgpServiceCommunities | No |
> | connections | Sí |
> | ddosCustomPolicies | Sí |
> | ddosProtectionPlans | Sí |
> | dnsOperationStatuses | No |
> | dnszones | Sí |
> | dnszones/A | No |
> | dnszones/AAAA | No |
> | dnszones/all | No |
> | dnszones/CAA | No |
> | dnszones/CNAME | No |
> | dnszones/MX | No |
> | dnszones/NS | No |
> | dnszones/PTR | No |
> | dnszones/recordsets | No |
> | dnszones/SOA | No |
> | dnszones/SRV | No |
> | dnszones/TXT | No |
> | expressRouteCircuits | Sí |
> | expressRouteCrossConnections | Sí |
> | expressRouteGateways | Sí |
> | expressRoutePorts | Sí |
> | expressRouteServiceProviders | No |
> | firewallPolicies | Sí |
> | frontdoors | Sí |
> | frontdoorWebApplicationFirewallManagedRuleSets | No |
> | frontdoorWebApplicationFirewallPolicies | Sí |
> | getDnsResourceReference | No |
> | internalNotify | No |
> | loadBalancers | Sí |
> | localNetworkGateways | Sí |
> | natGateways | Sí |
> | networkIntentPolicies | Sí |
> | networkInterfaces | Sí |
> | networkProfiles | Sí |
> | networkSecurityGroups | Sí |
> | networkWatchers | Sí |
> | networkWatchers/connectionMonitors | Sí |
> | networkWatchers/flowLogs | Sí |
> | networkWatchers/lenses | Sí |
> | networkWatchers/pingMeshes | Sí |
> | p2sVpnGateways | Sí |
> | privateDnsOperationStatuses | No |
> | privateDnsZones | Sí |
> | privateDnsZones/A | No |
> | privateDnsZones/AAAA | No |
> | privateDnsZones/all | No |
> | privateDnsZones/CNAME | No |
> | privateDnsZones/MX | No |
> | privateDnsZones/PTR | No |
> | privateDnsZones/SOA | No |
> | privateDnsZones/SRV | No |
> | privateDnsZones/TXT | No |
> | privateDnsZones/virtualNetworkLinks | Sí |
> | privateEndpoints | Sí |
> | privateLinkServices | Sí |
> | publicIPAddresses | Sí |
> | publicIPPrefixes | Sí |
> | routeFilters | Sí |
> | routeTables | Sí |
> | serviceEndpointPolicies | Sí |
> | trafficManagerGeographicHierarchies | No |
> | trafficmanagerprofiles | Sí |
> | trafficmanagerprofiles/heatMaps | No |
> | trafficManagerUserMetricsKeys | No |
> | virtualHubs | Sí |
> | virtualNetworkGateways | Sí |
> | virtualNetworks | Sí |
> | virtualNetworks / subnets | No |
> | virtualNetworkTaps | Sí |
> | virtualWans | Sí |
> | vpnGateways | Sí |
> | vpnSites | Sí |
> | webApplicationFirewallPolicies | Sí |

## <a name="microsoftnotebooks"></a>Microsoft.Notebooks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | NotebookProxies | No |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | espacios de nombres | Sí |
> | namespaces/notificationHubs | Sí |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | osNamespaces | Sí |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | HyperVSites | Sí |
> | ImportSites | Sí |
> | ServerSites | Sí |
> | VMwareSites | Sí |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |
> | deletedWorkspaces | No |
> | linkTargets | No |
> | storageInsightConfigs | No |
> | workspaces | Sí |
> | workspaces / dataExports | No |
> | workspaces/dataSources | No |
> | workspaces/linkedServices | No |
> | workspaces/linkedStorageAccounts | No |
> | workspaces/metadata | No |
> | workspaces/query | No |
> | workspaces/scopedPrivateLinkProxies | No |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | managementassociations | No |
> | managementconfigurations | Sí |
> | solutions | Sí |
> | views | Sí |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | legacyPeerings | No |
> | peerAsns | No |
> | peerings | Sí |
> | peeringServiceCountries | No |
> | peeringServiceProviders | No |
> | peeringServices | Sí |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | policyEvents | No |
> | policyMetadata | No |
> | policyStates | No |
> | policyTrackedResources | No |
> | remediations | No |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | consoles | No |
> | dashboards | Sí |
> | userSettings | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | privateLinkServicesForPowerBI | Sí |
> | tenants | Sí |
> | tenants / workspaces | No |
> | workspaceCollections | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | capacities | Sí |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | providerRegistrations | No |
> | providerRegistrations / defaultRollouts | No |
> | providerRegistrations/resourceTypeRegistrations | No |
> | rollouts | Sí |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Áreas de trabajo | Sí |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | backupProtectedItems | No |
> | vaults | Sí |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | OpenShiftClusters | Sí |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | espacios de nombres | Sí |
> | namespaces/authorizationrules | No |
> | namespaces/hybridconnections | No |
> | namespaces/hybridconnections/authorizationrules | No |
> | namespaces / privateEndpointConnections | No |
> | namespaces/wcfrelays | No |
> | namespaces/wcfrelays/authorizationrules | No |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | Consultas | Sí |
> | resourceChangeDetails | No |
> | resourceChanges | No |
> | resources | No |
> | resourcesHistory | No |
> | subscriptionsStatus | No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | availabilityStatuses | No |
> | childAvailabilityStatuses | No |
> | childResources | No |
> | emergingissues | No |
> | events | No |
> | impactedResources | No |
> | metadata | No |
> | Notificaciones | No |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | calculateTemplateHash | No |
> | deployments | No |
> | deployments/operations | No |
> | deploymentScripts | Sí |
> | deploymentScripts/logs | No |
> | vínculos | No |
> | notifyResourceJobs | No |
> | providers | No |
> | resourceGroups | No |
> | subscriptions | No |
> | templateSpecs | Sí |
> | templateSpecs / versions | Sí |
> | tenants | No |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applications | Sí |
> | saasresources | No |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | resourceHealthMetadata | No |
> | searchServices | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | adaptiveNetworkHardenings | No |
> | advancedThreatProtectionSettings | No |
> | alerts | No |
> | alertsSuppressionRules | No |
> | allowedConnections | No |
> | applicationWhitelistings | No |
> | assessmentMetadata | No |
> | assessments | No |
> | autoDismissAlertsRules | No |
> | automations | Sí |
> | AutoProvisioningSettings | No |
> | Compliances | No |
> | dataCollectionAgents | No |
> | deviceSecurityGroups | No |
> | discoveredSecuritySolutions | No |
> | externalSecuritySolutions | No |
> | InformationProtectionPolicies | No |
> | iotSecuritySolutions | Sí |
> | iotSecuritySolutions/analyticsModels | No |
> | iotSecuritySolutions/analyticsModels/aggregatedAlerts | No |
> | iotSecuritySolutions/analyticsModels/aggregatedRecommendations | No |
> | iotSecuritySolutions / iotAlerts | No |
> | iotSecuritySolutions / iotAlertTypes | No |
> | jitNetworkAccessPolicies | No |
> | directivas | No |
> | pricings | No |
> | regulatoryComplianceStandards | No |
> | regulatoryComplianceStandards/regulatoryComplianceControls | No |
> | regulatoryComplianceStandards/regulatoryComplianceControls/regulatoryComplianceAssessments | No |
> | secureScoreControlDefinitions | No |
> | secureScoreControls | No |
> | secureScores | No |
> | secureScores/secureScoreControls | No |
> | securityContacts | No |
> | securitySolutions | No |
> | securitySolutionsReferenceData | No |
> | securityStatuses | No |
> | securityStatusesSummaries | No |
> | serverVulnerabilityAssessments | No |
> | configuración | No |
> | subAssessments | No |
> | tareas | No |
> | topologies | No |
> | workspaceSettings | No |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | diagnosticSettings | No |
> | diagnosticSettingsCategories | No |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | aggregations | No |
> | alertRules | No |
> | alertRuleTemplates | No |
> | automationRules | No |
> | bookmarks | No |
> | cases | No |
> | dataConnectors | No |
> | dataConnectorsCheckRequirements | No |
> | entities | No |
> | entityQueries | No |
> | incidents | No |
> | officeConsents | No |
> | configuración | No |
> | threatIntelligence | No |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | consoleServices | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | espacios de nombres | Sí |
> | namespaces/authorizationrules | No |
> | namespaces/disasterrecoveryconfigs | No |
> | namespaces/eventgridfilters | No |
> | namespaces/networkrulesets | No |
> | namespaces/queues | No |
> | namespaces/queues/authorizationrules | No |
> | namespaces/topics | No |
> | namespaces/topics/authorizationrules | No |
> | namespaces/topics/subscriptions | No |
> | namespaces/topics/subscriptions/rules | No |
> | premiumMessagingRegions | No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applications | Sí |
> | clusters | Sí |
> | clusters/applications | No |
> | containerGroups | Sí |
> | containerGroupSets | Sí |
> | edgeclusters | Sí |
> | edgeclusters/applications | No |
> | managedclusters | Sí |
> | managedclusters/nodetypes | No |
> | networks | Sí |
> | secretstores | Sí |
> | secretstores/certificates | No |
> | secretstores/secrets | No |
> | volumes | Sí |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applications | Sí |
> | containerGroups | Sí |
> | gateways | Sí |
> | networks | Sí |
> | secrets | Sí |
> | volumes | Sí |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | providerRegistrations | No |
> | providerRegistrations/resourceTypeRegistrations | No |
> | rollouts | Sí |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | SignalR | Sí |
> | SignalR/eventGridFilters | No |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | hybridUseBenefits | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | applicationDefinitions | Sí |
> | applications | Sí |
> | jitRequests | Sí |

## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | managedInstances | Sí |
> | managedInstances/databases | Sí |
> | managedInstances/databases/backupShortTermRetentionPolicies | No |
> | managedInstances/databases/schemas/tables/columns/sensitivityLabels | No |
> | managedInstances/databases/vulnerabilityAssessments | No |
> | managedInstances/databases/vulnerabilityAssessments/rules/baselines | No |
> | managedInstances/encryptionProtector | No |
> | managedInstances/keys | No |
> | managedInstances/restorableDroppedDatabases/backupShortTermRetentionPolicies | No |
> | managedInstances/vulnerabilityAssessments | No |
> | servers | Sí |
> | servers/administrators | No |
> | servers/communicationLinks | No |
> | servers/databases | Sí |
> | servers/encryptionProtector | No |
> | servers/firewallRules | No |
> | servers/keys | No |
> | servers/restorableDroppedDatabases | No |
> | servers/serviceobjectives | No |
> | servers/tdeCertificates | No |
> | virtualClusters | No |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | SqlVirtualMachineGroups | Sí |
> | SqlVirtualMachineGroups/AvailabilityGroupListeners | No |
> | SqlVirtualMachines | Sí |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | storageAccounts | Sí |
> | storageAccounts/blobServices | No |
> | storageAccounts/fileServices | No |
> | storageAccounts/queueServices | No |
> | storageAccounts/services | No |
> | storageAccounts/services/metricDefinitions | No |
> | storageAccounts/tableServices | No |
> | usages | No |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | caches | Sí |
> | caches/storageTargets | No |
> | usageModels | No |

## <a name="microsoftstoragereplication"></a>Microsoft.StorageReplication

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | replicationGroups | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | storageSyncServices | Sí |
> | storageSyncServices/registeredServers | No |
> | storageSyncServices/syncGroups | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No |
> | storageSyncServices/syncGroups/serverEndpoints | No |
> | storageSyncServices/workflows | No |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | storageSyncServices | Sí |
> | storageSyncServices/registeredServers | No |
> | storageSyncServices/syncGroups | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No |
> | storageSyncServices/syncGroups/serverEndpoints | No |
> | storageSyncServices/workflows | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | storageSyncServices | Sí |
> | storageSyncServices/registeredServers | No |
> | storageSyncServices/syncGroups | No |
> | storageSyncServices/syncGroups/cloudEndpoints | No |
> | storageSyncServices/syncGroups/serverEndpoints | No |
> | storageSyncServices/workflows | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | managers | Sí |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | clusters | Sí |
> | streamingjobs | Sí |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | cancel | No |
> | CreateSubscription | No |
> | enable | No |
> | rename | No |
> | SubscriptionDefinitions | No |
> | SubscriptionOperations | No |
> | subscriptions | No |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | privateLinkHubs | Sí |
> | workspaces | Sí |
> | workspaces / bigDataPools | Sí |
> | workspaces / operationStatuses | No |
> | workspaces / sqlPools | Sí |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | environments | Sí |
> | environments/accessPolicies | No |
> | environments/eventsources | Sí |
> | environments/referenceDataSets | Sí |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | almacenes | Sí |
> | stores / accessPolicies | No |
> | stores/services | No |
> | stores/services/tokens | No |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | imageTemplates | Sí |
> | imageTemplates / runOutputs | No |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | ArcZones | Sí |
> | ResourcePools | Sí |
> | VCenters | Sí |
> | VirtualMachines | Sí |
> | VirtualMachineTemplates | Sí |
> | VirtualNetworks | Sí |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | dedicatedCloudNodes | Sí |
> | dedicatedCloudServices | Sí |
> | virtualMachines | Sí |

## <a name="microsoftvmwareonazure"></a>Microsoft.VMwareOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | privateClouds | Sí |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | devices | Sí |
> | registeredSubscriptions | No |
> | vendors | No |
> | vendors / skus | No |
> | vendors/vnfs | No |
> | virtualNetworkFunctionSkus | No |
> | vnfs | Sí |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | accounts | Sí |
> | plans | Sí |
> | registeredSubscriptions | No |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | apiManagementAccounts | No |
> | apiManagementAccounts/apiAcls | No |
> | apiManagementAccounts/apis | No |
> | apiManagementAccounts/apis/apiAcls | No |
> | apiManagementAccounts/apis/connectionAcls | No |
> | apiManagementAccounts/apis/connections | No |
> | apiManagementAccounts/apis/connections/connectionAcls | No |
> | apiManagementAccounts/apis/localizedDefinitions | No |
> | apiManagementAccounts/connectionAcls | No |
> | apiManagementAccounts/connections | No |
> | billingMeters | No |
> | certificates | Sí |
> | connectionGateways | Sí |
> | connections | Sí |
> | customApis | Sí |
> | deletedSites | No |
> | hostingEnvironments | Sí |
> | hostingEnvironments / eventGridFilters | No |
> | hostingEnvironments/multiRolePools | No |
> | hostingEnvironments/workerPools | No |
> | kubeEnvironments | Sí |
> | publishingUsers | No |
> | de películas | No |
> | resourceHealthMetadata | No |
> | runtimes | No |
> | serverFarms | Sí |
> | serverFarms/eventGridFilters | No |
> | sites | Sí |
> | sites/config  | No |
> | sites/eventGridFilters | No |
> | sites/hostNameBindings | No |
> | sites/networkConfig | No |
> | sites/premieraddons | Sí |
> | sites/slots | Sí |
> | sites/slots/eventGridFilters | No |
> | sites/slots/hostNameBindings | No |
> | sites/slots/networkConfig | No |
> | sourceControls | No |
> | staticSites | Sí |
> | validate | No |
> | verifyHostingEnvironmentVnet | No |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | diagnosticSettings | No |
> | diagnosticSettingsCategories | No |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | multipleActivationKeys | Sí |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | DeviceServices | Sí |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | workloads | Sí |
> | workloads / instances | No |
> | workloads / versions | No |
> | workloads / versions / artifacts | No |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Eliminación del modo completo |
> | ------------- | ----------- |
> | components | No |
> | componentsSummary | No |
> | monitorInstances | No |
> | monitorInstancesSummary | No |
> | monitors | No |
> | notificationSettings | No |

## <a name="next-steps"></a>Pasos siguientes

Para obtener los mismos datos que un archivo de valores separados por comas, descargue [complete-mode-deletion.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/complete-mode-deletion.csv).
