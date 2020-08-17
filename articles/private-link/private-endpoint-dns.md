---
title: Configuración de DNS para puntos de conexión privados de Azure
description: Conozca la configuración de DNS para puntos de conexión privados de Azure
services: private-link
author: mblanco77
ms.service: private-link
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: allensu
ms.openlocfilehash: df1896caaa0cba1f62dc1466124b393337fa8c83
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985791"
---
# <a name="azure-private-endpoint-dns-configuration"></a>Configuración de DNS para puntos de conexión privados de Azure


Al conectarse a un recurso de vínculo privado mediante un nombre de dominio completo (FQDN) como parte de la cadena de conexión, es importante establecer correctamente la configuración de DNS para que se resuelva en la dirección IP privada asignada. Es posible que los servicios de Microsoft Azure existentes ya tengan una configuración de DNS que usar al conectarse a través de un punto de conexión público. Esta configuración se debe invalidar para realizar la conexión mediante el punto de conexión privado. 
 
La interfaz de red asociada con el punto de conexión privado contiene el conjunto completo de información necesaria para configurar el DNS, incluidos el FQDN y las direcciones IP privadas asignadas a un recurso de vínculo privado determinado. 
 
Puede usar las siguientes opciones para establecer la configuración de DNS para los puntos de conexión privados: 
- **Use el archivo de host (solo se recomienda para pruebas)** . Puede usar el archivo de host en una máquina virtual para invalidar el DNS.  
- **Use una zona DNS privada**. Puede usar [zonas DNS privadas](../dns/private-dns-privatednszone.md) para invalidar la resolución DNS de un punto de conexión privado determinado. Una zona DNS privada se puede vincular a la red virtual para resolver dominios específicos.
- **Use el reenviador DNS (opcional)** . Puede usar su reenviador DNS a fin de invalidar la resolución DNS para un determinado recurso de vínculo privado. Si el [servidor DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) se hospeda en una red virtual, puede crear una regla de reenvío de DNS para usar una zona DNS privada con el fin de simplificar la configuración de todos los recursos de vínculo privado.
 
> [!IMPORTANT]
> No se recomienda invalidar una zona que esté en uso activamente para resolver puntos de conexión públicos. Las conexiones a los recursos no podrán resolverse correctamente sin el reenvío de DNS al DNS público. Para evitar problemas, cree un nombre de dominio diferente o siga el nombre sugerido para cada servicio que aparece a continuación. 

## <a name="azure-services-dns-zone-configuration"></a>Configuración de zonas DNS de los servicios de Azure
Los servicios de Azure crearán un registro de DNS de nombre canónico (CNAME) en el servicio DNS público para redirigir la resolución a los nombres de dominio privados sugeridos. Puede invalidar la resolución con la dirección IP privada de los puntos de conexión privados. 
 
No es necesario que las aplicaciones cambien la dirección URL de conexión. Al intentar resolver mediante un servicio DNS público, el servidor DNS se resolverá ahora en los puntos de conexión privados. El proceso no afecta a las aplicaciones existentes. 

En el caso de los servicios de Azure, use los nombres de zona recomendados tal y como se describe en la tabla siguiente:

