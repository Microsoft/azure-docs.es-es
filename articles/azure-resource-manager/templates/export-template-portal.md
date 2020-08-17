---
title: Exportación de plantilla en Azure Portal
description: Use Azure Portal para exportar una plantilla de Azure Resource Manager desde los recursos de la suscripción.
ms.topic: conceptual
ms.date: 07/29/2020
ms.openlocfilehash: 0262517df11f0d91920fd7e44f96ff532ffbe63f
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423242"
---
# <a name="single-and-multi-resource-export-to-a-template-in-azure-portal"></a>Exportación de uno y varios recursos a una plantilla en Azure Portal

Para ayudar con la creación de plantillas de Azure Resource Manager, puede exportar una plantilla desde los recursos existentes. La plantilla exportada le ayudará a comprender la sintaxis y las propiedades JSON que implementan los recursos. Para automatizar las implementaciones futuras, comience con la plantilla exportada y modifíquela para su escenario.

Resource Manager permite elegir uno o más recursos para exportarlos a una plantilla. Se puede centrar en los recursos exactos que necesita en la plantilla.

En este artículo se muestra cómo exportar plantillas mediante el portal. También puede usar la [CLI de Azure](../management/manage-resource-groups-cli.md#export-resource-groups-to-templates), [Azure PowerShell](../management/manage-resource-groups-powershell.md#export-resource-groups-to-templates) o la [API REST](/rest/api/resources/resourcegroups/exporttemplate).

## <a name="choose-the-right-export-option"></a>Elección de la opción de exportación correcta

Hay dos maneras de exportar una plantilla:

* **Exportar desde el grupo de recursos o el recurso**. Esta opción genera una plantilla nueva a partir de los recursos existentes. La plantilla exportada es una "instantánea" del estado actual del grupo de recursos. Puede exportar un grupo de recursos completo o recursos específicos dentro de ese grupo de recursos.

* **Exportar antes de la implementación o desde el historial**. Esta opción recupera una copia exacta de una plantilla usada para la implementación.

Según la opción que elija, las plantillas exportadas tendrán distintas calidades.

| Desde el grupo de recursos o el recurso | Antes de la implementación o desde el historial |
| --------------------- | ----------------- |
| La plantilla es una instantánea del estado actual de los recursos. Incluye todos los cambios manuales que se hayan realizado después de la implementación. | En la plantilla solo se muestra el estado de los recursos en el momento de la implementación. No se incluyen los cambios manuales que haya realizado después de la implementación. |
| Puede seleccionar los recursos de un grupo de recursos que se van a exportar. | Se incluyen todos los recursos para una implementación específica. No se puede seleccionar un subconjunto de esos recursos o agregar recursos que se hayan agregado en otro momento. |
| En la plantilla se incluyen todas las propiedades de los recursos, incluidas algunas que normalmente no se establecerían durante la implementación. Es posible que quiera quitar o borrar estas propiedades antes de volver a usar la plantilla. | En la plantilla solo se incluyen las propiedades necesarias para la implementación. La plantilla está lista para usarse. |
| Probablemente en la plantilla no se incluyan todos los parámetros que necesita para su reutilización. La mayoría de los valores de propiedad están codificados de forma rígida en la plantilla. Para volver a implementar la plantilla en otros entornos, tendrá que agregar parámetros que aumenten la capacidad de configurar los recursos.  También puede anular la selección de **Incluir parámetros** y así podrá crear sus propios parámetros. | En la plantilla se incluyen parámetros que facilitan la tarea de volver a implementar en otros entornos. |

Exporte la plantilla desde un grupo de recursos o un recurso cuando:

* Tenga que capturar cambios de los recursos que se han realizado después de la implementación original.
* Quiera seleccionar los recursos que se van a exportar.

Exporte la plantilla antes de la implementación o desde el historial cuando:

* Quiera una plantilla fácil de reutilizar.
* No necesite incluir los cambios realizados después de la implementación original.

## <a name="limitations"></a>Limitaciones

Al realizar una exportación de un grupo de recursos o un recurso, la plantilla exportada se genera a partir de los [esquemas publicados](https://github.com/Azure/azure-resource-manager-schemas/tree/master/schemas) para cada tipo de recurso. En ocasiones, el esquema no tiene la versión más reciente de un tipo de recurso. Compruebe la plantilla exportada para asegurarse de que incluye las propiedades necesarias. Si es preciso, edite la plantilla exportada para usar la versión de API que necesite.

La característica de exportación de plantillas no admite la exportación de recursos de Azure Data Factory. Para más información sobre cómo exportar recursos de Data Factory, consulte [Copia o clonación de una factoría de datos en Azure Data Factory](https://aka.ms/exportTemplateViaAdf).

Para exportar los recursos creados mediante el modelo de implementación clásica, debe [migrarlos al modelo de implementación de Resource Manager](https://aka.ms/migrateclassicresourcetoarm).

Si recibe una advertencia al exportar una plantilla que indica que un tipo de recurso no se ha exportado, aún puede detectar las propiedades de ese recurso. Para obtener información sobre las distintas opciones para ver las propiedades de los recursos, consulte [Descubra las propiedades de los recursos](view-resources.md). También puede consultar la [API de REST de Azure](/rest/api/azure/) para el tipo de recurso.

## <a name="export-template-from-a-resource-group"></a>Exportación de la plantilla desde un grupo de recursos

Para exportar uno o varios recursos de un grupo de recursos:

1. Seleccione el grupo de recursos que contiene los recursos que quiere exportar.

1. Seleccione uno o varios recursos activando las casillas.  Para seleccionar todo, active la casilla situada a la izquierda de **Nombre**. El elemento de menú **Exportar plantilla** solo se habilita después de seleccionar al menos un recurso.

   ![Exportar todos los recursos](./media/export-template-portal/select-all-resources.png)

    En la captura de pantalla, solo se selecciona la cuenta de almacenamiento.
1. Seleccione **Exportar plantilla**.

1. Se mostrará la plantilla exportada, que estará disponible para descargar e implementar.

   ![Mostrar la plantilla](./media/export-template-portal/show-template.png)

   **Incluir parámetros** está seleccionado de forma predeterminada.  Cuando se selecciona esta opción, se incluirán todos los parámetros de plantilla cuando se genere la plantilla. Si desea crear sus propios parámetros, desactive esta casilla para no incluirlos.

## <a name="export-template-from-a-resource"></a>Exportación de la plantilla desde un recurso

Para exportar un recurso:

1. Seleccione el grupo de recursos que contiene el recurso que quiere exportar.

1. Seleccione el recurso que quiere exportar para abrir el recurso.

1. Para ese recurso, seleccione **Exportar plantilla** en el panel de la izquierda.

   ![Exportación del recurso](./media/export-template-portal/export-single-resource.png)

1. Se mostrará la plantilla exportada, que estará disponible para descargar e implementar. La plantilla solo contiene ese único recurso. **Incluir parámetros** está seleccionado de forma predeterminada.  Cuando se selecciona esta opción, se incluirán todos los parámetros de plantilla cuando se genere la plantilla. Si desea crear sus propios parámetros, desactive esta casilla para no incluirlos.

## <a name="export-template-before-deployment"></a>Exportación de la plantilla antes de la implementación

1. Seleccione el servicio de Azure que quiera implementar.

1. Rellene los valores para el nuevo servicio.

1. Después de pasar la validación, pero antes de iniciar la implementación, seleccione **Descargar una plantilla para la automatización**.

   ![Descarga de una plantilla](./media/export-template-portal/download-before-deployment.png)

1. Se mostrará la plantilla y estará disponible para descargarla e implementarla.


## <a name="export-template-after-deployment"></a>Exportación de la plantilla después de la implementación

Puede exportar la plantilla que se ha usado para implementar los recursos existentes. La plantilla que se obtiene es exactamente la que se ha usado para la implementación.

1. Seleccione el grupo de recursos que quiera exportar.

1. Seleccione el vínculo situado bajo **Implementaciones**.

   ![Selección del historial de implementación](./media/export-template-portal/select-deployment-history.png)

1. Seleccione una de las implementaciones del historial de implementación.

   ![Selección de la implementación](./media/export-template-portal/select-details.png)

1. Seleccione **Plantilla**. Se mostrará la plantilla que se ha usado para esta implementación y estará disponible para descargarla.

   ![Seleccionar plantilla](./media/export-template-portal/show-template-from-history.png)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a exportar plantillas con la [CLI de Azure](../management/manage-resource-groups-cli.md#export-resource-groups-to-templates), [Azure PowerShell](../management/manage-resource-groups-powershell.md#export-resource-groups-to-templates) o la [API REST](/rest/api/resources/resourcegroups/exporttemplate).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).
