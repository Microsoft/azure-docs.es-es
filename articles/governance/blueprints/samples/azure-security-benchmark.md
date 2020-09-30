---
title: Introducción al ejemplo de plano técnico de Azure Security Benchmark
description: Introducción al ejemplo de plano técnico de Azure Security Benchmark. Este ejemplo de plano técnico ayuda a los clientes a evaluar determinados controles concretos.
ms.date: 06/02/2020
ms.topic: sample
ms.openlocfilehash: 4113f350c21087dbda5f69d9c3981e2d169ffc00
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651908"
---
# <a name="azure-security-benchmark-blueprint-sample"></a>Ejemplo de plano técnico de Azure Security Benchmark

El ejemplo de plano técnico de Azure Security Benchmark proporciona límites de protección de la gobernanza mediante [Azure Policy](../../policy/overview.md) que le ayudarán a evaluar los controles específicos de [Azure Security Benchmark](../../../security/benchmarks/overview.md). Este plano técnico ayuda a los clientes a implementar un conjunto básico de directivas para cualquier arquitectura implementada de Azure en la que tengan intención de implementar controles de Azure Security Benchmark.

## <a name="control-mapping"></a>Asignación de controles

La [asignación de controles de Azure Policy](../../policy/samples/azure-security-benchmark.md) proporciona detalles sobre las definiciones de directiva incluidas en este plano técnico y cómo se asignan estas definiciones de directiva a los **dominios de cumplimiento** y los **controles** en Azure Security Benchmark. Cuando se asignan a una arquitectura, Azure Policy evalúa los recursos para detectar posibles incumplimientos de las definiciones de directiva asignadas. Para más información, consulte [Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Implementación

Para implementar el ejemplo de plano técnico de Azure Security Benchmark de Azure Blueprints, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

### <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de **Azure Security Benchmark** en _Otras muestras_ y seleccione el nombre para elegir este ejemplo.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico de Azure Security Benchmark.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

### <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la alineación con las recomendaciones de Azure Security Benchmark.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios**, como "Primera versión publicada del ejemplo de plano técnico de Azure Security Benchmark". A continuación, seleccione **Publicar** en la parte inferior de la página.

### <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

### <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de usuarios excluidos del grupo de administradores de VM de Windows|Lista separada por punto y coma de los miembros que se deben excluir en el grupo Administradores local. Por ejemplo: Administrador; miUsuario1; miUsuario2|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de usuarios que deben incluirse en el grupo de administradores de VM de Windows|Lista separada por punto y coma de los miembros que deben incluirse en el grupo de administradores local. Por ejemplo: Administrador; miUsuario1; miUsuario2|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de usuarios que *solo* debe incluir el grupo de administradores de VM Windows|Lista separada por punto y coma de todos los miembros esperados del grupo Administradores local. Por ejemplo: Administrador; miUsuario1; miUsuario2|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de regiones en las que se debe habilitar Network Watcher|Para ver una lista completa de regiones, use Get-AzLocation.|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Red virtual en la que se deben conectar las VM|Ejemplo: /subscriptions/YourSubscriptionId/resourceGroups/YourResourceGroupName/providers/Microsoft.Network/virtualNetworks/Name|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Puerta de enlace de red que deben usar las redes virtuales|Ejemplo: /subscriptions/YourSubscriptionId/resourceGroups/YourResourceGroup/providers/Microsoft.Network/virtualNetworkGateways/Name|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de identificadores de área de trabajo en los que deben conectarse los agentes de Log Analytics|Una lista separada por signos de punto y coma de los identificadores de área de trabajo a los que debe estar conectado el agente de Log Analytics|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Lista de tipos de recursos que deben tener los registros de diagnóstico habilitados|Audite la configuración de diagnóstico para los tipos de recursos seleccionados.|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Versión de PHP más reciente|Versión de PHP más reciente admitida para App Services|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Versión de Java más reciente|Versión de Java más reciente admitida para App Services|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Versión más reciente de Python para Windows|Versión de Python más reciente admitida para App Services|
|Auditar las recomendaciones de Azure Security Benchmark e implementar extensiones de VM compatibles específicas|Asignación de directiva|Versión más reciente de Python para Linux|Versión de Python más reciente admitida para App Services|

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).