---
title: Protección de los nuevos recursos con bloqueos de recursos de plano técnico
description: Aprenda a usar los bloqueos de recursos de Azure Blueprints Solo lectura y No eliminar para proteger los recursos recién implementados.
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/28/2019
ms.topic: tutorial
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: d315fb5fe3ce7844946e6a9405a9a5f6a0be8b9d
ms.sourcegitcommit: c174d408a5522b58160e17a87d2b6ef4482a6694
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59791616"
---
# <a name="protect-new-resources-with-azure-blueprints-resource-locks"></a>Protección de los nuevos recursos con bloqueos de recursos de Azure Blueprints

Los [bloqueos de recursos](../concepts/resource-locking.md) de Azure Blueprints impedirán la alteración de los recursos recién implementados, incluso por parte de una cuenta con el rol de _propietario_. Esta protección puede incorporarse a los recursos creados por un artefacto de plantilla de Resource Manager en la definición del plano técnico.

Se tratan los siguientes pasos:

> [!div class="checklist"]
> - Creación de una nueva definición del plano técnico
> - Marcado de la definición del plano técnico como **Publicado**
> - Asignación de la definición del plano técnico a una suscripción existente
> - Inspección del nuevo grupo de recursos
> - Anulación de la asignación del plano técnico para quitar los bloqueos

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-new-blueprint-definition"></a>Creación de una nueva definición del plano técnico

En primer lugar, cree la nueva definición del plano técnico.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque la muestra de plano técnico **Plano técnico en blanco** en la parte superior de la página y seleccione **Empezar con un plano técnico en blanco**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia de la muestra de plano técnico. Para este tutorial, usaremos el nombre _locked-storageaccount_.
   - **Descripción del plano técnico**: describe la definición del plano técnico. Use "For testing blueprint resource locking on deployed resources" (Para probar el bloqueo de recursos del plano técnico en los recursos implementados).
   - **Ubicación de definición**: use los puntos suspensivos y seleccione la suscripción o grupo de administración donde se va a guardar la definición del plano técnico.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Agregue un grupo de recursos a la suscripción: seleccione la fila **+ Agregar artefacto...** en **Suscripción**.
   Seleccione "Grupo de recursos" para _Tipo de artefacto_. Establezca el _nombre para mostrar el artefacto_ en **RGtoLock**.
   Deje los campos _Nombre del grupo de recursos_ y _Ubicación_ en blanco, pero asegúrese de que la casilla esté activada en cada propiedad para hacerlos **parámetros dinámicos**. Haga clic en **Agregar** para agregar este artefacto al plano técnico.

