---
title: Configuración de redes virtuales y firewalls de Azure Storage | Microsoft Docs
description: Configure la seguridad de red por capas para la cuenta de almacenamiento mediante los firewalls de Azure Storage y Azure Virtual Network.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/16/2020
ms.author: tamram
ms.reviewer: santoshc
ms.subservice: common
ms.openlocfilehash: 3ec4e6ee49052657210fffa8976b661c1a9e5948
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88827467"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>Configuración de redes virtuales y firewalls de Azure Storage

Azure Storage proporciona un modelo de seguridad por capas. Este modelo le permite proteger y controlar el nivel de acceso a las cuentas de almacenamiento que exigen sus aplicaciones y entornos empresariales, en función del tipo y el subconjunto de redes usadas. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a una cuenta de almacenamiento. Puede limitar el acceso a su cuenta de almacenamiento a las solicitudes procedentes de direcciones IP especificadas, intervalos IP o una lista de subredes de instancias de Azure Virtual Network (VNet).

Las cuentas de almacenamiento tienen un punto de conexión público accesible a través de Internet. También puede crear [puntos de conexión privados para la cuenta de almacenamiento](storage-private-endpoints.md), lo que asigna una dirección IP privada de la red virtual a la cuenta de almacenamiento, y protege todo el tráfico entre la red virtual y la cuenta de almacenamiento a través de un vínculo privado. El firewall de almacenamiento de Azure proporciona acceso de control de acceso para el punto de conexión público de la cuenta de almacenamiento. También puede usar el firewall para bloquear todo el acceso a través del punto de conexión público al usar puntos de conexión privados. La configuración del firewall de almacenamiento también permite seleccionar servicios de la plataforma de Azure de confianza para acceder a la cuenta de almacenamiento de forma segura.

Una aplicación que accede a una cuenta de almacenamiento cuando las reglas de red están en vigor aún requiere la autorización adecuada para la solicitud. La autorización es compatible con las credenciales de Azure Active Directory (Azure AD) (para blobs y colas), con una clave de acceso de cuenta válida o un token de SAS.

