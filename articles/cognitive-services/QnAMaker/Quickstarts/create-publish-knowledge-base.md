---
title: 'Inicio rápido: Creación, entrenamiento y publicación de la base de conocimientos: QnA Maker'
description: Puede crear una base de conocimiento (KB) de QnA Maker a partir de contenido propio, como por ejemplo preguntas más frecuentes o manuales de productos. En este artículo se incluye un ejemplo de la creación de una base de conocimiento de QnA Maker a partir de una página web sencilla de preguntas más frecuentes para responder a preguntas de QnA Maker.
ms.topic: quickstart
ms.date: 05/26/2020
ms.openlocfilehash: 3cc38ca49820b1a97ec11c890bfd0ef1670f6eef
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89535856"
---
# <a name="quickstart-create-train-and-publish-your-qna-maker-knowledge-base"></a>Inicio rápido: Creación, entrenamiento y publicación de la base de conocimiento de QnA Maker

Puede crear una base de conocimiento (KB) de QnA Maker a partir de contenido propio, como por ejemplo preguntas más frecuentes o manuales de productos. En este artículo se incluye un ejemplo de la creación de una base de conocimiento de QnA Maker a partir de una página web sencilla de preguntas más frecuentes para responder a preguntas de QnA Maker.

## <a name="prerequisites"></a>Prerrequisitos

> [!div class="checklist"]
> * Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.
> * Un [recurso](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) de QnA Maker creado en Azure Portal. Recuerde el identificador de Azure Active Directory, la suscripción y el nombre de recurso de QnA que seleccionó al crear el recurso.

## <a name="create-your-first-qna-maker-knowledge-base"></a>Creación de su primera base de conocimiento de QnA Maker

1. Inicie sesión en el portal de [QnAMaker.ai](https://QnAMaker.ai) con las credenciales de Azure.

1. En el portal de QnA Maker, seleccione **Create a knowledge base** (Crear una base de conocimiento).

1. En la página **Crear**, omita el **paso 1** si ya tiene el recurso de QnA Maker.

    Si aún no ha creado el recurso, seleccione **Create a QnA service** (Crear un servicio QnA). Se le dirigirá a [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) para configurar un servicio QnA Maker en la suscripción. Recuerde el identificador de Azure Active Directory, la suscripción y el nombre de recurso de QnA que seleccionó al crear el recurso.

    Cuando haya terminado de crear el recurso en Azure Portal, vuelva al portal de QnA Maker, actualice la página del explorador y continúe con el **paso 2**.

1. En el **paso 3**, seleccione Active Directory, la suscripción, el servicio (recurso) y el idioma de todas las bases de conocimiento creadas en el servicio.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/qnaservice-selection.png" alt-text="Captura de pantalla de la selección de una base de conocimiento del servicio QnA Maker":::

1. En el **paso 3**, asigne el nombre **Mi KB de QnA de ejemplo** a la base de conocimiento.

1. En el **paso 4**, establezca la configuración con los siguientes valores:

    |Configuración|Value|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Habilitar extracción en varios turnos de direcciones URL, archivos .pdf o .docx).|Activado|
    |**Texto de respuesta predeterminado**| `Quickstart - default answer not found.`|
    |**+ Agregar dirección URL**|`https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting`|
    |**Charla**|Seleccione **Professional**|

1. En el **paso 5**, seleccione **Create your KB** (Crear base de conocimiento).

    El proceso de extracción tarda unos minutos en leer el documento e identificar las preguntas y respuestas.

    Cuando QnA Maker crea correctamente la base de conocimiento, se abre la página **Knowledge base** (Base de conocimiento). En esta página puede modificar el contenido de la base de conocimiento.

## <a name="add-a-new-question-and-answer-set"></a>Incorporación de un nuevo conjunto de pregunta y respuesta

1. En el portal de QnA Maker, en la página **Edit** (Editar), seleccione **+ Add QnA Pair** (Agregar par de QnA).
1. Agregue la siguiente pregunta:

    `How many Azure services are used by a knowledge base?`

1. Agregue la respuesta con el formato _Markdown_:

    ` * Azure QnA Maker service\n* Azure Cognitive Search\n* Azure web app\n* Azure app plan`

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/add-question-and-answer.png" alt-text="Agregue la pregunta como texto y la respuesta con formato Markdown.":::

    El símbolo de Markdown `*` se usa para los puntos de viñeta. `\n` se usa para una nueva línea.

    En la página **Edit** (Editar) se muestra el símbolo de Markdown. Cuando use el panel **Test** (Probar) más adelante, verá que el formato Markdown se muestra correctamente.

## <a name="save-and-train"></a>Guardar y entrenar

