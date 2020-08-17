---
title: Implementación de una aplicación en un conjunto de escalado de máquinas virtuales de Azure
description: Obtenga información sobre cómo implementar aplicaciones en las instancias de máquinas virtuales Linux y Windows de un conjunto de escalado
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/29/2018
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: 90cda1e8d48e6c7f9e925d6d1b9e11b5d2514e93
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080529"
---
# <a name="deploy-your-application-on-virtual-machine-scale-sets"></a>Implementación de la aplicación en conjuntos de escalado de máquinas virtuales

Para ejecutar aplicaciones en las instancias de máquinas virtuales (VM) de un conjunto de escalado, primero debe instalar los componentes de la aplicación y los archivos necesarios. En este artículo se presentan distintas formas de crear una imagen de máquina virtual personalizada para las instancias de un conjunto de escalado o de ejecutar automáticamente la instalación de scripts en instancias de máquinas virtuales existentes. También puede obtener información sobre cómo administrar actualizaciones del sistema operativo de la aplicación en un conjunto de escalado.


## <a name="build-a-custom-vm-image"></a>Creación de una imagen de máquina virtual personalizada
Al usar una de las imágenes de la plataforma Azure para crear las instancias en el conjunto de escalado, no se instala ni configura software adicional. Puede automatizar la instalación de estos componentes, aunque esto se sumará al tiempo que tarden en aprovisionarse instancias de máquinas virtuales en los conjuntos de escalado. Si aplica muchos cambios de configuración a las instancias de máquinas virtuales, habrá una sobrecarga de administración con esos scripts y tareas de configuración.

Para reducir la administración de configuración y el tiempo de aprovisionamiento de una máquina virtual, puede crear una imagen de máquina virtual personalizada que esté lista para ejecutar la aplicación tan pronto como se aprovisione una instancia en el conjunto de escalado. Para obtener más información sobre cómo crear y utilizar una imagen de máquina virtual personalizada con un conjunto de escalado, vea los siguientes tutoriales:

- [CLI de Azure](tutorial-use-custom-image-cli.md)
- [Azure PowerShell](tutorial-use-custom-image-powershell.md)


## <a name="install-an-app-with-the-custom-script-extension"></a><a name="already-provisioned"></a>Instalación de una aplicación con la extensión de script personalizado
La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión. Para más información sobre cómo instalar una aplicación con una extensión de script personalizado, consulte los siguientes tutoriales:

- [CLI de Azure](tutorial-install-apps-cli.md)
- [Azure PowerShell](tutorial-install-apps-powershell.md)
- [Plantilla de Azure Resource Manager](tutorial-install-apps-template.md)


## <a name="install-an-app-to-a-windows-vm-with-powershell-dsc"></a>Instalación de una aplicación en una máquina virtual Windows con PowerShell DSC
[PowerShell Desired State Configuration (DSC)](/powershell/scripting/dsc/overview/overview) es una plataforma de administración que se usa para definir la configuración de las máquinas de destino. Las configuraciones de DSC definen lo que se debe instalar en una máquina y cómo configurar el host. Un motor de administración de configuración local (LCM) se ejecuta en cada nodo de destino que procesa las acciones requeridas en función de las configuraciones insertadas.

La extensión PowerShell DSC permite personalizar instancias de máquinas virtuales en un conjunto de escalado con PowerShell. En el ejemplo siguiente:

- Indica a las instancias de máquina virtual que descarguen un paquete de DSC de GitHub: *https://github.com/Azure-Samples/compute-automation-configurations/raw/master/dsc.zip*
- Establece la extensión para ejecutar un script de instalación: `configure-http.ps1`
- Obtiene información sobre un conjunto de escalado con [Get-AzVmss](/powershell/module/az.compute/get-azvmss)
- Aplica la extensión a las instancias de VM con [Update-AzVmss](/powershell/module/az.compute/update-azvmss)

La extensión DSC se aplica a las instancias de máquinas virtuales *myScaleSet* del grupo de recursos denominado *myResourceGroup*. Escriba sus propios nombres, como se indica a continuación:

```powershell
# Define the script for your Desired Configuration to download and run
$dscConfig = @{
  "wmfVersion" = "latest";
  "configuration" = @{
    "url" = "https://github.com/Azure-Samples/compute-automation-configurations/raw/master/dsc.zip";
    "script" = "configure-http.ps1";
    "function" = "WebsiteTest";
  };
}

# Get information about the scale set
$vmss = Get-AzVmss `
                -ResourceGroupName "myResourceGroup" `
                -VMScaleSetName "myScaleSet"

# Add the Desired State Configuration extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
    -VirtualMachineScaleSet $vmss `
    -Publisher Microsoft.Powershell `
    -Type DSC `
    -TypeHandlerVersion 2.24 `
    -Name "DSC" `
    -Setting $dscConfig

