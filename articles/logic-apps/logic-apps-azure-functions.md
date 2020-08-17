---
title: Incorporación y llamada a Azure Functions desde Azure Logic Apps
description: Llamada y ejecución de código personalizado en su instancia de Azure Functions desde tareas y flujos de trabajo automatizados en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/01/2019
ms.custom: devx-track-javascript
ms.openlocfilehash: e853255f0fc1bc95a4a7fb1658fc41d4fe705c41
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420063"
---
# <a name="call-azure-functions-from-azure-logic-apps"></a>Llamada a funciones de Azure desde Azure Logic Apps

Cuando quiera ejecutar únicamente código que realice un trabajo específico en sus aplicaciones lógicas, puede crear sus propias funciones con [Azure Functions](../azure-functions/functions-overview.md). Este servicio le ayuda a crear funciones de Node.js, C# y F#, por lo que no tiene que crear una aplicación completa o la infraestructura para ejecutar el código. También puede [llamar a aplicaciones lógicas desde Azure Functions](#call-logic-app). Azure Functions proporciona informática sin servidor en la nube y es útil para realizar tareas, como las siguientes:

* Extender el comportamiento de la aplicación lógica con funciones en Node.js o C#.
* Realizar cálculos en el flujo de trabajo de las aplicaciones lógicas.
* Aplicar formato avanzado o campos de proceso a las aplicaciones lógicas.

Para ejecutar fragmentos de código sin necesidad de crear las funciones de Azure, obtenga información sobre cómo [agregar y ejecutar código en línea](../logic-apps/logic-apps-add-run-inline-code.md).

