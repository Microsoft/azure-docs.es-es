---
title: Creación de una función que se integre con Azure Logic Apps
description: Creación de una función que se integre con Azure Logic Apps y Azure Cognitive Services para categorizar las opiniones de tweet y enviar notificaciones en caso de que las opiniones sean negativas.
author: craigshoemaker
ms.assetid: 60495cc5-1638-4bf0-8174-52786d227734
ms.topic: tutorial
ms.date: 04/27/2020
ms.author: cshoe
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: a7cdeb7bfde7396026b782382b34228c309b37d7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088152"
---
# <a name="create-a-function-that-integrates-with-azure-logic-apps"></a>Creación de una función que se integre con Azure Logic Apps

Azure Functions se integra con Azure Logic Apps en el diseñador de Logic Apps. Esta integración permite usar la capacidad de proceso de Azure Functions en las orquestaciones con otros servicios de Azure y de terceros. 

En este tutorial se muestra cómo utilizar Azure Functions con Logic Apps y Cognitive Services en Azure para ejecutar el análisis de opiniones de entradas de Twitter. Una función desencadenada por HTTP clasifica los tweets en verdes, amarillos o rojos, en función de la puntuación de la opinión. Se envía un correo electrónico cuando se detecta una opinión deficiente. 

![imagen de los dos primeros pasos de una aplicación en el diseñador de Logic Apps](media/functions-twitter-email/00-logic-app-overview.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un recurso de API de Cognitive Services.
> * Crear una función que clasifica las opiniones de tweet.
> * Crear una aplicación de lógica que se conecta a Twitter.
> * Agregar detección de opiniones a la aplicación lógica. 
> * Conectar la aplicación lógica a la función.
> * Enviar un correo electrónico en función de la respuesta de la función.

## <a name="prerequisites"></a>Prerrequisitos

+ Una cuenta de [Twitter](https://twitter.com/) activa. 
+ Una cuenta de [Outlook.com](https://outlook.com/) (para enviar las notificaciones).

> [!NOTE]
> Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar el conector de Gmail solo con aplicaciones y servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

+ Este tema usa como punto de partida los recursos creados en [Creación de la primera función desde Azure Portal](functions-create-first-azure-function.md).
Si aún no lo hecho, lleve a cabo estos pasos ahora para crear la aplicación de función.

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

Cognitive Services APIs están disponibles en Azure como recursos individuales. Use la API Text Analytics para detectar la opinión de los tweets que se supervisan.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.

3. Haga clic en **IA y Machine Learning** > **Análisis de texto**. Después, use la configuración especificada en la tabla para crear el recurso.

    ![Página Creación de recursos de Cognitive](media/functions-twitter-email/01-create-text-analytics.png)

    | Configuración      |  Valor sugerido   | Descripción                                        |
    | --- | --- | --- |
    | **Nombre** | MyCognitiveServicesAccnt | Elija un nombre de cuenta único. |
    | **Ubicación** | Oeste de EE. UU. | Use la ubicación más cercana. |
    | **Plan de tarifa** | F0 | Comience con la tarifa más baja. Si se queda sin llamadas, aumente a un nivel superior.|
    | **Grupos de recursos** | myResourceGroup | Utilice el mismo grupo de recursos para todos los servicios de este tutorial.|

4. Haga clic en **Crear** para crear el recurso. 

5. Haga clic en **Información general** y copie el valor de **Punto de conexión** en un editor de texto. Este valor se utiliza al crear una conexión a Cognitive Services API.

    ![Configuración de Cognitive Services](media/functions-twitter-email/02-cognitive-services.png)

6. En la columna de navegación de la izquierda, haga clic en **Claves** y, luego, copie el valor de **Clave 1** y péguelo en un editor de texto. La clave se usa para conectar la aplicación lógica a Cognitive Services API. 
 
    ![Claves de Cognitive Services](media/functions-twitter-email/03-cognitive-serviecs-keys.png)

## <a name="create-the-function-app"></a>Crear la aplicación de función

Azure Functions proporciona una excelente manera de descargar tareas de procesamiento en un flujo de trabajo de aplicaciones lógicas. Este tutorial utiliza una función desencadenada por HTTP para procesar las puntuaciones de opinión de los tweet procedentes de Cognitive Services y devolver un valor de clasificación.  

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-an-http-trigger-function"></a>Creación de una función desencadenada por HTTP  

1. En el menú de la izquierda de la ventana **Funciones**, seleccione **Funciones** y, a continuación, seleccione **Agregar** en el menú superior.

2. En la ventana **Nueva función**, seleccione **Desencadenador de HTTP**.

    ![Elegir la función de desencadenador de HTTP](./media/functions-twitter-email/06-function-http-trigger.png)

3. En la página **Nueva función**, seleccione **Crear función**.

4. En la nueva función de desencadenador de HTTP, seleccione **Código y prueba** en el menú de la izquierda, reemplace el contenido del archivo `run.csx` por el código siguiente y, a continuación, seleccione **Guardar**:

    ```csharp
    #r "Newtonsoft.Json"
    
    using System;
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        string category = "GREEN";
    
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        log.LogInformation(string.Format("The sentiment score received is '{0}'.", requestBody));
    
        double score = Convert.ToDouble(requestBody);
    
        if(score < .3)
        {
            category = "RED";
        }
        else if (score < .6) 
        {
            category = "YELLOW";
        }
    
        return requestBody != null
            ? (ActionResult)new OkObjectResult(category)
            : new BadRequestObjectResult("Please pass a value on the query string or in the request body");
    }
    ```

    El código de la función devuelve una clasificación de color basada en la puntuación de la opinión recibida en la solicitud. 

5. Para probar la función, seleccione **Test** en el menú superior. En la pestaña **Entrada**, escriba un valor de `0.2` en el **Cuerpo** y, a continuación, seleccione **Ejecutar**. Se devuelve un valor de **ROJO** en el **Contenido de respuesta HTTP** en la pestaña **salida**. 

    :::image type="content" source="./media/functions-twitter-email/07-function-test.png" alt-text="Definir la configuración del proxy":::

Ahora, tiene una función que clasifica las puntuaciones de opinión. A continuación, cree una aplicación lógica que integre la función con la API de Twitter y Cognitive Services. 

## <a name="create-a-logic-app"></a>Creación de una aplicación lógica   

1. En Azure Portal, haga clic en el botón **Crear un recurso** de la esquina superior izquierda.

2. Haga clic en **Web** > **Aplicación lógica**.
 
3. Luego, escriba un valor para **Nombre** como `TweetSentiment` y use la configuración especificada en la tabla.

    ![Creación de una aplicación lógica en Azure Portal](./media/functions-twitter-email/08-logic-app-create.png)

    | Configuración      |  Valor sugerido   | Descripción                                        |
    | ----------------- | ------------ | ------------- |
    | **Nombre** | TweetSentiment | Elija un nombre adecuado para la aplicación. |
    | **Grupos de recursos** | myResourceGroup | Elija el mismo grupo de recursos existente que antes. |
    | **Ubicación** | Este de EE. UU. | Elija una ubicación cercana a usted. |    

4. Cuando haya especificado los valores de configuración adecuados, haga clic en **Crear** para crear la aplicación lógica. 

5. Una vez creada la aplicación, haga clic en la nueva aplicación lógica anclada al panel. A continuación, en el diseñador de Logic Apps, desplácese hacia abajo y haga clic en la plantilla **Aplicación lógica en blanco**. 

    ![Plantilla Aplicación lógica en blanco](media/functions-twitter-email/09-logic-app-create-blank.png)

Ahora puede usar el diseñador de Logic Apps para agregar servicios y desencadenadores a la aplicación.

## <a name="connect-to-twitter"></a>Conexión a Twitter

En primer lugar, cree una conexión a la cuenta de Twitter. La aplicación lógica sondea si hay tweets, que desencadenan la ejecución de la aplicación.

1. En el diseñador, haga clic en el servicio **Twitter** y haga clic en el desencadenador **Cuando se publica un nuevo tweet**. Inicie sesión en su cuenta de Twitter y autorice a Logic Apps para usar su cuenta.

2. Use la configuración del desencadenador de Twitter como se especifica en la tabla. 

    ![Configuración del conector de Twitter](media/functions-twitter-email/10-tweet-settings.png)

    | Configuración      |  Valor sugerido   | Descripción                                        |
    | ----------------- | ------------ | ------------- |
    | **Texto de búsqueda** | #Azure | Use un hashtag lo suficientemente popular como para generar nuevos tweets en el intervalo elegido. Si usa el nivel Gratis y el hashtag es demasiado popular, puede agotar rápidamente la cuota de transacciones de la API Cognitive Services. |
    | **Intervalo** | 15 | El tiempo transcurrido entre solicitudes a Twitter, en unidades de frecuencia. |
    | **Frecuencia** | Minute | La unidad de frecuencia utilizada para el sondeo de Twitter.  |

3.  Haga clic en **Guardar** para conectarse a su cuenta de Twitter. 

Ahora la aplicación está conectada a Twitter. A continuación, va a conectar a análisis de texto para detectar la opinión de los tweets recopilados.

## <a name="add-sentiment-detection"></a>Agregar la detección de opiniones

1. Haga clic en **Nuevo paso** y, a continuación, en **Agregar una acción**.

2. En **Elegir una acción**, escriba **Análisis de texto** y, después, haga clic en la acción **Detectar sentimiento**.
    
    ![Nuevo paso y Agregar una acción](media/functions-twitter-email/11-detect-sentiment.png)

3. Escriba un nombre de conexión como `MyCognitiveServicesConnection`, pegue la clave de Cognitive Services API y el punto de conexión de Cognitive Services que reservó en el editor de texto y haga clic en **Crear**.

    ![Nuevo paso y Agregar una acción](media/functions-twitter-email/12-connection-settings.png)

4. Luego, escriba **Texto del tweet** en el cuadro de texto y haga clic en **Nuevo paso**.

    ![Especificación del texto que se va a analizar](media/functions-twitter-email/13-analyze-tweet-text.png)

Una vez configurada la detección de opinión, puede agregar una conexión a la función que consume los resultados de la puntuación de la opinión.

## <a name="connect-sentiment-output-to-your-function"></a>Conexión de la salida de opiniones con la función

1. En el Diseñador de Logic Apps, haga clic en **Nuevo paso** > **Agregar una acción**, filtre por **Azure Functions** y haga clic en **Choose an Azure function** (Elegir una función de Azure).

    ![Detectar sentimiento](media/functions-twitter-email/14-azure-functions.png)
  
4. Seleccione la aplicación de función que creó anteriormente.

    ![Seleccionar la función](media/functions-twitter-email/15-select-function.png)

5. Seleccione la función que creó para este tutorial.

    ![Seleccionar la función](media/functions-twitter-email/16-select-function.png)

4. En **Cuerpo de la solicitud**, haga clic en **Puntuación** y, a continuación, en **Guardar**.

    ![Score](media/functions-twitter-email/17-function-input-score.png)

Ahora, la función se desencadena cuando se envía una puntuación de opiniones desde la aplicación lógica. La función devuelve una clasificación de códigos de color a la aplicación lógica. A continuación, agregue una notificación por correo electrónico que se envía cuando la función devuelve un valor de opinión de **RED** (rojo). 

## <a name="add-email-notifications"></a>Agregar notificaciones por correo electrónico

La última parte del flujo de trabajo consiste en desencadenar el envío de un correo electrónico cuando la opinión obtiene una puntuación _RED_ (rojo). Este artículo utiliza un conector de Outlook.com. Puede realizar pasos similares para utilizar un conector de Gmail o de Office 365 Outlook.   

1. En el diseñador de Logic Apps, haga clic en **Nuevo paso** > **Agregar una condición**. 

    ![Agregar una condición a la aplicación lógica.](media/functions-twitter-email/18-add-condition.png)

2. Haga clic en **Elegir un valor** y, a continuación, haga clic en **Cuerpo**. Seleccione **Es igual a**, haga clic en **Elegir un valor**, escriba `RED` y haga clic en **Guardar**. 

    ![Elegir una acción para la condición.](media/functions-twitter-email/19-condition-settings.png)    

3. En **IF TRUE**, haga clic en **Agregar una acción**, busque `outlook.com`, haga clic en **Enviar un correo electrónico** e inicie sesión en su cuenta de Outlook.com.

    ![Configure el correo electrónico para la acción de envío de correo electrónico.](media/functions-twitter-email/20-add-outlook.png)

    > [!NOTE]
    > Si no tiene una cuenta de Outlook.com, puede elegir otro conector, como Gmail u Office 365 Outlook

4. En la acción **Enviar un correo electrónico**, use la configuración de correo electrónico que se especifica en la tabla. 

    ![Configure el correo electrónico para la acción de envío de correo electrónico.](media/functions-twitter-email/21-configure-email.png)
    
| Configuración      |  Valor sugerido   | Descripción  |
| ----------------- | ------------ | ------------- |
| **To** | Escriba su dirección de correo electrónico | La dirección de correo electrónico que recibe la notificación. |
| **Subject** | Detectada opinión de tweet negativa  | La línea de asunto de la notificación de correo electrónico.  |
| **Cuerpo** | Texto de tweet, ubicación | Haga clic en los parámetros **Texto de tweet** y **Ubicación**. |

1. Haga clic en **Save**(Guardar).

Ahora que el flujo de trabajo se ha completado, puede habilitar la aplicación lógica y comprobar la ejecución de la función.

## <a name="test-the-workflow"></a>Probar el flujo de trabajo

1. En el diseñador de Logic Apps, haga clic en **Ejecutar** para iniciar la aplicación.

2. En la columna izquierda, haga clic en **Información general** para ver el estado de la aplicación lógica. 
 
    ![Estado de ejecución de la aplicación lógica](media/functions-twitter-email/22-execution-history.png)

3. (Opcional) Haga clic en una de las ejecuciones para ver los detalles de la ejecución.

4. Vaya a la función, vea los registros y compruebe que los valores de opinión se reciben y procesan.
 
    ![Ver los registros de función](media/functions-twitter-email/sent.png)

5. Cuando se detecta una opinión potencialmente negativa, recibirá un correo electrónico. Si no ha recibido un correo electrónico, puede cambiar el código de la función para devolver siempre RED (rojo):

    ```csharp
    return (ActionResult)new OkObjectResult("RED");
    ```

    Después de comprobar las notificaciones de correo electrónico, vuelva a cambiar al código original:

    ```csharp
    return requestBody != null
        ? (ActionResult)new OkObjectResult(category)
        : new BadRequestObjectResult("Please pass a value on the query string or in the request body");
    ```

    > [!IMPORTANT]
    > Después de completar este tutorial, debe deshabilitar la aplicación lógica. Al deshabilitar la aplicación, evita recibir cargos por ejecuciones y agotar las transacciones de la API Cognitive Services.

Ya ha visto lo fácil que es integrar Functions en un flujo de trabajo de Logic Apps.

## <a name="disable-the-logic-app"></a>Deshabilitar la aplicación lógica

Para deshabilitar la aplicación lógica, haga clic en **Información general** y, a continuación, haga clic en **Deshabilitar** en la parte superior de la pantalla. Al deshabilitar la aplicación se evita que se ejecute, así como incurrir en cargos sin eliminar la aplicación.

![Registros de la función](media/functions-twitter-email/disable-logic-app.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un recurso de API de Cognitive Services.
> * Crear una función que clasifica las opiniones de tweet.
> * Crear una aplicación de lógica que se conecta a Twitter.
> * Agregar detección de opiniones a la aplicación lógica. 
> * Conectar la aplicación lógica a la función.
> * Enviar un correo electrónico en función de la respuesta de la función.

Continúe con el siguiente tutorial para aprender a crear una API sin servidor para la función.

> [!div class="nextstepaction"] 
> [Creación de una API sin servidor mediante Azure Functions](functions-create-serverless-api.md)

Para más información acerca de Logic Apps, consulte [Azure Logic Apps](../logic-apps/logic-apps-overview.md).
