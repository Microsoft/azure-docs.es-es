---
title: Implementación del ejemplo de plano técnico ISO 27001
description: Pasos de implementación para el ejemplo de plano técnico independiente ISO 27001, incluidos los detalles de los parámetros del artefacto de plano técnico.
ms.date: 07/13/2020
ms.topic: sample
ms.openlocfilehash: 6580c4f8bc66238ad45fe3d969caa7a4b3b4ce82
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006813"
---
# <a name="deploy-the-iso-27001-blueprint-sample"></a>Implementación del ejemplo de plano técnico ISO 27001

Para implementar el ejemplo de plano técnico de Azure Blueprints ISO 27001, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **ISO 27001** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico ISO 27001.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la norma ISO 27001.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios** como "Primera versión publicada del ejemplo de plano técnico según la norma ISO 27001". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de plano técnico

     Muchos de los artefactos de la definición de plano técnico usan los parámetros definidos en esta sección para proporcionar coherencia.

     - **Ubicación permitida de los recursos y grupos de recursos**: Valor que indica las ubicaciones permitidas para los grupos de recursos y los recursos.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux|Asignación de directiva|Área de trabajo de Log Analytics para VM Scale Sets (VMSS) para Linux|Si este área de trabajo está fuera del ámbito de la asignación, debe conceder manualmente los permisos de "colaborador de Log Analytics" (o similar) al identificador de la entidad de seguridad de la asignación de la directiva.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Linux que se agregarán al ámbito.|Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: \[\]|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux|Asignación de directiva|Área de trabajo de Log Analytics para máquinas virtuales Linux|Si este área de trabajo está fuera del ámbito de la asignación, debe conceder manualmente los permisos de "colaborador de Log Analytics" (o similar) al identificador de la entidad de seguridad de la asignación de la directiva.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Linux que se agregarán al ámbito.|Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: \[\]|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows|Asignación de directiva|Área de trabajo de Log Analytics para VM Scale Sets (VMSS) para Windows|Si este área de trabajo está fuera del ámbito de la asignación, debe conceder manualmente los permisos de "colaborador de Log Analytics" (o similar) al identificador de la entidad de seguridad de la asignación de la directiva.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Windows que se agregarán al ámbito.|Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: \[\]|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows|Asignación de directiva|Área de trabajo de Log Analytics para máquinas virtuales Windows|Si este área de trabajo está fuera del ámbito de la asignación, debe conceder manualmente los permisos de "colaborador de Log Analytics" (o similar) al identificador de la entidad de seguridad de la asignación de la directiva.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Windows que se agregarán al ámbito.|Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: \[\]|
|SKU de cuenta de almacenamiento permitida|Asignación de directiva|Lista de SKU de almacenamiento permitidas|Lista de las SKU que se pueden especificar para las cuentas de almacenamiento.|
|SKU de máquina virtual permitida|Asignación de directiva|Lista de SKU de máquina virtual permitidas|Lista de las SKU que se pueden especificar para las máquinas virtuales.|
|Iniciativa de plano técnico para ISO 27001|Asignación de directiva|Lista de tipos de recursos que deben tener los registros de diagnóstico habilitados|Lista de tipos de recursos para auditar si la opción de registro de diagnóstico no está habilitada. Los valores aceptables se pueden encontrar en [esquemas de los registros de diagnóstico de Azure Monitor](../../../../azure-monitor/platform/resource-logs-schema.md#service-specific-schemas).|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de ISO 27001, visite los siguientes artículos para obtener información sobre la arquitectura y la asignación de controles:

> [!div class="nextstepaction"]
> [Plano técnico para la norma ISO 27001: información general](./index.md)
> [Plano técnico para la norma ISO 27001: control de la asignación](./control-mapping.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
