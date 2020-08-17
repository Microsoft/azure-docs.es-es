---
title: Detección de personajes animados con Video Indexer
titleSuffix: Azure Media Services
description: En este tema se muestra cómo usar la detección de personajes animados con Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 11/19/2019
ms.author: juliako
ms.openlocfilehash: 3b449f68b25fce19dc034d2a0db0ae5ce8a28ec7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047485"
---
# <a name="animated-character-detection-preview"></a>Detección de personajes animados (versión preliminar)

Video Indexer de Azure Media Services admite la detección, la agrupación y el reconocimiento de personajes en contenido animado a través de la [integración con la versión personalizada de Cognitive Services](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/). Esta funcionalidad está disponible tanto a través del portal como de la API.

Después de cargar un vídeo animado con un modelo de animación específico, Video Indexer extrae los fotogramas clave, detecta los personajes animados en estos fotogramas, agrupa los personajes similares y elige el mejor ejemplo. A continuación, envía los personajes agrupados a Custom Vision, que identifica los personajes en función de los modelos en los que se entrenó. 

Antes de empezar a entrenar el modelo, los personajes se detectan sin nombre. Al agregar nombres y entrenar el modelo, Video Indexer reconocerá los personajes y les asignará el nombre correspondiente.

## <a name="flow-diagram"></a>Diagrama de flujo

En el siguiente diagrama se muestra el flujo del proceso de detección de personajes animados.

![Diagrama de flujo](./media/animated-characters-recognition/flow.png)

## <a name="accounts"></a>Cuentas

En función de cada tipo de cuenta de Video Indexer, habrá disponibles diferentes conjuntos de características. Para obtener información sobre cómo conectar su cuenta a Azure, consulte [Creación de una cuenta de Video Indexer conectada a Azure](connect-to-azure.md).

* Cuenta de prueba: Video Indexer usa una cuenta de Custom Vision interna para crear el modelo y conectarlo a su cuenta de Video Indexer. 
* Cuenta de pago: la cuenta de Custom Vision se conecta a su cuenta de Video Indexer (si aún no tiene una, primero debe crear una cuenta).

### <a name="trial-vs-paid"></a>De prueba frente a de pago

|Funcionalidad|Versión de prueba|De pago|
|---|---|---|
|Cuenta de Custom Vision|Se administra en segundo plano por Video Indexer. |Su cuenta de Custom Vision está conectada a Video Indexer.|
|Número de modelos de animación|Uno|Hasta 100 modelos por cuenta (limitación de Custom Vision).|
|Entrenamiento del modelo|Video Indexer entrena el modelo para buscar nuevos personajes en otros ejemplos de personajes existentes.|El propietario de la cuenta entrena el modelo cuando está listo para realizar cambios.|
|Opciones avanzadas en Custom Vision|Sin acceso al portal de Custom Vision.|Puede ajustar los modelos usted mismo en el portal de Custom Vision.|

## <a name="use-the-animated-character-detection-with-portal"></a>Uso de la detección de personajes animados con el portal 

En esta sección se describen los pasos que debe seguir para empezar a usar el modelo de detección de personajes animados. 

Como en las cuentas de evaluación la integración de Custom Vision se administra mediante Video Indexer, puede empezar a crear y usar el modelo de caracteres animados y saltarse la siguiente sección ("Conexión de la cuenta de Custom Vision").

### <a name="connect-your-custom-vision-account-paid-accounts-only"></a>Conexión de la cuenta de Custom Vision (solo cuentas de pago)

Si es propietario de una cuenta de pago de Video Indexer, primero debe conectar una cuenta de Custom Vision. Si aún no tiene una cuenta de Custom Vision, cree una. Para obtener más información, consulte [Custom Vision](../../cognitive-services/custom-vision-service/home.md).

> [!NOTE]
> Las dos cuentas deben estar en la misma región. La integración de Custom Vision no se admite actualmente en la región de Japón.

#### <a name="connect-a-custom-vision-account-with-api"></a>Conexión de una cuenta de Custom Vision con la API 

Siga estos pasos para conectar su cuenta de Custom Vision a Video Indexer, o bien para cambiar la cuenta de Custom Vision que está conectada actualmente a Video Indexer:

