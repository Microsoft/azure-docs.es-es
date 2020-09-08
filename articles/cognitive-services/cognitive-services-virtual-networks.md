---
title: Virtual Networks
titleSuffix: Azure Cognitive Services
description: Configure la seguridad de red por niveles para sus recursos de Cognitive Services.
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: dapine
ms.openlocfilehash: 808d42c821272882bbf0e01a36e49f7f10b30efa
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88505034"
---
# <a name="configure-azure-cognitive-services-virtual-networks"></a>Configuración de redes virtuales de Azure Cognitive Services

Azure Cognitive Services proporciona un modelo de seguridad por niveles. Este modelo le permite proteger las cuentas de Cognitive Services en un subconjunto específico de redes. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a la cuenta. Puede limitar el acceso a sus recursos con el filtrado de solicitudes. Puede permitir solo las solicitudes procedentes de direcciones IP especificadas, intervalos de IP o una lista de subredes de instancias de [Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

Una aplicación que accede a un recurso de Cognitive Services cuando las reglas de red están en vigor requiere autorización. La autorización es compatible con las credenciales de [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) o con una clave de API válida.

> [!IMPORTANT]
> Al activar las reglas de firewall para su cuenta de Cognitive Services, se bloquean las solicitudes de datos entrantes de forma predeterminada. Para permitir las solicitudes, se debe cumplir una de las siguientes condiciones:

> * La solicitud debe originarse en un servicio que funcione en una instancia de Azure Virtual Network (VNet) en la lista de subredes permitidas de la cuenta de Cognitive Services de destino. El punto de conexión en las solicitudes originadas en la red virtual debe establecerse como el [subdominio personalizado](cognitive-services-custom-subdomains.md) de la cuenta de Cognitive Services.
> * O bien, la solicitud debe originarse en una lista de direcciones IP permitidas.
>
> Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Escenarios

Para proteger el recurso de Cognitive Services, primero debe configurar una regla para denegar el acceso al tráfico desde todas las redes (incluido el tráfico de Internet) de forma predeterminada. A continuación, debe configurar las reglas que conceden acceso al tráfico desde redes virtuales específicas. Esta configuración le permite crear un límite de red seguro para las aplicaciones. También puede configurar reglas para conceder acceso al tráfico desde intervalos de direcciones IP de Internet públicos, lo que permite habilitar conexiones desde clientes locales o específicos de Internet.

Se aplican reglas de red en todos los protocolos de red para Azure Cognitive Services, incluidos REST y WebSocket. Para tener acceso a los datos mediante herramientas como las consolas de prueba de Azure, deben configurarse reglas de red explícitas. Puede aplicar reglas de red a recursos de Cognitive Services existentes o al crear nuevos recursos de Cognitive Services. Una vez que se apliquen las reglas de red, se aplicarán a todas las solicitudes.

## <a name="supported-regions-and-service-offerings"></a>Ofertas de servicio y regiones admitidas

Se admiten redes virtuales en [regiones en las que Cognitive Services está disponible](https://azure.microsoft.com/global-infrastructure/services/). Si Cognitive Services no aparece en la lista, actualmente no admite redes virtuales.

> [!div class="checklist"]
> * [Anomaly Detector](./anomaly-detector/index.yml)
> * [Computer Vision](./computer-vision/index.yml)
> * [Content Moderator](./content-moderator/index.yml)
> * [Custom Vision](./custom-vision-service/index.yml)
> * [Face](./face/index.yml)
> * [Form Recognizer](./form-recognizer/index.yml)
> * [Language Understanding](./luis/index.yml)
> * [Personalizer](./personalizer/index.yml)
> * [Text Analytics](./text-analytics/index.yml)
> * [QnA Maker](./qnamaker/index.yml)
> * [Translator Text](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#virtual-network-support)
> * [Lector inmersivo](./immersive-reader/index.yml)

## <a name="service-tags"></a>Etiquetas de servicio

Cognitive Services admite etiquetas de servicio para la configuración de reglas de red. Los servicios de la lista siguiente se incluyen en la etiqueta de servicio **CognitiveServicesManagement**.

> [!div class="checklist"]
> * [Anomaly Detector](./anomaly-detector/index.yml)
> * [Computer Vision](./computer-vision/index.yml)
> * [Content Moderator](./content-moderator/index.yml)
> * [Custom Vision](./custom-vision-service/index.yml)
> * [Face](./face/index.yml)
> * [Form Recognizer](./form-recognizer/index.yml)
> * [Language Understanding (LUIS)](./luis/index.yml)
> * [Personalizer](./personalizer/index.yml)
> * [Text Analytics](./text-analytics/index.yml)
> * [QnA Maker](./qnamaker/index.yml)
> * [Traductor](./translator/index.yml)
> * [Servicio Voz](./speech-service/index.yml)
> * [Lector inmersivo](./immersive-reader/index.yml)

## <a name="change-the-default-network-access-rule"></a>Modificación de la regla de acceso de red predeterminada

De forma predeterminada, los recursos de Cognitive Services aceptan conexiones de clientes en cualquier red. Para limitar el acceso a redes seleccionadas, primero debe cambiar la acción predeterminada.

> [!WARNING]
> La realización de cambios en reglas de red puede afectar a la capacidad de las aplicaciones de conexión a Azure Cognitive Services. Si se establece la regla de red predeterminada en **denegar**, se bloquea el acceso a los datos, a no ser que se apliquen también las reglas de red específicas para **conceder** acceso. Asegúrese de conceder acceso a las redes permitidas con reglas de red antes de cambiar la regla predeterminada para denegar el acceso. Si permite enumerar las direcciones IP de su red local, asegúrese de agregar todas las direcciones IP públicas salientes posibles de dicha red.

### <a name="managing-default-network-access-rules"></a>Administración de las reglas de acceso de red predeterminadas

Puede administrar las reglas predeterminadas de acceso a redes para los recursos de Cognitive Services a través de Azure Portal, PowerShell o la CLI de Azure.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Vaya al recurso de Cognitive Services que desea proteger.

1. Seleccione el menú **ADMINISTRACIÓN DE RECURSOS** de administración de recursos denominado **Red virtual**.

   ![Opción Red virtual](media/vnet/virtual-network-blade.png)

1. Para denegar el acceso de forma predeterminada, elija permitir el acceso desde **Redes seleccionadas**. Solo con la opción **Redes seleccionadas**, sin las opciones **Redes virtuales** ni **Intervalos de direcciones** configuradas, todo el acceso se deniega de forma efectiva. Cuando se deniega todo el acceso, no se permiten solicitudes que intenten consumir el recurso de Cognitive Services. Azure Portal, Azure PowerShell o la CLI de Azure se pueden seguir usando para configurar el recurso de Cognitive Services.
1. Para permitir el tráfico desde todas las redes, elija permitir el acceso desde **Todas las redes**.

   ![Denegación de redes virtuales](media/vnet/virtual-network-deny.png)

1. Seleccione **Guardar** para aplicar los cambios.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps), o bien seleccione **Probar**.

1. Visualice el estado de la regla predeterminada para el recurso de Cognitive Services.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).DefaultAction
    ```

1. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Deny
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

1. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Allow
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli), o bien seleccione **Probar**.

1. Visualice el estado de la regla predeterminada para el recurso de Cognitive Services.

    ```azurecli-interactive
    az cognitiveservices account show \
        -g "myresourcegroup" -n "myaccount" \
        --query networkRuleSet.defaultAction
    ```

1. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Deny
    ```

1. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Allow
    ```

***

## <a name="grant-access-from-a-virtual-network"></a>Concesión de acceso desde una red virtual

Puede configurar recursos de Cognitive Services para permitir el acceso solo desde subredes específicas. Las subredes permitidas pueden pertenecer a una red virtual de la misma suscripción o de una suscripción diferente, incluidas las suscripciones que pertenecen a un inquilino de Azure Active Directory diferente.

Habilite un [punto de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) para Azure Cognitive Services dentro de la red virtual. El punto de conexión de servicio enruta el tráfico desde la red virtual a través de una ruta de acceso óptima al servicio Azure Cognitive Services. Las identidades de la red virtual y la subred también se transmiten con cada solicitud. Luego, los administradores pueden configurar reglas de red para el recurso de Cognitive Services que permitan que se reciban solicitudes desde subredes específicas en una red virtual. Los clientes a los que se concedió acceso a través de estas reglas de red deben seguir cumpliendo los requisitos de autorización del recurso de Cognitive Services para acceder a los datos.

Cada recurso de Cognitive Services admite hasta 100 reglas de red virtual, que se pueden combinar con [reglas de red de IP](#grant-access-from-an-internet-ip-range).

### <a name="required-permissions"></a>Permisos necesarios

Para aplicar una regla de red virtual a un recurso de Cognitive Services, el usuario debe tener permisos apropiados para las subredes que se agregan. El permiso necesario es el rol *Colaborador* predeterminado, o bien el rol *Colaborador de Cognitive Services*. También se pueden agregar los permisos necesarios a las definiciones de roles personalizados.

El recurso de Cognitive Services y las redes virtuales a las que se concedió acceso pueden estar en distintas suscripciones, incluidas las suscripciones que forman parte de un inquilino de Azure AD diferente.

> [!NOTE]
> La configuración de reglas que conceden acceso a subredes en redes virtuales que forman parte de un inquilino de Azure Active Directory diferente solo se admiten actualmente a través de PowerShell, la CLI y las API REST. Estas reglas no se pueden configurar mediante Azure Portal, aunque se puedan ver en el portal.

### <a name="managing-virtual-network-rules"></a>Administración de reglas de red virtual

Puede administrar las reglas de red virtual para los recursos de Cognitive Services a través de Azure Portal, PowerShell o la CLI de Azure.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Vaya al recurso de Cognitive Services que desea proteger.

1. Seleccione el menú **ADMINISTRACIÓN DE RECURSOS** de administración de recursos denominado **Red virtual**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. Para conceder acceso a una red virtual con una regla de red existente, en **Redes virtuales**, seleccione  **Agregar red virtual existente**.

   ![Adición de una red virtual existente](media/vnet/virtual-network-add-existing.png)

1. Seleccione las opciones **Redes virtuales** y **Subredes** y, a continuación, haga clic en **Habilitar**.

   ![Adición de detalles de red virtual existente](media/vnet/virtual-network-add-existing-details.png)

1. Para crear una nueva red virtual y concederle acceso, seleccione **Agregar nueva red virtual**.

   ![Adición de una nueva red virtual](media/vnet/virtual-network-add-new.png)

1. Proporcione la información necesaria para crear la nueva red virtual y, luego, seleccione **Crear**.

   ![Creación de una red virtual](media/vnet/virtual-network-create.png)

    > [!NOTE]
    > Si un punto de conexión de servicio para Azure Cognitive Services no se ha configurado previamente para la red virtual y las subredes seleccionadas, se puede configurar como parte de esta operación.
    >
    > Actualmente, solo se muestran las redes virtuales que pertenecen al mismo inquilino de Azure Active Directory para su selección durante la creación de la regla. Para conceder acceso a una subred de una red virtual que pertenece a otro inquilino, use PowerShell, la CLI o la API REST.

1. Para quitar una regla de red virtual o subred, seleccione **...** para abrir el menú contextual de la red virtual o la subred y seleccione **Quitar**.

   ![Eliminación de una red virtual](media/vnet/virtual-network-remove.png)

1. Seleccione **Guardar** para aplicar los cambios.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps), o bien seleccione **Probar**.

1. Enumere las reglas de red virtual.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).VirtualNetworkRules
    ```

1. Habilite el punto de conexión de servicio para Azure Cognitive Services en una red virtual y una subred existentes.

    ```azurepowershell-interactive
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" `
        -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" `
        -AddressPrefix "10.0.0.0/24" `
        -ServiceEndpoint "Microsoft.CognitiveServices" | Set-AzVirtualNetwork
    ```

1. Agregue una regla de red para una red virtual y subred.

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

    > [!TIP]
    > Para agregar una regla de red para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un parámetro completo **VirtualNetworkResourceId** con el formato "/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name".

1. Quite una regla de red para una red virtual y subred.

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli), o bien seleccione **Probar**.

