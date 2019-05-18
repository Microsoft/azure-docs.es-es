---
title: Compatibilidad con la operación de traslado por tipo de recurso de Azure
description: Enumera los tipos de recursos de Azure que se pueden trasladar a un nuevo grupo de recursos o suscripción.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: reference
ms.date: 5/16/2019
ms.author: tomfitz
ms.openlocfilehash: d662607eaaabb8ccfad89f625165c542af81b0fa
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65794516"
---
# <a name="move-operation-support-for-resources"></a>Compatibilidad con la operación de traslado para recursos
En este artículo se indica si un tipo de recurso de Azure es compatible con la operación de traslado. Aunque un tipo de recurso sea compatible con la operación de traslado, puede haber condiciones que impidan que el recurso se traslade. Para obtener información sobre las condiciones que afectan a las operaciones de desplazamiento, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](resource-group-move-resources.md).

Para obtener los mismos datos como un archivo de valores separados por comas, descargue [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).

## <a name="microsoftaad"></a>Microsoft.AAD
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| domainservices | No | No |

## <a name="microsoftaadiam"></a>microsoft.aadiam
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| tenants | No | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| actionrules | Sí | Sí |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| servidores | Sí | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| service | SÍ | Sí |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| configurationstores | Sí | Sí |

## <a name="microsoftappservice"></a>Microsoft.AppService
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| apiapps | No | No |
| appidentities | No | No |
| gateways | No | No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| policyassignments | No | No |

## <a name="microsoftautomation"></a>Microsoft.Automation
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| automationaccounts | Sí | Sí |
| automationaccounts/configurations | Sí | Sí |
| automationaccounts/runbooks | Sí | Sí |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| b2cdirectories | Sí | Sí |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| registrations | Sí | Sí |

## <a name="microsoftbackup"></a>Microsoft.Backup
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| backupvault | No | No |

## <a name="microsoftbatch"></a>Microsoft.Batch
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| batchaccounts | Sí | Sí |

## <a name="microsoftbatchai"></a>Microsoft.BatchAI
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| clusters | No | No |
| fileservers | No | No |
| trabajos | No | No |
| workspaces | No | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| mapapis | No | No |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| biztalk | Sí | Sí |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| blockchainmembers | Sí | Sí |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| blueprintassignments | No | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| botservices | Sí | Sí |

## <a name="microsoftcache"></a>Microsoft.Cache
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| redis | Sí | Sí |

## <a name="microsoftcdn"></a>Microsoft.Cdn
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| perfiles | Sí | Sí |
| profiles/endpoints | Sí | Sí |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| certificateorders | Sí | Sí |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| domainnames | Sí | No |
| virtualmachines | Sí | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| networksecuritygroups | No | No |
| reservedips | No | No |
| virtualnetworks | No | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| storageaccounts | Sí | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftcompute"></a>Microsoft.Compute
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| availabilitysets | Sí | Sí |
| Discos | Sí | Sí |
| galleries | No | No |
| galleries/images | No | No |
| galleries/images/versions | No | No |
| hostgroups | No | No |
| hostgroups/hosts | No | No |
| imágenes | Sí | Sí |
| proximityplacementgroups | No | No |
| restorepointcollections | No | No |
| sharedvmimages | No | No |
| sharedvmimages/versions | No | No |
| instantáneas | Sí | Sí |
| virtualmachines | Sí | Sí |
| virtualmachines/extensions | Sí | Sí |
| virtualmachinescalesets | Sí | Sí |

## <a name="microsoftcontainer"></a>Microsoft.Container
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| containergroups | No | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| containergroups | No | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| registries | Sí | Sí |
| registries/buildtasks | Sí | Sí |
| registries/replications | Sí | Sí |
| registries/tasks | Sí | Sí |
| registries/webhooks | Sí | Sí |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| containerservices | No | No |
| managedclusters | No | No |
| openshiftmanagedclusters | No | No |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| applications | Sí | Sí |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| conectores | Sí | Sí |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| hubs | Sí | Sí |

