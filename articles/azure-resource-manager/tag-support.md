---
title: Compatibilidad de etiquetas de Azure Resource Manager para los recursos
description: Muestra los tipos de recursos de Azure que admiten etiquetas. Proporciona detalles de todos los servicios de Azure.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: reference
ms.date: 05/10/2019
ms.author: tomfitz
ms.openlocfilehash: 7ef37323fb8150e3a6b52800bfafa2585ae328c2
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523235"
---
# <a name="tag-support-for-azure-resources"></a>Compatibilidad de etiquetas de los recursos de Azure
En este artículo se describe si un tipo de recurso admite [etiquetas](resource-group-using-tags.md). La columna con la etiqueta **admite etiquetas** indica si el tipo de recurso tiene una propiedad de la etiqueta. La columna con la etiqueta **etiqueta en el informe de costo** indica si ese tipo de recurso pasa a la etiqueta para el informe de costo.

Para obtener los mismos datos como un archivo de valores separados por comas, descargue [tag-support.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/tag-support.csv).

## <a name="microsoftaad"></a>Microsoft.AAD
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| DomainServices | Sí | Sí |
| DomainServices/oucontainer | No | No |

## <a name="microsoftaadiam"></a>microsoft.aadiam
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| diagnosticSettings | No |  No |
| diagnosticSettingsCategories | No |  No |

## <a name="microsoftaddons"></a>Microsoft.Addons
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| supportProviders | No |  No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| aadsupportcases | No |  No |
| addsservices | No |  No |
| agents | No |  No |
| anonymousapiusers | No |  No |
| configuración | No |  No |
| registros | No |  No |
| Informes | No |  No |
| services | No |  No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| configuraciones | No |  No |
| generateRecommendations | No |  No |
| recomendaciones | No |  No |
| suppressions | No |  No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| actionRules | No |  No |
| alertas | No |  No |
| alertsList | No |  No |
| alertsSummary | No |  No |
| alertsSummaryList | No |  No |
| smartDetectorAlertRules | No |  No |
| smartDetectorRuntimeEnvironments | No |  No |
| smartGroups | No |  No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| servidores | Sí | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| reportFeedback | No |  No |
| servicio | SÍ | Sí |
| validateServiceName | No |  No |

## <a name="microsoftattestation"></a>Microsoft.Attestation
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| attestationProviders | No |  No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| classicAdministrators | No |  No |
| denyAssignments | No |  No |
| elevateAccess | No |  No |
| locks | No |  No |
| permisos | No |  No |
| policyAssignments | No |  No |
| policyDefinitions | No |  No |
| policySetDefinitions | No |  No |
| providerOperations | No |  No |
| roleAssignments | No |  No |
| roleDefinitions | No |  No |

## <a name="microsoftautomation"></a>Microsoft.Automation
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| automationAccounts | Sí | Sí |
| automationAccounts/configurations | Sí | Sí |
| automationAccounts/jobs | No |  No |
| automationAccounts/runbooks | Sí | Sí |
| automationAccounts/softwareUpdateConfigurations | No | No |
| automationAccounts/webhooks | No |  No |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| environments | No |  No |
| environments/accounts | No |  No |
| environments/accounts/namespaces | No |  No |
| environments/accounts/namespaces/configurations | No |  No |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| b2cDirectories | Sí | No |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| registrations | Sí | Sí |
| registrations/customerSubscriptions | No |  No |
| registrations/products | No |  No |

## <a name="microsoftbatch"></a>Microsoft.Batch
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| batchAccounts | Sí | Sí |

