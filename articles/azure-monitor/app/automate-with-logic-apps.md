---
title: Automatización de procesos de Azure Application Insights con Logic Apps
description: Obtenga información sobre cómo automatizar rápidamente los procesos repetibles mediante la adición de Application Insights Connector a la aplicación lógica.
ms.topic: conceptual
ms.date: 03/11/2019
ms.openlocfilehash: d023d774c130a9db513fbcf9452b05fa030a967a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323203"
---
# <a name="automate-application-insights-processes-by-using-logic-apps"></a>Automatización de procesos de Application Insights con Logic Apps

¿Se encuentra a menudo ejecutando las mismas consultas en los datos de telemetría para comprobar si el servicio está funcionando correctamente? ¿Desea automatizar estas consultas para encontrar tendencias y anomalías, y crear sus propios flujos de trabajo en función de ellas? El conector Azure Application Insights para Logic Apps es la herramienta adecuada para este fin.

> [!NOTE]
> El conector de Azure Application Insights se ha reemplazado por el [conector de Azure Monitor](../platform/logicapp-flow-connector.md) que se integra con Azure Active Directory en lugar de requerir una clave de API y también permite recuperar datos de un área de trabajo de Log Analytics.

Con esta integración, se pueden automatizar numerosos procesos sin tener que escribir una sola línea de código. Puede crear una aplicación lógica con Application Insights Connector para automatizar rápidamente cualquier proceso de Application Insights. 

También puede agregar acciones adicionales. La característica Logic Apps de Azure App Service realiza cientos de acciones disponibles. Por ejemplo, si usa una aplicación lógica, puede enviar automáticamente una notificación por correo electrónico o crear un error en Azure DevOps. También puede utilizar una de las muchas [plantillas](../../logic-apps/logic-apps-create-logic-apps-from-templates.md) disponibles para ayudar a acelerar el proceso de creación de la aplicación lógica. 

## <a name="create-a-logic-app-for-application-insights"></a>Creación de una aplicación lógica para Application Insights

En este tutorial, puede obtener información sobre cómo crear una aplicación lógica que usa el algoritmo de clústeres automáticos de Analytics para agrupar los atributos en los datos para una aplicación web. El flujo envía automáticamente los resultados por correo electrónico, y esto es solo un ejemplo de cómo puede usar Application Insights Analytics y Logic Apps conjuntamente. 

### <a name="step-1-create-a-logic-app"></a>Paso 1: Creación de una aplicación lógica
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Crear un recurso**, seleccione **Web y móvil** y, a continuación, seleccione **Logic App**.

    ![Ventana de la aplicación lógica nueva](./media/automate-with-logic-apps/1createlogicapp.png)

### <a name="step-2-create-a-trigger-for-your-logic-app"></a>Paso 2: Creación de un desencadenador para la aplicación lógica
1. En la ventana del **Diseñador de aplicación lógica**, en **Empezar con un desencadenador común**, seleccione **Periodicidad**.

    ![Ventana del Diseñador de aplicaciones lógicas](./media/automate-with-logic-apps/2logicappdesigner.png)

1. En el cuadro **Intervalo**, escriba **1** y, luego, en el cuadro **Frecuencia**, seleccione **Día**.

    ![Ventana "Recurrencia" del Diseñador de aplicaciones lógicas](./media/automate-with-logic-apps/3recurrence.png)

### <a name="step-3-add-an-application-insights-action"></a>Paso 3: Incorporación de una acción de Application Insights
1. Haga clic en **New Step** (Nuevo paso).

1. En el cuadro de búsqueda **Elegir una acción**, escriba **Azure Application Insights**.

1. En **Acciones**, haga clic en **Azure Application Insights: Visualizar consulta de análisis**.

    ![Ventana "Elegir una acción" del Diseñador de aplicaciones lógicas](./media/automate-with-logic-apps/4visualize.png)

### <a name="step-4-connect-to-an-application-insights-resource"></a>Paso 4: Conexión a un recurso de Application Insights

