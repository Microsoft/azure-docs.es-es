---
title: ¿Qué es un punto de conexión privado de Azure?
description: Información sobre el punto de conexión privado de Azure
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: allensu
ms.openlocfilehash: 7456402605328592d4f5677767bcd985941173ec
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639841"
---
# <a name="what-is-azure-private-endpoint"></a>¿Qué es un punto de conexión privado de Azure?

Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual, colocando el servicio de manera eficaz en su red virtual. El servicio podría ser un servicio de Azure como Azure Storage, Azure Cosmos DB, SQL, etc. o su propio [servicio Private Link](private-link-service-overview.md).
  
## <a name="private-endpoint-properties"></a>Propiedades del punto de conexión privado 
 Un punto de conexión privado especifica las siguientes propiedades: 


|Propiedad  |Descripción |
|---------|---------|
|Nombre    |    Nombre único dentro del grupo de recursos.      |
|Subnet    |  Subred para implementar y asignar direcciones IP privadas de una red virtual. Para conocer los requisitos de subred, consulte la sección Limitaciones de este artículo.         |
|Recurso de vínculo privado    |   El recurso de vínculo privado al que conectarse mediante el identificador o el alias del recurso, de la lista de tipos disponibles. Se generará un identificador de red único para todo el tráfico enviado a este recurso.       |
|Subrecurso de destino   |      Subrecurso que se va a conectar. Cada tipo de recurso de vínculo privado tiene diferentes opciones para seleccionar según las preferencias.    |
|Método de aprobación de conexión    |  Automático o manual. Según los permisos del control de acceso basado en rol (RBAC), el punto de conexión privado se puede aprobar automáticamente. Si intenta conectarse a un recurso de vínculo privado sin RBAC, use el método manual para permitir que el propietario del recurso apruebe la conexión.        |
|Mensaje de solicitud     |  Puede especificar un mensaje para que las conexiones solicitadas se aprueben manualmente. Este mensaje se puede usar para identificar una solicitud específica.        |
|Estado de conexión   |   Propiedad de solo lectura que especifica si el punto de conexión privado está activo. Solo los puntos de conexión privados en un estado aprobado se pueden usar para enviar tráfico. Estados adicionales disponibles: <br>-**Aprobado**: La conexión se aprobó de forma automática o manual y está lista para usarse.</br><br>-**Pendiente**: La conexión se creó manualmente y está pendiente de aprobación por parte del propietario del recurso de vínculo privado.</br><br>-**Rechazado**: El propietario del recurso de vínculo privado rechazó la conexión.</br><br>-**Desconectado**: El propietario del recurso de vínculo privado quitó la conexión. El punto de conexión privado se vuelve informativo y debe eliminarse para la limpieza. </br>|