## <a name="microsoftbilling"></a>Microsoft.Billing
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| billingAccounts | No |  No |
| billingAccounts/billingProfiles | No |  No |
| billingAccounts/billingProfiles/billingSubscriptions | No |  No |
| billingAccounts/billingProfiles/invoices | No |  No |
| billingAccounts/billingProfiles/invoices/pricesheet | No |  No |
| billingAccounts/billingProfiles/operationStatus | No |  No |
| billingAccounts/billingProfiles/paymentMethods | No |  No |
| billingAccounts/billingProfiles/policies | No |  No |
| billingAccounts/billingProfiles/pricesheet | No |  No |
| billingAccounts/billingProfiles/products | No |  No |
| billingAccounts/billingProfiles/transactions | No |  No |
| billingAccounts/billingSubscriptions | No |  No |
| billingAccounts/departments | No |  No |
| billingAccounts/eligibleOffers | No |  No |
| billingAccounts/enrollmentAccounts | No |  No |
| billingAccounts/invoices | No |  No |
| billingAccounts/invoiceSections | No |  No |
| billingAccounts/invoiceSections/billingSubscriptions | No |  No |
| billingAccounts/invoiceSections/billingSubscriptions/transfer | No |  No |
| billingAccounts/invoiceSections/importRequests | No |  No |
| billingAccounts/invoiceSections/initiateImportRequest | No |  No |
| billingAccounts/invoiceSections/initiateTransfer | No |  No |
| billingAccounts/invoiceSections/operationStatus | No |  No |
| billingAccounts/invoiceSections/products | No |  No |
| billingAccounts/invoiceSections/transfers | No |  No |
| billingAccounts/products | No |  No |
| billingAccounts/projects | No |  No |
| billingAccounts/projects/billingSubscriptions | No |  No |
| billingAccounts/projects/importRequests | No |  No |
| billingAccounts/projects/initiateImportRequest | No |  No |
| billingAccounts/projects/operationStatus | No |  No |
| billingAccounts/projects/products | No |  No |
| billingAccounts/transactions | No |  No |
| billingPeriods | No |  No |
| BillingPermissions | No |  No |
| billingProperty | No |  No |
| BillingRoleAssignments | No |  No |
| BillingRoleDefinitions | No |  No |
| CreateBillingRoleAssignment | No |  No |
| departments | No |  No |
| enrollmentAccounts | No |  No |
| importRequests | No |  No |
| importRequests/acceptImportRequest | No |  No |
| importRequests/declineImportRequest | No |  No |
| invoices | No |  No |
| transfers | No |  No |
| transfers/acceptTransfer | No |  No |
| transfers/declineTransfer | No |  No |
| transfers/operationStatus | No |  No |
| usagePlans | No |  No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| mapApis | Sí | Sí |
| updateCommunicationPreference | No |  No |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| BizTalk | Sí | Sí |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| blueprintAssignments | No |  No |
| blueprintAssignments/assignmentOperations | No |  No |
| blueprintAssignments/operations | No |  No |
| blueprints | No |  No |
| blueprints/artifacts | No |  No |
| blueprints/versions | No |  No |
| blueprints/versions/artifacts | No |  No |

## <a name="microsoftbotservice"></a>Microsoft.BotService
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| botServices | Sí | Sí |
| botServices/channels | No |  No |
| botServices/connections | No |  No |

## <a name="microsoftcache"></a>Microsoft.Cache
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| Redis | Sí | Sí |
| RedisConfigDefinition | No |  No |

## <a name="microsoftcapacity"></a>Microsoft.Capacity
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| appliedReservations | No |  No |
| calculatePrice | No |  No |
| catalogs | No |  No |
| commercialReservationOrders | No |  No |
| reservationOrders | No |  No |
| reservationOrders/calculateRefund | No |  No |
| reservationOrders/merge | No |  No |
| reservationOrders/reservations | No |  No |
| reservationOrders/reservations/revisions | No |  No |
| reservationOrders/return | No |  No |
| reservationOrders/split | No |  No |
| reservationOrders/swap | No |  No |
| reservations | No |  No |
| recursos | No |  No |
| validateReservationOrder | No |  No |

## <a name="microsoftcdn"></a>Microsoft.Cdn
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| edgenodes | No |  No |
| perfiles | Sí | Sí |
| profiles/endpoints | Sí | Sí |
| profiles/endpoints/customdomains | No |  No |
| profiles/endpoints/origins | No |  No |
| validateProbe | No |  No |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| certificateOrders | Sí | Sí |
| certificateOrders/certificates | No |  No |
| validateCertificateRegistrationInformation | No |  No |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| capabilities | No |  No |
| domainNames | No |  No |
| domainNames/capabilities | No |  No |
| domainNames/internalLoadBalancers | No |  No |
| domainNames/serviceCertificates | No |  No |
| domainNames/slots | No |  No |
| domainNames/slots/roles | No |  No |
| moveSubscriptionResources | No |  No |
| operatingSystemFamilies | No |  No |
| operatingSystems | No |  No |
| quotas | No |  No |
| resourceTypes | No |  No |
| validateSubscriptionMoveAvailability | No |  No |
| virtualMachines | No |  No |
| virtualMachines/diagnosticSettings | No |  No |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| classicInfrastructureResources | No |  No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| capabilities | No |  No |
| expressRouteCrossConnections | No |  No |
| expressRouteCrossConnections/peerings | No |  No |
| gatewaySupportedDevices | No |  No |
| networkSecurityGroups | No |  No |
| quotas | No |  No |
| reservedIps | No |  No |
| virtualNetworks | No |  No |
| virtualNetworks/remoteVirtualNetworkPeeringProxies | No |  No |
| virtualNetworks/virtualNetworkPeerings | No |  No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| capabilities | No |  No |
| Discos | No |  No |
| imágenes | No |  No |
| osImages | No |  No |
| osPlatformImages | No |  No |
| publicImages | No |  No |
| quotas | No |  No |
| storageAccounts | No |  No |
| storageAccounts/services | No |  No |
| storageAccounts/services/diagnosticSettings | No |  No |
| storageAccounts/vmImages | No |  No |
| vmImages | No |  No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftcommerce"></a>Microsoft.Commerce
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| RateCard | No |  No |
| UsageAggregates | No |  No |