Para completar este paso, necesita un identificador de aplicación y una clave de API para el recurso. Puede recuperarlos desde Azure Portal, como se muestra en el diagrama siguiente:

![Identificador de aplicación en Azure Portal](./media/automate-with-logic-apps/5apiaccess.png)

![Identificador de aplicación en Azure Portal](./media/automate-with-logic-apps/6apikey.png)

Proporcione un nombre para la conexión, el identificador de aplicación y la clave de API.

![Ventana de conexión del flujo del Diseñador de aplicaciones lógicas](./media/automate-with-logic-apps/7connection.png)

### <a name="step-5-specify-the-analytics-query-and-chart-type"></a>Paso 5: Especificación del tipo de gráfico y consulta de análisis
En el siguiente ejemplo, la consulta selecciona las solicitudes con error del último día y las pone en correlación con las excepciones que se han producido como parte de la operación. Analytics correlaciona las solicitudes erróneas en función del identificador operation_Id. La consulta, a continuación, segmenta los resultados mediante el algoritmo de clúster automático. 

Al crear sus propias consultas, asegúrese de comprobar que funcionan correctamente en Analytics antes de agregarlas al flujo.

1. En el cuadro **Consulta** , agregue la siguiente consulta de análisis:

    ```
    requests
    | where timestamp > ago(1d)
    | where success == "False"
    | project name, operation_Id
    | join ( exceptions
        | project problemId, outerMessage, operation_Id
    ) on operation_Id
    | evaluate autocluster()
    ```

1. En el cuadro **Tipo de gráfico**, seleccione **Tabla Html**.

    ![Pantalla de configuración de consulta de Analytics](./media/automate-with-logic-apps/8query.png)

### <a name="step-6-configure-the-logic-app-to-send-email"></a>Paso 6: Configuración de la aplicación lógica para enviar correo electrónico

1. Haga clic en **New Step** (Nuevo paso).

1. En el cuadro de búsqueda, escriba **Office 365 Outlook**.

1. Haga clic en **Office 365 Outlook: Enviar un correo electrónico**.

    ![Selección de Office 365 Outlook](./media/automate-with-logic-apps/9sendemail.png)

1. En la ventana **Enviar un correo electrónico**, haga lo siguiente:

   a. Escriba la dirección de correo electrónico del destinatario.

   b. Escriba un asunto para el correo electrónico.

   c. Haga clic en cualquier parte del cuadro **Cuerpo** y, en el menú de contenido dinámico que se abre a la derecha, seleccione **Cuerpo**.
    
   d. Haga clic en la lista desplegable **Agregar nuevo parámetro** y seleccione Datos adjuntos y Es HTML.

      ![Configuración de Office 365 Outlook](./media/automate-with-logic-apps/10emailbody.png)

      ![Configuración de Office 365 Outlook](./media/automate-with-logic-apps/11emailparameter.png)

1. En el menú de contenido dinámico, haga lo siguiente:

    a. Seleccione **Nombre de datos adjuntos**.

    b. Seleccione **Contenido de datos adjuntos**.
    
    c. En el campo **Es HTML**, seleccione **Sí**.

      ![Pantalla de configuración de correo electrónico de Office 365](./media/automate-with-logic-apps/12emailattachment.png)

### <a name="step-7-save-and-test-your-logic-app"></a>Paso 7: Guardado y prueba de la aplicación lógica
* Haga clic en **Guardar** para guardar los cambios.

Puede esperar a que el desencadenador ejecute la aplicación lógica, o puede ejecutarla inmediatamente si selecciona **Ejecutar**.

![Pantalla de creación de la aplicación lógica](./media/automate-with-logic-apps/13save.png)

Cuando se ejecuta la aplicación lógica, los destinatarios que haya especificado en la lista de correo electrónico recibirán un correo electrónico similar al siguiente:

![Mensaje de correo electrónico de la aplicación lógica](./media/automate-with-logic-apps/flow9.png)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo crear [consultas de análisis](../log-query/get-started-queries.md).
- Más información acerca de [Logic Apps](../../logic-apps/logic-apps-overview.md).



<!--Link references-->

