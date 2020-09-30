---
title: 'Inicio rápido: implementación de un contenedor de Docker en una instancia de contenedor (PowerShell)'
description: En este inicio rápido, usará Azure PowerShell para implementar rápidamente una aplicación web en contenedores que se ejecuta en una instancia de contenedor aislada de Azure
services: container-instances
manager: gwallace
ms.service: container-instances
ms.topic: quickstart
ms.date: 03/21/2019
ms.custom: seodec18, mvc
ms.openlocfilehash: c7002d8a83e58a9089ee3c3840b0397d63e2f198
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89565589"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-azure-powershell"></a>Inicio rápido: Implementación de una instancia de contenedor en Azure mediante Azure PowerShell

Use Azure Container Instances para ejecutar contenedores de Docker sin servidor en Azure con sencillez y velocidad. Implemente una aplicación en una instancia de contenedor a petición cuando no necesite una plataforma de orquestación de contenedores completa, como Azure Kubernetes Service.

En esta guía de inicio rápido, va a usar Azure PowerShell para implementar un contenedor de Windows aislado y hacer que su aplicación esté disponible con un nombre de dominio completo (FQDN). Unos pocos segundos después de ejecutar un comando de implementación único, puede ir a la aplicación que se ejecuta en el contenedor:

![Aplicación implementada en Azure Container Instances vista en el explorador][qs-powershell-01]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell localmente, para este tutorial se requiere el módulo de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Las instancias de contenedores de Azure, al igual que otros recursos de Azure, se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.

Primero, cree un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus* con el siguiente comando [New-AzResourceGroup][New-AzResourceGroup]:

 ```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-container"></a>Crear un contenedor

Ahora que tiene un grupo de recursos, puede ejecutar un contenedor en Azure. Para crear una instancia de contenedor con Azure PowerShell, proporcione un nombre de grupo de recursos, un nombre de instancia de contenedor y una imagen de contenedor de Docker al cmdlet [New-AzContainerGroup][New-AzContainerGroup]. En este inicio rápido, usará la imagen `mcr.microsoft.com/windows/servercore/iis:nanoserver` pública. Esta imagen empaqueta Microsoft Internet Information Services (IIS) para ejecutarlo en Nano Server.

Puede exponer los contenedores en Internet mediante la especificación para que se abran uno o varios puertos, o la especificación de una etiqueta de nombre DNS o ambas. En este artículo de inicio rápido, se implementa un contenedor con una etiqueta de nombre DNS para que IIS sea públicamente accesible.

Ejecute un comando similar al siguiente para iniciar una instancia de contenedor. Establezca un valor `-DnsNameLabel` que sea único dentro de la región de Azure en la que cree la instancia. Si recibe un mensaje de error "DNS name label not available" (La etiqueta de nombre DNS no está disponible), pruebe otra etiqueta de nombre DNS diferente.

 ```azurepowershell-interactive
New-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer -Image mcr.microsoft.com/windows/servercore/iis:nanoserver -OsType Windows -DnsNameLabel aci-demo-win
```

Al cabo de unos segundos, debe recibir una respuesta de Azure. El estado `ProvisioningState` del contenedor es inicialmente **En creación**, pero debería cambiar a **Correcto** en un minuto o dos. Compruebe el estado de la implementación mediante el cmdlet [Get-AzContainerGroup][Get-AzContainerGroup]:

 ```azurepowershell-interactive
Get-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer
```

El estado de aprovisionamiento, el nombre de dominio completo (FQDN) y la dirección IP del contenedor aparecen en la salida del cmdlet:

```console
PS Azure:\> Get-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer


ResourceGroupName        : myResourceGroup
Id                       : /subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/mycontainer
Name                     : mycontainer
Type                     : Microsoft.ContainerInstance/containerGroups
Location                 : eastus
Tags                     :
ProvisioningState        : Creating
Containers               : {mycontainer}
ImageRegistryCredentials :
RestartPolicy            : Always
IpAddress                : 52.226.19.87
DnsNameLabel             : aci-demo-win
Fqdn                     : aci-demo-win.eastus.azurecontainer.io
Ports                    : {80}
OsType                   : Windows
Volumes                  :
State                    : Pending
Events                   : {}
```

Una vez que el estado `ProvisioningState` del contenedor es **Correcto**, vaya a su `Fqdn` en el explorador. Si ve una página web parecida a la siguiente, enhorabuena. Ha implementado correctamente una aplicación que se ejecuta en un contenedor de Docker en Azure.

![IIS implementado mediante Azure Container Instances visualizado en el explorador][qs-powershell-01]

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con el contenedor, elimínelo con el cmdlet [Remove-AzContainerGroup][Remove-AzContainerGroup]:

 ```azurepowershell-interactive
Remove-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una instancia de contenedor de Azure a partir de una imagen en un repositorio público de Docker Hub. Si quiere compilar una imagen de contenedor e implementarla desde un registro de contenedor privado de Azure, vaya al tutorial de Azure Container Instances.

> [!div class="nextstepaction"]
> [Tutorial de Azure Container Instances](./container-instances-tutorial-prepare-app.md)

<!-- IMAGES -->
[qs-powershell-01]: ./media/container-instances-quickstart-powershell/qs-powershell-01.png

<!-- LINKS -->
[New-AzResourceGroup]: /powershell/module/az.resources/new-Azresourcegroup
[New-AzContainerGroup]: /powershell/module/az.containerinstance/new-Azcontainergroup
[Get-AzContainerGroup]: /powershell/module/az.containerinstance/get-Azcontainergroup
[Remove-AzContainerGroup]: /powershell/module/az.containerinstance/remove-Azcontainergroup