## <a name="microsoftcompute"></a>Microsoft.Compute
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| availabilitySets | Sí | Sí |
| Discos | Sí | Sí |
| imágenes | Sí | Sí |
| restorePointCollections | Sí | Sí |
| restorePointCollections/restorePoints | No |  No |
| sharedVMImages | Sí | Sí |
| sharedVMImages/versions | Sí | Sí |
| instantáneas | Sí | Sí |
| virtualMachines | Sí | Sí |
| virtualMachines/diagnosticSettings | No |  No |
| virtualMachines/extensions | Sí | Sí |
| virtualMachineScaleSets | Sí | Sí |
| virtualMachineScaleSets/extensions | No |  No |
| virtualMachineScaleSets/networkInterfaces | No |  No |
| virtualMachineScaleSets/publicIPAddresses | No |  No |
| virtualMachineScaleSets/virtualMachines | No |  No |
| virtualMachineScaleSets/virtualMachines/networkInterfaces | No |  No |

## <a name="microsoftconsumption"></a>Microsoft.Consumption
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| AggregatedCost | No |  No |
| Saldos | No |  No |
| Presupuestos | No |  No |
| Charges | No |  No |
| CostTags | No |  No |
| credits | No |  No |
| events | No |  No |
| Previsiones | No |  No |
| lots | No |  No |
| Catálogos de soluciones | No |  No |
| Pricesheets | No |  No |
| products | No |  No |
| ReservationDetails | No |  No |
| ReservationRecommendations | No |  No |
| ReservationSummaries | No |  No |
| ReservationTransactions | No |  No |
| `Tags` | No |  No |
| Términos | No |  No |
| UsageDetails | No |  No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| containerGroups | Sí | Sí |
| serviceAssociationLinks | No |  No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| registries | Sí | Sí |
| registries/builds | No |  No |
| registries/builds/cancel | No |  No |
| registries/builds/getLogLink | No |  No |
| registries/buildTasks | Sí | Sí |
| registries/buildTasks/steps | No |  No |
| registries/eventGridFilters | No |  No |
| registries/getBuildSourceUploadUrl | No |  No |
| registries/GetCredentials | No |  No |
| registries/importImage | No |  No |
| registries/queueBuild | No |  No |
| registries/regenerateCredential | No |  No |
| registries/regenerateCredentials | No |  No |
| registries/replications | Sí | Sí |
| registries/runs | No |  No |
| registries/runs/cancel | No |  No |
| registries/scheduleRun | No |  No |
| registries/tasks | Sí | Sí |
| registries/updatePolicies | No |  No |
| registries/webhooks | Sí | Sí |
| registries/webhooks/getCallbackConfig | No |  No |
| registries/webhooks/ping | No |  No |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| containerServices | Sí | Sí |
| managedClusters | Sí | Sí |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| aplicaciones | Sí | Sí |
| updateCommunicationPreference | No |  No |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| Alertas | No |  No |
| BillingAccounts | No |  No |
| Conectores | Sí | Sí |
| Departments | No |  No |
| Dimensiones | No |  No |
| EnrollmentAccounts | No |  No |
| Consultar | No |  No |
| registro | No |  No |
| Reportconfigs | No |  No |
| Informes | No |  No |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| hubs | Sí | Sí |
| hubs/authorizationPolicies | No |  No |
| hubs/connectors | No |  No |
| hubs/connectors/mappings | No |  No |
| hubs/interactions | No |  No |
| hubs/kpi | No |  No |
| hubs/links | No |  No |
| hubs/profiles | No |  No |
| hubs/roleAssignments | No |  No |
| hubs/roles | No |  No |
| hubs/suggestTypeSchema | No |  No |
| hubs/views | No |  No |
| hubs/widgetTypes | No |  No |

