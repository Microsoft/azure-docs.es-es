---
title: Uso de Azure Monitor Logs con Azure Logic Apps y Power Automate
description: Obtenga información acerca de cómo puede utilizar Azure Logic Apps y Power Automate para automatizar rápidamente los procesos repetibles mediante el conector de Azure Monitor.
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/13/2020
ms.openlocfilehash: 2ef7095d186902425adb5065c470325be1283023
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475743"
---
# <a name="azure-monitor-logs-connector-for-logic-apps-and-power-automate"></a>Conector de Azure Monitor Logs para Logic Apps y Power Automate
[Azure Logic Apps](../../logic-apps/index.yml) y [Power Automate](https://flow.microsoft.com) le permite crear flujos de trabajo automatizados, con cientos de acciones para diversos servicios. El conector de Azure Monitor Logs le permite crear flujos de trabajo destinados a recuperar datos de un área de trabajo de Log Analytics o de una aplicación Application Insights en Azure Monitor. En este artículo se describen las acciones que se incluyen con el conector y se proporciona un tutorial para crear un flujo de trabajo con estos datos.

Por ejemplo, puede crear una aplicación lógica para usar los datos de registro de Azure Monitor incluidos en una notificación de correo electrónico de Office 365, crear un error en Azure DevOps o publicar un mensaje de Slack.  Puede desencadenar un flujo de trabajo mediante una programación simple o a partir de alguna acción de un servicio conectado, como cuando se recibe un correo electrónico o un tweet. 

## <a name="connector-limits"></a>Límites de conector
El conector de registros de Azure Monitor tiene estos límites:
* Tamaño máximo de datos: 16 MB
* Tamaño máximo de respuesta de consultas de 100 MB
* Número máximo de registros: 500.000
* Tiempo de espera máximo de consulta de 110 segundos.

En función del tamaño de los datos y de la consulta que use, el conector puede alcanzar sus límites y producir un error. Puede solucionar estos casos al ajustar la periodicidad del desencadenador para que se ejecute con más frecuencia y consulte menos datos. Puede usar consultas que agreguen los datos para devolver menos registros y columnas.

## <a name="actions"></a>Acciones
En la tabla siguiente se describen las acciones incluidas con el conector de Azure Monitor Logs. Ambos le permiten ejecutar una consulta de registro en un área de trabajo de Log Analytics o una aplicación de Application Insights. La diferencia radica en la forma en que se devuelven los datos.

> [!NOTE]
> El conector de Azure Monitor Logs reemplaza el [conector de Azure Log Analytics](/connectors/azureloganalytics/) y el [conector de Azure Application Insights](/connectors/applicationinsights/). Este conector proporciona la misma funcionalidad que los demás y es el método preferido para ejecutar una consulta en un área de trabajo de Log Analytics o una aplicación de Application Insights.


| Acción | Descripción |
|:---|:---|
| [Ejecutar la consulta y mostrar los resultados](/connectors/azuremonitorlogs/#run-query-and-list-results) | Devuelve cada fila como su propio objeto. Utilice esta acción si desea trabajar con cada fila por separado en el resto del flujo de trabajo. Normalmente, la acción va seguida de una [actividad Foreach](../../logic-apps/logic-apps-control-flow-loops.md#foreach-loop). |
| [Ejecutar la consulta y visualizar los resultados](/connectors/azuremonitorlogs/#run-query-and-visualize-results) | Devuelve todas las filas del conjunto de resultados como un objeto formateado único. Utilice esta acción si desea utilizar el conjunto de resultados en el resto del flujo de trabajo, por ejemplo para enviar los resultados en un correo.  |

## <a name="walkthroughs"></a>Tutoriales
En los siguientes tutoriales se muestra el uso de los conectores de Azure Monitor en Azure Logic Apps. Puede realizar este mismo ejemplo con Power Automate. La única diferencia es cómo crear el flujo de trabajo inicial y ejecutarlo cuando haya finalizado. La configuración del flujo de trabajo y las acciones es la misma en ambos casos. Consulte [Creación de un flujo desde una plantilla en Power Automate](/power-automate/get-started-logic-template) para empezar.


### <a name="create-a-logic-app"></a>Crear una aplicación lógica

Vaya a **Logic Apps** en Azure Portal y haga clic en **Agregar**. Seleccione las opciones de **Suscripción**, **Grupo de recursos** y **Región** para almacenar la nueva aplicación lógica y, a continuación, asígnele un nombre único. Puede activar la opción de configuración de **Log Analytics** para recopilar información sobre los eventos y datos en tiempo de ejecución, tal y como se describe en [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](../../logic-apps/monitor-logic-apps-log-analytics.md). Esta configuración no es necesaria para usar el conector de Azure Monitor Logs.

![Creación de la aplicación lógica](media/logicapp-flow-connector/create-logic-app.png)


Seleccione **Revisar y crear** y, a continuación, **Crear**. Cuando la implementación finalice, haga clic en **Ir al recurso** para abrir el **Diseñador de aplicaciones lógicas**.

### <a name="create-a-trigger-for-the-logic-app"></a>Creación de un desencadenador para la aplicación lógica
En **Empezar con un desencadenador común**, seleccione **Periodicidad**. Esta acción crea una aplicación lógica que se ejecuta automáticamente a intervalos regulares. En el cuadro **Frecuencia** de la acción, seleccione **Día** y, en el cuadro **Intervalo**, escriba **1** para ejecutar el flujo de trabajo una vez al día.

![Acción de periodicidad](media/logicapp-flow-connector/recurrence-action.png)

## <a name="walkthrough-mail-visualized-results"></a>Tutorial: Envío de los resultados visualizados por correo
En el siguiente tutorial se muestra cómo crear una aplicación lógica que envía los resultados de una consulta de registro de Azure Monitor por correo electrónico. 

### <a name="add-azure-monitor-logs-action"></a>Adición de una acción de Azure Monitor Logs
Haga clic en **+ Nuevo paso** para agregar una acción que se ejecute después de la acción de periodicidad. En **Elegir una acción**, escriba **azure monitor** y, a continuación, seleccione **Azure Monitor Logs**.

![Acción de Azure Monitor Logs](media/logicapp-flow-connector/select-azure-monitor-connector.png)

Haga clic en **Azure Log Analytics: ejecutar consulta y ver resultados**.

![Acción Ejecutar la consulta y visualizar los resultados](media/logicapp-flow-connector/select-query-action-visualize.png)


### <a name="add-azure-monitor-logs-action"></a>Adición de una acción de Azure Monitor Logs

Seleccione las opciones de **Suscripción** y **Grupo de recursos** para su área de trabajo de Log Analytics. Seleccione *Área de trabajo de Log Analytics* en **Tipo de recurso** y, a continuación, seleccione el nombre del área de trabajo en **Nombre del recurso**.

En la ventana **Consulta**, agregue la siguiente consulta de registro.  

```Kusto
Event
| where EventLevelName == "Error" 
| where TimeGenerated > ago(1day)
| summarize TotalErrors=count() by Computer
| sort by Computer asc   
```

Seleccione *Establecer en la consulta* en **Intervalo de tiempo** y **Tabla HTML** en **Tipo de gráfico**.
   
![Acción Ejecutar la consulta y visualizar los resultados](media/logicapp-flow-connector/run-query-visualize-action.png)

La cuenta asociada a la conexión actual enviará el correo. Puede hacer clic en **Cambiar conexión** para especificar otra cuenta.

### <a name="add-email-action"></a>Adición de una acción de correo electrónico

Haga clic en **+ Nuevo paso** y, a continuación, en **+ Agregar una acción**. En **Elegir una acción**, escriba **outlook** y, a continuación, seleccione **Office 365 Outlook**.

![Selección del conector de Outlook](media/logicapp-flow-connector/select-outlook-connector.png)

Seleccione **Enviar correo electrónico (V2)** .

![Ventana de selección de Office 365 Outlook](media/logicapp-flow-connector/select-mail-action.png)

Haga clic en cualquier lugar del cuadro **Cuerpo** para abrir una ventana **Contenido dinámico** con los valores de las acciones anteriores en la aplicación lógica. Seleccione **Ver más** y, a continuación, **Cuerpo**, que es el resultado de la consulta de la acción de Log Analytics.

![Selección de cuerpo](media/logicapp-flow-connector/select-body.png)

Especifique la dirección de correo electrónico de un destinatario en la ventana **Para** y un asunto para el correo electrónico en **Asunto**. 

![Acción de correo](media/logicapp-flow-connector/mail-action.png)


### <a name="save-and-test-your-logic-app"></a>Guardado y prueba de la aplicación lógica
Haga clic en **Guardar** y, a continuación, en **Ejecutar** para realizar una serie de pruebas de la aplicación lógica.

![Guardar y ejecutar](media/logicapp-flow-connector/save-run.png)


Cuando se complete la aplicación lógica, compruebe el correo del destinatario que especificó.  Debe haber recibido un correo electrónico con un cuerpo similar al siguiente:

![Correo electrónico de ejemplo](media/logicapp-flow-connector/sample-mail.png)



## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [consultas de registro en Azure Monitor](../log-query/log-query-overview.md).
- Más información acerca de [Logic Apps](../../logic-apps/index.yml).
- Más información sobre [Power Automate](https://flow.microsoft.com).