Estos son algunos detalles importantes acerca de los puntos de conexión privados: 
- El punto de conexión privado permite la conectividad entre los consumidores de la misma red virtual, las redes virtuales emparejadas de forma regional, las redes virtuales emparejadas de forma global y las instalaciones locales que usan [VPN](https://azure.microsoft.com/services/vpn-gateway/) o [Express Route](https://azure.microsoft.com/services/expressroute/) y servicios con la tecnología Private Link.
 
- Las conexiones de red solo pueden ser iniciadas por clientes que se conecten al punto de conexión privado. Los proveedores de servicios no tienen ninguna configuración de enrutamiento para iniciar conexiones en los consumidores del servicio. Las conexiones solo se pueden establecer en una dirección.

- Al crear un punto de conexión privado, también se crea una interfaz de red de solo lectura para el ciclo de vida del recurso. A la interfaz se le asigna una dirección IP privada desde la subred que se corresponde con el recurso de Private Link. El valor de la dirección IP privada no cambia durante todo el ciclo de vida del punto de conexión privado.
 
- El punto de conexión privado debe implementarse en la misma región que la red virtual. 
 
- El recurso de vínculo privado se puede implementar en una región distinta a la de la red virtual y el punto de conexión privado.
 
- Se pueden crear varios puntos de conexión privados mediante el mismo recurso de vínculo privado. En el caso de una sola red que use una configuración de servidor DNS común, el procedimiento recomendado es usar un único punto de conexión privado para un recurso de vínculo privado determinado para evitar entradas duplicadas o conflictos en la resolución de DNS. 
 
- Se pueden crear varios puntos de conexión privados en la misma subred o en subredes diferentes dentro de la misma red virtual. Existen límites en cuanto al número de puntos de conexión privados que se pueden crear en una suscripción. Para más información, consulte el artículo acerca de los  [límites de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).

- La suscripción del recurso de vínculo privado también se debe registrar con el proveedor de recursos Micosoft.Network. Para más información, consulte  [Proveedores de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types).

 
## <a name="private-link-resource"></a>Recurso de vínculo privado 
Un recurso de vínculo privado es el destino de un punto de conexión privado determinado. A continuación se muestra una lista de los tipos de recursos de vínculo privado disponibles: 
 
|Nombre del recurso de vínculo privado  |Tipo de recurso   |Subrecursos  |
|---------|---------|---------|
|**Servicio Private Link** (su propio servicio)   |  Microsoft.Network/privateLinkServices       | empty |
|**Azure Automation** |  Microsoft.Automation/automationAccounts | Webhook, DSCAndHybridWorker |
|**Azure SQL Database** | Microsoft.Sql/servers    |  Sql Server (sqlServer)        |
|**Azure Synapse Analytics** | Microsoft.Sql/servers    |  Sql Server (sqlServer)        | 
|**Almacenamiento de Azure**  | Microsoft.Storage/storageAccounts    |  Blob (blob, blob_secondary)<BR> Tabla (table, table_secondary)<BR> Cola (queue, queue_secondary)<BR> Archivo (file, file_secondary)<BR> Web (web, web_secondary)        |
|**Azure Data Lake Storage Gen2**  | Microsoft.Storage/storageAccounts    |  Blob (blob, blob_secondary)<BR> Sistema de archivos Data Lake Gen2 (dfs, dfs_secondary)       |
|**Azure Cosmos DB** | Microsoft.AzureCosmosDB/databaseAccounts    | Sql, MongoDB, Cassandra, Gremlin, Table|
|**Azure Database for PostgreSQL: servidor único** | Microsoft.DBforPostgreSQL/servers    | postgresqlServer |
|**Azure Database for MySQL** | Microsoft.DBforMySQL/servers    | mysqlServer |
|**Azure Database for MariaDB** | Microsoft.DBforMariaDB/servers    | mariadbServer |
|**Azure IoT Hub** | Microsoft.Devices/IotHubs    | iotHub |
|**Azure Key Vault** | Microsoft.KeyVault/vaults    | almacén |
|**Azure Kubernetes Service: API de Kubernetes** | Microsoft.ContainerService/managedClusters    | management |
|**Azure Search** | Microsoft.Search/searchService| searchService|  
|**Azure Container Registry** | Microsoft.ContainerRegistry/registries    | Registro |
|**Azure App Configuration** | Microsoft.Appconfiguration/configurationStores    | configurationStores |
|**Azure Backup** | Microsoft.RecoveryServices/vaults    | almacén |
|**Centro de eventos de Azure** | Microsoft.EventHub/namespaces    | espacio de nombres |
|**Azure Service Bus** | Microsoft.ServiceBus/namespaces | espacio de nombres |
|**Azure Relay** | Microsoft.Relay/namespaces | espacio de nombres |
|**Azure Event Grid** | Microsoft.EventGrid/topics    | topic |
|**Azure Event Grid** | Microsoft.EventGrid/domains    | dominio |
|**Azure WebApps** | Microsoft.Web/sites    | sites |
|**Azure Machine Learning** | Microsoft.MachineLearningServices/workspaces    | área de trabajo |
|**SignalR** | Microsoft.SignalRService/SignalR    | signalR |
|**Azure Monitor** | Microsoft.Insights/privateLinkScopes    | azuremonitor |
|**Cognitive Services** | (Microsoft.CognitiveServices/accounts    | account |
|**Azure File Sync** | Microsoft.StorageSync/storageSyncServices    | Afs |
    
  

  
 
## <a name="network-security-of-private-endpoints"></a>Seguridad de red de los puntos de conexión privados 
Cuando se usan puntos de conexión privados para los servicios de Azure, el tráfico se protege en un recurso de vínculo privado específico. La plataforma realiza un control de acceso para validar las conexiones de red que solo alcanzan el recurso de vínculo privado especificado. Para acceder a recursos adicionales dentro del mismo servicio de Azure, se requieren puntos de conexión privados adicionales. 
 
Puede impedir por completo que las cargas de trabajo accedan a puntos de conexión públicos para conectarse a un servicio de Azure admitido. Este control brinda una capa de seguridad de red adicional a los recursos al proporcionar una protección integrada de filtración que impide el acceso a otros recursos hospedados en el mismo servicio de Azure. 
 
## <a name="access-to-a-private-link-resource-using-approval-workflow"></a>Acceso a un recurso de vínculo privado mediante el flujo de trabajo de aprobación 
Puede conectarse a un recurso de vínculo privado mediante los siguientes métodos de aprobación de la conexión:
- Aprobación **automática** cuando posee el recurso de vínculo privado específico o tiene permisos sobre él. El permiso necesario se basa en el tipo de recurso de vínculo privado con el siguiente formato: Microsoft.\<Provider>/<tipo_recurso>/privateEndpointConnectionApproval/action
- Solicitud **manual** si no tiene el permiso necesario y desea solicitar acceso. Se iniciará un flujo de trabajo de aprobación. El punto de conexión privado y la conexión de punto de conexión privado subsiguiente se crearán en un estado "Pendiente". El propietario del recurso de vínculo privado es responsable de aprobar la conexión. Una vez aprobada, el punto de conexión privado se habilita para enviar el tráfico normalmente, como se muestra en el siguiente diagrama de flujo de trabajo de aprobación.  

![aprobación de flujo de trabajo](media/private-endpoint-overview/private-link-paas-workflow.png)
 
El propietario del recurso de vínculo privado puede realizar las siguientes acciones a través de una conexión de punto de conexión privado: 
- Revisar todos los detalles de las conexiones de los puntos de conexión privados. 
- Aprobar una conexión de punto de conexión privado. El punto de conexión privado correspondiente se habilitará para enviar tráfico al recurso de vínculo privado. 
- Rechazar una conexión de punto de conexión privado. El punto de conexión privado correspondiente se actualizará para reflejar el estado.
- Eliminar una conexión de punto de conexión privado en cualquier estado. El punto de conexión privado correspondiente se actualizará con un estado desconectado para reflejar la acción. El propietario del punto de conexión privado solo puede eliminar el recurso en este momento. 
 
> [!NOTE]
> Solo un punto de conexión privado en un estado aprobado puede enviar tráfico a un recurso de vínculo privado determinado. 

### <a name="connecting-using-alias"></a>Conexión mediante alias
El alias es un moniker único que se genera cuando el propietario del servicio crea el servicio de vínculo privado detrás de un equilibrador de carga estándar. El propietario del servicio puede compartir este alias con sus consumidores sin conexión. Los consumidores pueden solicitar una conexión al servicio de vínculo privado mediante el URI de recurso o el alias. Si desea conectarse mediante el alias, debe crear un punto de conexión privado mediante el método de aprobación de conexión manual. Para usar el método de aprobación de conexión manual, establezca el parámetro solicitud manual en true durante el flujo de creación del punto de conexión privado. Consulte [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint?view=azps-2.6.0) y [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create) para obtener más información. 

## <a name="dns-configuration"></a>Configuración de DNS 
Al conectarse a un recurso de vínculo privado mediante un nombre de dominio completo (FQDN) como parte de la cadena de conexión, es importante establecer correctamente la configuración de DNS para que se resuelva en la dirección IP privada asignada. Es posible que los servicios de Azure existentes ya tengan una configuración de DNS que usar al conectarse a través de un punto de conexión público. Este se debe invalidar para conectarse con el punto de conexión privado. 
 
La interfaz de red asociada con el punto de conexión privado contiene el conjunto completo de información necesaria para configurar el DNS, incluidos el FQDN y las direcciones IP privadas asignadas a un recurso de vínculo privado determinado. 

Para obtener información detallada y completa acerca de los procedimientos recomendados y las recomendaciones para configurar DNS para puntos de conexión privados, consulte el [artículo sobre la configuración de DNS en puntos de conexión privados](private-endpoint-dns.md).



 
## <a name="limitations"></a>Limitaciones
 
En la tabla siguiente se incluye una lista de las limitaciones conocidas al usar puntos de conexión privados: 


|Limitación |Descripción |Mitigación  |
|---------|---------|---------|
|Las reglas de grupo de seguridad de red (NSG) y las rutas definidas por el usuario no se aplican al punto de conexión privado.    |El grupo de seguridad de red no se admite en los puntos de conexión privados. Si bien las subredes que contienen el punto de conexión privado pueden tener un grupo de seguridad de red asociado, las reglas no serán efectivas en el tráfico procesado por el punto de conexión privado. Debe tener la [aplicación de directivas de red deshabilitada](disable-private-endpoint-network-policy.md) para implementar puntos de conexión privados en una subred. El grupo de seguridad de red se sigue aplicando en otras cargas de trabajo hospedadas en la misma subred. Las rutas de cualquier subred de cliente utilizarán un prefijo /32, lo que cambia el comportamiento de enrutamiento predeterminado requiere un UDR similar.  | Controle el tráfico mediante el uso de reglas del grupo de seguridad de red para el tráfico saliente en los clientes de origen. Implementación de rutas individuales con un prefijo /32 para invalidar rutas de punto de conexión privado. Todavía se admiten los registros de flujo de NSG y la información de supervisión de las conexiones salientes y se pueden usar.        |


## <a name="next-steps"></a>Pasos siguientes
- [Creación de un punto de conexión privado para SQL Database mediante Azure Portal](create-private-endpoint-portal.md)
- [Creación de un punto de conexión privado para SQL Database mediante PowerShell](create-private-endpoint-powershell.md)
- [Creación de un punto de conexión privado para SQL Database mediante la CLI](create-private-endpoint-cli.md)
- [Creación de un punto de conexión privado para la cuenta de almacenamiento mediante el portal](create-private-endpoint-storage-portal.md)
- [Creación de un punto de conexión privado para la cuenta de Azure Cosmos mediante el portal](../cosmos-db/how-to-configure-private-endpoints.md)
- [Creación del propio servicio Private Link con Azure PowerShell](create-private-link-service-powershell.md)
- [Creación de su propio servicio Private Link para Azure Database for PostgreSQL: servidor único con el Portal](../postgresql/howto-configure-privatelink-portal.md)
- [Creación de su propio servicio Private Link para Azure Database for PostgreSQL: servidor único con la CLI](../postgresql/howto-configure-privatelink-cli.md)
- [Creación de su propio servicio Private Link para Azure Database for MySQL con el Portal](../mysql/howto-configure-privatelink-portal.md)
- [Creación de su propio servicio Private Link para Azure Database for MySQL con la CLI](../mysql/howto-configure-privatelink-cli.md)
- [Creación de su propio servicio Private Link para Azure Database for MariaDB con el Portal](../mariadb/howto-configure-privatelink-portal.md)
- [Creación de su propio servicio Private Link para Azure Database for MariaDB con la CLI](../mariadb/howto-configure-privatelink-cli.md)