## <a name="microsoftdatabox"></a>Microsoft.DataBox
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| trabajos | Sí | Sí |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| DataBoxEdgeDevices | Sí | Sí |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| workspaces | Sí | No |
| workspaces/virtualNetworkPeerings | No |  No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| catalogs | Sí | Sí |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| connectionManagers | Sí | Sí |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| dataFactories | Sí | No |
| dataFactories/diagnosticSettings | No |  No |
| dataFactorySchema | No |  No |
| factories | Sí | No |
| factories/integrationRuntimes | No |  No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |
| accounts/dataLakeStoreAccounts | No |  No |
| accounts/storageAccounts | No |  No |
| accounts/storageAccounts/containers | No |  No |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |
| accounts/eventGridFilters | No |  No |
| accounts/firewallRules | No |  No |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| services | Sí | Sí |
| services/projects | Sí | Sí |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| servidores | Sí | Sí |
| servers/recoverableServers | No |  No |
| servers/virtualNetworkRules | No |  No |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| servidores | Sí | Sí |
| servers/recoverableServers | No |  No |
| servers/virtualNetworkRules | No |  No |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| servidores | Sí | Sí |
| servers/advisors | No |  No |
| servers/queryTexts | No |  No |
| servers/recoverableServers | No |  No |
| servers/topQueryStatistics | No |  No |
| servers/virtualNetworkRules | No |  No |
| servers/waitStatistics | No |  No |

## <a name="microsoftdevices"></a>Microsoft.Devices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| IotHubs | Sí | Sí |
| IotHubs/eventGridFilters | No |  No |
| ProvisioningServices | Sí | Sí |
| usages | No |  No |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| controllers | Sí | Sí |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| labs | Sí | Sí |
| labs/serviceRunners | Sí | Sí |
| labs/virtualMachines | Sí | Sí |
| schedules | Sí | Sí |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| databaseAccountNames | No |  No |
| databaseAccounts | Sí | Sí |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| domains | Sí | Sí |
| domains/domainOwnershipIdentifiers | No |  No |
| generateSsoRequest | No |  No |
| topLevelDomains | No |  No |
| validateDomainRegistrationInformation | No |  No |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| lcsprojects | No |  No |
| lcsprojects/clouddeployments | No |  No |
| lcsprojects/connectors | No |  No |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| domains | Sí | No |
| domains/topics | No |  No |
| eventSubscriptions | No |  No |
| extensionTopics | No |  No |
| topics | Sí | No |
| topicTypes | No |  No |

## <a name="microsofteventhub"></a>Microsoft.EventHub
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| clusters | Sí | No |
| namespaces | Sí | No |
| namespaces/authorizationrules | No |  No |
| namespaces/disasterrecoveryconfigs | No |  No |
| namespaces/eventhubs | No |  No |
| namespaces/eventhubs/authorizationrules | No |  No |
| namespaces/eventhubs/consumergroups | No |  No |

## <a name="microsoftfeatures"></a>Microsoft.Features
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| features | No |  No |
| providers | No |  No |

## <a name="microsoftgallery"></a>Microsoft.Gallery
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| enroll | No |  No |
| galleryitems | No |  No |
| generateartifactaccessuri | No |  No |
| myareas | No |  No |
| myareas/areas | No |  No |
| myareas/areas/areas | No |  No |
| myareas/areas/areas/galleryitems | No |  No |
| myareas/areas/galleryitems | No |  No |
| myareas/galleryitems | No |  No |
| registro | No |  No |
| recursos | No |  No |
| retrieveresourcesbyid | No |  No |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| guestConfigurationAssignments | No |  No |
| software | No |  No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| hanaInstances | Sí |  Sí |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| clusters | Sí | Sí |
| clusters/applications | No |  No |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| trabajos | Sí | Sí |

## <a name="microsoftinformationprotection"></a>Microsoft.InformationProtection
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| labelGroups | No |  No |
| labelGroups/labels | No |  No |
| labelGroups/labels/conditions | No |  No |
| labelGroups/labels/subLabels | No |  No |
| labelGroups/labels/subLabels/conditions | No |  No |

