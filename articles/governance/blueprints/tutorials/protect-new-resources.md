---
title: 'Tutorial: Protección de nuevos recursos con bloqueos'
description: En este tutorial, usará las opciones Solo lectura y No eliminar de los bloqueos de recursos de Azure Blueprints para proteger los recursos recién implementados.
ms.date: 08/27/2020
ms.topic: tutorial
ms.openlocfilehash: 3ed75a1dee925f2a55ac46705a171bec5fc1d30e
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048577"
---
# <a name="tutorial-protect-new-resources-with-azure-blueprints-resource-locks"></a>Tutorial: Protección de los nuevos recursos con bloqueos de recursos de Azure Blueprints

Con los [bloqueos de recursos](../concepts/resource-locking.md) de Azure Blueprints puede proteger los recursos recién implementados para que no sea puedan alterar, ni siquiera por parte de una cuenta con el rol de _propietario_. Esta protección se puede agregar a las definiciones de recursos del plano técnico creadas por un artefacto de la plantilla de Azure Resource Manager (plantilla ARM).

En este tutorial va a completar estos pasos:

> [!div class="checklist"]
> - Creación de una definición de plano técnico
> - Marcado de la definición del plano técnico como **Publicado**
> - Asignación de la definición del plano técnico a una suscripción existente
> - Inspección del nuevo grupo de recursos
> - Anular la asignación del plano técnico para quitar los bloqueos

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-a-blueprint-definition"></a>Creación de una definición de plano técnico

En primer lugar, cree la definición del plano técnico.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione **Crear** en **Crear un plano técnico**.

1. Busque la muestra de plano técnico **Plano técnico en blanco** en la parte superior de la página. Seleccione **Empezar con un plano técnico en blanco**.

1. Especifique esta información en la pestaña **Datos básicos**:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico. Para este tutorial, usaremos el nombre **locked-storageaccount**.
   - **Descripción del plano técnico**: Agregue una descripción para la definición del plano técnico. Use **For testing blueprint resource locking on deployed resources** (Para probar el bloqueo de recursos del plano técnico en los recursos implementados).
   - **Ubicación de definición**: seleccione el botón de puntos suspensivos (...) y luego seleccione la suscripción o grupo de administración donde se va a guardar la definición del plano técnico.

1. Seleccione la pestaña **Artefactos** en la parte superior de la página o seleccione **Siguiente: Artefactos** en la parte inferior de la página.

1. Agregue un grupo de recursos en el nivel de suscripción:
   1. Seleccione la fila **Agregar artefacto** en **Suscripción**.
   1. Seleccione **Grupo de recursos** en **Tipo de artefacto**.
   1. Establezca el **nombre para mostrar el artefacto** en **RGtoLock**.
   1. Deje los cuadros **Nombre del grupo de recursos** y **Ubicación** en blanco, pero asegúrese de que la casilla esté activada en cada propiedad para hacerlos **parámetros dinámicos**.
   1. Seleccione **Agregar** para agregar el artefacto al plano técnico.

1. Agregue una plantilla en el grupo de recursos:
   1. Seleccione la fila **Agregar artefacto** en la entrada **RGtoLock**.
   1. Seleccione **Plantilla de Azure Resource Manager** en **Tipo de artefacto** y en **Nombre para mostrar del artefacto** seleccione **StorageAccount** y deje **Descripción** en blanco.
   1. En la pestaña **Plantilla**, pegue la siguiente plantilla de Resource Manager en el cuadro del editor. Tras pegar la plantilla, seleccione **Agregar** para agregar el artefacto al plano técnico.

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

Después de que aparezca la notificación del portal **La definición del plano técnico se guardó correctamente**, vaya al paso siguiente.

## <a name="publish-the-blueprint-definition"></a>Publicación de la definición del plano técnico

Ahora su definición del plano técnico se ha creado en su entorno. Se crea en el modo **Borrador** y debe publicarse antes de que se pueda asignar e implementar.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico **locked-storageaccount** y, después, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En el nuevo panel de la derecha, escriba **1.0** en **Versión**. Esta propiedad es útil si se realiza algún cambio más adelante. Escriba **Notas de cambios**, como **First version published for locking blueprint deployed resources** (Primera versión publicada para bloquear los recursos implementados del plano técnico). A continuación, seleccione **Publicar** en la parte inferior de la página.

