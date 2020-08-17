---
title: 'Tutorial: Adición de SQL Managed Instance a un grupo de conmutación por error'
titleSuffix: Azure SQL Managed Instance
description: En este tutorial, aprenderá a crear un grupo de conmutación por error entre una instancia principal y una secundaria de Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: sashan, carlrab
ms.date: 08/27/2019
ms.openlocfilehash: 47f33d8b1a7792487491cbe7f2ddb5c7f5b087af
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002993"
---
# <a name="tutorial-add-sql-managed-instance-to-a-failover-group"></a>Tutorial: Adición de SQL Managed Instance a un grupo de conmutación por error
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Agregue instancias administradas de Azure SQL Managed Instance a un grupo de conmutación por error. En este artículo, aprenderá a:

> [!div class="checklist"]
> - Crear una instancia administrada principal.
> - Crear una instancia administrada secundaria como parte de [un grupo de conmutación por error](../database/auto-failover-group-overview.md). 
> - Probar la conmutación por error.

  > [!NOTE]
  > - Al completar este tutorial, asegúrese de configurar los recursos con los [requisitos previos para configurar grupos de conmutación por error para Instancia administrada de SQL](../database/auto-failover-group-overview.md#enabling-geo-replication-between-managed-instances-and-their-vnets). 
  > - La creación de una instancia administrada puede tardar bastante tiempo. Como resultado, este tutorial podría tardar varias horas en completarse. Para obtener más información sobre los tiempos de aprovisionamiento, consulte [Operaciones de administración de Instancia administrada de SQL](sql-managed-instance-paas-overview.md#management-operations). 
  > - Las instancias administradas que participan en un grupo de conmutación por error requieren [Azure ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md) o dos puertas de enlace de VPN conectadas. No se admite el emparejamiento de VNet global. En este tutorial se proporcionan los pasos para crear y conectar las puertas de enlace de VPN. Sáltese estos pasos si ya tiene ExpressRoute configurado. 


## <a name="prerequisites"></a>Prerrequisitos

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para completar este tutorial, asegúrese de disponer de los siguientes elementos: 

- Suscripción a Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/) si aún no tiene una.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para completar el tutorial, asegúrese de que cuenta con los elementos siguientes:

- Suscripción a Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/) si aún no tiene una.
- [Azure PowerShell](/powershell/azure/)

---


## <a name="1---create-a-resource-group-and-primary-managed-instance"></a>1\. Creación de un grupo de recursos y una instancia administrada principal

En este paso, creará el grupo de recursos y la instancia administrada principal del grupo de conmutación por error mediante Azure Portal o PowerShell. 


# <a name="portal"></a>[Portal](#tab/azure-portal) 

Cree el grupo de recursos y la instancia administrada principal mediante Azure Portal. 