## <a name="microsoftinsights"></a>microsoft.insights
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| actiongroups | Sí | Sí |
| activityLogAlerts | Sí | Sí |
| alertrules | Sí | Sí |
| automatedExportSettings | No |  No |
| autoscalesettings | Sí | Sí |
| baseline | No |  No |
| calculatebaseline | No |  No |
| components | Sí | Sí |
| components/events | No |  No |
| components/pricingPlans | No |  No |
| components/query | No |  No |
| diagnosticSettings | No |  No |
| diagnosticSettingsCategories | No |  No |
| eventCategories | No |  No |
| eventtypes | No |  No |
| extendedDiagnosticSettings | No |  No |
| logDefinitions | No |  No |
| logprofiles | No |  No |
| registros | No |  No |
| metricAlerts | Sí | Sí |
| migrateToNewPricingModel | No |  No |
| myWorkbooks | No |  No |
| Consultas | No |  No |
| rollbackToLegacyPricingModel | No |  No |
| scheduledqueryrules | Sí | Sí |
| vmInsightsOnboardingStatuses | No |  No |
| webtests | Sí | Sí |
| libros | Sí | Sí |

## <a name="microsoftintune"></a>Microsoft.Intune
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| diagnosticSettings | No |  No |
| diagnosticSettingsCategories | No |  No |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| IoTApps | Sí | Sí |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| Gráfico | Sí | Sí |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| deletedVaults | No |  No |
| vaults | Sí | Sí |
| vaults/accessPolicies | No |  No |
| vaults/secrets | No |  No |

## <a name="microsoftkusto"></a>Microsoft.Kusto
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| clusters | Sí | Sí |
| clusters/databases | No |  No |
| clusters/databases/dataconnections | No |  No |
| clusters/databases/eventhubconnections | No |  No |

## <a name="microsoftlabservices"></a>Microsoft.LabServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| labaccounts | Sí | Sí |
| Usuarios | No |  No |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftloganalytics"></a>Microsoft.LogAnalytics
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| registros | No |  No |

## <a name="microsoftlogic"></a>Microsoft.Logic
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| integrationAccounts | Sí | Sí |
| workflows | Sí | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| commitmentPlans | Sí | Sí |
| webServices | Sí | Sí |
| Áreas de trabajo | Sí | Sí |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |
| accounts/workspaces | Sí | Sí |
| accounts/workspaces/projects | Sí | Sí |
| teamAccounts | Sí | Sí |
| teamAccounts/workspaces | Sí | Sí |
| teamAccounts/workspaces/projects | Sí | Sí |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| workspaces | Sí | Sí |
| workspaces/computes | No |  No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| Identidades | No |  No |
| userAssignedIdentities | Sí | 

## <a name="microsoftmanagement"></a>Microsoft.Management
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| getEntities | No |  No |
| managementGroups | No |  No |
| recursos | No |  No |
| startTenantBackfill | No |  No |
| tenantBackfillStatus | No |  No |

## <a name="microsoftmaps"></a>Microsoft.Maps
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |
| accounts/eventGridFilters | No |  No |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| offers | No |  No |
| offerTypes | No |  No |
| offerTypes/publishers | No |  No |
| offerTypes/publishers/offers | No |  No |
| offerTypes/publishers/offers/plans | No |  No |
| offerTypes/publishers/offers/plans/agreements | No |  No |
| offerTypes/publishers/offers/plans/configs | No |  No |
| offerTypes/publishers/offers/plans/configs/importImage | No |  No |
| privategalleryitems | No |  No |
| products | No |  No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| classicDevServices | Sí | Sí |
| updateCommunicationPreference | No |  No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| agreements | No |  No |
| offertypes | No |  No |

## <a name="microsoftmedia"></a>Microsoft.Media
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| mediaservices | Sí | Sí |
| mediaservices/accountFilters | No |  No |
| mediaservices/assets | No |  No |
| mediaservices/assets/assetFilters | No |  No |
| mediaservices/contentKeyPolicies | No |  No |
| mediaservices/eventGridFilters | No |  No |
| mediaservices/liveEventOperations | No |  No |
| mediaservices/liveEvents | Sí | Sí |
| mediaservices/liveEvents/liveOutputs | No |  No |
| mediaservices/liveOutputOperations | No |  No |
| mediaservices/streamingEndpointOperations | No |  No |
| mediaservices/streamingEndpoints | Sí | Sí |
| mediaservices/streamingLocators | No |  No |
| mediaservices/streamingPolicies | No |  No |
| mediaservices/transforms | No |  No |
| mediaservices/transforms/jobs | No |  No |

## <a name="microsoftmigrate"></a>Microsoft.Migrate
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| projects | Sí | Sí |

