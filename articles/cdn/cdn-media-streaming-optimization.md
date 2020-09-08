---
title: Optimización del streaming multimedia con Azure CDN
description: Obtenga información sobre las opciones para optimizar los medios de streaming en Azure Content Delivery Network, como el uso compartido de caché parcial y el tiempo de espera de relleno de caché.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/01/2018
ms.author: allensu
ms.openlocfilehash: c3ab722f182e32cf2f3aca6bb2f3d5a9598264af
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192614"
---
# <a name="media-streaming-optimization-with-azure-cdn"></a>Optimización del streaming multimedia con Azure CDN 
 
El uso de vídeo de alta definición está aumentando en Internet, lo que crea problemas de entrega eficaz de archivos grandes. Los clientes esperan una reproducción fluida de vídeo bajo demanda o de recursos de vídeo en vivo en una variedad de redes y clientes en todo el mundo. Un mecanismo de entrega rápido y eficiente de los archivos de streaming multimedia es fundamental para garantizar una experiencia sin complicaciones y divertida para el consumidor.  

El streaming en vivo de contenido multimedia es especialmente difícil de entregar, debido a los grandes tamaños y al número de usuarios simultáneos. Los retardos prolongados hacen que los usuarios se marchen. Como las transmisiones en vivo no se pueden almacenar en caché por adelantado y las latencias de gran tamaño no son aceptables para el público, los fragmentos de vídeo se deben entregar de una manera oportuna. 

Los patrones de solicitud de streaming también ofrecen algunos desafíos nuevos. Cuando se publica para vídeo bajo demanda una transmisión en vivo popular o una nueva serie, es posible que miles o millones de espectadores soliciten la transmisión al mismo tiempo. En este caso, la consolidación de solicitud inteligente es vital para no sobrecargar los servidores de origen cuando los recursos todavía no están almacenados en caché.
 

## <a name="media-streaming-optimizations-for-azure-cdn-from-microsoft"></a>Optimizaciones de streaming multimedia para Azure CDN de Microsoft

Los puntos de conexión **Azure CDN Estándar de Microsoft** proporcionan streaming de recursos multimedia directamente mediante el uso del tipo de optimización de entrega web general. 

La optimización del streaming multimedia de **Azure CDN Estándar de Microsoft** es eficaz para el streaming multimedia en vivo o de vídeo bajo demanda donde se usan fragmentos multimedia individuales para la entrega. Este proceso es diferente a la transferencia de un solo recurso de gran tamaño a través de descarga progresiva o mediante solicitudes de intervalo de bytes. Para información sobre ese estilo de entrega multimedia, consulte [Optimización de descarga de archivos grandes mediante Azure CDN](cdn-large-file-optimization.md).

Los tipos de optimización de entrega multimedia general o de vídeo bajo demanda usan Azure Content Delivery Network (CDN) con optimizaciones de back-end para entregar los recursos multimedia más rápidamente. También usan configuraciones para recursos multimedia en función de procedimientos recomendados aprendidos con el tiempo.

### <a name="partial-cache-sharing"></a>Uso compartido de caché parcial
El uso compartido de caché parcial permite que la red CDN entregue contenido parcialmente almacenado en caché a las nuevas solicitudes. Por ejemplo, si la primera solicitud a la red CDN produce un error de caché, la solicitud se envía al origen. Aunque este contenido incompleto se cargue en la memoria caché de la red CDN, las demás solicitudes a la red CDN pueden empezar a obtener estos datos. 


## <a name="media-streaming-optimizations-for-azure-cdn-from-verizon"></a>Optimizaciones de streaming multimedia para Azure CDN de Verizon

Los puntos de conexión **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon** proporcionan streaming de recursos multimedia directamente mediante el uso del tipo de optimización de entrega web general. Algunas características en la red CDN ayudan directamente en la entrega de recursos multimedia de forma predeterminada.

### <a name="partial-cache-sharing"></a>Uso compartido de caché parcial

El uso compartido de caché parcial permite que la red CDN entregue contenido parcialmente almacenado en caché a las nuevas solicitudes. Por ejemplo, si la primera solicitud a la red CDN produce un error de caché, la solicitud se envía al origen. Aunque este contenido incompleto se cargue en la memoria caché de la red CDN, las demás solicitudes a la red CDN pueden empezar a obtener estos datos. 

### <a name="cache-fill-wait-time"></a>Tiempo de espera de relleno de caché

 La característica del tiempo de espera de relleno de caché exige al servidor perimetral mantener todas las solicitudes posteriores para el mismo recurso hasta que los encabezados de respuesta HTTP lleguen desde el servidor de origen. Si los encabezados de respuesta HTTP del servidor de origen llegan antes de que el temporizador expire, todas las solicitudes colocadas en espera se sirven desde la memoria caché creciente. Al mismo tiempo, la memoria caché se rellena con datos del origen. De forma predeterminada, el tiempo de espera de relleno de caché se establece en 3.000 milisegundos. 

 
## <a name="media-streaming-optimizations-for-azure-cdn-from-akamai"></a>Optimizaciones de streaming multimedia para Azure CDN de Akamai
 
**Azure CDN Estándar de Akamai** ofrece una característica que proporciona streaming de recursos multimedia de manera eficaz a usuarios de todo el mundo a escala. La característica reduce las latencias porque reduce la carga en los servidores de origen. Esta característica está disponible con el plan de tarifa estándar de Akamai. 