## <a name="microsoftdatabox"></a>Microsoft.DataBox
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| trabajos | No | No |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| databoxedgedevices | No | No |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| workspaces | No | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| catalogs | Sí | Sí |
| datacatalogs | No | No |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| connectionmanagers | No | No |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| packages | No | No |
| plans | No | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| datafactories | Sí | Sí |
| factories | Sí | Sí |

## <a name="microsoftdatalake"></a>Microsoft.DataLake
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| datalakeaccounts | No | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| services | No | No |
| services/projects | No | No |
| slots | No | No |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| servidores | Sí | Sí |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| servidores | Sí | Sí |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| servergroups | No | No |
| servidores | Sí | Sí |
| serversv2 | Sí | Sí |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| artifactsources | Sí | Sí |
| rollouts | Sí | Sí |
| servicetopologies | Sí | Sí |
| servicetopologies/services | Sí | Sí |
| servicetopologies/services/serviceunits | Sí | Sí |
| steps | Sí | Sí |

## <a name="microsoftdevices"></a>Microsoft.Devices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| elasticpools | No | No |
| elasticpools/iothubtenants | No | No |
| iothubs | Sí | Sí |
| provisioningservices | Sí | Sí |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| controllers | No | No |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| labcenters | No | No |
| labs | Sí | No |
| laboratorios y entornos | Sí | Sí |
| labs/servicerunners | Sí | Sí |
| labs/virtualmachines | Sí | No |
| schedules | Sí | Sí |

## <a name="microsoftdns"></a>microsoft.dns
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| dnszones | No | No |
| dnszones/a | No | No |
| dnszones/aaaa | No | No |
| dnszones/cname | No | No |
| dnszones/mx | No | No |
| dnszones/ptr | No | No |
| dnszones/srv | No | No |
| dnszones/txt | No | No |
| trafficmanagerprofiles | No | No |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| databaseaccounts | Sí | Sí |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| domains | Sí | Sí |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| services | Sí | Sí |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| domains | Sí | Sí |
| topics | Sí | Sí |

## <a name="microsofteventhub"></a>Microsoft.EventHub
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| clusters | Sí | Sí |
| namespaces | Sí | Sí |

## <a name="microsoftgenomics"></a>Microsoft.Genomics
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| hanainstances | Sí | Sí |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| clusters | Sí | Sí |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| services | Sí | Sí |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| máquinas | No | No |

## <a name="microsofthybriddata"></a>Microsoft.HybridData
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| datamanagers | Sí | Sí |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| trabajos | Sí | Sí |

## <a name="microsoftinsights"></a>microsoft.insights
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |
| actiongroups | Sí | Sí |
| activitylogalerts | No | No |
| alertrules | Sí | Sí |
| autoscalesettings | Sí | Sí |
| components | Sí | Sí |
| guestdiagnosticsettings | No | No |
| metricalerts | No | No |
| notificationgroups | No | No |
| notificationrules | No | No |
| scheduledqueryrules | No | No |
| webtests | Sí | Sí |
| libros | Sí | Sí |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| iotapps | Sí | Sí |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| checknameavailability | Sí | Sí |
| grafo | Sí | Sí |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| hsmpools | No | No |
| vaults | Sí | Sí |

## <a name="microsoftkusto"></a>Microsoft.Kusto
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| clusters | Sí | Sí |

## <a name="microsoftlabservices"></a>Microsoft.LabServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| labaccounts | Sí | Sí |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |

## <a name="microsoftlogic"></a>Microsoft.Logic
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| hostingenvironments | No | No |
| integrationaccounts | Sí | Sí |
| integrationserviceenvironments | No | No |
| isolatedenvironments | No | No |
| workflows | Sí | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| commitmentplans | Sí | Sí |
| webservices | Sí | No |
| workspaces | Sí | Sí |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| operationalizationclusters | Sí | Sí |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |
| accounts/workspaces | No | No |
| accounts/workspaces/projects | No | No |
| teamaccounts | No | No |
| teamaccounts/workspaces | No | No |
| teamaccounts/workspaces/projects | No | No |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftmachinelearningoperationalization"></a>Microsoft.MachineLearningOperationalization
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| hostingaccounts | No | No |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| workspaces | No | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| userassignedidentities | No | No |

