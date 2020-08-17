---
title: 'Guía de inicio rápido: creación de una máquina virtual Linux con Azure PowerShell'
description: En esta guía de inicio rápido, aprenderá a usar Azure PowerShell para crear una máquina virtual Linux.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/31/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: e3d400726bfb65b2548bc773ffb460fe1ad426a0
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513458"
---
# <a name="quickstart-create-a-linux-virtual-machine-in-azure-with-powershell"></a>Inicio rápido: Creación de una máquina virtual Linux en Azure con PowerShell

El módulo de Azure PowerShell se usa para crear y administrar recursos de Azure desde la línea de comandos de PowerShell o en scripts. En esta guía de inicio rápido se muestra cómo usar el módulo de Azure PowerShell para implementar una máquina virtual Linux en Azure. En este inicio rápido se usa la imagen de Marketplace de Ubuntu 18.04 LTS de Canonical. Para ver la máquina virtual en acción, conéctese a ella mediante SSH e instale el servidor web NGINX.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-ssh-key-pair"></a>Creación del par de claves SSH

Use [ssh-keygen](https://www.ssh.com/ssh/keygen/) para crear un par de claves SSH. Si ya tiene un par de claves SSH, puede omitir este paso.


```azurepowershell-interactive
ssh-keygen -m PEM -t rsa -b 4096
```

Se le pedirá que proporcione un especifique un nombre de archivo para el par de claves, o bien puede presionar **Entrar** para usar la ubicación predeterminada de `/home/<username>/.ssh/id_rsa`. También podrá crear una contraseña para las claves, en caso de que lo desee.

Para más información sobre cómo crear pares de claves SSH, consulte [Uso de claves SSH con Windows](ssh-from-windows.md).

Si crea el par de claves SSH mediante Cloud Shell, este se almacenará en una [cuenta de almacenamiento que Cloud Shell crea automáticamente](../../cloud-shell/persisting-shell-storage.md). No elimine la cuenta de almacenamiento ni el recurso compartido de archivos que hay en ella contiene hasta que haya recuperado las claves o perderá el acceso a la máquina virtual. 

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y administran los recursos de Azure.

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-virtual-network-resources"></a>Creación de recursos de virtual

Cree una red virtual, una subred, una dirección IP pública. Estos recursos se utilizan para proporcionar conectividad de red con la máquina virtual y conectarla a Internet:

```azurepowershell-interactive
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myVNET" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"
```

Cree una regla de tráfico y de grupo de seguridad de red de Azure. El grupo de seguridad de red protege la máquina virtual con reglas de entrada y de salida. En el ejemplo siguiente, se crea una regla de entrada para el puerto TCP 22 que permite conexiones SSH. Para permitir el tráfico web de entrada, también se crea una regla de entrada para el puerto TCP 80.

```azurepowershell-interactive
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleSSH"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 22 `
  -Access "Allow"

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleWWW"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1001 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access "Allow"

# Create a network security group
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myNetworkSecurityGroup" `
  -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

Cree una tarjeta de interfaz de red (NIC) virtual con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). La NIC virtual conecta la máquina virtual a una subred, un grupo de seguridad de red y una dirección IP pública.

```azurepowershell-interactive
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface `
  -Name "myNic" `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Para crear una máquina virtual en PowerShell, cree una configuración que tenga valores como, por ejemplo, la imagen que se va a usar, el tamaño y las opciones de autenticación. Después, la configuración se utiliza para crear la máquina virtual.

Defina las credenciales de SSH, la información del sistema operativo y el tamaño de máquina virtual. En este ejemplo, la clave SSH se almacena en `~/.ssh/id_rsa.pub`. 

```azurepowershell-interactive
# Define a credential object
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig `
  -VMName "myVM" `
  -VMSize "Standard_D1" | `
Set-AzVMOperatingSystem `
  -Linux `
  -ComputerName "myVM" `
  -Credential $cred `
  -DisablePasswordAuthentication | `
Set-AzVMSourceImage `
  -PublisherName "Canonical" `
  -Offer "UbuntuServer" `
  -Skus "18.04-LTS" `
  -Version "latest" | `
Add-AzVMNetworkInterface `
  -Id $nic.Id

# Configure the SSH key
$sshPublicKey = cat ~/.ssh/id_rsa.pub
Add-AzVMSshPublicKey `
  -VM $vmconfig `
  -KeyData $sshPublicKey `
  -Path "/home/azureuser/.ssh/authorized_keys"
```

Ahora, combine las definiciones de configuración anteriores para crearlas con [New-AzVM](/powershell/module/az.compute/new-azvm):

```azurepowershell-interactive
New-AzVM `
  -ResourceGroupName "myResourceGroup" `
  -Location eastus -VM $vmConfig
```

La implementación de la máquina virtual tardará unos minutos. Cuando finalice la implementación, pase a la siguiente sección.

## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

Cree una conexión SSH con la máquina virtual mediante la dirección IP pública. Para ver la dirección IP pública de la máquina virtual, use el cmdlet [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress):

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

Utilice el mismo shell que utilizó para crear su par de claves SSH, pegue el siguiente comando en la shell para crear una sesión de SSH. Reemplace *10.111.12.123* por la dirección IP de su máquina virtual.

```bash
ssh azureuser@10.111.12.123
```

Cuando se le solicite, el nombre de usuario de inicio de sesión es *azureuser*. Si se usa una frase de contraseña con las claves SSH, debe especificarla cuando se le solicite.


## <a name="install-nginx"></a>Instalación de NGINX

Para ver la máquina virtual en acción, instale al servidor de web de NGINX. En la sesión de SSH, actualice los orígenes de paquete e instale el paquete NGINX más reciente.

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

Cuando haya terminado, escriba `exit` para salir de la sesión de SSH.


## <a name="view-the-web-server-in-action"></a>Visualización del servidor web en acción

Use un explorador web de su elección para ver la página de bienvenida predeterminada de NGINX. Escriba la dirección IP pública de la máquina virtual como dirección web. La dirección IP pública se encuentra en la página de introducción de la máquina virtual o como parte de la cadena de conexión de SSH que usó anteriormente.

![Página de principal predeterminada de NGINX](./media/quick-create-cli/nginix-welcome-page.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, implementó una máquina virtual sencilla, creó un grupo de seguridad de red y una regla e instaló un servidor web básico. Para más información acerca de las máquinas virtuales de Azure, continúe con el tutorial de máquinas virtuales Linux.

> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Linux de Azure](./tutorial-manage-vm.md)
