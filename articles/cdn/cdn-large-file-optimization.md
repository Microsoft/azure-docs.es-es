---
title: Optimización de descarga de archivos grandes con Azure CDN
description: Obtenga información sobre cómo se pueden optimizar las descargas de archivos de gran tamaño en Azure Content Delivery Network. En este artículo se incluyen varios escenarios.
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
ms.openlocfilehash: 6258baf37d00d35da3b7c95519caabdfcaa34b2a
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192649"
---
# <a name="large-file-download-optimization-with-azure-cdn"></a>Optimización de descarga de archivos grandes con Azure CDN

El tamaño de los archivos de contenido enviado por Internet sigue creciendo debido a la mejora de las funcionalidades y los gráficos, junto con el enriquecimiento del contenido multimedia. Este crecimiento se debe a muchos factores: el acceso de banda ancha, los dispositivos de almacenamiento de bajo costo, el aumento generalizado del vídeo de alta definición y los dispositivos conectados a Internet (IoT). Un mecanismo de entrega rápido y eficiente de los archivos grandes es fundamental para garantizar una experiencia sin complicaciones y divertida para el consumidor.

La entrega de archivos grandes tiene varios desafíos. En primer lugar, el tiempo medio para descargar un archivo grande puede ser importante porque es posible que las aplicaciones no descarguen todos los datos de forma secuencial. En algunos casos, es posible que las aplicaciones descarguen la última parte de un archivo antes de la primera. Cuando se solicita solo una pequeña cantidad de un archivo o un usuario pone en pausa una descarga, se puede producir un error en la descarga. También es posible que se retrase la descarga hasta que la red de entrega de contenido (CDN) recupere todo el archivo del servidor de origen. 

En segundo lugar, la latencia entre el equipo del usuario y el archivo determina la velocidad a la que puede ver el contenido. Además, los problemas de capacidad y congestión de la red también afectan al rendimiento. Las mayores distancias entre servidores y usuarios generan más oportunidades de que se produzca la pérdida de paquetes, lo que reduce la calidad. La reducción de la calidad causada por un rendimiento limitado y una mayor pérdida de paquetes es posible que aumente el tiempo de espera para que finalice la descarga de un archivo. 

En tercer lugar, muchos archivos grandes no se envían en su totalidad. Es posible que los usuarios cancelen una descarga a la mitad o que solo vean los primeros minutos de un vídeo MP4 largo. Por tanto, las empresas de entrega de software y de elementos multimedia solo quieren proporcionar la parte del archivo que se solicita. La distribución eficaz de las partes solicitadas reduce el tráfico de salida desde el servidor de origen. La distribución eficaz también reduce la presión de memoria y E/S en el servidor de origen. 


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-from-microsoft"></a>Optimización de la entrega de archivos grandes con Azure CDN de Microsoft

Los puntos de conexión **Azure CDN Estándar de Microsoft** entregan archivos grandes sin límite en el tamaño de archivo. De forma predeterminada, se activan características adicionales para agilizar la entrega de archivos grandes.

### <a name="object-chunking"></a>Fragmentación de objetos 

**Azure CDN Estándar de Microsoft** emplea una técnica llamada fragmentación de objetos. Cuando se solicita un archivo grande, la red CDN recupera partes más pequeñas del archivo del origen. Una vez que el servidor POP de CDN recibe una solicitud de archivo completo o de intervalo de bytes, el servidor perimetral de CDN solicita el archivo al origen en fragmentos de 8 MB. 

Una vez que llega el fragmento al perímetro de la red CDN, se almacena en caché y se sirve inmediatamente al usuario. Después, la red CDN realiza una captura previa del siguiente fragmento en paralelo. Este captura previa garantiza que el contenido sigue estando un fragmento por delante del usuario, lo que reduce la latencia. Este proceso continúa hasta que se descarga todo el archivo (si se solicita), todos los intervalos de bytes están disponibles (si se solicitan) o el cliente finaliza la conexión. 

