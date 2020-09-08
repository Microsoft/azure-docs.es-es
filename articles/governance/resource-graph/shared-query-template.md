---
title: 'Inicio rápido: Creación de una consulta compartida con plantillas'
description: En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear una consulta compartida de Resource Graph que cuenta las máquinas virtuales por sistema operativo.
ms.date: 07/06/2020
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: cc9da201b10b697f125e8ffe7402f23f5eaa8362
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685534"
---
# <a name="quickstart-create-a-shared-query-by-using-an-arm-template"></a>Inicio rápido: Creación de una consulta compartida mediante una plantilla de Resource Manager

Las consultas de Resource Graph se pueden guardar como una _consulta privada_ o una _consulta compartida_. Una consulta privada se guarda en el perfil del portal del usuario y no es visible para otros usuarios. Una consulta compartida es un objeto de Resource Manager que se puede compartir con otros mediante permisos y acceso basado en roles. Una consulta compartida proporciona una ejecución común y coherente de la detección de recursos. En este inicio rápido se usa una plantilla de Azure Resource Manager para crear una consulta compartida.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementar la plantilla de Resource Manager para crear una consulta compartida en Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fresourcegraph-sharedquery-countos%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

En este inicio rápido, creará una consulta compartida denominada _Recuento de máquinas virtuales por sistema operativo_. Para probar esta consulta en el SDK o en el portal con el explorador de Resource Graph, consulte [Ejemplos: recuento de máquinas virtuales por sistema operativo](./samples/starter.md#count-os).

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/resourcegraph-sharedquery-countos/).

:::code language="json" source="~/quickstart-templates/resourcegraph-sharedquery-countos/azuredeploy.json":::

El recurso definido en la plantilla es el siguiente:

- [Microsoft.ResourceGraph/queries](/azure/templates/microsoft.resourcegraph/queries)

## <a name="deploy-the-template"></a>Implementación de la plantilla

> [!NOTE]
> Azure Resource Graph es un servicio gratuito. Para más información, consulte [Información general de Azure Resource Graph](./overview.md).

1. Seleccione la siguiente imagen para iniciar sesión en Azure Portal y abrir la plantilla:

   :::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementar la plantilla de Resource Manager para crear una consulta compartida en Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fresourcegraph-sharedquery-countos%2Fazuredeploy.json":::

1. Seleccione o escriba los siguientes valores:

   | Nombre | Value |
   |------|-------|
   | Suscripción | Seleccione su suscripción a Azure. |
   | Resource group | Seleccione **Crear**, especifique un nombre y seleccione **Aceptar**. |
   | Location | Seleccione una región. Por ejemplo, **Centro de EE. UU**. |
   | Nombre de la consulta | Deje el valor predeterminado **Recuento de máquinas virtuales por sistema operativo**. |
   | Código de consulta | Deje el valor predeterminado `Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)` |
   | Descripción de la consulta | Deje el valor predeterminado **Esta consulta compartida cuenta todos los recursos de máquina virtual y resume el tipo de sistema operativo.** |
   | Acepto los términos y condiciones indicados anteriormente | (Seleccionar) |

1. Seleccione **Comprar**.

Algunos recursos adicionales:

- Puede encontrar más plantillas de ejemplo en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Para ver la referencia de plantilla, vaya a la [referencia de plantilla de Azure](/azure/templates/microsoft.resourcegraph/allversions).
- Para aprender a desarrollar plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Para información sobre la implementación de nivel de suscripción, consulte [Create resource groups and resources at the subscription level](../../azure-resource-manager/templates/deploy-to-subscription.md) (Creación de grupos de recursos y recursos en el nivel de suscripción).

## <a name="validate-the-deployment"></a>Validación de la implementación

Para ejecutar la nueva consulta compartida, siga estos pasos:

1. En la barra de búsqueda del portal, busque **Consultas de Resource Graph** y selecciónelas.

1. Seleccione la consulta compartida denominada **Recuento de máquinas virtuales por sistema operativo** y, después, seleccione la pestaña **Resultados** en la página **Información general**.

Como alternativa, la consulta compartida se puede abrir desde el explorador de Resource Graph:

1. En la barra de búsqueda del portal, busque **Explorador de Resource Graph** y selecciónelo.

1. Seleccione el botón **Abrir una consulta**.

1. Cambie **Tipo** a _Consultas compartidas_. Si no ve **Recuento de máquinas virtuales por sistema operativo** en la lista, use el cuadro de filtro para limitar los resultados. Una vez que esté visible la consulta compartida **Recuento de máquinas virtuales por sistema operativo**, seleccione su nombre.

1. Una vez cargada la consulta, seleccione el botón **Ejecutar consulta**. Los resultados se muestran en la pestaña **Resultados** que aparece a continuación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la consulta compartida creada, siga estos pasos:

1. En la barra de búsqueda del portal, busque **Consultas de Resource Graph** y selecciónelas.

1. Active la casilla situada junto a la consulta compartida denominada **Recuento de máquinas virtuales por sistema operativo**.

1. Seleccione el botón **Eliminar** en la parte superior de la página.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una consulta compartida de Resource Graph.

Para más información sobre las consultas compartidas, continúe con el tutorial:

> [!div class="nextstepaction"]
> [Administración de consultas en Azure Portal](./tutorials/create-share-query.md)