> [!IMPORTANT]
> La activación de las reglas de firewall para la cuenta de almacenamiento bloquea las solicitudes entrantes para los datos de forma predeterminada, a menos que las solicitudes procedan de un servicio que funcione en una instancia de Azure Virtual Network (VNet) o desde direcciones IP públicas permitidas. Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc.
>
> Puede conceder acceso a los servicios de Azure que funcionan desde una VNet al permitir el tráfico de la subred que hospeda la instancia de servicio. Puede habilitar también un número limitado de escenarios mediante el mecanismo [Excepciones](#exceptions) que se describe más adelante. Para acceder a los datos de la cuenta de almacenamiento mediante Azure Portal, deberá estar en una máquina situada dentro del límite de confianza (IP o red virtual) que haya configurado.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Escenarios

Para proteger la cuenta de almacenamiento, primero debe configurar una regla para denegar el acceso al tráfico desde todas las redes (incluido el tráfico de Internet) en el punto de conexión público de forma predeterminada. A continuación, debe configurar las reglas que conceden acceso al tráfico desde redes virtuales específicas. También puede configurar reglas para conceder acceso al tráfico desde intervalos de direcciones IP de Internet públicos, lo que permite habilitar conexiones desde clientes locales o específicos de Internet. Esta configuración le permite crear un límite de red seguro para las aplicaciones.

Puede combinar reglas de firewall que permitan el acceso desde redes virtuales específicas y desde intervalos de direcciones IP públicas en la misma cuenta de almacenamiento. Las reglas de firewall de almacenamiento se pueden aplicar a cuentas de almacenamiento existentes o al crear nuevas cuentas de almacenamiento.

Las reglas de firewall de almacenamiento se aplican al punto de conexión público de una cuenta de almacenamiento. No se necesitan reglas de acceso de firewall para permitir el tráfico de los puntos de conexión privados de una cuenta de almacenamiento. El proceso de aprobación de la creación de un punto de conexión privado concede acceso implícito al tráfico desde la subred que hospeda el punto de conexión privado.

Se aplican reglas de red en todos los protocolos de red para Azure Storage, incluidos REST y SMB. Para tener acceso a los datos mediante herramientas como Azure Portal, el Explorador de Storage y AZCopy, deben configurarse reglas de red explícitas.

Una vez que se apliquen las reglas de red, se aplicarán a todas las solicitudes. Los tokens de SAS que conceden acceso a una dirección IP específica sirven para limitar el acceso del titular del token, pero no conceden un acceso nuevo más allá de las reglas de red configuradas.

El tráfico de disco de máquina virtual (incluidas las operaciones de montaje y desmontaje y las operaciones de E/S de disco) no se ve afectado por las reglas de red. El acceso de REST a los blobs en páginas está protegido por las reglas de red.

Las cuentas de almacenamiento clásicas no admiten firewalls y redes virtuales.

Puede usar discos no administrados en cuentas de almacenamiento con reglas de red aplicadas para crear copias de seguridad y restaurar máquinas virtuales mediante la creación de una excepción. Este proceso se documenta en la sección [Excepciones](#exceptions) de este artículo. Las excepciones del firewall no se aplican a los discos administrados, puesto que ya son administrados por Azure.

## <a name="change-the-default-network-access-rule"></a>Modificación de la regla de acceso de red predeterminada

De forma predeterminada, las cuentas de almacenamiento aceptan conexiones de clientes en cualquier red. Para limitar el acceso a redes seleccionadas, primero debe cambiar la acción predeterminada.

> [!WARNING]
> La realización de cambios en reglas de red puede afectar a la capacidad de las aplicaciones de conexión a Azure Storage. Si se establece la regla de red predeterminada en **denegar**, se bloquea el acceso a los datos, a no ser que se apliquen también las reglas de red específicas para **conceder** acceso. Asegúrese de conceder acceso a las redes permitidas con reglas de red antes de cambiar la regla predeterminada para denegar el acceso.

### <a name="managing-default-network-access-rules"></a>Administración de las reglas de acceso de red predeterminadas

Puede administrar las reglas predeterminadas de acceso a redes para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="azure-portal"></a>Azure portal

1. Vaya a la cuenta de almacenamiento que quiere proteger.

1. Haga clic en el menú de configuración denominado **Firewalls y redes virtuales**.

1. Para denegar el acceso de forma predeterminada, elija permitir el acceso desde **Redes seleccionadas**. Para permitir el tráfico desde todas las redes, elija permitir el acceso desde **Todas las redes**.

1. Haga clic en **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>PowerShell

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

1. Visualice el estado de la regla predeterminada para la cuenta de almacenamiento.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").DefaultAction
    ```

1. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Deny
    ```

1. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Allow
    ```

#### <a name="cliv2"></a>CLIv2

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

1. Visualice el estado de la regla predeterminada para la cuenta de almacenamiento.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.defaultAction
    ```

1. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Deny
    ```

1. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Allow
    ```

## <a name="grant-access-from-a-virtual-network"></a>Concesión de acceso desde una red virtual

Puede configurar las cuentas de almacenamiento para permitir el acceso solo desde subredes específicas. Las subredes permitidas pueden pertenecer a una red virtual de la misma suscripción o a las de una suscripción diferente, incluidas las suscripciones que pertenecen a un inquilino de Azure Active Directory diferente.

Habilite un [punto de conexión de servicio](/azure/virtual-network/virtual-network-service-endpoints-overview) para Azure Storage dentro de la red virtual. El punto de conexión de servicio enruta el tráfico desde la red virtual a través de una ruta de acceso óptima al servicio Azure Storage. Las identidades de la red virtual y la subred también se transmiten con cada solicitud. Luego, los administradores pueden configurar reglas de red para la cuenta de almacenamiento que permitan que se reciban solicitudes desde subredes específicas en una red virtual. Los clientes a los que se concedió acceso a través de estas reglas de red deben seguir cumpliendo los requisitos de autorización de la cuenta de almacenamiento para acceder a los datos.

Cada cuenta de almacenamiento admite un máximo de 200 reglas de red virtual, que se pueden combinar con [reglas de red IP](#grant-access-from-an-internet-ip-range).

### <a name="available-virtual-network-regions"></a>Regiones de red virtual disponibles

En general, los puntos de conexión de servicio funcionan entre redes virtuales e instancias de servicio en la misma región de Azure. Cuando se usan los puntos de conexión de servicio con Azure Storage, este ámbito crece para incluir la [región emparejada](/azure/best-practices-availability-paired-regions). Los puntos de conexión de servicio permiten la continuidad durante una conmutación por error regional, así como un acceso a las instancias de almacenamiento con redundancia geográfica de solo lectura (RA-GRS). Las reglas de red que conceden acceso de una red virtual a una cuenta de almacenamiento también conceden acceso a cualquier instancia de RA-GRS.

Al planear la recuperación ante desastres durante una interrupción regional, debe crear las redes virtuales en la región emparejada por adelantado. Habilite puntos de conexión de servicio para Azure Storage, con reglas de red que concedan acceso desde estas redes virtuales alternativas. A continuación, aplique estas reglas a las cuentas de almacenamiento con redundancia geográfica.

> [!NOTE]
> Los puntos de conexión de servicio no se aplican al tráfico fuera de la región de la red virtual y el par de región designado. Solo puede aplicar reglas de red que concedan acceso desde redes virtuales a las cuentas de almacenamiento en la región principal de una cuenta de almacenamiento o en la región emparejada designada.

### <a name="required-permissions"></a>Permisos necesarios

Para aplicar una regla de red virtual a una cuenta de almacenamiento, el usuario debe tener permisos apropiados para las subredes que se van a agregar. El permiso necesario es *Join Service to a Subnet* (Unir el servicio a una subred) y se incluye en el rol integrado *Colaborador de la cuenta de almacenamiento*. También se puede agregar a definiciones de roles personalizados.

La cuenta de almacenamiento y las redes virtuales a las que se concedió acceso pueden estar en distintas suscripciones, incluidas suscripciones que formen parte del un inquilino de Azure AD diferente.

> [!NOTE]
> La configuración de reglas que conceden acceso a subredes en redes virtuales que forman parte de un inquilino de Azure Active Directory diferente solo se admiten actualmente a través de PowerShell, la CLI y las API REST. Estas reglas no se pueden configurar mediante Azure Portal, aunque se puedan ver en el portal.

### <a name="managing-virtual-network-rules"></a>Administración de reglas de red virtual

Puede administrar las reglas de red virtual para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="azure-portal"></a>Azure portal

1. Vaya a la cuenta de almacenamiento que quiere proteger.

1. Haga clic en el menú de configuración denominado **Firewalls y redes virtuales**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. Para conceder acceso a una red virtual con una nueva regla de red, en **Redes virtuales**, haga clic en **Agregar red virtual existente**, seleccione las opciones **Redes virtuales** y **Subredes** y, luego, haga clic en **Agregar**. Para crear una nueva red virtual y concederle acceso, haga clic en **Agregar nueva red virtual**. Proporcione la información necesaria para crear la nueva red virtual y, luego, haga clic en **Crear**.

    > [!NOTE]
    > Si un punto de conexión de servicio para Azure Storage no se ha configurado previamente para la red virtual y las subredes seleccionadas, se puede configurar como parte de esta operación.
    >
    > Actualmente, solo se muestran las redes virtuales que pertenecen al mismo inquilino de Azure Active Directory para su selección durante la creación de la regla. Para conceder acceso a una subred de una red virtual que pertenece a otro inquilino, use PowerShell, la CLI o la API REST.

1. Para quitar una regla de red virtual o subred, haga clic en **...** para abrir el menú contextual de la red virtual o la subred y haga clic en **Quitar**.

1. Haga clic en **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>PowerShell

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

1. Enumere las reglas de red virtual.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").VirtualNetworkRules
    ```

1. Habilite el punto de conexión de servicio para Azure Storage en una red virtual y subred existentes.

    ```powershell
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.0.0.0/24" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    ```

1. Agregue una regla de red para una red virtual y subred.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

    > [!TIP]
    > Para agregar una regla de red para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un parámetro completo **VirtualNetworkResourceId** con el formato "/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name".

1. Quite una regla de red para una red virtual y subred.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

#### <a name="cliv2"></a>CLIv2

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

1. Enumere las reglas de red virtual.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query virtualNetworkRules
    ```

1. Habilite el punto de conexión de servicio para Azure Storage en una red virtual y subred existentes.

    ```azurecli
    az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.Storage"
    ```

1. Agregue una regla de red para una red virtual y subred.

    ```azurecli
    $subnetid=(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

    > [!TIP]
    > Para agregar una regla para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un identificador de subred completo con el formato "/subscriptions/\<subscription-ID\>/resourceGroups/\<resourceGroup-Name\>/providers/Microsoft.Network/virtualNetworks/\<vNet-name\>/subnets/\<subnet-name\>".
    >
    > Puede usar el parámetro **subscription** para recuperar el identificador de subred de una red virtual que pertenezca a otro inquilino de Azure AD.

1. Quite una regla de red para una red virtual y subred.

    ```azurecli
    $subnetid=(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

## <a name="grant-access-from-an-internet-ip-range"></a>Concesión de acceso desde un intervalo IP de Internet

Puede configurar las cuentas de almacenamiento para permitir el acceso desde intervalos específicos de direcciones IP de Internet público. Esta configuración concede acceso a los servicios específicos basados en Internet y redes locales, y bloquea el tráfico de Internet general.

Proporcione los intervalos de dirección de Internet mediante la [notación CIDR](https://tools.ietf.org/html/rfc4632) en el formulario *16.17.18.0/24* o como direcciones IP individuales como *16.17.18.19*.

   > [!NOTE]
   > Los intervalos de dirección pequeños con tamaños de prefijos "/31" o "/32" no son compatibles. Estos intervalos se deberían configurar utilizando reglas de direcciones IP individuales.

Las reglas de red IP solo se permiten para direcciones IP de **Internet público**. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)) en las reglas de IP. Las redes privadas incluyen direcciones que comienzan por _10.*_ , _172.16.*_  - _172.31.*_ y _192.168.*_ .

   > [!NOTE]
   > Las reglas de red IP no tienen ningún efecto en las solicitudes que proceden de la misma región de Azure que la cuenta de almacenamiento. Use [reglas de red virtual](#grant-access-from-a-virtual-network) para permitir solicitudes de la misma región.

  > [!NOTE]
  > Los servicios implementados en la misma región que la cuenta de almacenamiento usan direcciones IP privadas de Azure para la comunicación. Por lo tanto, no puede restringir el acceso a servicios específicos de Azure en función de su intervalo de direcciones IP salientes públicas.

Solo se admiten direcciones IPV4 para la configuración de reglas de firewall de almacenamiento.

Cada cuenta de almacenamiento admite hasta 100 reglas de red IP.

### <a name="configuring-access-from-on-premises-networks"></a>Configuración del acceso desde redes locales

Para conceder acceso desde las redes locales a la cuenta de almacenamiento con una regla de red IP, debe identificar las direcciones IP orientadas a Internet que usa su red. Para obtener ayuda, póngase en contacto con el administrador de red.

Si usa [ExpressRoute](/azure/expressroute/expressroute-introduction) desde las instalaciones para pares públicos o el emparejamiento de Microsoft, tiene que identificar las direcciones IP de NAT que se usan. Para el emparejamiento público, cada circuito ExpressRoute usa de forma predeterminada dos direcciones IP de NAT que se aplican al tráfico del servicio de Azure cuando el tráfico entra en la red troncal de Microsoft Azure. Para el emparejamiento de Microsoft, el cliente o el proveedor de servicios proporciona las direcciones IP de NAT que se utilizan. Para permitir el acceso a los recursos de servicio, tiene que permitir estas direcciones IP públicas en la configuración del firewall de IP de recursos. Para encontrar las direcciones de IP de circuito de ExpressRoute de los pares públicos, [abra una incidencia de soporte técnico con ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) a través de Azure Portal. Obtenga más información sobre [NAT para ExpressRoute para emparejamiento público y de Microsoft.](/azure/expressroute/expressroute-nat#nat-requirements-for-azure-public-peering)

### <a name="managing-ip-network-rules"></a>Administración de reglas de red IP

Puede administrar las reglas de red IP para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="azure-portal"></a>Azure portal

1. Vaya a la cuenta de almacenamiento que quiere proteger.

1. Haga clic en el menú de configuración denominado **Firewalls y redes virtuales**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. Para conceder acceso al intervalo IP de Internet, escriba la dirección IP o el intervalo de direcciones (en formato CIDR) en **Firewall** > **Intervalo de direcciones**.

1. Para quitar una regla de red IP, haga clic en el icono de la Papelera junto al intervalo de direcciones.

1. Haga clic en **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>PowerShell

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

1. Enumere las reglas de red IP.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").IPRules
    ```

1. Agregue una regla de red para una dirección IP individual.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

1. Agregue una regla de red para un intervalo de direcciones IP.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

1. Quite una regla de red para una dirección IP individual.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

1. Quite una regla de red para un intervalo de direcciones IP.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

#### <a name="cliv2"></a>CLIv2

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

1. Enumere las reglas de red IP.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query ipRules
    ```

1. Agregue una regla de red para una dirección IP individual.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

1. Agregue una regla de red para un intervalo de direcciones IP.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

1. Quite una regla de red para una dirección IP individual.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

1. Quite una regla de red para un intervalo de direcciones IP.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

## <a name="exceptions"></a>Excepciones

Las reglas de red ayudan a crear un entorno seguro para las conexiones entre las aplicaciones y los datos para la mayoría de los escenarios. Sin embargo, algunas aplicaciones dependen de servicios de Azure que no se pueden aislar de forma exclusiva a través de las reglas de red virtual o de dirección IP. Pero estos servicios se deben conceder al almacenamiento para habilitar toda la funcionalidad de la aplicación. En tales situaciones, puede usar la opción ***Permitir servicios de Microsoft de confianza…*** para habilitar el acceso de dichos servicios a los datos, registros o análisis.

### <a name="trusted-microsoft-services"></a>Servicios de Microsoft de confianza

Algunos servicios de Microsoft funcionan desde redes que no se pueden incluir en las reglas de red. Puede conceder a un subconjunto de estos servicios de Microsoft de confianza el acceso a la cuenta de almacenamiento, a la vez que mantiene las reglas de red para otras aplicaciones. Estos servicios de confianza usarán una autenticación sólida para conectarse a su cuenta de almacenamiento de forma segura. Hemos habilitado dos modos de acceso de confianza para los servicios de Microsoft.

- Los recursos de algunos servicios, **cuando están registrados en su suscripción**, pueden acceder a su cuenta de almacenamiento **de la misma suscripción** para ciertas operaciones, como la escritura de registros o la realización de copias de seguridad.
- Los recursos de algunos servicios pueden conceder acceso explícito a su cuenta de almacenamiento. Para ello, **asignan un rol de Azure** a su identidad administrada asignada por el sistema.


Al habilitar la opción **Allow trusted Microsoft services...** (Permitir servicios de Microsoft de confianza), se concede acceso a los recursos de los siguientes servicios registrados en la misma suscripción que la cuenta de almacenamiento para un conjunto limitado de operaciones, tal como se describe:

| Servicio                  | Nombre del proveedor de recursos     | Operaciones permitidas                 |
|:------------------------ |:-------------------------- |:---------------------------------- |
| Azure Backup             | Microsoft.RecoveryServices | Ejecute copias de seguridad y restauraciones de discos no administrados en máquinas virtuales de IAAS. (no se necesita para discos administrados). [Más información](/azure/backup/backup-introduction-to-azure-backup). |
| Azure Data Box           | Microsoft.DataBox          | Permite la importación de datos en Azure mediante Data Box. [Más información](/azure/databox/data-box-overview). |
| Azure DevTest Labs       | Microsoft.DevTestLab       | Creación de imagen personalizada e instalación de artefactos. [Más información](../../devtest-labs/devtest-lab-overview.md). |
| Azure Event Grid         | Microsoft.EventGrid        | Habilite la publicación de eventos de Blob Storage y permita que Event Grid publique en las colas de almacenamiento. Obtenga información sobre los [eventos de Blob Storage](/azure/event-grid/event-sources) y la [publicación en las colas](/azure/event-grid/event-handlers). |
| Azure Event Hubs         | Microsoft.EventHub         | Archivo de datos con Event Hubs Capture. [Más información](/azure/event-hubs/event-hubs-capture-overview). |
| Azure File Sync          | Microsoft.StorageSync      | Permite transformar el servidor de archivos local en una memoria caché para recursos compartidos de archivos de Azure. Permite la sincronización de varios sitios, la recuperación rápida ante desastres y la copia de seguridad en la nube. [Más información](../files/storage-sync-files-planning.md) |
| HDInsight de Azure          | Microsoft.HDInsight        | Aprovisione el contenido inicial del sistema de archivos predeterminado para un nuevo clúster de HDInsight. [Más información](/azure/hdinsight/hdinsight-hadoop-use-blob-storage). |
| Azure Import/Export      | Microsoft.ImportExport     | Permite la importación de datos en Azure y la exportación de datos desde Azure mediante el servicio de importación y exportación. [Más información](/azure/storage/common/storage-import-export-service).  |
| Azure Monitor            | Microsoft.Insights         | Permite escribir datos de supervisión en una cuenta de almacenamiento protegida, incluidos los registros de recursos, los registros de inicio de sesión y de auditoría de Azure Active Directory y los registros de Microsoft Intune. [Más información](/azure/monitoring-and-diagnostics/monitoring-roles-permissions-security). |
| Conexión a Azure         | Microsoft.Network          | Almacene y analice los registros de tráfico de red, incluidos los servicios de Network Watcher y Análisis de tráfico. [Más información](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview). |
| Azure Site Recovery      | Microsoft.SiteRecovery     | Habilite la replicación para la recuperación ante desastres de máquinas virtuales de IaaS de Azure al usar la caché habilitada para firewall, el origen o las cuentas de almacenamiento de destino.  [Más información](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-enable-replication). |

El valor **Permitir servicios de Microsoft de confianza...** también permite que una instancia concreta de los servicios siguientes acceda a la cuenta de almacenamiento si [asigna un rol de Azure ](storage-auth-aad.md#assign-azure-roles-for-access-rights) explícitamente a la [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/overview.md) de esa instancia de recurso. En ese caso, el ámbito de acceso de la instancia corresponde al rol de Azure que se asigna a la identidad administrada.

| Servicio                        | Nombre del proveedor de recursos                 | Propósito            |
| :----------------------------- | :------------------------------------- | :----------------- |
| Azure API Management           | Microsoft.ApiManagement/service        | Habilita el acceso del servicio API Management a las cuentas de almacenamiento detrás del firewall mediante directivas. [Más información](/azure/api-management/api-management-authentication-policies#use-managed-identity-in-send-request-policy). |
| Azure Cognitive Search         | Microsoft.Search/searchServices        | Habilita los servicios de Cognitive Search para acceder a las cuentas de almacenamiento con fines de indexación, proceso y consulta. |
| Tareas de Azure Container Registry | Microsoft.ContainerRegistry/registries | ACR Tasks puede acceder a las cuentas de almacenamiento al compilar imágenes de contenedor. |
| Azure Data Factory             | Microsoft.DataFactory/factories        | Permite el acceso a las cuentas de almacenamiento a través del tiempo de ejecución de ADF. |
| Azure Data Share               | Microsoft.DataShare/accounts           | Permite el acceso a las cuentas de almacenamiento a través de Data Share. |
| Azure IoT Hub                  | Microsoft.Devices/IotHubs              | Permite que los datos de un centro de IoT se escriban en almacenamiento de blobs. [Más información](../../iot-hub/virtual-network-support.md#egress-connectivity-to-storage-account-endpoints-for-routing) |
| Azure Logic Apps               | Microsoft.Logic/workflows              | Permite a las aplicaciones lógicas acceder a las cuentas de almacenamiento. [Más información](/azure/logic-apps/create-managed-service-identity#authenticate-access-with-managed-identity). |
| Servicio Azure Machine Learning | Microsoft.MachineLearningServices      | Las áreas de trabajo autorizadas de Azure Machine Learning escriben los resultados del experimento, los modelos y los registros en Blob Storage y leen los datos. [Más información](/azure/machine-learning/how-to-enable-virtual-network#use-a-storage-account-for-your-workspace). | 
| Azure Synapse Analytics (anteriormente SQL Data Warehouse)       | Microsoft.Sql                          | Permite importar y exportar los datos de instancias de SQL Database específicas mediante la instrucción COPY o con PolyBase. [Más información](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview). |
| Azure SQL Database       | Microsoft.Sql                          | Permite [importar](https://docs.microsoft.com/sql/t-sql/statements/bulk-insert-transact-sql?view=sql-server-ver15#f-importing-data-from-a-file-in-azure-blob-storage) de datos de cuentas de almacenamiento y [escribir datos de auditoría de](https://docs.microsoft.com/azure/azure-sql/database/audit-write-storage-account-behind-vnet-firewall) en cuentas de almacenamiento detrás del firewall. |
| Azure Stream Analytics         | Microsoft.StreamAnalytics             | Permite que los datos de un trabajo de streaming se escriban en Blob Storage. Esta funcionalidad actualmente está en su versión preliminar. [Más información](/azure/stream-analytics/blob-output-managed-identity). |
| Azure Synapse Analytics        | Microsoft.Synapse/workspaces          | Permite el acceso a los datos de Azure Storage desde Synapse Analytics. |


### <a name="storage-analytics-data-access"></a>Acceso a datos de análisis de almacenamiento

En algunos casos, se requiere acceso para leer registros recursos y métricas desde fuera del límite de red. Al configurar el acceso de los servicios de confianza a la cuenta de almacenamiento, puede permitir el acceso de lectura a los archivos de registro, las tablas de métricas o ambos. [Obtenga más información sobre cómo trabajar con analíticas de almacenamiento](/azure/storage/storage-analytics).

### <a name="managing-exceptions"></a>Administración de excepciones

Puede administrar las excepciones de reglas de red a través de Azure Portal, PowerShell o la CLI de Azure v2.

#### <a name="azure-portal"></a>Azure portal

1. Vaya a la cuenta de almacenamiento que quiere proteger.

1. Haga clic en el menú de configuración denominado **Firewalls y redes virtuales**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. En **Excepciones**, seleccione las excepciones que quiere conceder.

1. Haga clic en **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>PowerShell

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

1. Vea las excepciones para las reglas de red de la cuenta de almacenamiento.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount").Bypass
    ```

1. Configure las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass AzureServices,Metrics,Logging
    ```

1. Quite las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass None
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, la eliminación de las excepciones no tendrá ningún efecto.

#### <a name="cliv2"></a>CLIv2

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

1. Vea las excepciones para las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.bypass
    ```

1. Configure las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass Logging Metrics AzureServices
    ```

1. Quite las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass None
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, la eliminación de las excepciones no tendrá ningún efecto.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de los puntos de conexión de servicio de red de Azure en [Puntos de conexión de servicio](/azure/virtual-network/virtual-network-service-endpoints-overview).

Profundice en la seguridad de Azure Storage en la [Guía de seguridad de Azure Storage](../blobs/security-recommendations.md).