1. Vaya a [www.customvision.ai](https://www.customvision.ai) e inicie sesión.
1. Copie las siguientes claves: 

    * Clave de entrenamiento (para el recurso de entrenamiento)
    * Clave de predicción (para el recurso de predicción)
    * Punto de conexión 
    * Identificador del recurso de predicción
    
    > [!NOTE]
    > Para proporcionar todas las claves, debe tener dos recursos independientes en Custom Vision, uno para el entrenamiento y otro para la predicción.
1. Inicie sesión en [Video Indexer](https://vi.microsoft.com/).
1. Haga clic en el signo de interrogación de la esquina superior derecha de la página y elija **API Reference** (Referencia de la API).
1. Asegúrese de que está suscrito a API Management haciendo clic en la pestaña **Products** (Productos). Si tiene una API conectada, puede continuar con el paso siguiente; de lo contrario, suscríbase. 
1. En el portal para desarrolladores, haga clic en **Complete API Reference** (Completar referencia de la API) y vaya a **Operations** (Operaciones).  
1. Seleccione **Connect Custom Vision Account (PREVIEW)** [Conectar cuenta de Custom Vision (VERSIÓN PRELIMINAR)] y haga clic en **Try it** (Probar).
1. Rellene los campos obligatorios, así como el token de acceso y haga clic en **Send** (Enviar). 

    Para obtener más información sobre cómo obtener el token de Video Indexer, vaya al [portal para desarrolladores](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Account-Access-Token?) y consulte la [documentación pertinente](video-indexer-use-apis.md#obtain-access-token-using-the-authorization-api).  
1. Cuando la llamada devuelva la respuesta 200 OK, significa que la cuenta está conectada.
1. Para comprobar la conexión, vaya al portal de [Video Indexer](https://vi.microsoft.com/):
1. Haga clic en el botón **Content model customization** Personalización del modelo de contenido() en la esquina superior derecha de la página.
1. Vaya a la pestaña **Animated characters** (Personajes animados).
1. Una vez que haga clic en Administrar modelos en Custom Vision**, se le transferirá a la cuenta de Custom Vision que acaba de conectar.

> [!NOTE]
> Actualmente, solo se admiten los modelos creados a través de Video Indexer. Los modelos que se crean a través de Custom Vision no estarán disponibles. Además, el procedimiento recomendado es editar los modelos creados a través de Video Indexer solo con la plataforma de Video Indexer, ya que los cambios realizados a través de Custom Vision pueden provocar resultados no deseados.

### <a name="create-an-animated-characters-model"></a>Creación de un modelo de personajes animados

1. Vaya al sitio web de [Video Indexer](https://vi.microsoft.com/) e inicie sesión.
1. Haga clic en el botón de personalización del modelo de contenido en la esquina superior derecha de la página.

    ![Personalización del modelo de contenido](./media/animated-characters-recognition/content-model-customization.png)
1. Vaya a la pestaña **Animated characters** (Personajes animados) en la sección de personalización de modelos.
1. Haga clic en **Agregar modelo**.
1. Asigne un nombre al modelo y haga clic en Entrar para guardarlo.

> [!NOTE]
> El procedimiento recomendado es tener un modelo de Custom Vision para cada serie animada. 

### <a name="index-a-video-with-an-animated-model"></a>Indexación de un vídeo con un modelo animado

1. Haga clic en el botón **Cargar** en la parte superior de la página.
1. Elija un vídeo para cargar (desde un archivo o una dirección URL).
1. Haga clic en **Opciones avanzadas**.
1. En **People / Animated characters** (Personas/personajes animados), elija **Animation models** (Modelos de animación).
1. Si tiene un modelo, se elegirá automáticamente y, si tiene varios modelos, puede elegir el que proceda del menú desplegable.
1. Haga clic en el botón para cargar.
1. Una vez que se indexa el vídeo, verá los personajes detectados en la sección **Animated characters** (Personajes animados) del panel **Información**.

> [!NOTE] 
> Antes de etiquetar y entrenar el modelo, todos los personajes animados se denominarán "Unknown #X" (Desconocido X). Después de entrenar el modelo, también se reconocerán.

### <a name="customize-the-animated-characters-models"></a>Personalización de los modelos de personajes animados

1. Etiquete y entrene el modelo.

    1. Etiquete el personaje detectado editando su nombre. Una vez que se entrena un personaje en el modelo, se reconocerá en el siguiente vídeo indexado con ese modelo. 
    1. Para etiquetar un personaje animado en el vídeo, vaya a la pestaña **Información** y haga clic en el botón **Editar** en la esquina superior derecha de la ventana.
    1. En el panel **Información**, haga clic en cualquiera de los personajes animados detectados y cambie sus nombres "Unknown #X" (Desconocido X), o los que asignara previamente al personaje, por otros.
    1. Después de escribir el nuevo nombre, haga clic en el icono de verificación junto a él. Esto guarda el nuevo nombre en el modelo en Video Indexer.
    1. Una vez que haya terminado de editar todos los nombres que desee, debe entrenar el modelo.

        Abra la página de personalización y haga clic en la pestaña de personalización; a continuación, haga clic en la pestaña **Caracteres animados** y haga clic en el botón **Entrenar** para entrenar el modelo.
         
        Si tiene una cuenta de pago, puede hacer clic en el vínculo **Administrar modelos en Custom Vision** (tal como se muestra a continuación). Se le derivará entonces a la página del modelo en **Custom Vision**.
 
        ![Personalización del modelo de contenido](./media/animated-characters-recognition/content-model-customization-tab.png)

     1. Una vez entrenado, cualquier vídeo que se indexe o se vuelva a indexar con ese modelo reconocerá los personajes entrenados. 
    Las cuentas de pago que tienen acceso a su cuenta de Custom Vision pueden ver los modelos y las imágenes etiquetadas allí. Obtenga más información [sobre cómo mejorar el clasificador en Custom Vision.](../../cognitive-services/custom-vision-service/getting-started-improving-your-classifier.md)

1. Elimine un personaje animado.

    1. Para eliminar un personaje animado de la información del vídeo, vaya a la pestaña **Información** y haga clic en el botón **Editar** en la esquina superior derecha de la ventana.
    1. Elija el personaje animado y, a continuación, haga clic en el botón **Eliminar** bajo su nombre.

    > [!NOTE]
    > Esto eliminará la información de este vídeo, pero no afectará al modelo.

1. Elimine un modelo.

    1. Haga clic en el botón de **personalización del modelo de contenido** en el menú superior y haga clic en la pestaña **Animated characters** (Personajes animados).
    1. Haga clic en el icono de puntos suspensivos a la derecha del modelo que desea eliminar y, a continuación, en el botón para eliminar.
    
    * Cuenta de pago: el modelo se desconectará de Video Indexer y no podrá volver a conectarlo.
    * Cuenta de prueba: el modelo también se eliminará de Custom Vision. 
    
        > [!NOTE]
        > En una cuenta de prueba, solo tiene un modelo que puede usar. Después de eliminarlo, no puede entrenar otros modelos.

## <a name="use-the-animated-character-detection-with-api"></a>Uso de la detección de personajes animados con API 

1. Conecte una cuenta de Custom Vision.

    Si es propietario de una cuenta de pago de Video Indexer, primero debe conectar una cuenta de Custom Vision. <br/>
    Si aún no tiene una cuenta de Custom Vision, cree una. Para obtener más información, consulte [Custom Vision](../../cognitive-services/custom-vision-service/home.md).

    [Conecte la cuenta de Custom Vision con API](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Connect-Custom-Vision-Account?tags=&pattern=&groupBy=tag).
1. Cree un modelo de personajes animados.

    Use la API de [creación de modelo de animación](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Create-Animation-Model?&groupBy=tag).
1. Indexe o vuelva a indexar un vídeo.

    Use la API de [reindexación](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-Index-Video?). 
1. Personalice los modelos de personajes animados.

    Use la API de [entrenamiento de modelo de animación](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Train-Animation-Model?&groupBy=tag).

### <a name="view-the-output"></a>Visualización de la salida

Vea los personajes animados en el archivo JSON generado.

```json
"animatedCharacters": [
    {
    "videoId": "e867214582",
    "confidence": 0,
    "thumbnailId": "00000000-0000-0000-0000-000000000000",
    "seenDuration": 201.5,
    "seenDurationRatio": 0.3175,
    "isKnownCharacter": true,
    "id": 4,
    "name": "Bunny",
    "appearances": [
        {
            "startTime": "0:00:52.333",
            "endTime": "0:02:02.6",
            "startSeconds": 52.3,
            "endSeconds": 122.6
        },
        {
            "startTime": "0:02:40.633",
            "endTime": "0:03:16.6",
            "startSeconds": 160.6,
            "endSeconds": 196.6
        },
    ]
    },
]
```

## <a name="limitations"></a>Limitaciones

* Actualmente, la funcionalidad de "identificación de animación" no se admite en la región de Asia Oriental.
* Es posible que los personajes que aparezcan pequeños o lejanos en el vídeo no se identifiquen correctamente si la calidad del vídeo es deficiente.
* Se recomienda usar un modelo por conjunto de personajes animados (por ejemplo, por serie animada).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Video Indexer](video-indexer-overview.md)
