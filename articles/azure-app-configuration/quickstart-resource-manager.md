---
title: Inicio rápido para la implementación automatizada de máquina virtual con Azure App Configuration
description: En este inicio rápido se muestra cómo usar el módulo Azure PowerShell y las plantillas de Azure Resource Manager para implementar un almacén de Azure App Configuration. Y a continuación, utilizar los valores del almacén para implementar una máquina virtual.
author: lisaguthrie
ms.author: lcozzens
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: azure-app-configuration
ms.custom:
- mvc
- subject-armqs
ms.openlocfilehash: 7b7dd00d3495c24733ecdc213e0e25f8bc9640eb
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661476"
---
# <a name="quickstart-automated-vm-deployment-with-app-configuration-and-resource-manager-template-arm-template"></a>Inicio rápido: Implementación automatizada de máquinas virtuales con App Configuration y la plantilla de Resource Manager

Aprenda a usar las plantillas de Azure Resource Manager y Azure PowerShell para implementar un almacén de Azure App Configuration, a agregar pares clave-valor al almacén y a usarlos en el almacén para implementar un recurso de Azure en este ejemplo una máquina virtual de Azure.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-configuration-store%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-templates"></a>Examen de las plantillas

La plantillas que se usan en este inicio rápido forman parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/). La [primera plantilla](https://azure.microsoft.com/resources/templates/101-app-configuration-store/) crea un almacén de App Configuration:

:::code language="json" source="~/quickstart-templates/101-app-configuration-store/azuredeploy.json" range="1-37" highlight="27-35":::

En la plantilla, se define un recurso de Azure:

- [Microsoft.AppConfiguration/configurationStores](/azure/templates/microsoft.appconfiguration/2019-10-01/configurationstores): cree un almacén de App Configuration.

La [segunda plantilla](https://azure.microsoft.com/resources/templates/101-app-configuration/) crea una máquina virtual con los pares clave-valor del almacén. Antes de este paso, es preciso agregar los pares clave-valor, para lo que deben usarse Azure Portal o la CLI de Azure.

:::code language="json" source="~/quickstart-templates/101-app-configuration/azuredeploy.json" range="1-217" highlight="77, 181,189":::

## <a name="deploy-the-templates"></a>Implementación de las plantillas

### <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea un almacén de App Configuration.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-configuration-store%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores.

    - **Suscripción**: seleccione la suscripción de Azure que se usa para crear el almacén de App Configuration.
    - **Grupo de recursos**: seleccione **Crear nuevo** para seleccionar un grupo de recursos, a menos que desee usar uno existente.
    - **Región**: seleccione la ubicación del grupo de recursos.  Por ejemplo **Este de EE. UU.**
    - **Nombre del almacén de configuración**: escriba el nombre de un almacén de App Configuration nuevo.
    - **Ubicación**: especifique la ubicación del almacén de App Configuration.  Use el valor predeterminado.
    - **Nombre de SKU**: especifique el nombre de la SKU del almacén de App Configuration. Use el valor predeterminado.

1. Seleccione **Revisar + crear**.
1. Compruebe que la página muestra **Validación superada** y, luego, seleccione **Crear**.

Anote el nombre del grupo de recursos y el nombre del almacén de App Configuration.  Cuando implemente la máquina virtual necesitará estos valores.
### <a name="add-vm-configuration-key-values"></a>Incorporación de pares clave-valor de configuración de máquina virtual

Tras crear un almacén de App Configuration, puede usar Azure Portal o la CLI de Azure para agregar pares clave-valor al almacén.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al almacén de App Configuration que acaba de crear.
1. Seleccione **Explorador de configuración** en el menú de la izquierda.
1. Haga clic en **Crear** para agregar los siguientes pares clave-valor:

   |Clave|Value|Etiqueta|
   |-|-|-|
   |windowsOsVersion|2019-Datacenter|template|
   |diskSizeGB|1023|template|

   Conserve **Tipo de contenido** vacío.

Para usar la CLI de Azure, consulte [Uso de pares clave-valor en un almacén de Azure App Configuration](./scripts/cli-work-with-keys.md).

### <a name="deploy-vm-using-stored-key-values"></a>Implementación de máquina virtual con pares clave-valor almacenados

Ahora que ha agregado pares clave-valor al almacén, ya está preparado para implementar una máquina virtual mediante una plantilla de Azure Resource Manager. La plantilla hace referencia a las claves **windowsOsVersion** y **diskSizeGB** que creó.

> [!WARNING]
> Las plantillas de Resource Manager no pueden hacer referencia a claves de un almacén de App Configuration que tiene habilitado Private Link.

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una máquina virtual mediante los pares clave-valor del almacén de App Configuration.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-configuration%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores.

    - **Suscripción**: seleccione la suscripción de Azure que se usa para crear la máquina virtual.
    - **Grupo de recursos**: especifique el mismo grupo de recursos que el almacén de App Configuration, o bien seleccione **Crear nuevo** para crear un grupo de recursos.
    - **Región**: seleccione la ubicación del grupo de recursos.  Por ejemplo **Este de EE. UU.**
    - **Ubicación**: especifique la ubicación de la máquina virtual. Use el valor predeterminado.
    - **Nombre de usuario del administrador**: especifique el nombre de usuario del administrador para la máquina virtual.
    - **Contraseña de administrador**: especifique la contraseña del administrador de la máquina virtual.
    - **Etiqueta de nombre de dominio**: especifique un nombre de dominio único.
    - **Nombre de la cuenta de almacenamiento**: especifique un nombre único para una cuenta de almacenamiento asociada a la máquina virtual.
    - **App Config Store Resource Group** (Grupo de recursos del almacén de App Configuration): el grupo de recursos que contiene el almacén de App Configuration.
    - **App Config Store Name** (Nombre del almacén de App Configuration): el nombre del almacén de Azure App Configuration.
    - **VM Sku Key** (Clave de SKU de VM): especifique **windowsOsVersion**.  Este es el nombre del valor de la clave que agregó al almacén.
    - **Disk Size Key** (Clave de tamaño de disco): especifique **diskSizeGB**. Este es el nombre del valor de la clave que agregó al almacén.

1. Seleccione **Revisar + crear**.
1. Compruebe que la página muestra **Validación superada** y, luego, seleccione **Crear**.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la máquina virtual que acaba de crear.
1. Seleccione **Información general** en el menú de la izquierda y compruebe que el valor de **SKU** es **2019-Datacenter**.
1. Seleccione **Discos** en el menú de la izquierda y compruebe que el tamaño del disco de datos es **2013**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine el grupo de recursos, el almacén de App Configuration y todos los recursos relacionados. Si tiene planes para utilizar el almacén de App Configuration o la máquina virtual en el futuro, puede omitir su eliminación. Si no va a seguir usando este trabajo, ejecute el siguiente cmdlet para eliminar todos los recursos creados en este inicio rápido:

```azurepowershell-interactive
Remove-AzResourceGroup `
  -Name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado una máquina virtual con una plantilla de Azure Resource Manager y pares de clave-valor de Azure App Configuration.

Para obtener información acerca de cómo crear otras aplicaciones con Azure App Configuration, continúe con el siguiente artículo:

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de una aplicación ASP.NET Core con Azure App Configuration](quickstart-aspnet-core-app.md)