En la esquina superior derecha, haga clic en **Save and train** (Guardar y entrenar) para guardar las modificaciones y entrenar a QnA Maker. Las modificaciones no se conservan a menos que se guarden.

## <a name="test-the-knowledge-base"></a>Prueba de la base de conocimiento

1. En la esquina superior derecha del portal de QnA Maker, haga clic en **Test** (Probar) para comprobar que los cambios realizados han surtido efecto.
1. Escriba una consulta de usuario de ejemplo en el cuadro de texto.

    `How many Azure services are used by a knowledge base?`

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-panel-in-qna-maker.png" alt-text="Escriba una consulta de usuario de ejemplo en el cuadro de texto.":::

1. Haga clic en **Inspect** (Inspeccionar) para examinar la respuesta con más detalle. La ventana de prueba se usa para probar los cambios realizados en la base de conocimiento antes de que esta se publique.

1. Vuelva a hacer clic en **Test** (Probar) para cerrar el panel **Test** (Probar).

## <a name="publish-the-knowledge-base"></a>Publicación de una base de conocimiento

Cuando se publica una base de conocimiento, su contenido se mueve desde el índice `test` a un índice `prod` en Azure Search.

![Captura de pantalla del traslado de contenido de la base de conocimiento](../media/qnamaker-how-to-publish-kb/publish-prod-test.png)

1. En el portal de QnA Maker, seleccione **Publish** (Publicar). Después, para confirmar, haga clic en **Publish** (Publicar) en la página.

    El servicio QnA Maker se publica correctamente. Puede usar el punto de conexión del código de la aplicación o el bot.

    ![Captura de pantalla de una publicación correcta](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint.png)

## <a name="create-a-bot"></a>Creación de un bot

Después de la publicación, puede crear un bot en la página **Publicar**:

* Puede crear varios bots rápidamente, todos ellos apuntando a la misma base de conocimiento para diferentes regiones o planes de precios para los bots individuales.
* Si desea solo un bot para la base de conocimiento, utilice el vínculo **View all your bots on the Azure portal** (Ver todos los bots en Azure Portal) para ver una lista de los bots actuales.

Si realiza cambios en la base de conocimiento y vuelve a publicarla, no es necesario realizar ninguna acción más con el bot. Ya está configurado para funcionar con la base de conocimiento y funciona con todos los cambios futuros de la base de conocimiento. Cada vez que publique una base de conocimiento, todos los bots conectados a ella se actualizarán automáticamente.

1. En el portal de QnA Maker, en la página **Publish** (Publicar), seleccione **Create bot** (Crear bot). Este botón solo aparece cuando se publica la base de conocimiento.

    ![Captura de pantalla de creación de un bot](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page.png)

1. Se abre una nueva pestaña del explorador para Azure Portal, con la página de creación de Azure Bot Service. Configure Azure Bot Service. El bot y QnA Maker pueden compartir el plan del servicio de aplicación web pero no se puede compartir la aplicación web. Esto significa que el **nombre de aplicación** para el bot debe ser diferente del nombre de aplicación para el servicio QnA Maker.

    * **Sí**
        * Cambie el identificador del bot si no es único.
        * Seleccione el lenguaje del SDK. Una vez creado el bot, puede descargar el código en el entorno de desarrollo local y continuar el proceso de desarrollo.
    * **No**
        * No cambie las siguientes opciones en Azure Portal al crear el bot. Se rellenan automáticamente para la base de conocimiento existente:
           * Clave de autenticación de QnA
           * Plan de App Service y ubicación


1. Una vez creado el bot, abra el recurso **Servicio de bots**.
1. En **Administración del bot**, seleccione **Probar en Chat en web**.
1. En el aviso de chat **Escriba su mensaje**, escriba:

    `Azure services?`

    El bot de chat muestra una respuesta de la base de conocimiento.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-web-chat.png" alt-text="Escriba una consulta de usuario en el chat en web de prueba.":::

## <a name="what-did-you-accomplish"></a>¿Qué ha logrado?

Ha creado una nueva base de conocimiento, ha agregado una dirección URL pública a esta, ha agregado su propio par de QnA y ha entrenado, probado y publicado la base de conocimiento.

Después de publicar la base de conocimiento, ha creado un bot y lo ha probado.

Todo esto se ha realizado en unos minutos sin tener que escribir ningún código ni limpiar el contenido.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a continuar con el siguiente inicio rápido, elimine los recursos de QnA Maker y Bot Framework en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de preguntas con metadatos](add-question-metadata-portal.md)

Para obtener más información:

* [Formato Markdown en las respuestas](../reference-markdown-format.md)
* [Orígenes de datos](../concepts/knowledge-base.md) de QnA Maker


