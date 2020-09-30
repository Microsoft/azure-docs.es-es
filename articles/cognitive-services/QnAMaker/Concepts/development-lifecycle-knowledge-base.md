---
title: 'Ciclo de vida de una base de conocimiento: QnA Maker'
description: QnA Maker aprende mejor en un ciclo iterativo de cambios en el modelo, ejemplos de expresiones, publicación y recopilación de datos de las consultas de punto de conexión.
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 7bbf729fa80e4b41a85b8dfd1080decea1bae108
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89612278"
---
# <a name="knowledge-base-lifecycle-in-qna-maker"></a>Ciclo de vida de una base de conocimiento de QnA Maker
QnA Maker aprende mejor en un ciclo iterativo de cambios en el modelo, ejemplos de expresiones, publicación y recopilación de datos de las consultas de punto de conexión.

![Ciclo de creación](../media/qnamaker-concepts-lifecycle/kb-lifecycle.png)

## <a name="creating-a-qna-maker-knowledge-base"></a>Creación de una base de conocimiento de QnA Maker
El punto de conexión de la base de conocimiento de QnA Maker proporciona una respuesta mejor para las consultas de usuario basadas en el contenido de la base de conocimiento. La creación de una base de conocimiento es una acción que se realiza una sola vez para configurar un repositorio de contenido de preguntas, respuestas y los metadatos asociados. Se puede crear una knowledge base mediante el rastreo de contenido ya existente, como los orígenes siguientes:

- Páginas de preguntas más frecuentes
- Manuales de productos
- Pares de preguntas y respuestas

Aprenda cómo [crear una base de conocimiento](../quickstarts/create-publish-knowledge-base.md).

## <a name="testing-and-updating-the-knowledge-base"></a>Probar y actualizar la base de conocimiento

La base de conocimiento está lista para probarla una vez que se llena con el contenido, bien editorialmente o mediante extracción automática. Las pruebas interactivas se pueden realizar en el portal de QnA Maker, a través del panel **Prueba**. Escriba consultas comunes de los usuarios. A continuación, compruebe que las respuestas se devuelvan con la respuesta correcta y una puntuación de confianza suficiente.


* **Para corregir las puntuaciones de confianza baja**: agregue preguntas alternativas.
* **Cuando una consulta devuelve incorrectamente la [respuesta predeterminada](../How-to/change-default-answer.md)**: agregue nuevas respuestas a la pregunta correcta.

Este bucle ajustado de prueba-actualización continúa hasta que esté satisfecho con los resultados. Vea cómo [probar la base de conocimiento](../How-To/test-knowledge-base.md).

Para KB de gran tamaño, use pruebas automatizadas con [generateAnswer API](../how-to/metadata-generateanswer-usage.md#get-answer-predictions-with-the-generateanswer-api) y la propiedad `isTest` del cuerpo que consulta la base de conocimiento `test` en lugar de la base de conocimiento publicada.

```json
{
  "question": "example question",
  "top": 3,
  "userId": "Default",
  "isTest": true
}
```

## <a name="publish-the-knowledge-base"></a>Publicación de la base de conocimiento
Cuando haya terminado de probar la base de conocimiento, puede publicarla. Al publicar, se inserta la versión más reciente de la base de conocimiento probada en un índice de Azure Cognitive Search dedicado que representa la base de conocimiento **publicada**. También se crea un punto de conexión al que puede llamar en su aplicación o bot de chat.

Debido a la acción de publicación, ninguno de los cambios adicionales que realice en la versión de prueba de knowledge base afectará a la versión publicada. La versión publicada puede estar activa en una aplicación de producción.

Cada una de estas bases de conocimiento se pueden probar por separado. Mediante las API, puede dirigirse a la versión de prueba de la base de conocimiento con la propiedad del cuerpo `isTest` en la llamada de generateAnswer.

Vea cómo [publicar la base de conocimiento](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base).

## <a name="monitor-usage"></a>Supervisión de uso
Para poder registrar los registros de chat de su servicio, deberá habilitar Application Insights cuando [cree su servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md).

Puede obtener diversos datos de análisis de su uso del servicio. Obtenga más información sobre cómo utilizar Application Insights para obtener [datos de análisis del servicio QnA Maker](../How-To/get-analytics-knowledge-base.md).

Según la información que obtenga de los análisis, realice [las actualizaciones correspondientes en su base de conocimiento](../How-To/edit-knowledge-base.md).

## <a name="version-control-for-data-in-your-knowledge-base"></a>Control de versiones para datos en su base de conocimiento

El control de versiones para los datos se proporciona a través de las características de importación/exportación en la página **Configuración** del portal de QnA Maker.

Puede realizar una copia de seguridad de una base de conocimiento al exportarla en formato `.tsv` o `.xls`. Una vez exportado, incluya este archivo en la comprobación de control de código fuente normal.

Cuando necesite volver a una versión específica, debe importar ese archivo desde el sistema local. Una base de conocimiento exportada **debe** usarse solo a través de la opción de importación en la página **Configuración**. No se puede usar como origen de datos de un archivo o URL de documento. De este modo, reemplazará las preguntas y respuestas que se encuentran actualmente en la base de conocimiento con el contenido del archivo importado.

## <a name="test-and-production-knowledge-base"></a>Base de conocimiento de prueba y producción
Una base de conocimiento es el repositorio de conjuntos de preguntas y respuestas que se crea, se mantiene y se usa en QnA Maker. Cada recurso de QnA Maker puede contener varias bases de conocimiento.

Una base de conocimiento tiene dos estados: *prueba* y *publicada*.

### <a name="test-knowledge-base"></a>Base de conocimiento de prueba

La *knowledge base de prueba* es la versión editada y guardada actualmente. La versión de prueba se ha probado para comprobar la exactitud y la integridad de las respuestas. Los cambios realizados en la base de conocimientos de prueba no afectan al usuario final de la aplicación o bot de chat. La base de conocimiento de prueba se conoce como `test` en la solicitud HTTP. La base de conocimiento `test` está disponible a través del panel interactivo **Prueba** del portal de QnA Maker.

### <a name="production-knowledge-base"></a>Base de conocimiento de producción

La *base de conocimiento publicada* es la versión que se utiliza en la aplicación o en el bot de chat. Al publicar una knowledge base, el contenido de la versión de prueba se coloca en la versión publicada. La knowledge base publicada es la versión que utiliza la aplicación o el bot de chat. Asegúrese de que el contenido sea correcto y se haya probado. La base de conocimiento publicada se conoce como `prod` en la solicitud HTTP.


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Sugerencias de aprendizaje activo](./active-learning-suggestions.md)