## <a name="microsoftnetwork"></a>Microsoft.Network
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| applicationGateways | Sí | No |
| applicationSecurityGroups | Sí | Sí |
| azureFirewallFqdnTags | No |  No |
| azureFirewalls | Sí | No |
| bgpServiceCommunities | No |  No |
| conexiones | Sí | Sí |
| ddosCustomPolicies | Sí | Sí |
| ddosProtectionPlans | Sí | Sí |
| dnsOperationStatuses | No |  No |
| dnszones | Sí | Sí |
| dnszones/A | No |  No |
| dnszones/AAAA | No |  No |
| dnszones/all | No |  No |
| dnszones/CAA | No |  No |
| dnszones/CNAME | No |  No |
| dnszones/MX | No |  No |
| dnszones/NS | No |  No |
| dnszones/PTR | No |  No |
| dnszones/recordsets | No |  No |
| dnszones/SOA | No |  No |
| dnszones/SRV | No |  No |
| dnszones/TXT | No |  No |
| expressRouteCircuits | Sí  | No |
| expressRouteServiceProviders | No |  No |
| puertas principales | Sí | Sí |
| frontdoorWebApplicationFirewallPolicies | Sí | Sí |
| getDnsResourceReference | No |  No |
| interfaceEndpoints | Sí | Sí |
| internalNotify | No |  No |
| loadBalancers | Sí | No |
| localNetworkGateways | Sí | Sí |
| natGateways | Sí | Sí |
| networkIntentPolicies | Sí | Sí |
| networkInterfaces | Sí | Sí |
| networkProfiles | Sí | Sí |
| networkSecurityGroups | Sí | Sí |
| networkWatchers | Sí | No |
| networkWatchers/connectionMonitors | Sí | No |
| networkWatchers/lenses | Sí | No |
| networkWatchers/pingMeshes | Sí | No |
| privateLinkServices | Sí | Sí |
| publicIPAddresses | Sí | Sí |
| publicIPPrefixes | Sí | Sí |
| routeFilters | Sí | Sí |
| routeTables | Sí | Sí |
| serviceEndpointPolicies | Sí | Sí |
| trafficManagerGeographicHierarchies | No |  No |
| trafficmanagerprofiles | Sí | Sí |
| trafficmanagerprofiles/heatMaps | No |  No |
| virtualHubs | Sí | Sí |
| virtualNetworkGateways | Sí | No |
| virtualNetworks | Sí | Sí |
| virtualNetworks/subnets | No |  No |
| virtualNetworkTaps | Sí | Sí |
| virtualWans | Sí | Sí |
| vpnGateways | Sí | No |
| vpnSites | Sí | Sí |
| webApplicationFirewallPolicies | Sí | Sí |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| namespaces | Sí | No |
| namespaces/notificationHubs | Sí | No |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| dispositivos | No |  No |
| linkTargets | No |  No |
| storageInsightConfigs | No |  No |
| workspaces | Sí | Sí |
| workspaces/dataSources | No |  No |
| workspaces/linkedServices | No |  No |
| workspaces/query | No |  No |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| managementassociations | No |  No |
| managementconfigurations | Sí | Sí |
| solutions | Sí | SÍ |
| views | SÍ | Sí |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| policyEvents | No |  No |
| policyStates | No |  No |
| policyTrackedResources | No |  No |
| remediations | No |  No |

## <a name="microsoftportal"></a>Microsoft.Portal
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| consoles | No |  No |
| dashboards | Sí | Sí |
| userSettings | No |  No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| workspaceCollections | Sí | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| capacities | Sí | Sí |

## <a name="microsoftprojectoxford"></a>Microsoft.ProjectOxford
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuentas | Sí | Sí |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| backupProtectedItems | No |  No |
| vaults | Sí | Sí |

## <a name="microsoftrelay"></a>Microsoft.Relay
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| namespaces | Sí | Sí |
| namespaces/authorizationrules | No |  No |
| namespaces/hybridconnections | No |  No |
| namespaces/hybridconnections/authorizationrules | No |  No |
| namespaces/wcfrelays | No |  No |
| namespaces/wcfrelays/authorizationrules | No |  No |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| recursos | No |  No |
| subscriptionsStatus | No |  No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| availabilityStatuses | No |  No |
| childAvailabilityStatuses | No |  No |
| childResources | No |  No |
| events | No |  No |
| impactedResources | No |  No |
| notificaciones | No |  No |

## <a name="microsoftresources"></a>Microsoft.Resources
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| implementaciones | No |  No |
| deployments/operations | No |  No |
| links | No |  No |
| notifyResourceJobs | No |  No |
| providers | No |  No |
| resourceGroups | No |  No |
| recursos | No |  No |
| suscripciones | No |  No |
| subscriptions/providers | No |  No |
| subscriptions/resourceGroups | No |  No |
| subscriptions/resourcegroups/resources | No |  No |
| subscriptions/resources | No |  No |
| subscriptions/tagnames | No |  No |
| subscriptions/tagNames/tagValues | No |  No |
| tenants | No |  No |

