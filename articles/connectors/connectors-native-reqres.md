---
title: Recepción y respuesta de llamadas mediante HTTPS
description: Uso de Azure Logic Apps para controlar solicitudes HTTPS entrantes de servicios externos
services: logic-apps
ms.suite: integration
ms.reviewers: jonfan, logicappspm
ms.topic: conceptual
ms.date: 05/29/2020
tags: connectors
ms.openlocfilehash: ae34840c04c3a1d2fb3646046792c97ed6f521a0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289443"
---
# <a name="receive-and-respond-to-inbound-https-requests-in-azure-logic-apps"></a>Recepción y respuesta de solicitudes HTTPS entrantes en Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el desencadenador de solicitud y la acción de respuesta integrados, puede crear tareas y flujos de trabajo automatizados que reciben solicitudes HTTP entrantes y responden a ellas. Por ejemplo, puede hacer que la aplicación lógica:

* Reciba una solicitud HTTP de datos en una base de datos local, y responda a ella.

* Desencadene un flujo de trabajo cuando se produzca un evento de webhook externo.

* Reciba una llamada HTTPS de otra aplicación lógica, y responda e ella.

El desencadenador de solicitud admite [Azure Active Directory Open Authentication](../active-directory/develop/index.yml) (Azure AD OAuth) para autorizar llamadas entrantes a la aplicación lógica. Para obtener más información sobre la habilitación de esta autenticación, vea [Proteger el acceso y los datos en Azure Logic Apps - Habilitación de la autenticación Azure AD OAuth](../logic-apps/logic-apps-securing-a-logic-app.md#enable-oauth).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción, puede [registrarse para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Conocimientos básicos sobre [aplicaciones lógicas](../logic-apps/logic-apps-overview.md). Si es la primera vez que interactúa con las aplicaciones lógicas, consulte el artículo sobre [cómo crear la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="tls-support"></a>

## <a name="transport-layer-security-tls"></a>Seguridad de la capa de transporte (TLS)

* Las llamadas de entrada *solo* admiten Seguridad de la capa de transporte (TLS) 1.2. Si obtiene errores de protocolo de enlace TLS, asegúrese de usar TLS 1.2. Para más información, consulte [Solución del problema de TLS 1.0](/security/solving-tls1-problem). Las llamadas de salida admiten TLS 1.0, 1.1 y 1.2, en función de la capacidad del punto de conexión de destino.

* Las llamadas de entrada admiten estos conjuntos de cifrado:

  * TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384

  * TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256

  * TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

  * TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

  * TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384

  * TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256

  * TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

  * TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

<a name="add-request"></a>

## <a name="add-request-trigger"></a>Agregar un desencadenador de solicitud

Este desencadenador integrado crea un punto de conexión HTTPS invocable manualmente que *solo* puede recibir solicitudes HTTPS entrantes. Cuando se produce este evento, el desencadenador se activa y ejecuta la aplicación lógica. Para obtener más información sobre la definición JSON subyacente del desencadenador y sobre cómo llamar a este desencadenador, vea [Tipo de desencadenador de solicitud](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) y [Flujos de trabajo de llamada, desencadenador o anidamiento con puntos de conexión HTTP en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Crear una aplicación lógica en blanco.

1. Cuando se abra el Diseñador de aplicación lógica, en el cuadro de búsqueda, especifique `http request` como filtro. En la lista de desencadenadores, seleccione el desencadenador **Cuando se recibe una solicitud HTTP**, que es el primer paso del flujo de trabajo de la aplicación lógica.

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

      ![Generar un esquema a partir de la carga](./media/connectors-native-reqres/generate-from-sample-payload.png)

   1. Escriba la carga de ejemplo y seleccione **Listo**.

      ![Generar un esquema a partir de la carga](./media/connectors-native-reqres/enter-payload.png)

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

   Por ejemplo, puede usar una herramienta como [Postman](https://www.getpostman.com/) para enviar HTTP POST. Si ha [habilitado Azure Active Directory Open Authentication](../logic-apps/logic-apps-securing-a-logic-app.md#enable-oauth) (Azure AD OAuth) para autorizar llamadas entrantes al desencadenador de solicitud, llame al desencadenador mediante una [dirección URL de Firma de acceso compartido (SAS)](../logic-apps/logic-apps-securing-a-logic-app.md#sas) o mediante un token de autenticación, pero no es posible usar ambos. El token de autenticación debe especificar el tipo `Bearer` en el encabezado de autorización. Para obtener más información, vea [Proteger el acceso y los datos en Azure Logic Apps - Acceso a desencadenadores de solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-triggers).

Para obtener más información sobre la definición JSON subyacente del desencadenador y sobre cómo llamar a este desencadenador, vea estos temas: [Tipo de desencadenador de solicitud](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) y [Llamada, desencadenamiento o anidamiento de aplicaciones lógicas con puntos de conexión HTTP en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

### <a name="trigger-outputs"></a>Salidas del desencadenador

Aquí encontrará más información sobre las salidas del desencadenador de solicitud:

| Nombre de la propiedad JSON | Tipo de datos | Descripción |
|--------------------|-----------|-------------|
| `headers` | Object | Objeto JSON que describe los encabezados de la solicitud |
| `body` | Object | Objeto JSON que describe el contenido del cuerpo de la solicitud |
||||

<a name="add-response"></a>

## <a name="add-a-response-action"></a>Adición de una acción de respuesta

Puede usar la acción de respuesta para responder con una carga (datos) a una solicitud HTTP entrante, pero solo en una aplicación lógica que se desencadene mediante una solicitud HTTP. Puede agregar la acción de respuesta en cualquier punto del flujo de trabajo. Para obtener más información sobre la definición JSON subyacente de este desencadenador, consulte la sección [Tipo de acción de respuesta](../logic-apps/logic-apps-workflow-actions-triggers.md#response-action).

La aplicación lógica solo mantiene abierta la solicitud entrante durante un [tiempo limitado](../logic-apps/logic-apps-limits-and-config.md#request-limits). Suponiendo que el flujo de trabajo de la aplicación lógica incluye una acción de respuesta, si la aplicación lógica no devuelve ninguna respuesta después de que transcurra este tiempo, la aplicación lógica devuelve `504 GATEWAY TIMEOUT` al autor de llamada. De lo contrario, si la aplicación lógica no incluye ninguna acción de respuesta, dicha aplicación lógica devuelve inmediatamente una respuesta `202 ACCEPTED` al autor de la llamada.

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

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba "respuesta" como filtro y seleccione la acción **Respuesta**.

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

* [Conectores de Logic Apps](../connectors/apis-list.md)

