---
title: 'Tutorial: Instalación de aplicaciones en un conjunto de escalado con plantillas de Azure'
description: Aprenda a usar las plantillas de Azure Resource Manager para instalar aplicaciones en conjuntos de escalado de máquinas virtuales con la extensión de script personalizado.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: template
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 19eb5ae89598a0ebe040f1ffda5afd2b9e3d5e7e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059255"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-an-azure-template"></a>Tutorial: Instalación de aplicaciones en conjuntos de escalado de máquinas virtuales con una plantilla de Azure
Para ejecutar aplicaciones en las instancias de máquinas virtuales (VM) de un conjunto de escalado, primero debe instalar los componentes de la aplicación y los archivos necesarios. En un tutorial anterior, aprendió a crear y usar una imagen de máquina virtual personalizada para implementar las instancias de máquina virtual. Esta imagen personalizada incluía instalaciones y configuraciones manuales de aplicaciones. También puede automatizar la instalación de aplicaciones en un conjunto de escalado después de implementar cada instancia de máquina virtual, o actualizar una aplicación que ya se ejecuta en un conjunto de escalado. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="what-is-the-azure-custom-script-extension"></a>¿Qué es la extensión de script personalizado de Azure?
La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión.

La extensión de script personalizado se integra con las plantillas de Azure Resource Manager y también se puede usar con la CLI de Azure, Azure PowerShell, Azure Portal o la API REST. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-linux.md).

Para ver la extensión de script personalizado en acción, cree un conjunto de escalado que instala el servidor web de NGINX y genera el nombre de host de la instancia de máquina virtual del conjunto de escalado. La siguiente definición de extensión de script personalizado descarga un script de ejemplo de GitHub, instala los paquetes necesarios y, luego, escribe el nombre de host de la instancia de máquina virtual en una página HTML básica.


## <a name="create-custom-script-extension-definition"></a>Creación de una definición de extensión de script personalizado
Al definir un conjunto de escalado de máquinas virtuales con una plantilla de Azure, el proveedor de recursos *Microsoft.Compute/virtualMachineScaleSets* puede incluir una sección sobre las extensiones. En *extensionsProfile* se detalla lo que se aplica a las instancias de máquina virtual de un conjunto de escalado. Para usar la extensión de script personalizado, debe especificar un publicador *Microsoft.Azure.Extensions* y un tipo *CustomScript*.

La propiedad *fileUris* se utiliza para definir los paquetes o scripts de instalación de origen. Para iniciar el proceso de instalación, los scripts necesarios se definen en *commandToExecute*. En el ejemplo siguiente, se define un script de ejemplo de GitHub que instala y configura el servidor web NGINX:

```json
"extensionProfile": {
  "extensions": [
    {
      "name": "AppInstall",
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"
          ],
          "commandToExecute": "bash automate_nginx.sh"
        }
      }
    }
  ]
}
```

Para obtener un ejemplo completo de una plantilla de Azure que implementa un conjunto de escalado y la extensión de script personalizado, consulte [https://github.com/Azure-Samples/compute-automation-configurations/blob/master/scale_sets/azuredeploy.json](https://github.com/Azure-Samples/compute-automation-configurations/blob/master/scale_sets/azuredeploy.json). Esta plantilla de ejemplo se usa en la sección siguiente.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Vamos a usar la plantilla de ejemplo para crear un conjunto de escalado y aplicar la extensión de script personalizado. En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az group deployment create](/cli/azure/group/deployment). Cuando se le solicite, proporcione su propio nombre de usuario y la contraseña que se utilizan como credenciales para cada instancia de máquina virtual:

```azurecli-interactive
az group deployment create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/scale_sets/azuredeploy.json
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

Cada instancia de máquina virtual del conjunto de escalado descarga y ejecuta el script de GitHub. En un ejemplo más complejo, se podrían instalar varios componentes y archivos de aplicaciones. Si el conjunto de escalado se escala verticalmente, las nuevas instancias de máquina virtual aplican automáticamente la misma definición de extensión de script personalizado e instalan la aplicación necesaria.


## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para ver el servidor web en acción, obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip). En el ejemplo siguiente se obtiene la dirección IP de *myScaleSetPublicIP* que se ha creado como parte del conjunto de escalado:

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetPublicIP \
  --query [ipAddress] \
  --output tsv
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Página web básica de NGINX](media/tutorial-install-apps-template/running-nginx.png)

Deje el explorador web abierto para que pueda ver una versión actualizada en el paso siguiente.


## <a name="update-app-deployment"></a>Actualización de la implementación de aplicaciones
A lo largo del ciclo de vida de un conjunto de escalado, puede que deba implementar una versión actualizada de la aplicación. Con la extensión de script personalizado, puede hacer referencia a un script de implementación actualizado y, luego, volver a aplicar la extensión al conjunto de escalado. Cuando se creó el conjunto de escalado en un paso anterior, el parámetro *upgradePolicy* se estableció en *Automatic*. Esta configuración permite que las instancias de máquina virtual del conjunto de escalado se actualicen y apliquen automáticamente la versión más reciente de la aplicación.

Para actualizar la definición de la extensión de script personalizado, edite la plantilla para que haga referencia a un nuevo script de instalación. Debe usarse un nuevo nombre de archivo para la extensión de script personalizado para que reconozca el cambio. La extensión de script personalizado no examina el contenido del script para determinar los cambios. La definición siguiente usa un script de instalación actualizado con *_v2* anexado al nombre:

```json
"extensionProfile": {
  "extensions": [
    {
      "name": "AppInstall",
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx_v2.sh"
          ],
          "commandToExecute": "bash automate_nginx_v2.sh"
        }
      }
    }
  ]
}
```

Aplique de nuevo la configuración de la extensión de script personalizado a las instancias de máquina virtual del conjunto de escalado con [az group deployment create](/cli/azure/group/deployment). La plantilla *azuredeployv2.json* se usa para aplicar la versión actualizada de la aplicación. En la práctica, la plantilla *azuredeploy.json* existente se edita para que haga referencia al script de instalación actualizado, tal y como se muestra en la sección anterior. Cuando se le solicite, escriba el mismo nombre de usuario y contraseña que usó para crear por primera vez el conjunto de escalado:

```azurecli-interactive
az group deployment create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/scale_sets/azuredeploy_v2.json
```

Todas las instancias de máquina virtual del conjunto de escalado se actualizan automáticamente con la versión más reciente de la página web de ejemplo. Para ver la versión actualizada, actualice el sitio web en el explorador:

![Página web actualizada de NGINX](media/tutorial-install-apps-template/running-nginx-updated.png)


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group). El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a instalar y actualizar automáticamente aplicaciones en un conjunto de escalado con las plantillas de Azure:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Vaya al siguiente tutorial para aprender cómo se escala automáticamente el conjunto de escalado.

> [!div class="nextstepaction"]
> [Escalado automático de los conjuntos de escalado](tutorial-autoscale-template.md)