## <a name="microsoftmaps"></a>Microsoft.Maps
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | Sí | Sí |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| classicdevservices | No | No |

## <a name="microsoftmedia"></a>Microsoft.Media
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| mediaservices | Sí | Sí |
| mediaservices/liveevents | Sí | Sí |
| mediaservices/streamingendpoints | Sí | Sí |

## <a name="microsoftmigrate"></a>Microsoft.Migrate
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| assessmentprojects | No | No |
| migrateprojects | No | No |
| projects | No | No |

## <a name="microsoftnetapp"></a>Microsoft.NetApp
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| netappaccounts | No | No |
| netappaccounts/capacitypools | No | No |
| netappaccounts/capacitypools/volumes | No | No |
| netappaccounts/capacitypools/volumes/mounttargets | No | No |
| netappaccounts/capacitypools/volumes/snapshots | No | No |

## <a name="microsoftnetwork"></a>Microsoft.Network
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| applicationgateways | No | No |
| applicationgatewaywebapplicationfirewallpolicies | No | No |
| applicationsecuritygroups | Sí | Sí |
| azurefirewalls | Sí | Sí |
| bastionhosts | No | No |
| conexiones | Sí | Sí |
| ddoscustompolicies | Sí | Sí |
| ddosprotectionplans | No | No |
| dnszones | Sí | Sí |
| expressroutecircuits | No | No |
| expressroutecrossconnections | No | No |
| expressroutegateways | No | No |
| expressrouteports | No | No |
| puertas principales | Sí | Sí |
| frontdoorwebapplicationfirewallpolicies | Sí | Sí |
| loadbalancers | Sí | Sí |
| localnetworkgateways | Sí | Sí |
| natgateways | Sí | Sí |
| networkintentpolicies | Sí | Sí |
| networkinterfaces | Sí | Sí |
| networkprofiles | No | No |
| networksecuritygroups | Sí | Sí |
| networkwatchers | Sí | Sí |
| networkwatchers/connectionmonitors | Sí | Sí |
| networkwatchers/lenses | Sí | Sí |
| networkwatchers/pingmeshes | Sí | Sí |
| p2svpngateways | No | No |
| privatednszones | Sí | Sí |
| privatednszones/virtualnetworklinks | Sí | Sí |
| privateendpoints | No | No |
| privatelinkservices | No | No |
| publicipaddresses | Sí | Sí |
| publicipprefixes | Sí | Sí |
| routefilters | No | No |
| routetables | Sí | Sí |
| securegateways | Sí | Sí |
| serviceendpointpolicies | Sí | Sí |
| trafficmanagerprofiles | Sí | Sí |
| virtualhubs | No | No |
| virtualnetworkgateways | Sí | Sí |
| virtualnetworks | Sí | Sí |
| virtualnetworktaps | No | No |
| virtualwans | No | No |
| vpngateways | No | No |
| vpnsites | No | No |
| webapplicationfirewallpolicies | Sí | Sí |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| namespaces | Sí | Sí |
| namespaces/notificationhubs | Sí | Sí |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| workspaces | Sí | Sí |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| managementconfigurations | Sí | Sí |
| solutions | Sí | SÍ |
| views | SÍ | Sí |

## <a name="microsoftpeering"></a>Microsoft.Peering
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| emparejamientos | No | No |

## <a name="microsoftportal"></a>Microsoft.Portal
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| dashboards | Sí | Sí |

## <a name="microsoftportalsdk"></a>Microsoft.PortalSdk
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| rootresources | No | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| workspacecollections | Sí | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| capacities | Sí | Sí |

## <a name="microsoftprojectoxford"></a>Microsoft.ProjectOxford
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| cuentas | No | No |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| vaults | Sí | Sí |