Este paso permite la asignación del plano técnico a una suscripción. Aunque se haya publicado la definición del plano técnico, se pueden hacer cambios. Si realiza cambios, es preciso publicar la definición con un valor de versión nuevo para hacer un seguimiento de las diferencias entre las distintas versiones de la misma definición del plano técnico.

Después de que aparezca la notificación del portal **La definición del plano técnico se publicó correctamente**, vaya al paso siguiente.

## <a name="assign-the-blueprint-definition"></a>Asignación de la definición del plano técnico

Después de publicar la definición del plano técnico puede asignarla a una suscripción del grupo de administración donde la guardó. En este paso se proporcionan los parámetros necesarios para hacer que cada implementación de la definición del plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico **locked-storageaccount** y, después, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - **Conceptos básicos**

     - **Suscripciones**: seleccione una o varias de las suscripciones que estén en el grupo de administración en el que se guardó la definición del plano técnico. Si selecciona varias suscripciones, se creará una asignación para cada una de ellas mediante los parámetros que especifique.
     - **Nombre de asignación**: el nombre se rellena previamente en función del nombre de la definición del plano técnico. Queremos esta asignación para representar el bloqueo del nuevo grupo de recursos, de modo que cambie el nombre de asignación por **assignment-locked-storageaccount-TestingBPLocks**.
     - **Ubicación**: seleccione la región en la que se crea la identidad administrada. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
       Para este tutorial, seleccione **Este de EE. UU. 2**.
     - **Versión de definición de Blueprint**: seleccione la versión publicada, **1.0**, de la definición del plano técnico.

   - **Asignación de bloqueo**

     Seleccione el modo de bloqueo de plano técnico **Solo lectura**. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - **Identidad administrada**

     Use la opción predeterminada: **Asignado por el sistema**. Para más información, vea [Identidades administradas](../../../active-directory/managed-identities-azure-resources/overview.md).

   - **Parámetros de artefacto**

     Los parámetros definidos en esta sección se aplican al artefacto en que se definen. Estos son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. En cada artefacto, establezca el valor del parámetro en lo que se ve en la columna **Valor**.

     |Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Value|Descripción|
     |-|-|-|-|-|
     |Grupo de recursos de RGtoLock|Resource group|Nombre|TestingBPLocks|Define el nombre del nuevo grupo de recursos al que se aplican los bloqueos de plano técnico.|
     |Grupo de recursos de RGtoLock|Resource group|Location|Oeste de EE. UU. 2|Define la ubicación del nuevo grupo de recursos al que se aplican los bloqueos de plano técnico.|
     |StorageAccount|Plantilla de Resource Manager|storageAccountType (StorageAccount)|Standard_GRS|La SKU de almacenamiento. El valor predeterminado es _Standard_LRS_.|

1. Después de que haya especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página.

Este paso implementa los recursos definidos y configura la **asignación de bloqueo** seleccionada. Pueden tardar un máximo de 30 minutos en aplicar los bloqueos del plano técnico.

Después de que aparezca la notificación del portal **La definición del plano técnico se asignó correctamente**, vaya al paso siguiente.

## <a name="inspect-resources-deployed-by-the-assignment"></a>Inspección de los recursos implementados por la asignación

La asignación crea el grupo de recursos _TestingBPLocks_ y la cuenta de almacenamiento que ha implementado la plantilla de Resource Manager. El nuevo grupo de recursos y el estado de bloqueo seleccionado se muestran en la página de detalles de asignación.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación de plano técnico **assignment-locked-storageaccount-TestingBPLocks** y, después, selecciónela.

   En esta página, se puede ver que la asignación se realizó correctamente y que los recursos se implementaron con el nuevo estado de bloqueo de plano técnico. Si se actualiza la asignación, el menú desplegable **Operación de asignación** muestra detalles sobre la implementación de cada versión de definición. Puede seleccionar el grupo de recursos para abrir la página de propiedades.