> [!NOTE]
> La integración entre Logic Apps y Azure Functions actualmente no funciona con los espacios habilitados.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una aplicación de función de Azure, que es un contenedor para las funciones de Azure, junto con su función de Azure. Si no tiene una aplicación de función, [cree primero la aplicación de función](../azure-functions/functions-create-first-azure-function.md). Puede crear una función ya sea fuera de su aplicación lógica en Azure Portal o bien [de manera independiente de la aplicación lógica](#create-function-designer) en el Diseñador de aplicación lógica.

* Cuando se trabaja con aplicaciones lógicas, se aplican los mismos requisitos a las funciones y a las aplicaciones de función, ya sean nuevas o existentes:

  * La aplicación de función y la aplicación lógica deben tener la misma suscripción de Azure.

  * Las nuevas aplicaciones de función deben usar .NET o JavaScript, como la pila en tiempo de ejecución. Cuando se agrega una nueva función a las aplicaciones de función existentes, puede seleccionar C# o JavaScript.

  * La función usa la plantilla **desencadenador HTTP**.

    Esta plantilla puede aceptar contenido que tenga el tipo `application/json` de la aplicación lógica. Cuando se agrega una función de Azure a la aplicación lógica, el Diseñador de aplicación lógica muestra las funciones personalizadas creadas a partir de esta plantilla dentro de la suscripción de Azure.

  * La función no usa rutas personalizadas a menos que haya especificado una [definición de OpenAPI](../azure-functions/functions-openapi-definition.md) (lo que antes se conocía como [archivo Swagger](https://swagger.io/)).

  * Si ha especificado una definición de OpenAPI para la función, el Diseñador de aplicación lógica le ofrece una experiencia más completa al trabajar con parámetros de función. Para que la aplicación lógica pueda buscar funciones con definiciones de OpenAPI y acceder a estas, [siga estos pasos para configurar la aplicación de función](#function-swagger).

* La aplicación lógica en la que quiere agregar la función, incluido un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) como primer paso de la aplicación lógica

  Para poder agregar acciones que ejecuten funciones, la aplicación lógica se debe iniciar con un desencadenador. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="function-swagger"></a>

## <a name="find-functions-that-have-openapi-descriptions"></a>Búsqueda de funciones que tengan descripciones de OpenAPI

Para obtener una experiencia más rica al trabajar con parámetros de función en el Diseñador de aplicación lógica, [genere una definición de OpenAPI](../azure-functions/functions-openapi-definition.md), que antes se conocía como [archivo Swagger](https://swagger.io/), para su función. Para configurar la aplicación de función de manera que la aplicación lógica pueda buscar y usar funciones con descripciones de Swagger, siga estos pasos:

1. Asegúrese de que la aplicación de función se está ejecutando.

1. En la aplicación de función, siga estos pasos para configurar el [uso compartido de recursos entre orígenes (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) de manera que se permitan todos los orígenes:

   1. En la lista **Aplicaciones de función**, seleccione la que corresponda. En el panel derecho, seleccione **Características de la plataforma** > **CORS**.

      ![Seleccione la aplicación de función > "Características de la plataforma" > "CORS"](./media/logic-apps-azure-functions/function-platform-features-cors.png)

   1. En **CORS**, agregue el carácter comodín asterisco ( **`*`** ), quite los demás orígenes de la lista de la lista y seleccione **Guardar**.

      ![Establecimiento de "CORS * en el carácter comodín "*"](./media/logic-apps-azure-functions/function-platform-features-cors-origins.png)

## <a name="access-property-values-inside-http-requests"></a>Acceso a los valor de propiedad de las solicitudes HTTP

Las funciones de Webhook pueden aceptar solicitudes HTTP como entradas y pasarlas a otras funciones. Por ejemplo, aunque Logic Apps tiene [funciones que convierten los valores de fecha y hora](../logic-apps/workflow-definition-language-functions-reference.md), este ejemplo básico de función de JavaScript muestra cómo acceder a una propiedad de un objeto de solicitud que se ha pasado a la función y realizar operaciones en ese valor de propiedad. Para acceder a las propiedades dentro de los objetos, en este ejemplo se usa el [operador punto (.)](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Property_accessors):

```javascript
function convertToDateString(request, response){
   var data = request.body;
   response = {
      body: data.date.ToDateString();
   }
}
```

Esto es lo que ocurre dentro de esta función:

1. La función crea una variable `data` y asigna el objeto `body` dentro del objeto `request` a esa variable. La función utiliza el operador punto (.) para hacer referencia al objeto `body` dentro del objeto `request`:

   ```javascript
   var data = request.body;
   ```

1. La función ahora puede acceder a la propiedad `date` a través de la variable `data` y convertir ese valor de propiedad del tipo DateTime al tipo DateString mediante una llamada a la función `ToDateString()`. La función también devuelve el resultado a través de la propiedad `body` en la respuesta de la función:

   ```javascript
   body: data.date.ToDateString();
   ```

Ahora que ha creado la función de Azure, siga los pasos para saber cómo [agregar funciones a las aplicaciones lógicas](#add-function-logic-app).

<a name="create-function-designer"></a>

## <a name="create-functions-inside-logic-apps"></a>Creación de funciones en aplicaciones lógicas

Puede crear funciones de Azure directamente desde el flujo de trabajo de la aplicación lógica mediante la acción integrada de Azure Functions en el diseñador de aplicaciones lógicas, pero solo puede usar este método para funciones de Azure escritas en JavaScript. En el caso de otros idiomas, puede crear funciones de Azure mediante la experiencia de Azure Functions en Azure Portal. Para más información, consulte [Creación de su primera función en Azure Portal](../azure-functions/functions-create-first-azure-function.md).

Sin embargo, antes de poder crear cualquier función de Azure, debe tener una aplicación de funciones de Azure que actúe como contenedor de las funciones. Si no tiene una aplicación de función, cree primero la aplicación de función. Consulte [Creación de su primera función en Azure Portal](../azure-functions/functions-create-first-azure-function.md).

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. Para crear y agregar la función, siga los pasos que se apliquen a su escenario:

   * En el último paso del flujo de trabajo de la aplicación lógica, seleccione **Nuevo paso**.

   * Entre los pasos existentes del flujo de trabajo de la aplicación lógica, mueva el mouse sobre la flecha, seleccione el signo más (+) y luego seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba "azure functions" como filtro. En la lista acciones, seleccione la acción **Elegir una función de Azure**, por ejemplo:

   ![Búsqueda de "Azure Functions"](./media/logic-apps-azure-functions/find-azure-functions-action.png)

1. En la lista de aplicaciones de función, seleccione la que corresponda. Cuando se abra la lista de acciones, seleccione esta acción: **Crear nueva función**

   ![Selección de la aplicación de función](./media/logic-apps-azure-functions/select-function-app-create-function.png)

1. En el editor de la definición de función, defínala:

   1. En el cuadro **Nombre de la función**, proporcione un nombre para la función.

   1. En el cuadro **Código**, agregue el código a la plantilla, incluida la respuesta y la carga que desea que se devuelva a la aplicación lógica cuando la función termine de ejecutarse. Seleccione **Crear** cuando haya terminado.

   Por ejemplo:

   ![Definición de la función](./media/logic-apps-azure-functions/add-code-function-definition.png)

   En el código de la plantilla, el objeto *`context`* hace referencia al mensaje que envía la aplicación lógica mediante el campo **Cuerpo de la solicitud** en un paso posterior. Para acceder a las propiedades del objeto `context` desde dentro de la función, use esta sintaxis:

   `context.body.<property-name>`

   Por ejemplo, para hacer referencia a la propiedad `content` dentro del objeto `context`, use esta sintaxis:

   `context.body.content`

   El código de plantilla también incluye una variable `input`, que almacena el valor del parámetro `data` de modo que la función puede realizar operaciones sobre ese valor. Dentro de las funciones de JavaScript, la variable `data` es también un acceso directo de `context.body`.

   > [!NOTE]
   > Aquí, la propiedad `body` se aplica al objeto `context` y no es la misma que el token **Body** de la salida de una acción, que también se podría pasar a la función.

1. En el cuadro **Cuerpo de la solicitud**, proporcione la entrada de su función, que debe tener el formato de un objeto de notación de objetos JavaScript (JSON).

   Esta entrada es el *objeto de contexto* o el mensaje que envía la aplicación lógica a la función. Al hacer clic en el campo **Cuerpo de la solicitud**, aparece la lista de contenido dinámico, por lo que puede seleccionar tokens para las salidas de los pasos anteriores. En este ejemplo se especifica que la carga de contexto contiene una propiedad denominada `content` que tiene el valor del token **From** del desencadenador de correo electrónico.

   ![Ejemplo de "cuerpo de la solicitud": carga del objeto de contexto](./media/logic-apps-azure-functions/function-request-body-example.png)

   Aquí, el objeto de contexto no se convierte en cadena, por lo que el contenido del objeto se agrega directamente a la carga de JSON. Sin embargo, cuando el objeto de contexto no sea un token JSON que pasa una cadena, un objeto JSON o una matriz JSON, recibirá un error. Por tanto, si en este ejemplo se usa en su lugar el token **Received Time**, puede convertir el objeto de token en una cadena agregando comillas dobles.

   ![Conversión del objeto a cadena](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

1. Para especificar otros detalles, como el método de uso, los encabezados de solicitud, los parámetros de consulta o la autenticación, abra la lista **Agregar nuevo parámetro** y seleccione las opciones que desee. Para la autenticación, las opciones varían según la función seleccionada. Consulte [Habilitar la autenticación de Azure Functions](#enable-authentication-functions).

<a name="add-function-logic-app"></a>

## <a name="add-existing-functions-to-logic-apps"></a>Incorporación de funciones existentes a aplicaciones lógicas

Para llamar a funciones de Azure existentes desde las aplicaciones lógicas, puede agregar funciones de Azure como cualquier otra acción en el Diseñador de aplicación lógica.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el paso en el que quiera agregar la función, seleccione **Nuevo paso**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba "azure functions" como filtro. En la lista acciones, seleccione la acción **Elegir una función de Azure**.

   ![Búsqueda de "Azure Functions"](./media/logic-apps-azure-functions/find-azure-functions-action.png)

1. En la lista de aplicaciones de función, seleccione la que corresponda. Cuando aparezca la lista de funciones, seleccione la que corresponda.

   ![Selección de la aplicación de función y la función de Azure](./media/logic-apps-azure-functions/select-function-app-existing-function.png)

   Para las funciones con definiciones de API (descripciones de Swagger) y que [se configuran de manera que la aplicación lógica pueda buscarlas y acceder a ellas](#function-swagger), puede seleccionar **Acciones de Swagger**.

   ![Selección de la aplicación de función, "Acciones de Swagger" y la función de Azure](./media/logic-apps-azure-functions/select-function-app-existing-function-swagger.png)

1. En el cuadro **Cuerpo de la solicitud**, proporcione la entrada de su función, que debe tener el formato de un objeto de notación de objetos JavaScript (JSON).

   Esta entrada es el *objeto de contexto* o el mensaje que envía la aplicación lógica a la función. Al hacer clic en el campo **Cuerpo de la solicitud**, aparece la lista de contenido dinámico, por lo que puede seleccionar tokens para las salidas de los pasos anteriores. En este ejemplo se especifica que la carga de contexto contiene una propiedad denominada `content` que tiene el valor del token **From** del desencadenador de correo electrónico.

   ![Ejemplo de "cuerpo de la solicitud": carga del objeto de contexto](./media/logic-apps-azure-functions/function-request-body-example.png)

   Aquí, el objeto de contexto no se convierte en cadena, por lo que el contenido del objeto se agrega directamente a la carga de JSON. Sin embargo, cuando el objeto de contexto no sea un token JSON que pasa una cadena, un objeto JSON o una matriz JSON, recibirá un error. Por tanto, si en este ejemplo se usa en su lugar el token **Received Time**, puede convertir el objeto de token en una cadena agregando comillas dobles:

   ![Conversión del objeto a cadena](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

1. Para especificar otros detalles, como el método de uso, los encabezados de solicitud, los parámetros de consulta o la autenticación, abra la lista **Agregar nuevo parámetro** y seleccione las opciones que desee. Para la autenticación, las opciones varían según la función seleccionada. Consulte [Habilitar la autenticación de Azure Functions](#enable-authentication-functions).

<a name="call-logic-app"></a>

## <a name="call-logic-apps-from-azure-functions"></a>Llamada a aplicaciones lógicas desde Azure Functions

Cuando quiera desencadenar una aplicación lógica desde dentro de una función de Azure, la aplicación lógica debe comenzar con un desencadenador que proporcione un punto de conexión al que se pueda llamar. Por ejemplo, puede iniciar la aplicación lógica con el desencadenador **HTTP**, **Request**, **Azure Queues** o **Event Grid**. Desde la función, envíe una solicitud HTTP POST a la dirección URL del desencadenador e incluya la carga que quiere que procese esa aplicación lógica. Para obtener más información, consulte [Llamada, desencadenamiento o anidación de aplicaciones lógicas](../logic-apps/logic-apps-http-endpoint.md).

<a name="enable-authentication-functions"></a>

## <a name="enable-authentication-for-azure-functions"></a>Habilitar la autenticación de Azure Functions

Para autenticar fácilmente el acceso a otros recursos que están protegidos por Azure Active Directory (Azure AD) sin tener que iniciar sesión ni proporcionar credenciales o secretos, la aplicación lógica puede usar una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) (anteriormente conocida como Managed Service Identity o MSI). Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que proporcionar secretos o cambiarlos. Obtenga más información sobre [Servicios de Azure que admiten las identidades administradas para la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

Si configura la aplicación lógica para que use la identidad administrada asignada por el sistema o una identidad asignada por el usuario creada de forma manual, las funciones de Azure de la aplicación lógica también pueden usar la misma identidad para la autenticación. Para más información sobre la compatibilidad de la autenticación con Azure Functions en aplicaciones lógicas, consulte [Agregar autenticación a llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

Para configurar y usar la identidad administrada con la función, siga estos pasos:

1. Habilite la identidad administrada en la aplicación lógica y configure el acceso de dicha identidad al recurso de destino. Consulte [Autenticar el acceso a los recursos de Azure mediante identidades administradas en Azure Logic Apps](../logic-apps/create-managed-service-identity.md).

1. Siga estos pasos para habilitar la autenticación en Azure Functions y la aplicación de función:

   * [Configurar la autenticación anónima en la función](#set-authentication-function-app)
   * [Configurar la autenticación de Azure AD en la aplicación de función](#set-azure-ad-authentication)

<a name="set-authentication-function-app"></a>

### <a name="set-up-anonymous-authentication-in-your-function"></a>Configurar la autenticación anónima en la función

Para usar la identidad administrada de la aplicación lógica en la función de Azure, ha establecido el nivel de autenticación de la función en anónimo. De lo contrario, la aplicación lógica produce un error "BadRequest".

1. En [Azure Portal](https://portal.azure.com), busque y seleccione la aplicación lógica. En estos pasos se usa "FabrikamFunctionApp" como la aplicación de función de ejemplo.

1. En el panel de la aplicación de función, seleccione **Características de la plataforma**. En **Herramientas de desarrollo**, seleccione **Herramientas avanzadas (Kudu)** .

   ![Abrir las herramientas avanzadas para Kudu](./media/logic-apps-azure-functions/open-advanced-tools-kudu.png)

1. En la barra de título del sitio web de Kudu, en el menú **Debug Console** (Consola de depuración), seleccione **CMD**.

   ![En el menú de la consola de depuración, seleccione la opción "CMD".](./media/logic-apps-azure-functions/open-debug-console-kudu.png)

1. Después de que aparezca la siguiente página, en la lista de carpetas, seleccione **site** > **wwwroot** > *su-función*. En estos pasos se usa "FabrikamAzureFunction" como la función de ejemplo.

   ![Seleccione"site" > "wwwroot" > su función](./media/logic-apps-azure-functions/select-site-wwwroot-function-folder.png)

1. Abra el archivo `function.json` para edición.

   ![Haga clic en Editar para el archivo "function.json"](./media/logic-apps-azure-functions/edit-function-json-file.png)

1. En el objeto `bindings`, compruebe si existe la propiedad `authLevel`. Si la propiedad existe, establezca el valor de la propiedad en `anonymous`. De lo contrario, agregue esa propiedad y establezca el valor.

   ![Agregue la propiedad "authLevel" y establézcala en "Anonymous".](./media/logic-apps-azure-functions/set-authentication-level-function-app.png)

1. Cuando haya terminado, guarde la configuración y continúe con la siguiente sección.

<a name="set-azure-ad-authentication"></a>

### <a name="set-up-azure-ad-authentication-for-your-function-app"></a>Configurar la autenticación de Azure AD en la aplicación de función

Antes de iniciar esta tarea, busque y coloque estos valores para su uso posterior:

* El id. de objeto que se genera para la identidad asignada por el sistema que representa a la aplicación lógica.

  * Para generar este id. de objeto, [habilite la identidad asignada por el sistema de la aplicación lógica](../logic-apps/create-managed-service-identity.md#azure-portal-system-logic-app).

  * De lo contrario, para buscar este id. de objeto, abra la aplicación lógica en el Diseñador de aplicación lógica. En el menú de la aplicación lógica, en **Configuración**, seleccione **Identidad** > **Asignado por el sistema**.

* El id. de directorio del inquilino en Azure Active Directory (Azure AD)

  Para obtener el id. de directorio del inquilino, puede ejecutar el comando [`Get-AzureAccount`](/powershell/module/servicemanagement/azure.service/get-azureaccount) de PowerShell. O bien, en Azure Portal, siga estos pasos:

  1. En [Azure Portal](https://portal.azure.com), busque y seleccione la aplicación lógica.

  1. Busque y seleccione el inquilino de Azure AD. En estos pasos se usa "fabrikam" como inquilino de ejemplo.

  1. En el menú del inquilino, en **Administrar**, seleccione **Propiedades**.

  1. Copie el id. de directorio del inquilino, por ejemplo, y guárdelo para su uso posterior.

     ![Buscar y copiar el id. de directorio del inquilino de Azure AD](./media/logic-apps-azure-functions/azure-active-directory-tenant-id.png)

* El id. de recurso para el recurso de destino al que quiere acceder

  * Para buscar estos id. de recurso, consulte [Servicios de Azure que admiten Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

  > [!IMPORTANT]
  > Este id. de recurso debe coincidir exactamente con el valor que Azure AD espera, incluidas las barras diagonales finales necesarias.

  Este id. de recurso es también el mismo valor que usará más adelante en la propiedad **Audiencia** cuando [configure la acción de función que se debe usar para la identidad asignada por el sistema](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

Ahora está listo para configurar la autenticación de Azure AD para la aplicación de función.

1. En [Azure Portal](https://portal.azure.com), busque y seleccione la aplicación lógica.

1. En el panel de la aplicación de función, seleccione **Características de la plataforma**. En **Redes**, seleccione **Autenticación/autorización**.

   ![Ver la configuración de autenticación y autorización](./media/logic-apps-azure-functions/view-authentication-authorization-settings.png)

1. Cambie la opción **Autenticación de App Service** a **Activado**. En la lista desplegable **Acción necesaria cuando la solicitud no está autenticada**, seleccione **Iniciar sesión con Azure Active Directory**. En **Proveedores de autenticación,** seleccione **Azure Active Directory**.

   ![Activar la autenticación con Azure AD](./media/logic-apps-azure-functions/turn-on-authentication-azure-active-directory.png)

1. En el panel **Configuración de Azure Active Directory**, siga estos pasos:

   1. Establezca la opción **Modo de administración** en **Avanzado**.

   1. En la propiedad **Identificador de cliente**, escriba el id. de objeto de la identidad asignada por el sistema de la aplicación lógica.

   1. En la propiedad **URL del emisor**, escriba la dirección URL de `https://sts.windows.net/` y anexe el id. de directorio del inquilino de Azure AD.

      `https://sts.windows.net/<Azure-AD-tenant-directory-ID>`

   1. En la propiedad **Audiencias de token permitidas**, escriba el id. de recurso para el recurso de destino al que desea acceder.

      Este id. de recurso es el mismo valor que usará más adelante en la propiedad **Audiencia** cuando [configure la acción de función que se debe usar para la identidad asignada por el sistema](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

   En este punto, su versión tiene un aspecto similar al de este ejemplo:

   ![Configuración de la autenticación de Azure Active Directory](./media/logic-apps-azure-functions/azure-active-directory-authentication-settings.png)

1. Cuando finalice, seleccione **Aceptar**.

1. Vuelva al Diseñador de aplicaciones lógicas y siga los [pasos para autenticar el acceso con la identidad administrada](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre los [conectores de Logic Apps](../connectors/apis-list.md)
