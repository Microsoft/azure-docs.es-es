---
title: Integración del panel de Power BI con Azure Stream Analytics
description: En este artículo se describe cómo usar un panel de Power BI en tiempo real para visualizar datos fuera de un trabajo de Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/05/2019
ms.openlocfilehash: 4d0ee48a42814ab2c4229c5f8f239aa1a1c28544
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87314227"
---
# <a name="stream-analytics-and-power-bi-a-real-time-analytics-dashboard-for-streaming-data"></a>Stream Analytics y Power BI: panel de análisis en tiempo real de flujo de datos

Azure Stream Analytics permite aprovechar una de las principales herramientas de inteligencia empresarial, [Microsoft Power BI](https://powerbi.com/). En este artículo, aprenderá a crear herramientas de inteligencia empresarial personalizadas utilizando Power BI como salida para los trabajos de Azure Stream Analytics. También aprenderá a crear y usar un panel en tiempo real.

Este artículo continúa a partir del tutorial [Detección de fraudes en tiempo real](stream-analytics-real-time-fraud-detection.md) de Stream Analytics. Se basa en el flujo de trabajo creado en ese tutorial y agrega una salida de Power BI para que se puedan visualizar llamadas telefónicas fraudulentas detectadas por un trabajo de Streaming Analytics. 

Puede ver un [vídeo](https://www.youtube.com/watch?v=SGUpT-a99MA) que muestra este escenario.


## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, asegúrese de que dispone de lo siguiente:

* Una cuenta de Azure.
* Una cuenta de Power BI Pro. Puede usar una cuenta profesional o una cuenta educativa.
* Versión completa del tutorial [Detección de fraudes en tiempo real](stream-analytics-real-time-fraud-detection.md). El tutorial incluye una aplicación que genera metadatos de llamada telefónica ficticia. En el tutorial, se crea un centro de eventos y se envían los datos de llamada telefónica de streaming al centro de eventos. Escriba una consulta que detecte llamadas fraudulentas (llamadas del mismo número a la vez en distintas ubicaciones). 


## <a name="add-power-bi-output"></a>Incorporación del resultado de Power BI
En el tutorial de detección de fraudes en tiempo real, la salida se envía a Azure Blob Storage. En esta sección se agrega una salida que envía información a Power BI.

1. En Azure Portal, abra la cuenta de Stream Analytics que creó antes. Si usó el nombre sugerido, el trabajo se llama `sa_frauddetection_job_demo`.

2. En el menú izquierdo, seleccione **Salidas** en **Topología de trabajo**. Luego, seleccione **+ Agregar** y elija **Power BI** en el menú desplegable.

3. Seleccione **+ Agregar** > **Power BI**. Luego, rellene el formulario con los siguientes detalles y seleccione **Autorizar**:

   |**Configuración**  |**Valor sugerido**  |
   |---------|---------|
   |Alias de salida  |  CallStream PowerBI  |
   |Nombre del conjunto de datos  |   sa-dataset  |
   |Nombre de la tabla |  fraudulent-calls  |

   ![Configuración de la salida de Stream Analytics](media/stream-analytics-power-bi-dashboard/configure-stream-analytics-output.png)

   > [!WARNING]
   > Si Power BI tiene un conjunto de datos y una tabla con los mismos nombres que especifica en el trabajo de Stream Analytics, se sobrescribirán los existentes.
   > Se recomienda no crear explícitamente este conjunto de datos y la tabla en la cuenta de Power BI. Se crearán automáticamente cuando inicie el trabajo de Stream Analytics y este empiece a enviar salida a Power BI. Si la consulta de trabajo no genera ningún resultado, el conjunto de datos y la tabla no se crean.
   >

4. Después de seleccionar **Autorizar**, se abre una ventana emergente y se le pide que escriba las credenciales para autenticarse en su cuenta de Power BI. Una vez que la autorización se realice correctamente, **guarde** la configuración.

8. Haga clic en **Crear**.

El conjunto de datos se crea con la siguiente configuración:

* **defaultRetentionPolicy: BasicFIFO**: los datos son FIFO y el máximo de filas son 200 000 filas.
* **defaultMode: pushStreaming**: el conjunto de datos admite iconos de streaming y objetos visuales tradicionales basados en informes (lo que también se conoce como inserción).

Actualmente, no se pueden crear conjuntos de datos con otras marcas.

Para más información sobre conjuntos de datos de Power BI, consulte la referencia de la [API de REST de Power BI](https://msdn.microsoft.com/library/mt203562.aspx).


## <a name="write-the-query"></a>Escritura de la consulta

1. Cierre la hoja **Salidas** y vuelva a la hoja del trabajo.

2. Haga clic en el cuadro **Consulta**. 

3. Escriba la siguiente consulta. Esta consulta es similar a la consulta de autocombinación que creó en el tutorial de detección de fraudes. La diferencia es que esta consulta envía los resultados a la nueva salida generada (`CallStream-PowerBI`). 

    >[!NOTE]
    >Si no asignó el nombre `CallStream` a la entrada en el tutorial de detección de fraudes, sustituya el nombre por `CallStream` en las cláusulas **FROM** y **JOIN** de la consulta.

   ```SQL
   /* Our criteria for fraud:
   Calls made from the same caller to two phone switches in different locations (for example, Australia and Europe) within five seconds */

   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "CallStream-PowerBI"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime

   /* Where the caller is the same, as indicated by IMSI (International Mobile Subscriber Identity) */
   ON CS1.CallingIMSI = CS2.CallingIMSI

   /* ...and date between CS1 and CS2 is between one and five seconds */
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5

   /* Where the switch location is different */
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. Haga clic en **Save**(Guardar).


## <a name="test-the-query"></a>Prueba de la consulta

Esta sección es opcional pero conveniente. 

1. Si la aplicación TelcoStreaming no se ejecuta en este momento, siga estos pasos para iniciarla:

    * Abra el símbolo del sistema.
    * Vaya a la carpeta donde se encuentran los archivos telcogenerator.exe y telcodatagen.exe.config modificado.
    * Ejecute el siguiente comando:

       `telcodatagen.exe 1000 .2 2`

2. En la página **Consulta** del trabajo de Stream Analytics, haga clic en los puntos que aparecen junto a la entrada `CallStream` y, después, seleccione **Datos de ejemplo de la entrada**.

3. Especifique que quiere datos correspondientes a tres minutos y haga clic en **OK**. Espere a que se le notifique que se ha realizado un muestreo de los datos.

4. Haga clic en **Probar** y revise los resultados.

## <a name="run-the-job"></a>Ejecutar el trabajo

1. Asegúrese de que la aplicación TelcoStreaming se ejecuta.

2. Vaya a la página **Información general** del trabajo de Stream Analytics y seleccione **Iniciar**.

    ![Inicio del trabajo de Stream Analytics](./media/stream-analytics-power-bi-dashboard/stream-analytics-sa-job-start-output.png)

El trabajo de Stream Analytics empieza a buscar llamadas fraudulentas en el flujo entrante. También crea el conjunto de datos y la tabla en Power BI y empieza a enviarles datos sobre las llamadas fraudulentas.


## <a name="create-the-dashboard-in-power-bi"></a>Creación del panel en Power BI

1. Vaya a [Powerbi.com](https://powerbi.com) e inicie sesión con su cuenta profesional o educativa. Si la consulta del trabajo de Stream Analytics genera resultados, verá que el conjunto de datos ya se ha creado:

    ![Ubicación del conjunto de datos de streaming en Power BI](./media/stream-analytics-power-bi-dashboard/stream-analytics-streaming-dataset.png)

2. En el área de trabajo, haga clic en **+&nbsp;Crear**.

    ![Botón Crear en el área de trabajo de Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard.png)

3. Cree otro panel y asígnele el nombre `Fraudulent Calls`.

    ![Creación de un panel y asignación de un nombre en el área de trabajo de Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard-name.png)

4. En la parte superior de la ventana, haga clic en **Agregar icono**, seleccione **DATOS DE TRANSMISIÓN PERSONALIZADOS** y luego haga clic en **Siguiente**.

    ![Icono de conjunto de datos de streaming personalizado en Power BI](./media/stream-analytics-power-bi-dashboard/custom-streaming-data.png)

5. En **YOUR DATSETS** (SUS CONJUNTOS DE DATOS), seleccione el conjunto de datos y haga clic en **Siguiente**.

    ![Conjunto de datos de streaming en Power BI](./media/stream-analytics-power-bi-dashboard/your-streaming-dataset.png)

6. En **Tipo de visualización**, seleccione **Tarjeta** y luego, en la lista **Campos**, seleccione **fraudulentcalls**.

    ![Detalles de visualización del nuevo icono](./media/stream-analytics-power-bi-dashboard/add-fraudulent-calls-tile.png)

7. Haga clic en **Next**.

8. Rellene detalles del icono, como el título y el subtítulo.

    ![Título y subtítulo del nuevo icono](./media/stream-analytics-power-bi-dashboard/pbi-new-tile-details.png)

9. Haga clic en **Aplicar**.

    Ahora tiene un contador de fraudes.

    ![Contador de fraudes en el panel de Power BI](./media/stream-analytics-power-bi-dashboard/power-bi-fraud-counter-tile.png)

8. Siga de nuevo los pasos para agregar un icono (a partir del paso 4). Esta vez, haga lo siguiente:

    * Cuando obtenga el **Tipo de visualización**, seleccione **Gráfico de líneas**. 
    * Agregue un eje y seleccione **windowend**. 
    * Agregue un valor y seleccione **fraudulentcalls**.
    * En **Período de tiempo para mostrar**, seleccione los últimos 10 minutos.

      ![Creación de un icono de gráfico de líneas en Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-tile-line-chart.png)

9. Haga clic en **Siguiente**, agregue el título y el subtítulo y haga clic en **Aplicar**.

     El panel de Power BI le ofrece ahora dos vistas de datos sobre las llamadas fraudulentas detectadas en los datos de streaming.

     ![Panel de Power BI finalizado que muestra dos iconos de llamadas fraudulentas](./media/stream-analytics-power-bi-dashboard/pbi-dashboard-fraudulent-calls-finished.png)

## <a name="learn-about-limitations-and-best-practices"></a>Más información sobre limitaciones y prácticas recomendadas
Actualmente, se puede llamar a Power BI una vez por segundo aproximadamente. Los objetos visuales de streaming admiten paquetes de 15 KB. Si el tamaño es superior, se producirá un error en los objetos visuales de streaming (pero la inserción continuará funcionando). Gracias a estas limitaciones, Power BI se presta de forma más natural a los casos en los que Azure Stream Analytics realiza una reducción considerable de la carga de datos. Se recomienda utilizar una ventana de saltos de tamaño constante o una ventana de salto para asegurarse de que la inserción de datos es a lo sumo de una inserción por segundo y de que la consulta se ajusta a los requisitos de capacidad de procesamiento.

Puede utilizar la siguiente ecuación para calcular el valor que asignar a la ventana en segundos:

![Ecuación para calcular el valor para asignar a la ventana en segundos](./media/stream-analytics-power-bi-dashboard/compute-window-seconds-equation.png)  

Por ejemplo:

* Tiene 1000 dispositivos que envían datos a intervalos de un segundo.
* Está usando Power BI Pro SKU que admite 1 000 000 de filas por hora.
* Desea publicar la cantidad media de datos por dispositivo en Power BI.

En consecuencia, la ecuación se convierte en:

![Ecuación según los criterios de ejemplo](./media/stream-analytics-power-bi-dashboard/power-bi-example-equation.png)  

Con esta configuración, se puede cambiar la consulta original a la siguiente:

```SQL
    SELECT
        MAX(hmdt) AS hmdt,
        MAX(temp) AS temp,
        System.TimeStamp AS time,
        dspl
    INTO "CallStream-PowerBI"
    FROM
        Input TIMESTAMP BY time
    GROUP BY
        TUMBLINGWINDOW(ss,4),
        dspl
```

### <a name="renew-authorization"></a>Renovar la autorización
Si la contraseña ha cambiado desde que se creó o autenticó por última vez el trabajo, tendrá que volver a autenticar la cuenta de Power BI. Si Azure Multi-Factor Authentication se configura en el inquilino de Azure Active Directory (Azure AD), también debe renovar la autorización de Power BI cada dos semanas. Si no se renueva, podrían aparecer síntomas, como la ausencia de salida del trabajo o un `Authenticate user error` en los registros de operaciones.

De forma similar, si un trabajo intenta iniciarse después de que el token haya caducado, se producirá un error y no se iniciará. Para resolver este problema, detenga el trabajo en ejecución y vaya a la salida de Power BI. A fin de evitar que se pierdan datos, seleccione el vínculo **Renovar autorización** y reinicie el trabajo desde la **Hora de la última detención**.

Después de que la autorización se haya actualizado con Power BI, se mostrará una alerta verde en el área de autorización para indicar que el problema se ha resuelto.

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Salidas de Stream Analytics](stream-analytics-define-outputs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
