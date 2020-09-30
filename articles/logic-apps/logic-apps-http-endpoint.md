---
title: Llamada, desencadenamiento o anidamiento de aplicaciones lógicas mediante el uso de desencadenadores de solicitud
description: Configuración de puntos de conexión HTTPS para llamadas, desencadenamientos o anidamiento de flujos de trabajo de aplicaciones lógicas en Azure Logic Apps
services: logic-apps
ms.workload: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 08/27/2020
ms.openlocfilehash: 5032676848536f0b9498cf4beecf86277484a901
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230813"
---
# <a name="call-trigger-or-nest-logic-apps-by-using-https-endpoints-in-azure-logic-apps"></a>Llamada, desencadenamiento o anidamiento de aplicaciones lógicas con puntos de conexión HTTPS en Azure Logic Apps

Para que se pueda llamar a la aplicación lógica a través de una dirección URL con el fin de que pueda recibir solicitudes entrantes de otros servicios, puede exponer de forma nativa un punto de conexión HTTPS sincrónico como un desencadenador en esa aplicación lógica. Con esta funcionalidad, puede llamar a la aplicación lógica desde otras aplicaciones lógicas y crear un patrón de puntos de conexión a los que se puede llamar. Para configurar un punto de conexión al que se pueda llamar para controlar las llamadas entrantes, puede usar cualquiera de estos tipos de desencadenador:

* [Solicitud](../connectors/connectors-native-reqres.md)
* [Webhook HTTP](../connectors/connectors-native-webhook.md)
* Desencadenadores de conectores administrados del tipo [ApiConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) que pueden recibir solicitudes HTTPS entrantes

En este artículo se muestra cómo crear un punto de conexión al que se puede llamar en la aplicación lógica mediante el desencadenador de solicitud y llamar a ese punto de conexión desde otra aplicación lógica. Todos los principios se aplican de forma idéntica a los otros tipos de desencadenadores que se pueden usar para recibir solicitudes entrantes.

