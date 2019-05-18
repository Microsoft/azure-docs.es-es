---
title: Creación de un entorno a partir de un ejemplo de plano técnico
description: Use un ejemplo de plano técnico para crear una definición de plano técnico que configura dos grupos de recursos y configura una asignación de roles para cada uno.
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/05/2019
ms.topic: tutorial
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 4f400e45d8defc304cf58c4bd05fa19f16d0501b
ms.sourcegitcommit: c174d408a5522b58160e17a87d2b6ef4482a6694
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59785986"
---
# <a name="create-an-environment-from-a-blueprint-sample"></a>Creación de un entorno a partir de un ejemplo de plano técnico

Los planos técnicos de ejemplo proporcionan ejemplos de lo que se puede hacer con Azure Blueprints. Cada uno de ellos es un ejemplo con una intención o un propósito específicos, pero no crea un entorno completo por sí mismo. Cada uno está pensado como un punto de partida para explorar con Azure Blueprints mediante varias combinaciones de artefactos, diseños y parámetros incluidos.

El siguiente tutorial usa el plano técnico de ejemplo **Grupos de recursos con RBAC** para presentar diferentes aspectos del servicio Blueprints. Se tratan los siguientes pasos:

> [!div class="checklist"]
> - Crear una definición de plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente
> - Inspeccionar los recursos implementados para la asignación
> - Anular la asignación del plano técnico para quitar los bloqueos

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-blueprint-definition-from-sample"></a>Crear una definición de plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico. La importación crea un plano técnico en el entorno basado en el ejemplo.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **Grupos de recursos con RBAC** en _Otros ejemplos_ y seleccione **Usar esta muestra**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico. Para este tutorial, usaremos el nombre _two-rgs-with-role-assignments_.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione la suscripción o el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Este ejemplo define dos grupos de recursos con los nombres para mostrar _ProdRG_ y _PreProdRG_. El nombre final y la ubicación de cada grupo de recursos se establecen durante la asignación del plano técnico. Al grupo de recursos _ProdRG_ se le asigna el rol _Colaborador_ y al grupo de recursos _PreProdRG_ se le asignan los roles _Propietario_ y _Lectores_. Los roles asignados en la definición son estáticos, pero el usuario, la aplicación o el grupo a los que se asigna el rol se definen durante la asignación del plano técnico.

1. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

En este paso, se crea una copia de la definición del plano técnico de ejemplo en la suscripción o el grupo de administración seleccionados. La definición del plano técnico guardada se administra como cualquier plano técnico creado desde cero. Puede guardar el ejemplo en el grupo de administración o en la suscripción tantas veces como sea necesario. Sin embargo, a cada copia se le debe proporcionar un nombre único.

Una vez que aparezca la notificación del portal **La definición del plano técnico se guardó correctamente**, vaya al paso siguiente.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico puede personalizarse en el entorno y según sus necesidades. Para este tutorial, no haremos ningún cambio.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico _two-rgs-with-role-assignments_ y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En el nuevo panel de la derecha, defina la **Versión** como _1.0_ para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Proporcione **Notas de cambios** como "Primera versión publicada de los grupos de recursos con el ejemplo de plano técnico de RBAC". A continuación, seleccione **Publicar** en la parte inferior de la página.

Este paso permite la asignación del plano técnico a una suscripción. Tras su publicación, aún es posible realizar cambios. Los cambios adicionales requieren la publicación con un nuevo valor de **versión** para realizar un seguimiento de las diferencias entre las distintas versiones de la misma definición del plano técnico.

Una vez que aparezca la notificación del portal **La definición del plano técnico se publicó correctamente**, vaya al paso siguiente.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar la definición del plano técnico _two-rgs-with-role-assignments_ y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena previa y automáticamente en función del nombre de la definición del plano técnico.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
       Para este tutorial, seleccione _Este de EE. UU. 2_.
     - **Versión de definición de Blueprint**: en **Publicada**, elija la versión _1.0_ de la copia de la definición del plano técnico de ejemplo.

   - Asignación de bloqueo

     Seleccione el modo de bloqueo de plano técnico _Solo lectura_. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción _Asignado por el sistema_ predeterminada. Para más información, vea [Identidades administradas](../../../active-directory/managed-identities-azure-resources/overview.md).

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para cada artefacto, establezca el valor del parámetro en lo que se define en la columna **Valor**. Para `{Your ID}`, seleccione la cuenta de usuario de Azure.

     |Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Valor|DESCRIPCIÓN|
     |-|-|-|-|-|
     |Grupo de recursos ProdRG|Grupos de recursos|NOMBRE|ProductionRG|Define el nombre del primer grupo de recursos.|
     |Grupo de recursos ProdRG|Grupos de recursos|Ubicación|Oeste de EE. UU. 2|Define la ubicación del primer grupo de recursos.|
     |Colaborador|Asignación de roles|Usuario o grupo|{Su identificador}|Define a qué usuario o grupo conceder la asignación del rol _Colaborador_ dentro del primer grupo de recursos.|
     |Grupo de recursos PreProdRG|Grupos de recursos|NOMBRE|PreProductionRG|Define el nombre del segundo grupo de recursos.|
     |Grupo de recursos PreProdRG|Grupos de recursos|Ubicación|Oeste de EE. UU.|Define la ubicación del segundo grupo de recursos.|
     |Propietario|Asignación de roles|Usuario o grupo|{Su identificador}|Define a qué usuario o grupo conceder la asignación del rol _Propietario_ dentro del segundo grupo de recursos.|
     |Lectores|Asignación de roles|Usuario o grupo|{Su identificador}|Define a qué usuario o grupo conceder la asignación del rol _Lectores_ dentro del segundo grupo de recursos.|

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página.