1. Enumere las reglas de red virtual.

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" \
        --query virtualNetworkRules
    ```

1. Habilite el punto de conexión de servicio para Azure Cognitive Services en una red virtual y una subred existentes.

    ```azurecli-interactive
    az network vnet subnet update -g "myresourcegroup" -n "mysubnet" \
    --vnet-name "myvnet" --service-endpoints "Microsoft.CognitiveServices"
    ```

1. Agregue una regla de red para una red virtual y subred.

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule addition
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

    > [!TIP]
    > Para agregar una regla para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un identificador de subred completo con el formato "/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name".
    > 
    > Puede usar el parámetro **subscription** para recuperar el identificador de subred de una red virtual que pertenezca a otro inquilino de Azure AD.

1. Quite una regla de red para una red virtual y subred.

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule removal
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

***

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

## <a name="grant-access-from-an-internet-ip-range"></a>Concesión de acceso desde un intervalo IP de Internet

Puede configurar recursos de Cognitive Services para permitir el acceso desde intervalos específicos de direcciones IP de Internet públicas. Esta configuración concede acceso a servicios específicos y redes locales, y bloquea el tráfico de Internet general de forma eficaz.

Proporcione intervalos de direcciones de Internet permitidos mediante la [notación CIDR](https://tools.ietf.org/html/rfc4632) en el formulario `16.17.18.0/24` o como direcciones IP individuales como `16.17.18.19`.

   > [!Tip]
   > Los intervalos de dirección pequeños con tamaños de prefijos "/31" o "/32" no son compatibles. Estos intervalos se deberían configurar utilizando reglas de direcciones IP individuales.

Las reglas de red IP solo se permiten para direcciones IP de **Internet público**. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)) en las reglas de IP. Las redes privadas incluyen direcciones que comienzan por `10.*`, `172.16.*` - `172.31.*` y `192.168.*`.

   > [!NOTE]
   > Las reglas de red IP no tienen ningún efecto en las solicitudes que proceden de la misma región de Azure que el recurso de Cognitive Services. Use [reglas de red virtual](#grant-access-from-a-virtual-network) para permitir solicitudes de la misma región.

Solo se admiten direcciones IPV4 en este momento. Cada recurso de Cognitive Services admite hasta 100 reglas de red IP, que se pueden combinar con [reglas de red virtual](#grant-access-from-a-virtual-network).

### <a name="configuring-access-from-on-premises-networks"></a>Configuración del acceso desde redes locales

Para conceder acceso desde las redes locales al recurso de Cognitive Services con una regla de red IP, debe identificar las direcciones IP orientadas a Internet que usa su red. Para obtener ayuda, póngase en contacto con el administrador de red.

Si usa [ExpressRoute](../expressroute/expressroute-introduction.md) localmente para el emparejamiento público o el emparejamiento de Microsoft, deberá identificar las direcciones IP de NAT. Para el emparejamiento público, cada circuito de ExpressRoute usa dos direcciones IP de NAT de forma predeterminada. Cada una de estas se aplica al tráfico del servicio de Azure cuando el tráfico entra en la red troncal de Microsoft Azure. Para el emparejamiento de Microsoft, las direcciones IP de NAT que se utilizan las proporciona el cliente o el proveedor de servicios. Para permitir el acceso a los recursos de servicio, tiene que permitir estas direcciones IP públicas en la configuración del firewall de IP de recursos. Para encontrar las direcciones de IP de circuito de ExpressRoute de los pares públicos, [abra una incidencia de soporte técnico con ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) a través de Azure Portal. Obtenga más información sobre [NAT para ExpressRoute para emparejamiento público y de Microsoft.](../expressroute/expressroute-nat.md#nat-requirements-for-azure-public-peering)

### <a name="managing-ip-network-rules"></a>Administración de reglas de red IP

Puede administrar las reglas de red IP para los recursos de Cognitive Services a través de Azure Portal, PowerShell o la CLI de Azure.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Vaya al recurso de Cognitive Services que desea proteger.

1. Seleccione el menú **ADMINISTRACIÓN DE RECURSOS** de administración de recursos denominado **Red virtual**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. Para conceder acceso al intervalo IP de Internet, escriba la dirección IP o el intervalo de direcciones (en [formato CIDR](https://tools.ietf.org/html/rfc4632)) en **Firewall** > **Intervalo de direcciones**. Solo se aceptan direcciones IP públicas válidas (no reservadas).

   ![Adición de un intervalo de IP](media/vnet/virtual-network-add-ip-range.png)

1. Para quitar una regla de red IP, haga clic en el icono de la Papelera <span class="docon docon-delete x-hidden-focus"></span> junto al intervalo de direcciones.

   ![Eliminación de intervalo de IP](media/vnet/virtual-network-delete-ip-range.png)

1. Seleccione **Guardar** para aplicar los cambios.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps), o bien seleccione **Probar**.

1. Enumere las reglas de red IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).IPRules
    ```