1. Agregue la plantilla en el grupo de recursos: seleccione la fila **+ Agregar artefacto...** en la entrada **RGtoLock**. Seleccione "Plantilla de Azure Resource Manager" para _Tipo de artefacto_, establezca _Nombre para mostrar del artefacto_ en "Cuenta de almacenamiento" y deje _Descripción_ en blanco. En la pestaña **Plantilla** en el cuadro del editor, pegue la siguiente plantilla de Resource Manager. Una vez pegada la plantilla, seleccione **Agregar** para agregar este artefacto al plano técnico.

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
           "storageAccountType": {
               "type": "string",
               "defaultValue": "Standard_LRS",
               "allowedValues": [
                   "Standard_LRS",
                   "Standard_GRS",
                   "Standard_ZRS",
                   "Premium_LRS"
               ],
               "metadata": {
                   "description": "Storage Account type"
               }
           }
       },
       "variables": {
           "storageAccountName": "[concat('store', uniquestring(resourceGroup().id))]"
       },
       "resources": [{
           "type": "Microsoft.Storage/storageAccounts",
           "name": "[variables('storageAccountName')]",
           "location": "[resourceGroup().location]",
           "apiVersion": "2018-07-01",
           "sku": {
               "name": "[parameters('storageAccountType')]"
           },
           "kind": "StorageV2",
           "properties": {}
       }],
       "outputs": {
           "storageAccountName": {
               "type": "string",
               "value": "[variables('storageAccountName')]"
           }
       }
   }
   ```

1. Seleccione **Guardar borrador** en la parte inferior de la página.

En este paso se crea la definición del plano técnico en la suscripción o grupo de administración seleccionados.

Una vez que aparezca la notificación del portal **La definición del plano técnico se guardó correctamente**, vaya al paso siguiente.

## <a name="publish-the-blueprint-definition"></a>Publicación de la definición del plano técnico

Ahora su definición del plano técnico se ha creado en su entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico _locked-storageaccount_ y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En el nuevo panel de la derecha, proporcione la **versión** _1.0_. Esta propiedad es útil si realiza una modificación posteriormente. Proporcione **Notas de cambios** como "First version published for locking blueprint deployed resources" (Primera versión publicada para bloquear los recursos implementados del plano técnico). A continuación, seleccione **Publicar** en la parte inferior de la página.

Este paso permite la asignación del plano técnico a una suscripción. Tras su publicación, aún es posible realizar cambios. Los cambios adicionales requieren la publicación con un nuevo valor de **versión** para realizar un seguimiento de las diferencias entre las distintas versiones de la misma definición del plano técnico.

Una vez que aparezca la notificación del portal **La definición del plano técnico se publicó correctamente**, vaya al paso siguiente.

## <a name="assign-the-blueprint-definition"></a>Asignación de la definición del plano técnico

Una vez que se haya **publicado** correctamente la definición del plano técnico, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la definición del plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico _locked-storageaccount_ y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o más de las suscripciones que están en el grupo de administración donde guardó la definición del plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena previa y automáticamente en función del nombre de la definición del plano técnico. Queremos esta asignación para representar el bloqueo del nuevo grupo de recursos, de modo que cambie el nombre de asignación por _assignment-locked-storageaccount-TestingBPLocks_.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
       Para este tutorial, seleccione _Este de EE. UU. 2_.
     - **Versión de definición de Blueprint**: elija la versión **publicada** _1.0_ de la definición del plano técnico.

   - Asignación de bloqueo

     Seleccione el modo de bloqueo de plano técnico _Solo lectura_. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción _Asignado por el sistema_ predeterminada. Para más información, vea [Identidades administradas](../../../active-directory/managed-identities-azure-resources/overview.md).

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para cada artefacto, establezca el valor del parámetro en lo que se define en la columna **Valor**.

     |Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Valor|DESCRIPCIÓN|
     |-|-|-|-|-|
     |Grupo de recursos de RGtoLock|Grupos de recursos|NOMBRE|TestingBPLocks|Define el nombre del nuevo grupo de recursos al que se aplican los bloqueos de plano técnico.|
     |Grupo de recursos de RGtoLock|Grupos de recursos|Ubicación|Oeste de EE. UU. 2|Define la ubicación del nuevo grupo de recursos al que se aplican los bloqueos de plano técnico.|
     |StorageAccount|Plantilla de Resource Manager|storageAccountType (StorageAccount)|Standard_GRS|Seleccione la SKU de almacenamiento. El valor predeterminado es _Standard_LRS_.|

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página.

Este paso implementa los recursos definidos y configura la **asignación de bloqueo** seleccionada. Los bloqueos de plano técnico pueden tardar hasta 30 minutos en aplicarse.

Una vez que aparezca la notificación del portal **La definición del plano técnico se asignó correctamente**, vaya al paso siguiente.

## <a name="inspect-resources-deployed-by-the-assignment"></a>Inspección de los recursos implementados por la asignación

La asignación creó el grupo de recursos _TestingBPLocks_ y la cuenta de almacenamiento implementados por el artefacto de plantilla de Resource Manager. El nuevo grupo de recursos y el estado de bloqueo seleccionado se muestran en la página de detalles de asignación.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación de plano técnico _assignment-locked-storageaccount-TestingBPLocks_ y, a continuación, selecciónela.

   En esta página, podemos ver que la asignación se realizó correctamente y los recursos se implementaron con el nuevo estado de bloqueo de plano técnico. Si se actualiza la asignación, el menú desplegable **Operación de asignación** muestra detalles sobre la implementación de cada versión de definición. Se puede hacer clic en el grupo de recursos para abrir directamente la página de propiedades.

1. Seleccione el grupo de recursos **TestingBPLocks**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, a continuación, la pestaña **Asignaciones de roles**.

   Aquí vemos que la asignación de plano técnico _assignment-locked-storageaccount-TestingBPLocks_ tiene el rol de _propietario_ tal como se usó para implementar y bloquear el grupo de recursos.

1. Seleccione la pestaña **Asignaciones de denegación**.

   La asignación de plano técnico creó una [asignación de denegación](../../../role-based-access-control/deny-assignments.md) en el grupo de recursos implementados para forzar el modo de bloqueo de plano técnico _Solo lectura_. La asignación de denegación evita que alguien con los derechos adecuados en la pestaña _Asignaciones de roles_ tome medidas específicas. La asignación de denegación afecta a _Todas las entidades de seguridad_.

   Para información acerca de cómo excluir una entidad de seguridad de una asignación de denegación, consulte cómo [bloquear los recursos de planos técnicos](../concepts/resource-locking.md#exclude-a-principal-from-a-deny-assignment).

1. Seleccione la asignación de denegación y, a continuación, seleccione la página **Permisos denegados** de la izquierda.

   La asignación de denegación evita todas las operaciones con la configuración de **\*** y **acción**, pero permite el acceso de lectura excluyendo **\*/read** a través de **NotActions**.

1. En la ruta de navegación de Azure Portal, seleccione **TestingBPLocks - Control de acceso (IAM)**. A continuación, seleccione la página **Información general** de la izquierda y, a continuación, el botón **Eliminación de un grupo de recursos**. Escriba el nombre _TestingBPLocks_ para confirmar la eliminación y seleccione **Eliminar** en la parte inferior del panel.

   Se muestra la notificación del portal **Delete resource group TestingBPLocks failed** (Error al eliminar el grupo de recursos TestingBPLocks). El error indica que, aunque su cuenta tiene permiso para eliminar el grupo de recursos, la asignación de plano técnico deniega el acceso. Recuerde que seleccionamos el modo de bloqueo de plano técnico _Solo lectura_ durante la asignación de plano técnico. El bloqueo de plano técnico impide que una cuenta con permiso, incluso el _propietario_, elimine el recurso. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

En estos pasos se muestra que ahora nuestros recursos implementados están protegidos con bloqueos de plano técnico que evitaron la eliminación no deseada, incluso desde una cuenta con permiso.

## <a name="unassign-the-blueprint"></a>Cancelación de la asignación del plano técnico

El último paso consiste en quitar la asignación de la definición del plano técnico. Al quitarse la asignación no se quitan los artefactos modificados.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación de plano técnico _assignment-locked-storageaccount-TestingBPLocks_ y, a continuación, selecciónela.

1. Seleccione el botón **Cancelar la asignación de plano técnico** en la parte superior de la página. Lea la advertencia en el cuadro de diálogo de confirmación y, a continuación, seleccione **Aceptar**.

   Con la asignación de plano técnico quitada, también se quitan los bloqueos de plano técnico. Los recursos creados pueden eliminarse una vez más por medio de una cuenta con permisos.

1. Seleccione **Grupos de recursos** en el menú de Azure y, a continuación, seleccione **TestingBPLocks**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, a continuación, la pestaña **Asignaciones de roles**.

La seguridad para el grupo de recursos muestra que la asignación de plano técnico ya no tiene acceso de _propietario_.

Una vez que aparezca la notificación del portal **La eliminación de la asignación del plano técnico se completó correctamente**, vaya al paso siguiente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando termine con este tutorial, elimine los siguientes recursos:

- Grupo de recursos _TestingBPLocks_
- Definición del plano técnico _locked-storageaccount_

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [ciclo de vida del plano técnico](../concepts/lifecycle.md)
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.