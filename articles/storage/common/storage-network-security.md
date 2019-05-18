---
title: Configuración de redes virtuales y firewalls de Azure Storage | Microsoft Docs
description: Configure la seguridad de red por niveles para la cuenta de almacenamiento.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 56a25c95d9bc01078b3eff3729a8a693ee3cf510
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65520254"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>Configuración de redes virtuales y firewalls de Azure Storage

Azure Storage proporciona un modelo de seguridad por capas. Este modelo le permite proteger las cuentas de almacenamiento en un conjunto específico de redes admitidas. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a una cuenta de almacenamiento.

Una aplicación que accede a una cuenta de almacenamiento cuando las reglas de red están en vigor requiere la autorización adecuada en la solicitud. Autorización es compatible con las credenciales de Azure Active Directory (Azure AD) para blobs y colas, con una clave de acceso de cuenta válida o con un token de SAS.

> [!IMPORTANT]
> La activación de las reglas de firewall para la cuenta de almacenamiento bloquea las solicitudes entrantes para los datos de forma predeterminada, a menos que las solicitudes procedan de un servicio que funcione en una red virtual (VNet) de Azure. Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc.
>
> Puede conceder acceso a los servicios de Azure que funcionan desde una VNet al permitir la subred de la instancia de servicio. Habilite un número limitado de escenarios mediante el mecanismo [Excepciones](#exceptions) que se describe en la sección siguiente. Para acceder a Azure Portal, deberá estar en una máquina situada dentro del límite de confianza (IP o red virtual) que haya configurado.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Escenarios

Configure las cuentas de almacenamiento para denegar el acceso al tráfico desde todas las redes (incluido el tráfico de Internet) de forma predeterminada. A continuación, conceda acceso para el tráfico desde redes virtuales específicas. Esta configuración le permite crear un límite de red seguro para las aplicaciones. También puede conceder acceso a intervalos de direcciones IP de Internet públicos, lo que permite habilitar conexiones desde clientes locales o específicos de Internet.

Se aplican reglas de red en todos los protocolos de red para Azure Storage, incluidos REST y SMB. Para acceder a los datos con herramientas como Azure Portal, el Explorador de Storage y AZCopy, se requieren reglas de red explícitas.

Puede aplicar reglas de red a cuentas de almacenamiento existente o crear nuevas cuentas de almacenamiento.

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

#### <a name="azure-portal"></a>Azure Portal

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

Puede configurar las cuentas de almacenamiento para permitir el acceso solo desde redes virtuales específicas.

Habilite un [punto de conexión de servicio](/azure/virtual-network/virtual-network-service-endpoints-overview) para Azure Storage dentro de la red virtual. Este punto de conexión proporciona al tráfico una ruta óptima hasta el servicio de Azure Storage. Las identidades de la red virtual y la subred también se transmiten con cada solicitud. Luego, los administradores pueden configurar reglas de red para la cuenta de almacenamiento que permitan que se reciban solicitudes desde subredes específicas en la red virtual. Los clientes a los que se concedió acceso a través de estas reglas de red deben seguir cumpliendo los requisitos de autorización de la cuenta de almacenamiento para acceder a los datos.

Cada cuenta de almacenamiento admite hasta 100 reglas de red virtual, que se pueden combinar con [reglas de red de IP](#grant-access-from-an-internet-ip-range).

### <a name="available-virtual-network-regions"></a>Regiones de red virtual disponibles

En general, los puntos de conexión de servicio funcionan entre redes virtuales e instancias de servicio en la misma región de Azure. Cuando se usan los puntos de conexión de servicio con Azure Storage, este ámbito crece para incluir la [región emparejada](/azure/best-practices-availability-paired-regions). Los puntos de conexión de servicio permiten la continuidad durante una conmutación por error regional, así como un acceso a las instancias de almacenamiento con redundancia geográfica de solo lectura (RA-GRS). Las reglas de red que conceden acceso de una red virtual a una cuenta de almacenamiento también conceden acceso a cualquier instancia de RA-GRS.

Al planear la recuperación ante desastres durante una interrupción regional, debe crear las redes virtuales en la región emparejada por adelantado. Habilite puntos de conexión de servicio para Azure Storage, con reglas de red que concedan acceso desde estas redes virtuales alternativas. A continuación, aplique estas reglas a las cuentas de almacenamiento con redundancia geográfica.

> [!NOTE]
> Los puntos de conexión de servicio no se aplican al tráfico fuera de la región de la red virtual y el par de región designado. Solo puede aplicar reglas de red que concedan acceso desde redes virtuales a las cuentas de almacenamiento en la región principal de una cuenta de almacenamiento o en la región emparejada designada.

### <a name="required-permissions"></a>Permisos necesarios

Para aplicar una regla de red virtual a una cuenta de almacenamiento, el usuario debe tener permisos apropiados para las subredes que se van a agregar. El permiso necesario es *Join Service to a Subnet* (Unir el servicio a una subred) y se incluye en el rol integrado *Colaborador de la cuenta de almacenamiento*. También se puede agregar a definiciones de roles personalizados.

La cuenta de almacenamiento y las redes virtuales a las que se concedió acceso pueden estar en distintas suscripciones, pero esas suscripciones deben formar parte del mismo inquilino de Azure AD.

### <a name="managing-virtual-network-rules"></a>Administración de reglas de red virtual

Puede administrar las reglas de red virtual para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="azure-portal"></a>Azure Portal

1. Vaya a la cuenta de almacenamiento que quiere proteger.

1. Haga clic en el menú de configuración denominado **Firewalls y redes virtuales**.

1. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

1. Para conceder acceso a una red virtual con una nueva regla de red, en **Redes virtuales**, haga clic en **Agregar red virtual existente**, seleccione las opciones **Redes virtuales** y **Subredes** y, luego, haga clic en **Agregar**. Para crear una nueva red virtual y concederle acceso, haga clic en **Agregar nueva red virtual**. Proporcione la información necesaria para crear la nueva red virtual y, luego, haga clic en **Crear**.

    > [!NOTE]
    > Si un punto de conexión de servicio para Azure Storage no se ha configurado previamente para la red virtual y las subredes seleccionadas, se puede configurar como parte de esta operación.

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

Las reglas de red IP solo se permiten para direcciones IP de **Internet público**. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)) en las reglas de IP. Las redes privadas incluyen direcciones que comienzan por _10.*_, _172.16.*_ - _172.31.*_ y _192.168.*_.

   > [!NOTE]
   > Las reglas de red IP no tienen ningún efecto en las solicitudes que proceden de la misma región de Azure que la cuenta de almacenamiento. Use [reglas de red virtual](#grant-access-from-a-virtual-network) para permitir solicitudes de la misma región.

Solo se admiten direcciones IPV4 en este momento.

Cada cuenta de almacenamiento admite hasta 100 reglas de red de IP, que se pueden combinar con [reglas de red virtual](#grant-access-from-a-virtual-network).

### <a name="configuring-access-from-on-premises-networks"></a>Configuración del acceso desde redes locales

Para conceder acceso desde las redes locales a la cuenta de almacenamiento con una regla de red IP, debe identificar las direcciones IP orientadas a Internet que usa su red. Para obtener ayuda, póngase en contacto con el administrador de red.

Si usa [ExpressRoute](/azure/expressroute/expressroute-introduction) desde las instalaciones para pares públicos o el emparejamiento de Microsoft, tiene que identificar las direcciones IP de NAT que se usan. Para el emparejamiento público, cada circuito ExpressRoute usa de forma predeterminada dos direcciones IP de NAT que se aplican al tráfico del servicio de Azure cuando el tráfico entra en la red troncal de Microsoft Azure. Para el emparejamiento de Microsoft, las direcciones IP de NAT que se utilizan las proporciona el cliente o el proveedor de servicios. Para permitir el acceso a los recursos de servicio, tiene que permitir estas direcciones IP públicas en la configuración del firewall de IP de recursos. Para encontrar las direcciones de IP de circuito de ExpressRoute de los pares públicos, [abra una incidencia de soporte técnico con ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) a través de Azure Portal. Obtenga más información sobre [NAT para ExpressRoute para emparejamiento público y de Microsoft.](/azure/expressroute/expressroute-nat#nat-requirements-for-azure-public-peering)

### <a name="managing-ip-network-rules"></a>Administración de reglas de red IP

Puede administrar las reglas de red IP para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="azure-portal"></a>Azure Portal

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

Las reglas de red pueden habilitar una configuración de red segura para la mayoría de los escenarios. Sin embargo, hay algunos casos donde se deben conceder excepciones para permitir la funcionalidad completa. Puede configurar cuentas de almacenamiento con excepciones para los servicios de Microsoft de confianza y para el acceso a los datos de análisis de almacenamiento.

### <a name="trusted-microsoft-services"></a>Servicios de Microsoft de confianza

Algunos servicios de Microsoft que interactúan con las cuentas de almacenamiento funcionan desde redes a las que no se puede conceder acceso a través de reglas de red.

Para ayudar a que este tipo de servicio funcione según lo previsto, permita que el conjunto de servicios de Microsoft de confianza omita las reglas de red. Estos servicios usarán luego una autenticación sólida para acceder a la cuenta de almacenamiento.

Si habilita la excepción **Permitir que los servicios de Microsoft de confianza…**, se concede acceso a la cuenta de almacenamiento a los siguientes servicios (cuando se han registrado en su suscripción):

|Servicio|Nombre del proveedor de recursos|Propósito|
|:------|:---------------------|:------|
|Azure Backup|Microsoft.RecoveryServices|Ejecute copias de seguridad y restauraciones de discos no administrados en máquinas virtuales de IAAS. (no se necesita para discos administrados). [Más información](/azure/backup/backup-introduction-to-azure-backup).|
|Azure Data Box|Microsoft.DataBox|Permite la importación de datos en Azure mediante el cuadro de datos. [Más información](/azure/databox/data-box-overview).|
|Azure DevTest Labs|Microsoft.DevTestLab|Creación de imagen personalizada e instalación de artefactos. [Más información](/azure/devtest-lab/devtest-lab-overview).|
|Azure Event Grid|Microsoft.EventGrid|Habilite la publicación de eventos de Blob Storage y permita que Event Grid publique en las colas de almacenamiento. Obtenga información sobre los [eventos de Blob Storage](/azure/event-grid/event-sources) y la [publicación en las colas](/azure/event-grid/event-handlers).|
|Azure Event Hubs|Microsoft.EventHub|Archivo de datos con Event Hubs Capture. [Más información](/azure/event-hubs/event-hubs-capture-overview).|
|HDInsight de Azure|Microsoft.HDInsight|Aprovisionar el contenido inicial del sistema de archivos predeterminado para un nuevo clúster de HDInsight. [Más información](https://azure.microsoft.com/blog/enhance-hdinsight-security-with-service-endpoints/).|
|Azure Monitor|Microsoft.Insights|Permite la escritura de los datos de supervisión en una cuenta segura de almacenamiento. [Más información](/azure/monitoring-and-diagnostics/monitoring-roles-permissions-security).|
|Conexión a Azure|Microsoft.Network|Almacenamiento y análisis de los registros de tráfico de red. [Más información](/azure/network-watcher/network-watcher-packet-capture-overview).|
|Azure Site Recovery|Microsoft.SiteRecovery |Configure la recuperación ante desastres al habilitar la replicación para máquinas virtuales de IaaS de Azure. Esto es necesario si usa una cuenta de almacenamiento en caché habilitada mediante firewall o una cuenta de almacenamiento de origen o una cuenta de almacenamiento de destino.  [Más información](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-enable-replication).|
|Azure SQL Data Warehouse|Microsoft.Sql|Permite importar y exportar los escenarios de instancias específicas de las bases de datos SQL mediante PolyBase. [Más información](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview).|

### <a name="storage-analytics-data-access"></a>Acceso a datos de análisis de almacenamiento

En algunos casos, se requiere acceso para leer registros de diagnóstico y métricas desde fuera del límite de red. Puede conceder excepciones a las reglas de red para permitir acceso de lectura a las tablas de métricas, a archivos de registro de cuentas de almacenamiento o a ambos. [Obtenga más información sobre cómo trabajar con analíticas de almacenamiento](/azure/storage/storage-analytics).

### <a name="managing-exceptions"></a>Administración de excepciones

Puede administrar las excepciones de reglas de red a través de Azure Portal, PowerShell o la CLI de Azure v2.

#### <a name="azure-portal"></a>Azure Portal

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

Profundice en la seguridad de Azure Storage en la [Guía de seguridad de Azure Storage](storage-security-guide.md).
