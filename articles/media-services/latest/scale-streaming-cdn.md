---
title: Transmisión de contenido con la integración de CDN
titleSuffix: Azure Media Services
description: Obtenga información sobre la transmisión de contenido con la integración de CDN, así como la captura previa y la característica Origin-Assist CDN-Prefetch.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: e1ea0a43783fb7abdc17655e3a3431d125d426f8
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291286"
---
# <a name="stream-content-with-cdn-integration"></a>Transmisión de contenido con la integración de CDN

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Content Delivery Network (CDN) ofrece a los desarrolladores una solución global para la entrega rápida de contenido con alto ancho de banda a los usuarios mediante el almacenamiento en caché del contenido en nodos físicos estratégicamente situados en todo el mundo.  

CDN almacena en caché el contenido transmitido desde un [punto de conexión de streaming (origen)](streaming-endpoint-concept.md) de Media Services por códec, por protocolo de streaming, por velocidad de bits, por formato de contenedor y por cifrado/DRM. Para cada combinación de formato-velocidad de bits-cifrado protocolo-contenedor códec-streaming, habrá una caché de CDN independiente.

El contenido popular se servirá directamente desde la caché de CDN, siempre que el fragmento de vídeo esté almacenado en caché. Es probable que el contenido en vivo se almacene en caché porque suele haber muchas personas mirando exactamente lo mismo. El contenido a petición puede ser un poco más complejo porque podría haber parte del contenido que fuera popular y otra que no. Si hay millones de recursos de vídeo que no son populares (solo uno o dos visores a la semana), pero hay miles de personas que miran vídeos diferentes, la red CDN pasa a ser mucho menos efectiva.

También debe tener en cuenta cómo funciona el streaming adaptable. Cada fragmento de vídeo individual se almacena en caché como una entidad independiente. Por ejemplo, imagine la primera vez que se mira un vídeo determinado. Si el visor omite parte y mira solo unos segundos aquí y allá, solo los fragmentos de vídeo asociados con lo que miró esa persona se almacenan en la caché de CDN. Con el streaming adaptable, se suelen tener de 5 a 7 velocidades de bits de vídeo distintas. Si una persona está mirando a una velocidad de bits y otra persona a una velocidad de bits diferente, se almacenan en caché cada uno por separado en la red CDN. Incluso si dos personas están mirando a la misma velocidad de bits podrían hacer streaming a través de protocolos diferentes. Cada protocolo (HLS, MPEG-DASH, Smooth Streaming) se almacena en caché por separado. Por lo tanto, cada velocidad de bits y protocolo se almacenan en caché por separado y solo se almacenan en caché los fragmentos de vídeo que se han solicitado.

A excepción del entorno de prueba, se recomienda que CDN esté habilitado para los puntos de conexión de streaming Estándar y Premium. Cada tipo de punto de conexión de streaming tiene un límite de rendimiento admitido diferente.
Es difícil realizar un cálculo preciso del número máximo de streamings simultáneos que admite un punto de conexión de streaming, ya que hay varios factores que se deben tener en cuenta. Entre ellas se incluyen las siguientes:

- Velocidades de bits máximas usadas para el streaming.
- Comportamiento de búfer previo y de conmutación del reproductor. Los reproductores intentan transferir en ráfagas los segmentos desde un origen y usan la velocidad de carga para calcular el cambio de velocidad de bits adaptable. Si un punto de conexión de streaming se aproxima a la saturación, los tiempos de respuesta pueden variar y los reproductores empiezan a cambiar a una calidad inferior. Dado que esto reduce la carga en los reproductores de punto de conexión de streaming, revierta a una mayor calidad creando desencadenadores de conmutación no deseados.
En general, es seguro calcular el número máximo de streamings tomando el rendimiento máximo de los puntos de conexión de streaming y dividiéndolo por la velocidad de bits máxima (suponiendo que todos los reproductores usen la velocidad de bits más alta). Por ejemplo, puede tener un punto de conexión de streaming estándar que esté limitado a 600 Mbps y la velocidad de bits máxima de 3 Mbps. En este caso, se admiten aproximadamente 200 secuencias simultáneas a la velocidad de bits más alta. Recuerde también tener en cuentan los requisitos de ancho de banda de audio. Aunque es posible que una secuencia de audio solo se transmita a 128 kbps, el streaming total aumenta rápidamente cuando se multiplica por el número de secuencias simultáneas.