1. Seleccione el grupo de recursos **TestingBPLocks**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda. Luego, seleccione la pestaña **Asignaciones de roles**.

   Aquí vemos que la asignación de plano técnico _assignment-locked-storageaccount-TestingBPLocks_ tiene el rol de _propietario_. Tiene este rol porque este rol se usó para implementar y bloquear el grupo de recursos.

1. Seleccione la pestaña **Asignaciones de denegación**.

   La asignación de plano técnico creó una [asignación de denegación](../../../role-based-access-control/deny-assignments.md) en el grupo de recursos implementados para forzar el modo de bloqueo de plano técnico **Solo lectura**. La asignación de denegación evita que alguien con los derechos adecuados en la pestaña **Asignaciones de roles** tome medidas específicas. La asignación de denegación afecta a _Todas las entidades de seguridad_.

   Para información acerca de cómo excluir una entidad de seguridad de una asignación de denegación, consulte cómo [bloquear los recursos de planos técnicos](../concepts/resource-locking.md#exclude-a-principal-from-a-deny-assignment).

1. Seleccione la asignación de denegación y, después, seleccione la página **Permisos denegados** de la izquierda.

   La asignación de denegación evita todas las operaciones con la configuración de **\*** y **acción**, pero permite el acceso de lectura, para lo que excluye **\*/read** mediante **NotActions**.

1. En la ruta de navegación de Azure Portal, seleccione **TestingBPLocks - Control de acceso (IAM)** . A continuación, seleccione la página **Información general** de la izquierda y, después, el botón **Eliminación de un grupo de recursos**. Escriba el nombre **TestingBPLocks** para confirmar la eliminación y seleccione **Eliminar** en la parte inferior del panel.

   Aparece la notificación del portal **Delete resource group TestingBPLocks failed** (Error al eliminar el grupo de recursos TestingBPLocks). El error indica que, aunque la cuenta tiene permiso para eliminar el grupo de recursos, la asignación de plano técnico deniega el acceso. Recuerde que seleccionamos el modo de bloqueo de plano técnico **Solo lectura** durante la asignación de plano técnico. El bloqueo de plano técnico impide que una cuenta con permiso, incluso el _propietario_, elimine el recurso. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

En estos pasos se muestra que los recursos implementados ahora están protegidos con bloqueos de plano técnico que evitan la eliminación no deseada, incluso desde una cuenta con permiso para hacerlo.

## <a name="unassign-the-blueprint"></a>Cancelación de la asignación del plano técnico

El último paso consiste en quitar la asignación de la definición del plano técnico. La eliminación de la asignación no quita los artefactos asociados.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación de plano técnico **assignment-locked-storageaccount-TestingBPLocks** y, después, selecciónela.

1. Seleccione **Cancelar la asignación de plano técnico**  en la parte superior de la página. Lea la advertencia en el cuadro de diálogo de confirmación y, después, seleccione **Aceptar**.

   Cuando la asignación de plano técnico se quita, también se quitan los bloqueos de plano técnico. Los recursos pueden eliminarse una vez más por medio de una cuenta con los permisos apropiados.

1. Seleccione **Grupos de recursos** en el menú de Azure y, después, seleccione **TestingBPLocks**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, después, seleccione la pestaña **Asignaciones de roles**.

La seguridad para el grupo de recursos muestra que la asignación de plano técnico ya no tiene acceso de _propietario_.

Después de que aparezca la notificación del portal **La eliminación de la asignación del plano técnico se realizó correctamente**, vaya al paso siguiente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando finalice el tutorial, elimine estos recursos:

- Grupo de recursos _TestingBPLocks_
- Definición del plano técnico _locked-storageaccount_

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a proteger los nuevos recursos implementados con Azure Blueprints. Para más información sobre Azure Blueprints, vaya el artículo sobre el ciclo de vida de los planos técnicos.

> [!div class="nextstepaction"]
> [Más información acerca del ciclo de vida del plano técnico](../concepts/lifecycle.md)