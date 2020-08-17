---
title: 'Preguntas más frecuentes sobre Video Indexer: Azure'
titleSuffix: Azure Media Services
description: En este artículo se ofrecen respuestas a preguntas frecuentes sobre Video Indexer de Azure Media Services.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/12/2020
ms.author: juliako
ms.openlocfilehash: d533735af340b23e8a79b6c5953df748a548609c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046970"
---
# <a name="video-indexer-frequently-asked-questions"></a>Preguntas más frecuentes sobre Video Indexer

En este artículo se proporcionan respuestas a las preguntas más frecuentes sobre Video Indexer.

## <a name="general-questions"></a>Preguntas generales

### <a name="what-is-video-indexer"></a>¿Qué es Video Indexer?

Video Indexer es un servicio de inteligencia artificial que forma parte de Microsoft Azure Media Services. Video Indexer proporciona una orquestación de varios modelos de aprendizaje automático que permiten extraer fácilmente información detallada de un vídeo. Para proporcionar información precisa y avanzada, Video Indexer utiliza varios canales del vídeo: audio, voz y objetos visuales. La información de Video Indexer se puede usar de muchas maneras, como mejorar la detectabilidad de contenido y la accesibilidad, crear oportunidades de monetización o crear experiencias que usen esta información. Video Indexer proporciona una interfaz web para pruebas, configuración y personalización de los modelos en la cuenta. Los desarrolladores pueden usar una API REST para integrar Video Indexer en el sistema de producción. 

### <a name="what-can-i-do-with-video-indexer"></a>¿Qué puedo hacer con Video Indexer?

Algunas de las operaciones que Video Indexer puede realizar en los archivos multimedia son:

* Identificación y extracción de diálogos e identificación de hablantes.
* Identificación y extracción del texto en pantalla de un vídeo.
* Detección de objetos en un archivo de vídeo.
* Identificación de marcas (por ejemplo: Microsoft) en las pistas de audio y en el texto en pantalla de un vídeo.
* Detección y reconocimiento de caras en una base de datos de caras de personas famosas y en una base de datos de caras definida por el usuario.
* Extracción de los temas tratados, aunque no necesariamente mencionados, en el contenido de audio y vídeo.
* Creación de subtítulos a partir de la pista de audio.

Para más información y características de Video Indexer, consulte [Introducción](video-indexer-overview.md).

### <a name="how-do-i-get-started-with-video-indexer"></a>¿Cómo empiezo a usar Video Indexer?

Video Indexer incluye una oferta de evaluación gratuita que proporciona 600 minutos en la interfaz web y 2400 minutos a través de la API. También puede [iniciar sesión en la interfaz web de Video Indexer](https://www.videoindexer.ai/) y probarlo usted mismo con cualquier identidad web y sin tener que configurar una suscripción de Azure. Siga [este sencillo laboratorio de introducción](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md) para obtener una idea más clara de cómo usar Video Indexer.

Para indexar archivos de vídeo y audio a escala, debe conectar Video Indexer a una suscripción de Microsoft Azure de pago. Puede obtener más información sobre los precios en la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/).

Puede obtener más información sobre cómo comenzar en [Comenzar](video-indexer-get-started.md).

### <a name="do-i-need-coding-skills-to-use-video-indexer"></a>¿Necesito habilidades de codificación para usar Video Indexer?