La optimización del streaming multimedia de **Azure CDN Estándar de Akamai** es eficaz para el streaming multimedia en vivo o de vídeo bajo demanda donde se usan fragmentos multimedia individuales para la entrega. Este proceso es diferente a la transferencia de un solo recurso de gran tamaño a través de descarga progresiva o mediante solicitudes de intervalo de bytes. Para obtener información sobre ese estilo de entrega multimedia, vea [Optimización de archivos grandes](cdn-large-file-optimization.md).

Los tipos de optimización de entrega multimedia general o de vídeo bajo demanda usan una red CDN con optimizaciones de back-end para entregar los recursos multimedia más rápidamente. También usan configuraciones para recursos multimedia en función de procedimientos recomendados aprendidos con el tiempo.

### <a name="configure-an-akamai-cdn-endpoint-to-optimize-media-streaming"></a>Configuración de un punto de conexión de CDN de Akamai para optimizar el streaming multimedia
 
Puede configurar el punto de conexión de la red de entrega de contenido (CDN) para optimizar la entrega de archivos grandes a través de Azure Portal. También puede usar las API de REST o cualquiera de los SDK de cliente para hacer esto. En los pasos siguientes se muestra el proceso a través de Azure Portal para un perfil de **Azure CDN Estándar de Akamai**:

1. Para agregar un nuevo punto de conexión, en la página **Perfil de CDN**, seleccione **Punto de conexión**.
  
    ![Nuevo punto de conexión](./media/cdn-media-streaming-optimization/cdn-new-akamai-endpoint.png)

2. En la lista desplegable **Optimizado para**, seleccione **Streaming multimedia de vídeo a petición** para los recursos de vídeo bajo demanda. Si realiza una combinación de streaming en vivo y bajo demanda, seleccione **Streaming multimedia general**.

    ![Streaming seleccionado](./media/cdn-media-streaming-optimization/02_Creating.png) 
 
Después de crear el punto de conexión, aplica la optimización para todos los archivos que coincidan con determinados criterios. En la sección siguiente se describe este proceso. 

### <a name="caching"></a>Almacenamiento en memoria caché

Si **Azure CDN Estándar de Akamai** detecta que el recurso es un fragmento o manifiesto de streaming, usa tiempos de expiración de almacenamiento en caché diferentes a los de la entrega web general. (Vea la lista completa en la tabla siguiente). Como siempre, se respetan los encabezados Cache-Control o Expires enviados desde el origen. Si el recurso no es un recurso multimedia, almacena en caché mediante los tiempos de expiración para la entrega web general.

El tiempo de almacenamiento en caché negativo corto es útil para la descarga de origen cuando muchos usuarios solicitan un fragmento que no existe todavía. Un ejemplo es una transmisión en directo en la que los paquetes no están disponibles desde el origen en ese segundo. Un intervalo de almacenamiento en caché más largo también ayuda a descargar solicitudes del origen, ya que el contenido de vídeo no suele modificarse.

| Almacenamiento en memoria caché  | Entrega web general | Streaming multimedia general | Streaming multimedia de vídeo a petición  
|--- | --- | --- | ---
| Almacenamiento en caché: Positive <br> HTTP 200, 203, 300, <br> 301, 302 y 410 | 7 días |365 días | 365 días   
| Almacenamiento en caché: Negative <br> HTTP 204, 305, 404, <br> y 405 | None | 1 segundo | 1 segundo
 
### <a name="deal-with-origin-failure"></a>Tratamiento del error de origen  

La entrega multimedia general y la entrega multimedia de vídeo bajo demanda también tienen tiempo de espera de origen y un registro de reintentos basado en los procedimientos recomendados para patrones de solicitud típicos. Por ejemplo, dado que la entrega multimedia general es para la entrega multimedia en vivo y de vídeo bajo demanda, usa un tiempo de espera de conexión más corto debido a la importancia intrínseca del tiempo del streaming en vivo.

Cuando se agota el tiempo de espera de una conexión, la red CDN lo intenta un número de veces antes de enviar al cliente un error de "504 - Tiempo de espera agotado para la puerta de enlace". 

Cuando un archivo cumple la lista de condiciones de tamaño y tipo de archivo, la red CDN usa el comportamiento de streaming multimedia. De lo contrario, se usa la entrega web general.
   
### <a name="conditions-for-media-streaming-optimization"></a>Condiciones para la optimización de streaming multimedia 

En la tabla siguiente se enumera el conjunto de criterios que se deben cumplir para la optimización de streaming multimedia: 
 
Tipos de streaming admitidos | Extensiones de archivo  
--- | ---  
Apple HLS | m3u8, m3u, m3ub, clave, ts y aac
Adobe HDS | f4m, f4x, drmmeta, arranque, f4f,<br>Estructura de dirección URL Seg-Frag <br> (expresión regular de coincidencia: ^(/.*)Seq(\d+)-Frag(\d+))
DASH | mpd, guión, divx, ismv, m4s, m4v, mp4,. mp4v, <br> sidx, webm, mp4a, m4a y isma
Streaming con velocidad de transmisión adaptable | /manifest/, /QualityLevels/Fragments/
  
 