# Update the scale set and apply the Desired State Configuration extension to the VM instances
Update-AzVmss `
    -ResourceGroupName "myResourceGroup" `
    -Name "myScaleSet"  `
    -VirtualMachineScaleSet $vmss
```

Si la directiva de actualización del conjunto de escalado es *manual*, actualice las instancias de VM con [Update-AzVmssInstance](/powershell/module/az.compute/update-azvmssinstance). Este cmdlet aplica la configuración del conjunto de escalado actualizado a las instancias de máquinas virtuales e instala la aplicación.


## <a name="install-an-app-to-a-linux-vm-with-cloud-init"></a>Instalación de una aplicación en una máquina virtual Linux con cloud-init
[cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) es un enfoque ampliamente usado para personalizar una máquina virtual Linux la primera vez que se arranca. Puede usar cloud-init para instalar paquetes y escribir archivos o para configurar los usuarios y la seguridad. Como cloud-init se ejecuta durante el proceso de arranque inicial, no hay pasos adicionales o agentes requeridos que aplicar a la configuración.

cloud-init también funciona entre distribuciones. Por ejemplo, no use **apt-get install** o **yum install** para instalar un paquete. En su lugar, puede definir una lista de paquetes que se van a instalar. Cloud-init usará automáticamente la herramienta de administración de paquetes nativos para la distribución de Linux (distro) que seleccione.

Para obtener más información, incluido un archivo *cloud-init.txt* de ejemplo, consulte [Uso de cloud-init para personalizar máquinas virtuales de Azure](../virtual-machines/linux/using-cloud-init.md).

Para crear un conjunto de escalado y usar un archivo cloud-init, agregue el parámetro `--custom-data` al comando [az vmss create](/cli/azure/vmss) y especifique el nombre de un archivo cloud-int. En el siguiente ejemplo se crea un conjunto de escalado denominado *myScaleSet* en *myResourceGroup* y configuran instancias de máquinas virtuales con un archivo denominado *cloud-init.txt*. Escriba sus propios nombres, como se indica a continuación:

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys
```


### <a name="install-applications-with-os-updates"></a>Instalación de aplicaciones con actualizaciones del sistema operativo
Si hay nuevas versiones del sistema operativo disponibles, puede usar o crear una nueva imagen personalizada e [implementar actualizaciones del sistema operativo](virtual-machine-scale-sets-upgrade-scale-set.md) en un conjunto de escalado. Cada instancia de máquinas virtuales se actualiza a la última imagen especificada. Puede usar una imagen personalizada con la aplicación preinstalada, la extensión de script personalizado o PowerShell DSC para que la aplicación esté automáticamente disponible a medida que realice la actualización. Es posible que sea necesaria la planeación del mantenimiento de la aplicación a medida que realice este proceso para garantizar que no haya problemas de compatibilidad de versión.

Si usa una imagen de máquina virtual personalizada con la aplicación preinstalada, podría integrar las actualizaciones de la aplicación con una canalización de implementación para crear las nuevas imágenes e implementar actualizaciones del sistema operativo en el conjunto de escalado. Este enfoque permite a la canalización elegir las últimas compilaciones de aplicación, crear y validar una imagen de máquina virtual y, a continuación, actualizar las instancias de máquinas virtuales del conjunto de escalado. Para ejecutar una canalización de implementación que cree e implemente actualizaciones de la aplicación en imágenes de máquina virtual personalizadas, podría [crear una imagen de Packer e implementarla con Azure DevOps Services](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset), o bien usar otra plataforma, como [Spinnaker](https://www.spinnaker.io/) o [Jenkins](https://jenkins.io/).


## <a name="next-steps"></a>Pasos siguientes
Al crear e implementar aplicaciones en los conjuntos de escalado, puede revisar [Scale Set Design Overview](virtual-machine-scale-sets-design-overview.md) (Información general sobre el diseño del conjunto de escalado). Para obtener más información sobre cómo administrar el conjunto de escalado, consulte [Use PowerShell to manage your scale set](./virtual-machine-scale-sets-manage-powershell.md) (Uso de PowerShell para administrar el conjunto de escalado).
