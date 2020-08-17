---
title: Programación de tareas y flujos de trabajo periódicos
description: Programación y ejecución de tareas y flujos de trabajo automatizados periódicos con el desencadenador de periodicidad en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: 8706ae12d45b9c6667ae99078d479f0e907840fc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87007562"
---
# <a name="create-schedule-and-run-recurring-tasks-and-workflows-with-the-recurrence-trigger-in-azure-logic-apps"></a>Creación, programación y ejecución de tareas y flujos de trabajo periódicos con el desencadenador de periodicidad en Azure Logic Apps

Para ejecutar con regularidad las tareas, procesos o trabajos en una programación específica, puede iniciar el flujo de trabajo de la aplicación lógica con el desencadenador de **periodicidad-programación** integrado. Puede establecer una fecha y hora, así como una zona horaria para iniciar el flujo de trabajo y una periodicidad para la repetición de ese flujo de trabajo. Si se pierden las periodicidades por alguna razón, por ejemplo, debido a interrupciones o flujos de trabajo deshabilitados, este desencadenador no procesa las periodicidades perdidas, sino que las reinicia en el siguiente intervalo programado. Para obtener más información sobre los desencadenadores y las acciones de programación integrados, consulte [Programar y ejecutar tareas y flujos de trabajo automatizados y periódicos con Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

A continuación se muestran algunos patrones compatibles con este desencadenador junto con periodicidades más avanzadas y programaciones complejas:

* Ejecutar inmediatamente y repetir cada *n* segundos, minutos, horas, días, semanas o meses.

* Empezar a una fecha y hora específicas, y después ejecutar y repetir cada *n* segundos, minutos, horas, días, semanas o meses.

* Ejecutar y repetir una o varias veces al día, por ejemplo, a las 8:00 a. m. y a las 5:00 p. m.

* Ejecutar y repetir cada semana, pero solo días específicos, como el sábado y domingo.

* Ejecutar y repetir cada semana, pero solo días específicos a horas concretas, como de lunes a viernes, a las 8:00 a. m. y a las 5:00 p. m.

Para ver las diferencias entre este desencadenador y el desencadenador de ventana deslizante, o para obtener más información sobre la programación de flujos de trabajo periódicos, consulte [Programación y ejecución de tareas, procesos y flujos de trabajo automatizados periódicos con Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

> [!TIP]
> Si desea desencadenar la aplicación lógica y ejecutarla solo una vez en el futuro, consulte [Ejecución de trabajos una sola vez](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#run-once).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Conocimientos básicos sobre [aplicaciones lógicas](../logic-apps/logic-apps-overview.md). Si es la primera vez que interactúa con las aplicaciones lógicas, consulte el artículo sobre [cómo crear la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-recurrence-trigger"></a>Adición de desencadenador "Periodicidad"

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Crear una aplicación lógica en blanco.

1. Cuando se abra el Diseñador de aplicaciones lógicas, en el cuadro de búsqueda, especifique `recurrence` como filtro. En la lista de desencadenadores, seleccione este desencadenador como primer paso en el flujo de trabajo de la aplicación lógica: **Periodicidad**

   ![Seleccionar desencadenador "Periodicidad"](./media/connectors-native-recurrence/add-recurrence-trigger.png)

1. Defina el intervalo y la frecuencia de periodicidad. En este ejemplo, defina estas propiedades para ejecutar el flujo de trabajo todas las semanas.

   ![Establecer el intervalo y la frecuencia](./media/connectors-native-recurrence/recurrence-trigger-details.png)

   | Propiedad | Nombre JSON | Obligatorio | Tipo | Descripción |
   |----------|-----------|----------|------|-------------|
   | **Intervalo** | `interval` | Sí | Entero | Entero positivo que describe la frecuencia con la que se ejecuta el flujo de trabajo. Estos son los intervalos mínimo y máximo: <p>- Mes: 1-16 meses <br>- Semana: 1-71 semanas <br>- Día: 1-500 días <br>- Hora: 1-12 000 horas <br>- Minuto: 1-72 000 minutos <br>- Segundo: 1-9 999 999 segundos<p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. |
   | **Frecuencia** | `frequency` | Sí | String | Unidad de tiempo que se usa para la periodicidad: **Segundo**, **Minuto**, **Hora**, **Día**, **Semana** o **Mes** |
   ||||||

   > [!IMPORTANT]
   > Cuando las periodicidades no especifican opciones de programación avanzadas, las futuras periodicidades se basan en la hora de la última ejecución.
   > Las horas de inicio de estas periodicidades pueden cambiar debido a factores como la latencia durante las llamadas de almacenamiento. Para asegurarse de que la aplicación lógica no pierde una periodicidad, especialmente cuando la frecuencia se especifica en días o unidades superiores, use una de estas opciones:
   > 
   > * Proporcione una hora de inicio para la periodicidad.
   > 
   > * Especifique las horas y los minutos en que se ejecutará la periodicidad mediante las propiedades **A estas horas** y **En estos minutos**.
   > 
   > * Use el [desencadenador de ventana deslizante](../connectors/connectors-native-sliding-window.md), en lugar del de periodicidad.

1. Para definir opciones de programación avanzadas, abra la lista **Agregar nuevo parámetro**. Las opciones que seleccione aparecerán en el desencadenador después de la selección.

   ![Opciones avanzadas de programación](./media/connectors-native-recurrence/recurrence-trigger-more-options-details.png)

   | Propiedad | Nombre JSON | Obligatorio | Tipo | Descripción |
   |----------|-----------|----------|------|-------------|
   | **Zona horaria** | `timeZone` | No | String | Solo se aplica cuando se especifica una hora de inicio porque este desencadenador no acepta [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Seleccione la zona horaria que desea aplicar. |
   | **Hora de inicio** | `startTime` | No | String | Proporcione una fecha y hora de inicio en este formato: <p>AAAA-MM-DDThh:mm:ss si selecciona una zona horaria <p>O bien <p>AAAA-MM-DDThh:mm:ssZ si no selecciona una zona horaria <p>Por ejemplo, si desea la fecha del 18 de septiembre de 2020 a las 14:00, especifique entonces "2020-09-18T14:00:00" y especifique una zona horaria como "Hora estándar del Pacífico". O bien, especifique "2020-09-18T14:00:00Z" sin una zona horaria. <p>**Nota:** Esta hora de inicio tiene un máximo de 49 años en el futuro y debe seguir la [especificación de fecha y hora ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) en [formato de fecha y hora UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), pero sin una [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Si no se selecciona una zona horaria, debe agregar la letra "Z" al final sin espacios. Esta "Z" se refiere al equivalente de [hora náutica](https://en.wikipedia.org/wiki/Nautical_time). <p>Para las programaciones simples, la hora de inicio es la primera aparición, mientras que para programaciones complejas, el desencadenador no se activa antes de la hora de inicio. [ *¿De qué formas puedo usar la fecha y hora de inicio?* ](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time) |
   | **En estos días** | `weekDays` | No | Cadena o matriz de cadenas | Si selecciona "Semana", puede seleccionar uno o más días que desee ejecutar el flujo de trabajo: **Lunes**, **Martes**, **Miércoles**, **Jueves**, **Viernes**, **Sábado** y **Domingo** |
   | **A estas horas** | `hours` | No | Entero o matriz de enteros | Si selecciona "Day" o "Week", puede especificar uno o varios enteros de 0 a 23 como las horas del día en las que desea ejecutar el flujo de trabajo. <p><p>Por ejemplo, si especifica "10", "12" y "14", obtendrá las 10:00, las 12:00 y las 14:00 para las horas del día, pero los minutos del día se calcularán en función del momento en el que se inicie la periodicidad. Para establecer minutos específicos del día (por ejemplo, las 10:00, las 12:00 y las 14:00), especifique esos valores mediante la propiedad **En estos minutos**. |
   | **En estos minutos** | `minutes` | No | Entero o matriz de enteros | Si selecciona "Day" o "Week", puede seleccionar uno o varios enteros de 0 a 59 como los minutos de la hora en los que desea ejecutar el flujo de trabajo. <p>Por ejemplo, puede especificar "30" como la marca de minuto y, utilizando el ejemplo anterior para las horas del día, obtendrá 10:30 a. m., 12:30 p. m. y las 2:30 p. m. <p>**Nota**: A veces, la marca de tiempo de la ejecución desencadenada puede variar hasta 1 minuto con respecto a la hora programada. Si tiene que pasar la marca de tiempo exactamente como está programada a las acciones posteriores, puede usar expresiones de plantilla para cambiar la marca de tiempo en consecuencia. Para obtener más información, vea [Funciones de fecha y hora para expresiones](../logic-apps/workflow-definition-language-functions-reference.md#date-time-functions). |
   |||||

   Por ejemplo, supongamos que hoy es viernes, 4 de septiembre de 2020. El siguiente desencadenador de periodicidad no se inicia *antes* de la fecha y hora de inicio, establecida el viernes, 18 de septiembre de 2020, a las 8:00 a. m. PST. Sin embargo, la programación de periodicidad está establecida para las 10:30 a. m., las 12:30 p. m. y las 2:30 p. m., solo los lunes. Por tanto, el desencadenador inicia y crea una instancia de flujo de trabajo de aplicación lógica por primera vez el lunes a las 10:30 a. m. Para más información sobre cómo funcionan las horas de inicio, vea estos [ejemplos de hora de inicio](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time).

   Las futuras ejecuciones se realizan a las 12:30 p. m. y a las 2:30 p. m. ese mismo día. Cada periodicidad crea su propia instancia de flujo de trabajo. Después de eso, toda la programación se repite el próximo lunes. [ *¿Qué otros ejemplos se hay?* ](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#example-recurrences)

   ![Ejemplo de programación avanzada](./media/connectors-native-recurrence/recurrence-trigger-advanced-schedule-options.png)

   > [!NOTE]
   > El desencadenador muestra una vista previa de la periodicidad especificada solo si selecciona la frecuencia "Day" o "Week".

1. Ahora cree el flujo de trabajo restante con otras acciones. Para ver más acciones que se puedan agregar, consulte [Conectores para Azure Logic Apps](../connectors/apis-list.md).

## <a name="workflow-definition---recurrence"></a>Definición de flujo de trabajo: periodicidad

En la definición de flujo de trabajo subyacente de la aplicación lógica, que usa JSON, podrá ver la [definición del desencadenador de periodicidad](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger) con las opciones que eligió. Para ver esta definición, en la barra de herramientas del diseñador, elija **Vista Código**. Para volver al diseñador, elija la barra de herramientas del diseñador, **Diseñador**.

En este ejemplo se muestra el aspecto que podrá tener una definición de desencadenador de periodicidad en una definición de flujo de trabajo subyacente:

``` json
"triggers": {
   "Recurrence": {
      "type": "Recurrence",
      "recurrence": {
         "frequency": "Week",
         "interval": 1,
         "schedule": {
            "hours": [
               10,
               12,
               14
            ],
            "minutes": [
               30
            ],
            "weekDays": [
               "Monday"
            ]
         },
         "startTime": "2020-09-07T14:00:00Z",
         "timeZone": "Pacific Standard Time"
      }
   }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Pausa de flujos de trabajo con acciones de retraso](../connectors/connectors-native-delay.md)
* [Conectores de Logic Apps](../connectors/apis-list.md)
