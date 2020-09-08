---
title: Integración de Azure Relay con el servicio Azure Private Link
description: Información sobre la integración de Azure Relay con el servicio Azure Private Link
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: e5c35f9333378a5f0b87956e8a916491d51e3cb3
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719434"
---
# <a name="integrate-azure-relay-with-azure-private-link-preview"></a>Integración de Azure Relay con Azure Private Link (versión preliminar)
El **servicio Azure Private Link** le permite acceder a los servicios de Azure (por ejemplo, Azure Relay, Azure Service Bus, Azure Event Hubs, Azure Storage y Azure Cosmos DB) y a los servicios de asociados o clientes hospedados por Azure mediante un punto de conexión privado de la red virtual. Para más información, consulte [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

Un **punto de conexión privado** es una interfaz de red que permite que las cargas de trabajo que se ejecutan en una red virtual se conecten de forma privada y segura a un servicio que tiene un **recurso de vínculo privado** (por ejemplo, un espacio de nombres de Relay). El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesitan puertas de enlace, dispositivos NAT, ExpressRoute, conexiones de VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft y elimina la exposición a la red pública de Internet. Puede proporcionar un nivel de granularidad en el control de acceso si permite conexiones a espacios de nombres de Azure Relay específicos. 


> [!IMPORTANT]
> Esta funcionalidad actualmente está en su **versión preliminar**. 
>
> Actualmente se admiten conexiones de vínculo privado en los clientes del remitente. 


## <a name="add-a-private-endpoint-using-azure-portal"></a>Incorporación de un punto de conexión privado mediante Azure Portal

### <a name="prerequisites"></a>Requisitos previos
Para integrar un espacio de nombres de Azure Relay con Azure Private Link (versión preliminar), necesitará los permisos o las entidades siguientes:

- Un espacio de nombres de Azure Relay.
- Una red virtual de Azure.
- Una subred en la red virtual.
- Permisos de propietario o colaborador para la red virtual.

El punto de conexión privado y la red virtual deben estar en la misma región. Al seleccionar una región para el punto de conexión privado mediante el portal, solo se filtran automáticamente las redes virtuales que se encuentran en dicha región. El espacio de nombres puede estar en una región diferente.

El punto de conexión privado usa una dirección IP privada en la red virtual.

### <a name="steps"></a>Pasos
Para obtener instrucciones paso a paso sobre cómo crear un nuevo espacio de nombres de Azure Relay con entidades, consulte [Creación de un espacio de nombres de Relay mediante Azure Portal](relay-create-namespace-portal.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. En la barra de búsqueda, escriba **Retransmisiones**.
3. En la lista, seleccione el **espacio de nombres** al que desea agregar un punto de conexión privado.
4. Seleccione la pestaña **Redes** en **Configuración**.
5. Seleccione la pestaña **Conexiones de puntos de conexión privadas (versión preliminar)** en la parte superior de la página.
6. Seleccione el botón **+ Punto de conexión privado** en la parte superior de la página.

    ![Incorporación del botón de un punto de conexión privado](./media/private-link-service/add-private-endpoint-button.png)
7. En la página **Conceptos básicos**, siga estos pasos: 
    1. Seleccione la **suscripción de Azure** donde desea crear el punto de conexión privado. 
    2. Seleccione el **grupo de recursos** para el recurso de punto de conexión privado.
    3. Escriba el **Nombre** del punto de conexión privado. 
    5. Seleccione la **región** del punto de conexión privado. El punto de conexión privado debe estar en la misma región que la red virtual, que puede no ser la misma que la del espacio de nombres de Azure Relay al que se está conectando. 
    6. Seleccione **Siguiente: Recurso >** en la parte inferior de la página.

        ![Creación de un punto de conexión privado: página Conceptos básicos](./media/private-link-service/create-private-endpoint-basics-page.png)
8. En la página **Recurso**, siga estos pasos:
    1. En el método de conexión, si selecciona **Conectarse a un recurso de Azure en mi directorio**, tiene acceso de propietario o colaborador al espacio de nombres y dicho espacio de nombres está en el mismo directorio que el punto de conexión privado, siga estos pasos: 
        1. Seleccione la **suscripción de Azure** en la que existe el **espacio de nombres de Azure Relay**. 
        2. En **Tipo de recurso**, seleccione **Microsoft.Relay/namespaces** en **Tipo de recurso**.
        3. En **Recurso**, seleccione un espacio de nombres de Relay de la lista desplegable. 
        4. Confirme que **Subrecurso de destino** está establecido en **espacio de nombres**.
        5. Seleccione **Siguiente: Configuración >** situado en la parte inferior de la página. 
        
            ![Creación de un punto de conexión privado: página Recurso](./media/private-link-service/create-private-endpoint-resource-page.png)    
    2. Si selecciona **Conéctese a un recurso de Azure por identificador de recurso o alias** porque el espacio de nombres no está en el mismo directorio que el punto de conexión privado, siga estos pasos:
        1. Escriba el **identificador de recurso** o **alias**. Puede ser el identificador o alias del recurso que alguien haya compartido con usted. La forma más fácil de obtener el identificador de recurso es desplazarse hasta el espacio de nombres de Azure Relay de Azure Portal y copiar la parte del identificador URI a partir de `/subscriptions/`. Aquí tiene un ejemplo: `/subscriptions/000000000-0000-0000-0000-000000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Relay/namespaces/myrelaynamespace.` 
        2. En **Subrecurso de destino**, escriba **espacio de nombres**. Este es el tipo de subrecurso al que puede acceder el punto de conexión privado.
        3. (Opcional) Escriba un **mensaje de solicitud**. El propietario del recurso ve este mensaje mientras administra la conexión del punto de conexión privado.
        4. Después, seleccione **Next (Siguiente): Configuración >** situado en la parte inferior de la página.

            ![Creación de un punto de conexión privado: conexión mediante el identificador de recurso](./media/private-link-service/connect-resource-id.png)
9. En la página **Configuración**, seleccione la subred de una red virtual en la que desee implementar el punto de conexión privado. 
    1. Seleccione una **red virtual**. En la lista desplegable, solo se muestran las redes virtuales de la suscripción y la ubicación seleccionadas actualmente. 
    2. Seleccione una **subred** de la red virtual que seleccionó. 
    3. Habilite **Integrar con la zona DNS privada** para integrar el punto de conexión privado en una zona DNS privada. 
    
        Para conectar de forma privada con el punto de conexión privado, necesita un registro DNS. Se recomienda integrar el punto de conexión privado con una **zona DNS privada**. También se pueden usar los servidores DNS propios o bien crear registros DNS con los archivos de host de las máquinas virtuales. Para obtener más información, consulte [Configuración de DNS para puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md). En este ejemplo, está seleccionada la opción **Integrar con la zona DNS privada** y se creará una zona DNS privada automáticamente. 
    3. Seleccione **Siguiente: Etiquetas >** situado en la parte inferior de la página. 

        ![Creación de un punto de conexión privado: página Configuración](./media/private-link-service/create-private-endpoint-configuration-page.png)
10. En la página **Etiquetas**, cree todas las etiquetas (nombres y valores) que desee asociar al punto de conexión privado y la zona DNS privada (si tenía la opción habilitada). Después, en la parte inferior de la página, seleccione el botón **Revisar y crear**. 
11. En **Revisar y crear**, revise toda la configuración y seleccione **Crear** para crear el punto de conexión privado.
    
    ![Creación de un punto de conexión privado: página Revisar y crear](./media/private-link-service/create-private-endpoint-review-create-page.png)
12. En la página **Punto de conexión privado**, puede ver el estado de la conexión del punto de conexión privado. Si es el propietario del espacio de nombres de Relay o tiene acceso de administración sobre este y ha seleccionado la opción **Conectarse a un recurso de Azure en mi directorio** en **Método de conexión**, la conexión del punto de conexión debe ser **Aprobada automáticamente**. Si está en el estado **pendiente**, consulte la sección [Administración de puntos de conexión privados desde Azure Portal](#manage-private-endpoints-using-azure-portal).

    ![Página Punto de conexión privado](./media/private-link-service/private-endpoint-page.png)
13. Vuelva a la página **Redes** del **espacio de nombres** y cambie a la pestaña **Conexiones de punto de conexión privado (versión preliminar)** . Debería ver el punto de conexión privado que ha creado. 

    ![Punto de conexión privado creado](./media/private-link-service/private-endpoint-created.png)

## <a name="add-a-private-endpoint-using-powershell"></a>Incorporación de un punto de conexión privado mediante PowerShell
En el ejemplo siguiente se muestra cómo usar Azure PowerShell para crear una conexión de punto de conexión privado a un espacio de nombres de Azure Relay.

El punto de conexión privado y la red virtual deben estar en la misma región. El espacio de nombres de Azure Relay puede estar en una región diferente. Y el punto de conexión privado usa una dirección IP privada en la red virtual.

```azurepowershell-interactive

$rgName = "<RESOURCE GROUP NAME>"
$vnetlocation = "<VNET LOCATION>"
$vnetName = "<VIRTUAL NETWORK NAME>"
$subnetName = "<SUBNET NAME>"
$namespaceLocation = "<NAMESPACE LOCATION>"
$namespaceName = "<NAMESPACE NAME>"
$peConnectionName = "<PRIVATE ENDPOINT CONNECTION NAME>"

# create resource group
az group create -l $vnetLocation -n $rgName

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName $rgName `
                    -Location $vnetlocation `
                    -Name $vnetName `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name $subnetName `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork

# create a relay namespace
$namespaceResource = New-AzResource -Location $namespaceLocation -ResourceName $namespaceName -ResourceGroupName $rgName -Properties @{} -ResourceType "Microsoft.Relay/namespaces" 

# create a private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name $peConnectionName `
                                -PrivateLinkServiceId $namespaceResource.ResourceId `
                                -GroupId "namespace"

# get subnet object that you will use in the next step                                
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName  $rgName -Name $vnetName
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                                | Where-Object  {$_.Name -eq $subnetName}  
   
# now, create private endpoint   
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $rgName  `
                                -Name $vnetName   `
                                -Location $vnetlocation `
                                -Subnet  $subnet   `
                                -PrivateLinkServiceConnection $privateEndpointConnection

(Get-AzResource -ResourceId $namespaceResource.ResourceId -ExpandProperties).Properties


```

## <a name="manage-private-endpoints-using-azure-portal"></a>Administración de puntos de conexión privados desde Azure Portal

Cuando se crea un punto de conexión privado, se debe aprobar la conexión. Si el recurso (espacio de nombres de Relay) para el que va a crear un punto de conexión privado está en su directorio, puede aprobar la solicitud de conexión siempre que tenga privilegios de administración sobre el espacio de nombres de Relay. Si se va a conectar a un espacio de nombres de Relay para el que no tiene acceso de administración, debe esperar a que el propietario de ese recurso apruebe la solicitud de conexión.

Hay cuatro estados de aprovisionamiento:

| Acción del servicio | Estado de punto de conexión privado del consumidor del servicio | Descripción |
|--|--|--|
| None | Pending | La conexión se crea manualmente y está pendiente de aprobación por parte del propietario del espacio de nombres de Azure Relay. |
| Aprobación | Aprobado | La conexión se aprobó de forma automática o manual y está lista para usarse. |
| Reject | Rechazada | El propietario del espacio de nombres de Azure Relay rechazó la conexión. |
| Remove | Escenario desconectado | El propietario del espacio de nombres de Azure Relay quitó la conexión, el punto de conexión privado se vuelve informativo y debe eliminarse para la limpieza. |
 
###  <a name="approve-reject-or-remove-a-private-endpoint-connection"></a>Aprobación, rechazo o eliminación de una conexión de punto de conexión privado

1. Inicie sesión en Azure Portal.
1. En la barra de búsqueda, escriba **Retransmisión**.
1. Seleccione el **espacio de nombres** que desea administrar.
1. Seleccione la pestaña **Redes**.
5. Vaya a la sección correspondiente a continuación según la operación que desee: aprobar, rechazar o quitar. 

### <a name="approve-a-private-endpoint-connection"></a>Aprobación de una conexión de punto de conexión privado

1. Si hay alguna conexión pendiente, verá una conexión que aparece con el estado **Pendiente** como estado de aprovisionamiento. 
2. Seleccione el **punto de conexión privado** que desea aprobar.
3. Seleccione el botón **Aprobar**.

    ![Aprobación de un punto de conexión privado](./media/private-link-service/private-endpoint-approve.png)
4. En la página **Aprobación de la conexión** escriba un **comentario** opcional y seleccione **Yes**. Si selecciona **No**, no ocurrirá nada. 

    ![Página Aprobar conexión](./media/private-link-service/approve-connection-page.png)
5. Ahora puede ver que el estado de la conexión de la lista ha cambiado a **Aprobado**.

### <a name="reject-a-private-endpoint-connection"></a>Rechazo de una conexión de punto de conexión privado

1. Si hay conexiones del punto de conexión privado que desea rechazar, ya sea una solicitud pendiente o una conexión existente que se aprobó antes, seleccione la conexión del punto de conexión y haga clic en el botón **Rechazar**.

    ![Botón Rechazar](./media/private-link-service/private-endpoint-reject.png)
2. En la página **Rechazo de la conexión**, escriba un comentario opcional y seleccione **Sí**. Si selecciona **No**, no ocurrirá nada. 

    ![Página Rechazar conexión](./media/private-link-service/reject-connection-page.png)
3. Ahora puede ver que el estado de la conexión de la lista ha cambiado a **Rechazado**.


### <a name="remove-a-private-endpoint-connection"></a>Eliminación de una conexión de punto de conexión privado

1. Para eliminar una conexión de punto de conexión privado, selecciónela en la lista y seleccione **Eliminar** en la barra de herramientas. 

    ![Botón Quitar](./media/private-link-service/remove-endpoint.png)
2. En la página **Eliminar conexión**, seleccione **Sí** para confirmar la eliminación del punto de conexión privado. Si selecciona **No**, no ocurrirá nada. 

    ![Página Eliminar conexión](./media/private-link-service/delete-connection-page.png)
3. Ahora puede ver que el estado ha cambiado a **Desconectado**. A continuación, verá que el punto de conexión desaparece de la lista. 

## <a name="validate-that-the-private-link-connection-works"></a>Validación de que la conexión de vínculo privado funciona
Debe validar que los recursos de la red virtual del punto de conexión privado se están conectando a su espacio de nombres de Azure Relay a través de su dirección IP privada.

Para esta prueba, cree una máquina virtual siguiendo los pasos que encontrará en [Creación de una máquina virtual Windows en Azure Portal](../virtual-machines/windows/quick-create-portal.md).

Haga clic en la pestaña **Redes**: 

1. Especifique **Red virtual** y **Subred**. Debe seleccionar la instancia de Virtual Network en la que implementó el punto de conexión privado.
2. Especifique un recurso de **dirección IP pública**.
3. En **Grupo de seguridad de red de NIC**, seleccione **Ninguno**.
4. En **Equilibrio de carga**, seleccione **No**.

Conéctese a la VM, abra la línea de comandos y ejecute el siguiente comando:

```console
nslookup <your-relay-namespace-name>.servicebus.windows.net
```

Debería ver un resultado con el siguiente aspecto. 

```console
Non-authoritative answer:
Name:    <namespace-name>.privatelink.servicebus.windows.net
Address:  10.0.0.4 (private IP address associated with the private endpoint)
Aliases:  <namespace-name>.servicebus.windows.net
```

## <a name="limitations-and-design-considerations"></a>Limitaciones y consideraciones de diseño

### <a name="design-considerations"></a>Consideraciones de diseño
- El punto de conexión privado de Azure Relay está en **versión preliminar pública**. 
- Para más información sobre los precios, consulte [Precios de Azure Private Link (versión preliminar)](https://azure.microsoft.com/pricing/details/private-link/).

### <a name="limitations"></a>Limitaciones 
- Número máximo de puntos de conexión privados por espacio de nombres de Azure Relay: 64.
- Número máximo de espacios de nombres de Azure Relay con puntos de conexión privados por suscripción: 64.
- Las reglas de grupo de seguridad de red (NSG) y las rutas definidas por el usuario no se aplican al punto de conexión privado. Para más información, consulte [Servicio Azure Private Link: Limitaciones](../private-link/private-link-service-overview.md#limitations)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [servicio Azure Private Link](../private-link/private-link-service-overview.md).
- Más información acerca de [Azure Relay](relay-what-is-it.md)
