---
title: Recepción y respuesta de llamadas mediante HTTPS
description: Uso de Azure Logic Apps para controlar solicitudes HTTPS entrantes de servicios externos
services: logic-apps
ms.suite: integration
ms.reviewers: jonfan, logicappspm
ms.topic: conceptual
ms.date: 08/27/2020
tags: connectors
ms.openlocfilehash: 05ce944d195cf43f860fc2b39975a736a4454c05
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226521"
---
# <a name="receive-and-respond-to-inbound-https-requests-in-azure-logic-apps"></a>Recepción y respuesta de solicitudes HTTPS entrantes en Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el desencadenador de solicitud y la acción de respuesta integrados, puede crear tareas y flujos de trabajo automatizados que pueden recibir solicitudes entrantes a través de HTTPS. Para enviar solicitudes salientes en su lugar, use el [desencadenador HTTP o la acción HTTP](../connectors/connectors-native-http.md) integrados.

Por ejemplo, puede hacer que la aplicación lógica:

* Reciba una solicitud HTTP de datos en una base de datos local, y responda a ella.

* Desencadene un flujo de trabajo cuando se produzca un evento de webhook externo.

* Reciba una llamada HTTPS de otra aplicación lógica, y responda e ella.

En este artículo se muestra cómo usar el desencadenador de solicitud y la acción de respuesta para que la aplicación lógica pueda recibir y responder a las llamadas entrantes.

