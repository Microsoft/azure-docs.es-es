---
title: 'Cómo probar una base de conocimiento: QnA Maker'
description: La prueba de la base de conocimiento de QnA Maker es una parte importante de un proceso iterativo para mejorar la precisión de las respuestas devueltas. Puede probar la base de conocimiento mediante una interfaz de chat mejorada que también permite realizar ediciones.
ms.topic: conceptual
ms.date: 03/05/2020
ms.openlocfilehash: 9c6d7fc9a421ce466ecd91aaac5c2b83f42a1624
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650941"
---
# <a name="test-your-knowledge-base-in-qna-maker"></a>Prueba de una base de conocimiento en QnA Maker

La prueba de la base de conocimiento de QnA Maker es una parte importante de un proceso iterativo para mejorar la precisión de las respuestas devueltas. Puede probar la base de conocimiento mediante una interfaz de chat mejorada que también permite realizar ediciones.

## <a name="interactively-test-in-qna-maker-portal"></a>Prueba interactiva en QnA Maker Portal

1. Acceda a la base de conocimiento; para ello, seleccione su nombre en la página **Mis bases de conocimiento**.
1. Para acceder al panel deslizante Prueba, seleccione **Prueba** en el panel superior de la aplicación.
1. Escriba una consulta en el cuadro de texto y presione ENTRAR.
1. Se devuelve como resultado la respuesta con mejor coincidencia de la base de conocimiento.

### <a name="clear-test-panel"></a>Borrar el panel de prueba

Para borrar las consultas de prueba especificadas y sus resultados de la consola de pruebas, seleccione **Empezar de nuevo** en la esquina superior izquierda del panel Probar.

### <a name="close-test-panel"></a>Cerrar el panel Prueba

Para cerrar el panel Prueba, vuelva a seleccionar el botón **Prueba**. Mientras el panel Prueba está abierto, no puede editar el contenido de la base de conocimiento.

### <a name="inspect-score"></a>Inspeccionar puntuación

Puede inspeccionar los detalles del resultado de las pruebas en el panel Inspeccionar.

1.  Con el panel deslizante Prueba abierto, seleccione **Inspeccionar** para obtener más detalles sobre dicha respuesta.

    ![Inspeccionar respuestas](../media/qnamaker-how-to-test-kb/inspect.png)

2.  Aparece el panel Inspección. El panel incluye la intención de mayor puntuación, así como cualquier entidad identificada. El panel muestra el resultado de la expresión seleccionada.

### <a name="correct-the-top-scoring-answer"></a>Corrección de la respuesta con mayor puntuación

Si la respuesta con mayor puntuación es incorrecta, seleccione la respuesta correcta en la lista y haga clic en **Guardar y entrenar**.

![Corrección de la respuesta con mayor puntuación](../media/qnamaker-how-to-test-kb/choose-answer.png)

### <a name="add-alternate-questions"></a>Agregar preguntas alternativas

Puede agregar formas alternativas de una pregunta a una respuesta determinada. Escriba las respuestas alternativas en el cuadro de texto y presiones ENTRAR para agregarlas. Seleccione **Guardar y entrenar** para almacenar las actualizaciones.

![Agregar preguntas alternativas](../media/qnamaker-how-to-test-kb/add-alternate-question.png)

### <a name="add-a-new-answer"></a>Adición de una nueva respuesta

Puede agregar una respuesta nueva si cualquiera de las respuestas existentes coincidentes son incorrectas o la respuesta no existe en la base de conocimiento (ninguna coincidencia adecuada encontrada en la base de conocimiento).

En la parte inferior de la lista de las respuestas, use el cuadro de texto para escribir una nueva respuesta y presione Entrar para agregarla.

Seleccione **Guardar y entrenar** para persistir esta respuesta. Se ha agregado un par pregunta-respuesta nuevo a la base de conocimiento.

> [!NOTE]
> Todas las modificaciones realizadas en la base de conocimiento solo se guardan si presiona el botón **Guardar y entrenar**.

### <a name="test-the-published-knowledge-base"></a>Prueba de la base de conocimiento publicada

Puede probar la versión publicada de la base de conocimiento en el panel prueba. Una vez publicada la base de conocimiento, seleccione el cuadro **Published KB** (Base de conocimiento publicada) y envíe una consulta para obtener resultados de esta.

![Prueba en una base de conocimiento publicada](../media/qnamaker-how-to-test-kb/test-against-published-kb.png)

## <a name="batch-test-with-tool"></a>Pruebas por lotes con herramienta

Use la herramienta de pruebas por lotes cuando quiera:

* determinar la mejor respuesta y la puntuación de un conjunto de preguntas
* validar la respuesta esperada para el conjunto de preguntas

