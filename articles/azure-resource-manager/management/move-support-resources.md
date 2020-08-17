---
title: Compatibilidad con la operación de traslado por tipo de recurso
description: Enumera los tipos de recursos de Azure que se pueden trasladar a un nuevo grupo de recursos o suscripción.
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 16197210326d73284a4a83edc7876e4faddded86
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079500"
---
# <a name="move-operation-support-for-resources"></a>Compatibilidad con la operación de traslado para recursos

En este artículo se indica si un tipo de recurso de Azure es compatible con la operación de traslado. También proporciona información sobre las condiciones especiales que se deben tener en cuenta a la hora de mover un recurso.

Vaya a un espacio de nombres del proveedor de recursos:
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.AppService](#microsoftappservice)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.BizTalkServices](#microsoftbiztalkservices)
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
> - [Microsoft.ClassicSubscription](#microsoftclassicsubscription)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.ContentModerator](#microsoftcontentmoderator)
> - [Microsoft.CortanaAnalytics](#microsoftcortanaanalytics)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CostManagementExports](#microsoftcostmanagementexports)
> - [Microsoft.CustomerInsights](#microsoftcustomerinsights)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataConnect](#microsoftdataconnect)
> - [Microsoft.DataExchange](#microsoftdataexchange)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataLake](#microsoftdatalake)
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
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
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
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.LocationBasedServices](#microsoftlocationbasedservices)
> - [Microsoft.LocationServices](#microsoftlocationservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningCompute](#microsoftmachinelearningcompute)
> - [Microsoft.MachineLearningExperimentation](#microsoftmachinelearningexperimentation)
> - [Microsoft.MachineLearningModelManagement](#microsoftmachinelearningmodelmanagement)
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
> - [Microsoft.PowerPlatform](#microsoftpowerplatform)
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
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServerManagement](#microsoftservermanagement)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.StreamAnalyticsExplorer](#microsoftstreamanalyticsexplorer)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [microsoft.support](#microsoftsupport)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [microsoft.visualstudio](#microsoftvisualstudio)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WorkloadBuilder](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | domainservices | No | No |
> | domainservices/oucontainer | No | No |
> | locations | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | No | No |
> | diagnosticsettingscategories | No | No |
> | operaciones | No | No |
> | privatelinkforazuread | Sí | Sí |
> | tenants | Sí | Sí |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operationresults | No | No |
> | operaciones | No | No |
> | supportproviders | No | No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aadsupportcases | No | No |
> | addsservices | No | No |
> | agents | No | No |
> | anonymousapiusers | No | No |
> | configuración | No | No |
> | logs | No | No |
> | operaciones | No | No |
> | reports | No | No |
> | servicehealthmetrics | No | No |
> | services | No | No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | configuraciones | No | No |
> | generaterecommendations | No | No |
> | metadata | No | No |
> | operaciones | No | No |
> | de películas | No | No |
> | suppressions | No | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | actionRules | Sí | Sí |
> | alerts | No | No |
> | alertslist | No | No |
> | alertsmetadata | No | No |
> | alertssummary | No | No |
> | alertssummarylist | No | No |
> | operaciones | No | No |
> | smartdetectoralertrules | Sí | Sí |
> | smartgroups | No | No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | servers | Sí | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> Un servicio API Management establecido en la SKU de consumo no se puede mover.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checkfeedbackrequired | No | No |
> | checknameavailability | No | No |
> | checkservicenameavailability | No | No |
> | operaciones | No | No |
> | reportfeedback | No | No |
> | service | Sí | Sí |
> | validateservicename | No | No |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | configurationstores | Sí | Sí |
> | configurationstores/eventgridfilters | No | No |
> | locations | No | No |
> | locations/operationsstatus | No | No |
> | operaciones | No | No |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatus | No | No |
> | operaciones | No | No |
> | spring | Sí | Sí |
> | spring/apps | No | No |
> | spring/apps/deployments | No | No |

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | apiapps | No | No |
> | appidentities | No | No |
> | gateways | No | No |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | attestationproviders | Sí | Sí |
> | operaciones | No | No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checkaccess | No | No |
> | classicadministrators | No | No |
> | dataaliases | No | No |
> | denyassignments | No | No |
> | elevateaccess | No | No |
> | findorphanroleassignments | No | No |
> | locks | No | No |
> | operaciones | No | No |
> | operationstatus | No | No |
> | permisos | No | No |
> | policyassignments | No | No |
> | policydefinitions | No | No |
> | policysetdefinitions | No | No |
> | privatelinkassociations | No | No |
> | provideroperations | No | No |
> | resourcemanagementprivatelinks | No | No |
> | roleassignments | No | No |
> | roleassignmentsusagemetrics | No | No |
> | roledefinitions | No | No |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Los runbooks deben encontrarse en el mismo grupo de recursos que la cuenta de Automation.
>
> Para obtener más información, consulte [Traslado de la cuenta de Azure Automation a otra suscripción](../../automation/how-to/move-account.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | automationaccounts | Sí | Sí |
> | automationaccounts/configurations | Sí | Sí |
> | automationaccounts/jobs | No | No |
> | automationaccounts/privateendpointconnectionproxies | No | No |
> | automationaccounts/privateendpointconnections | No | No |
> | automationaccounts/privatelinkresources | No | No |
> | automationaccounts/runbooks | Sí | Sí |
> | automationaccounts/softwareupdateconfigurations | No | No |
> | automationaccounts/webhooks | No | No |
> | operaciones | No | No |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checkquotaavailability | No | No |
> | locations/checktrialavailability | No | No |
> | operaciones | No | No |
> | privateclouds | Sí | Sí |
> | privateclouds/clusters | No | No |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | Sí | Sí |
> | b2ctenants | No | No |
> | checknameavailability | No | No |
> | operaciones | No | No |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datacontrollers | No | No |
> | hybriddatamanagers | No | No |
> | operaciones | No | No |
> | postgresinstances | No | No |
> | sqlinstances | No | No |
> | sqlmanagedinstances | No | No |
> | sqlserverinstances | No | No |
> | sqlserverregistrations | Sí | Sí |
> | sqlserverregistrations/sqlservers | No | No |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | No | No |
> | operaciones | No | No |
> | registrations | Sí | Sí |
> | registrations/customersubscriptions | No | No |
> | registrations/products | No | No |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |
> | operaciones | No | No |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | batchaccounts | Sí | Sí |
> | locations | No | No |
> | locations/accountoperationresults | No | No |
> | locations/checknameavailability | No | No |
> | locations/quotas | No | No |
> | operaciones | No | No |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | billingaccounts | No | No |
> | billingaccounts/agreements | No | No |
> | billingaccounts/billingpermissions | No | No |
> | billingaccounts/billingprofiles | No | No |
> | billingaccounts/billingprofiles/availablebalance | No | No |
> | billingaccounts/billingprofiles/billingpermissions | No | No |
> | billingaccounts/billingprofiles/billingroleassignments | No | No |
> | billingaccounts/billingprofiles/billingroledefinitions | No | No |
> | billingaccounts/billingprofiles/billingsubscriptions | No | No |
> | billingaccounts/billingprofiles/createbillingroleassignment | No | No |
> | billingaccounts/billingprofiles/customers | No | No |
> | billingaccounts/billingprofiles/instructions | No | No |
> | billingaccounts/billingprofiles/invoices | No | No |
> | billingaccounts/billingprofiles/invoices/pricesheet | No | No |
> | billingaccounts/billingprofiles/invoices/transactions | No | No |
> | billingaccounts/billingprofiles/invoicesections | No | No |
> | billingaccounts/billingprofiles/invoicesections/billingpermissions | No | No |
> | billingaccounts/billingprofiles/invoicesections/billingroleassignments | No | No |
> | billingaccounts/billingprofiles/invoicesections/billingroledefinitions | No | No |
> | billingaccounts/billingprofiles/invoicesections/billingsubscriptions | No | No |
> | billingaccounts/billingprofiles/invoicesections/createbillingroleassignment | No | No |
> | billingaccounts/billingprofiles/invoicesections/initiatetransfer | No | No |
> | billingaccounts/billingprofiles/invoicesections/products | No | No |
> | billingaccounts/billingprofiles/invoicesections/products/transfer | No | No |
> | billingaccounts/billingprofiles/invoicesections/products/updateautorenew | No | No |
> | billingaccounts/billingprofiles/invoicesections/transactions | No | No |
> | billingaccounts/billingprofiles/invoicesections/transfers | No | No |
> | billingaccounts/billingprofiles/patchoperations | No | No |
> | billingaccounts/billingprofiles/paymentmethods | No | No |
> | billingaccounts/billingprofiles/policies | No | No |
> | billingaccounts/billingprofiles/pricesheet | No | No |
> | billingaccounts/billingprofiles/pricesheetdownloadoperations | No | No |
> | billingaccounts/billingprofiles/products | No | No |
> | billingaccounts/billingprofiles/transactions | No | No |
> | billingaccounts/billingroleassignments | No | No |
> | billingaccounts/billingroledefinitions | No | No |
> | billingaccounts/billingsubscriptions | No | No |
> | billingaccounts/billingsubscriptions/invoices | No | No |
> | billingaccounts/createbillingroleassignment | No | No |
> | billingaccounts/createinvoicesectionoperations | No | No |
> | billingaccounts/customers | No | No |
> | billingaccounts/customers/billingpermissions | No | No |
> | billingaccounts/customers/billingsubscriptions | No | No |
> | billingaccounts/customers/initiatetransfer | No | No |
> | billingaccounts/customers/policies | No | No |
> | billingaccounts/customers/products | No | No |
> | billingaccounts/customers/transactions | No | No |
> | billingaccounts/customers/transfers | No | No |
> | billingaccounts/departments | No | No |
> | billingaccounts/enrollmentaccounts | No | No |
> | billingaccounts/invoices | No | No |
> | billingaccounts/invoicesections | No | No |
> | billingaccounts/invoicesections/billingsubscriptionmoveoperations | No | No |
> | billingaccounts/invoicesections/billingsubscriptions | No | No |
> | billingaccounts/invoicesections/billingsubscriptions/transfer | No | No |
> | billingaccounts/invoicesections/elevate | No | No |
> | billingaccounts/invoicesections/initiatetransfer | No | No |
> | billingaccounts/invoicesections/patchoperations | No | No |
> | billingaccounts/invoicesections/productmoveoperations | No | No |
> | billingaccounts/invoicesections/products | No | No |
> | billingaccounts/invoicesections/products/transfer | No | No |
> | billingaccounts/invoicesections/products/updateautorenew | No | No |
> | billingaccounts/invoicesections/producttransfersresults | No | No |
> | billingaccounts/invoicesections/transactions | No | No |
> | billingaccounts/invoicesections/transfers | No | No |
> | billingaccounts/lineofcredit | No | No |
> | billingaccounts/listinvoicesectionswithcreatesubscriptionpermission | No | No |
> | billingaccounts/operationresults | No | No |
> | billingaccounts/patchoperations | No | No |
> | billingaccounts/paymentmethods | No | No |
> | billingaccounts/products | No | No |
> | billingaccounts/transactions | No | No |
> | billingperiods | No | No |
> | billingpermissions | No | No |
> | billingproperty | No | No |
> | billingroleassignments | No | No |
> | billingroledefinitions | No | No |
> | createbillingroleassignment | No | No |
> | departments | No | No |
> | enrollmentaccounts | No | No |
> | invoices | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | operationstatus | No | No |
> | transfers | No | No |
> | transfers/accepttransfer | No | No |
> | transfers/declinetransfer | No | No |
> | transfers/operationstatus | No | No |
> | transfers/validatetransfer | No | No |
> | validateaddress | No | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | listcommunicationpreference | No | No |
> | mapapis | No | No |
> | operaciones | No | No |
> | updatecommunicationpreference | No | No |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | biztalk | No | No |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | blockchainmembers | No | No |
> | cordamembers | No | No |
> | locations | No | No |
> | locations/blockchainmemberoperationresults | No | No |
> | locations/checknameavailability | No | No |
> | locations/listconsortiums | No | No |
> | locations/watcheroperationresults | No | No |
> | operaciones | No | No |
> | watchers | No | No |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |
> | tokenservices | No | No |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | blueprintassignments | No | No |
> | blueprintassignments/assignmentoperations | No | No |
> | blueprintassignments/operations | No | No |
> | blueprints | No | No |
> | blueprints/artifacts | No | No |
> | blueprints/versions | No | No |
> | blueprints/versions/artifacts | No | No |
> | operaciones | No | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | botservices | Sí | Sí |
> | botservices/channels | No | No |
> | botservices/connections | No | No |
> | checknameavailability | No | No |
> | listauthserviceproviders | No | No |
> | operaciones | No | No |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> Si la instancia de Azure Redis Cache está configurada con una red virtual, la instancia no se puede mover a otra suscripción. Consulte las [Limitaciones de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/operationresults | No | No |
> | locations/operationsstatus | No | No |
> | operaciones | No | No |
> | redis | Sí | Sí |
> | redis/eventgridfilters | No | No |
> | redis/privatelinkresources | No | No |
> | redisenterprise | No | No |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | appliedreservations | No | No |
> | calculateexchange | No | No |
> | calculateprice | No | No |
> | calculatepurchaseprice | No | No |
> | catalogs | No | No |
> | checkoffers | No | No |
> | checkpurchasestatus | No | No |
> | checkscopes | No | No |
> | commercialreservationorders | No | No |
> | cambio | No | No |
> | listbenefits | No | No |
> | operaciones | No | No |
> | placepurchaseorder | No | No |
> | reservationorders | No | No |
> | reservationorders/availablescopes | No | No |
> | reservationorders/calculaterefund | No | No |
> | reservationorders/merge | No | No |
> | reservationorders/reservations | No | No |
> | reservationorders/reservations/availablescopes | No | No |
> | reservationorders/reservations/revisions | No | No |
> | reservationorders/return | No | No |
> | reservationorders/split | No | No |
> | reservationorders/swap | No | No |
> | reservations | No | No |
> | resources | No | No |
> | validatereservationorder | No | No |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | cdnwebapplicationfirewallmanagedrulesets | No | No |
> | cdnwebapplicationfirewallpolicies | Sí | Sí |
> | checknameavailability | No | No |
> | checkresourceusage | No | No |
> | edgenodes | No | No |
> | operationresults | No | No |
> | operationresults/profileresults | No | No |
> | operationresults/profileresults/endpointresults | No | No |
> | operationresults/profileresults/endpointresults/customdomainresults | No | No |
> | operationresults/profileresults/endpointresults/origingroupresults | No | No |
> | operationresults/profileresults/endpointresults/originresults | No | No |
> | operaciones | No | No |
> | profiles | Sí | Sí |
> | profiles/endpoints | Sí | Sí |
> | profiles/endpoints/customdomains | No | No |
> | profiles/endpoints/origingroups | No | No |
> | profiles/endpoints/origins | No | No |
> | validateprobe | No | No |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | certificateorders | Sí | Sí |
> | certificateorders/certificates | No | No |
> | operaciones | No | No |
> | validatecertificateregistrationinformation | No | No |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capabilities | No | No |
> | checkdomainnameavailability | No | No |
> | domainnames | Sí | No |
> | domainnames/capabilities | No | No |
> | domainnames/internalloadbalancers | No | No |
> | domainnames/servicecertificates | No | No |
> | domainnames/slots | No | No |
> | domainnames/slots/roles | No | No |
> | domainnames/slots/roles/metricdefinitions | No | No |
> | domainnames/slots/roles/metrics | No | No |
> | movesubscriptionresources | No | No |
> | operatingsystemfamilies | No | No |
> | operatingsystems | No | No |
> | operaciones | No | No |
> | operationstatuses | No | No |
> | quotas | No | No |
> | resourcetypes | No | No |
> | validatesubscriptionmoveavailability | No | No |
> | virtualmachines | Sí | No |
> | virtualmachines/diagnosticsettings | No | No |
> | virtualmachines/metricdefinitions | No | No |
> | virtualmachines/metrics | No | No |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | classicinfrastructureresources | No | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capabilities | No | No |
> | expressroutecrossconnections | No | No |
> | expressroutecrossconnections/peerings | No | No |
> | gatewaysupporteddevices | No | No |
> | networksecuritygroups | No | No |
> | operaciones | No | No |
> | quotas | No | No |
> | reservedips | No | No |
> | virtualnetworks | No | No |
> | virtualnetworks/remotevirtualnetworkpeeringproxies | No | No |
> | virtualnetworks/virtualnetworkpeerings | No | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capabilities | No | No |
> | checkstorageaccountavailability | No | No |
> | disks | No | No |
> | images | No | No |
> | operaciones | No | No |
> | osimages | No | No |
> | osplatformimages | No | No |
> | publicimages | No | No |
> | quotas | No | No |
> | storageaccounts | Sí | No |
> | storageaccounts/blobservices | No | No |
> | storageaccounts/fileservices | No | No |
> | storageaccounts/metricdefinitions | No | No |
> | storageaccounts/metrics | No | No |
> | storageaccounts/queueservices | No | No |
> | storageaccounts/services | No | No |
> | storageaccounts/services/diagnosticsettings | No | No |
> | storageaccounts/services/metricdefinitions | No | No |
> | storageaccounts/services/metrics | No | No |
> | storageaccounts/tableservices | No | No |
> | storageaccounts/vmimages | No | No |
> | vmimages | No | No |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | checkdomainavailability | No | No |
> | locations | No | No |
> | locations/checkskuavailability | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |
> | ratecard | No | No |
> | usageaggregates | No | No |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!IMPORTANT]
> Vea [Guía de movimiento de Virtual Machines](./move-limitations/virtual-machines-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availabilitysets | Sí | Sí |
> | diskaccesses | No | No |
> | diskencryptionsets | No | No |
> | disks | Sí | Sí |
> | galleries | No | No |
> | galleries/images | No | No |
> | galleries/images/versions | No | No |
> | hostgroups | No | No |
> | hostgroups/hosts | No | No |
> | images | Sí | Sí |
> | locations | No | No |
> | locations/artifactpublishers | No | No |
> | locations/capsoperations | No | No |
> | locations/diskoperations | No | No |
> | locations/loganalytics | No | No |
> | locations/operations | No | No |
> | locations/publishers | No | No |
> | locations/runcommands | No | No |
> | locations/usages | No | No |
> | locations/virtualmachines | No | No |
> | locations/vmsizes | No | No |
> | locations/vsmoperations | No | No |
> | operaciones | No | No |
> | proximityplacementgroups | Sí | Sí |
> | restorepointcollections | No | No |
> | restorepointcollections/restorepoints | No | No |
> | sharedvmextensions | No | No |
> | sharedvmimages | No | No |
> | sharedvmimages/versions | No | No |
> | snapshots | Sí | Sí |
> | sshpublickeys | No | No |
> | virtualmachines | Sí | Sí |
> | virtualmachines/extensions | Sí | Sí |
> | virtualmachines/metricdefinitions | No | No |
> | virtualmachines/runcommands | No | No |
> | virtualmachinescalesets | Sí | Sí |
> | virtualmachinescalesets/extensions | No | No |
> | virtualmachinescalesets/networkinterfaces | No | No |
> | virtualmachinescalesets/publicipaddresses | No | No |
> | virtualmachinescalesets/virtualmachines | No | No |
> | virtualmachinescalesets/virtualmachines/networkinterfaces | No | No |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aggregatedcost | No | No |
> | balances | No | No |
> | budgets | No | No |
> | charges | No | No |
> | costtags | No | No |
> | credits | No | No |
> | events | No | No |
> | forecasts | No | No |
> | lots | No | No |
> | marketplaces | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | operationstatus | No | No |
> | pricesheets | No | No |
> | products | No | No |
> | reservationdetails | No | No |
> | reservationrecommendationdetails | No | No |
> | reservationrecommendations | No | No |
> | reservationsummaries | No | No |
> | reservationtransactions | No | No |
> | etiquetas | No | No |
> | tenants | No | No |
> | terms | No | No |
> | usagedetails | No | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | containergroups | No | No |
> | locations | No | No |
> | locations/cachedimages | No | No |
> | locations/capabilities | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operations | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |
> | serviceassociationlinks | No | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/authorize | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operationresults | No | No |
> | locations/setupauth | No | No |
> | operaciones | No | No |
> | registries | Sí | Sí |
> | registries / agentpools | Sí | Sí |
> | registries/agentpools/listqueuestatus | No | No |
> | registries/builds | No | No |
> | registries/builds/cancel | No | No |
> | registries/builds/getloglink | No | No |
> | registries/buildtasks | Sí | Sí |
> | registries/buildtasks/listsourcerepositoryproperties | No | No |
> | registries/buildtasks/steps | No | No |
> | registries/buildtasks/steps/listbuildarguments | No | No |
> | registries/eventgridfilters | No | No |
> | registries/exportpipelines | No | No |
> | registries/generatecredentials | No | No |
> | registries/getbuildsourceuploadurl | No | No |
> | registries/getcredentials | No | No |
> | registries/importimage | No | No |
> | registries/importpipelines | No | No |
> | registries/listbuildsourceuploadurl | No | No |
> | registries/listcredentials | No | No |
> | registries/listpolicies | No | No |
> | registries/listusages | No | No |
> | registries/pipelineruns | No | No |
> | registries/privateendpointconnectionproxies | No | No |
> | registries/privateendpointconnectionproxies/validate | No | No |
> | registries/privateendpointconnections | No | No |
> | registries/privatelinkresources | No | No |
> | registries/queuebuild | No | No |
> | registries/regeneratecredential | No | No |
> | registries/regeneratecredentials | No | No |
> | registries/replications | Sí | Sí |
> | registries/runs | No | No |
> | registries/runs/cancel | No | No |
> | registries/runs/listlogsasurl | No | No |
> | registries/schedulerun | No | No |
> | registries/scopemaps | No | No |
> | registries / taskruns | No | No |
> | registries/taskruns/listdetails | No | No |
> | registries/tasks | Sí | Sí |
> | registries/tasks/listdetails | No | No |
> | registries/tokens | No | No |
> | registries/updatepolicies | No | No |
> | registries/webhooks | Sí | Sí |
> | registries/webhooks/getcallbackconfig | No | No |
> | registries/webhooks/listevents | No | No |
> | registries/webhooks/ping | No | No |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | containerservices | No | No |
> | locations | No | No |
> | locations/openshiftclusters | No | No |
> | locations/operationresults | No | No |
> | locations/operations | No | No |
> | locations/orchestrators | No | No |
> | managedclusters | No | No |
> | openshiftmanagedclusters | No | No |
> | operaciones | No | No |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | No | No |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | alerts | No | No |
> | billingaccounts | No | No |
> | budgets | No | No |
> | cloudconnectors | No | No |
> | conectores | Sí | Sí |
> | departments | No | No |
> | dimensions | No | No |
> | enrollmentaccounts | No | No |
> | exports | No | No |
> | externalbillingaccounts | No | No |
> | externalbillingaccounts/alerts | No | No |
> | externalbillingaccounts/dimensions | No | No |
> | externalbillingaccounts/forecast | No | No |
> | externalbillingaccounts/query | No | No |
> | externalsubscriptions | No | No |
> | externalsubscriptions/alerts | No | No |
> | externalsubscriptions/dimensions | No | No |
> | externalsubscriptions/forecast | No | No |
> | externalsubscriptions/query | No | No |
> | forecast | No | No |
> | operaciones | No | No |
> | Query | No | No |
> | registro | No | No |
> | reportconfigs | No | No |
> | reports | No | No |
> | configuración | No | No |
> | showbackrules | No | No |
> | views | No | No |

## <a name="microsoftcostmanagementexports"></a>Microsoft.CostManagementExports

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hubs | No | No |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |
> | Solicitudes | No | No |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | associations | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | resourceproviders | Sí | Sí |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | jobs | No | No |
> | locations | No | No |
> | locations/availableskus | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/regionconfiguration | No | No |
> | locations/validateaddress | No | No |
> | locations/validateinputs | No | No |
> | operaciones | No | No |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availableskus | No | No |
> | databoxedgedevices | Sí | Sí |
> | databoxedgedevices/checknameavailability | No | No |
> | operaciones | No | No |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/getnetworkpolicies | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | workspaces | No | No |
> | workspaces/dbworkspaces | No | No |
> | workspaces/virtualnetworkpeerings | No | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | catalogs | Sí | Sí |
> | checknameavailability | No | No |
> | datacatalogs | No | No |
> | locations | No | No |
> | locations/jobs | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | connectionmanagers | No | No |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | packages | No | No |
> | plans | No | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checkazuredatafactorynameavailability | No | No |
> | checkdatafactorynameavailability | No | No |
> | datafactories | Sí | Sí |
> | datafactories/diagnosticsettings | No | No |
> | datafactories/metricdefinitions | No | No |
> | datafactoryschema | No | No |
> | factories | Sí | Sí |
> | factories/integrationruntimes | No | No |
> | locations | No | No |
> | locations/configurefactoryrepo | No | No |
> | locations/getfeaturevalue | No | No |
> | operaciones | No | No |

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datalakeaccounts | No | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | accounts/datalakestoreaccounts | No | No |
> | accounts/storageaccounts | No | No |
> | accounts/storageaccounts/containers | No | No |
> | accounts/storageaccounts/containers/listsastokens | No | No |
> | locations | No | No |
> | locations/capability | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | accounts/eventgridfilters | No | No |
> | accounts/firewallrules | No | No |
> | locations | No | No |
> | locations/capability | No | No |
> | locations/checknameavailability | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operationresults | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | services | No | No |
> | services/projects | No | No |
> | slots | No | No |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | backupvaults | No | No |
> | locations | No | No |
> | operaciones | No | No |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | accounts/shares | No | No |
> | accounts/shares/datasets | No | No |
> | accounts/shares/invitations | No | No |
> | accounts/shares/providersharesubscriptions | No | No |
> | accounts/shares/synchronizationsettings | No | No |
> | accounts/sharesubscriptions | No | No |
> | accounts/sharesubscriptions/consumersourcedatasets | No | No |
> | accounts/sharesubscriptions/datasetmappings | No | No |
> | accounts/sharesubscriptions/triggers | No | No |
> | listinvitations | No | No |
> | locations | No | No |
> | locations/consumerinvitations | No | No |
> | locations/operationresults | No | No |
> | locations/rejectinvitation | No | No |
> | operaciones | No | No |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/azureasyncoperation | No | No |
> | locations/operationresults | No | No |
> | locations/performancetiers | No | No |
> | locations/privateendpointconnectionazureasyncoperation | No | No |
> | locations/privateendpointconnectionoperationresults | No | No |
> | locations/privateendpointconnectionproxyazureasyncoperation | No | No |
> | locations/privateendpointconnectionproxyoperationresults | No | No |
> | locations/recommendedactionsessionsazureasyncoperation | No | No |
> | locations/recommendedactionsessionsoperationresults | No | No |
> | locations/securityalertpoliciesazureasyncoperation | No | No |
> | locations/securityalertpoliciesoperationresults | No | No |
> | locations/serverkeyazureasyncoperation | No | No |
> | locations/serverkeyoperationresults | No | No |
> | operaciones | No | No |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/privateendpointconnectionproxies | No | No |
> | servers/privateendpointconnections | No | No |
> | servers/privatelinkresources | No | No |
> | servers/querytexts | No | No |
> | servers/recoverableservers | No | No |
> | servers/topquerystatistics | No | No |
> | servers/virtualnetworkrules | No | No |
> | servers/waitstatistics | No | No |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/administratorazureasyncoperation | No | No |
> | locations/administratoroperationresults | No | No |
> | locations/azureasyncoperation | No | No |
> | locations/operationresults | No | No |
> | locations/performancetiers | No | No |
> | locations/privateendpointconnectionazureasyncoperation | No | No |
> | locations/privateendpointconnectionoperationresults | No | No |
> | locations/privateendpointconnectionproxyazureasyncoperation | No | No |
> | locations/privateendpointconnectionproxyoperationresults | No | No |
> | locations/recommendedactionsessionsazureasyncoperation | No | No |
> | locations/recommendedactionsessionsoperationresults | No | No |
> | locations/securityalertpoliciesazureasyncoperation | No | No |
> | locations/securityalertpoliciesoperationresults | No | No |
> | locations/serverkeyazureasyncoperation | No | No |
> | locations/serverkeyoperationresults | No | No |
> | operaciones | No | No |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/keys | No | No |
> | servers/privateendpointconnectionproxies | No | No |
> | servers/privateendpointconnections | No | No |
> | servers/privatelinkresources | No | No |
> | servers/querytexts | No | No |
> | servers/recoverableservers | No | No |
> | servers/topquerystatistics | No | No |
> | servers/virtualnetworkrules | No | No |
> | servers/waitstatistics | No | No |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/administratorazureasyncoperation | No | No |
> | locations/administratoroperationresults | No | No |
> | locations/azureasyncoperation | No | No |
> | locations/operationresults | No | No |
> | locations/performancetiers | No | No |
> | locations/privateendpointconnectionazureasyncoperation | No | No |
> | locations/privateendpointconnectionoperationresults | No | No |
> | locations/privateendpointconnectionproxyazureasyncoperation | No | No |
> | locations/privateendpointconnectionproxyoperationresults | No | No |
> | locations/recommendedactionsessionsazureasyncoperation | No | No |
> | locations/recommendedactionsessionsoperationresults | No | No |
> | locations/securityalertpoliciesazureasyncoperation | No | No |
> | locations/securityalertpoliciesoperationresults | No | No |
> | locations/serverkeyazureasyncoperation | No | No |
> | locations/serverkeyoperationresults | No | No |
> | operaciones | No | No |
> | servergroups | No | No |
> | servers | Sí | Sí |
> | servers/advisors | No | No |
> | servers/keys | No | No |
> | servers/privateendpointconnectionproxies | No | No |
> | servers/privateendpointconnections | No | No |
> | servers/privatelinkresources | No | No |
> | servers/querytexts | No | No |
> | servers/recoverableservers | No | No |
> | servers/topquerystatistics | No | No |
> | servers/virtualnetworkrules | No | No |
> | servers/waitstatistics | No | No |
> | serversv2 | Sí | Sí |
> | singleservers | Sí | Sí |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | artifactsources | Sí | Sí |
> | operationresults | No | No |
> | operaciones | No | No |
> | rollouts | Sí | Sí |
> | servicetopologies | Sí | Sí |
> | servicetopologies/services | Sí | Sí |
> | servicetopologies/services/serviceunits | Sí | Sí |
> | steps | Sí | Sí |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applicationgroups | Sí | Sí |
> | applicationgroups/applications | No | No |
> | applicationgroups/desktops | No | No |
> | applicationgroups/startmenuitems | No | No |
> | hostpools | Sí | Sí |
> | hostpools/sessionhosts | No | No |
> | hostpools/sessionhosts/usersessions | No | No |
> | hostpools/usersessions | No | No |
> | operaciones | No | No |
> | workspaces | Sí | Sí |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | checkprovisioningservicenameavailability | No | No |
> | elasticpools | No | No |
> | elasticpools/iothubtenants | No | No |
> | iothubs | Sí | Sí |
> | iothubs/eventgridfilters | No | No |
> | iothubs/securitysettings | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | provisioningservices | Sí | Sí |
> | usages | No | No |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | pipelines | Sí | Sí |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | controllers | Sí | Sí |
> | controllers/listconnectiondetails | No | No |
> | locations | No | No |
> | locations/checkcontainerhostmapping | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | labcenters | No | No |
> | labs | Sí | No |
> | labs/environments | Sí | Sí |
> | labs/servicerunners | Sí | Sí |
> | labs/virtualmachines | Sí | No |
> | locations | No | No |
> | locations/operations | No | No |
> | operaciones | No | No |
> | schedules | Sí | Sí |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | digitaltwinsinstances | No | No |
> | digitaltwinsinstances/operationresults | No | No |
> | locations | No | No |
> | operaciones | No | No |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | databaseaccountnames | No | No |
> | databaseaccounts | Sí | Sí |
> | locations | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operationresults | No | No |
> | locations/operationsstatus | No | No |
> | operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checkdomainavailability | No | No |
> | domains | Sí | Sí |
> | domains/domainownershipidentifiers | No | No |
> | generatessorequest | No | No |
> | listdomainrecommendations | No | No |
> | operaciones | No | No |
> | topleveldomains | No | No |
> | validatedomainregistrationinformation | No | No |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |
> | services | Sí | Sí |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | domains | Sí | Sí |
> | domains/topics | No | No |
> | eventsubscriptions | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. |
> | extensiontopics | No | No |
> | locations | No | No |
> | locations/eventsubscriptions | No | No |
> | locations/operationresults | No | No |
> | locations/operationsstatus | No | No |
> | locations/topictypes | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | operationsstatus | No | No |
> | partnernamespaces | Sí | Sí |
> | partnernamespaces/eventchannels | No | No |
> | partnerregistrations | No | No |
> | partnertopics | Sí | Sí |
> | partnertopics/eventsubscriptions | No | No |
> | systemtopics | Sí | Sí |
> | systemtopics/eventsubscriptions | No | No |
> | topics | Sí | Sí |
> | topictypes | No | No |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availableclusterregions | No | No |
> | checknameavailability | No | No |
> | checknamespaceavailability | No | No |
> | clusters | Sí | Sí |
> | locations | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/disasterrecoveryconfigs | No | No |
> | namespaces/disasterrecoveryconfigs/checknameavailability | No | No |
> | namespaces/eventhubs | No | No |
> | namespaces/eventhubs/authorizationrules | No | No |
> | namespaces/eventhubs/consumergroups | No | No |
> | namespaces/networkrulesets | No | No |
> | operaciones | No | No |
> | sku | No | No |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | experimentworkspaces | No | No |
> | locations | No | No |
> | locations/operations | No | No |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | espacios de nombres | Sí | Sí |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | featureproviders | No | No |
> | features | No | No |
> | operaciones | No | No |
> | providers | No | No |
> | subscriptionfeatureregistrations | No | No |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | automanagedaccounts | No | No |
> | automanagedvmconfigurationprofiles | No | No |
> | guestconfigurationassignments | No | No |
> | operaciones | No | No |
> | software | No | No |
> | softwareupdateprofile | No | No |
> | softwareupdates | No | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hanainstances | No | No |
> | locations | No | No |
> | locations/operations | No | No |
> | locations/operationsstatus | No | No |
> | operaciones | No | No |
> | sapmonitors | Sí | Sí |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | dedicatedhsms | No | No |
> | locations | No | No |
> | operaciones | No | No |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> Puede mover clústeres de HDInsight a una nueva suscripción o un nuevo grupo de recursos. Sin embargo, no puede mover entre suscripciones los recursos de red vinculados al clúster de HDInsight (por ejemplo, la red virtual, una NIC o un equilibrador de carga). Además, tampoco se puede mover a un nuevo grupo de recursos una NIC que está conectada a una máquina virtual del clúster.
>
> Al mover un clúster de HDInsight a una nueva suscripción, mueva primero otros recursos (por ejemplo, la cuenta de almacenamiento). Después, mover el clúster de HDInsight.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | Sí | Sí |
> | clusters/applications | No | No |
> | clusters/operationresults | No | No |
> | locations | No | No |
> | locations/azureasyncoperations | No | No |
> | locations/billingspecs | No | No |
> | locations/capabilities | No | No |
> | locations/operationresults | No | No |
> | locations/usages | No | No |
> | locations/validatecreaterequest | No | No |
> | operaciones | No | No |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |
> | services | Sí | Sí |
> | services/privateendpointconnections | No | No |
> | services/privatelinkresources | No | No |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatus | No | No |
> | machines | Sí | Sí |
> | machines/extensions | Sí | Sí |
> | operaciones | No | No |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datamanagers | Sí | Sí |
> | operaciones | No | No |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | devices | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | vnfs | No | No |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | components | No | No |
> | locations | No | No |
> | networkscopes | No | No |
> | operaciones | No | No |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | jobs | Sí | Sí |
> | locations | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftinsights"></a>microsoft.insights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | actiongroups | Sí | Sí |
> | activitylogalerts | No | No |
> | alertrules | Sí | Sí |
> | autoscalesettings | Sí | Sí |
> | baseline | No | No |
> | calculatebaseline | No | No |
> | components | Sí | Sí |
> | components/events | No | No |
> | components/linkedstorageaccounts | No | No |
> | components/metadata | No | No |
> | components/metrics | No | No |
> | components/pricingplans | No | No |
> | components/query | No | No |
> | datacollectionrules | No | No |
> | diagnosticsettings | No | No |
> | diagnosticsettingscategories | No | No |
> | eventcategories | No | No |
> | eventtypes | No | No |
> | extendeddiagnosticsettings | No | No |
> | guestdiagnosticsettings | No | No |
> | listmigrationdate | No | No |
> | locations | No | No |
> | locations/operationresults | No | No |
> | logdefinitions | No | No |
> | logprofiles | No | No |
> | logs | No | No |
> | metricalerts | No | No |
> | metricbaselines | No | No |
> | metricbatch | No | No |
> | metricdefinitions | No | No |
> | metricnamespaces | No | No |
> | Métricas | No | No |
> | migratealertrules | No | No |
> | migratetonewpricingmodel | No | No |
> | myworkbooks | No | No |
> | notificationgroups | No | No |
> | operaciones | No | No |
> | privatelinkscopeoperationstatuses | No | No |
> | privatelinkscopes | No | No |
> | privatelinkscopes/privateendpointconnectionproxies | No | No |
> | privatelinkscopes/privateendpointconnections | No | No |
> | privatelinkscopes/scopedresources | No | No |
> | rollbacktolegacypricingmodel | No | No |
> | scheduledqueryrules | Sí | Sí |
> | Topología | No | No |
> | transacciones | No | No |
> | vminsightsonboardingstatuses | No | No |
> | webtests | Sí | Sí |
> | webtests/gettestresultfile | No | No |
> | workbooks | Sí | Sí |
> | workbooktemplates | Sí | Sí |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | apptemplates | No | No |
> | checknameavailability | No | No |
> | checksubdomainavailability | No | No |
> | iotapps | Sí | Sí |
> | operaciones | No | No |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | Sí | Sí |
> | graph | Sí | Sí |
> | operaciones | No | No |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> Los almacenes de claves usados para el cifrado de discos no se pueden mover a grupos de recursos de la misma suscripción ni entre suscripciones.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | deletedvaults | No | No |
> | hsmpools | No | No |
> | locations | No | No |
> | locations/deletedvaults | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/operationresults | No | No |
> | managedhsms | No | No |
> | operaciones | No | No |
> | vaults | Sí | Sí |
> | vaults/accesspolicies | No | No |
> | vaults/eventgridfilters | No | No |
> | vaults/secrets | No | No |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | connectedclusters | Sí | Sí |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | registeredsubscriptions | No | No |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | sourcecontrolconfigurations | No | No |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | Sí | Sí |
> | clusters/attacheddatabaseconfigurations | No | No |
> | clusters/databases | No | No |
> | clusters/databases/dataconnections | No | No |
> | clusters/databases/eventhubconnections | No | No |
> | clusters / databases / principalassignments | No | No |
> | clusters / principalassignments | No | No |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | labaccounts | No | No |
> | locations | No | No |
> | locations/operations | No | No |
> | operaciones | No | No |
> | users | No | No |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hostingenvironments | No | No |
> | integrationaccounts | Sí | Sí |
> | integrationserviceenvironments | Sí | No |
> | integrationserviceenvironments / managedapis | Sí | No |
> | isolatedenvironments | No | No |
> | locations | No | No |
> | locations/workflows | No | No |
> | operaciones | No | No |
> | workflows | Sí | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | commitmentplans | No | No |
> | locations | No | No |
> | locations/operations | No | No |
> | locations/operationsstatus | No | No |
> | operaciones | No | No |
> | webservices | Sí | No |
> | workspaces | Sí | Sí |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operationalizationclusters | No | No |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |
> | accounts/workspaces | No | No |
> | accounts/workspaces/projects | No | No |
> | teamaccounts | No | No |
> | teamaccounts/workspaces | No | No |
> | teamaccounts/workspaces/projects | No | No |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/computeoperationsstatus | No | No |
> | locations/quotas | No | No |
> | locations/updatequotas | No | No |
> | locations/usages | No | No |
> | locations/vmsizes | No | No |
> | locations/workspaceoperationsstatus | No | No |
> | operaciones | No | No |
> | workspaces | No | No |
> | workspaces/computes | No | No |
> | workspaces/eventgridfilters | No | No |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applyupdates | No | No |
> | configurationassignments | No | No |
> | maintenanceconfigurations | Sí | Sí |
> | updates | No | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | Identidades | No | No |
> | operaciones | No | No |
> | userassignedidentities | No | No |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managednetworks | No | No |
> | managednetworks / managednetworkgroups | No | No |
> | managednetworks / managednetworkpeeringpolicies | No | No |
> | notificación | No | No |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | No | No |
> | operaciones | No | No |
> | operationstatuses | No | No |
> | registrationassignments | No | No |
> | registrationdefinitions | No | No |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | getentities | No | No |
> | managementgroups | No | No |
> | managementgroups/settings | No | No |
> | operationresults | No | No |
> | operationresults/asyncoperation | No | No |
> | operaciones | No | No |
> | resources | No | No |
> | starttenantbackfill | No | No |
> | tenantbackfillstatus | No | No |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | accounts/eventgridfilters | No | No |
> | accounts / privateatlases | Sí | Sí |
> | operaciones | No | No |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | listavailableoffers | No | No |
> | offers | No | No |
> | offertypes | No | No |
> | offertypes/publishers | No | No |
> | offertypes/publishers/offers | No | No |
> | offertypes/publishers/offers/plans | No | No |
> | offertypes/publishers/offers/plans/agreements | No | No |
> | offertypes/publishers/offers/plans/configs | No | No |
> | offertypes/publishers/offers/plans/configs/importimage | No | No |
> | operaciones | No | No |
> | privategalleryitems | No | No |
> | privatestoreclient | No | No |
> | privatestores | No | No |
> | privatestores/offers | No | No |
> | products | No | No |
> | publishers | No | No |
> | publishers/offers | No | No |
> | publishers/offers/amendments | No | No |
> | registro | No | No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | classicdevservices | No | No |
> | listcommunicationpreference | No | No |
> | operaciones | No | No |
> | updatecommunicationpreference | No | No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | agreements | No | No |
> | offertypes | No | No |
> | operaciones | No | No |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | mediaservices | Sí | Sí |
> | mediaservices/accountfilters | No | No |
> | mediaservices/assets | No | No |
> | mediaservices/assets/assetfilters | No | No |
> | mediaservices/contentkeypolicies | No | No |
> | mediaservices/eventgridfilters | No | No |
> | mediaservices/liveeventoperations | No | No |
> | mediaservices/liveevents | Sí | Sí |
> | mediaservices/liveevents/liveoutputs | No | No |
> | mediaservices/liveoutputoperations | No | No |
> | mediaservices/streamingendpointoperations | No | No |
> | mediaservices/streamingendpoints | Sí | Sí |
> | mediaservices/streaminglocators | No | No |
> | mediaservices/streamingpolicies | No | No |
> | mediaservices/transforms | No | No |
> | mediaservices/transforms/jobs | No | No |
> | operaciones | No | No |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | appclusters | No | No |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | assessmentprojects | No | No |
> | locations | No | No |
> | locations/assessmentoptions | No | No |
> | locations/checknameavailability | No | No |
> | migrateprojects | No | No |
> | movecollections | No | No |
> | operaciones | No | No |
> | projects | No | No |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | holographicsbroadcastaccounts | No | No |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | objectunderstandingaccounts | No | No |
> | operaciones | No | No |
> | remoterenderingaccounts | Sí | Sí |
> | spatialanchorsaccounts | Sí | Sí |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | netappaccounts | No | No |
> | netappaccounts / backuppolicies | No | No |
> | netappaccounts/capacitypools | No | No |
> | netappaccounts/capacitypools/volumes | No | No |
> | netappaccounts/capacitypools/volumes/mounttargets | No | No |
> | netappaccounts/capacitypools/volumes/snapshots | No | No |
> | operaciones | No | No |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!IMPORTANT]
> Consulte [Guía de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applicationgatewayavailablerequestheaders | No | No |
> | applicationgatewayavailableresponseheaders | No | No |
> | applicationgatewayavailableservervariables | No | No |
> | applicationgatewayavailablessloptions | No | No |
> | applicationgatewayavailablewafrulesets | No | No |
> | applicationgateways | No | No |
> | applicationgatewaywebapplicationfirewallpolicies | No | No |
> | applicationsecuritygroups | Sí | Sí |
> | azurefirewallfqdntags | No | No |
> | azurefirewalls | No | No |
> | bastionhosts | No | No |
> | bgpservicecommunities | No | No |
> | checkfrontdoornameavailability | No | No |
> | checktrafficmanagernameavailability | No | No |
> | connections | Sí | Sí |
> | ddoscustompolicies | Sí | Sí |
> | ddosprotectionplans | No | No |
> | dnsoperationresults | No | No |
> | dnsoperationstatuses | No | No |
> | dnszones | Sí | Sí |
> | dnszones/a | No | No |
> | dnszones/aaaa | No | No |
> | dnszones/all | No | No |
> | dnszones/caa | No | No |
> | dnszones/cname | No | No |
> | dnszones/mx | No | No |
> | dnszones/ns | No | No |
> | dnszones/ptr | No | No |
> | dnszones/recordsets | No | No |
> | dnszones/soa | No | No |
> | dnszones/srv | No | No |
> | dnszones/txt | No | No |
> | expressroutecircuits | No | No |
> | expressroutegateways | No | No |
> | expressrouteserviceproviders | No | No |
> | firewallpolicies | Sí | Sí |
> | frontdooroperationresults | No | No |
> | frontdoors | No | No |
> | frontdoors/frontendendpoints | No | No |
> | frontdoorwebapplicationfirewallmanagedrulesets | No | No |
> | frontdoorwebapplicationfirewallpolicies | No | No |
> | getdnsresourcereference | No | No |
> | internalnotify | No | No |
> | ipallocations | Sí | Sí |
> | ipgroups | Sí | Sí |
> | loadbalancers | Sí: SKU básico<br>No: SKU estándar | Sí: SKU básico<br>No: SKU estándar |
> | localnetworkgateways | Sí | Sí |
> | locations | No | No |
> | locations/autoapprovedprivatelinkservices | No | No |
> | locations/availabledelegations | No | No |
> | locations/availableprivateendpointtypes | No | No |
> | locations/availableservicealiases | No | No |
> | locations/baremetaltenants | No | No |
> | locations/batchnotifyprivateendpointsforresourcemove | No | No |
> | locations/batchvalidateprivateendpointsforresourcemove | No | No |
> | locations/checkacceleratednetworkingsupport | No | No |
> | locations/checkdnsnameavailability | No | No |
> | locations/checkprivatelinkservicevisibility | No | No |
> | locations/commitinternalazurenetworkmanagerconfiguration | No | No |
> | locations/effectiveresourceownership | No | No |
> | locations/nfvoperationresults | No | No |
> | locations/nfvoperations | No | No |
> | locations/operationresults | No | No |
> | locations/operations | No | No |
> | locations/servicetags | No | No |
> | locations/setresourceownership | No | No |
> | locations/supportedvirtualmachinesizes | No | No |
> | locations/usages | No | No |
> | locations/validateresourceownership | No | No |
> | locations/virtualnetworkavailableendpointservices | No | No |
> | natgateways | No | No |
> | networkexperimentprofiles | No | No |
> | networkintentpolicies | Sí | Sí |
> | networkinterfaces | Sí | Sí |
> | networkprofiles | No | No |
> | networksecuritygroups | Sí | Sí |
> | networkwatchers | Sí | No |
> | networkwatchers/connectionmonitors | Sí | No |
> | networkwatchers / flowlogs | Sí | No |
> | networkwatchers/pingmeshes | Sí | No |
> | operaciones | No | No |
> | p2svpngateways | No | No |
> | privatednsoperationresults | No | No |
> | privatednsoperationstatuses | No | No |
> | privatednszones | Sí | Sí |
> | privatednszones/a | No | No |
> | privatednszones/aaaa | No | No |
> | privatednszones/all | No | No |
> | privatednszones/cname | No | No |
> | privatednszones/mx | No | No |
> | privatednszones/ptr | No | No |
> | privatednszones/soa | No | No |
> | privatednszones/srv | No | No |
> | privatednszones/txt | No | No |
> | privatednszones/virtualnetworklinks | Sí | Sí |
> | privatednszonesinternal | No | No |
> | privateendpointredirectmaps | No | No |
> | privateendpoints | Sí | Sí |
> | privatelinkservices | No | No |
> | publicipaddresses | Sí: SKU básico<br>No: SKU estándar | Sí: SKU básico<br>No: SKU estándar |
> | publicipprefixes | Sí | Sí |
> | routefilters | No | No |
> | routetables | Sí | Sí |
> | securitypartnerproviders | Sí | Sí |
> | serviceendpointpolicies | Sí | Sí |
> | trafficmanagergeographichierarchies | No | No |
> | trafficmanagerprofiles | Sí | Sí |
> | trafficmanagerprofiles/heatmaps | No | No |
> | trafficmanagerusermetricskeys | No | No |
> | virtualhubs | No | No |
> | virtualnetworkgateways | Sí | Sí |
> | virtualnetworks | Sí | Sí |
> | virtualnetworktaps | No | No |
> | virtualrouters | Sí | Sí |
> | virtualwans | No | No |
> | vpngateways (Virtual WAN) | No | No |
> | vpnserverconfigurations | No | No |
> | vpnsites (Virtual WAN) | No | No |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | checknamespaceavailability | No | No |
> | espacios de nombres | Sí | Sí |
> | namespaces/notificationhubs | Sí | Sí |
> | operationresults | No | No |
> | operaciones | No | No |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | osnamespaces | Sí | Sí |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hypervsites | No | No |
> | importsites | No | No |
> | operaciones | No | No |
> | serversites | No | No |
> | vmwaresites | No | No |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).
>
> Las áreas de trabajo que tienen una cuenta de automatización vinculada no se pueden mover. Antes de comenzar una operación de movimiento, asegúrese de desvincular todas las cuentas de automatización.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |
> | deletedworkspaces | No | No |
> | linktargets | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | storageinsightconfigs | No | No |
> | workspaces | Sí | Sí |
> | workspaces/datasources | No | No |
> | workspaces/linkedservices | No | No |
> | workspaces/linkedstorageaccounts | No | No |
> | workspaces/metadata | No | No |
> | workspaces/query | No | No |
> | workspaces/scopedprivatelinkproxies | No | No |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managementassociations | No | No |
> | managementconfigurations | Sí | Sí |
> | operaciones | No | No |
> | solutions | Sí | Sí |
> | views | Sí | Sí |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checkserviceprovideravailability | No | No |
> | legacypeerings | No | No |
> | operaciones | No | No |
> | peerasns | No | No |
> | peeringlocations | No | No |
> | peerings | No | No |
> | peeringservicecountries | No | No |
> | peeringservicelocations | No | No |
> | peeringserviceproviders | No | No |
> | peeringservices | No | No |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | asyncoperationresults | No | No |
> | operaciones | No | No |
> | policyevents | No | No |
> | policystates | No | No |
> | policytrackedresources | No | No |
> | remediations | No | No |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | consoles | No | No |
> | dashboards | Sí | Sí |
> | locations | No | No |
> | locations/consoles | No | No |
> | locations/usersettings | No | No |
> | operaciones | No | No |
> | usersettings | No | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | workspacecollections | Sí | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capacities | Sí | Sí |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |

## <a name="microsoftpowerplatform"></a>Microsoft.PowerPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |
> | checknameavailability | No | No |
> | operaciones | No | No |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availableaccounts | No | No |
> | providerregistrations | No | No |
> | providerregistrations/resourcetyperegistrations | No | No |
> | rollouts | No | No |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | workspaces | No | No |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!IMPORTANT]
> Vea [Guía de movimiento de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | backupprotecteditems | No | No |
> | locations | No | No |
> | locations/allocatedstamp | No | No |
> | locations/allocatestamp | No | No |
> | locations/backupaadproperties | No | No |
> | locations/backupcrossregionrestore | No | No |
> | locations/backupcrrjob | No | No |
> | locations/backupcrrjobs | No | No |
> | locations/backupcrroperationresults | No | No |
> | locations/backupcrroperationsstatus | No | No |
> | locations/backupprevalidateprotection | No | No |
> | locations/backupstatus | No | No |
> | locations/backupvalidatefeatures | No | No |
> | locations/checknameavailability | No | No |
> | operaciones | No | No |
> | replicationeligibilityresults | No | No |
> | vaults | Sí | Sí |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationresults | No | No |
> | locations/operationsstatus | No | No |
> | openshiftclusters | No | No |
> | operaciones | No | No |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/hybridconnections | No | No |
> | namespaces/hybridconnections/authorizationrules | No | No |
> | namespaces/privateendpointconnections | No | No |
> | namespaces/wcfrelays | No | No |
> | namespaces/wcfrelays/authorizationrules | No | No |
> | operaciones | No | No |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |
> | Consultas | Sí | Sí |
> | resourcechangedetails | No | No |
> | resourcechanges | No | No |
> | resources | No | No |
> | resourceshistory | No | No |
> | subscriptionsstatus | No | No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availabilitystatuses | No | No |
> | childavailabilitystatuses | No | No |
> | childresources | No | No |
> | emergingissues | No | No |
> | events | No | No |
> | metadata | No | No |
> | Notificaciones | No | No |
> | operaciones | No | No |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | calculatetemplatehash | No | No |
> | checkpolicycompliance | No | No |
> | checkresourcename | No | No |
> | deployments | No | No |
> | deployments/operations | No | No |
> | deploymentscripts | No | No |
> | deploymentscripts/logs | No | No |
> | vínculos | No | No |
> | locations | No | No |
> | locations/deploymentscriptoperationresults | No | No |
> | notifyresourcejobs | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | providers | No | No |
> | resourcegroups | No | No |
> | resources | No | No |
> | subscriptions | No | No |
> | subscriptions/locations | No | No |
> | subscriptions/operationresults | No | No |
> | subscriptions/providers | No | No |
> | subscriptions/resourcegroups | No | No |
> | subscriptions/resourcegroups/resources | No | No |
> | subscriptions/resources | No | No |
> | subscriptions/tagnames | No | No |
> | subscriptions/tagnames/tagvalues | No | No |
> | etiquetas | No | No |
> | templatespecs | No | No |
> | templatespecs/versions | No | No |
> | tenants | No | No |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | Sí | No |
> | checkmoderneligibility | No | No |
> | checknameavailability | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | saasresources | No | No |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> No puede mover varios recursos de Search de regiones diferentes en una operación. En su lugar, muévalos en operaciones independientes.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | checkservicenameavailability | No | No |
> | operaciones | No | No |
> | resourcehealthmetadata | No | No |
> | searchservices | Sí | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | adaptivenetworkhardenings | No | No |
> | advancedthreatprotectionsettings | No | No |
> | alerts | No | No |
> | allowedconnections | No | No |
> | applicationwhitelistings | No | No |
> | assessmentmetadata | No | No |
> | assessments | No | No |
> | autodismissalertsrules | No | No |
> | automations | Sí | Sí |
> | autoprovisioningsettings | No | No |
> | complianceresults | No | No |
> | compliances | No | No |
> | datacollectionagents | No | No |
> | devicesecuritygroups | No | No |
> | discoveredsecuritysolutions | No | No |
> | externalsecuritysolutions | No | No |
> | informationprotectionpolicies | No | No |
> | iotsecuritysolutions | Sí | Sí |
> | iotsecuritysolutions/analyticsmodels | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedalerts | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedrecommendations | No | No |
> | jitnetworkaccesspolicies | No | No |
> | locations | No | No |
> | locations/alerts | No | No |
> | locations/allowedconnections | No | No |
> | locations/applicationwhitelistings | No | No |
> | locations/discoveredsecuritysolutions | No | No |
> | locations/externalsecuritysolutions | No | No |
> | locations/jitnetworkaccesspolicies | No | No |
> | locations/securitysolutions | No | No |
> | locations/securitysolutionsreferencedata | No | No |
> | locations/tasks | No | No |
> | locations/topologies | No | No |
> | operaciones | No | No |
> | directivas | No | No |
> | pricings | No | No |
> | regulatorycompliancestandards | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols/regulatorycomplianceassessments | No | No |
> | securitycontacts | No | No |
> | securitysolutions | No | No |
> | securitysolutionsreferencedata | No | No |
> | securitystatuses | No | No |
> | securitystatusessummaries | No | No |
> | servervulnerabilityassessments | No | No |
> | configuración | No | No |
> | subassessments | No | No |
> | tareas | No | No |
> | topologies | No | No |
> | workspacesettings | No | No |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aggregations | No | No |
> | alertrules | No | No |
> | alertruletemplates | No | No |
> | automationrules | No | No |
> | bookmarks | No | No |
> | cases | No | No |
> | dataconnectors | No | No |
> | dataconnectorscheckrequirements | No | No |
> | entities | No | No |
> | entityqueries | No | No |
> | incidents | No | No |
> | officeconsents | No | No |
> | operaciones | No | No |
> | configuración | No | No |
> | threatintelligence | No | No |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | consoleservices | No | No |
> | locations | No | No |
> | locations/consoleservices | No | No |
> | operaciones | No | No |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | gateways | No | No |
> | nodes | No | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | checknamespaceavailability | No | No |
> | locations | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | espacios de nombres | Sí | Sí |
> | namespaces/authorizationrules | No | No |
> | namespaces/disasterrecoveryconfigs | No | No |
> | namespaces/disasterrecoveryconfigs/checknameavailability | No | No |
> | namespaces/eventgridfilters | No | No |
> | namespaces/networkrulesets | No | No |
> | namespaces/queues | No | No |
> | namespaces/queues/authorizationrules | No | No |
> | namespaces/topics | No | No |
> | namespaces/topics/authorizationrules | No | No |
> | namespaces/topics/subscriptions | No | No |
> | namespaces/topics/subscriptions/rules | No | No |
> | operaciones | No | No |
> | premiummessagingregions | No | No |
> | sku | No | No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | No | No |
> | clusters | Sí | Sí |
> | clusters/applications | No | No |
> | containergroups | No | No |
> | containergroupsets | No | No |
> | edgeclusters | No | No |
> | locations | No | No |
> | locations/clusterversions | No | No |
> | locations/environments | No | No |
> | locations/operationresults | No | No |
> | locations/operations | No | No |
> | managedclusters | No | No |
> | networks | No | No |
> | operaciones | No | No |
> | secretstores | No | No |
> | volumes | No | No |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | Sí | Sí |
> | containergroups | No | No |
> | gateways | Sí | Sí |
> | locations | No | No |
> | locations/applicationoperations | No | No |
> | locations/gatewayoperations | No | No |
> | locations/networkoperations | No | No |
> | locations/secretoperations | No | No |
> | locations/volumeoperations | No | No |
> | networks | Sí | Sí |
> | operaciones | No | No |
> | secrets | Sí | Sí |
> | volumes | Sí | Sí |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | rollouts | No | No |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operationstatuses | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |
> | signalr | Sí | Sí |
> | signalr/eventgridfilters | No | No |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hybridusebenefits | No | No |
> | operaciones | No | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | No | No |
> | applications | No | No |
> | jitrequests | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> La base de datos y el servidor deben residir en el mismo grupo de recursos. Cuando se mueve un servidor SQL Server, se mueven también todas sus bases de datos. Este comportamiento se aplica a las bases de datos de Azure SQL Database y Azure Synapse Analytics.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | instancepools | No | No |
> | locations | Sí | Sí |
> | locations/administratorazureasyncoperation | No | No |
> | locations/administratoroperationresults | No | No |
> | locations/auditingsettingsazureasyncoperation | No | No |
> | locations/auditingsettingsoperationresults | No | No |
> | locations/capabilities | No | No |
> | locations/databaseazureasyncoperation | No | No |
> | locations/databaseoperationresults | No | No |
> | locations/databaserestoreazureasyncoperation | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/deletevirtualnetworkorsubnetsazureasyncoperation | No | No |
> | locations/deletevirtualnetworkorsubnetsoperationresults | No | No |
> | locations/dnsaliasasyncoperation | No | No |
> | locations/dnsaliasoperationresults | No | No |
> | locations/elasticpoolazureasyncoperation | No | No |
> | locations/elasticpooloperationresults | No | No |
> | locations/encryptionprotectorazureasyncoperation | No | No |
> | locations/encryptionprotectoroperationresults | No | No |
> | locations/extendedauditingsettingsazureasyncoperation | No | No |
> | locations/extendedauditingsettingsoperationresults | No | No |
> | locations/failovergroupazureasyncoperation | No | No |
> | locations/failovergroupoperationresults | No | No |
> | locations/firewallrulesazureasyncoperation | No | No |
> | locations/firewallrulesoperationresults | No | No |
> | locations/instancefailovergroupazureasyncoperation | No | No |
> | locations/instancefailovergroupoperationresults | No | No |
> | locations/instancefailovergroups | No | No |
> | locations/instancepoolazureasyncoperation | No | No |
> | locations/instancepooloperationresults | No | No |
> | locations/jobagentazureasyncoperation | No | No |
> | locations/jobagentoperationresults | No | No |
> | locations/longtermretentionbackupazureasyncoperation | No | No |
> | locations/longtermretentionbackupoperationresults | No | No |
> | locations/longtermretentionbackups | No | No |
> | locations/longtermretentionmanagedinstancebackupazureasyncoperation | No | No |
> | locations/longtermretentionmanagedinstancebackupoperationresults | No | No |
> | locations/longtermretentionmanagedinstancebackups | No | No |
> | locations/longtermretentionmanagedinstances | No | No |
> | locations/longtermretentionpolicyazureasyncoperation | No | No |
> | locations/longtermretentionpolicyoperationresults | No | No |
> | locations/longtermretentionservers | No | No |
> | locations/manageddatabaseazureasyncoperation | No | No |
> | locations/manageddatabasecompleterestoreazureasyncoperation | No | No |
> | locations/manageddatabasecompleterestoreoperationresults | No | No |
> | locations/manageddatabaseoperationresults | No | No |
> | locations/manageddatabaserestoreazureasyncoperation | No | No |
> | locations/manageddatabaserestoreoperationresults | No | No |
> | locations/managedinstanceazureasyncoperation | No | No |
> | locations/managedinstanceencryptionprotectorazureasyncoperation | No | No |
> | locations/managedinstanceencryptionprotectoroperationresults | No | No |
> | locations/managedinstancekeyazureasyncoperation | No | No |
> | locations/managedinstancekeyoperationresults | No | No |
> | locations/managedinstancelongtermretentionpolicyazureasyncoperation | No | No |
> | locations/managedinstancelongtermretentionpolicyoperationresults | No | No |
> | locations/managedinstanceoperationresults | No | No |
> | locations/managedinstancetdecertazureasyncoperation | No | No |
> | locations/managedinstancetdecertoperationresults | No | No |
> | locations/managedserversecurityalertpoliciesazureasyncoperation | No | No |
> | locations/managedserversecurityalertpoliciesoperationresults | No | No |
> | locations/managedshorttermretentionpolicyazureasyncoperation | No | No |
> | locations/managedshorttermretentionpolicyoperationresults | No | No |
> | locations/notifyazureasyncoperation | No | No |
> | locations/privateendpointconnectionazureasyncoperation | No | No |
> | locations/privateendpointconnectionoperationresults | No | No |
> | locations/privateendpointconnectionproxyazureasyncoperation | No | No |
> | locations/privateendpointconnectionproxyoperationresults | No | No |
> | locations/securityalertpoliciesazureasyncoperation | No | No |
> | locations/securityalertpoliciesoperationresults | No | No |
> | locations/serveradministratorazureasyncoperation | No | No |
> | locations/serveradministratoroperationresults | No | No |
> | locations/serverazureasyncoperation | No | No |
> | locations/serverkeyazureasyncoperation | No | No |
> | locations/serverkeyoperationresults | No | No |
> | locations/serveroperationresults | No | No |
> | locations/shorttermretentionpolicyazureasyncoperation | No | No |
> | locations/shorttermretentionpolicyoperationresults | No | No |
> | locations/syncagentoperationresults | No | No |
> | locations/syncdatabaseids | No | No |
> | locations/syncgroupoperationresults | No | No |
> | locations/syncmemberoperationresults | No | No |
> | locations/tdecertazureasyncoperation | No | No |
> | locations/tdecertoperationresults | No | No |
> | locations/usages | No | No |
> | locations/virtualclusterazureasyncoperation | No | No |
> | locations/virtualclusteroperationresults | No | No |
> | locations/virtualnetworkrulesazureasyncoperation | No | No |
> | locations/virtualnetworkrulesoperationresults | No | No |
> | locations/vulnerabilityassessmentscanazureasyncoperation | No | No |
> | locations/vulnerabilityassessmentscanoperationresults | No | No |
> | managedinstances | No | No |
> | managedinstances/administrators | No | No |
> | managedinstances/databases | No | No |
> | managedinstances/databases/backuplongtermretentionpolicies | No | No |
> | managedinstances/databases/vulnerabilityassessments | No | No |
> | managedinstances/metricdefinitions | No | No |
> | managedinstances/metrics | No | No |
> | managedinstances/recoverabledatabases | No | No |
> | managedinstances/tdecertificates | No | No |
> | managedinstances/vulnerabilityassessments | No | No |
> | operaciones | No | No |
> | servers | Sí | Sí |
> | servers/administratoroperationresults | No | No |
> | servers/administrators | No | No |
> | servers/advisors | No | No |
> | servers/aggregateddatabasemetrics | No | No |
> | servers/auditingpolicies | No | No |
> | servers/auditingsettings | No | No |
> | servers/automatictuning | No | No |
> | servers/communicationlinks | No | No |
> | servers/connectionpolicies | No | No |
> | servers/databases | Sí | Sí |
> | servers/databases/advisors | No | No |
> | servers/databases/auditingpolicies | No | No |
> | servers/databases/auditingsettings | No | No |
> | servers/databases/auditrecords | No | No |
> | servers/databases/automatictuning | No | No |
> | servers/databases/backuplongtermretentionpolicies | No | No |
> | servers/databases/backupshorttermretentionpolicies | No | No |
> | servers/databases/connectionpolicies | No | No |
> | servers/databases/datamaskingpolicies | No | No |
> | servers/databases/datamaskingpolicies/rules | No | No |
> | servers/databases/extensions | No | No |
> | servers/databases/geobackuppolicies | No | No |
> | servers/databases/metricdefinitions | No | No |
> | servers/databases/metrics | No | No |
> | servers/databases/recommendedsensitivitylabels | No | No |
> | servers/databases/securityalertpolicies | No | No |
> | servers/databases/syncgroups | No | No |
> | servers/databases/syncgroups/syncmembers | No | No |
> | servers/databases/topqueries | No | No |
> | servers/databases/topqueries/querytext | No | No |
> | servers/databases/transparentdataencryption | No | No |
> | servers/databases/vulnerabilityassessment | No | No |
> | servers/databases/vulnerabilityassessments | No | No |
> | servers/databases/vulnerabilityassessmentscans | No | No |
> | servers/databases/vulnerabilityassessmentsettings | No | No |
> | servers/databases/workloadgroups | No | No |
> | servers/databasesecuritypolicies | No | No |
> | servers/disasterrecoveryconfiguration | No | No |
> | servers/dnsaliases | No | No |
> | servers/elasticpoolestimates | No | No |
> | servers/elasticpools | Sí | Sí |
> | servers/elasticpools/advisors | No | No |
> | servers/elasticpools/metricdefinitions | No | No |
> | servers/elasticpools/metrics | No | No |
> | servers/encryptionprotector | No | No |
> | servers/extendedauditingsettings | No | No |
> | servers/failovergroups | No | No |
> | servers/import | No | No |
> | servers/importexportoperationresults | No | No |
> | servers / jobaccounts | Sí | Sí |
> | servers / jobagents | Sí | Sí |
> | servers/jobagents/jobs | No | No |
> | servers/jobagents/jobs/executions | No | No |
> | servers/jobagents/jobs/steps | No | No |
> | servers/keys | No | No |
> | servers/operationresults | No | No |
> | servers/recommendedelasticpools | No | No |
> | servers/recoverabledatabases | No | No |
> | servers/restorabledroppeddatabases | No | No |
> | servers/securityalertpolicies | No | No |
> | servers/serviceobjectives | No | No |
> | servers/syncagents | No | No |
> | servers/tdecertificates | No | No |
> | servers/usages | No | No |
> | servers/virtualnetworkrules | No | No |
> | servers/vulnerabilityassessments | No | No |
> | virtualclusters | Sí | Sí |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/availabilitygrouplisteneroperationresults | No | No |
> | locations/operationtypes | No | No |
> | locations/sqlvirtualmachinegroupoperationresults | No | No |
> | locations/sqlvirtualmachineoperationresults | No | No |
> | operaciones | No | No |
> | sqlvirtualmachinegroups | Sí | Sí |
> | sqlvirtualmachinegroups/availabilitygrouplisteners | No | No |
> | sqlvirtualmachines | Sí | Sí |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | locations | No | No |
> | locations/asyncoperations | No | No |
> | locations/checknameavailability | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |
> | storageaccounts | Sí | Sí |
> | storageaccounts/blobservices | No | No |
> | storageaccounts/fileservices | No | No |
> | storageaccounts/listaccountsas | No | No |
> | storageaccounts/listservicesas | No | No |
> | storageaccounts/queueservices | No | No |
> | storageaccounts/services | No | No |
> | storageaccounts/services/metricdefinitions | No | No |
> | storageaccounts/tableservices | No | No |
> | usages | No | No |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | caches | No | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/checknameavailability | No | No |
> | locations/operationresults | No | No |
> | locations/operations | No | No |
> | locations/workflows | No | No |
> | operaciones | No | No |
> | storagesyncservices | Sí | Sí |
> | storagesyncservices/registeredservers | No | No |
> | storagesyncservices/syncgroups | No | No |
> | storagesyncservices/syncgroups/cloudendpoints | No | No |
> | storagesyncservices/syncgroups/serverendpoints | No | No |
> | storagesyncservices/workflows | No | No |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | No | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | No | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managers | No | No |
> | operaciones | No | No |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> Los trabajos de Stream Analytics no se pueden mover si se encuentran en estado de ejecución.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |
> | locations | No | No |
> | locations/quotas | No | No |
> | operaciones | No | No |
> | streamingjobs | Sí | Sí |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | environments | No | No |
> | environments/eventsources | No | No |
> | instances | No | No |
> | instances/environments | No | No |
> | instances/environments/eventsources | No | No |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | cancel | No | No |
> | createsubscription | No | No |
> | enable | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | rename | No | No |
> | subscriptiondefinitions | No | No |
> | subscriptionoperations | No | No |
> | subscriptions | No | No |

## <a name="microsoftsupport"></a>microsoft.support

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | operationresults | No | No |
> | operaciones | No | No |
> | operationsstatus | No | No |
> | services | No | No |
> | services/problemclassifications | No | No |
> | supporttickets | No | No |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | checknameavailability | No | No |
> | operaciones | No | No |
> | workspaces | Sí | Sí |
> | workspaces / bigdatapools | Sí | Sí |
> | workspaces/operationresults | No | No |
> | workspaces/operationstatuses | No | No |
> | workspaces / sqlpools | Sí | Sí |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | environments | Sí | Sí |
> | environments/accesspolicies | No | No |
> | environments/eventsources | Sí | Sí |
> | environments/referencedatasets | Sí | Sí |
> | operaciones | No | No |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | almacenes | Sí | Sí |
> | stores/accesspolicies | No | No |
> | stores/services | No | No |
> | stores/services/tokens | No | No |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | imagetemplates | No | No |
> | imagetemplates/runoutputs | No | No |
> | locations | No | No |
> | locations/operations | No | No |
> | operaciones | No | No |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!IMPORTANT]
> Para cambiar la suscripción de Azure DevOps, vea [Cambiar la suscripción de Azure usada para la facturación](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | account | No | No |
> | account/extension | No | No |
> | account/project | No | No |
> | checknameavailability | No | No |
> | operaciones | No | No |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | arczones | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | resourcepools | No | No |
> | vcenters | No | No |
> | virtualmachines | No | No |
> | virtualmachinetemplates | No | No |
> | virtualnetworks | No | No |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | dedicatedcloudnodes | No | No |
> | dedicatedcloudservices | No | No |
> | locations | No | No |
> | locations/availabilities | No | No |
> | locations/operationresults | No | No |
> | locations/privateclouds | No | No |
> | locations/privateclouds/resourcepools | No | No |
> | locations/privateclouds/virtualmachinetemplates | No | No |
> | locations/privateclouds/virtualnetworks | No | No |
> | locations/usages | No | No |
> | operaciones | No | No |
> | virtualmachines | No | No |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | devices | No | No |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | vnfs | No | No |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |
> | operaciones | No | No |
> | plans | No | No |
> | registeredsubscriptions | No | No |

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availablestacks | No | No |
> | billingmeters | No | No |
> | certificates | No | Sí |
> | checknameavailability | No | No |
> | connectiongateways | Sí | Sí |
> | connections | Sí | Sí |
> | customapis | Sí | Sí |
> | deletedsites | No | No |
> | deploymentlocations | No | No |
> | georegions | No | No |
> | hostingenvironments | No | No |
> | hostingenvironments/eventgridfilters | No | No |
> | hostingenvironments/multirolepools | No | No |
> | hostingenvironments/workerpools | No | No |
> | ishostingenvironmentnameavailable | No | No |
> | ishostnameavailable | No | No |
> | isusernameavailable | No | No |
> | kubeenvironments | Sí | Sí |
> | listsitesassignedtohostname | No | No |
> | locations | No | No |
> | locations/apioperations | No | No |
> | locations/connectiongatewayinstallations | No | No |
> | locations/deletedsites | No | No |
> | locations/deletevirtualnetworkorsubnets | No | No |
> | locations/extractapidefinitionfromwsdl | No | No |
> | locations/getnetworkpolicies | No | No |
> | locations/listwsdlinterfaces | No | No |
> | locations/managedapis | No | No |
> | locations/operationresults | No | No |
> | locations/operations | No | No |
> | locations/runtimes | No | No |
> | operaciones | No | No |
> | publishingusers | No | No |
> | de películas | No | No |
> | resourcehealthmetadata | No | No |
> | runtimes | No | No |
> | serverfarms | Sí | Sí |
> | serverfarms/eventgridfilters | No | No |
> | sites | Sí | Sí |
> | sites/eventgridfilters | No | No |
> | sites/hostnamebindings | No | No |
> | sites/networkconfig | No | No |
> | sites/premieraddons | Sí | Sí |
> | sites/slots | Sí | Sí |
> | sites/slots/eventgridfilters | No | No |
> | sites/slots/hostnamebindings | No | No |
> | sites/slots/networkconfig | No | No |
> | sourcecontrols | No | No |
> | staticsites | No | No |
> | validate | No | No |
> | verifyhostingenvironmentvnet | No | No |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | multipleactivationkeys | No | No |
> | operaciones | No | No |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | deviceservices | No | No |
> | operaciones | No | No |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | locations | No | No |
> | locations/operationstatuses | No | No |
> | operaciones | No | No |
> | workloads | No | No |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | components | No | No |
> | componentssummary | No | No |
> | monitorinstances | No | No |
> | monitorinstancessummary | No | No |
> | monitors | No | No |
> | notificationsettings | No | No |
> | operaciones | No | No |

## <a name="third-party-services"></a>Servicios de terceros

Actualmente los servicios de terceros no son compatibles con la operación de traslado.

## <a name="next-steps"></a>Pasos siguientes
Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

Para obtener los mismos datos como un archivo de valores separados por comas, descargue [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).
