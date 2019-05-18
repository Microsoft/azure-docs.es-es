---
title: Introducción a Azure Media Services v3 | Microsoft Docs
description: En este artículo se proporciona información general de alto nivel sobre Media Services y vínculos a artículos para obtener más información.
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: azure media services, transmisión, difusión, en vivo, sin conexión
ms.service: media-services
ms.devlang: multiple
ms.topic: overview
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 05/07/2019
ms.author: juliako
ms.custom: mvc
ms.openlocfilehash: 4fb42014e2c5389cf2a849ddf5114ab1f59c9e28
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65414018"
---
# <a name="azure-media-services-v3-overview"></a>Introducción a Azure Media Services v3

Azure Media Services es una plataforma basada en la nube que le permite compilar soluciones que consiguen un streaming de vídeo con calidad de difusión, mejoran la accesibilidad y la distribución, analizan el contenido, etc. Tanto si es un desarrollador de aplicaciones, un centro de llamadas, una autoridad gubernamental o una empresa de entretenimiento, Media Services le ayuda a crear aplicaciones que ofrecen experiencias multimedia de alta calidad a grandes audiencias de los navegadores y dispositivos móviles más populares en la actualidad. 

> [!NOTE]
> Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](developers-guide.md) admitidos.

## <a name="what-can-i-do-with-media-services"></a>¿Qué puedo hacer con Media Services?

Media Services le permite generar una variedad de flujos de trabajo multimedia en la nube. A continuación, se indican algunos ejemplos de lo que se puede lograr con Media Services.  

* Entregue vídeos en varios formatos para que se puedan reproducir en una amplia variedad de navegadores y dispositivos. Para la entrega de streaming en vivo y bajo petición a varios clientes (dispositivos móviles, televisión, equipos, etc.), el contenido de audio y vídeo debe codificarse y comprimirse adecuadamente. Para ver cómo entregar y hacer streaming de este tipo de contenido, consulte [Quickstart: Encode and stream files](stream-files-dotnet-quickstart.md) (Inicio rápido: codificación y streaming de archivos).
* Haga streaming de eventos deportivos para una gran audiencia en línea, como fútbol, béisbol, deportes universitarios o de instituto, etc. 
* Retransmita reuniones y eventos públicos, como plenos de ayuntamientos, juntas municipales y reuniones de órganos legislativos.
* Analice contenido de audio o vídeos grabados. Por ejemplo, para lograr una mayor satisfacción del cliente, las organizaciones pueden extraer texto a voz y crear paneles e índices de búsqueda. A continuación, pueden extraer inteligencia de las quejas habituales, los orígenes de las quejas y otros datos relevantes.
* Cree un servicio de suscripción de vídeo y haga streaming de contenido protegido por DRM cuando un cliente (por ejemplo, un estudio cinematográfico) necesite restringir el acceso y el uso de una obra con derechos de autor.
* Entregue contenido sin conexión para su reproducción en aviones, trenes y automóviles. Es posible que un cliente tenga que descargar el contenido en su teléfono o tableta para reproducirlo cuando tenga previsto desconectarse de la red.
* Implemente una plataforma de vídeo educativa con Azure Media Services y [Azure Cognitive Services APIs](https://docs.microsoft.com/azure/#pivot=products&panel=ai) para subtítulos de voz a texto, traducción a varios idiomas, etc. 
* Use Azure Media Services en conjunción con [Azure Cognitive Services APIs](https://docs.microsoft.com/azure/#pivot=products&panel=ai) para agregar subtítulos y leyendas a los vídeos. con el fin de llegar a más público (por ejemplo, personas con discapacidades auditivas o usuarios que quieren leer el contenido en un idioma diferente).
* Habilite Azure CDN para lograr un gran escalado y mejorar el control de picos de carga puntuales (por ejemplo, al inicio de un evento de lanzamiento de un producto). 

## <a name="how-can-i-get-started-with-v3"></a>¿Cómo puedo comenzar con v3? 

Aprenda a codificar y a empaquetar contenido, a hacer streaming de vídeos a petición, a realizar difusión en directo y a analizar vídeos con Media Services v3. En los tutoriales, las referencias de API y otra documentación, se muestra cómo entregar de forma segura secuencias de audio o vídeo, a petición y en directo, que se escalan a millones de usuarios.

> [!TIP]
> Antes de empezar a desarrollar, revise:<br/>* [Conceptos fundamentales](concepts-overview.md) (incluye conceptos importantes: empaquetado, codificación, protección, etc.)<br/>* [Desarrollo con las API de Media Services v3](media-services-apis-overview.md) (incluye información acerca de cómo acceder a las API, convenciones de nomenclatura, etc.)

### <a name="quickstarts"></a>Guías de inicio rápido  

Los inicios rápidos muestran instrucciones básicas para que los nuevos clientes prueben rápidamente Media Services.

* [Transmitir en secuencias archivos de vídeo (.NET)](stream-files-dotnet-quickstart.md)
* [Hacer streaming de archivos de vídeo: CLI](stream-files-cli-quickstart.md)
* [Hacer streaming de archivos de vídeo: Node.js](stream-files-nodejs-quickstart.md)
    
### <a name="tutorials"></a>Tutoriales 

Los tutoriales muestran procedimientos basados en un escenario para algunas de las principales tareas de Media Services.

* [Codificación de archivos remotos y streaming de vídeo: REST](stream-files-tutorial-with-rest.md)
* [Codificación de archivos cargados y streaming de vídeo: .NET](stream-files-tutorial-with-api.md)
* [Streaming en vivo: .NET](stream-live-tutorial-with-api.md)
* [Análisis del vídeo: .NET](analyze-videos-tutorial-with-api.md)
* [Cifrado dinámico AES-128: .NET](protect-with-aes128.md)
    
### <a name="how-to-guides"></a>Guías de procedimientos

Los artículos contienen ejemplos de código que muestran cómo realizar una tarea. En esta sección, encontrará muchos ejemplos. Esta es solo una pequeña muestra:

* [Creación de una cuenta: CLI](create-account-cli-how-to.md)
* [Acceso a las API: CLI](access-api-cli-how-to.md)
* [Comenzar a desarrollar con SDK](developers-guide.md)
* [Codificación con HTTPS como entrada de trabajo: .NET](job-input-from-http-how-to.md)  
* [Supervisión de eventos: Portal](monitor-events-portal-how-to.md)
* [Cifrado dinámico con varias soluciones de DRM: .NET](protect-with-drm.md) 
* [Procedimiento de codificación con una transformación personalizada: CLI](custom-preset-cli-howto.md)

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Información sobre conceptos fundamentales](concepts-overview.md)