Puede usar la interfaz web de Video Indexer para evaluar, configurar y administrar la cuenta **sin necesidad de código**.  Cuando esté listo para desarrollar aplicaciones más complejas, puede usar [Video Indexer API](https://api-portal.videoindexer.ai/) para integrarlo en sus propias aplicaciones, sitios web o [flujos de trabajo personalizados que usen tecnologías sin servidor, como Azure Logic Apps](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/) o Azure Functions.

### <a name="do-i-need-machine-learning-skills-to-use-video-indexer"></a>¿Necesito habilidades de aprendizaje automático para usar Video Indexer?

No, Video Indexer proporciona la integración de varios modelos de aprendizaje automático en una canalización. Con la indexación de un archivo de audio o vídeo a través de Video Indexer se recupera un conjunto completo de información extraído en una escala de tiempo compartida sin habilidades de aprendizaje automático ni conocimientos sobre los algoritmos necesarios del lado del cliente.

### <a name="what-media-formats-does-video-indexer-support"></a>¿Qué formatos multimedia admite Video Indexer?

Video Indexer es compatible con los formatos multimedia más comunes. Vea la lista de los [formatos estándar de Azure Media Encoder](../latest/media-encoder-standard-formats.md) para más detalles.

### <a name="how-do-i-upload-a-media-file-into-video-indexer-and-what-are-the-limitations"></a>¿Cómo cargo un archivo multimedia en Video Indexer y cuáles son las limitaciones?

En el portal web de Video Indexer puede cargar un archivo multimedia mediante el cuadro de diálogo de carga de archivos o indicando una dirección URL que directamente hospede el archivo de origen (consulte el [ejemplo](https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/Ignite-short.mp4)). Cualquier dirección URL que hospede el contenido multimedia mediante iFrame o código para insertar no funcionará (consulte el [ejemplo](https://www.videoindexer.ai/accounts/7e1282e8-083c-46ab-8c20-84cae3dc289d/videos/5cfa29e152/?t=4.11)). 

Para obtener más información, vea esta [guía paso a paso](./upload-index-videos.md).

#### <a name="limitations"></a>Limitaciones

* El nombre del vídeo no puede superar los 80 caracteres.
* Si carga un vídeo mediante una matriz de bytes, el tamaño del vídeo se limita a 2 GB (30 GB al usar la dirección URL). 

Para obtener la lista completa, consulte [Consideraciones y limitaciones para la carga](upload-index-videos.md#uploading-considerations-and-limitations).

### <a name="how-long-does-it-take-video-indexer-to-extract-insights-from-media"></a>¿Cuánto tiempo tarda Video Indexer en extraer información de los archivos multimedia?

El tiempo necesario para indexar un archivo de audio o vídeo, tanto mediante Video Indexer API como mediante la interfaz web, depende de varios parámetros, como la longitud del archivo y su calidad, la cantidad de información que se encuentra en él, el número de [unidades reservadas](../previous/media-services-scale-media-processing-overview.md) disponibles y si el [punto de conexión de streaming](../previous/media-services-streaming-endpoints-overview.md) está habilitado o no. Recomendamos que ejecute algunos archivos de prueba con su propio contenido y tome un promedio para obtener una idea más clara.

### <a name="can-i-create-customized-workflows-to-automate-processes-with-video-indexer"></a>¿Puedo crear flujos de trabajo personalizados para automatizar los procesos con Video Indexer?

Sí, puede integrar Video Indexer en tecnologías sin servidor, como Logic Apps, Flow y [Azure Functions](https://azure.microsoft.com/services/functions/). Más detalles sobre los conectores de [Logic Apps](https://azure.microsoft.com/services/logic-apps/) y [Flow](https://flow.microsoft.com/en-us/) para Video Indexer [aquí](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/). Puede ver algunos proyectos de automatización realizados por partners en el repositorio de [ejemplos de Video Indexer](https://github.com/Azure-Samples/media-services-video-indexer).

### <a name="in-which-azure-regions-is-video-indexer-available"></a>¿En qué regiones de Azure está disponible Video Indexer?

Puede ver en qué regiones de Azure está disponible Video Indexer en la página [regiones](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services&regions=all).

### <a name="can-i-customize-video-indexer-models-for-my-specific-use-case"></a>¿Puedo personalizar los modelos de Video Indexer para mi caso de uso concreto? 

Sí. En Video Indexer puede personalizar algunos de los modelos disponibles para que se adapten mejor a sus necesidades. 

Por ejemplo, el modelo Person admite de serie 1 millón de caras de reconocimiento de famosos, pero también puede entrenarlo para que reconozca otras caras que no están en esa base de datos. 

Para obtener detalles, vea los artículos sobre la personalización de los modelos [Person](customize-person-model-overview.md), [Brands](customize-brands-model-overview.md) y [Language](customize-language-model-overview.md). 

###  <a name="can-i-edit-the-videos-in-my-library"></a>¿Puedo editar los vídeos de mi biblioteca?

Sí. Presione el botón **Editar el vídeo** de la pantalla de la biblioteca o el botón **Abrir en el editor** de la pantalla del reproductor para ir a la pestaña **Proyectos**. Puede crear un proyecto nuevo y agregar más vídeos de la biblioteca para editarlos a la vez. Una vez que haya terminado, puede representar el vídeo y descargarlo. 

Si quiere obtener información sobre el nuevo vídeo, puede indexarlo con Video Indexer y aparecerá en la biblioteca con su información.

### <a name="can-i-index-multiple-audio-streams-or-channels"></a>¿Puedo indexar varios canales o secuencias de audio?

Si hay varias secuencias de audio, Video Indexer toma la primera que encuentra y procesa solo esa. En cualquier secuencia de audio que procesa Video Indexer, toma los distintos canales (si los hay) y los procesa como mono. En el caso de la manipulación de secuencias/canales, puede usar comandos ffmpeg en el archivo antes de indexarlo.

### <a name="what-is-the-sla-for-video-indexer"></a>¿Qué es el Acuerdo de Nivel de Servicio para Video Indexer?

El Acuerdo de Nivel de Servicio de Azure Media Service abarca Video Indexer y se encuentra en la página del [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/media-services/v1_2/). El Acuerdo de Nivel de Servicio solo se aplica a las cuentas de pago de Video Indexer y no a la evaluación gratuita.

## <a name="privacy-questions"></a>Cuestiones de privacidad

### <a name="are-video-and-audio-files-indexed-by-video-indexer-stored"></a>¿Se almacenan los archivos de audio y vídeo indexados por Video Indexer?

Sí, a menos que elimine el archivo de Video Indexer mediante el sitio web o Video Indexer API, los archivos de audio y vídeo se almacenan. Para la evaluación gratuita, los archivos de audio y vídeo que indexe se almacenan en la región de Azure Este de EE. UU. En caso contrario, los archivos de audio y vídeo se almacenan en la cuenta de almacenamiento de su suscripción de Azure.

### <a name="can-i-delete-my-files-that-are-stored-in-video-indexer-portal"></a>¿Puedo eliminar mi archivos almacenados en el portal de Video Indexer?

Sí, siempre puede eliminar los archivos de audio y vídeo, así como los metadatos y la información extraída de ellos por Video Indexer. Una vez eliminado un archivo de Video Indexer, este, y sus metadatos e información se eliminan permanentemente de Video Indexer. Sin embargo, si ha implementado su propia solución de copia de seguridad en Azure Storage, el archivo permanece en el almacenamiento de Azure.

### <a name="can-i-control-user-access-to-my-video-indexer-account"></a>¿Puedo controlar el acceso de usuario a mi cuenta de Video Indexer?

Sí, solo los administradores de cuenta pueden invitar y dejar de invitar a personas a sus cuentas, así como asignar quién tiene privilegios de edición y quién tiene acceso de solo lectura.

### <a name="who-has-access-to-my-video-and-audio-files-that-have-been-indexed-andor-stored-by-video-indexer-and-the-metadata-and-insights-that-were-extracted"></a>¿Quién tiene acceso a los archivos de audio y vídeos que Video Indexer ha indexado o almacenado y a los metadatos y la información que se extrajeron?

Cualquiera que tenga el vínculo al contenido de vídeo o audio y su información puede acceder al contenido cuya privacidad haya configurado como pública. Al contenido de vídeo o audio que tiene Privado como configuración de privacidad solo pueden acceder los usuarios que han sido invitados a la cuenta de ese contenido. La configuración de privacidad del contenido también se aplica a los metadatos y la información que Video Indexer extrae. La configuración de privacidad se asigna al carga el archivo de vídeo o audio. También puede cambiar la configuración de privacidad después de indexación.

### <a name="what-access-does-microsoft-have-to-my-video-or-audio-files-that-have-been-indexed-andor-stored-by-video-indexer-and-the-metadata-and-insights-that-were-extracted"></a>¿Qué tipo de acceso tiene Microsoft a mis archivos de audio y vídeo que Video Indexer ha indexado o almacenado y a los metadatos y la información que se extrajeron?

De conformidad con los [Términos de los servicios en línea de Azure](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31) (OST), el contenido es totalmente suyo y Microsoft solo puede acceder a él y a los metadatos e información que Video Indexer extrae de él de conformidad con los Términos de los servicios en línea y la declaración de privacidad de Microsoft.

### <a name="are-the-custom-models-that-i-build-in-my-video-indexer-account-available-to-other-accounts"></a>¿Los modelos personalizados que creo en mi cuenta de Video Indexer están disponibles para otras cuentas?

 No, los modelos personalizados que cree en su cuenta no están disponibles para ninguna otra cuenta. Video Indexer actualmente le permite crear modelos personalizados de [marcas](customize-brands-model-overview.md), [idioma](customize-language-model-overview.md) y [persona](customize-person-model-overview.md) en su cuenta. Estos modelos solo están disponibles en la cuenta en la que los ha creado.
  
### <a name="is-the-content-indexed-by-video-indexer-kept-within-the-azure-region-where-i-am-using-video-indexer"></a>¿Se mantiene el contenido indexado por Video Indexer dentro de la región de Azure donde estoy usando Video Indexer?

Sí, el contenido y su información se mantienen dentro de la región de Azure a menos que tenga una configuración manual en su suscripción de Azure que use varias regiones de Azure. 

### <a name="what-is-the-privacy-policy-for-video-indexer"></a>¿Qué es la directiva de privacidad de Video Indexer?

Video Indexer está cubierto por la [declaración de privacidad de Microsoft](https://privacy.microsoft.com/privacystatement). En la declaración de privacidad se explican los datos personales que Microsoft procesa, cómo los procesa y los propósitos de este procesamiento. Para más información sobre la privacidad, visite [Microsoft Trust Center](https://www.microsoft.com/trustcenter).

### <a name="what-certifications-does-video-indexer-have"></a>¿Qué certificaciones tiene Video Indexer?

Video Indexer actualmente tiene la certificación SOC. Para revisar la certificación de Video Indexer, consulte [Microsoft Trust Center](https://www.microsoft.com/trustcenter/compliance/complianceofferings?product=Azure).

### <a name="what-is-the-difference-between-private-and-public-videos"></a>¿Cuál es la diferencia entre los vídeos privados y públicos? 

Cuando se cargan vídeos en Video Indexer, puede elegir entre dos valores de privacidad: privado y público. Los vídeos públicos son accesibles para cualquier usuario, incluidos los usuarios anónimos y no identificados. Los privados están restringidos exclusivamente a los miembros de la cuenta. 

### <a name="i-tried-to-upload-a-video-as-public-and-it-was-flagged-for-inappropriate-or-offensive-content-what-does-that-mean"></a>He intentado cargar un vídeo como público y se ha marcado como contenido inadecuado u ofensivo, ¿qué significa eso? 

Al cargar un vídeo en Video Indexer, los algoritmos y los modelos realizan un análisis de contenido automático con el fin de asegurarse de que no se presente públicamente ningún contenido inadecuado. Si se detecta que un vídeo es sospechoso de tener contenido explícito, no es posible establecerlo como público. Pero los miembros de la cuenta pueden acceder a él como un vídeo privado (verlo, descargar la información y los artefactos extraídos, y realizar otras operaciones disponibles para los miembros de la cuenta).   

Para establecer el vídeo para acceso público, puede: 

* Crear su propia capa de interfaz (como aplicación o sitio web) y usarla para interactuar con el servicio Video Indexer. De este modo, el vídeo sigue siendo privado en el portal y los usuarios pueden interactuar con él a través de su interfaz. Por ejemplo, puede seguir obteniendo la información o permitir la visualización del vídeo en su propia interfaz. 
* Solicitar una revisión humana del contenido, lo que daría lugar a la eliminación de la restricción siempre que el contenido no sea explícito. 

    Esta opción se puede contemplar si el sitio web de Video Indexer es usado directamente por los usuarios como la capa de interfaz y para visualización pública (sin autenticar). 

## <a name="api-questions"></a>Preguntas sobre la API

### <a name="what-apis-does-video-indexer-offer"></a>¿Qué API ofrece Video Indexer?

Video Indexer APIs permiten indexar, extraer metadatos, administrar recursos, traducir, insertar, personalizar modelos, y mucho más. Para información más detallada sobre el uso de Video Indexer API, consulte [Video Indexer Developer Portal](https://api-portal.videoindexer.ai/) (Portal para desarrolladores de Video Indexer).

### <a name="what-client-sdks-does-video-indexer-offer"></a>¿Qué SDK de cliente ofrece Video Indexer?

Actualmente no se ofrecen SDK de cliente. El equipo de Video Indexer está trabajando en los SDK y los planes para entregarlos pronto.

### <a name="how-do-i-get-started-with-video-indexers-api"></a>¿Cómo empiezo a usar la API de Video Indexer?

Siga [Tutorial: get started with the Video Indexer API](video-indexer-use-apis.md) (Tutorial: introducción a la API de Video Indexer).

### <a name="what-is-the-difference-between-the-video-indexer-api-and-the-azure-media-service-v3-api"></a>¿Cuál es la diferencia entre Video Indexer API y Azure Media Services v3 API?

Actualmente algunas de las características de Video Indexer API y Azure Media Service v3 API se superponen. Más información sobre la comparación de ambos servicios [aquí](compare-video-indexer-with-media-services-presets.md).

### <a name="what-is-an-api-access-token-and-why-do-i-need-it"></a>¿Qué es un token de acceso de API y por qué lo necesito?

La API de Video Indexer contiene Authorization API y Operations API. Authorizations API contiene las llamadas que ofrecen el token de acceso. Cada llamada a Operations API debe asociarse con un token de acceso, correspondiente al ámbito de autorización de la llamada.

Los tokens de acceso son necesarios para usar las API de Video Indexer por motivos de seguridad. Esto garantiza que todas las llamadas proceden de usted o los usuarios que tengan permisos de acceso a su cuenta. 

### <a name="what-is-the-difference-between-account-access-token-user-access-token-and-video-access-token"></a>¿Cuál es la diferencia entre el token de acceso de cuenta, el token de acceso de usuario y el token de acceso de vídeo?

* Nivel de cuenta: los tokens de acceso de nivel de cuenta permiten realizar operaciones en el nivel de cuenta o de vídeo. Por ejemplo, cargar un vídeo, enumerar todos los vídeos u obtener información detallada de un vídeo.
* Nivel de usuario: los tokens de acceso de nivel de usuario permiten realizar operaciones en el nivel de usuario. Por ejemplo, obtener cuentas asociadas.
* Nivel de vídeo: los tokens de acceso de nivel de vídeo permiten realizar operaciones en el nivel de vídeo. Por ejemplo, obtener información detallada del vídeo, descargar títulos, obtener widgets, etc.

### <a name="how-often-do-i-need-to-get-a-new-access-token-when-do-access-tokens-expire"></a>¿Con qué frecuencia es necesario obtener un nuevo token de acceso? ¿Cuándo expiran los tokens de acceso?

Los tokens de acceso expiran cada hora, por lo que tendrá que generar un nuevo token de acceso cada hora. 

### <a name="what-are-the-login-options-to-video-indexer-developer-portal"></a>¿Cuáles son las opciones de inicio de sesión para el portal para desarrolladores de Video Indexer?

Puede iniciar sesión con Azure AD o con una cuenta de Microsoft, Google o Facebook. 

Una vez que haya registrado su cuenta de correo electrónico mediante un proveedor de identidades, no podrá usarla con otro proveedor de identidades.

## <a name="billing-questions"></a>Preguntas sobre facturación

### <a name="how-much-does-video-indexer-cost"></a>¿Cuánto cuesta Video Indexer?

Video Indexer usa un sencillo modelo de precios de pago por uso según la duración de entrada del contenido que se indexa. Es posible que se apliquen cargos adicionales para la codificación, el streaming, el almacenamiento, el uso de la red y las unidades reservadas multimedia. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/)para más información.

### <a name="when-am-i-billed-for-using-video-indexer"></a>¿Cuándo se me factura el uso de Video Indexer?

Al enviar un vídeo para a indexarlo, el usuario define la indexación de modo que sea análisis de vídeo, de audio o ambos. Esto determina la SKU que se va a cobrar. Si se produce un error de nivel crítico durante el procesamiento, se devuelve un código de error como respuesta. En este caso, no se realiza facturación.  Un error crítico puede provocarlo un error en nuestro código o un fallo crítico en una dependencia interna del servicio. Errores como una incorrecta identificación o la extracción de conclusiones no se consideran críticos y se devuelve una respuesta. En los casos en los que se devuelva una respuesta válida (código sin errores), se realiza facturación.
 
### <a name="does-video-indexer-offer-a-free-trial"></a>¿Video Indexer ofrece una evaluación gratuita?

Sí, Video Indexer ofrece una evaluación gratuita que ofrece un servicio completo y la funcionalidad de API. Hay una cuota de 600 minutos de vídeos para los usuarios de la interfaz web y de 2400 minutos para los usuarios de la API. 

## <a name="next-steps"></a>Pasos siguientes

[Información general](video-indexer-overview.md)
