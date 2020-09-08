---
title: Implementación del ejemplo de plano técnico de Media
description: Pasos de implementación para el ejemplo de plano técnico de Media, incluidos los detalles de los parámetros del artefacto de plano técnico.
ms.date: 08/13/2020
ms.topic: sample
ms.openlocfilehash: c8ac1726bc5007c1790f5efbc45c72f9712fcdf2
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88208432"
---
# <a name="deploy-the-media-blueprint-sample"></a>Implementación del ejemplo de plano técnico de Media

Para implementar el ejemplo de plano técnico de Media, tiene que realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de **Media** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla del estándar establecido.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios**, como "Primera versión publicada del ejemplo de plano técnico de Media". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

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

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux |Asignación de directiva |Área de trabajo de Log Analytics para máquinas virtuales Linux |Para obtener más información, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../../../../azure-monitor/learn/quick-create-workspace.md). |
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux |Asignación de directiva |Opcional: Lista de imágenes de VM que han admitido el sistema operativo Linux que se agregarán al ámbito. |Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: `[]` |
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows |Asignación de directiva |Opcional: Lista de imágenes de VM que han admitido el sistema operativo Windows que se agregarán al ámbito. |Se puede usar una matriz vacía para indicar que no hay parámetros opcionales: `[]` |
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows |Asignación de directiva |Área de trabajo de Log Analytics para máquinas virtuales Windows |Para obtener más información, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../../../../azure-monitor/learn/quick-create-workspace.md). |
|\[Versión preliminar\]: Auditoría de los controles de Media e implementación de las extensiones de VM específicas para admitir los requisitos de auditoría |Asignación de directiva |Identificación del área de trabajo de Log Analytics para el que deberían configurarse las máquinas virtuales |Este es el identificador (GUID) del área de trabajo de Log Analytics para el que deberían estar configuradas las máquinas virtuales. |
|\[Versión preliminar\]: Auditoría de los controles de Media e implementación de las extensiones de VM específicas para admitir los requisitos de auditoría |Asignación de directiva |Lista de tipos de recursos que deben tener los registros de diagnóstico habilitados |Lista de tipos de recursos para auditar si la opción de registro de diagnóstico no está habilitada. Los valores aceptables se pueden encontrar en [esquemas de los registros de diagnóstico de Azure Monitor](../../../../azure-monitor/platform/resource-logs-schema.md#service-specific-schemas). |
|\[Versión preliminar\]: Auditoría de los controles de Media e implementación de las extensiones de VM específicas para admitir los requisitos de auditoría |Asignación de directiva |Grupo de administradores |Grupo. Ejemplo: `Administrator; myUser1; myUser2` |
|\[Versión preliminar\]: Auditoría de los controles de Media e implementación de las extensiones de VM específicas para admitir los requisitos de auditoría |Asignación de directiva |Lista de usuarios que deben incluirse en el grupo de administradores de máquinas virtuales Windows |Lista separada por punto y coma de los miembros que deben incluirse en el grupo de administradores local. Ejemplo: `Administrator; myUser1; myUser2` |
|Implementar Advanced Threat Protection en las cuentas de almacenamiento |Asignación de directiva |Efecto |Puede encontrar información sobre los efectos de las directivas en [Comprender los efectos de Azure Policy](../../../policy/concepts/effects.md). |
|Implementación de auditorías en servidores SQL Server |Asignación de directiva |Valor en días para el período de retención (0 indica retención ilimitada) |Días de retención (opcional; _180_ días si no se especifica) |
|Implementación de auditorías en servidores SQL Server |Asignación de directiva |Nombre del grupo de recursos para la cuenta de almacenamiento para la auditoría del servidor de SQL |La auditoría escribe los eventos de base de datos en un registro de auditoría en la cuenta de Azure Storage (se creará una cuenta de almacenamiento en cada región en la que se cree una instancia de SQL Server que compartirán todos los servidores de esa región). Importante: para el correcto funcionamiento de la auditoría, no elimine ni cambie el nombre del grupo de recursos o de las cuentas de almacenamiento. |
|Implementación de la configuración de diagnóstico de grupos de seguridad de red |Asignación de directiva |Prefijo de la cuenta de almacenamiento para diagnósticos de grupos de seguridad de red |Este prefijo se combina con la ubicación del grupo de seguridad de red para formar el nombre de la cuenta de almacenamiento que se creó. |
|Implementación de la configuración de diagnóstico de grupos de seguridad de red |Asignación de directiva |Nombre del grupo de recursos para la cuenta de almacenamiento para el diagnóstico de grupos de seguridad de red (deben existir) |El grupo de recursos en el que se ha creado la cuenta de almacenamiento. Este grupo de recursos ya debe existir. |

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de Media, consulte los siguientes artículos para obtener información general y sobre la asignación de controles:

> [!div class="nextstepaction"]
> [Planos técnicos de Media: información general](./index.md)
> [Media blueprints - Control mapping](./control-mapping.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
