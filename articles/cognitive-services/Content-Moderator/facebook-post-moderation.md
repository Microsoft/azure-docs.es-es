---
title: 'Tutorial: Moderación de contenido de Facebook: Content Moderator'
titlesuffix: Azure Cognitive Services
description: En este tutorial, aprenderá a utilizar Content Moderator basado en aprendizaje automático para ayudar a moderar los comentarios y las publicaciones de Facebook.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: tutorial
ms.date: 01/18/2019
ms.author: pafarley
ms.openlocfilehash: 5d31285ca305ba7fefdf31b4a97e3183f58b3e3b
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233807"
---
# <a name="tutorial-moderate-facebook-posts-and-commands-with-azure-content-moderator"></a>Tutorial: Moderación de comandos y publicaciones de Facebook con Azure Content Moderator

En este tutorial, aprenderá a utilizar Azure Content Moderator para ayudar a moderar los comentarios y las publicaciones de una página de Facebook. Facebook enviará el contenido publicado por los visitantes al servicio Content Moderator. A continuación, los flujos de trabajo de Content Moderator publicarán el contenido o crearán revisiones dentro de la herramienta de revisión, dependiendo de los umbrales y puntuaciones del contenido. Vea el [vídeo de demostración de Build 2017](https://channel9.msdn.com/Events/Build/2017/T6033) para obtener un ejemplo de trabajo de este escenario.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Crear un equipo de Content Moderator.
> * Crear funciones de Azure que escuchen los eventos HTTP de Content Moderator y Facebook.
> * Vincule una página de Facebook a Content Moderator mediante una aplicación de Facebook.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

En este diagrama se ilustra cada componente de este escenario:

![Diagrama de Content Moderator que recibe información de Facebook a través de "FBListener" y envía información a través de "CMListener"](images/tutorial-facebook-moderation.png)

> [!IMPORTANT]
> En 2018, Facebook puso en marcha una investigación más estricta de las aplicaciones de Facebook. No podrá completar los pasos de este tutorial si la aplicación no se ha revisado y aprobado por el equipo de revisión de Facebook.

## <a name="prerequisites"></a>Requisitos previos

- Una clave de suscripción de Content Moderator. Siga las instrucciones de [Creación de una cuenta de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) para suscribirse al servicio Content Moderator y obtener su clave.
- Una [cuenta de Facebook](https://www.facebook.com/).

## <a name="create-a-review-team"></a>Creación de un equipo de revisión

Consulte la guía de inicio rápido [Cómo familiarizarse con Content Moderator](quick-start.md) para obtener instrucciones sobre cómo registrarse en la [herramienta de revisión de Content Moderator](https://contentmoderator.cognitive.microsoft.com/) y crear un equipo de revisión. Anote el valor de **Id. de equipo** en la página **Credenciales**.

## <a name="configure-image-moderation-workflow"></a>Configuración del flujo de trabajo de moderación de imágenes

Consulte la guía [Definición, prueba y uso de flujos de trabajo](review-tool-user-guide/workflows.md) para crear un flujo de trabajo de imágenes personalizadas. Esto permitirá a Content Moderator comprobar de forma automática imágenes de Facebook y enviar algunas a la herramienta de revisión. Tome nota del **nombre** del flujo de trabajo.

## <a name="configure-text-moderation-workflow"></a>Configuración del flujo de trabajo de moderación de texto

Una vez más, consulte la guía [Definición, prueba y uso de flujos de trabajo](review-tool-user-guide/workflows.md); esta vez, cree un flujo de trabajo de texto personalizado. Esto permitirá a Content Moderator comprobar automáticamente el contenido de texto. Tome nota del **nombre** del flujo de trabajo.

![Configuración del flujo de trabajo de texto](images/text-workflow-configure.PNG)

Pruebe el flujo de trabajo con el botón **Execute Workflow** (Ejecutar flujo de trabajo).

![Prueba del flujo de trabajo de texto](images/text-workflow-test.PNG)

## <a name="create-azure-functions"></a>Creación de funciones de Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/) y siga estos pasos:

1. Cree una aplicación de función de Azure, tal y como se muestra en la página [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-create-function-app-portal).
1. Vaya a la instancia de Function App recién creada.
1. Dentro de esta, vaya a la pestaña **Características de la plataforma** y seleccione **Configuración**. En la sección **Configuración de la aplicación** de la página siguiente, seleccione **Nueva configuración de la aplicación** para agregar los siguientes pares de clave/valor:
    
    | Nombre del valor de configuración de la aplicación | value   | 
    | -------------------- |-------------|
    | cm:TeamId   | Identificador del equipo de Content Moderator.  | 
    | cm:SubscriptionKey | Clave de suscripción de Content Moderator. Consulte [Credenciales](review-tool-user-guide/credentials.md). |
    | cm:Region | Nombre de la región de Content Moderator, sin espacios en blanco. |
    | cm:ImageWorkflow | Nombre del flujo de trabajo para ejecutar en imágenes. |
    | cm:TextWorkflow | Nombre del flujo de trabajo para ejecutar en texto. |
    | cm:CallbackEndpoint | Dirección URL de la aplicación de función CMListener que creará más adelante en esta guía. |
    | fb:VerificationToken | Token de secreto que cree, que se usa para suscribirse a los efectos de fuente de Facebook. |
    | fb:PageAccessToken | El token de acceso de la API gráfica de Facebook no expira y permite a la función ocultar o eliminar publicaciones en su nombre. Esto se producirá en un paso posterior. |

    Haga clic en el botón **Guardar** de la parte superior de la página.

1. Vuelva a la pestaña **Características de la plataforma**. Use el botón **+** del panel izquierdo para abrir el panel **Nueva función**. La función que va a crear recibirá eventos de Facebook.

    ![Panel Azure Functions con el botón Agregar función resaltado.](images/new-function.png)

    1. Haga clic en el icono **Desencadenador HTTP**.
    1. Escriba el nombre **FBListener**. El campo **Nivel de autorización** debe establecerse en **Función**.
    1. Haga clic en **Create**(Crear).
    1. Reemplace el contenido de **run.csx** por el contenido de **FbListener/run.csx**

    [!code-csharp[FBListener: csx file](~/samples-fbPageModeration/FbListener/run.csx?range=1-154)]

1. Cree una nueva función **Desencadenador HTTP** llamada **CMListener**. Esta función recibe eventos de Content Moderator. Reemplace el contenido de **run.csx** por el contenido de **CMListener/run.csx**

    [!code-csharp[FBListener: csx file](~/samples-fbPageModeration/CmListener/run.csx?range=1-110)]

---

## <a name="configure-the-facebook-page-and-app"></a>Configuración de la página y la aplicación de Facebook

1. Cree una aplicación de Facebook.

    ![página del desarrollador de facebook](images/facebook-developer-app.png)

    1. Vaya al [sitio para desarrolladores de Facebook](https://developers.facebook.com/).
    1. Haga clic en **My Apps** (Mis aplicaciones).
    1. Agregue una nueva aplicación.
    1. asígnele un nombre
    1. Seleccione **Webhooks -> Configurar**
    1. Seleccione **Página** en el menú desplegable y seleccione **Suscribirse a este objeto**
    1. Indique la **dirección URL de FBListener** como dirección URL de devolución de llamada, y el **token de comprobación** que configuró en las **opciones de la aplicación de función**.
    1. Una vez suscrito, vaya hasta la fuente y seleccione **Subscribe** (Suscribirse).
    1. Haga clic en botón **Test** (Probar) de la fila **Feed** (Fuente) para enviar un mensaje de prueba a BListener Azure Function y luego presione el botón **Send to My Server** (Enviar a mi servidor). Debería ver la solicitud que se recibe en su instancia de FBListener.

1. Cree una página de Facebook.

    > [!IMPORTANT]
    > En 2018, Facebook puso en marcha una investigación más estricta de las aplicaciones de Facebook. No podrá ejecutar las secciones 2, 3 y 4 si la aplicación no se ha revisado y aprobado por el equipo de revisión de Facebook.

    1. Vaya a [Facebook](https://www.facebook.com/bookmarks/pages) y cree una **nueva página de Facebook**.
    1. Permita a la app de Facebook acceder a esta página; para ello, siga estos pasos:
        1. Vaya al [Explorador de Graph API](https://developers.facebook.com/tools/explorer/).
        1. Seleccione **Application** (Aplicación).
        1. Seleccione **Page Access Token** (Token de acceso a página) y envíe una solicitud **Get**.
        1. Haga clic en **Page ID** (Id. de página) en la respuesta.
        1. Ahora, anexe **/subscribed_apps** a la dirección URL y envíe una solicitud **Get** (respuesta vacía).
        1. Envíe una solicitud **Post**. Obtendrá la respuesta como **success: true**.

3. Cree un token de acceso a Graph API que no expire.

    1. Vaya al [Explorador de Graph API](https://developers.facebook.com/tools/explorer/).
    2. Seleccione la opción **Application** (Aplicación).
    3. Seleccione la opción **Get User Access Token** (Obtener token de acceso de usuario).
    4. En **Select Permissions** (Seleccionar permisos), seleccione las opciones **manage_pages** y **publish_pages**.
    5. Usaremos el **token de acceso** (token de breve duración) en el paso siguiente.

4. Usamos Postman en los pasos siguientes.

    1. Abra **Postman** (u obténgalo [aquí](https://www.getpostman.com/)).
    2. Importe estos dos archivos:
        1. [Colección de Postman](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/Facebook%20Permanant%20Page%20Access%20Token.postman_collection.json)
        2. [Entorno de Postman](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/FB%20Page%20Access%20Token%20Environment.postman_environment.json)       
    3. Actualice estas variables de entorno:
    
        | Clave | Valor   | 
        | -------------------- |-------------|
        | appId   | Inserte aquí el identificador de la aplicación de Facebook.  | 
        | appSecret | Inserte aquí el secreto de la aplicación de Facebook. | 
        | short_lived_token | Inserte el token de acceso de usuario de corta duración que generó en el paso anterior. |
    4. Ahora, ejecute las 3 API enumeradas en la colección: 
        1. Seleccione **Generate Long-Lived Access Token** (Generar token de acceso de larga duración) y haga clic en **Send** (Enviar).
        2. Seleccione **Get User ID** (Obtener id. de usuario) y haga clic en **Send** (Enviar).
        3. Seleccione **Get Permanent Page Access Token** (Obtener token de acceso de página permanente) y haga clic en **Send** (Enviar).
    5. Copie el valor de **access_token** de la respuesta y asígnelo a la opción de la aplicación, **fb:PageAccessToken**.

La solución envía todas las imágenes y el texto que se publican en la página de Facebook a Content Moderator. A continuación, se invocan los flujos de trabajo que configuró anteriormente. El contenido que no supere los criterios definidos en los flujos de trabajo se pasará a las revisiones dentro de la herramienta de revisión. El resto del contenido se publica automáticamente.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configurará un programa para analizar las imágenes de productos con el fin de etiquetarlos por tipo de producto para que un equipo de revisión pueda tomar decisiones informadas sobre la moderación de contenido. Después, obtendrá más información acerca de los detalles de la moderación de imágenes.

> [!div class="nextstepaction"]
> [Moderación de imágenes](./image-moderation-api.md)
