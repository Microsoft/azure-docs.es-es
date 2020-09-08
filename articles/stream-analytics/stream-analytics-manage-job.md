---
title: 'Tutorial: Creación y administración de un trabajo de Stream Analytics con Azure Portal'
description: En este tutorial se proporciona una demostración de un extremo a otro completa de cómo usar Azure Stream Analytics para analizar las llamadas fraudulentas en una secuencia de llamadas de teléfono.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc
ms.date: 06/03/2019
ms.openlocfilehash: 577a80f04ad186ab1575fa78db3fa59402d6058f
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "83697385"
---
# <a name="tutorial-analyze-phone-call-data-with-stream-analytics-and-visualize-results-in-power-bi-dashboard"></a>Tutorial: Analizar los datos de llamadas de teléfono con Stream Analytics y visualizar los resultados en un panel de Power BI

En este tutorial se enseña cómo analizar los datos de las llamadas de teléfono mediante Azure Stream Analytics. Los datos de las llamadas de teléfono, que genera una aplicación cliente, contienen varias llamadas fraudulentas que filtrará el trabajo de Stream Analytics.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Generar datos de ejemplo de llamadas de teléfono y enviarlos a Azure Event Hubs
> * Creación de un trabajo de Stream Analytics
> * Configuración de la entrada y salida del trabajo
> * Definir una consulta para filtrar las llamadas fraudulentas
> * Probar e iniciar el trabajo
> * Visualizar los resultados en Power BI

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, realice las siguientes acciones:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).
* Inicie sesión en [Azure Portal](https://portal.azure.com/).
* Descargue la aplicación de generación de eventos de llamada telefónica [TelcoGenerator.zip](https://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip) del Centro de descarga de Microsoft u obtenga el código fuente de [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator).
* Necesitará la cuenta de Power BI.

## <a name="create-an-azure-event-hub"></a>Crear un centro de eventos de Azure

Para que Stream Analytics pueda analizar el flujo de datos de las llamadas fraudulentas, los datos se deben enviar a Azure. En este tutorial, enviará los datos a Azure mediante [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-what-is-event-hubs).

Siga estos pasos para crear una instancia de Event Hub y enviar datos de la llamada a dicha instancia:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Crear un recurso** > **Internet de las cosas** > **Event Hubs**.

   ![Creación de un centro de eventos de Azure mediante Azure Portal](media/stream-analytics-manage-job/find-event-hub-resource.png)
3. Rellene el panel **Crear espacio de nombres** con los siguientes valores:

   |**Configuración**  |**Valor sugerido** |**Descripción**  |
   |---------|---------|---------|
   |Nombre     | myEventHubsNS        |  Un nombre único para identificar el espacio de nombres del centro de eventos.       |
   |Subscription     |   \<Your subscription\>      |   Seleccione una suscripción de Azure donde quiere crear el centro de eventos.      |
   |Resource group     |   MyASADemoRG      |  Seleccione **Crear nuevo** y escriba un nuevo nombre de grupo de recursos para la cuenta.       |
   |Location     |   Oeste de EE. UU. 2      |    Ubicación donde se puede implementar el espacio de nombres del centro de eventos.     |

4. Use las opciones predeterminadas en el resto de la configuración y seleccione **Crear**.

   ![Creación de un espacio de nombres del centro de eventos en Azure Portal](media/stream-analytics-manage-job/create-event-hub-namespace.png)

5. Cuando se haya terminado de implementar el espacio de nombres, vaya a **Todos los recursos** y busque *myEventHubNS*en la lista de recursos de Azure. Seleccione *myEventHubsNS* para abrirlo.
6. Luego, seleccione **+Event Hub** y en **Nombre**, escriba *MyEventHub* u otro nombre si así lo prefiere. Use las opciones predeterminadas en el resto de la configuración y seleccione **Crear**. Espere a que la implementación se realice correctamente.

   ![Configuración del centro de eventos en Azure Portal](media/stream-analytics-manage-job/create-event-hub-portal.png)

### <a name="grant-access-to-the-event-hub-and-get-a-connection-string"></a>Concesión de acceso al centro de eventos y obtención de una cadena de conexión

Para que una aplicación pueda enviar datos a Azure Event Hubs, el centro de eventos debe tener una directiva que permita el acceso adecuado. La directiva de acceso genera una cadena de conexión que incluye la información de autorización.

1. Vaya al centro de eventos que ha creado en el paso anterior, MyEventHub*. Seleccione **Directivas de acceso compartido** en **Configuración** y, luego, seleccione **+Agregar**.

2. Asigne a la directiva el nombre **MyPolicy** y asegúrese de que la opción **Administrar** está seleccionada. Seleccione **Crear**.

   ![Creación de una directiva de acceso compartido para el centro de eventos](media/stream-analytics-manage-job/create-event-hub-access-policy.png)

3. Una vez creada la directiva, seleccione esta opción para abrirla y busque **Cadena de conexión: clave principal**. Seleccione el botón de **copia** azul situado junto a la cadena de conexión.

   ![Guardar la cadena de conexión de la directiva de acceso compartido](media/stream-analytics-manage-job/save-connection-string.png)

4. Pegue la cadena de conexión en un editor de texto. Necesitará esta cadena de conexión en la sección siguiente.

   La cadena de conexión tiene el siguiente aspecto:

   `Endpoint=sb://<Your event hub namespace>.servicebus.windows.net/;SharedAccessKeyName=<Your shared access policy name>;SharedAccessKey=<generated key>;EntityPath=<Your event hub name>`

   Observe que la cadena de conexión contiene varios pares clave-valor, separados por punto y coma: **Endpoint**, **SharedAccessKeyName**, **SharedAccessKey** y **EntityPath**.

## <a name="start-the-event-generator-application"></a>Inicio de la aplicación de generación de eventos

Antes de iniciar la aplicación TelcoGenerator, debe configurarla para enviar datos a los centros de eventos de Azure que creó anteriormente.

1. Extraiga el contenido del archivo [TelcoGenerator.zip](https://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip).
2. Abra el archivo `TelcoGenerator\TelcoGenerator\telcodatagen.exe.config` en un editor de texto de su elección (como hay más de un archivo de configuración, asegúrese de abrir el correcto).

3. Actualice el elemento `<appSettings>` en el archivo de configuración con los detalles siguientes:

   * Como valor de la clave *EventHubName* use el valor de EntityPath en la cadena de conexión.
   * Establezca el valor de la clave *Microsoft.ServiceBus.ConnectionString* en la cadena de conexión sin el valor de EntityPath (no olvide quitar el punto y coma que lo precede).

4. Guarde el archivo.
5. A continuación abra una ventana de comandos y cambie a la carpeta en la que ha descomprimido la aplicación TelcoGenerator. Después, escriba el comando siguiente:

   ```cmd
   telcodatagen.exe 1000 0.2 2
   ```

   Este comando toma los parámetros siguientes:
   * Número de registros de datos de llamada por hora.
   * Porcentaje de probabilidad de fraude, que es la frecuencia con la que la aplicación debe simular una llamada fraudulenta. El valor 0,2 indica que aproximadamente el 20 % de los registros de llamada tendrán un aspecto fraudulento.
   * Duración en horas, que es el número de horas que debe ejecutarse la aplicación. La aplicación se puede detener en cualquier momento. Para ello, solo hay que finalizar el proceso (**Ctrl+C**) en la línea de comandos.

   Después de unos segundos, la aplicación comienza a mostrar los registros de llamada telefónica en la pantalla tal y como los envía al centro de eventos. Los datos de llamada de teléfono contienen los siguientes campos:

   |**Registro**  |**Definición**  |
   |---------|---------|
   |CallrecTime    |  Marca de tiempo para la hora de inicio de la llamada.       |
   |SwitchNum     |  Conmutador de teléfono que se usa para conectar la llamada. En este ejemplo, los conmutadores son cadenas que representan el país o la región de origen (Estados Unidos, China, Reino Unido, Alemania o Australia).       |
   |CallingNum     |  Número de teléfono del autor de la llamada.       |
   |CallingIMSI     |  Identidad del suscriptor móvil internacional (IMSI). Identificador único del autor de la llamada.       |
   |CalledNum     |   Número de teléfono del destinatario de la llamada.      |
   |CalledIMSI     |  Identidad del suscriptor móvil internacional (IMSI). Identificador único del destinatario de la llamada.       |

## <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics

Ahora que tiene un flujo de eventos de llamada, puede crear un trabajo de Stream Analytics que lea los datos desde el centro de eventos.

1. Para crear un trabajo de Stream Analytics, vaya a [Azure Portal](https://portal.azure.com/).

2. Haga clic en **Crear un recurso** > **Internet de las cosas** > **Trabajo de Stream Analytics**.

3. Rellene el panel **Nuevo trabajo de Stream Analytics** con los valores siguientes:

   |**Configuración**  |**Valor sugerido**  |**Descripción**  |
   |---------|---------|---------|
   |Nombre del trabajo     |  ASATutorial       |   Un nombre único para identificar el espacio de nombres del centro de eventos.      |
   |Suscripción    |  \<Your subscription\>   |   Seleccione una suscripción de Azure donde desea crear el trabajo.       |
   |Resource group   |   MyASADemoRG      |   Seleccione **Use existing** (Usar existente) y, después, escriba un nuevo nombre de grupo de recursos para la cuenta.      |
   |Location   |    Oeste de EE. UU. 2     |      Ubicación donde se puede implementar el trabajo. Es conveniente colocar el trabajo y el centro de eventos en la misma región para obtener el mejor rendimiento y no pagar por transferir datos de una región a otra.      |
   |Entorno de hospedaje    | Nube        |     Los trabajos de Stream Analytics se pueden implementar en la nube o en dispositivos perimetrales. Si elige la nube, podrá implementar en la nube de Azure y, si elige implementar en un dispositivo perimetral, podrá hacerlo en un dispositivo de IoT Edge.    |
   |Unidades de streaming     |    1       |      Las unidades de streaming representan los recursos informáticos que se necesitan para ejecutar un trabajo. De forma predeterminada, este valor se establece en 1. Para información sobre el escalado de unidades de streaming, consulte el artículo [Descripción y ajuste de las unidades de streaming](stream-analytics-streaming-unit-consumption.md).      |

4. Use las opciones predeterminadas en el resto de la configuración, seleccione **Crear** y espere a que la implementación se realice correctamente.

   ![Creación de un trabajo de Azure Stream Analytics](media/stream-analytics-manage-job/create-stream-analytics-job.png)

## <a name="configure-job-input"></a>Configuración de la entrada del trabajo

El siguiente paso es definir un origen de entrada para que el trabajo lea los datos mediante el centro de eventos que creó en la sección anterior.

1. En Azure Portal, abra el panel **Todos los recursos** y busque el trabajo de Stream Analytics *ASATutorial*.

2. En la sección **Topología de trabajo** del panel de trabajos de Stream Analytics, seleccione la opción **Entradas**.

3. Haga clic en **Agregar entrada de flujo** y seleccione **Event Hub** (Centro de eventos). Rellene el panel con los siguientes valores:

   |**Configuración**  |**Valor sugerido**  |**Descripción**  |
   |---------|---------|---------|
   |Alias de entrada     |  CallStream       |  Proporcione un nombre descriptivo para identificar la entrada. El alias de entrada debe tener una longitud de entre 3 y 63 caracteres, y solo puede incluir caracteres alfanuméricos, guiones y guiones bajos.       |
   |Suscripción    |   \<Your subscription\>      |   Seleccione la suscripción de Azure en la que creó el centro de eventos. El centro de eventos puede estar en la misma suscripción que el trabajo de Stream Analytics, o en otra diferente.       |
   |Espacio de nombres del centro de eventos    |  myEventHubsNS       |  Seleccione el espacio de nombres del centro de eventos que creó en la sección anterior. Todos los espacios de nombres del centro de eventos disponibles en su suscripción actual se muestran en la lista desplegable.       |
   |Nombre del centro de eventos    |   MyEventHub      |  Seleccione el centro de eventos que creó en la sección anterior. Todos los centros de eventos disponibles en la suscripción actual se muestran en la lista desplegable.       |
   |Nombre de la directiva del centro de eventos   |  MyPolicy       |  Seleccione la directiva de acceso compartido del centro de eventos que creó en la sección anterior. Todas las directivas de los centros de eventos disponibles en la suscripción actual se muestran en la lista desplegable.       |

4. Use las opciones predeterminadas en el resto de los valores y seleccione **Guardar**.

   ![Configuración de la entrada de Azure Stream Analytics](media/stream-analytics-manage-job/configure-stream-analytics-input.png)

## <a name="configure-job-output"></a>Configuración de la salida del trabajo

El último paso consiste en definir un receptor de salida para el trabajo en el que se puedan escribir los datos transformados. En este tutorial, los datos se generan y se visualizan con Power BI.

1. En Azure Portal, abra el panel **Todos los recursos** y busque el trabajo de Stream Analytics *ASATutorial*.

2. En la sección **Topología de trabajo** del panel de trabajos de Stream Analytics, seleccione la opción **Salidas**.

3. Seleccione **+ Agregar** > **Power BI**. Luego, rellene el formulario con los siguientes detalles y seleccione **Autorizar**:

   |**Configuración**  |**Valor sugerido**  |
   |---------|---------|
   |Alias de salida  |  MyPBIoutput  |
   |Nombre del conjunto de datos  |   ASAdataset  |
   |Nombre de la tabla |  ASATable  |

   ![Configuración de la salida de Stream Analytics](media/stream-analytics-manage-job/configure-stream-analytics-output.png)

4. Después de seleccionar **Autorizar**, se abre una ventana emergente y se le pide que escriba las credenciales para autenticarse en su cuenta de Power BI. Una vez que la autorización se realice correctamente, **guarde** la configuración.

## <a name="define-a-query-to-analyze-input-data"></a>Definición de una consulta para analizar los datos de entrada

El siguiente paso consiste en crear una transformación que analice los datos en tiempo real. Definirá la consulta de transformación mediante el [lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference). La consulta que se ha usado en este tutorial detecta llamadas fraudulentas en los datos del teléfono.

En este ejemplo, se realizan llamadas fraudulentas desde el mismo usuario en un intervalo de cinco segundos, pero en ubicaciones separadas. Por ejemplo, el mismo usuario no puede legítimamente realizar una llamada desde Estados Unidos y Australia al mismo tiempo. Para definir la consulta de transformación del trabajo de Stream Analytics:

1. En Azure Portal, abra el panel **Todos los recursos** y vaya al trabajo de Stream Analytics que creó antes, **ASATutorial**.

2. En la sección **Topología de trabajo** del panel de trabajos de Stream Analytics, seleccione la opción **Consulta**. En la ventana de consulta se enumeran las entradas y salidas que configuradas para el trabajo y en ella puede crear una consulta para transformar el flujo de entrada.

3. Reemplace la consulta existente en el editor por la siguiente consulta, que realiza una autocombinación en un intervalo de 5 segundos de datos de la llamada:

   ```sql
   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "MyPBIoutput"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime
   ON CS1.CallingIMSI = CS2.CallingIMSI
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

   Para comprobar si hay llamadas fraudulentas, puede realizar una autocombinación de los datos de streaming según el valor de `CallRecTime`. Luego, puede buscar registros de llamada en los que el valor `CallingIMSI` (el número donde se origina) sea el mismo, pero el valor `SwitchNum` (país o región de origen) sea distinto. Cuando use una operación JOIN con datos de streaming, la combinación debe proporcionar algunos límites en la diferencia de tiempo que puede haber entre filas coincidentes. Dado que los datos de streaming no tienen final, los límites de tiempo de la relación se especifican dentro de la cláusula **ON** de la combinación mediante la función [DATEDIFF](https://docs.microsoft.com/stream-analytics-query/datediff-azure-stream-analytics).

   Esta consulta es como una combinación SQL normal, salvo por la función **DATEDIFF**. La función **DATEDIFF** que se usa en esta consulta es específica de Stream Analytics y debe aparecer dentro de la cláusula `ON...BETWEEN`.

4. **Guarde** la consulta.

   ![Definición de la consulta de Stream Analytics en Azure Portal](media/stream-analytics-manage-job/define-stream-analytics-query.png)

## <a name="test-your-query"></a>Prueba de la consulta

Puede probar una consulta en el editor de consultas y usar para ello datos de ejemplo. Ejecute los siguientes pasos para probar la consulta:

1. Asegúrese de que la aplicación TelcoGenerator se ejecuta y genera registros de llamadas de teléfono.

2. En el panel **Consulta**, seleccione los puntos que aparecen junto a la entrada **CallStream** y seleccione *Datos de ejemplo de la entrada*.

3. Establezca **Minutos** en 3 y seleccione **Aceptar**. Se realiza el muestreo de tres minutos de datos del flujo de entrada y se le notifica cuando los datos de ejemplo están listos. Puede ver el estado del muestreo en la barra de notificación.

   Los datos de muestra se almacenan temporalmente y están disponibles mientras tenga abierta la ventana de consulta. Si cierra la ventana de consulta, los datos de muestra se descartan y tendrá que crear un nuevo conjunto de datos de muestra si desea realizar la prueba. Como alternativa, puede usar un archivo JSON con datos de muestra de [GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/telco.json) y luego cargarlo para usarlo como datos de muestra para la entrada *CallStream*.

   ![Representación visual de cómo realizar un muestreo de datos de entrada para Stream Analytics](media/stream-analytics-manage-job/sample-input-data-asa.png)

4. Seleccione **Probar** para probar la consulta. Debería ver los siguientes resultados:

   ![Salida de la prueba de consulta de Stream Analytics](media/stream-analytics-manage-job/sample-test-output-restuls.png)

## <a name="start-the-job-and-visualize-output"></a>Inicio del trabajo y visualización de la salida

1. Para iniciar el trabajo, vaya al panel **Información general** del trabajo y seleccione **Iniciar**.

2. Seleccione **Ahora** como la hora de inicio de la salida del trabajo y seleccione **Iniciar**. El estado del trabajo se puede ver en la barra de notificación.

3. Una vez que el trabajo se inicia correctamente, vaya a [Power BI](https://powerbi.com/) e inicie sesión con su cuenta profesional o educativa. Si la consulta del trabajo de Stream Analytics genera resultados, el conjunto de datos *ASAdataset* que creó existe en la pestaña **Conjuntos de datos**.

4. En el área de trabajo de Power BI, seleccione **+ Crear** para crear un panel denominado *Fraudulent Calls* (Llamadas fraudulentas).

5. En la parte superior de la ventana, seleccione **Agregar icono**. Luego, seleccione **Datos de transmisión personalizados** y **Siguiente**. Elija **ASAdataset** en **Sus conjuntos de datos**. Seleccione **Tarjeta** en la lista desplegable **Tipo de visualización** y agregue **fraudulentcalls** a **Campos**. Seleccione **Siguiente** para escribir el nombre del icono y, después, seleccione **Aplicar** para crear el icono.

   ![Creación de iconos de paneles de Power BI](media/stream-analytics-manage-job/create-power-bi-dashboard-tiles.png)

6. Siga de nuevo el paso 5, con las siguientes opciones:
   * Cuando llegue a Tipo de visualización, elija Gráfico de líneas.
   * Agregue un eje y seleccione **windowend**.
   * Agregue un valor y seleccione **fraudulentcalls**.
   * En **Período de tiempo para mostrar**, seleccione los últimos 10 minutos.

7. El panel debe ser similar al del ejemplo siguiente una vez que se agreguen ambos iconos. Observe que, si se ejecutan la aplicación del remitente del centro de eventos y la aplicación de Stream Analytics, el panel de Power BI se actualiza periódicamente cuando llegan nuevos datos.

   ![Ver los resultados en el panel de Power BI](media/stream-analytics-manage-job/power-bi-results-dashboard.png)

## <a name="embedding-your-power-bi-dashboard-in-a-web-application"></a>Inserción del panel de Power BI en una aplicación web

En esta parte del tutorial, usará una aplicación web [ASP.NET](https://asp.net/) de ejemplo que ha creado el equipo de Power BI para insertar el panel. Para más información sobre cómo insertar paneles, consulte el artículo [Inserción con Power BI](https://docs.microsoft.com/power-bi/developer/embedding).

Para configurar la aplicación, vaya al repositorio de GitHub [PowerBI-Developer-Samples](https://github.com/Microsoft/PowerBI-Developer-Samples) y siga las instrucciones de la sección **User Owns Data** (El usuario es propietario de los datos) (use las direcciones URL de redireccionamiento y de la página principal en el subapartado **integrate-web-app**). Como se va a usar el ejemplo del panel, utilice el código de ejemplo **integrate-web-app** que se encuentra en el [repositorio de GitHub](https://github.com/microsoft/PowerBI-Developer-Samples/tree/master/.NET%20Framework/Embed%20for%20your%20organization/integrate-web-app).
Cuando esté satisfecho con la aplicación que se ejecuta en el explorador, siga estos pasos para insertar el panel que creó anteriormente en la página web:

1. Seleccione **Sign in to Power BI** (Iniciar sesión en Power BI). Mediante esta acción se concede a la aplicación acceso a los paneles de la cuenta de Power BI.

2. Seleccione el botón **Get Dashboards** (Obtener paneles), que muestra los paneles de su cuenta en una tabla. Busque el nombre del panel que creó anteriormente **powerbi-embedded-dashboard** y copie el valor de **EmbedUrl** correspondiente.

3. Por último, pegue el valor de **EmbedUrl** en el campo de texto correspondiente y seleccione **Embed Dashboard** (Insertar panel). Ahora puede ver el mismo panel insertado en una aplicación web.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un trabajo sencillo de Stream Analytics, ha analizado los datos de entrada y ha presentado los resultados en un panel de Power BI. Para más información sobre los trabajos de Stream Analytics, continúe con el tutorial siguiente:

> [!div class="nextstepaction"]
> [Ejecución de Azure Functions dentro de trabajos de Stream Analytics](stream-analytics-with-azure-functions.md)
