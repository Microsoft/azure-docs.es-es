---
title: Integración de Azure Service Bus con Azure Private Link Service
description: Aprenda a integrar Azure Service Bus con Azure Private Link Service
author: spelluru
ms.author: spelluru
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: 4f3b67794d1a7f3935c79c70f18b8bd4a1e0d7ef
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716629"
---
# <a name="allow-access-to-azure-service-bus-namespaces-via-private-endpoints"></a>Permiso para acceder a los espacios de nombres de Azure Service Bus a través de puntos de conexión privados

El servicio Azure Private Link le permite acceder a los servicios de Azure (por ejemplo, Azure Service Bus, Azure Storage y Azure Cosmos DB) y a los servicios de asociados o clientes hospedados por Azure mediante un **punto de conexión privado** de la red virtual.

Un punto de conexión privado es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. Puede conectarse a una instancia de un recurso de Azure, lo que le otorga el nivel más alto de granularidad en el control de acceso.

Para más información, consulte [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

>[!WARNING]
> La implementación de puntos de conexión privados puede evitar que otros servicios de Azure interactúen con Service Bus.
>
> Los servicios de confianza de Microsoft no se admiten cuando se usan instancias de Virtual Network.
>
> Estos son los escenarios comunes de Azure que no funcionan con instancias de Virtual Network (tenga en cuenta que la lista **NO** está completa).
> - Integración con Azure Event Grid
> - Enrutamientos de Azure IoT Hub
> - Azure IoT Device Explorer
>
> Los siguientes servicios de Microsoft deben estar en una red virtual
> - Azure App Service
> - Azure Functions

> [!IMPORTANT]
> Esta característica es compatible con el nivel **premium** de Azure Service Bus. Para más información sobre el nivel premium, consulte el artículo [Niveles de mensajería Premium y Estándar de Service Bus](service-bus-premium-messaging.md).


## <a name="add-a-private-endpoint-using-azure-portal"></a>Incorporación de un punto de conexión privado mediante Azure Portal

### <a name="prerequisites"></a>Requisitos previos

Para integrar un espacio de nombres de Service Bus con Azure Private Link, necesitará las siguientes entidades o permisos:

- Un espacio de nombres de Service Bus.
- Una red virtual de Azure.
- Una subred en la red virtual. Puede usar la subred **predeterminada**. 
- Permisos de propietario o colaborador para el espacio de nombres de Service Bus y la red virtual.

El punto de conexión privado y la red virtual deben estar en la misma región. Al seleccionar una región para el punto de conexión privado mediante el portal, solo se filtran automáticamente las redes virtuales que se encuentran en dicha región. El espacio de nombres de Service Bus puede estar en una región diferente. Y el punto de conexión privado usa una dirección IP privada en la red virtual.

### <a name="steps"></a>steps

Si ya tiene un espacio de nombres existente, puede crear un punto de conexión privado siguiendo estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. En la barra de búsqueda, escriba **Service Bus**.
3. En la lista, seleccione el **espacio de nombres** al que desea agregar un punto de conexión privado.
2. En el menú de la izquierda, seleccione la opción **Redes** en **Configuración**. 

    > [!NOTE]
    > Puede ver la pestaña **Redes** solo para los espacios de nombres **prémium**.  
    
    De forma predeterminada, está seleccionada la opción **Redes seleccionadas**. Si no agrega al menos una regla de firewall de IP o una red virtual en esta página, se podrá acceder al espacio de nombres desde la red pública de Internet (mediante la clave de acceso).

    :::image type="content" source="./media/service-bus-ip-filtering/default-networking-page.png" alt-text="Página Redes: predeterminada" lightbox="./media/service-bus-ip-filtering/default-networking-page.png":::
    
    Si selecciona la opción **Todas las redes**, el espacio de nombres de Service Bus aceptará conexiones procedentes de cualquier dirección IP (mediante la clave de acceso). Esta configuración predeterminada es equivalente a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0. 

    ![Firewall: opción Todas las redes seleccionada](./media/service-bus-ip-filtering/firewall-all-networks-selected.png)
5. Para permitir el acceso al espacio de nombres a través de puntos de conexión privados, seleccione la pestaña **Conexiones de puntos de conexión privado** en la parte superior de la página.
6. Seleccione el botón **+ Punto de conexión privado** en la parte superior de la página.

    ![Incorporación del botón de un punto de conexión privado](./media/private-link-service/private-link-service-3.png)
7. En la página **Conceptos básicos**, siga estos pasos: 
    1. Seleccione la **suscripción de Azure** donde desea crear el punto de conexión privado. 
    2. Seleccione el **grupo de recursos** para el recurso de punto de conexión privado.
    3. Escriba el **Nombre** del punto de conexión privado. 
    5. Seleccione la **región** del punto de conexión privado. El punto de conexión privado debe estar en la misma región que la red virtual, que puede no ser la misma que la del recurso de Private Link al que se está conectando. 
    6. Seleccione **Siguiente: Recurso >** en la parte inferior de la página.

        ![Creación de un punto de conexión privado: página Conceptos básicos](./media/private-link-service/create-private-endpoint-basics-page.png)
8. En la página **Recurso**, siga estos pasos:
    1. Como método de conexión, si selecciona **Conectarse a un recurso de Azure en mi directorio.** , siga estos pasos:   
        1. Seleccione la **suscripción de Azure** en la que existe el **espacio de nombres de Service Bus**. 
        2. En **Tipo de recurso**, seleccione **Microsoft.ServiceBus/namespaces** para el **tipo de recurso**.
        3. En **Recurso**, seleccione un espacio de nombres de Service Bus de la lista desplegable. 
        4. Confirme que **Subrecurso de destino** está establecido en **espacio de nombres**.
        5. Seleccione **Siguiente: Configuración >** situado en la parte inferior de la página. 
        
            ![Creación de un punto de conexión privado: página Recurso](./media/private-link-service/create-private-endpoint-resource-page.png)
    2. Si selecciona **Conéctese a un recurso de Azure por identificador de recurso o alias.** , siga estos pasos:
        1. Escriba el **identificador de recurso** o **alias**. Puede ser el identificador o alias del recurso que alguien haya compartido con usted. La forma más fácil de obtener el identificador de recurso es desplazarse hasta el espacio de nombres de Service Bus de Azure Portal y copiar la parte del identificador URI a partir de `/subscriptions/`. Vea la imagen siguiente como ejemplo. 
        2. En **Subrecurso de destino**, escriba **espacio de nombres**. Este es el tipo de subrecurso al que puede acceder el punto de conexión privado. 
        3. (Opcional) Escriba un **mensaje de solicitud**. El propietario del recurso ve este mensaje mientras administra la conexión del punto de conexión privado. 
        4. Después, seleccione **Next (Siguiente): Configuración >** situado en la parte inferior de la página. 

            ![Creación de un punto de conexión privado: conexión mediante el identificador de recurso](./media/private-link-service/connect-resource-id.png)
9. En la página **Configuración**, seleccione la subred de una red virtual en la que desee implementar el punto de conexión privado. 
    1. Seleccione una **red virtual**. En la lista desplegable, solo se muestran las redes virtuales de la suscripción y la ubicación seleccionadas actualmente. 
    2. Seleccione una **subred** de la red virtual que seleccionó. 
    3. Seleccione **Siguiente: Etiquetas >** situado en la parte inferior de la página. 

        ![Creación de un punto de conexión privado: página Configuración](./media/private-link-service/create-private-endpoint-configuration-page.png)
10. En la página **Etiquetas**, cree cualquier etiqueta (nombres y valores) que desee asociar al recurso de punto de conexión privado. Después, en la parte inferior de la página, seleccione el botón **Revisar y crear**. 
11. En **Revisar y crear**, revise toda la configuración y seleccione **Crear** para crear el punto de conexión privado.
    
    ![Creación de un punto de conexión privado: página Revisar y crear](./media/private-link-service/create-private-endpoint-review-create-page.png)
12. Confirme que se ha creado el punto de conexión privado. Si es el propietario del recurso y ha seleccionado la opción **Connect to an Azure resource in my directory** (Conectar a un recurso de Azure en mi directorio) en **Connection method** (Método de conexión), la conexión del punto de conexión debe ser **aprobada automáticamente**. Si está en el estado **pendiente**, consulte la sección [Administración de puntos de conexión privados desde Azure Portal](#manage-private-endpoints-using-azure-portal).

    ![Punto de conexión privado creado](./media/private-link-service/private-endpoint-created.png)

## <a name="add-a-private-endpoint-using-powershell"></a>Incorporación de un punto de conexión privado mediante PowerShell
En el ejemplo siguiente se muestra cómo usar Azure PowerShell para crear una conexión de punto de conexión privado a un espacio de nombres de Service Bus.

El punto de conexión privado y la red virtual deben estar en la misma región. El espacio de nombres de Service Bus puede estar en una región diferente. Y el punto de conexión privado usa una dirección IP privada en la red virtual.

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

# create premium service bus namespace
$namespaceResource = New-AzResource -Location $namespaceLocation -ResourceName $namespaceName -ResourceGroupName $rgName -Sku @{name = "Premium"; capacity = 1} -Properties @{} -ResourceType "Microsoft.ServiceBus/namespaces" -

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

Cuando se crea un punto de conexión privado, se debe aprobar la conexión. Si el recurso para el que va a crear un punto de conexión privado está en el directorio, puede aprobar la solicitud de conexión siempre que tenga permisos suficientes. Si se va a conectar a un recurso de Azure en otro directorio, debe esperar a que el propietario de ese recurso apruebe la solicitud de conexión.

Hay cuatro estados de aprovisionamiento:

| Acción del servicio | Estado de punto de conexión privado del consumidor del servicio | Descripción |
|--|--|--|
| None | Pending | La conexión se crea manualmente y está pendiente de aprobación por parte del propietario del recurso de Private Link. |
| Aprobación | Aprobado | La conexión se aprobó de forma automática o manual y está lista para usarse. |
| Reject | Rechazada | El propietario del recurso de vínculo privado rechazó la conexión. |
| Remove | Escenario desconectado | El propietario del recurso del vínculo privado quitó la conexión, el punto de conexión privado se vuelve informativo y debe eliminarse para la limpieza. |
 
###  <a name="approve-reject-or-remove-a-private-endpoint-connection"></a>Aprobación, rechazo o eliminación de una conexión de punto de conexión privado

1. Inicie sesión en Azure Portal.
1. En la barra de búsqueda, escriba **Service Bus**.
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

    ![Estado de la conexión: aprobada](./media/private-link-service/connection-status-approved.png)

### <a name="reject-a-private-endpoint-connection"></a>Rechazo de una conexión de punto de conexión privado

1. Si hay conexiones del punto de conexión privado que desea rechazar, ya sea una solicitud pendiente o una conexión existente que se aprobó antes, seleccione la conexión del punto de conexión y haga clic en el botón **Rechazar**.

    ![Botón Rechazar](./media/private-link-service/private-endpoint-reject.png)
2. En la página **Rechazo de la conexión**, escriba un comentario opcional y seleccione **Sí**. Si selecciona **No**, no ocurrirá nada. 

    ![Página Rechazar conexión](./media/private-link-service/reject-connection-page.png)
3. Ahora puede ver que el estado de la conexión de la lista ha cambiado a **Rechazado**. 

    ![Punto de conexión rechazado](./media/private-link-service/endpoint-rejected.png)


### <a name="remove-a-private-endpoint-connection"></a>Eliminación de una conexión de punto de conexión privado

1. Para eliminar una conexión de punto de conexión privado, selecciónela en la lista y seleccione **Eliminar** en la barra de herramientas. 

    ![Botón Quitar](./media/private-link-service/remove-endpoint.png)
2. En la página **Eliminar conexión**, seleccione **Sí** para confirmar la eliminación del punto de conexión privado. Si selecciona **No**, no ocurrirá nada. 

    ![Página Eliminar conexión](./media/private-link-service/delete-connection-page.png)
3. Ahora puede ver que el estado ha cambiado a **Desconectado**. A continuación, verá que el punto de conexión desaparece de la lista. 

## <a name="validate-that-the-private-link-connection-works"></a>Validación de que la conexión de vínculo privado funciona

Debe comprobar que los recursos de la red virtual del punto de conexión privado se conectan al espacio de nombres de Service Bus mediante una dirección IP privada y que tienen la integración correcta de la zona DNS privada.

En primer lugar, cree una máquina virtual siguiendo los pasos que encontrará en [Creación de una máquina virtual Windows en Azure Portal](../virtual-machines/windows/quick-create-portal.md).

Haga clic en la pestaña **Redes**: 

1. Especifique **Red virtual** y **Subred**. Debe seleccionar la instancia de Virtual Network en la que implementó el punto de conexión privado.
2. Especifique un recurso de **dirección IP pública**.
3. En **Grupo de seguridad de red de NIC**, seleccione **Ninguno**.
4. En **Equilibrio de carga**, seleccione **No**.

Conéctese a la máquina virtual, abra la línea de comandos y ejecute el siguiente comando:

```console
nslookup <service-bus-namespace-name>.servicebus.windows.net
```

Debería ver un resultado con el siguiente aspecto. 

```console
Non-authoritative answer:
Name:    <service-bus-namespace-name>.privatelink.servicebus.windows.net
Address:  10.0.0.4 (private IP address associated with the private endpoint)
Aliases:  <service-bus-namespace-name>.servicebus.windows.net
```

## <a name="limitations-and-design-considerations"></a>Limitaciones y consideraciones de diseño

**Precios**: Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).

**Limitaciones**:  Esta característica está disponible en todas las regiones públicas de Azure.

**Número máximo de puntos de conexión privados por espacio de nombres de Service Bus**: 120.

Para más información, consulte [Servicio Azure Private Link: Limitaciones](../private-link/private-link-service-overview.md#limitations)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Azure Private Link](../private-link/private-link-service-overview.md).
- Más información sobre [Azure Service Bus](service-bus-messaging-overview.md)