1. Seleccione **Azure SQL** en el menú izquierdo de Azure Portal. Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y, a continuación, escriba `Azure SQL` en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo. 
1. Seleccione **+ Agregar** para abrir la página **Select SQL deployment option** (Seleccionar la opción de implementación de SQL). Para ver más información acerca de las distintas bases de datos, seleccione **Mostrar detalles** en el icono **Bases de datos**.
1. Seleccione **Crear** en el icono **Instancias administradas de SQL**. 

    ![Seleccione Instancia administrada de SQL](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. En la página **Crear instancia administrada de Azure SQL**, desde la pestaña **Aspectos básicos**:
    1. En **Detalles del proyecto**, seleccione su **Suscripción** en la lista desplegable y, a continuación, elija **Crear nuevo** en grupo de recursos. Escriba un nombre para el grupo de recursos, como `myResourceGroup`. 
    1. En **Detalles de la instancia administrada de SQL**, proporcione el nombre de la instancia administrada y la región en la que quiere implementar la instancia administrada. Deje el campo **Proceso y almacenamiento** con el valor predeterminado. 
    1. En **Cuenta de administrador** proporcione un inicio de sesión de administrador (como `azureuser`) y una contraseña de administrador compleja. 

    ![Creación de una instancia administrada principal](./media/failover-group-add-instance-tutorial/primary-sql-mi-values.png)

1. Deje el resto de la configuración en sus valores predeterminados y seleccione **Revisar y crear** para revisar la configuración de la Instancia administrada de SQL. 
1. Seleccione **Crear** para crear la instancia administrada principal. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree el grupo de recursos y la instancia administrada principal mediante PowerShell. 

   ```powershell-interactive
   # Connect-AzAccount
   # The SubscriptionId in which to create these objects
   $SubscriptionId = '<Subscription-ID>'
   # Create a random identifier to use as subscript for the different resource names
   $randomIdentifier = $(Get-Random)
   # Set the resource group name and location for SQL Managed Instance
   $resourceGroupName = "myResourceGroup-$randomIdentifier"
   $location = "eastus"
   $drLocation = "eastus2"
   
   # Set the networking values for your primary managed instance
   $primaryVNet = "primaryVNet-$randomIdentifier"
   $primaryAddressPrefix = "10.0.0.0/16"
   $primaryDefaultSubnet = "primaryDefaultSubnet-$randomIdentifier"
   $primaryDefaultSubnetAddress = "10.0.0.0/24"
   $primaryMiSubnetName = "primaryMISubnet-$randomIdentifier"
   $primaryMiSubnetAddress = "10.0.0.0/24"
   $primaryMiGwSubnetAddress = "10.0.255.0/27"
   $primaryGWName = "primaryGateway-$randomIdentifier"
   $primaryGWPublicIPAddress = $primaryGWName + "-ip"
   $primaryGWIPConfig = $primaryGWName + "-ipc"
   $primaryGWAsn = 61000
   $primaryGWConnection = $primaryGWName + "-connection"
   
   
   # Set the networking values for your secondary managed instance
   $secondaryVNet = "secondaryVNet-$randomIdentifier"
   $secondaryAddressPrefix = "10.128.0.0/16"
   $secondaryDefaultSubnet = "secondaryDefaultSubnet-$randomIdentifier"
   $secondaryDefaultSubnetAddress = "10.128.0.0/24"
   $secondaryMiSubnetName = "secondaryMISubnet-$randomIdentifier"
   $secondaryMiSubnetAddress = "10.128.0.0/24"
   $secondaryMiGwSubnetAddress = "10.128.255.0/27"
   $secondaryGWName = "secondaryGateway-$randomIdentifier"
   $secondaryGWPublicIPAddress = $secondaryGWName + "-IP"
   $secondaryGWIPConfig = $secondaryGWName + "-ipc"
   $secondaryGWAsn = 62000
   $secondaryGWConnection = $secondaryGWName + "-connection"
   
   
   
   # Set the SQL Managed Instance name for the new managed instances
   $primaryInstance = "primary-mi-$randomIdentifier"
   $secondaryInstance = "secondary-mi-$randomIdentifier"
   
   # Set the admin login and password for SQL Managed Instance
   $secpasswd = "PWD27!"+(New-Guid).Guid | ConvertTo-SecureString -AsPlainText -Force
   $mycreds = New-Object System.Management.Automation.PSCredential ("azureuser", $secpasswd)
   
   
   # Set the SQL Managed Instance service tier, compute level, and license mode
   $edition = "General Purpose"
   $vCores = 8
   $maxStorage = 256
   $computeGeneration = "Gen5"
   $license = "LicenseIncluded" #"BasePrice" or LicenseIncluded if you have don't have SQL Server license that can be used for AHB discount
   
   # Set failover group details
   $vpnSharedKey = "mi1mi2psk"
   $failoverGroupName = "failovergroup-$randomIdentifier"
   
   # Show randomized variables
   Write-host "Resource group name is" $resourceGroupName
   Write-host "Password is" $secpasswd
   Write-host "Primary Virtual Network name is" $primaryVNet
   Write-host "Primary default subnet name is" $primaryDefaultSubnet
   Write-host "Primary SQL Managed Instance subnet name is" $primaryMiSubnetName
   Write-host "Secondary Virtual Network name is" $secondaryVNet
   Write-host "Secondary default subnet name is" $secondaryDefaultSubnet
   Write-host "Secondary SQL Managed Instance subnet name is" $secondaryMiSubnetName
   Write-host "Primary SQL Managed Instance name is" $primaryInstance
   Write-host "Secondary SQL Managed Instance name is" $secondaryInstance
   Write-host "Failover group name is" $failoverGroupName
   
   # Suppress networking breaking changes warning (https://aka.ms/azps-changewarnings
   Set-Item Env:\SuppressAzurePowerShellBreakingChangeWarnings "true"
   
   # Set the subscription context
   Set-AzContext -SubscriptionId $subscriptionId 
   
   # Create the resource group
   Write-host "Creating resource group..."
   $resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
   $resourceGroup
   
   # Configure the primary virtual network
   Write-host "Creating primary virtual network..."
   $primaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -Name $primaryVNet `
                         -AddressPrefix $primaryAddressPrefix
   
                     Add-AzVirtualNetworkSubnetConfig `
                         -Name $primaryMiSubnetName `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                     | Set-AzVirtualNetwork
   $primaryVirtualNetwork
   
   
   # Configure the primary managed instance subnet
   Write-host "Configuring primary MI subnet..."
   $primaryVirtualNetwork = Get-AzVirtualNetwork -Name $primaryVNet -ResourceGroupName $resourceGroupName
   
   
   $primaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $primaryMiSubnetName `
                           -VirtualNetwork $primaryVirtualNetwork
   $primaryMiSubnetConfig
   
   # Configure the network security group management service
   Write-host "Configuring primary MI subnet..."
   
   $primaryMiSubnetConfigId = $primaryMiSubnetConfig.Id
   
   $primaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'primaryNSGMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryNSGMiManagementService
   
   # Configure the route table management service
   Write-host "Configuring primary MI route table management service..."
   
   $primaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'primaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryRouteTableMiManagementService
   
   # Configure the primary network security group
   Write-host "Configuring primary network security group..."
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -Name $primaryMiSubnetName `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NetworkSecurityGroup $primaryNSGMiManagementService `
                         -RouteTable $primaryRouteTableMiManagementService | `
                       Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryNSGMiManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $PrimaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $PrimaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   Write-host "Primary network security group configured successfully."
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "primaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Primary network route table configured successfully."
   
   
   # Create the primary managed instance
   
   Write-host "Creating primary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $primaryInstance `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -SubnetId $primaryMiSubnetConfigId `
                         -AdministratorCredential $mycreds `
                         -StorageSizeInGB $maxStorage `
                         -VCore $vCores `
                         -Edition $edition `
                         -ComputeGeneration $computeGeneration `
                         -LicenseType $license
   Write-host "Primary SQL Managed Instance created successfully."
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos de Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crea una tabla de rutas. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Actualiza una configuración de subred para una red virtual.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtiene un grupo de seguridad de red. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Agrega una configuración de regla de seguridad de red a un grupo de seguridad de red. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Actualiza un grupo de seguridad de red.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Agrega una ruta a una tabla de rutas. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Actualiza una tabla de rutas.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crea una instancia administrada.  |

---

## <a name="2---create-secondary-virtual-network"></a>2\. Creación de una red virtual secundaria

Si usa Azure Portal para crear la instancia administrada, tendrá que crear la red virtual por separado, ya que hay un requisito de que la subred de la instancia administrada principal y secundaria no tenga intervalos superpuestos. Si usa PowerShell para configurar la instancia administrada, vaya al paso 3. 

# <a name="portal"></a>[Portal](#tab/azure-portal) 

Para comprobar el intervalo de subred de la red virtual principal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a su grupo de recursos y seleccione la red virtual para la instancia principal.  
2. Seleccione **Subredes** en **Configuración** y anote el **Intervalo de direcciones**. El intervalo de direcciones de subred perteneciente a la red virtual para la instancia administrada secundaria no puede superponerse con el intervalo anotado. 


   ![Subred principal](./media/failover-group-add-instance-tutorial/verify-primary-subnet-range.png)

Para crear una red virtual, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** y busque *red virtual*. 
1. Seleccione la opción **Red virtual** publicada por Microsoft y, a continuación, seleccione **Crear** en la página siguiente. 
1. Rellene los campos obligatorios para configurar la red virtual para la instancia administrada secundaria y después seleccione **Crear**. 

   En la tabla siguiente se muestran los valores necesarios para la red virtual secundaria:

    | **Campo** | Value |
    | --- | --- |
    | **Nombre** |  Nombre de la red virtual que va a usar la instancia administrada secundaria, como `vnet-sql-mi-secondary`. |
    | **Espacio de direcciones** | Espacio de direcciones de la red virtual, como `10.128.0.0/16`. | 
    | **Suscripción** | Suscripción en la que residen la instancia administrada principal y el grupo de recursos. |
    | **Región** | Ubicación en la que implementará la instancia administrada secundaria. |
    | **Subred** | Nombre de la subred. De forma predeterminada, se proporciona `default`. |
    | **Intervalo de direcciones**| Intervalo de direcciones para la subred. Debe ser distinto al intervalo de direcciones de subred que usó la red virtual de la instancia administrada principal, como `10.128.0.0/24`.  |
    | &nbsp; | &nbsp; |

    ![Valores de red virtual secundaria](./media/failover-group-add-instance-tutorial/secondary-virtual-network.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Este paso es necesario solo si utiliza Azure Portal para implementar SQL Managed Instance. Si usa PowerShell, vaya al paso 3. 

---

## <a name="3---create-a-secondary-managed-instance"></a>3\. Creación de una instancia administrada secundaria
En este paso, creará una instancia administrada secundaria en Azure Portal, que también configurará las redes entre las dos instancias administradas. 

La segunda instancia administrada debe:
- Estar vacía. 
- Tener una subred y un intervalo de direcciones IP distintos de los de la instancia administrada principal. 

# <a name="portal"></a>[Portal](#tab/azure-portal) 

Cree una instancia administrada secundaria mediante Azure Portal. 

1. Seleccione **Azure SQL** en el menú izquierdo de Azure Portal. Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y, a continuación, escriba `Azure SQL` en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo. 
1. Seleccione **+ Agregar** para abrir la página **Select SQL deployment option** (Seleccionar la opción de implementación de SQL). Para ver más información acerca de las distintas bases de datos, seleccione **Mostrar detalles** en el icono **Bases de datos**.
1. Seleccione **Crear** en el icono **Instancias administradas de SQL**. 

    ![Seleccione Instancia administrada de SQL](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. En la pestaña **Aspectos básicos** de la página **Crear instancia administrada de Azure SQL**, rellene los campos obligatorios para configurar la instancia administrada secundaria. 

   En la tabla siguiente se muestran los valores necesarios para la instancia administrada secundaria:
 
    | **Campo** | Value |
    | --- | --- |
    | **Suscripción** |  Suscripción en la que reside la instancia administrada principal. |
    | **Grupos de recursos**| Grupo de recursos en el que reside la instancia administrada principal. |
    | **Nombre de la Instancia administrada de SQL** | Nombre de la nueva instancia administrada secundaria, por ejemplo, `sql-mi-secondary`.  | 
    | **Región**| Ubicación de la instancia administrada secundaria.  |
    | **Inicio de sesión de administrador de la Instancia administrada de SQL** | Inicio de sesión que quiere usar para la nueva instancia administrada secundaria, como `azureuser`. |
    | **Contraseña** | Contraseña compleja que usará el inicio de sesión del administrador para la nueva instancia administrada secundaria.  |
    | &nbsp; | &nbsp; |

1. En la pestaña **Redes**, como **Red virtual**, seleccione la red virtual que creó para la instancia administrada secundaria de la lista desplegable.

   ![Redes de instancia administrada secundaria](./media/failover-group-add-instance-tutorial/networking-settings-for-secondary-mi.png)

1. En la pestaña **Configuración adicional**, para **Replicación geográfica** elija **Sí** en _Usar como secundario de conmutación por error_. Seleccione la instancia administrada principal en la lista desplegable. 
    
   Asegúrese de que la intercalación y la zona horaria coinciden con las de la instancia administrada principal. La instancia administrada principal creada en este tutorial usó el valor predeterminado de intercalación `SQL_Latin1_General_CP1_CI_AS` y la zona horaria `(UTC) Coordinated Universal Time`. 

   ![Redes de la instancia administrada secundaria](./media/failover-group-add-instance-tutorial/secondary-mi-failover.png)

1. Seleccione **Revisar y crear** para revisar la configuración de la instancia administrada secundaria. 
1. Seleccione **Crear** para crear la instancia administrada secundaria. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree una instancia administrada secundaria mediante PowerShell. 

   ```powershell-interactive
   # Configure the secondary virtual network
   Write-host "Configuring secondary virtual network..."
   
   $SecondaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $drlocation `
                         -Name $secondaryVNet `
                         -AddressPrefix $secondaryAddressPrefix
   
   Add-AzVirtualNetworkSubnetConfig `
                         -Name $secondaryMiSubnetName `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -AddressPrefix $secondaryMiSubnetAddress `
                       | Set-AzVirtualNetwork
   $SecondaryVirtualNetwork
   
   # Configure the secondary managed instance subnet
   Write-host "Configuring secondary MI subnet..."
   
   $SecondaryVirtualNetwork = Get-AzVirtualNetwork -Name $secondaryVNet `
                                   -ResourceGroupName $resourceGroupName
   
   $secondaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $secondaryMiSubnetName `
                           -VirtualNetwork $SecondaryVirtualNetwork
   $secondaryMiSubnetConfig
   
   # Configure the secondary network security group management service
   Write-host "Configuring secondary network security group management service..."
   
   $secondaryMiSubnetConfigId = $secondaryMiSubnetConfig.Id
   
   $secondaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'secondaryToMIManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryNSGMiManagementService
   
   # Configure the secondary route table MI management service
   Write-host "Configuring secondary route table MI management service..."
   
   $secondaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'secondaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryRouteTableMiManagementService
   
   # Configure the secondary network security group
   Write-host "Configuring secondary network security group..."
   
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -Name $secondaryMiSubnetName `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NetworkSecurityGroup $secondaryNSGMiManagementService `
                         -RouteTable $secondaryRouteTableMiManagementService `
                       | Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryToMIManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $secondaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $secondaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "secondaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Secondary network security group configured successfully."
   
   # Create the secondary managed instance
   
   $primaryManagedInstanceId = Get-AzSqlInstance -Name $primaryInstance -ResourceGroupName $resourceGroupName | Select-Object Id
   
   
   Write-host "Creating secondary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $secondaryInstance `
                     -ResourceGroupName $resourceGroupName `
                     -Location $drLocation `
                     -SubnetId $secondaryMiSubnetConfigId `
                     -AdministratorCredential $mycreds `
                     -StorageSizeInGB $maxStorage `
                     -VCore $vCores `
                     -Edition $edition `
                     -ComputeGeneration $computeGeneration `
                     -LicenseType $license `
                     -DnsZonePartner $primaryManagedInstanceId.Id
   Write-host "Secondary SQL Managed Instance created successfully."
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos de Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crea una tabla de rutas. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Actualiza una configuración de subred para una red virtual.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtiene un grupo de seguridad de red. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Agrega una configuración de regla de seguridad de red a un grupo de seguridad de red. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Actualiza un grupo de seguridad de red.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Agrega una ruta a una tabla de rutas. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Actualiza una tabla de rutas.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crea una instancia administrada.  |

---

## <a name="4---create-a-primary-gateway"></a>4\. Creación de una puerta de enlace principal 

Para que dos instancias administradas participen en un grupo de conmutación por error, debe haber una puerta de enlace o ExpressRoute configurados entre las redes virtuales de las dos instancias administradas para permitir la comunicación en red. Si decide configurar [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md) en lugar de conectar dos puertas de enlace de VPN, vaya al [paso 7](#7---create-a-failover-group).  

En este artículo se proporcionan los pasos para crear las dos puertas de enlace de VPN y conectarlas, pero puede ir directamente a la creación del grupo de conmutación por error si ha configurado ExpressRoute en su lugar. 


# <a name="portal"></a>[Portal](#tab/azure-portal)

Cree la puerta de enlace para la red virtual de la instancia administrada principal mediante Azure Portal. 


1. En [Azure Portal](https://portal.azure.com), navegue al grupo de recursos y seleccione el recurso **Red virtual** para la instancia administrada principal. 
1. Seleccione **Subredes** en **Configuración** y, a continuación, seleccione para agregar una nueva **Subred de puerta de enlace**. Deje los valores predeterminados. 

   ![Adición de una puerta de enlace para la instancia administrada principal](./media/failover-group-add-instance-tutorial/add-subnet-gateway-primary-vnet.png)

1. Una vez creada la puerta de enlace de subred, seleccione **Crear un recurso** en el panel de navegación izquierdo y, a continuación, escriba `Virtual network gateway` en el cuadro de búsqueda. Seleccione el recurso **Puerta de enlace de red virtual** publicado por **Microsoft**. 

   ![Creación de una nueva puerta de enlace de red virtual](./media/failover-group-add-instance-tutorial/create-virtual-network-gateway.png)

1. Rellene los campos obligatorios para configurar la puerta de enlace de la instancia administrada principal. 

   En la tabla siguiente se muestran los valores necesarios para la puerta de enlace de la instancia administrada principal:
 
    | **Campo** | Value |
    | --- | --- |
    | **Suscripción** |  Suscripción en la que reside la instancia administrada principal. |
    | **Nombre** | Nombre de la puerta de enlace de red virtual, como `primary-mi-gateway`. | 
    | **Región** | Región donde se encuentra la instancia administrada principal. |
    | **Tipo de puerta de enlace** | Seleccione **VPN**. |
    | **Tipo de VPN** | seleccione **Basada en rutas**. |
    | **SKU**| Deje el valor predeterminado de `VpnGw1`. |
    | **Ubicación**| Ubicación donde se encuentran la instancia administrada principal y la red virtual principal.   |
    | **Red virtual**| Seleccione la red virtual que se creó en la sección 2, como `vnet-sql-mi-primary`. |
    | **Dirección IP pública**| Seleccione **Crear nuevo**. |
    | **Nombre de la dirección IP pública**| Escriba un nombre para la dirección IP, como `primary-gateway-IP`. |
    | &nbsp; | &nbsp; |

1. Deje los demás valores predeterminados y, a continuación, seleccione **Revisar y crear** para revisar la configuración de la puerta de enlace de red virtual.

   ![Configuración de la puerta de enlace principal](./media/failover-group-add-instance-tutorial/settings-for-primary-gateway.png)

1. Seleccione **Crear** para crear la nueva puerta de enlace de red virtual. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree la puerta de enlace para la red virtual de la instancia administrada principal mediante PowerShell. 

   ```powershell-interactive
   # Create the primary gateway
   Write-host "Adding GatewaySubnet to primary VNet..."
   Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $primaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $primaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName
   $primaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $primaryVirtualNetwork
   
   Write-host "Creating primary gateway..."
   Write-host "This will take some time."
   $primaryGWPublicIP = New-AzPublicIpAddress -Name $primaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $location -AllocationMethod Dynamic
   $primaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $primaryGWIPConfig `
            -Subnet $primaryGatewaySubnet -PublicIpAddress $primaryGWPublicIP
   
   $primaryGateway = New-AzVirtualNetworkGateway -Name $primaryGWName -ResourceGroupName $resourceGroupName `
       -Location $location -IpConfigurations $primaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $primaryGWAsn
   $primaryGateway
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crea una configuración de IP para una puerta de enlace de red virtual. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crea una puerta de enlace de red virtual. |


---


## <a name="5---create-secondary-gateway"></a>5\. Creación de una puerta de enlace secundaria 
En este paso, va a crear la puerta de enlace para la red virtual de la instancia administrada secundaria mediante Azure Portal. 


# <a name="portal"></a>[Portal](#tab/azure-portal)

Mediante Azure Portal, repita los pasos de la sección anterior para crear la subred y la puerta de enlace de la red virtual para la instancia administrada secundaria. Rellene los campos obligatorios para configurar la puerta de enlace para la instancia administrada secundaria. 

   En la tabla siguiente se muestran los valores necesarios para la puerta de enlace de la instancia administrada secundaria:

   | **Campo** | Value |
   | --- | --- |
   | **Suscripción** |  Suscripción en la que reside la instancia administrada secundaria. |
   | **Nombre** | Nombre de la puerta de enlace de red virtual, como `secondary-mi-gateway`. | 
   | **Región** | Región donde se encuentra la instancia administrada secundaria. |
   | **Tipo de puerta de enlace** | Seleccione **VPN**. |
   | **Tipo de VPN** | seleccione **Basada en rutas**. |
   | **SKU**| Deje el valor predeterminado de `VpnGw1`. |
   | **Ubicación**| Ubicación donde se encuentran la instancia administrada secundaria y la red virtual secundaria.   |
   | **Red virtual**| Seleccione la red virtual que se creó en la sección 2, como `vnet-sql-mi-secondary`. |
   | **Dirección IP pública**| Seleccione **Crear nuevo**. |
   | **Nombre de la dirección IP pública**| Escriba un nombre para la dirección IP, como `secondary-gateway-IP`. |
   | &nbsp; | &nbsp; |

   ![Configuración de la puerta de enlace secundaria](./media/failover-group-add-instance-tutorial/settings-for-secondary-gateway.png)


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree la puerta de enlace para la red virtual de la instancia administrada secundaria mediante PowerShell. 

   ```powershell-interactive
   # Create the secondary gateway
   Write-host "Creating secondary gateway..."
   
   Write-host "Adding GatewaySubnet to secondary VNet..."
   Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $secondaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $secondaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName
   $secondaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $secondaryVirtualNetwork
   $drLocation = $secondaryVirtualNetwork.Location
   
   Write-host "Creating primary gateway..."
   Write-host "This will take some time."
   $secondaryGWPublicIP = New-AzPublicIpAddress -Name $secondaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $drLocation -AllocationMethod Dynamic
   $secondaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $secondaryGWIPConfig `
            -Subnet $secondaryGatewaySubnet -PublicIpAddress $secondaryGWPublicIP
   
   $secondaryGateway = New-AzVirtualNetworkGateway -Name $secondaryGWName -ResourceGroupName $resourceGroupName `
       -Location $drLocation -IpConfigurations $secondaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $secondaryGWAsn
   $secondaryGateway
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crea una configuración de IP para una puerta de enlace de red virtual. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crea una puerta de enlace de red virtual. |

---


## <a name="6---connect-the-gateways"></a>6\. Conexión de las puertas de enlace
En este paso, cree una conexión bidireccional entre las dos puertas de enlace de las dos redes virtuales. 


# <a name="portal"></a>[Portal](#tab/azure-portal)

Conecte las dos puertas de enlace mediante Azure Portal. 


1. Seleccione **Crear un recurso** en [Azure Portal](https://portal.azure.com).
1. Escriba `connection` en el cuadro de búsqueda y, después, presione Entrar para buscar. Esto lo llevará al recurso **Conexión**, publicado por Microsoft.
1. Seleccione **Crear** para crear su conexión. 
1. En la pestaña **Aspectos básicos**, seleccione los siguientes valores y luego seleccione **Aceptar**. 
    1. Seleccione `VNet-to-VNet` para el **tipo de conexión**. 
    1. Seleccione la suscripción en la lista desplegable. 
    1. Seleccione el grupo de recursos de la instancia de SQL Managed Instance en la lista desplegable. 
    1. Seleccione la ubicación de la instancia administrada principal en la lista desplegable. 
1. En la pestaña **Configuración**, seleccione o escriba los valores siguientes y, después, seleccione **Aceptar**:
    1. Elija la puerta de enlace de red principal para la **Primera puerta enlace de red virtual**, como `Primary-Gateway`.  
    1. Elija la puerta de enlace de red secundaria para la **Segunda puerta enlace de red virtual**, como `Secondary-Gateway`. 
    1. Active la casilla situada junto a **Establecer conectividad bidireccional**. 
    1. Deje el nombre de la conexión principal predeterminado o cambie su nombre por un valor de su elección. 
    1. Proporcione una **clave compartida (PSK)** para la conexión, como `mi1m2psk`. 

   ![Crear una conexión de la puerta de enlace](./media/failover-group-add-instance-tutorial/create-gateway-connection.png)

1. En la pestaña **Resumen**, revise la configuración de la conexión bidireccional y, después, seleccione **Aceptar** para crear la conexión. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Conecte las dos puertas de enlace mediante PowerShell. 

   ```powershell-interactive
   # Connect the primary to secondary gateway
   Write-host "Connecting the primary gateway to secondary gateway..."
   New-AzVirtualNetworkGatewayConnection -Name $primaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $primaryGateway -VirtualNetworkGateway2 $secondaryGateway -Location $location `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $primaryGWConnection
   
   # Connect the secondary to primary gateway
   Write-host "Connecting the secondary gateway to primary gateway..."
   
   New-AzVirtualNetworkGatewayConnection -Name $secondaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $secondaryGateway -VirtualNetworkGateway2 $primaryGateway -Location $drLocation `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $secondaryGWConnection
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crea una conexión entre las dos puertas de enlace de redes virtuales.   |

---


## <a name="7---create-a-failover-group"></a>7\. Creación de un grupo de conmutación por error
En este paso, creará el grupo de conmutación por error y le agregará ambas instancias administradas. 


# <a name="portal"></a>[Portal](#tab/azure-portal)
Cree el grupo de conmutación por error mediante Azure Portal. 


1. Seleccione **Azure SQL** en el menú izquierdo de [Azure Portal](https://portal.azure.com). Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y, a continuación, escriba `Azure SQL` en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo. 
1. Seleccione la instancia administrada principal que creó en la primera sección, por ejemplo, `sql-mi-primary`. 
1. En **Configuración**, vaya a **Grupos de conmutación por error de instancias** y, a continuación, elija **Agregar grupo** para abrir la página **Grupo de conmutación por error de instancias**. 

   ![Adición de un grupo de conmutación por error](./media/failover-group-add-instance-tutorial/add-failover-group.png)

1. En la página **Grupo de conmutación por error de instancias**, escriba el nombre del grupo de conmutación por error, por ejemplo, `failovergrouptutorial`. A continuación, elija la instancia administrada secundaria (por ejemplo, `sql-mi-secondary`) en la lista desplegable. Seleccione **Crear** para crear el grupo de conmutación por error. 

   ![Creación de un grupo de conmutación por error](./media/failover-group-add-instance-tutorial/create-failover-group.png)

1. Una vez completada la implementación del grupo de conmutación por error, volverá a mostrarse la página **Grupo de conmutación por error**. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Cree el grupo de conmutación por error mediante PowerShell. 

   ```powershell-interactive
   Write-host "Creating the failover group..."
   $failoverGroup = New-AzSqlDatabaseInstanceFailoverGroup -Name $failoverGroupName `
        -Location $location -ResourceGroupName $resourceGroupName -PrimaryManagedInstanceName $primaryInstance `
        -PartnerRegion $drLocation -PartnerManagedInstanceName $secondaryInstance `
        -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
   $failoverGroup
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crea un nuevo grupo de conmutación por error de la Instancia administrada de Azure SQL.  |


---


## <a name="8---test-failover"></a>8\. Prueba de la conmutación por error
En este paso, se producirá un error en el grupo de conmutación por error en el servidor secundario y, a continuación, se realizará la conmutación por recuperación mediante Azure Portal. 


# <a name="portal"></a>[Portal](#tab/azure-portal)
Pruebe la conmutación por error mediante Azure Portal. 


1. Vaya a la instancia administrada _secundaria_ en [Azure Portal](https://portal.azure.com) y seleccione **Grupos de conmutación por error de instancias** en las opciones de configuración. 
1. Revise cuál instancia administrada es la principal y cuál es la secundaria. 
1. Seleccione **Conmutación por error** y, a continuación, seleccione **Sí** en la advertencia acerca de la desconexión de las sesiones TDS. 

   ![Conmutación por error del grupo de conmutación por error](./media/failover-group-add-instance-tutorial/failover-mi-failover-group.png)

1. Revise cuál instancia administrada es la principal y cuál es la secundaria. Si la conmutación por error se realiza correctamente, las dos instancias deben tener los roles cambiados. 

   ![Las instancias administradas cambian roles después de la conmutación por error](./media/failover-group-add-instance-tutorial/mi-switched-after-failover.png)

1. Vaya a la nueva instancia administrada _secundaria_ y seleccione **Conmutación por error** para que la instancia principal vuelva a tener el rol principal. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pruebe la conmutación por error mediante PowerShell. 

   ```powershell-interactive
    
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   
   # Fail over the primary managed instance to the secondary role
   Write-host "Failing primary over to the secondary location"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to secondary location"
   ```


Revierta el grupo de conmutación por error al servidor principal:

   ```powershell-interactive
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName
   
   # Fail the primary managed instance back to the primary role
   Write-host "Failing primary back to primary role"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to primary location"
   
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtiene o enumera los grupos de conmutación por error de la Instancia administrada de SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Ejecuta una conmutación por error de un grupo de conmutación por error de la Instancia administrada de SQL. | 

---



## <a name="clean-up-resources"></a>Limpieza de recursos
Para limpiar los recursos, elimine primero las instancias administradas, después el clúster virtual, a continuación el resto de los recursos y, por último, el grupo de recursos. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Vaya a su grupo de recursos en [Azure Portal](https://portal.azure.com). 
1. Seleccione las instancias administradas y después seleccione **Eliminar**. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**. Este proceso puede tardar algún tiempo en completarse en segundo plano y, hasta que finalice, no podrá eliminar el *clúster virtual* ni ningún otro recurso dependiente. Supervise la eliminación en la pestaña **Actividad** para confirmar que la instancia administrada se ha eliminado. 
1. Una vez que se elimine la instancia administrada, puede eliminar el *clúster virtual* si lo selecciona en el grupo de recursos y, después, elige **Eliminar**. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**. 
1. Elimine todos los recursos restantes. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**. 
1. Para eliminar el grupo de recursos, seleccione **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos (`myResourceGroup`) y, a continuación, seleccione **Eliminar**. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Tendrá que quitar el grupo de recursos dos veces. Al eliminar el grupo de recursos por primera vez, se eliminarán la instancia administrada y los clústeres virtuales, pero se producirá el mensaje de error `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Ejecute el comando Remove-AzResourceGroup una segunda vez para quitar los recursos residuales y el grupo de recursos.

```powershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing SQL Managed Instance and virtual cluster..."
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing residual resources and resource group..."
```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. |

---

## <a name="full-script"></a>Script completo

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add SQL Managed Instance to a failover group")]

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos de Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crea una tabla de rutas. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Actualiza una configuración de subred para una red virtual.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtiene un grupo de seguridad de red. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Agrega una configuración de regla de seguridad de red a un grupo de seguridad de red. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Actualiza un grupo de seguridad de red.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Agrega una ruta a una tabla de rutas. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Actualiza una tabla de rutas.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crea una instancia administrada.  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Devuelve información sobre Instancia administrada de Azure SQL. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crea una configuración de IP para una puerta de enlace de red virtual. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crea una puerta de enlace de red virtual. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crea una conexión entre las dos puertas de enlace de redes virtuales.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crea un nuevo grupo de conmutación por error de SQL Managed Instance.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtiene o enumera los grupos de conmutación por error de la Instancia administrada de SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Ejecuta una conmutación por error de un grupo de conmutación por error de la Instancia administrada de SQL. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. | 

# <a name="portal"></a>[Portal](#tab/azure-portal) 

No hay scripts disponibles para Azure Portal.

---

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró un grupo de conmutación por error entre dos instancias administradas. Ha aprendido a:

> [!div class="checklist"]
> - Crear una instancia administrada principal.
> - Crear una instancia administrada secundaria como parte de [un grupo de conmutación por error](../database/auto-failover-group-overview.md). 
> - Probar la conmutación por error.

Continúe con la siguiente guía de inicio rápido sobre cómo conectarse a SQL Managed Instance y cómo restaurar una base de datos en SQL Managed Instance: 

> [!div class="nextstepaction"]
> [Conexión a SQL Managed Instance](connect-vm-instance-configure.md)
> [Restauración de una base de datos en SQL Managed Instance](restore-sample-database-quickstart.md)


