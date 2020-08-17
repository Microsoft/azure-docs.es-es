---
title: Conexión de un servicio en la nube a un controlador de dominio personalizado | Microsoft Docs
description: Aprenda a conectar los roles web o de trabajo a un dominio de AD personalizado mediante PowerShell y la extensión de dominio de AD.
services: cloud-services
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: fa918a3a6894205ed36c4b576608e7a71e523a92
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092718"
---
# <a name="connecting-azure-cloud-services-roles-to-a-custom-ad-domain-controller-hosted-in-azure"></a>Conexión de los roles de Azure Cloud Services a un controlador de dominio de AD personalizado que se hospeda en Azure
En primer lugar, vamos a configurar una red virtual en Azure. A continuación, agregaremos un controlador de dominio de Active Directory (hospedado en una máquina virtual de Azure) a la red virtual. Después, agregaremos los roles de servicios en la nube existentes a la red virtual creada previamente y los conectaremos al controlador de dominio.

Antes de empezar, debemos tener en cuenta un par de cosas:

1. En este tutorial se usa PowerShell, así que asegúrese de tener Azure PowerShell instalado y listo para usar. Para obtener ayuda con la configuración de Azure PowerShell, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).
2. El controlador de dominio de AD y las instancias de rol web o de trabajo deben estar en la red virtual.

Siga esta guía paso a paso y, en caso de que surja algún problema, deje un comentario al final del artículo. Nos pondremos en contacto con usted (siempre leemos los comentarios).

La red a la que hace referencia el servicio en la nube **debe ser una red virtual clásica**.

## <a name="create-a-virtual-network"></a>Creación de una red virtual
En Azure se puede crear una red virtual mediante Azure Portal o PowerShell. En este tutorial, se usa PowerShell. Para crear una red virtual mediante Azure Portal, consulte [Creación de una red virtual](../virtual-network/quick-create-portal.md). Este artículo trata sobre la creación de una red virtual de Resource Manager, pero debe crear una red virtual clásica para los servicios en la nube. Para ello, en el portal, seleccione **Crear un recurso**, escriba *red virtual* en el cuadro **Buscar** y luego presione **Entrar**. En los resultados de búsqueda, en **Todo**, seleccione **Red virtual**. En **Seleccionar un modelo de implementación**, seleccione **Clásico** y luego haga clic en **Crear**. A continuación, puede seguir los pasos descritos en el artículo.

```powershell
#Create Virtual Network

$vnetStr =
@"<?xml version="1.0" encoding="utf-8"?>
<NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
    <VirtualNetworkConfiguration>
    <VirtualNetworkSites>
        <VirtualNetworkSite name="[your-vnet-name]" Location="West US">
        <AddressSpace>
            <AddressPrefix>[your-address-prefix]</AddressPrefix>
        </AddressSpace>
        <Subnets>
            <Subnet name="[your-subnet-name]">
            <AddressPrefix>[your-subnet-range]</AddressPrefix>
            </Subnet>
        </Subnets>
        </VirtualNetworkSite>
    </VirtualNetworkSites>
    </VirtualNetworkConfiguration>
</NetworkConfiguration>
"@;

$vnetConfigPath = "<path-to-vnet-config>"
Set-AzureVNetConfig -ConfigurationPath $vnetConfigPath
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual
Una vez que haya completado la configuración de la red virtual, deberá crear un controlador de dominio de AD. En este tutorial, se va a configurar un controlador de dominio de AD en una máquina virtual de Azure.

Para ello, cree una máquina virtual a través de PowerShell mediante los siguientes comandos:

```powershell
# Initialize variables
# VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.

$vnetname = '<your-vnet-name>'
$subnetname = '<your-subnet-name>'
$vmsvc1 = '<your-hosted-service>'
$vm1 = '<your-vm-name>'
$username = '<your-username>'
$password = '<your-password>'
$affgrp = '<your- affgrp>'

# Create a VM and add it to the Virtual Network

