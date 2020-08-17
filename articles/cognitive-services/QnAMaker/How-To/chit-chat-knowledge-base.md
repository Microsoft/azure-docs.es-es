---
title: Adición de charla a una base de conocimiento de QnA Maker
titleSuffix: Azure Cognitive Services
description: La incorporación de charlas personales al bot lo hace más atractivo y conversacional al crear la base de conocimiento. QnA Maker permite agregar fácilmente un conjunto previamente rellenado de las principales charlas a la base de conocimiento.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: seodec18
ms.openlocfilehash: 0aca072a9e659d242dec9dd45a12f9568d419599
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87124432"
---
# <a name="add-chit-chat-to-a-knowledge-base"></a>Adición de una charla a la base de conocimiento

La incorporación de charlas al bot lo hace más atractivo y conversacional. La característica de charla de QnA Maker permite agregar fácilmente un conjunto previamente rellenado de las principales charlas a la base de conocimiento (KB). Esto puede ser un punto de partida para la personalidad del bot y permite ahorrar el tiempo y el costo que supone escribirlas desde cero.

Este conjunto de datos tiene aproximadamente 100 escenarios de charla en la voz de varias personas, como profesional, amigable e ingenioso. Elija el rol que más se parezca a la voz del bot. Dada una consulta de usuario, QnA Maker intenta hacerla coincidir con la QnA de charla conocida más parecida.

A continuación se muestran algunos ejemplos de las distintas personalidades: Puede ver todos los [conjuntos de datos](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets) de personalidades junto con los detalles de las personalidades.

Para la consulta del usuario `When is your birthday?`, cada personalidad tiene una respuesta con estilo:

<!-- added quotes so acrolinx doesn't score these sentences -->
|Personalidad|Ejemplo|
|--|--|
|Profesional|La edad no va conmigo.|
|Descriptiva|Realmente no tengo una edad.|
|Ingenioso|No tengo caducidad.|
|Cariñoso|No tengo edad.|
|Entusiasta|Soy un bot, así que no tengo edad.|
||


## <a name="language-support"></a>Compatibilidad con idiomas

Los conjuntos de datos de charlas se admiten en los siguientes idiomas:

|Idioma|
|--|
|Chino|
|Inglés|
|Francés|
|Alemania|
|Italiano|
|Japonés|
|Coreano|
|Portugués|
|Español|


## <a name="add-chit-chat-during-kb-creation"></a>Adición de charlas durante la creación de la KB
Durante la creación de la base de conocimiento, después de agregar las direcciones URL y archivos de origen, hay una opción para agregar charlas. Elija la personalidad que quiera como base de la charla. Si no quiere agregar charlas o si ya tiene compatibilidad con la charla en los orígenes de datos, seleccione **None** (Ninguno).

## <a name="add-chit-chat-to-an-existing-kb"></a>Adición de charlas a una base de conocimiento existente
Seleccione la base de conocimiento y vaya hasta la página **Settings** (Configuración). Hay un vínculo a todos los conjuntos de datos de charla en el formato **.tsv** correspondiente. Descargue la personalidad que quiera y, después, cárguela como un origen de archivo. Asegúrese de no modificar el formato o los metadatos al descargar y cargar el archivo.

![Adición de charlas a una base de conocimiento existente](../media/qnamaker-how-to-chit-chat/add-chit-chat-dataset.png)

## <a name="edit-your-chit-chat-questions-and-answers"></a>Edición de las preguntas y respuestas de la charla
Cuando edite la base de conocimiento, verá un nuevo origen para la charla, según la personalidad que haya seleccionado. Ahora puede agregar preguntas modificadas o editar las respuestas, como con cualquier otro origen.

![Edición de preguntas y respuestas de la charla](../media/qnamaker-how-to-chit-chat/edit-chit-chat.png)

Para ver los metadatos, seleccione **View Options** (Opciones de vista) en la barra de herramientas y, luego, **Show metadata** (Mostrar metadatos).

## <a name="add-additional-chit-chat-questions-and-answers"></a>Adición de preguntas y respuestas adicionales de la charla
Puede agregar una charla nueva de QnA que no esté en el conjunto de datos predefinido. Asegúrese de que no va a duplicar un par de QnA que ya esté presente en el conjunto de la charla. Cuando se agregan nuevas preguntas y respuestas de charla, se agregan al origen **Editorial**. Para asegurarse de que el clasificador comprende que se trata de charla, agregue el par de clave y valor de metadatos "Editorial: chitchat", como se muestra en la imagen siguiente:

![![Agregar preguntas y respuestas de charla]\(../media/qnamaker-how-to-chit-chat/add-new-chit-chat.png)](../media/qnamaker-how-to-chit-chat/add-new-chit-chat.png#lightbox)

## <a name="delete-chit-chat-from-an-existing-kb"></a>Eliminación de la charla de una base de conocimiento existente
Seleccione la base de conocimiento y vaya hasta la página **Settings** (Configuración). El origen de charla específico aparece como un archivo, con el nombre de la personalidad seleccionado. Puede eliminarlo como un archivo de origen.

![Eliminación de la charla de la base de conocimiento](../media/qnamaker-how-to-chit-chat/delete-chit-chat.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Importación de una base de conocimiento](../Tutorials/migrate-knowledge-base.md)

## <a name="see-also"></a>Consulte también

[Introducción de QnA Maker](../Overview/overview.md)