### <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Cree un servicio QnA Maker](../Quickstarts/create-publish-knowledge-base.md) o use un servicio existente que utilice el idioma inglés.
* Descargar el archivo de [ejemplo de turno múltiple`.docx`](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)
* Descargue la [herramienta de realización de pruebas por lotes](https://aka.ms/qnamakerbatchtestingtool) y extraiga el archivo ejecutable del archivo `.zip`.

### <a name="sign-into-qna-maker-portal"></a>Inicio de sesión en el portal de QnA Maker

[Inicie sesión](https://www.qnamaker.ai/) en el portal de QnA Maker.

### <a name="create-a-new-knowledge-base-from-the-multi-turn-sampledocx-file"></a>Creación de una nueva base de conocimiento a partir del archivo multi-turn sample.docx

1. Seleccione **Create a knowledge base** (Crear una base de conocimiento) en la barra de menús.
1. Omita el **paso 1**, ya que debería tener un recurso de QnA Maker, así que vaya al **paso 2** para seleccionar la información de los recursos existente:
    * Identificador de Azure Active Directory
    * Nombre de la suscripción de Azure
    * Nombre del servicio Azure QnA
    * Idioma: inglés
1. Escriba `Multi-turn batch test quickstart` como nombre de la base de conocimiento.

1. En el **paso 4**, establezca la configuración con los siguientes valores:

    |Configuración|Value|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Habilitar extracción en varios turnos de direcciones URL, archivos .pdf o .docx).|Activado|
    |**Texto de respuesta predeterminado**| `Batch test - default answer not found.`|
    |**+ Agregar archivo**|Seleccione la lista de archivos `.docx` descargados en los requisitos previos.|
    |**Charla**|Seleccione **Professional**|

1. En el **paso 5**, seleccione **Create your KB** (Crear base de conocimiento).

    Cuando el proceso de creación finaliza, el portal muestra la base de conocimiento editable.

### <a name="save-train-and-publish-knowledge-base"></a>Almacenamiento, entrenamiento y publicación de la base de conocimiento

1. Seleccione **Save and Train** (Guardar y entrenar) en la barra de herramientas para guardar la base de conocimiento.
1. Seleccione **Publish** (Publicar) en la barra de herramientas y, después, vuelva a seleccionar **Publish** (Publicar) para publicar la base de conocimiento. La publicación hace que la base de conocimiento esté disponible para las consultas desde un punto de conexión de una dirección URL pública. Una vez completada la publicación, guarde la dirección URL del host y la información de la clave del punto de conexión que se muestra en la página **Publish** (Publicar).

    |Datos necesarios| Ejemplo|
    |--|--|
    |Host publicado|`https://YOUR-RESOURCE-NAME.azurewebsites.net`|
    |Clave publicada|`XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` (cadena de 32 caracteres que se muestra después de `Endpoint`)|
    |Id. de aplicación|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (cadena de 36 caracteres que se muestra como parte de `POST`) |

### <a name="create-batch-test-file-with-question-ids"></a>Creación de un archivo de prueba por lotes con identificadores de preguntas

Para usar la herramienta de pruebas por lotes, cree un archivo denominado `batch-test-data-1.tsv` con un editor de texto. El archivo debe estar en formato UTF-8 y tener las siguientes columnas separadas por una tabulación.

|Campos del archivo de entrada TSV|Notas|Ejemplo|
|--|--|--|
|Id. de base de conocimiento|El identificador de la base de conocimiento que se encuentra en la página Publish (Publicar). Pruebe varias bases de conocimiento en el mismo servicio a la vez en un solo archivo mediante el uso de diferentes identificadores de base de conocimiento en un único archivo.|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (cadena de 36 caracteres que se muestra como parte de `POST`) |
|Pregunta|El texto de la pregunta que un usuario escribiría. Máximo de 1000 caracteres|`How do I sign out?`|
|Etiquetas de metadatos|opcional|`topic:power` usa el formato _clave:valor_|
|Parámetro principal|opcional|`25`|
|Identificador de respuesta esperado|opcional|`13`|

Para esta base de conocimiento, agregue tres filas que tengan solo las dos columnas necesarias al archivo. La primera columna es el identificador de la base de conocimiento y la segunda debería ser la siguiente lista de preguntas:

|Columna 2: preguntas|
|--|
|`Use Windows Hello to sign in`|
|`Charge your Surface Pro 4`|
|`Get to know Windows 10`|

Estas preguntas son el texto exacto de la base de conocimiento y deberían devolver 100 como puntuación de confianza.

Después, agregue varias preguntas, similares a estas, pero no exactamente las mismas, en 3 filas más y use el mismo identificador de base de conocimiento:

|Columna 2: preguntas|
|--|
|`What is Windows Hello?`|
|`How do I charge the laptop?`|
|`What features are in Windows 10?`|

> [!CAUTION]
> Asegúrese de que cada columna está separada solo por un delimitador de tabulador. Los espacios iniciales o finales se agregan a los datos de la columna y provocarán que el programa produzca excepciones cuando el tipo o el tamaño sean incorrectos.

Cuando se abre en Excel, el archivo de prueba por lotes tiene un aspecto similar al de la siguiente imagen. Por motivos de seguridad el identificador de la base de conocimiento se ha reemplazado por `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. En el caso de su prueba por lotes, asegúrese de que la columna muestra el identificador de la base de conocimiento.

> [!div class="mx-imgBorder"]
> ![Entrada de la primera versión del archivo .tsv desde la prueba por lotes](../media/batch-test/batch-test-1-input.png)

### <a name="test-the-batch-file"></a>Prueba del archivo por lotes

Ejecute el programa de pruebas por lotes y use el siguiente formato de la CLI en la línea de comandos.

Reemplace `YOUR-RESOURCE-NAME` y `ENDPOINT-KEY` por los valores que ha elegido para el nombre del servicio y la clave del punto de conexión. Estos valores se encuentran en la página **Configuración** del portal de QnA Maker.

```console
batchtesting.exe batch-test-data-1.tsv https://YOUR-RESOURCE-NAME.azurewebsites.net ENDPOINT-KEY out.tsv
```
La prueba finaliza y genera el archivo `out.tsv`:

> [!div class="mx-imgBorder"]
> ![Salida de la primera versión del archivo .tsv desde la prueba por lotes](../media/batch-test/batch-test-1-output.png)

Por motivos de seguridad el identificador de la base de conocimiento se ha reemplazado por `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. En el caso de su prueba por lotes, la columna muestra el identificador de la base de conocimiento.

En la cuarta columna de la salida de prueba de la puntuación de confianza, se ve que las tres primeras preguntas han devuelto una puntuación de 100 según lo previsto, porque las preguntas aparecen exactamente así en la base de conocimiento. Las tres últimas preguntas, con el nuevo texto de la pregunta, no devuelven 100 como puntuación de confianza. Con el fin de aumentar la puntuación tanto de la prueba como de los usuarios, es preciso agregar más preguntas alternativas a la base de conocimiento.

### <a name="testing-with-the-optional-fields"></a>Prueba con los campos opcionales

Una vez que conozca el formato y el proceso, puede generar un archivo de prueba para ejecutarlo en la base de conocimiento desde un origen de datos como, por ejemplo, los registros de chats.

Al estar automatizados el origen de datos y el proceso, el archivo de prueba se puede ejecutar muchas veces, con distintas configuraciones, para determinar cuáles son los valores correctos.

Por ejemplo, si tiene un registro de chat y desea determinar qué texto del registro de chat se aplica a cada campo de metadatos, cree un archivo de prueba y establezca los campos de metadatos de cada fila. Ejecute la prueba y revise las filas que coinciden con los metadatos. Por lo general, las coincidencias deberían ser positivas, pero debe revisar los resultados, por si hubiera falsos positivos. Un falso positivo es una fila que coincide con los metadatos, pero que según el texto no debería coincidir.

### <a name="using-optional-fields-in-the-input-batch-test-file"></a>Uso de campos opcionales en el archivo de prueba por lotes de entrada

Use el gráfico siguiente para saber cómo buscar los datos opcionales en los valores de los campos.

|Número de columna|Columna opcional|Ubicación de los datos|
|--|--|--|
|3|metadata|Exporte la base de conocimiento en los pares _clave:valor_ existentes.|
|4|top|Se recomienda el valor predeterminado `25`.|
|5|Identificador del conjunto de preguntas y respuestas|Exporte los valores de identificador de la base de conocimiento existente. Tenga también en cuenta que los identificadores se devolvieron en el archivo de salida.|

### <a name="add-metadata-to-the-knowledge-base"></a>Incorporación de metadatos a la base de conocimiento

1. En el portal de QnA, en la página **Edit** (Editar), agregue los metadatos de `topic:power` a las siguientes preguntas:

    |Preguntas|
    |--|
    |Cargue Surface Pro 4|
    |Compruebe el nivel de batería|

    Dos pares de QnA tienen los metadatos establecidos.

    > [!TIP]
    > Para ver los metadatos y los identificadores de QnA de cada uno de los conjuntos, exporte la base de conocimiento. Seleccione la página **Setting** (Configuración) y, después, seleccione **Export** (Exportar) como archivo `.xls`. Busque el archivo descargado, ábralo con Excel y examine los metadatos y el identificador.

1. Seleccione **Save and train** (Guardar y entrenar), luego seleccione la página **Publish** (Publicar) y, finalmente, seleccione el botón **Publish** (Publicar). Estas acciones hacen que el cambio esté disponible para la prueba por lotes. Descargue la base de conocimiento desde la página **Configuración**.

    El archivo descargado tiene el formato correcto para los metadatos y el identificador del conjunto de preguntas y respuestas correcto. Use estos campos en la sección siguiente

    > [!div class="mx-imgBorder"]
    > ![Base de conocimiento exportada con metadatos](../media/batch-test/exported-knowledge-base-with-metadata.png)

### <a name="create-a-second-batch-test"></a>Creación de una segunda prueba por lotes

Hay dos escenarios principales para las pruebas por lotes:
* **Procesamiento de archivos del registro de chat**: determine la respuesta principal de una pregunta que anteriormente no se había detectado. La situación más habitual es cuando se necesita procesar un archivo de registro de consultas, por ejemplo, las preguntas del usuario de un bot de chat. Cree una prueba de archivo por lotes que tenga solo las columnas necesarias. La prueba devuelve la respuesta principal a cada pregunta. Eso no significa que esa respuesta sea la correcta. Una vez que complete la prueba, pase a la prueba de validación.
* **Prueba de validación**: valide la respuesta esperada. Esta prueba requiere la validación de todas las preguntas y respuestas esperadas coincidentes en la prueba por lotes, lo que puede requerir algún proceso manual.

En el procedimiento siguiente se supone que el escenario es procesar los registros de chat como sigue

1. Cree un archivo de prueba por lotes que incluya los datos opcionales, `batch-test-data-2.tsv`. Agregue las 6 filas del archivo de entrada de prueba por lotes original y, luego, agregue los metadatos, la respuesta principal y el identificador del par de QnA de cada fila.

    Para simular el proceso automatizado de comprobar el texto nuevo de los registros de chat con la base de conocimiento, establezca los metadatos de todas las columnas en el mismo valor: `topic:power`.

    > [!div class="mx-imgBorder"]
    > ![Entrada de la segunda versión del archivo .tsv desde la prueba por lotes](../media/batch-test/batch-test-2-input.png)

1. Vuelva a ejecutar la prueba, pero cambie los nombres de los archivos de entrada y salida para indicar que es la segunda prueba.

    > [!div class="mx-imgBorder"]
    > ![Salida de la segunda versión del archivo .tsv desde la prueba por lotes](../media/batch-test/batch-test-2-output.png)

### <a name="test-results-and-an-automated-test-system"></a>Resultados de la prueba y un sistema de pruebas automatizadas

Este archivo de salida de prueba se puede analizar como parte de una canalización de prueba continua automatizada.

Esta salida de prueba específica debe leerse de la siguiente forma: todas las filas se han filtrado con metadatos y como no todas las filas coincidían con los metadatos de la base de conocimiento, se ha devuelto la respuesta predeterminada para las filas no coincidentes ("no se encontró ninguna coincidencia exacta en la base de conocimiento"). De las filas que coincidían, se devolvieron el identificador de QnA y la puntuación.

Todas las filas devolvieron la etiqueta de incorrecta, ya que ninguna de ellas que coincidía con el identificador de respuesta esperado.

Con estos resultados verá que puede tomar un registro de chat y usar el texto como consulta de cada fila. Sin saber nada de los datos, los resultados le dicen mucho acerca de los datos que puede usar en adelante:

* metadatos
* identificador de QnA
* score

¿Fue buena idea filtrar con metadatos al realizar la prueba? Sí y no. El sistema de prueba debe crear archivos de prueba para cada par de metadatos, así como una prueba sin pares de metadatos.

### <a name="clean-up-resources"></a>Limpieza de recursos

Si va a dejar de probar la base de conocimiento, elimine la herramienta de archivo por lotes y los archivos de prueba.

Si va a dejar de usar esta base de conocimiento, elimínela, para lo que debe seguir estos pasos:

1. En el portal de QnA Maker, seleccione **My Knowledge bases** (Mis bases de conocimiento) en el menú superior.
1. En la lista de bases de conocimiento, seleccione el icono **Delete** (Eliminar) en la fila de la base de conocimiento de este inicio rápido.

La [documentación de referencia sobre la herramienta](../reference-tsv-format-batch-testing.md) incluye:

* el ejemplo de la línea de comandos de la herramienta
* el formato de los archivos de entrada y salida de TSV

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Publicación de una base de conocimientos](./publish-knowledge-base.md)