New-AzureQuickVM -Windows -ServiceName $vmsvc1 -Name $vm1 -ImageName $imgname -AdminUsername $username -Password $password -AffinityGroup $affgrp -SubnetNames $subnetname -VNetName $vnetname
```

## <a name="promote-your-virtual-machine-to-a-domain-controller"></a>Promoción de la máquina virtual a un controlador de dominio
Para configurar la máquina virtual como controlador de dominio de AD, deberá iniciar sesión en esta y configurarla.

Para iniciar sesión en la máquina virtual, puede obtener el archivo RDP a través de PowerShell con los siguientes comandos:

```powershell
# Get RDP file
Get-AzureRemoteDesktopFile -ServiceName $vmsvc1 -Name $vm1 -LocalPath <rdp-file-path>
```

Una vez que haya iniciado sesión en la máquina virtual, configúrela como controlador de dominio de AD, para lo que debe seguir la guía paso a paso acerca de la [configuración de un controlador de dominio de AD de cliente](https://social.technet.microsoft.com/wiki/contents/articles/12370.windows-server-2012-set-up-your-first-domain-controller-step-by-step.aspx).

## <a name="add-your-cloud-service-to-the-virtual-network"></a>Incorporación del servicio en la nube a la red virtual
A continuación, es preciso que agregue la implementación del servicio en la nube a la nueva red virtual. Para ello, modifique el archivo cscfg del servicio en la nube agregando a este las secciones relevantes con Visual Studio o el editor de su elección.

```xml
<ServiceConfiguration serviceName="[hosted-service-name]" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="[os-family]" osVersion="*">
    <Role name="[role-name]">
    <Instances count="[number-of-instances]" />
    </Role>
    <NetworkConfiguration>

    <!--optional-->
    <Dns>
        <DnsServers><DnsServer name="[dns-server-name]" IPAddress="[ip-address]" /></DnsServers>
    </Dns>
    <!--optional-->

    <!--VNet settings
        VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.-->
    <VirtualNetworkSite name="[virtual-network-name]" />
    <AddressAssignments>
        <InstanceAddress roleName="[role-name]">
        <Subnets>
            <Subnet name="[subnet-name]" />
        </Subnets>
        </InstanceAddress>
    </AddressAssignments>
    <!--VNet settings-->

    </NetworkConfiguration>
</ServiceConfiguration>
```

A continuación, compile el proyecto de servicios en la nube e impleméntelo en Azure. Para obtener ayuda con la implementación del paquete de servicios en la nube en Azure, consulte [Creación e implementación de un servicio en la nube](cloud-services-how-to-create-deploy-portal.md)

## <a name="connect-your-webworker-roles-to-the-domain"></a>Conexión de roles de web y de trabajo al dominio
Una vez implementado el proyecto de servicio en la nube en Azure, conecte las instancias de rol al dominio de AD personalizado mediante la extensión de dominio de AD. Para agregar la extensión de dominio de AD a la implementación existente de servicios en la nube y unirse al dominio personalizado, ejecute los siguientes comandos en PowerShell:

```powershell
# Initialize domain variables

$domain = '<your-domain-name>'
$dmuser = '$domain\<your-username>'
$dmpswd = '<your-domain-password>'
$dmspwd = ConvertTo-SecureString $dmpswd -AsPlainText -Force
$dmcred = New-Object System.Management.Automation.PSCredential ($dmuser, $dmspwd)

# Add AD Domain Extension to the cloud service roles

Set-AzureServiceADDomainExtension -Service <your-cloud-service-hosted-service-name> -Role <your-role-name> -Slot <staging-or-production> -DomainName $domain -Credential $dmcred -JoinOption 35
```

Eso es todo.

Los servicios en la nube deberían combinarse con el controlador de dominio personalizado. Si desea más información acerca de las distintas opciones disponibles para configurar la extensión de dominio de AD, use la ayuda de PowerShell. A continuación verá un par de ejemplos:

```powershell
help Set-AzureServiceADDomainExtension
help New-AzureServiceADDomainExtensionConfig
```