En este tema se describe la habilitación de la [integración de CDN](#enable-azure-cdn-integration). También se explica la captura previa (almacenamiento en caché activo) y el concepto [Origin-Assist CDN-Prefetch](#origin-assist-cdn-prefetch).

## <a name="considerations"></a>Consideraciones

- El [punto de conexión de streaming](streaming-endpoint-concept.md) `hostname` y la dirección URL de streaming se mantienen independientemente de si habilita CDN.
- Si necesita poder probar el contenido con o sin CDN, cree otro punto de conexión de streaming que no tenga la red CDN habilitada.

## <a name="enable-azure-cdn-integration"></a>Habilitación de la integración de Azure CDN

> [!IMPORTANT]
> No puede habilitar CDN con cuentas de prueba o cuentas educativas de Azure.
>
> La integración de la red CDN está habilitada en todos los centros de datos de Azure excepto para las regiones Gobierno Federal y China.

Después de aprovisionar un punto de conexión de streaming con la red CDN habilitada, hay un tiempo de espera definido en Media Services antes de realizar la actualización de DNS para asignar el punto de conexión de streaming al punto de conexión de CDN.

Si más adelante desea volver a habilitar o deshabilitar la red CDN, punto de conexión de streaming debe estar en estado **stopped** (detenido). Una vez iniciado el punto de conexión de streaming es posible que transcurran hasta dos horas hasta que la integración de Azure CDN se habilite y los cambios se activen en todos los POP de la red CDN. Sin embargo, puede iniciar el punto de conexión de streaming y transmitir sin interrupciones desde el mismo. Una vez completada la integración, la transmisión se realiza desde CDN. Durante el período de aprovisionamiento, el punto de conexión de streaming estará en estado **iniciando** y es posible que note una reducción en el rendimiento.

Cuando se crea el punto de conexión de streaming Estándar, se configura de forma predeterminada con Standard Verizon. Puede configurar los proveedores Premium Verizon o Standard Akamai mediante las API REST.

La integración de Azure Media Services con Azure CDN se implementa en **Azure CDN de Verizon** para puntos de conexión de streaming estándar. Los puntos de conexión de streaming premium pueden configurarse con todos los **proveedores y planes de tarifa de Azure CDN**.

> [!NOTE]
> Para más información sobre Azure CDN, consulte [Información general de la red CDN](../../cdn/cdn-overview.md).

## <a name="determine-if-a-dns-change-was-made"></a>Determinar si se realizó el cambio de DNS

Puede determinar si se ha realizado el cambio de DNS en un punto de conexión de streaming (el tráfico se dirige a la red Azure CDN) mediante <https://www.digwebinterface.com>. Si los resultados tienen los nombres de dominio azureedge.net, el tráfico se dirige ahora a la red CDN.

## <a name="origin-assist-cdn-prefetch"></a>Origin-Assist CDN-Prefetch

El almacenamiento en caché de CDN es un proceso reactivo. Si CDN puede predecir el siguiente objeto que se va a solicitar, también puede solicitar y almacenar en caché el siguiente objeto de forma proactiva. Con este proceso, se puede lograr un acierto de caché para todos los objetos (o la mayoría de ellos), por lo que se mejora el rendimiento.

El concepto de captura previa se esfuerza por colocar los objetos en el "perímetro de Internet" previendo que el reproductor lo solicitará de manera inminente, lo que reduce el tiempo de entrega del objeto al reproductor.

Para lograr este objetivo, un punto de conexión de streaming (origen) y CDN deben funcionar conjuntamente de dos formas:

- El origen de Media Services debe tener la "inteligencia" (Origin-Assist) para informar a CDN del siguiente objeto que se va a capturar previamente.
- CDN realiza la captura previa y el almacenamiento en caché (la parte de CDN-Prefetch). CDN también debe tener la "inteligencia" para notificar al origen si se trata de una captura previa o una captura normal y para controlar las respuestas 404. Además, debe tener una manera de evitar un bucle de captura previa sin fin.

### <a name="benefits"></a>Ventajas

Las ventajas de la característica *Origin-Assist CDN-Prefetch* incluyen:

- La captura previa mejora la calidad de la reproducción de vídeo, ya que coloca previamente los segmentos de vídeo previstos en el perímetro durante la reproducción. Esto reduce la latencia para el espectador y mejora los tiempos de descarga de los segmentos de vídeo. El resultado es un tiempo de inicio del vídeo más rápido y menos repeticiones de almacenamiento en búfer.
- Este concepto es aplicable al escenario general de CDN-Origin y no se limita a los elementos multimedia.
- Akamai ha agregado esta característica a [Akamai Cloud Embed (ACE)](https://learn.akamai.com/en-us/products/media_delivery/cloud_embed.html).

> [!NOTE]
> Esta característica todavía no es aplicable a la red CDN de Akamai integrada con el punto de conexión de streaming de Media Services. Sin embargo, está disponible para los clientes de Media Services que tienen un contrato de Akamai existente y requieren una integración personalizada entre la red CDN de Akamai y el origen de Media Services.

### <a name="how-it-works"></a>Funcionamiento

La compatibilidad de CDN con los encabezados `Origin-Assist CDN-Prefetch` (para el streaming a petición tanto en vivo como en vídeo) está disponible para los clientes que tienen un contrato directo con CDN de Akamai. La característica supone los siguientes intercambios de encabezados HTTP entre CDN de Akamai y el origen de Media Services:

|Encabezado HTTP|Valores|Remitente|Receptor|Propósito|
| ---- | ---- | ---- | ---- | ----- |
|`CDN-Origin-Assist-Prefetch-Enabled` | 1 (valor predeterminado) o 0 |CDN|Origen|Para indicar que la red CDN está habilitada para la captura previa.|
|`CDN-Origin-Assist-Prefetch-Path`| Ejemplo: <br/>Fragments(video=1400000000,format=mpd-time-cmaf)|Origen|CDN|Para proporcionar la ruta de acceso de captura previa de la red CDN.|
|`CDN-Origin-Assist-Prefetch-Request`|1 (solicitud de captura previa) o 0 (solicitud normal)|CDN|Origen|Para indicar que la solicitud de CDN es una captura previa.|

Para ver en acción parte del intercambio de encabezados, puede probar los pasos siguientes:

1. Use Postman o cURL para emitir una solicitud de segmento o fragmento de audio o vídeo al origen de Media Services. Asegúrese de agregar el encabezado `CDN-Origin-Assist-Prefetch-Enabled: 1` en la solicitud.
2. En la respuesta, se debería mostrar el encabezado `CDN-Origin-Assist-Prefetch-Path` y debería tener como valor una ruta de acceso relativa.

### <a name="supported-streaming-protocols"></a>Protocolos de streaming admitidos

La característica `Origin-Assist CDN-Prefetch` admite los siguientes protocolos de streaming para el streaming en vivo y a petición:

* HLS v3
* HLS v4
* HLS CMAF
* DASH (CSF)
* DASH (CMAF)
* Streaming con velocidad de transmisión adaptable

### <a name="faqs"></a>Preguntas más frecuentes

* ¿Qué ocurre si una dirección URL de ruta de acceso de captura previa no es válida y la captura previa de CDN obtiene una respuesta 404?

    CDN solo almacena en caché una respuesta 404 durante 10 segundos (u otro valor configurado).

* Supongamos que tiene un vídeo a petición. Si la captura previa de CDN está habilitada, ¿esta característica implica que una vez que un cliente solicite el primer segmento de vídeo, la captura previa iniciará un bucle para capturar previamente todos los segmentos de vídeo posteriores con la misma velocidad de bits?

    No, CDN-Prefetch solo se realiza después de una solicitud/respuesta iniciada por el cliente. Para evitar un bucle de captura previa, CDN-Prefetch nunca se desencadena con una captura previa.

* ¿La característica Origin-Assist CDN-Prefetch está siempre activada? ¿Cómo se puede activar o desactivar?

    Esta característica está desactivada de manera predeterminada. Los clientes deben activarla a través de la API de Akamai.

* En el caso del streaming en vivo, ¿qué pasaría con Origin-Assist si el siguiente segmento o fragmento no está disponible todavía?

    En este caso, el origen de Media Services no proporcionará el encabezado `CDN-Origin-Assist-Prefetch-Path` y no se ejecutará la característica CDN-Prefetch.

* ¿Cómo funciona `Origin-Assist CDN-Prefetch` con los filtros de manifiesto dinámico?

    Esta característica funciona independientemente del filtro del manifiesto. Cuando el siguiente fragmento está fuera de una ventana de filtro, su dirección URL se podrá encontrar en el manifiesto del cliente sin formato y, a continuación, se devolverá como el encabezado de respuesta de la captura previa de CDN. De este modo, CDN obtendrá la dirección URL de un fragmento filtrado del manifiesto DASH/HLS/Smooth. Sin embargo, el reproductor nunca realizará una solicitud GET a CDN para capturar ese fragmento, ya que dicho fragmento no se incluye en el manifiesto DASH/HLS/Smooth del reproductor (el reproductor no conoce la existencia de ese fragmento).

* ¿Se puede realizar una captura previa de DASH MPD, la lista de reproducción de HLS o el manifiesto Smooth?

    No, DASH MPD, la lista de reproducción maestra de HLS, la lista de reproducción de la variante HLS o la dirección URL del manifiesto Smooth no se agregan al encabezado de captura previa.

* ¿Las direcciones URL de captura previa son relativas o absolutas?

    Mientras que CDN de Akamai admite ambas opciones, el origen de Media Services solo proporciona direcciones URL relativas para la ruta de acceso de captura previa porque el uso de direcciones URL absolutas no presenta ninguna ventaja aparente.

* ¿Esta característica funciona con el contenido protegido con DRM?

    Sí. Ya que esta característica funciona en el nivel HTTP, no descodifica ni analiza ningún segmento o fragmento. No tiene en cuenta si el contenido está cifrado o no.

* ¿Esta característica funciona con la inserción de anuncios del lado del servidor (SSAI)?
    
    Funciona para el contenido original o principal (el contenido del vídeo original antes de la inserción de anuncios), ya que SSAI no cambia la marca de tiempo del contenido de origen indicada en el origen de Media Services. Si esta característica funciona con el contenido del anuncio, depende de si el origen del anuncio es compatible con Origin-Assist. Por ejemplo, si el contenido del anuncio también se hospeda en Azure Media Services (en el mismo origen o uno diferente), también se realizará la captura previa del contenido del anuncio.

* ¿Esta característica funciona con el contenido UHD/HEVC?

    Sí.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* Asegúrese de revisar el documento [punto de conexión de streaming (origen)](streaming-endpoint-concept.md).
* En el ejemplo [de este repositorio](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs) se muestra cómo iniciar el punto de conexión de streaming predeterminado con .NET.
