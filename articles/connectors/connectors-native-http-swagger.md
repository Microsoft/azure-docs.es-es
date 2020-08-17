---
title: Conexión con puntos de conexión REST desde Azure Logic Apps
description: Supervisión de los puntos de conexión REST en tareas, procesos y flujos de trabajo automatizados mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 11/01/2019
tags: connectors
ms.openlocfilehash: cf32938b534272a13af5891d6a31e64b8136a528
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281470"
---
# <a name="call-rest-endpoints-by-using-azure-logic-apps"></a>Llamada a puntos de conexión REST mediante Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el conector integrado de HTTP + Swagger, puede automatizar los flujos de trabajo que llaman periódicamente a cualquier punto de conexión REST a través de un [archivo Swagger](https://swagger.io) al compilar aplicaciones lógicas. El desencadenador y la acción HTTP + Swagger funcionan del mismo modo que el [desencadenador y la acción HTTP](connectors-native-http.md), pero ofrecen una mejor experiencia en el Diseñador de aplicación lógica al exponer la estructura de la API y los resultados que se describen en el archivo de Swagger. Para implementar un desencadenador de sondeo, siga el modelo de sondeo que se describe en [Creación de API personalizadas para llamar a otras API, servicios y sistemas desde aplicaciones lógicas](../logic-apps/logic-apps-create-api-app.md#polling-triggers).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La dirección URL del archivo de Swagger (no OpenAPI) que describe el punto de conexión REST de destino.

  Normalmente, el punto de conexión REST debe cumplir estos criterios para que el conector funcione:

  * El archivo de Swagger debe hospedarse en una URL HTTPS a la que se pueda acceder públicamente.

  * El archivo de Swagger debe tener la opción [Uso compartido de recursos entre orígenes (CORS)](/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) habilitada.

  Para hacer referencia a un archivo de Swagger que no está hospedado o que no cumple los requisitos de seguridad y entre orígenes, puede [cargar el archivo de Swagger en un contenedor de blobs de una cuenta de almacenamiento de Azure](#host-swagger) y habilitar CORS en esa cuenta de almacenamiento para poder hacer referencia al archivo.

  En los ejemplos de este tema se usa [Cognitive Services Face API](../cognitive-services/face/overview.md), que requiere una [clave de acceso y cuenta de Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

* La aplicación lógica desde la que quiere llamar al punto de conexión de destino. Para comenzar con el desencadenador HTTP + Swagger, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar la acción de HTTP + Swagger, inicie la aplicación lógica con el desencadenador que quiera. En este ejemplo se usa el desencadenador HTTP + Swagger como primer paso.

## <a name="add-an-http--swagger-trigger"></a>Agregar un desencadenador HTTP + Swagger

Este desencadenador integrado envía una solicitud HTTP a una URL de un archivo de Swagger que describe una API REST y devuelve una respuesta que incluye el contenido de ese archivo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en blanco en el diseñador de aplicación lógica.

1. En el diseñador, escriba "swagger" como filtro en el cuadro de búsqueda. En la lista **Desencadenadores**, seleccione el desencadenador **HTTP + Swagger**.

   ![Seleccionar el desencadenador HTTP + Swagger](./media/connectors-native-http-swagger/select-http-swagger-trigger.png)

1. En el cuadro **DIRECCIÓN URL DEL PUNTO DE CONEXIÓN DE SWAGGER**, escriba la URL del archivo de Swagger y seleccione **Siguiente**.

   En este ejemplo se usa la URL de Swagger que se encuentra en la región Oeste de EE. UU. de [Cognitive Services Face API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236):

   `https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/export?DocumentFormat=Swagger&ApiName=Face%20API%20-%20V1.0`

   ![Escribir la URL del punto de conexión de Swagger](./media/connectors-native-http-swagger/http-swagger-trigger-parameters.png)

1. Cuando el diseñador muestra las operaciones que describe el archivo de Swagger, seleccione la operación que quiera usar.

   ![Operaciones en el archivo de Swagger](./media/connectors-native-http-swagger/http-swagger-trigger-operations.png)

1. Proporcione los valores de los parámetros de desencadenador, que varían en función de la operación que haya seleccionado, que quiere incluir en la llamada de punto de conexión. Configure la periodicidad con la que quiere que el desencadenador llame al punto de conexión.

   En este ejemplo se cambia el nombre del desencadenador por "HTTP + Swagger trigger: Face - Detect" (Desencadenador HTTP + Swagger: detección de cara) para que el paso tenga un nombre más descriptivo.

   ![Detalles de la operación](./media/connectors-native-http-swagger/http-swagger-trigger-operation-details.png)

1. Para agregar otros parámetros disponibles, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera.

   Para obtener más información sobre los tipos de autenticación disponibles para HTTP + Swagger, consulte [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Continúe creando el flujo de trabajo de la aplicación lógica con acciones que se ejecuten cuando se activa el desencadenador.

1. Cuando haya finalizado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

## <a name="add-an-http--swagger-action"></a>Agregar una acción HTTP + Swagger

Esta acción integrada crea una solicitud HTTP a la URL del archivo de Swagger que describe una API REST y devuelve una respuesta que incluye el contenido de ese archivo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en el Diseñador de aplicaciones lógicas.

1. En el paso en el que quiera agregar la acción HTTP + Swagger, seleccione **Nuevo paso**.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En el diseñador, escriba "swagger" como filtro en el cuadro de búsqueda. En la lista **Acciones**, seleccione la acción **HTTP + Swagger**.

    ![Acción de selección de HTTP + Swagger](./media/connectors-native-http-swagger/select-http-swagger-action.png)

1. En el cuadro **DIRECCIÓN URL DEL PUNTO DE CONEXIÓN DE SWAGGER**, escriba la URL del archivo de Swagger y seleccione **Siguiente**.

   En este ejemplo se usa la URL de Swagger que se encuentra en la región Oeste de EE. UU. de [Cognitive Services Face API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236):

   `https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/export?DocumentFormat=Swagger&ApiName=Face%20API%20-%20V1.0`

   ![Escribir la URL del punto de conexión de Swagger](./media/connectors-native-http-swagger/http-swagger-action-parameters.png)

1. Cuando el diseñador muestra las operaciones que describe el archivo de Swagger, seleccione la operación que quiera usar.

   ![Operaciones en el archivo de Swagger](./media/connectors-native-http-swagger/http-swagger-action-operations.png)

1. Proporcione los valores de los parámetros de acción, que varían en función de la operación que haya seleccionado, que quiere incluir en la llamada de punto de conexión.

   En este ejemplo no hay parámetros, pero se cambia el nombre de la acción por "HTTP + Swagger action: Face - Identify" (Acción HTTP + Swagger: identificación de cara) para que el paso tenga un nombre más descriptivo.

   ![Detalles de la operación](./media/connectors-native-http-swagger/http-swagger-action-operation-details.png)

1. Para agregar otros parámetros disponibles, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera.

   Para obtener más información sobre los tipos de autenticación disponibles para HTTP + Swagger, consulte [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Cuando haya finalizado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

<a name="host-swagger"></a>

## <a name="host-swagger-in-azure-storage"></a>Hospedar Swagger en Azure Storage

Puede hacer referencia a un archivo de Swagger que no está hospedado o que no cumple los requisitos de seguridad y entre orígenes; para ello, cargue el archivo en un contenedor de blobs de una cuenta de almacenamiento de Azure y habilite CORS en esa cuenta de almacenamiento. Para crear, configurar y almacenar archivos de Swagger en Azure Storage, siga estos pasos:

1. [Creación de una cuenta de almacenamiento de Azure](../storage/common/storage-account-create.md).

1. Habilite CORS para el blob. En el menú de la cuenta de almacenamiento, seleccione **CORS**. En la pestaña **Blob service**, especifique estos valores y después seleccione **Guardar**.

   | Propiedad | Value |
   |----------|-------|
   | **Orígenes permitidos** | `*` |
   | **Métodos permitidos** | `GET`, `HEAD`, `PUT` |
   | **Encabezados permitidos** | `*` |
   | **Encabezados expuestos** | `*` |
   | **Antigüedad máxima** (en segundos) | `200` |
   |||

   Aunque en este ejemplo se usa [Azure Portal](https://portal.azure.com), puede usar una herramienta como [Explorador de Azure Storage](https://storageexplorer.com/), o bien configurar automáticamente este valor mediante este [script de PowerShell](https://github.com/logicappsio/EnableCORSAzureBlob/blob/master/EnableCORSAzureBlob.ps1) de ejemplo.

1. [Cree un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md). En el panel **Introducción** del contenedor, seleccione **Cambiar nivel de acceso**. En la lista **Nivel de acceso público**, seleccione **Blob (acceso de lectura anónimo solo para blobs)** y seleccione **Aceptar**.

1. [Cargue el archivo de Swagger en el contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md#upload-a-block-blob), ya sea mediante [Azure Portal](https://portal.azure.com) o el [Explorador de Azure Storage](https://storageexplorer.com/).

1. Para hacer referencia al archivo del contenedor de blobs, obtenga la dirección URL HTTPS que sigue este formato, que distingue entre mayúsculas y minúsculas, en Explorador de Microsoft Azure Storage:

   `https://<storage-account-name>.blob.core.windows.net/<blob-container-name>/<complete-swagger-file-name>?<query-parameters>`

## <a name="connector-reference"></a>Referencia de conectores

Aquí tiene más información sobre los resultados de una acción o un desencadenador HTTP + Swagger. La llamada de HTTP + Swagger devuelve esta información:

| Nombre de propiedad | Tipo | Descripción |
|---------------|------|-------------|
| headers | object | Encabezados de la solicitud |
| body | object | Objeto JSON | Objeto con el contenido del cuerpo de la solicitud |
| status code | int | Código de estado de la solicitud |
|||

| status code | Descripción |
|-------------|-------------|
| 200 | Aceptar |
| 202 | Accepted |
| 400 | Solicitud incorrecta |
| 401 | No autorizado |
| 403 | Prohibido |
| 404 | No encontrado |
| 500 | Error interno del servidor Error desconocido. |
|||

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