Este paso implementa los recursos definidos y configura la **asignación de bloqueo** seleccionada. Los bloqueos de plano técnico pueden tardar hasta 30 minutos en aplicarse.

Una vez que aparezca la notificación del portal **La definición del plano técnico se asignó correctamente**, vaya al paso siguiente.

## <a name="inspect-resources-deployed-by-the-assignment"></a>Inspección de los recursos implementados por la asignación

La asignación del plano técnico crea los artefactos definidos en la definición del plano técnico y realiza un seguimiento de ellos. Podemos ver el estado de los recursos en la página de asignación del plano técnico y consultando los recursos directamente.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación del plano técnico _Assignment-two-rgs-with-role-assignments_ y, a continuación, selecciónela.

   En esta página, podemos ver que la asignación se realizó correctamente y la lista de los recursos creados junto con el estado de bloqueo de plano técnico. Si se actualiza la asignación, el menú desplegable **Operación de asignación** muestra detalles sobre la implementación de cada versión de definición. Se puede hacer clic en cada recurso enumerado para abrir la página de propiedades de los recursos.

1. Seleccione el grupo de recursos **ProductionRG**.

   Observamos que el nombre del grupo de recursos es **ProductionRG** y no el nombre para mostrar _ProdRG_ del artefacto. Este nombre coincide con el valor establecido durante la asignación del plano técnico.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, a continuación, la pestaña **Asignaciones de roles**.

   Aquí observamos que a su cuenta se le ha concedido el rol _Colaborador_ en el ámbito de _Este recurso_. La asignación del plano técnico _Assignment-two-rgs-with-role-assignments_ tiene el rol _Propietario_ como se usó para crear el grupo de recursos. Estos permisos también se usan para administrar recursos con los bloqueos de plano técnico configurados.

1. En la ruta de navegación de Azure Portal, seleccione **Assignment-two-rgs-with-role-assignments** para retroceder una página y, a continuación, seleccione el grupo de recursos **PreProductionRG**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, a continuación, la pestaña **Asignaciones de roles**.

   Aquí observamos que a su cuenta se le han concedido los roles _Propietario_ y _Lector_, ambos en el ámbito de _Este recurso_. La asignación de plano técnico también tiene el rol _Propietario_ como el primer grupo de recursos.

1. Seleccione la pestaña **Asignaciones de denegación**.

   La asignación de plano técnico creó una [asignación de denegación](../../../role-based-access-control/deny-assignments.md) en el grupo de recursos implementados para forzar el modo de bloqueo de plano técnico _Solo lectura_. La asignación de denegación evita que alguien con los derechos adecuados en la pestaña _Asignaciones de roles_ tome medidas específicas. La asignación de denegación afecta a _Todas las entidades de seguridad_.

1. Seleccione la asignación de denegación y, a continuación, seleccione la página **Permisos denegados** de la izquierda.

   La asignación de denegación evita todas las operaciones con la configuración de **\*** y **acción**, pero permite el acceso de lectura excluyendo **\*/read** mediante **NotActions**.

1. En la ruta de navegación de Azure Portal, seleccione **PreProductionRG - Control de acceso (IAM)**. A continuación, seleccione la página **Información general** de la izquierda y, después, el botón **Eliminación de un grupo de recursos**. Escriba el nombre _PreProductionRG_ para confirmar la eliminación y seleccione **Eliminar** en la parte inferior del panel.

   Se muestra la notificación del portal **Delete resource group PreProductionRG failed** (Error al eliminar el grupo de recursos PreProductionRG). El error indica que, aunque su cuenta tiene permiso para eliminar el grupo de recursos, la asignación de plano técnico deniega el acceso. Recuerde que seleccionamos el modo de bloqueo de plano técnico _Solo lectura_ durante la asignación de plano técnico. El bloqueo de plano técnico impide que una cuenta con permiso, incluso el _propietario_, elimine el recurso. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

En estos pasos se muestra que los recursos se crearon según lo previsto y que los bloqueos del plano técnico evitaron la eliminación no deseada, incluso desde una cuenta con permiso.

## <a name="unassign-the-blueprint"></a>Cancelación de la asignación del plano técnico

El último paso consiste en quitar la asignación del plano técnico y los recursos que esta implementó.
La eliminación de la asignación no quita los artefactos implementados.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione la página **Planos técnicos asignados** de la izquierda. Use los filtros para buscar la asignación del plano técnico _Assignment-two-rgs-with-role-assignments_ y, a continuación, selecciónela.

1. Seleccione el botón **Cancelar la asignación de plano técnico** en la parte superior de la página. Lea la advertencia en el cuadro de diálogo de confirmación y, a continuación, seleccione **Aceptar**.

   Con la asignación de plano técnico quitada, también se quitan los bloqueos de plano técnico. Los recursos creados pueden eliminarse una vez más por medio de una cuenta con permisos.

1. Seleccione **Grupos de recursos** en el menú de Azure y, a continuación, seleccione **ProductionRG**.

1. Seleccione la página **Control de acceso (IAM)** de la izquierda y, a continuación, la pestaña **Asignaciones de roles**.

La seguridad de cada grupo de recursos todavía tiene las asignaciones de roles implementadas, pero la asignación de plano técnico ya no tiene acceso de _Propietario_.

Una vez que aparezca la notificación del portal **La eliminación de la asignación del plano técnico se completó correctamente**, vaya al paso siguiente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando termine con este tutorial, elimine los siguientes recursos:

- Grupo de recursos _ProductionRG_
- Grupo de recursos _PreProductionRG_
- Definición de plano técnico _two-rgs-with-role-assignments_

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [ciclo de vida del plano técnico](../concepts/lifecycle.md)
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md)
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Más información sobre la [actualización de las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.