Para más información sobre la solicitud de intervalo de bytes, vea [RFC 7233](https://tools.ietf.org/html/rfc7233).

La red CDN almacena en caché los fragmentos cuando se reciben. No es necesario almacenar el archivo entero en la caché de CDN. Las solicitudes posteriores para el archivo o los intervalos de bytes se sirven desde la caché de la red CDN. Si no se almacenan en caché todos los fragmentos en la red CDN, se usa la captura previa para solicitar fragmentos del origen. Esta optimización se basa en la capacidad del servidor de origen para admitir solicitudes de intervalo de bytes; si no las admite, esta optimización no es efectiva. 

### <a name="conditions-for-large-file-optimization"></a>Condiciones para la optimización de archivos grandes
Las características de optimización de archivos grandes para **Azure CDN Estándar de Microsoft** están desactivadas de forma predeterminada cuando se usa el tipo de optimización de entrega web general. No hay ningún límite en el tamaño de archivo máximo.


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-from-verizon"></a>Optimización de entrega de archivos grandes con Azure CDN de Verizon

Los puntos de conexión **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon** entregan grandes archivos sin límite en el tamaño de archivo. De forma predeterminada, se activan características adicionales para agilizar la entrega de archivos grandes.

### <a name="complete-cache-fill"></a>Relleno de la memoria caché completa

La característica de relleno de la memoria caché completa predeterminada permite que la red CDN extraiga un archivo en la memoria caché cuando se abandona o se pierde una solicitud inicial. 

El relleno de la memoria caché completa es más útil para los recursos grandes. Normalmente, los usuarios no los descargan de principio a fin. Usan la descarga progresiva. El comportamiento predeterminado fuerza al servidor perimetral para que inicie una captura de fondo del recurso desde el servidor de origen. Después, el recurso está en la caché local del servidor perimetral. Después de que el objeto completo se encuentre en caché, el servidor perimetral acepta las solicitudes de intervalo de bytes en la red CDN para el objeto almacenado en caché.

El comportamiento predeterminado se puede deshabilitar mediante el motor de reglas en **Azure CDN Premium de Verizon**.

### <a name="peer-cache-fill-hot-filing"></a>Creación de archivos activos de llenado de la caché del mismo nivel

La característica de creación de archivos activos de llenado de la caché del mismo nivel usa un sofisticado algoritmo propietario. Usa servidores perimetrales de almacenamiento en caché adicionales que se basan en métricas de solicitudes de ancho de banda y agregadas para responder a solicitudes de cliente de objetos grandes extremadamente populares. Esta característica evita una situación en la que se envía un gran número de solicitudes adicionales al servidor de origen del cliente. 

### <a name="conditions-for-large-file-optimization"></a>Condiciones para la optimización de archivos grandes

Las características de optimización de archivos grandes para **Azure CDN Estándar de Microsoft** y **Azure CDN Premium de Verizon** están desactivadas de forma predeterminada cuando se usa el tipo de optimización de entrega web general. No hay ningún límite en el tamaño de archivo máximo. 


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-standard-from-akamai"></a>Optimización de entrega de archivos grandes con Azure CDN Estándar de Akamai

Los puntos de conexión del perfil **Azure CDN Estándar de Akamai** ofrecen una característica que entrega archivos de gran tamaño de manera eficaz a usuarios de todo el mundo a escala. La característica reduce las latencias porque reduce la carga en los servidores de origen.

La característica de tipo de optimización de archivos grandes activa las optimizaciones y configuraciones de red para entregar archivos grandes de forma más rápida y correcta. La entrega web general con los puntos de conexión **Azure CDN Estándar de Akamai** almacena en caché los archivos solo de menos de 1,8 GB y puede tunelizar (no almacenar en caché) archivos de hasta 150 GB. La optimización de archivos grandes almacena en caché archivos de hasta 150 GB.

La optimización de archivos grandes es efectiva cuando se cumplen determinadas condiciones. Las condiciones incluyen cómo funciona el servidor de origen y los tipos y tamaños de los archivos que se solicitan. 

### <a name="configure-an-akamai-cdn-endpoint-to-optimize-delivery-of-large-files"></a>Configuración de un punto de conexión de CDN de Akamai para optimizar la entrega de archivos grandes

Puede configurar el punto de conexión **Azure CDN Estándar de Akamai** para optimizar la entrega de archivos grandes mediante Azure Portal. También puede usar las API de REST o cualquiera de los SDK de cliente para hacer esto. En los pasos siguientes se muestra el proceso a través de Azure Portal para un perfil de **Azure CDN Estándar de Akamai**:

1. Para agregar un nuevo punto de conexión, en la página **Perfil de CDN**, seleccione **Punto de conexión**.

    ![Nuevo punto de conexión](./media/cdn-large-file-optimization/cdn-new-akamai-endpoint.png)    
 
2. En la lista desplegable **Optimizado para**, seleccione **Descarga de archivos grandes**.

    ![Optimización de archivos grandes seleccionada](./media/cdn-large-file-optimization/cdn-large-file-select.png)


Después de crear el punto de conexión de la red CDN, aplica la optimización de archivos grandes para todos los archivos que coincidan con ciertos criterios. En la sección siguiente se describe este proceso.

### <a name="object-chunking"></a>Fragmentación de objetos 

En la optimización de archivos grandes con **Azure CDN Estándar de Akamai** se emplea una técnica denominada fragmentación de objetos. Cuando se solicita un archivo grande, la red CDN recupera partes más pequeñas del archivo del origen. Una vez que el servidor POP de CDN recibe una solicitud de archivo completo o de intervalo de bytes, comprueba si el tipo de archivo es compatible con esta optimización. También comprueba si el tipo de archivo cumple los requisitos de tamaño de archivo. Si el tamaño del archivo es mayor de 10 MB, el servidor perimetral de la red CDN solicita el archivo del origen en fragmentos de 2 MB. 

Una vez que llega el fragmento al perímetro de la red CDN, se almacena en caché y se sirve inmediatamente al usuario. Después, la red CDN realiza una captura previa del siguiente fragmento en paralelo. Este captura previa garantiza que el contenido sigue estando un fragmento por delante del usuario, lo que reduce la latencia. Este proceso continúa hasta que se descarga todo el archivo (si se solicita), todos los intervalos de bytes están disponibles (si se solicitan) o el cliente finaliza la conexión. 

Para más información sobre la solicitud de intervalo de bytes, vea [RFC 7233](https://tools.ietf.org/html/rfc7233).

La red CDN almacena en caché los fragmentos cuando se reciben. No es necesario almacenar el archivo entero en la caché de CDN. Las solicitudes posteriores para el archivo o los intervalos de bytes se sirven desde la caché de la red CDN. Si no se almacenan en caché todos los fragmentos en la red CDN, se usa la captura previa para solicitar fragmentos del origen. Esta optimización se basa en la capacidad del servidor de origen para admitir solicitudes de intervalo de bytes; si no las admite, esta optimización no es efectiva.

### <a name="caching"></a>Almacenamiento en memoria caché
La optimización de archivos grandes usa tiempos de expiración de almacenamiento en caché predeterminados distintos a los de la entrega web general. Esto marca la diferencia entre el almacenamiento en caché positivo y el negativo, en función de códigos de respuesta HTTP. Si el servidor de origen especifica un tiempo de expiración a través de un encabezado Cache-Control o Expires en la respuesta, la red CDN respeta ese valor. Si no se especifica el origen y el archivo coincide con las condiciones de tipo y tamaño para este tipo de optimización, la red CDN usa los valores predeterminados para la optimización de archivos grandes. En caso contrario, la red CDN usa los valores predeterminados para la entrega web general.

| Almacenamiento en memoria caché  | Web general | Optimización de archivos grandes 
--- | --- | --- 
Almacenamiento en caché: Positive <br> HTTP 200, 203, 300, <br> 301, 302 y 410 | 7 días |1 día  
Almacenamiento en caché: Negative <br> HTTP 204, 305, 404, <br> y 405 | None | 1 segundo 

### <a name="deal-with-origin-failure"></a>Tratamiento del error de origen

La duración del tiempo de espera de lectura de origen aumenta de dos segundos para la entrega web general a dos minutos para el tipo de optimización de archivos grandes. Este aumento contabiliza los tamaños de archivo más grandes para evitar que se agote el tiempo de espera de la conexión de forma prematura.

Cuando se agota el tiempo de espera de una conexión, la red CDN lo intenta un número de veces antes de enviar al cliente un error de "504 - Tiempo de espera agotado para la puerta de enlace". 

### <a name="conditions-for-large-file-optimization"></a>Condiciones para la optimización de archivos grandes

En la tabla siguiente se enumera el conjunto de criterios que se debe cumplir para la optimización de archivos grandes:

Condición | Valores 
--- | --- 
Tipos de archivo admitidos | 3g2, 3gp, asf, avi, bz2, dmg, exe, f4v, flv, <br> gz, hdp, iso, jxr, m4v, mkv, mov, mp4, <br> mpeg, mpg, mts, pkg, qt, rm, swf, tar, <br> tgz, wdp, webm, webp, wma, wmv, zip  
Tamaño de archivo mínimo | 10 MB 
Tamaño de archivo máximo | 150 GB 
Características del servidor de origen | Debe admitir solicitudes de intervalo de bytes 

## <a name="additional-considerations"></a>Consideraciones adicionales

Tenga en cuenta los siguientes aspectos adicionales con este tipo de optimización:

- El proceso de fragmentación genera solicitudes adicionales al servidor de origen. Pero el volumen global de datos que se entregan desde el origen es mucho menor. La fragmentación da como resultado mejores características de almacenamiento en caché en la red CDN.

- La presión de memoria y E/S se reduce en el origen porque se entregan fragmentos más pequeños del archivo.

- Para los fragmentos almacenados en caché en la red CDN, no hay ninguna solicitud adicional al origen hasta que el contenido expire o se elimine de la caché.

- Los usuarios pueden realizar solicitudes de intervalo a la red CDN y se tratan como cualquier archivo normal. La optimización solo se aplica si es un tipo de archivo válido y el intervalo de bytes está comprendido entre 10 MB y 150 GB. Si el tamaño medio de los archivos solicitados es menor que 10 MB, use en su lugar la entrega web general.