Para información sobre el cifrado, la seguridad y la autorización de llamadas entrantes para la aplicación lógica, como la [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), conocida anteriormente como Capa de sockets seguros (SSL) o la [Autenticación abierta de Azure Active Directory Azure (Azure AD OAuth)](../active-directory/develop/index.yml), consulte [Proteger el acceso y los datos: Acceso de llamadas entrantes para desencadenadores basados en solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción, puede [registrarse para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

<a name="add-request"></a>

## <a name="add-request-trigger"></a>Agregar un desencadenador de solicitud

Este desencadenador integrado crea un punto de conexión invocable manualmente que *solo* puede administrar solicitudes entrantes través de HTTPS. Cuando un autor de llamada envía una solicitud a este punto de conexión, el [desencadenador de solicitud](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) se activa y ejecuta la aplicación lógica. Para más información sobre cómo llamar a este desencadenador, consulte [Llamada, desencadenamiento o anidamiento de aplicaciones lógicas con puntos de conexión HTTPS en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

La aplicación lógica solo mantiene abierta una solicitud entrante durante un [tiempo limitado](../logic-apps/logic-apps-limits-and-config.md#request-limits). Suponiendo que la aplicación lógica incluya una [acción de respuesta](#add-response), si la aplicación lógica no envía ninguna respuesta al autor de la llamada después de que transcurra este tiempo, la aplicación lógica devuelve un estado `504 GATEWAY TIMEOUT` al autor de la llamada. Si la aplicación lógica no incluye una acción de respuesta, 
> devuelve inmediatamente un estado `202 ACCEPTED` al autor de la llamada.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Crear una aplicación lógica en blanco.

1. Cuando se abra el Diseñador de aplicación lógica, en el cuadro de búsqueda, especifique `http request` como filtro. En la lista de desencadenadores, seleccione el desencadenador **Cuando se recibe una solicitud HTTP**.

   ![Seleccionar desencadenador de solicitud](./media/connectors-native-reqres/select-request-trigger.png)

   El desencadenador de solicitud muestra estas propiedades:

   ![Desencadenador de solicitud](./media/connectors-native-reqres/request-trigger.png)

   | Nombre de propiedad | Nombre de la propiedad JSON | Obligatorio | Descripción |
   |---------------|--------------------|----------|-------------|
   | **URL de HTTP POST** | {none} | Sí | URL del punto de conexión que se genera después de guardar la aplicación lógica y se usa para llamar a la aplicación lógica. |
   | **Esquema JSON del cuerpo de la solicitud** | `schema` | No | Esquema JSON que describe las propiedades y los valores del cuerpo de la solicitud entrante. |
   |||||

1. En el cuadro **Esquema JSON de cuerpo de solicitud**, también puede especificar un esquema JSON que describa el cuerpo de la solicitud HTTP en la solicitud entrante, por ejemplo:

   ![Ejemplo de esquema JSON](./media/connectors-native-reqres/provide-json-schema.png)

   El diseñador usa este esquema para generar tokens para las propiedades de la solicitud. De este modo, la aplicación lógica puede analizar, usar y pasar los datos de la solicitud a través del desencadenador al flujo de trabajo.

   A continuación, le mostramos un esquema de ejemplo:

   ```json
   {
      "type": "object",
      "properties": {
         "account": {
            "type": "object",
            "properties": {
               "name": {
                  "type": "string"
               },
               "ID": {
                  "type": "string"
               },
               "address": {
                  "type": "object",
                  "properties": {
                     "number": {
                        "type": "string"
                     },
                     "street": {
                        "type": "string"
                     },
                     "city": {
                        "type": "string"
                     },
                     "state": {
                        "type": "string"
                     },
                     "country": {
                        "type": "string"
                     },
                     "postalCode": {
                        "type": "string"
                     }
                  }
               }
            }
         }
      }
   }
   ```

   Al escribir un esquema JSON, el diseñador muestra un recordatorio para incluir el encabezado `Content-Type` en la solicitud y establecer el valor del encabezado en `application/json`. Para obtener más información, consulte [Control de tipos de contenido](../logic-apps/logic-apps-content-type.md).

   ![Recordatorio para incluir el encabezado "Content-Type"](./media/connectors-native-reqres/include-content-type.png)

   Este es el aspecto de este encabezado en formato JSON:

   ```json
   {
      "Content-Type": "application/json"
   }
   ```

   Para generar un esquema JSON basado en la carga esperada (datos), puede usar una herramienta como [JSONSchema.net](https://jsonschema.net) o puede seguir estos pasos:

   1. En el desencadenador de solicitud, seleccione **Use sample payload to generate schema** (Usar una carga de ejemplo para generar el esquema).

      ![Captura de pantalla con la opción "Usar una carga de ejemplo para generar el esquema" seleccionada](./media/connectors-native-reqres/generate-from-sample-payload.png)

   1. Escriba la carga de ejemplo y seleccione **Listo**.

      ![Especificación de una carga de ejemplo para generar el esquema](./media/connectors-native-reqres/enter-payload.png)

      Este es el ejemplo de la carga:

      ```json
      {
         "account": {
            "name": "Contoso",
            "ID": "12345",
            "address": {
               "number": "1234",
               "street": "Anywhere Street",
               "city": "AnyTown",
               "state": "AnyState",
               "country": "USA",
               "postalCode": "11111"
            }
         }
      }
      ```

1. Para comprobar que la llamada entrante tiene un cuerpo de solicitud que coincide con el esquema especificado, siga estos pasos:

   1. En la barra de título del desencadenador de solicitud, seleccione el botón de puntos suspensivos ( **...** ).

   1. En la configuración del desencadenador, active **Validación de esquema** y seleccione **Listo**.

      Si el cuerpo de la solicitud de la llamada entrante no coincide con el esquema, el desencadenador devuelve un error `HTTP 400 Bad Request`.

1. Para especificar propiedades adicionales, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera agregar.

   | Nombre de propiedad | Nombre de la propiedad JSON | Obligatorio | Descripción |
   |---------------|--------------------|----------|-------------|
   | **Método** | `method` | No | Método que la solicitud entrante debe usar para llamar a la aplicación lógica |
   | **Ruta de acceso relativa** | `relativePath` | No | Ruta de acceso relativa del parámetro que la URL del punto de conexión de la aplicación lógica puede aceptar |
   |||||

   En este ejemplo, se agrega la propiedad **Method**:

   ![Agregar el parámetro Method](./media/connectors-native-reqres/add-parameters.png)

   La propiedad **Method** aparece en el desencadenador para que pueda seleccionar un método de la lista.

   ![Selección del método](./media/connectors-native-reqres/select-method.png)

1. Ahora, agregue otra acción como paso siguiente en el flujo de trabajo. En el desencadenador, seleccione **Paso siguiente** para poder buscar la acción que quiere agregar.

   Por ejemplo, puede responder a la solicitud [agregando una acción de respuesta](#add-response), que puede usar para devolver una respuesta personalizada y se describe más adelante en este tema.

   La aplicación lógica solo mantiene abierta la solicitud entrante durante un [tiempo limitado](../logic-apps/logic-apps-limits-and-config.md#request-limits). Suponiendo que el flujo de trabajo de la aplicación lógica incluye una acción de respuesta, si la aplicación lógica no devuelve ninguna respuesta después de que transcurra este tiempo, la aplicación lógica devuelve `504 GATEWAY TIMEOUT` al autor de llamada. De lo contrario, si la aplicación lógica no incluye ninguna acción de respuesta, dicha aplicación lógica devuelve inmediatamente una respuesta `202 ACCEPTED` al autor de la llamada.

1. Cuando haya terminado, guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   Este paso genera la URL que se usará para enviar la solicitud que desencadena la aplicación lógica. Para copiar esta URL, seleccione el icono de copia que se encuentra junto a la URL.

   ![URL que se usa para desencadenar la aplicación lógica](./media/connectors-native-reqres/generated-url.png)

   > [!NOTE]
   > Si quiere incluir el símbolo de hash o de almohadilla ( **#** ) en el URI al hacer una llamada al desencadenador de solicitud, use mejor esta versión codificada: `%25%23`

1. Para desencadenar la aplicación lógica, envíe una solicitud HTTP POST a la URL generada.

   Por ejemplo, puede usar una herramienta como [Postman](https://www.getpostman.com/) para enviar HTTP POST. Para obtener más información sobre la definición JSON subyacente del desencadenador y sobre cómo llamar a este desencadenador, vea estos temas: [Tipo de desencadenador de solicitud](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) y [Llamada, desencadenamiento o anidamiento de aplicaciones lógicas con puntos de conexión HTTP en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

## <a name="trigger-outputs"></a>Salidas del desencadenador

Aquí encontrará más información sobre las salidas del desencadenador de solicitud:

| Nombre de la propiedad JSON | Tipo de datos | Descripción |
|--------------------|-----------|-------------|
| `headers` | Object | Objeto JSON que describe los encabezados de la solicitud |
| `body` | Object | Objeto JSON que describe el contenido del cuerpo de la solicitud |
||||

<a name="add-response"></a>

## <a name="add-a-response-action"></a>Adición de una acción de respuesta

Al usar el desencadenador de solicitud para administrar las solicitudes entrantes, puede modelar la respuesta y devolver los resultados de la carga al autor de la llamada mediante la [acción de respuesta](../logic-apps/logic-apps-workflow-actions-triggers.md#response-action) integrada. Puede usar la acción de respuesta *solo* con el desencadenador de solicitud. Esta combinación con el desencadenador de solicitud y la acción de respuesta crea el [patrón de solicitud-respuesta](https://en.wikipedia.org/wiki/Request%E2%80%93response). Excepto dentro de los bucles Foreach, los bucles Until y las ramas paralelas, se puede agregar una acción de respuesta en cualquier lugar del flujo de trabajo.

> [!IMPORTANT]
> Si una acción de respuesta incluye estos encabezados, Logic Apps quita estos encabezados del mensaje de respuesta generado sin mostrar ninguna advertencia o error:
>
> * `Allow`
> * `Content-*` con estas excepciones: `Content-Disposition`, `Content-Encoding` y `Content-Type`
> * `Cookie`
> * `Expires`
> * `Last-Modified`
> * `Set-Cookie`
> * `Transfer-Encoding`
>
> Aunque Logic Apps no le impedirá guardar aplicaciones lógicas que tengan una acción de respuesta con estos encabezados, Logic Apps omite estos encabezados.

1. En el Diseñador de aplicación lógica, vaya al paso en que debe agregar una acción de respuesta y seleccione **Nuevo paso**.

   Por ejemplo, mediante el desencadenador de solicitud anterior:

   ![Adición de un nuevo paso](./media/connectors-native-reqres/add-response.png)

   Para agregar una acción entre un paso y otro, mueva el puntero por encima de la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `response` como filtro y seleccione la acción **Respuesta**.

   ![Seleccionar la acción Respuesta](./media/connectors-native-reqres/select-response-action.png)

   El desencadenador de solicitud se contrae en este ejemplo por motivos de simplicidad.

1. Agregue los valores necesarios para el mensaje de respuesta.

   En algunos campos, al hacer clic dentro de los cuadros, se abre la lista de contenido dinámico. A continuación, puede seleccionar los tokens que representan las salidas disponibles de los pasos anteriores del flujo de trabajo. Las propiedades del esquema especificado en el ejemplo anterior ahora aparecen en la lista de contenido dinámico.

   Por ejemplo, para el cuadro **Encabezados**, incluya `Content-Type` como nombre de clave y establezca el valor de clave en `application/json` como se mencionó anteriormente en este tema. Para el cuadro **Cuerpo**, puede seleccionar la salida del cuerpo del desencadenador en la lista de contenido dinámico.

   ![Detalles de la acción de respuesta](./media/connectors-native-reqres/response-details.png)

   Para ver los encabezados en formato JSON, seleccione **Switch to text view** (Cambiar a la vista de texto).

   ![Encabezados: cambiar a la vista de texto](./media/connectors-native-reqres/switch-to-text-view.png)

   Aquí encontrará más información sobre las propiedades que puede establecer en la acción Respuesta.

   | Nombre de propiedad | Nombre de la propiedad JSON | Obligatorio | Descripción |
   |---------------|--------------------|----------|-------------|
   | **Código de estado** | `statusCode` | Sí | Código de estado que se devolverá en la respuesta |
   | **Encabezados** | `headers` | No | Objeto JSON que describe uno o más encabezados que se incluirán en la respuesta |
   | **Cuerpo** | `body` | No | Cuerpo de la respuesta |
   |||||

1. Para especificar propiedades adicionales, como un esquema JSON para el cuerpo de respuesta, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera agregar.

1. Cuando haya terminado, guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

## <a name="next-steps"></a>Pasos siguientes

* [Proteger el acceso y datos: Acceso de llamadas entrantes para desencadenadores basados en solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
* [Conectores de Logic Apps](../connectors/apis-list.md)