Para información sobre el cifrado, la seguridad y la autorización de llamadas entrantes para la aplicación lógica, como la [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), conocida anteriormente como Capa de sockets seguros (SSL) o la [Autenticación abierta de Azure Active Directory Azure (Azure AD OAuth)](../active-directory/develop/index.yml), consulte [Proteger el acceso y los datos: Acceso de llamadas entrantes para desencadenadores basados en solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* La aplicación lógica en la que desea usar el desencadenador para crear el punto de conexión al que se puede llamar. Puede empezar con una aplicación lógica en blanco o con una aplicación lógica existente en la que puede reemplazar el desencadenador actual. Este ejemplo comienza con una aplicación lógica en blanco. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-a-callable-endpoint"></a>Creación de un punto de conexión al que se puede llamar

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Cree y abra una aplicación lógica en blanco en el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, seleccione **Integrado**. En el cuadro de búsqueda, escriba `request` como filtro. En la lista de desencadenadores, seleccione **Cuando se recibe una solicitud HTTP**.

   ![Buscar y seleccionar el desencadenador de solicitud](./media/logic-apps-http-endpoint/find-and-select-request-trigger.png)

1. De manera opcional, en el cuadro **Esquema JSON de cuerpo de solicitud**, puede especificar un esquema JSON que describa la carga o los datos que espera que reciba el desencadenador.

   El diseñador usa este esquema para generar tokens que representan los resultados del desencadenador. Luego puede hacer referencia fácilmente a estos resultados en el flujo de trabajo de la aplicación lógica. Obtenga más información sobre [tokens generados a partir de esquemas JSON](#generated-tokens).

   Para este ejemplo, escriba este esquema:

   ```json
      {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "streetNumber": {
                  "type": "string"
               },
               "streetName": {
                  "type": "string"
               },
               "town": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "string"
               }
            }
         }
      }
   }
    ```

   ![Proporcionar el esquema JSON para la acción de solicitud](./media/logic-apps-http-endpoint/manual-request-trigger-schema.png)

   O bien, puede generar un esquema JSON si proporciona una carga de ejemplo:

   1. En el desencadenador **Solicitud**, seleccione **Usar una carga de ejemplo para generar el esquema**.

   1. En el cuadro **Especificar o pegar una carga de JSON de ejemplo**, proporcione una carga de muestra, por ejemplo:

      ```json
      {
         "address": {
            "streetNumber": "00000",
            "streetName": "AnyStreet",
            "town": "AnyTown",
            "postalCode": "11111-1111"
        }
      }
      ```

   1. Cuando lo tenga todo preparado, seleccione **Listo**.

      El cuadro **Esquema JSON de cuerpo de solicitud** muestra ahora el esquema generado.

1. Guarde la aplicación lógica.

   En el cuadro **Dirección URL de HTTP POST** ahora muestra la dirección URL de devolución de llamada generada que otros servicios pueden usar para llamar a la aplicación lógica y desencadenarla. Esta dirección URL incluye parámetros de consulta que especifican una clave de firma de acceso compartido (SAS), la cual se usa para la autenticación.

   ![Dirección URL de devolución de llamadas generada para el punto de conexión](./media/logic-apps-http-endpoint/generated-endpoint-url.png)

1. Para copiar la dirección URL de devolución de llamada, tiene estas opciones:

   * A la derecha del cuadro **Dirección URL de HTTP POST**, seleccione **Copiar dirección URL** (icono Copiar archivos).

   * Realice esta llamada POST:

     `POST https://management.azure.com/{logic-app-resource-ID}/triggers/{endpoint-trigger-name}/listCallbackURL?api-version=2016-06-01`

   * Copie la dirección URL de devolución de llamada del panel de **Información general** de la aplicación lógica.

     1. En el menú de la aplicación lógica, seleccione **Introducción**.

     1. En la sección **Resumen**, seleccione **Ver el historial de desencadenadores**.

        ![Obtención de la dirección URL del punto de conexión en Azure Portal](./media/logic-apps-http-endpoint/find-manual-trigger-url.png)

     1. En **URL de devolución de llamadas [POST]** , copie la dirección URL:

        ![Copiar la dirección URL del punto de conexión en Azure Portal](./media/logic-apps-http-endpoint/copy-manual-trigger-callback-url-post.png)

<a name="select-method"></a>

## <a name="select-expected-request-method"></a>Seleccionar método de solicitud esperado

De manera predeterminada, el desencadenador de solicitud espera una solicitud POST. Puede especificar un método esperado diferente, pero solo uno.

1. En el desencadenador de solicitud, abra la lista **Agregar nuevo parámetro** y seleccione **Método**, que agrega esta propiedad al desencadenador.

   ![Agregar la propiedad "Método" al desencadenador](./media/logic-apps-http-endpoint/select-add-new-parameter-for-method.png)

1. En la lista **Método**, seleccione el método que el desencadenador debe esperar en su lugar. O bien, puede especificar un método personalizado.

   Por ejemplo, seleccione el método **GET** para que pueda probar la dirección URL del punto de conexión más adelante.

   ![Seleccionar el método de solicitud esperado por el desencadenador](./media/logic-apps-http-endpoint/select-method-request-trigger.png)

<a name="endpoint-url-parameters"></a>

## <a name="pass-parameters-through-endpoint-url"></a>Pasar parámetros a través de la dirección URL del punto de conexión

Si desea aceptar valores de parámetros a través de la dirección URL del punto de conexión, tiene estas opciones:

* [Acepte valores a través de los parámetros GET](#get-parameters) o parámetros de URL.

  Estos valores se pasan como pares de nombre-valor en la dirección URL del punto de conexión. Para esta opción, debe usar el método GET en el desencadenador de solicitud. En una acción posterior, puede obtener los valores de parámetro como salidas del desencadenador mediante la función `triggerOutputs()` en una expresión.

* [Acepte valores a través de una ruta de acceso relativa](#relative-path) para los parámetros en el desencadenador de solicitud.

  Estos valores se pasan a través de una ruta de acceso relativa en la dirección URL del punto de conexión. También debe [seleccionar explícitamente el método](#select-method) que espera el desencadenador. En una acción posterior, puede obtener los valores de parámetro como salidas del desencadenador al hacer referencia directamente a esas salidas.

<a name="get-parameters"></a>

### <a name="accept-values-through-get-parameters"></a>Aceptar valores a través de los parámetros GET

1. En el desencadenador de solicitud, abra la lista **Agregar nuevo parámetro**, agregue la propiedad **Método** al desencadenador y seleccione el método **GET**.

   Para obtener más información, consulte [Seleccionar método de solicitud esperado](#select-method).

1. En el desencadenador de solicitud, agregue la acción en la que desea usar el valor de parámetro. En este ejemplo, agregue la acción **Respuesta**.

   1. En el desencadenador de solicitud, seleccione **Nuevo paso** > **Agregar una acción**.
   
   1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `response` como filtro. En la lista de acciones, seleccione la acción **Respuesta**.

1. Para compilar la expresión `triggerOutputs()` que recupera el valor de parámetro, siga estos pasos:

   1. Haga clic dentro de la propiedad **Cuerpo** de la acción de respuesta para que aparezca la lista de contenido dinámico y seleccione **Expresión**.

   1. En el cuadro **Expresión**, escriba esta expresión, sustituya `parameter-name` por el nombre del parámetro y seleccione **Aceptar**.

      `triggerOutputs()['queries']['parameter-name']`

      ![Agregar la expresión "triggerOutputs()" al desencadenador](./media/logic-apps-http-endpoint/trigger-outputs-expression.png)

      En la propiedad **Cuerpo**, la expresión se resuelve en el token de `triggerOutputs()`.

      ![Expresión "triggerOutputs ()" resuelta](./media/logic-apps-http-endpoint/trigger-outputs-expression-token.png)

      Si guarda la aplicación lógica, sale del diseñador y vuelve al diseñador, el token muestra el nombre del parámetro especificado, por ejemplo:

      ![Expresión resuelta para el nombre de parámetro](./media/logic-apps-http-endpoint/resolved-expression-parameter-token.png)

      En la vista de código, la propiedad **Cuerpo** aparece en la definición de la acción de respuesta de la manera siguiente:

      `"body": "@{triggerOutputs()['queries']['parameter-name']}",`

      Por ejemplo, supongamos que desea pasar un valor para un parámetro denominado `postalCode`. La propiedad **Cuerpo** especifica la cadena `Postal Code: ` con un espacio al final, seguido de la expresión correspondiente:

      ![Agregar la expresión "triggerOutputs()" de ejemplo al desencadenador](./media/logic-apps-http-endpoint/trigger-outputs-expression-postal-code.png)

1. Para probar el punto de conexión al que se puede llamar, copie la URL de devolución de llamada del desencadenador de solicitud y pegue la dirección URL en otra ventana del explorador. En la dirección URL, agregue el nombre y el valor del parámetro después del signo de interrogación (`?`) a la dirección URL con el siguiente formato y presione Entrar.

   `...?{parameter-name=parameter-value}&api-version=2016-10-01...`

   `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?{parameter-name=parameter-value}&api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

   El explorador devuelve una respuesta con este texto: `Postal Code: 123456`

   ![Respuesta de la solicitud de envío a la URL de devolución de llamada](./media/logic-apps-http-endpoint/callback-url-returned-response.png)

1. Para colocar el nombre y el valor del parámetro en una posición diferente en la dirección URL, asegúrese de usar la y comercial (`&`) como prefijo, por ejemplo:

   `...?api-version=2016-10-01&{parameter-name=parameter-value}&...`

   En este ejemplo se muestra la dirección URL de devolución de llamada con el valor y el nombre del parámetro de ejemplo `postalCode=123456` en posiciones diferentes dentro de la dirección URL:

   * Primera posición: `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?postalCode=123456&api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

   * Segunda posición: `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?api-version=2016-10-01&postalCode=123456&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

> [!NOTE]
> Si quiere incluir la almohadilla o numeral ( **#** ) en el URI, use esta versión codificada en su lugar: `%25%23`.

<a name="relative-path"></a>

### <a name="accept-values-through-a-relative-path"></a>Aceptar valores a través de una ruta de acceso relativa

1. En el desencadenador de solicitud, abra la lista **Agregar nuevo parámetro** y seleccione **Puerta de acceso relativa** para agregar esta propiedad al desencadenador.

   ![Agregar la propiedad "Ruta de acceso relativa" al desencadenador](./media/logic-apps-http-endpoint/select-add-new-parameter-for-relative-path.png)

1. En la propiedad **Ruta de acceso relativa**, especifique la ruta de acceso relativa para el parámetro en el esquema JSON que quiera que acepte la dirección URL, por ejemplo, `/address/{postalCode}`.

   ![Especificar la ruta de acceso relativa para el parámetro](./media/logic-apps-http-endpoint/relative-path-url-value.png)

1. En el desencadenador de solicitud, agregue la acción en la que desea usar el valor de parámetro. En este ejemplo, agregue la acción **Respuesta**.

   1. En el desencadenador de solicitud, seleccione **Nuevo paso** > **Agregar una acción**.

   1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `response` como filtro. En la lista de acciones, seleccione la acción **Respuesta**.

1. En la propiedad **Cuerpo** de la acción de respuesta, incluya el token que representa el parámetro especificado en la ruta de acceso relativa del desencadenador.

   Por ejemplo, supongamos que quiere que la acción de respuesta devuelva `Postal Code: {postalCode}`.

   1. En la propiedad **Cuerpo**, escriba `Postal Code: ` con un espacio al final. Mantenga el cursor dentro del cuadro de edición para que la lista de contenido dinámico permanezca abierta.

   1. En la lista de contenido dinámico, en la sección **Cuando se recibe una solicitud HTTP**, seleccione el token **postalCode**.

      ![Agregar el parámetro al cuerpo de la respuesta](./media/logic-apps-http-endpoint/relative-url-with-parameter-token.png)

      La propiedad **Cuerpo** ahora incluye el parámetro seleccionado:

      ![Ejemplo de cuerpo de respuesta con el parámetro](./media/logic-apps-http-endpoint/relative-url-with-parameter.png)

1. Guarde la aplicación lógica.

   En el desencadenador de solicitud, la dirección URL de devolución de llamada se actualiza y ahora incluye la ruta de acceso relativa, por ejemplo:

   `https://prod-07.westus.logic.azure.com/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke/address/{postalCode}?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

1. Para probar el punto de conexión al que se puede llamar, copie la URL de devolución de llamada actualizada del desencadenador de solicitud, pegue la dirección URL en otra ventana del explorador, reemplace `{postalCode}` en la dirección URL con `123456`y presione Entrar.

   El explorador devuelve una respuesta con este texto: `Postal Code: 123456`

   ![Respuesta de la solicitud de envío a la URL de devolución de llamada](./media/logic-apps-http-endpoint/callback-url-returned-response.png)

> [!NOTE]
> Si quiere incluir la almohadilla o numeral ( **#** ) en el URI, use esta versión codificada en su lugar: `%25%23`.

## <a name="call-logic-app-through-endpoint-url"></a>Llamada a la aplicación lógica a través de la dirección URL de un punto de conexión

Después de crear el punto de conexión, puede desencadenar la aplicación lógica mediante el envío de una solicitud HTTPS `POST` a la dirección URL completa del punto de conexión. Las aplicaciones lógicas tienen compatibilidad integrada para los puntos de conexión de acceso directo.

<a name="generated-tokens"></a>

## <a name="tokens-generated-from-schema"></a>Tokens generados a partir de un esquema

Cuando se proporciona un esquema JSON en el desencadenador de solicitud, el Diseñador de aplicación lógica genera tokens para las propiedades de ese esquema. Después puede usar esos tokens para pasar datos a través del flujo de trabajo de la aplicación lógica.

Por ejemplo, si agrega más propiedades, como `"suite"`, al esquema JSON, los tokens de esas propiedades estarán disponibles para su uso en los pasos posteriores de su aplicación lógica. Este es el esquema JSON completo:

```json
   {
   "type": "object",
   "properties": {
      "address": {
         "type": "object",
         "properties": {
            "streetNumber": {
               "type": "string"
            },
            "streetName": {
               "type": "string"
            },
            "suite": {
               "type": "string"
            },
            "town": {
               "type": "string"
            },
            "postalCode": {
               "type": "string"
            }
         }
      }
   }
}
```

## <a name="create-nested-logic-apps"></a>Creación de aplicaciones lógicas anidadas

Puede anidar los flujos de trabajo en la aplicación lógica mediante la adición de otras aplicaciones de lógica que pueden recibir solicitudes. Para incluir estas aplicaciones lógicas, siga los pasos siguientes:

1. En el paso en el que quiera llamar a otra aplicación lógica, seleccione **Nuevo paso** > **Agregar una acción**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba `logic apps` como filtro. En la lista de acciones, seleccione **Elegir un flujo de trabajo de Logic Apps**.

   ![Anidar una aplicación lógica dentro de la aplicación lógica actual](./media/logic-apps-http-endpoint/choose-logic-apps-workflow.png)

   El diseñador muestra las aplicaciones lógicas que se pueden seleccionar.

1. Seleccione la aplicación lógica a la que se llamará desde la aplicación lógica actual.

   ![Seleccionar la aplicación lógica a la que se llamará desde la aplicación lógica actual](./media/logic-apps-http-endpoint/select-logic-app-to-nest.png)

## <a name="reference-content-from-an-incoming-request"></a>Referencia al contenido de una solicitud entrante

Si el tipo de contenido de la solicitud entrante es `application/json`, puede hacer referencia a las propiedades en la solicitud entrante. En caso contrario, este contenido se trata como una sola unidad binaria que se puede pasar a otras API. Para hacer referencia a este contenido dentro del flujo de trabajo de la aplicación lógica, primero debe convertir el contenido.

Por ejemplo, si va a pasar contenido de tipo `application/xml`, puede usar la [expresión `@xpath()`](../logic-apps/workflow-definition-language-functions-reference.md#xpath) para realizar una extracción de XPath, o bien puede usar la [expresión `@json()`](../logic-apps/workflow-definition-language-functions-reference.md#json) para convertir XML a JSON. Obtenga más información sobre cómo trabajar con los [tipos de contenido](../logic-apps/logic-apps-content-type.md) admitidos.

Para obtener la salida de una solicitud entrante, puede usar la [expresión `@triggerOutputs`](../logic-apps/workflow-definition-language-functions-reference.md#triggerOutputs). Por ejemplo, supongamos que tiene un resultado similar al de este ejemplo:

```json
{
   "headers": {
      "content-type" : "application/json"
   },
   "body": {
      "myProperty" : "property value"
   }
}
```

Para acceder específicamente a la propiedad `body`, puede usar la [expresión `@triggerBody()`](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody) como un acceso directo.

## <a name="respond-to-requests"></a>Respuesta a solicitudes

Es posible que a veces quiera responder a determinadas solicitudes que desencadenan una aplicación lógica mediante la devolución de contenido al autor de la llamada. Para construir el código de estado, el encabezado y el cuerpo de la respuesta, use la acción de respuesta. Esta acción puede aparecer en cualquier lugar de la aplicación lógica, no solo al final del flujo de trabajo. Si la aplicación lógica no incluye una acción de respuesta, el punto de conexión responde *inmediatamente* con un estado **202 - Aceptado**.

Para que el autor original de la llamada obtenga la respuesta de forma correcta, todos los pasos necesarios para la respuesta deben terminar dentro del [límite del tiempo de espera de la solicitud](./logic-apps-limits-and-config.md), a menos que se llame a la aplicación lógica desencadenada como una aplicación lógica anidada. Si no se devuelve ninguna respuesta dentro de este límite, la solicitud entrante agota el tiempo de espera y recibe la respuesta **408 - Tiempo de espera del cliente agotado**.

En cuanto a las aplicaciones lógicas anidadas, la aplicación lógica principal seguirá esperando una respuesta hasta que se completen todos los pasos, independientemente del tiempo que se necesite.

### <a name="construct-the-response"></a>Construcción de la respuesta

En el cuerpo de la respuesta, puede incluir varios encabezados y cualquier tipo de contenido. Por ejemplo, el encabezado de esta respuesta especifica que el tipo de contenido de la respuesta es `application/json` y que el cuerpo contiene valores de las propiedades `town` y `postalCode`, en función del esquema JSON descrito anteriormente en este tema para el desencadenador de solicitud.

![Proporcionar el contenido de la respuesta para la acción de respuesta HTTPS](./media/logic-apps-http-endpoint/content-for-response-action.png)

Las respuestas tienen estas propiedades:

| Propiedad (nombre) | Propiedad (JSON) | Descripción |
|--------------------|-----------------|-------------|
| **Código de estado** | `statusCode` | Código de estado HTTPS que se usará en la respuesta para la solicitud entrante. Este código puede ser cualquier código de estado válido que comience con 2xx, 4xx o 5xx. En cambio, no se permiten códigos de estado 3xx. |
| **Encabezados** | `headers` | Uno o más encabezados que se incluirán en la respuesta. |
| **Cuerpo** | `body` | Un objeto de cuerpo que puede ser una cadena, un objeto JSON o incluso contenido binario al que se hace referencia desde un paso anterior. |
||||

Para ver la definición de JSON de la acción de respuesta y la definición completa de JSON de la aplicación lógica, en la barra de herramientas del Diseñador de aplicación lógica, seleccione **Vista Código**.

``` json
"Response": {
   "type": "Response",
   "kind": "http",
   "inputs": {
      "body": {
         "postalCode": "@triggerBody()?['address']?['postalCode']",
         "town": "@triggerBody()?['address']?['town']"
      },
      "headers": {
         "content-type": "application/json"
      },
      "statusCode": 200
   },
   "runAfter": {}
}
```

## <a name="q--a"></a>Preguntas y respuestas

#### <a name="q-what-about-url-security"></a>P: ¿Qué se puede decir sobre la seguridad de las direcciones URL?

**R.** : Azure genera direcciones URL de devolución de llamada de la aplicación lógica de manera segura con una [firma de acceso compartido (SAS)](/rest/api/storageservices/delegate-access-with-shared-access-signature). Esta firma pasa como un parámetro de consulta y debe validarse antes de que se pueda ejecutar la aplicación lógica. Azure genera la firma mediante una combinación única de una clave secreta por aplicación lógica, el nombre del desencadenador y la operación que se realiza. Por tanto, a menos que alguien tenga acceso a la clave de aplicación lógica secreta, no se puede generar una firma válida.

> [!IMPORTANT]
> En sistemas de más alta seguridad y de producción, se recomienda encarecidamente no llamar a la aplicación lógica directamente desde el explorador. Los motivos son estos:
>
> * La clave de acceso compartido aparece en la dirección URL.
> * No puede administrar directivas de contenido de seguridad debido a los dominios compartidos entre los clientes de Azure Logic Apps.

#### <a name="q-can-i-configure-callable-endpoints-further"></a>P: ¿Puedo configurar más puntos de conexión que se puedan llamar?

**R.** : Sí, los puntos de conexión HTTPS admiten una configuración más avanzada con [Azure API Management](../api-management/api-management-key-concepts.md). Este servicio también ofrece la funcionalidad de administrar de forma coherente todas las API, incluidas las aplicaciones lógicas, configurar nombres de dominio personalizados, usar varios métodos de autenticación y muchas más, como, por ejemplo:

* [Cambio del método de solicitud](../api-management/api-management-advanced-policies.md#SetRequestMethod)
* [Cambio de los segmentos de dirección URL de la solicitud](../api-management/api-management-transformation-policies.md#RewriteURL)
* Configuración de los dominios de API Management en [Azure Portal](https://portal.azure.com/)
* Configuración de la directiva para comprobar la autenticación básica

## <a name="next-steps"></a>Pasos siguientes

* [Recepción de llamadas HTTPS entrantes y respuesta e ellas mediante Azure Logic Apps](../connectors/connectors-native-reqres.md)
* [Protección del acceso y los datos en Azure Logic Apps - Acceso a desencadenadores de solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
