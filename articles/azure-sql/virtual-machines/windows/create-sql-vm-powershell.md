---
title: Guía de uso de Azure PowerShell para aprovisionar SQL Server en una máquina virtual de Azure
description: Ofrece pasos y comandos de PowerShell para crear una VM de Azure con imágenes de la galería de máquinas virtuales de SQL Server.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: 98d50dd8-48ad-444f-9031-5378d8270d7b
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/21/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 94ac7282375660308f8dab37ae5bd874828f778a
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89074753"
---
# <a name="how-to-use-azure-powershell-to-provision-sql-server-on-azure-virtual-machines"></a>Uso de Azure PowerShell para aprovisionar SQL Server en Azure Virtual Machines

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En esta guía se describen las opciones para usar PowerShell con el fin de aprovisionar SQL Server en Azure Virtual Machines. Para obtener un ejemplo de Azure PowerShell simplificado que se basa en valores predeterminados, consulte el [inicio rápido de máquinas virtuales con SQL mediante Azure PowerShell](sql-vm-create-powershell-quickstart.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

## <a name="configure-your-subscription"></a>Configuración de su suscripción

1. Abra PowerShell y establezca el acceso a su cuenta de Azure mediante la ejecución del comando **Connect-AzAccount**.

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, escriba las credenciales. Use el mismo correo electrónico y la misma contraseña que usa para iniciar sesión en el portal de Azure.

## <a name="define-image-variables"></a>Definición de variables de imagen

Para poder reutilizar valores y simplificar la creación de scripts, empiece por definir una serie de variables. Cambie los valores de los parámetros como desee, pero tenga en cuenta las restricciones de nomenclatura relacionadas con la longitud de los nombres y los caracteres especiales al modificar los valores que se proporcionan.

### <a name="location-and-resource-group"></a>Ubicación y grupo de recursos

Defina la región de datos y el grupo de recursos donde desea crear los restantes recursos de la máquina virtual.

Modifíquelos como desee y, después, ejecute estos cmdlets para inicializar estas variables.

```powershell
$Location = "SouthCentralUS"
$ResourceGroupName = "sqlvm2"
```

### <a name="storage-properties"></a>Propiedades de almacenamiento

Defina la cuenta de almacenamiento y el tipo de almacenamiento que va a usar la máquina virtual.

Modifíquelos como desee y, después, ejecute el siguiente cmdlet para inicializar estas variables. Se recomienda usar [discos SSD Premium](../../../virtual-machines/disks-types.md#premium-ssd) para las cargas de trabajo de producción.

```powershell
$StorageName = $ResourceGroupName + "storage"
$StorageSku = "Premium_LRS"
```

### <a name="network-properties"></a>Propiedades de red

Defina las propiedades que va a usar la red en la máquina virtual. 

- interfaz de red
- Método de asignación de TCP/IP
- Nombre de la red virtual
- Nombre de la subred virtual
- Intervalo de direcciones IP de la red virtual
- Intervalo de direcciones IP de la subred
- Etiqueta de nombre de dominio público

Modifíquelos como desee y, después, ejecute este cmdlet para inicializar estas variables.

```powershell
$InterfaceName = $ResourceGroupName + "ServerInterface"
$NsgName = $ResourceGroupName + "nsg"
$TCPIPAllocationMethod = "Dynamic"
$VNetName = $ResourceGroupName + "VNet"
$SubnetName = "Default"
$VNetAddressPrefix = "10.0.0.0/16"
$VNetSubnetAddressPrefix = "10.0.0.0/24"
$DomainName = $ResourceGroupName
```

### <a name="virtual-machine-properties"></a>Propiedades de máquina virtual

Defina las siguientes propiedades:

- Nombre de la máquina virtual
- Nombre del equipo
- Tamaño de la máquina virtual
- Nombre del disco del sistema operativo de la máquina virtual

Modifíquelos como desee y, después, ejecute este cmdlet para inicializar estas variables.

```powershell
$VMName = $ResourceGroupName + "VM"
$ComputerName = $ResourceGroupName + "Server"
$VMSize = "Standard_DS13"
$OSDiskName = $VMName + "OSDisk"
```

### <a name="choose-a-sql-server-image"></a>Elegir una imagen de SQL Server

Utilice las siguientes variables para definir la imagen de SQL Server que se va a usar para la máquina virtual. 

1. En primer lugar, cree una lista de todas las ofertas de imágenes de SQL Server con el comando `Get-AzVMImageOffer`. Este comando enumera las imágenes actuales que están disponibles en Azure Portal y también las imágenes anteriores que solo se pueden instalar con PowerShell:

   ```powershell
   Get-AzVMImageOffer -Location $Location -Publisher 'MicrosoftSQLServer'
   ```

1. Para este tutorial, use las siguientes variables para especificar SQL Server 2017 en Windows Server 2016.

   ```powershell
   $OfferName = "SQL2017-WS2016"
   $PublisherName = "MicrosoftSQLServer"
   $Version = "latest"
   ```

1. Después, enumere las ediciones disponibles en su oferta.

   ```powershell
   Get-AzVMImageSku -Location $Location -Publisher 'MicrosoftSQLServer' -Offer $OfferName | Select Skus
   ```

1. Para este tutorial, utilice SQL Server 2017 Developer Edition (**SQLDEV**). La edición Developer ofrece licencias gratuitamente para desarrollo y pruebas, y solo se paga por el costo de ejecución de la máquina virtual.

   ```powershell
   $Sku = "SQLDEV"
   ```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Con el modelo de implementación de Resource Manager, el primer objeto que se crea es el grupo de recursos. Para crear un grupo de recursos de Azure y sus recursos, use el cmdlet [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Especifique las variables que inicializó anteriormente para el nombre del grupo de recursos y la ubicación.

Ejecute este cmdlet para crear un nuevo grupo de recursos.

```powershell
New-AzResourceGroup -Name $ResourceGroupName -Location $Location
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

La máquina virtual requiere recursos de almacenamiento tanto para el disco del sistema operativo como para los archivos de registro y de datos de SQL Server. Por motivos de simplicidad, creará un único disco para ambos. Posteriormente puede conectar discos adicionales y usar el cmdlet [Add-Azure Disk](/powershell/module/servicemanagement/azure.service/add-azuredisk) para colocar los archivos de registro y de datos de SQL Server en discos dedicados. Use el cmdlet [New-AzStorageAccount](https://docs.microsoft.com/powershell/module/az.storage/new-azstorageaccount) para crear una cuenta de almacenamiento estándar en el nuevo grupo de recursos. Especifique las variables que inicializó anteriormente para el nombre de la cuenta de almacenamiento, el nombre de la SKU del almacenamiento y la ubicación.

Ejecute este cmdlet para crear una nueva cuenta de almacenamiento.

```powershell
$StorageAccount = New-AzStorageAccount -ResourceGroupName $ResourceGroupName `
   -Name $StorageName -SkuName $StorageSku `
   -Kind "Storage" -Location $Location
```

> [!TIP]
> La creación de la cuenta de almacenamiento puede tardar unos minutos.

## <a name="create-network-resources"></a>Crear recursos de red

La máquina virtual requiere un número de recursos de red para la conectividad de red.

* Cada máquina virtual requiere una red virtual.
* Una red virtual debe tener al menos una subred definida.
* Una interfaz de red debe definirse con una dirección IP privada o pública.

### <a name="create-a-virtual-network-subnet-configuration"></a>Creación de una configuración de subred de una red virtual

Para empezar, cree una configuración de subred para la red virtual. Para este tutorial, cree una subred predeterminada mediante el cmdlet [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig). Especifique las variables que inicializó anteriormente para el nombre de la subred y el prefijo de la dirección.

> [!NOTE]
> Con este puede definir propiedades adicionales de la configuración de subred de la red virtual, pero eso está fuera del ámbito de este tutorial.

Ejecute este cmdlet para crear la configuración una de subred virtual.

```powershell
$SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix
```

### <a name="create-a-virtual-network"></a>Creación de una red virtual

A continuación, cree una red virtual en su nuevo grupo de recursos mediante el cmdlet [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork). Especifique las variables que inicializó anteriormente para el nombre, la ubicación y el prefijo de la dirección. Use la configuración de subred que ha definido en el paso anterior.

Ejecute este cmdlet para crear una red virtual.

```powershell
$VNet = New-AzVirtualNetwork -Name $VNetName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig
```

### <a name="create-the-public-ip-address"></a>Crear la dirección IP pública

Una vez que la red virtual está definida, debe configurar una dirección IP para poder conectarse a la máquina virtual. Para este tutorial, cree una dirección IP pública mediante el direccionamiento IP dinámico, con el fin de admitir la conectividad a Internet. Use el cmdlet [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) para crear la dirección IP pública en el nuevo grupo de recursos. Especifique las variables que inicializó anteriormente para el nombre, la ubicación, el método de asignación y la etiqueta del nombre de dominio DNS.

> [!NOTE]
> Con este cmdlet se pueden definir propiedades adicionales de la dirección IP pública, pero eso es algo que está fuera del ámbito de este tutorial inicial. También se puede crear una dirección privada o una dirección con una dirección estática, pero eso está también fuera del ámbito de este tutorial.

Ejecute este cmdlet para crear una dirección IP pública.

```powershell
$PublicIp = New-AzPublicIpAddress -Name $InterfaceName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName
```

### <a name="create-the-network-security-group"></a>Creación del grupo de seguridad de red

Para proteger el tráfico de la VM y SQL Server, cree un grupo de seguridad de red.

1. Primero, cree una regla de grupo de seguridad de red para que el Escritorio remoto (RPD) permita conexiones RDP.

   ```powershell
   $NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp `
      -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow
   ```
1. Configure una regla del grupo de seguridad de red que permita el tráfico en el puerto TCP 1433. Al hacerlo se habilitan las conexiones a SQL Server a través de Internet.

   ```powershell
   $NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp `
      -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow
   ```

1. Cree el grupo de seguridad de red.

   ```powershell
   $Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName `
      -Location $Location -Name $NsgName `
      -SecurityRules $NsgRuleRDP,$NsgRuleSQL
   ```

### <a name="create-the-network-interface"></a>Creación de la interfaz de red

Ya está listo para crear la interfaz de red de la máquina virtual. Use el cmdlet [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface) para crear la interfaz de red en el nuevo grupo de recursos. Especifique el nombre, la ubicación, la subred y la dirección IP pública definidas anteriormente.

Ejecute este cmdlet para crear una interfaz de red.

```powershell
$Interface = New-AzNetworkInterface -Name $InterfaceName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id `
   -NetworkSecurityGroupId $Nsg.Id
```

## <a name="configure-a-vm-object"></a>Configuración de un objeto de VM

Ahora que los recursos de almacenamiento y de red están definidos, esta listo para definir los recursos de proceso para la máquina virtual.

- Especifique el tamaño de máquina virtual y varias propiedades del sistema operativo.
- Especifique la interfaz de red que ha creado anteriormente.
- Defina el almacenamiento de blobs.
- Especifique el sistema operativo.

### <a name="create-the-vm-object"></a>Creación del objeto de VM

Comience por especificar el tamaño de la máquina virtual. En este tutorial, especifique un DS13. Use el cmdlet [New-AzVMConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azvmconfig) para crear un objeto de máquina virtual configurable. Especifique las variables que inicializó anteriormente para el nombre y el tamaño.

Ejecute este cmdlet para crear el objeto de máquina virtual.

```powershell
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
```

### <a name="create-a-credential-object-to-hold-the-name-and-password-for-the-local-administrator-credentials"></a>Creación de un objeto de credencial que contenga el nombre y la contraseña de las credenciales de administrador local

Para que pueda establecer las propiedades del sistema operativo de la máquina virtual, debe especificar las credenciales de la cuenta de administrador local en forma de cadena segura. Para ello, use el cmdlet [Get-Credential](https://technet.microsoft.com/library/hh849815.aspx).

Ejecute el siguiente cmdlet. Deberá escribir el nombre y la contraseña del administrador local de la máquina virtual en la ventana de solicitud de credenciales de PowerShell.

```powershell
$Credential = Get-Credential -Message "Type the name and password of the local administrator account."
```

### <a name="set-the-operating-system-properties-for-the-virtual-machine"></a>Establecimiento de las propiedades del sistema operativo de la máquina virtual

Ya está listo para establecer las propiedades del sistema operativo de la máquina virtual con el cmdlet [Set-AzVMOperatingSystem](https://docs.microsoft.com/powershell/module/az.compute/set-azvmoperatingsystem).

- Establezca el tipo de sistema operativo como Windows.
- Requiera la instalación del [agente de máquina virtual](../../../virtual-machines/extensions/agent-windows.md).
- Especifique que el cmdlet habilita la actualización automática.
- Especifique las variables que inicializó anteriormente para el nombre de la máquina virtual, el nombre del equipo y la credencial.

Ejecute este cmdlet para establecer las propiedades de sistema operativo de la máquina virtual.

```powershell
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine `
   -Windows -ComputerName $ComputerName -Credential $Credential `
   -ProvisionVMAgent -EnableAutoUpdate
```

### <a name="add-the-network-interface-to-the-virtual-machine"></a>Adición de la interfaz de red a la máquina virtual

A continuación, use el cmdlet [Add-AzVMNetworkInterface](https://docs.microsoft.com/powershell/module/az.compute/add-azvmnetworkinterface) para agregar la interfaz de red mediante la variable que ha definido anteriormente.

Ejecute este cmdlet para establecer la interfaz de red de la máquina virtual.

```powershell
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id
```

### <a name="set-the-blob-storage-location-for-the-disk-to-be-used-by-the-virtual-machine"></a>Establecimiento la ubicación de Almacenamiento de blobs en el disco que va a usar la máquina virtual

A continuación, establezca la ubicación del almacenamiento de blobs en el disco de la máquina virtual con las variables que definió anteriormente.

Ejecute este cmdlet para establecer la ubicación del almacenamiento de blobs.

```powershell
$OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"
```

### <a name="set-the-operating-system-disk-properties-for-the-virtual-machine"></a>Establecimiento de las propiedades del disco del sistema operativo de la máquina virtual

A continuación, establezca las propiedades del disco del sistema operativo de la máquina virtual mediante el cmdlet [Set-AzVMOSDisk](https://docs.microsoft.com/powershell/module/az.compute/set-azvmosdisk). 

- Especifique que el sistema operativo de la máquina virtual provendrá de una imagen.
- Establezca que el almacenamiento en caché sea de solo lectura (porque SQL Server está instalado en el mismo disco).
- Especifique las variables que inicializó anteriormente para el nombre de la máquina virtual y el disco del sistema operativo.

Ejecute este cmdlet para establecer las propiedades del disco del sistema operativo de la máquina virtual.

```powershell
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name `
   $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage
```

### <a name="specify-the-platform-image-for-the-virtual-machine"></a>Especificación de la imagen de la plataforma de la máquina virtual.

El último paso de la configuración es especificar la imagen de la plataforma de la máquina virtual. Para este tutorial, use la última imagen de SQL Server 2016 CTP. Use el cmdlet [Set-AzVMSourceImage](https://docs.microsoft.com/powershell/module/az.compute/set-azvmsourceimage) para usar esta imagen con las variables que definió anteriormente.

Ejecute este cmdlet para especificar la imagen de la plataforma de la máquina virtual.

```powershell
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine `
   -PublisherName $PublisherName -Offer $OfferName `
   -Skus $Sku -Version $Version
```

## <a name="create-the-sql-vm"></a>Creación de la máquina virtual con SQL

Ahora que ha terminado los pasos de la configuración, está listo para crear la máquina virtual. Use el cmdlet [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) para crear la máquina virtual mediante las variables que ha definido.

> [!TIP]
> La creación de la máquina virtual puede tardar unos minutos.

Ejecute este cmdlet para crear una máquina virtual.

```powershell
New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine
```

La máquina virtual se ha creado.

> [!NOTE]
> Si aparece un error en el diagnóstico de arranque, puede ignorarlo. Se crea una cuenta de almacenamiento estándar para el diagnóstico del arranque, ya que la cuenta de almacenamiento especificada para el disco de la máquina virtual es una cuenta de almacenamiento prémium.

## <a name="install-the-sql-iaas-agent"></a>Instalación del Agente de IaaS de SQL

Las máquinas virtuales de SQL Server son compatibles con características de administración automatizada con la [extensión del Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). Para instalar el agente en la nueva máquina virtual y registrarlo con el proveedor de recursos, ejecute el comando [New-AzSqlVM](/powershell/module/az.sqlvirtualmachine/new-azsqlvm) después de crear la máquina virtual. Especifique el tipo de licencia de la máquina virtual con SQL Server, eligiendo entre pago por uso o traiga su propia licencia mediante la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). Para más información acerca de las licencias, consulte [Modelo de licencia](licensing-model-azure-hybrid-benefit-ahb-change.md). 


   ```powershell
   New-AzSqlVM -ResourceGroupName $ResourceGroupName -Name $VMName -Location $Location -LicenseType <PAYG/AHUB> 
   ```


## <a name="stop-or-remove-a-vm"></a>Detención o eliminación de una máquina virtual

Si no necesita que la máquina virtual se ejecute continuamente, puede detenerla cuando no se esté usando y así evitar cargos innecesarios. El siguiente comando detiene la VM, pero la deja disponible para usarla en el futuro.

```powershell
Stop-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
```

También puede eliminar de forma definitiva todos los recursos asociados a la máquina virtual con el comando **Remove-AzResourceGroup**. Si lo hace, también se elimina la máquina virtual de forma permanente, así que use este comando con cuidado.

## <a name="example-script"></a>Script de ejemplo

El siguiente script contiene el script de PowerShell completo de este tutorial. Se da por hecho que ya ha configurado la suscripción de Azure para usarla con los comandos **Connect-AzAccount** y **Select-AzSubscription**.

```powershell
# Variables

## Global
$Location = "SouthCentralUS"
$ResourceGroupName = "sqlvm2"

## Storage
$StorageName = $ResourceGroupName + "storage"
$StorageSku = "Premium_LRS"

## Network
$InterfaceName = $ResourceGroupName + "ServerInterface"
$NsgName = $ResourceGroupName + "nsg"
$VNetName = $ResourceGroupName + "VNet"
$SubnetName = "Default"
$VNetAddressPrefix = "10.0.0.0/16"
$VNetSubnetAddressPrefix = "10.0.0.0/24"
$TCPIPAllocationMethod = "Dynamic"
$DomainName = $ResourceGroupName

##Compute
$VMName = $ResourceGroupName + "VM"
$ComputerName = $ResourceGroupName + "Server"
$VMSize = "Standard_DS13"
$OSDiskName = $VMName + "OSDisk"

##Image
$PublisherName = "MicrosoftSQLServer"
$OfferName = "SQL2017-WS2016"
$Sku = "SQLDEV"
$Version = "latest"

# Resource Group
New-AzResourceGroup -Name $ResourceGroupName -Location $Location

# Storage
$StorageAccount = New-AzStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageName -SkuName $StorageSku -Kind "Storage" -Location $Location

# Network
$SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix
$VNet = New-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName -Location $Location -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig
$PublicIp = New-AzPublicIpAddress -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName
$NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow
$NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow
$Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName -Location $Location -Name $NsgName -SecurityRules $NsgRuleRDP,$NsgRuleSQL
$Interface = New-AzNetworkInterface -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id -NetworkSecurityGroupId $Nsg.Id

# Compute
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$Credential = Get-Credential -Message "Type the name and password of the local administrator account."
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate #-TimeZone = $TimeZone
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id
$OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage

# Image
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName $PublisherName -Offer $OfferName -Skus $Sku -Version $Version

# Create the VM in Azure
New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine

# Add the SQL IaaS Extension, and choose the license type
New-AzSqlVM -ResourceGroupName $ResourceGroupName -Name $VMName -Location $Location -LicenseType <PAYG/AHUB> 
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear la máquina virtual, puede:

- Conectarse a la máquina virtual mediante RDP
- Configurar las opciones de SQL Server en el portal de la VM, incluidas las opciones siguientes:
   - [Configuración de almacenamiento](storage-configuration.md) 
   - [Tareas de administración automatizadas](sql-server-iaas-agent-extension-automate-management.md)
- [Configuración de la conectividad](ways-to-connect-to-sql.md)
- Conectar clientes y aplicaciones a la nueva instancia de SQL Server