1. Agregue una regla de red para una dirección IP individual.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Agregue una regla de red para un intervalo de direcciones IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Quite una regla de red para una dirección IP individual.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Quite una regla de red para un intervalo de direcciones IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli), o bien seleccione **Probar**.

1. Enumere las reglas de red IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" --query ipRules
    ```

1. Agregue una regla de red para una dirección IP individual.

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. Agregue una regla de red para un intervalo de direcciones IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

1. Quite una regla de red para una dirección IP individual.

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. Quite una regla de red para un intervalo de direcciones IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

***

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

## <a name="use-private-endpoints"></a>Usar puntos de conexión privados

Puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) para los recursos de Cognitive Services para que los clientes de una red virtual (VNet) puedan acceder de forma segura a los datos a través de una instancia de [Private Link](../private-link/private-link-overview.md). El punto de conexión privado usa una dirección IP del espacio de direcciones para el recurso de Cognitive Services. El tráfico de red entre los clientes de la red virtual y el recurso atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet.

Los puntos de conexión privados para recursos de Cognitive Services le permiten:

* Proteger el recurso de Cognitive Services al configurar el firewall para bloquear todas las conexiones del punto de conexión público del servicio de Cognitive Services.
* Aumentar la seguridad de la red virtual, ya que permite bloquear la filtración de datos de la red virtual.
* Conectarse de forma segura a los recursos de Cognitive Services desde las redes locales que se conectan a la red virtual mediante [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../expressroute/expressroute-locations.md) con emparejamiento privado.

### <a name="conceptual-overview"></a>Información general conceptual

Un punto de conexión privado es una interfaz de red especial para un recurso de Azure de la [red virtual](../virtual-network/virtual-networks-overview.md). Al crear un punto de conexión privado para el recurso de Cognitive Services, este proporciona conectividad segura entre los clientes de la red virtual y el recurso. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de la red virtual. La conexión entre el punto de conexión privado y el servicio de Cognitive Services usa un vínculo privado seguro.

Las aplicaciones de la red virtual se pueden conectar al servicio a través del punto de conexión privado sin problemas, ya que se usan las mismas cadenas de conexión y mecanismos de autorización que se usarían en cualquier otro caso. La excepción es el servicio de Voz, que requiere un punto de conexión independiente. Consulte la sección [Puntos de conexión privados con el servicio de Voz](#private-endpoints-with-the-speech-service). Los puntos de conexión privados se pueden usar con todos los protocolos que admita el recurso de Cognitive Services, incluidos REST y SMB.

Los puntos de conexión privados se pueden crear en subredes que usan [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md). Los clientes de una subred pueden conectarse a un recurso de Cognitive Services mediante un punto de conexión privado, al mismo tiempo que usan puntos de conexión de servicio para acceder a otros.

Cuando se crea un punto de conexión privado para un recurso de Cognitive Services en la red virtual, se envía una solicitud de consentimiento para su aprobación al propietario del recurso de Cognitive Services. Si el usuario que solicita la creación del punto de conexión privado también es propietario del recurso, esta solicitud de consentimiento se aprueba automáticamente.

Los propietarios de recursos de Cognitive Services pueden administrar las solicitudes de consentimiento y los puntos de conexión privados desde la pestaña *Puntos de conexión privados* del recurso de Cognitive Services en [Azure Portal](https://portal.azure.com).

### <a name="private-endpoints"></a>Puntos de conexión privados

Al crear el punto de conexión privado, debe especificar el recurso de Cognitive Services al que se conecta. Para más información acerca de la creación de un punto de conexión privado, consulte:

* [Creación de un punto de conexión privado mediante Private Link Center en Azure Portal](../private-link/create-private-endpoint-portal.md)
* [Creación de un punto de conexión privado mediante la CLI de Azure](../private-link/create-private-endpoint-cli.md)
* [Creación de un punto de conexión privado mediante Azure PowerShell](../private-link/create-private-endpoint-powershell.md)

### <a name="connecting-to-private-endpoints"></a>Conexión a puntos de conexión privados

Los clientes de una red virtual que usan el punto de conexión privado deben usar la misma cadena de conexión para el recurso de Cognitive Services que aquellos clientes que se conectan mediante el punto de conexión público. La excepción es el servicio de voz, que requiere un punto de conexión independiente. Consulte la sección [Puntos de conexión privados con el servicio de voz](#private-endpoints-with-the-speech-service). Confiamos en la resolución del DNS para enrutar automáticamente las conexiones desde la red virtual al recurso de Cognitive Services a través de un vínculo privado. El servicio de voz 

De forma predeterminada, se crea una [zona DNS privada](../dns/private-dns-overview.md) conectada a la red virtual con las actualizaciones necesarias para los puntos de conexión privados. Sin embargo, si usa su propio servidor DNS, puede que tenga que realizar cambios adicionales en la configuración de DNS. En la sección [Cambios de DNS](#dns-changes-for-private-endpoints) que aparece a continuación se describen las actualizaciones necesarias para los puntos de conexión privados.

### <a name="private-endpoints-with-the-speech-service"></a>Puntos de conexión privados con el servicio de voz

Cuando use puntos de conexión privados con el servicio de voz, debe usar un punto de conexión personalizado para llamar al servicio de voz. No se puede usar el punto de conexión global. El punto de conexión debe seguir este patrón: `{account}.{stt|tts|voice|dls}.speech.microsoft.com`.

### <a name="dns-changes-for-private-endpoints"></a>Cambios de DNS en puntos de conexión privados

Al crear un punto de conexión privado, el registro del recurso CNAME de DNS del recurso de Cognitive Services se actualiza a un alias de un subdominio con el prefijo "*privatelink*". De forma predeterminada, también se crea una [zona DNS privada](../dns/private-dns-overview.md), que se corresponde con el subdominio "*privatelink*, con los registros de recursos A de DNS para los puntos de conexión privados.

Cuando se resuelve la dirección URL del punto de conexión desde fuera de la red virtual con el punto de conexión privado, se resuelve en el punto de conexión público del recurso de Cognitive Services. Cuando se resuelve desde la red virtual que hospeda el punto de conexión privado, la dirección URL del punto de conexión se resuelve en la dirección IP del punto de conexión privado.

Esta estrategia permite el acceso al recurso de Cognitive Services mediante la misma cadena de conexión para los clientes de la red virtual que hospeda los puntos de conexión privados y los clientes que están fuera de esta.

Si va a usar un servidor DNS personalizado en la red, los clientes deben ser capaces de resolver el nombre de dominio completo (FQDN) del punto de conexión del recurso de Cognitive Services en la dirección IP del punto de conexión privado. Configure el servidor DNS para delegar el subdominio del vínculo privado en la zona DNS privada de la red virtual.

> [!TIP]
> Cuando use un servidor DNS personalizado o local, debe configurarlo para resolver el nombre del recurso de Cognitive Services del subdominio "privatelink" en la dirección IP del punto de conexión privado. Para ello, puede delegar el subdominio "privatelink" en la zona DNS privada de la red virtual, o bien configurar la zona DNS en el servidor DNS y agregar los registros D de DNS.

Para más información sobre cómo configurar su propio servidor DNS para que admita puntos de conexión privados, consulte los siguientes artículos:

* [Resolución de nombres de recursos en redes virtuales de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-that-uses-your-own-dns-server)
* [Configuración de DNS para puntos de conexión privados](https://docs.microsoft.com/azure/private-link/private-endpoint-overview#dns-configuration)

### <a name="pricing"></a>Precios

Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="next-steps"></a>Pasos siguientes

* Exploración de los distintos servicios de [Azure Cognitive Services](welcome.md).
* Más información sobre los [puntos de conexión de servicio de Azure Virtual Network](../virtual-network/virtual-network-service-endpoints-overview.md).
