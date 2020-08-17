---
title: Inicio rápido de la biblioteca cliente de Python para Content Moderator
titleSuffix: Azure Cognitive Services
description: En este inicio rápido empezará a usar la biblioteca cliente de Content Moderator de Azure Cognitive Services para Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 01/27/2020
ms.author: pafarley
ms.openlocfilehash: 9b246734e454e4ca93e0f3176c124cd8869bd015
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87298858"
---
Introducción a la biblioteca cliente de Content Moderator para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Content Moderator es un servicio cognitivo que comprueba el contenido de texto, imágenes y vídeo en busca de material que sea potencialmente ofensivo, peligroso o no deseable por algún otro motivo. Cuando se encuentra dicho material, el servicio aplica las etiquetas adecuadas (marcas) al contenido. Luego, la aplicación puede controlar el contenido marcado para cumplir las normativas o mantener el entorno adecuado para los usuarios.

Use la biblioteca cliente de Content Moderator para Python para:

* [Moderar texto](#moderate-text)
* [Usar una lista de términos personalizados](#use-a-custom-terms-list)
* [Moderar imágenes](#moderate-images)
* [Usar una lista de imágenes personalizadas](#use-a-custom-image-list)
* [Crear una revisión](#create-a-review)

[Documentación de referencia](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/contentmoderator?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-contentmoderator) | [Paquete (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-contentmoderator/) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Python 3.x](https://www.python.org/)

## <a name="setting-up"></a>Instalación

### <a name="create-a-content-moderator-azure-resource"></a>Creación de un recurso de Azure para Content Moderator

Los servicios de Azure Cognitive Services se representan por medio de recursos de Azure a los que se suscribe. Cree un recurso para Content Moderator con [Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) o la [CLI de Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli) en la máquina local. También puede:

* Ver este recurso en [Azure Portal](https://portal.azure.com/).

Después de obtener una clave del recurso, [cree las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave y la dirección URL del punto de conexión, llamadas `CONTENT_MODERATOR_SUBSCRIPTION_KEY` y `CONTENT_MODERATOR_ENDPOINT`, respectivamente.
 
### <a name="create-a-python-script"></a>Crear un script de Python

Cree un script de Python y ábralo en el editor o el IDE que prefiera. Luego, agregue las siguientes instrucciones `import` en la parte superior del archivo.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imports)]

Después, cree variables para la ubicación del punto de conexión del recurso y claves como variables de entorno. 

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_vars)]

> [!NOTE]
> Si ha creado las variables de entorno después de haber iniciado la aplicación, deberá cerrar y volver a abrir el editor, el IDE o el shell que lo ejecuta para acceder a ellas.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Puede instalar la biblioteca cliente de Content Moderator con el siguiente comando:

```console
pip install --upgrade azure-cognitiveservices-vision-contentmoderator
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de la biblioteca cliente de Python para Content Moderator.

|Nombre|Descripción|
|---|---|
|[ContentModeratorClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.content_moderator_client.contentmoderatorclient?view=azure-python)|Esta clase es necesaria para todas las funciones de Content Moderator. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases.|
|[ImageModerationOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.imagemoderationoperations?view=azure-python)|Esta clase proporciona la funcionalidad para analizar imágenes de contenido para adultos, información personal o caras humanas.|
|[TextModerationOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.textmoderationoperations?view=azure-python)|Esta clase proporciona la funcionalidad para analizar el texto y detectar el idioma, palabras obscenas, errores e información personal.|
[ReviewsOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.reviewsoperations?view=azure-python)|Esta clase proporciona la funcionalidad de las API de revisión, como los métodos para crear trabajos, flujos de trabajo personalizados y revisiones humanas.|

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Content Moderator para Python:

* [Autenticar el cliente](#authenticate-the-client)
* [Moderar texto](#moderate-text)
* [Usar una lista de términos personalizados](#use-a-custom-terms-list)
* [Moderar imágenes](#moderate-images)
* [Usar una lista de imágenes personalizadas](#use-a-custom-image-list)
* [Crear una revisión](#create-a-review)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este inicio rápido se da por supuesto que ha [creado variables de entorno](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) para la clave y el punto de conexión de Content Moderator.

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto [CognitiveServicesCredentials](https://docs.microsoft.com/python/api/msrest/msrest.authentication.cognitiveservicescredentials?view=azure-python) con la clave y úselo con el punto de conexión para crear un objeto [ContentModeratorClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.content_moderator_client.contentmoderatorclient?view=azure-python).

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_client)]

## <a name="moderate-text"></a>Moderar texto

En el código siguiente se usa un cliente de Content Moderator para analizar un cuerpo de texto e imprimir los resultados en la consola. Primero, cree una carpeta **text_files/** en la raíz del proyecto y agregue un archivo *content_moderator_text_moderation.txt*. Agregue su propio texto a este archivo o use el siguiente texto de ejemplo:

```
Is this a grabage email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
Crap is the profanity here. Is this information PII? phone 2065550111
```

Agregue una referencia a la nueva carpeta.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textfolder)]

Luego, agregue el siguiente código al script de Python.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textmod)]

## <a name="use-a-custom-terms-list"></a>Usar una lista de términos personalizados

En el código siguiente se muestra cómo administrar una lista de términos personalizados para la moderación de texto. Puede usar la clase [ListManagementTermListsOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.listmanagementtermlistsoperations?view=azure-python) para crear una lista de términos, administrar los términos individuales y mostrar en pantalla otros cuerpos de texto con los que comparar.

### <a name="get-sample-text"></a>Obtener texto de ejemplo

Para usar este ejemplo , debe crear una carpeta **text_files/** en la raíz del proyecto y agregar un archivo *content_moderator_term_list.txt*. Este archivo debe contener texto orgánico que se comparará con la lista de términos. Puede usar el siguiente texto de ejemplo:

```
This text contains the terms "term1" and "term2".
```

Agregue una referencia a la carpeta si aún no ha definido una.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textfolder)]

### <a name="create-a-list"></a>Crear una lista

Agregue el código siguiente al script de Python para crear una lista de términos personalizados y guarde su valor de identificación.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_create)]

### <a name="define-list-details"></a>Definir los detalles de la lista

Puede usar el identificador de una lista para editar su nombre y descripción.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_details)]

### <a name="add-a-term-to-the-list"></a>Agregar un término a la lista

En el código siguiente se agregan los términos `"term2"` y `"term1"` a la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_add)]

### <a name="get-all-terms-in-the-list"></a>Obtener todos los términos de la lista

Puede usar el identificador de lista para devolver todos los términos de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_getterms)]

### <a name="refresh-the-list-index"></a>Actualizar el índice de lista

Siempre que agregue o quite términos en la lista, debe actualizar el índice para poder usar la lista actualizada.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_refresh)]

### <a name="screen-text-against-the-list"></a>Comparar el texto en pantalla con la lista

La principal funcionalidad de la lista de términos personalizados es comparar un cuerpo de texto con la lista y encontrar si hay términos que coincidan. 

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_screen)]

### <a name="remove-a-term-from-a-list"></a>Eliminar un término de una lista

El código siguiente quita el término `"term1"` de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_remove)]

### <a name="remove-all-terms-from-a-list"></a>Eliminar todos los términos de una lista

Use el código siguiente para borrar una lista de todos sus términos.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_removeall)]

### <a name="delete-list"></a>Eliminar una lista

Use el código siguiente para eliminar una lista de términos personalizados.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_deletelist)]

## <a name="moderate-images"></a>Moderar imágenes

En el código siguiente se usa un cliente de Content Moderator, junto con un objeto [ImageModerationOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.imagemoderationoperations?view=azure-python), para analizar imágenes en busca de contenido para adultos y de tipo explícito.

### <a name="get-images"></a>Obtener imágenes

Defina una referencia a algunas imágenes para analizar.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemodvars)]

Después, agregue el código siguiente para recorrer en iteración las imágenes. El resto del código de esta sección irá dentro de este bucle.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_iterate)]

### <a name="check-for-adultracy-content"></a>Buscar contenido para adultos o de tipo explícito

El código siguiente comprueba la imagen de la dirección URL especificada en busca de contenido para adultos o de tipo explícito e imprime los resultados en la consola. Consulte la guía de [conceptos de moderación de imágenes](../../image-moderation-api.md) para información sobre lo que significan estos términos.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_ar)]

### <a name="check-for-visible-text"></a>Buscar texto visible

El código siguiente comprueba la imagen en busca de contenido de texto visible e imprime los resultados en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Buscar caras

El código siguiente comprueba la imagen en busca de caras humanas e imprime los resultados en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_face)]

## <a name="use-a-custom-image-list"></a>Usar una lista de imágenes personalizadas

En el código siguiente se muestra cómo administrar una lista personalizada de imágenes para la moderación de imágenes. Esta característica es útil si la plataforma recibe con frecuencia instancias del mismo conjunto de imágenes que quiere filtrar. Al mantener una lista de estas imágenes específicas, puede mejorar el rendimiento. La clase [ListManagementImageListsOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.listmanagementimagelistsoperations?view=azure-python) permite crear una lista de imágenes, administrar las imágenes individuales de la lista y comparar otras imágenes con ella.

Cree las siguientes variables de texto para almacenar las direcciones URL de imágenes que usará en este escenario.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelistvars)]

> [!NOTE]
> Esta no es la lista adecuada, sino una lista informal de imágenes que se agregarán a la sección `add images` del código.


### <a name="create-an-image-list"></a>Creación de una lista de imágenes

Agregue el código siguiente para crear una lista de imágenes y guarde una referencia a su identificador.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_create)]

### <a name="add-images-to-a-list"></a>Agregar imágenes a una lista

En el código siguiente se agregan todas las imágenes a la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_add)]

Defina la función auxiliar **add_images** en otra parte del script.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_addhelper)]

### <a name="get-images-in-list"></a>Obtener imágenes de la lista

El código siguiente imprime los nombres de todas las imágenes de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_getimages)]

### <a name="update-list-details"></a>Actualizar los detalles de la lista

Puede usar el identificador de lista para actualizar el nombre y la descripción de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_updatedetails)]

### <a name="get-list-details"></a>Obtener los detalles de la lista

Use el código siguiente para imprimir los detalles actuales de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_getdetails)]

### <a name="refresh-the-list-index"></a>Actualizar el índice de lista

Después de agregar o quitar imágenes, debe actualizar el índice de la lista antes de poder usarlo para mostrar otras imágenes.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_refresh)]

### <a name="match-images-against-the-list"></a>Buscar coincidencias de imágenes con la lista

La función principal de las listas de imágenes es comparar las nuevas imágenes y ver si hay coincidencias.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_match)]

### <a name="remove-an-image-from-the-list"></a>Quitar una imagen de la lista

El código siguiente quita un elemento de la lista. En este caso, se trata de una imagen que no coincide con la categoría de la lista.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_remove)]

### <a name="remove-all-images-from-a-list"></a>Quitar todas las imágenes de una lista

Use el código siguiente para borrar una lista de imágenes.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_removeall)]

### <a name="delete-the-image-list"></a>Eliminar la lista de imágenes

Use el código siguiente para eliminar una lista de imágenes determinada.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_delete)]

## <a name="create-a-review"></a>Creación de una revisión

Puede usar la biblioteca cliente de Python para Content Moderator para insertar contenido en la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) y que los moderadores humanos puedan revisarlo. Para más información sobre la herramienta de revisión, consulte la [guía conceptual de la herramienta de revisión](../../review-tool-user-guide/human-in-the-loop.md).

En el código siguiente se usa la clase [ReviewsOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.reviewsoperations?view=azure-python) para crear una revisión, recuperar su identificador y comprobar sus detalles después de recibir la entrada humana a través del portal web de la herramienta de revisión.

### <a name="get-review-credentials"></a>Obtener las credenciales de revisión

En primer lugar, inicie sesión en la herramienta de revisión y recupere el nombre del equipo. A continuación, asígnelo a la variable adecuada en el código. Opcionalmente, puede configurar un punto de conexión de devolución de llamada para recibir actualizaciones sobre la actividad de la revisión.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_vars)]

### <a name="create-an-image-review"></a>Crear una revisión de imágenes

Agregue el código siguiente para crear y publicar una revisión para la dirección URL de la imagen especificada. El código guarda una referencia al identificador de la revisión. 
[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_create)]

### <a name="get-review-details"></a>Obtención de detalles de la revisión

Use el código siguiente para comprobar los detalles de una revisión determinada. Después de crear la revisión, puede ir a la herramienta de revisión e interactuar con el contenido. Para obtener información sobre cómo hacerlo, consulte la [guía de procedimientos de revisión](https://docs.microsoft.com/azure/cognitive-services/content-moderator/review-tool-user-guide/review-moderated-images). Cuando haya terminado, puede volver a ejecutar este código y se recuperarán los resultados del proceso de revisión.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_getdetails)]

Si ha usado un punto de conexión de devolución de llamada en este escenario, debería recibir un evento en este formato:

```console
{'callback_endpoint': 'https://requestb.in/qmsakwqm',
 'content': '',
 'content_id': '3ebe16cb-31ed-4292-8b71-1dfe9b0e821f',
 'created_by': 'cspythonsdk',
 'metadata': [{'key': 'sc', 'value': 'True'}],
 'review_id': '201901i14682e2afe624fee95ebb248643139e7',
 'reviewer_result_tags': [{'key': 'a', 'value': 'True'},
                          {'key': 'r', 'value': 'True'}],
 'status': 'Complete',
 'sub_team': 'public',
 'type': 'Image'}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `python` en el archivo de inicio rápido.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca de Python para Content Moderator para realizar tareas de moderación. A continuación, lea una guía conceptual para aprender más sobre la moderación de imágenes u otro contenido multimedia.

> [!div class="nextstepaction"]
>[Conceptos de moderación de imágenes](https://docs.microsoft.com/azure/cognitive-services/content-moderator/image-moderation-api)

* [¿Qué es Azure Content Moderator?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/ContentModerator/ContentModeratorQuickstart.py).