| Tipo de recurso de Private Link / subrecurso |Nombre de zona de DNS privado | Reenviadores de zona DNS pública |
|---|---|---|---|
| Azure Automation / (Microsoft.Automation/automationAccounts) / Webhook, DSCAndHybridWorker | privatelink.azure-automation.net | azure-automation.net |
| Azure SQL Database (Microsoft.Sql/servers) / SQL Server | privatelink.database.windows.net | database.windows.net |
| Azure Synapse Analytics (Microsoft.Sql/servers) / SQL Server  | privatelink.database.windows.net | database.windows.net |
| Storage account (Microsoft.Storage/storageAccounts) / Blob (blob, blob_secondary) | privatelink.blob.core.windows.net | blob.core.windows.net |
| Storage account (Microsoft.Storage/storageAccounts) / Table (table, table_secondary) | privatelink.table.core.windows.net | table.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / Queue (queue, queue_secondary) | privatelink.queue.core.windows.net | queue.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / File (file, file_secondary) | privatelink.file.core.windows.net | file.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / Web (web, web_secondary) | privatelink.web.core.windows.net | web.core.windows.net |
| Sistema de archivos Azure Data Lake Gen2 (Microsoft.Storage/storageAccounts) / Data Lake File System Gen2 (dfs, dfs_secondary) | privatelink.dfs.core.windows.net | dfs.core.windows.net |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.com | documents.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.com | mongo.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.com | cassandra.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.com | gremlin.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Table | privatelink.table.cosmos.azure.com | table.cosmos.azure.com |
| Azure Database for PostgreSQL: servidor único (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.azure.com | postgres.database.azure.com |
| Azure Database for MySQL (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.azure.com | mysql.database.azure.com |
| Azure Database for MariaDB (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.azure.com | mariadb.database.azure.com |
| Azure Key Vault (Microsoft.KeyVault/vaults) / vault | privatelink.vaultcore.azure.net | vault.azure.net <br> vaultcore.azure.net |
| Azure Kubernetes Service: API de Kubernetes (Microsoft.ContainerService/managedClusters) / management | privatelink.{region}.azmk8s.io | {region}.azmk8s.io |
| Azure Search (Microsoft.Search/searchServices) / searchService | privatelink.search.windows.net | search.windows.net |
| Azure Container Registry (Microsoft.ContainerRegistry/registries) / registry | privatelink.azurecr.io | azurecr.io |
| Azure App Configuration (Microsoft.AppConfiguration/configurationStores) / configurationStore | privatelink.azconfig.io | azconfig.io |
| Azure Backup (Microsoft.RecoveryServices/vaults) / vault | privatelink.{region}.backup.windowsazure.com | {region}.backup.windowsazure.com |
| Azure Event Hubs (Microsoft.EventHub/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure Service Bus (Microsoft.ServiceBus/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure IoT Hub (Microsoft.Devices/IotHubs) / iotHub | privatelink.azure-devices.net | azure-devices.net |
| Azure Relay (Microsoft.Relay/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure Event Grid (Microsoft.EventGrid/topics) / topic | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure Event Grid (Microsoft.EventGrid/domains) / domain | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure Web Apps (Microsoft.Web/sites) / sites | privatelink.azurewebsites.net | azurewebsites.net |
| Azure Machine Learning (Microsoft.MachineLearningServices/workspaces) / workspace | privatelink.api.azureml.ms | api.azureml.ms |
| IoT Hub (Microsoft.Devices/IotHubs) / IotHub | privatelink.azure-devices.net | azure-devices.net |
| SignalR (Microsoft.SignalRService/SignalR ) / signalR | privatelink.service.signalr.net | service.signalr.net |
| Azure Monitor (Microsoft.Insights/privateLinkScopes) / azuremonitor | privatelink.monitor.azure.com<br/> privatelink.oms.opinsights.azure.com <br/> privatelink.ods.opinsights.azure.com <br/> privatelink.agentsvc.azure-automation.com | monitor.azure.com<br/> oms.opinsights.azure.com<br/> ods.opinsights.azure.com<br/> agentsvc.azure-automation.com |
| Cognitive Services (Microsoft.CognitiveServices/accounts) / account | privatelink.cognitiveservices.azure.com  | cognitiveservices.azure.com  |
| Azure File Sync (Microsoft.StorageSync/storageSyncServices) / afs |  privatelink.afs.azure.net  |  afs.azure.net  |

 
## <a name="dns-configuration-scenarios"></a>Escenarios de configuración de DNS

El FQDN de los servicios se resuelve automáticamente en una dirección IP pública. Para resolver la dirección IP privada del punto de conexión privado, debe cambiar la configuración de DNS en consecuencia.

DNS es un componente fundamental para que la aplicación funcione correctamente al resolver de forma correcta la dirección IP del punto de conexión privado.

En función de sus preferencias, los siguientes escenarios están disponibles para la resolución DNS integrada:

- [Cargas de trabajo de red virtual sin un servidor DNS personalizado](#virtual-network-workloads-without-custom-dns-server)
- [Cargas de trabajo locales que utilizan un reenviador DNS](#on-premises-workloads-using-a-dns-forwarder)
- [Red virtual y cargas de trabajo locales mediante un reenviador DNS](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder)


## <a name="virtual-network-workloads-without-custom-dns-server"></a>Cargas de trabajo de red virtual sin un servidor DNS personalizado

Esta configuración es adecuada para cargas de trabajo de red virtual sin un servidor DNS personalizado. En este escenario, el cliente consulta la dirección IP del punto de conexión privado en el servicio DNS proporcionado de Azure [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md). Azure DNS será responsable de la resolución DNS de las zonas DNS privadas.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. En el caso de otros servicios, puede ajustar el modelo mediante la siguiente referencia: [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red virtual de cliente

- Zona de DNS privado [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) con [registro tipo A](../dns/dns-zones-records.md#record-types)

- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En la captura de pantalla siguiente se muestra la secuencia de la resolución DNS de cargas de trabajo de red virtual que usan la zona DNS privada:

:::image type="content" source="media/private-endpoint-dns/single-vnet-azure-dns.png" alt-text="Red virtual única y DNS proporcionado por Azure":::

Este modelo se puede extender a varias redes virtuales del mismo nivel que están asociadas al mismo punto de conexión privado. Esto puede realizarse con la [adición de nuevos vínculos de red virtual](../dns/private-dns-virtual-network-links.md) a la zona DNS privada para todas las redes virtuales del mismo nivel.

> [!IMPORTANT]
> Se requiere una única zona DNS privada para esta configuración. La creación de varias zonas con el mismo nombre para diferentes redes virtuales necesitaría operaciones manuales para combinar los registros de DNS.

> [!IMPORTANT]
> Si usa un punto de conexión privado en un modelo radial de una suscripción diferente, vuelva a usar la misma zona DNS privada en el centro de conectividad.

En este escenario, hay una topología de red [radial](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) en la que las redes de radio que tienen un punto de conexión privado común y todas las redes virtuales de radio están vinculadas a la misma zona DNS privada. 

:::image type="content" source="media/private-endpoint-dns/hub-and-spoke-azure-dns.png" alt-text="Topología de redes de centro y radio con DNS proporcionado por Azure":::

## <a name="on-premises-workloads-using-a-dns-forwarder"></a>Cargas de trabajo locales que utilizan un reenviador DNS

Con el fin de que las cargas de trabajo locales puedan resolver un FQDN de un punto de conexión privado en la dirección IP privada, es necesario utilizar un reenviador DNS para implementar en Azure la resolución de la [zona DNS pública](#azure-services-dns-zone-configuration) del servicio de Azure.

El siguiente escenario es adecuado para una red local que tenga un reenviador DNS en Azure, que a su vez es responsable de resolver todas las consultas de DNS a través de un reenviador de nivel de servidor en el DNS [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md) proporcionado por Azure. 

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. Para otros servicios, puede ajustar el modelo mediante la siguiente referencia  [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitará los siguientes recursos:

- Red local
- Red virtual  [conectada a la red local](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/)
- Reenviador DNS implementado en Azure 
- Zonas de DNS privado  [privatelink.database.windows.net](../dns/private-dns-privatednszone.md)  con  [registro tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la secuencia de resolución DNS de una red local que utiliza un reenviador DNS implementado en Azure, donde la resolución la realiza una zona DNS privada [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md):

:::image type="content" source="media/private-endpoint-dns/on-premises-using-azure-dns.png" alt-text="Red local que utiliza Azure DNS":::

Esta configuración se puede extender para una red local que ya cuenta con una solución DNS. 
La solución DNS local debe configurarse para que reenvíe el tráfico DNS a Azure DNS a través de un  [reenviador condicional](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) que haga referencia al reenviador DNS implementado en Azure.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. Para otros servicios, puede ajustar el modelo mediante la siguiente referencia  [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitará los siguientes recursos:

- Red local con una solución DNS personalizada 
- Red virtual  [conectada a la red local](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/)
- Reenviador DNS implementado en Azure
- Zonas de DNS privado  [privatelink.database.windows.net](../dns/private-dns-privatednszone.md)   con  [registro tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la secuencia de resolución DNS de una red local que reenvía condicionalmente el tráfico DNS dirigido a Azure, donde la resolución la realiza una zona DNS privada  [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md).

> [!IMPORTANT]
> El reenvío condicional debe realizarse en el [reenviador de zona DNS pública](#azure-services-dns-zone-configuration). Por ejemplo  `database.windows.net` , en lugar de  **privatelink**.database.windows.net.

:::image type="content" source="media/private-endpoint-dns/on-premises-forwarding-to-azure.png" alt-text="Reenvío local a Azure DNS":::

## <a name="virtual-network-and-on-premises-workloads-using-a-dns-forwarder"></a>Red virtual y cargas de trabajo locales mediante un reenviador DNS

Para un enfoque general que sea adecuado para cargas de trabajo que necesitan acceso a un punto de conexión privado desde redes virtuales y locales, es necesario utilizar un reenviador DNS compartido para implementar en Azure la resolución de la [zona DNS pública](#azure-services-dns-zone-configuration) del servicio de Azure.

El siguiente escenario es adecuado para una red local que tenga un reenviador DNS en Azure y redes virtuales que necesiten acceso al punto de conexión privado ubicado en una red de concentrador compartida.  

Este reenviador DNS es responsable de resolver todas las consultas de DNS a través de un reenviador de nivel de servidor en el servicio DNS proporcionado por Azure [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md).

> [!IMPORTANT]
> Se requiere una única zona DNS privada para esta configuración. Todas las conexiones de cliente realizadas desde las [redes virtuales emparejadas locales](../virtual-network/virtual-network-peering-overview.md) deben usar también la misma zona DNS privada.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. Para otros servicios, puede ajustar el modelo mediante la siguiente referencia  [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitará los siguientes recursos:

- Red local
- Red virtual  [conectada a la red local](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/)
- [Red virtual emparejada](../virtual-network/virtual-network-peering-overview.md) 
- Reenviador DNS implementado en Azure
- Zonas de DNS privado  [privatelink.database.windows.net](../dns/private-dns-privatednszone.md)   con  [registro tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la secuencia de resolución DNS de una red local y virtual que utiliza un reenviador DNS implementado en Azure, donde la resolución la realiza una zona DNS privada  [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md):

:::image type="content" source="media/private-endpoint-dns/hybrid-scenario.png" alt-text="escenario híbrido":::

## <a name="next-steps"></a>Pasos siguientes
- [Más información sobre los puntos de conexión privados](private-endpoint-overview.md)