## <a name="microsoftsaas"></a>Microsoft.SaaS
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| aplicaciones | Sí | Sí |
| saasresources | No |  No |

## <a name="microsoftscheduler"></a>Microsoft.Scheduler
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| flujos | Sí | Sí |
| jobcollections | Sí | Sí |

## <a name="microsoftsearch"></a>Microsoft.Search
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| resourceHealthMetadata | No |  No |
| searchServices | Sí | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| advancedThreatProtectionSettings | No |  No |
| alertas | No |  No |
| allowedConnections | No |  No |
| appliances | No |  No |
| applicationWhitelistings | No |  No |
| AutoProvisioningSettings | No |  No |
| Compliances | No |  No |
| dataCollectionAgents | No |  No |
| discoveredSecuritySolutions | No |  No |
| externalSecuritySolutions | No |  No |
| InformationProtectionPolicies | No |  No |
| jitNetworkAccessPolicies | No |  No |
| monitoring | No |  No |
| monitoring/antimalware | No |  No |
| monitoring/baseline | No |  No |
| monitoring/patch | No |  No |
| directivas | No |  No |
| pricings | No |  No |
| securityContacts | No |  No |
| securitySolutions | No |  No |
| securitySolutionsReferenceData | No |  No |
| securityStatus | No |  No |
| securityStatus/endpoints | No |  No |
| securityStatus/subnets | No |  No |
| securityStatus/virtualMachines | No |  No |
| securityStatuses | No |  No |
| securityStatusesSummaries | No |  No |
| configuración | No |  No |
| tareas | No |  No |
| topologies | No |  No |
| workspaceSettings | No |  No |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| diagnosticSettings | No |  No |
| diagnosticSettingsCategories | No |  No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| namespaces | Sí | No |
| namespaces/authorizationrules | No |  No |
| namespaces/disasterrecoveryconfigs | No |  No |
| namespaces/eventgridfilters | No |  No |
| namespaces/queues | No |  No |
| namespaces/queues/authorizationrules | No |  No |
| namespaces/topics | No |  No |
| namespaces/topics/authorizationrules | No |  No |
| namespaces/topics/subscriptions | No |  No |
| namespaces/topics/subscriptions/rules | No |  No |
| premiumMessagingRegions | No |  No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| clusters | Sí | Sí |
| clusters/applications | No |  No |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| aplicaciones | Sí | Sí |
| gateways | Sí | Sí |
| Redes | Sí | Sí |
| secretos | Sí | Sí |
| volúmenes | Sí | Sí |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| SignalR | Sí | Sí |

## <a name="microsoftsolutions"></a>Microsoft.Solutions
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| applianceDefinitions | Sí | Sí |
| appliances | Sí | Sí |
| applicationDefinitions | Sí | Sí |
| aplicaciones | Sí | Sí |
| jitRequests | Sí | Sí |

## <a name="microsoftsql"></a>Microsoft.SQL
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| managedInstances | Sí | Sí |
| managedInstances/databases | Sí (vea la nota siguiente) | Sí |
| managedInstances/databases/backupShortTermRetentionPolicies | No | No |
| managedInstances/databases/schemas/tables/columns/sensitivityLabels | No | No |
| managedInstances/databases/vulnerabilityAssessments | No | No |
| managedInstances/databases/vulnerabilityAssessments/rules/baselines | No | No |
| managedInstances/encryptionProtector | No | No |
| managedInstances/keys | No | No |
| managedInstances/restorableDroppedDatabases/backupShortTermRetentionPolicies | No | No |
| managedInstances/vulnerabilityAssessments | No | No |
| servidores | Sí | Sí |
| servers/administrators | No |  No |
| servers/communicationLinks | No |  No |
| servers/databases | Sí (vea la nota siguiente) | Sí |
| servers/encryptionProtector | No |  No |
| servers/firewallRules | No |  No |
| servers/keys | No |  No |
| servers/restorableDroppedDatabases | No |  No |
| servers/serviceobjectives | No |  No |
| servers/tdeCertificates | No |  No |

> [!NOTE]
> La base de datos maestra no admite etiquetas, pero otras bases de datos, como las de Azure SQL Data Warehouse, sí lo hacen. Las bases de datos de Azure SQL Data Warehouse deben estar en estado activo (no en pausa).


## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| SqlVirtualMachineGroups | Sí | Sí |
| SqlVirtualMachineGroups/AvailabilityGroupListeners | No |  No |
| SqlVirtualMachines | Sí | Sí |

