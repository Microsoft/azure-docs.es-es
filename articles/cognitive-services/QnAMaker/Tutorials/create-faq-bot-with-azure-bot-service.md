---
title: 'Tutorial: Creación de un bot de preguntas frecuentes con Azure Bot Service'
description: En este tutorial se crea un bot de preguntas frecuentes sin código con QnA Maker y Azure Bot Service.
ms.topic: tutorial
ms.date: 08/31/2020
ms.openlocfilehash: 0ec3b381c4361234b29717164b6b718aff0ae294
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89652440"
---
# <a name="tutorial-create-an-faq-bot-with-azure-bot-service"></a>Tutorial: Creación de un bot de preguntas frecuentes con Azure Bot Service
Cree un bot de preguntas frecuentes con QnA Maker y Azure [Bot Service](https://azure.microsoft.com/en-us/services/bot-service/) sin código.

En este tutorial, aprenderá a:

<!-- green checkmark -->
> [!div class="checklist"]
> * Vincular una base de conocimiento de QnA Maker a una instancia de Azure Bot Service
> * Implementar el bot
> * Probar el bot en Chat en web
> * Destacar el bot en los canales admitidos

## <a name="create-and-publish-a-knowledge-base"></a>Crear y publicar una base de conocimiento

Siga el [inicio rápido](../Quickstarts/create-publish-knowledge-base.md) para crear una base de conocimiento. Una vez que la base de conocimiento se haya publicado correctamente, llegará a la página siguiente.

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
        * Cambie las siguientes opciones en Azure Portal al crear el bot. Se rellenan automáticamente para la base de conocimiento existente:
           * Clave de autenticación de QnA
           * Plan de App Service y ubicación


1. Una vez creado el bot, abra el recurso **Servicio de bots**.
1. En **Administración del bot**, seleccione **Probar en Chat en web**.
1. En el aviso de chat **Escriba su mensaje**, escriba:

    `Azure services?`

    El bot de chat muestra una respuesta de la base de conocimiento.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-web-chat.png" alt-text="Escriba una consulta de usuario en el chat en web de prueba.":::
1. Destacar el bot en los [canales admitidos](https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0&preserve-view=true) adicionales.