## <a name="microsoftrelay"></a>Microsoft.Relay
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| namespaces | Sí | Sí |

## <a name="microsoftsaas"></a>Microsoft.SaaS
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| applications | Sí | No |

## <a name="microsoftscheduler"></a>Microsoft.Scheduler
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| flujos | Sí | Sí |
| jobcollections | Sí | Sí |

## <a name="microsoftsearch"></a>Microsoft.Search
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| searchservices | Sí | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| iotsecuritysolutions | Sí | Sí |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| gateways | No | No |
| nodes | No | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| namespaces | Sí | Sí |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| aplicaciones | No | No |
| clusters | Sí | Sí |
| containergroups | No | No |
| containergroupsets | No | No |
| edgeclusters | No | No |
| Redes | No | No |
| secretstores | No | No |
| volúmenes | No | No |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| applications | Sí | Sí |
| containergroups | No | No |
| gateways | Sí | Sí |
| Redes | Sí | Sí |
| secretos | Sí | Sí |
| volúmenes | Sí | Sí |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| signalr | Sí | Sí |

## <a name="microsoftsiterecovery"></a>Microsoft.SiteRecovery
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| siterecoveryvault | No | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| appliancedefinitions | No | No |
| appliances | No | No |
| applicationdefinitions | No | No |
| applications | No | No |
| jitrequests | No | No |

## <a name="microsoftsql"></a>Microsoft.Sql
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| instancepools | Sí | Sí |
| managedinstances | Sí | Sí |
| managedinstances/databases | Sí | Sí |
| servidores | Sí | Sí |
| servers/databases | Sí | Sí |
| servers/elasticpools | Sí | Sí |
| virtualclusters | Sí | Sí |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| sqlvirtualmachinegroups | Sí | Sí |
| sqlvirtualmachines | Sí | Sí |

## <a name="microsoftsqlvm"></a>Microsoft.SqlVM
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| dwvm | No | No |

## <a name="microsoftstorage"></a>Microsoft.Storage
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| storageaccounts | Sí | Sí |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| Memorias caché | No | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| storagesyncservices | Sí | Sí |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| storagesyncservices | No | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| storagesyncservices | No | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| managers | No | No |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| streamingjobs | Sí | Sí |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| environments | No | No |
| environments/eventsources | No | No |
| instancias | No | No |
| instances/environments | No | No |
| instances/environments/eventsources | No | No |

## <a name="microsoftterraformoss"></a>Microsoft.TerraformOSS
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| providerregistrations | No | No |
| recursos | No | No |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| environments | Sí | Sí |
| environments/eventsources | Sí | Sí |
| environments/referencedatasets | Sí | Sí |

## <a name="microsofttoken"></a>Microsoft.Token
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| almacenes | No | No |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| imagetemplates | No | No |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| account | SÍ | Sí |
| account/extension | Sí | Sí |
| account/project | Sí | Sí |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| dedicatedcloudnodes | Sí | Sí |
| dedicatedcloudservices | Sí | Sí |
| virtualmachines | Sí | Sí |

## <a name="microsoftweb"></a>Microsoft.Web
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| certificados | No | Sí |
| connectiongateways | Sí | Sí |
| conexiones | Sí | Sí |
| customapis | Sí | Sí |
| hostingenvironments | No | No |
| serverfarms | Sí | Sí |
| sitios | Sí | Sí |
| sites/premieraddons | Sí | Sí |
| sites/slots | Sí | Sí |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| deviceservices | Sí | Sí |

## <a name="microsoftwindowsvirtualdesktop"></a>Microsoft.WindowsVirtualDesktop
| Tipo de recurso | Grupos de recursos | Subscription |
| ------------- | ----------- | ---------- |
| applicationgroups | No | No |
| hostpools | No | No |
| workspaces | No | No |

## <a name="third-party-services"></a>Servicios de terceros

Actualmente los servicios de terceros no son compatibles con la operación de traslado.

## <a name="next-steps"></a>Pasos siguientes
Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](resource-group-move-resources.md).