## <a name="microsoftstorage"></a>Microsoft.Storage
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| storageAccounts | Sí | Sí |
| storageAccounts/blobServices | No |  No |
| storageAccounts/fileServices | No |  No |
| storageAccounts/queueServices | No |  No |
| storageAccounts/services | No |  No |
| storageAccounts/tableServices | No |  No |
| usages | No |  No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| storageSyncServices | Sí | Sí |
| storageSyncServices/registeredServers | No |  No |
| storageSyncServices/syncGroups | No |  No |
| storageSyncServices/syncGroups/cloudEndpoints | No |  No |
| storageSyncServices/syncGroups/serverEndpoints | No |  No |
| storageSyncServices/workflows | No |  No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| managers | Sí | Sí |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| streamingjobs | Sí (vea la nota siguiente) | Sí |
| streamingjobs/diagnosticSettings | No |  No |

> [!NOTE]
> No se pueden agregar etiquetas cuando hay instancias de streamingjobs en ejecución. Detenga el recurso si desea agregar una etiqueta.

## <a name="microsoftsubscription"></a>Microsoft.Subscription
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| CreateSubscription | No |  No |
| SubscriptionDefinitions | No |  No |
| SubscriptionOperations | No |  No |

## <a name="microsoftsupport"></a>microsoft.support
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| supporttickets | No |  No |

## <a name="microsoftterraformoss"></a>Microsoft.TerraformOSS
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| providerRegistrations | Sí | Sí |
| recursos | Sí | Sí |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| environments | Sí | No |
| environments/accessPolicies | No |  No |
| environments/eventsources | Sí | No |
| environments/referenceDataSets | Sí | No |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| cuenta | SÍ | Sí |
| account/extension | Sí | Sí |
| account/project | Sí | Sí |

## <a name="microsoftweb"></a>Microsoft.Web
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| apiManagementAccounts | No |  No |
| apiManagementAccounts/apiAcls | No |  No |
| apiManagementAccounts/apis | No |  No |
| apiManagementAccounts/apis/apiAcls | No |  No |
| apiManagementAccounts/apis/connectionAcls | No |  No |
| apiManagementAccounts/apis/connections | No |  No |
| apiManagementAccounts/apis/connections/connectionAcls | No |  No |
| apiManagementAccounts/apis/localizedDefinitions | No |  No |
| apiManagementAccounts/connectionAcls | No |  No |
| apiManagementAccounts/connections | No |  No |
| billingMeters | No |  No |
| certificados | Sí | Sí |
| connectionGateways | Sí | Sí |
| conexiones | Sí | Sí |
| customApis | Sí | Sí |
| deletedSites | No |  No |
| funciones | No |  No |
| hostingEnvironments | Sí | No |
| hostingEnvironments/multiRolePools | No |  No |
| hostingEnvironments/multiRolePools/instances | No |  No |
| hostingEnvironments/workerPools | No |  No |
| hostingEnvironments/workerPools/instances | No |  No |
| publishingUsers | No |  No |
| recomendaciones | No |  No |
| resourceHealthMetadata | No |  No |
| runtimes | No |  No |
| serverFarms | Sí | Sí |
| serverFarms/workers | No |  No |
| sitios | Sí | Sí |
| sites/domainOwnershipIdentifiers | No |  No |
| sites/hostNameBindings | No |  No |
| sites/instances | No |  No |
| sites/instances/extensions | No |  No |
| sites/premieraddons | Sí | Sí |
| sites/recommendations | No |  No |
| sites/resourceHealthMetadata | No |  No |
| sites/slots | Sí | Sí |
| sites/slots/hostNameBindings | No |  No |
| sites/slots/instances | No |  No |
| sites/slots/instances/extensions | No |  No |
| sourceControls | No |  No |
| validate | No |  No |
| verifyHostingEnvironmentVnet | No |  No |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| diagnosticSettings | No |  No |
| diagnosticSettingsCategories | No |  No |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| DeviceServices | Sí | Sí |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor
| Tipo de recurso | Compatible con las etiquetas | En el informe de costo de etiquetas |
| ------------- | ----------- | ----------- |
| components | No |  No |
| componentsSummary | No |  No |
| monitorInstances | No |  No |
| monitorInstancesSummary | No |  No |
| monitors | No |  No |
| notificationSettings | No |  No |

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre cómo aplicar etiquetas a recursos, consulte [Uso de etiquetas para organizar los recursos de Azure](resource-group-using-tags.md).
