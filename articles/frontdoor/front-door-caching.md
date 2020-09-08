---
title: 'Azure Front Door: almacenamiento en caché | Microsoft Docs'
description: Este artículo le ayuda a comprender cómo Azure Front Door supervisa el estado de los servidores back-end.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 1cb24e4a959e7d32a3c3b5b69a39938df4efddfa
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399878"
---
# <a name="caching-with-azure-front-door"></a>Almacenamiento en caché con Azure Front Door
En el documento siguiente se especifica el comportamiento de Front Door con reglas de enrutamiento que han habilitado el almacenamiento en caché. Front Door es una red Content Delivery Network (CDN) moderna, junto con la aceleración de sitios dinámicos y el equilibrio de carga; también admite comportamientos de almacenamiento en caché como cualquier otra red CDN.

## <a name="delivery-of-large-files"></a>Suministro de archivos grandes
Azure Front Door proporciona archivos grandes sin un límite en el tamaño de los archivos. Front Door usa una técnica denominada fragmentación de objetos. Cuando se solicita un archivo grande, Front Door recupera partes más pequeñas del back-end. Después de recibir una solicitud de un archivo completo o intervalo de bytes, un entorno de Front Door solicita el archivo desde el back-end en fragmentos de 8 MB.

</br>Una vez que el fragmento llega al entorno de Front Door, se almacena en caché y se sirve inmediatamente al usuario. Después, Front Door realiza una captura previa del siguiente fragmento en paralelo. Este captura previa garantiza que el contenido sigue estando un fragmento por delante del usuario, lo que reduce la latencia. Este proceso continúa hasta que se descarga todo el archivo (si se solicita), todos los intervalos de bytes están disponibles (si se solicitan) o el cliente finaliza la conexión.

</br>Para más información sobre la solicitud de intervalo de bytes, vea [RFC 7233](https://web.archive.org/web/20171009165003/http://www.rfc-base.org/rfc-7233.html).
Front Door almacena en caché los fragmentos cuando se reciben, por lo que no es necesario poner todo el archivo en la caché de Front Door. Las solicitudes posteriores para el archivo o los intervalos de bytes se sirven desde la caché. Si no se almacenan en caché todos los fragmentos, se usa la captura previa para solicitar fragmentos del back-end. Esta optimización se basa en la capacidad del servidor de origen para admitir solicitudes de intervalo de bytes; si no las admite, esta optimización no es efectiva.

## <a name="file-compression"></a>Compresión de archivos
Front Door puede comprimir dinámicamente el contenido en el borde, lo que genera una respuesta más pequeña y rápida a sus clientes. Todos los archivos son aptos para la compresión. Sin embargo, un archivo debe tener un tipo MIME que sea apto para la lista de compresión. Actualmente, Front Door no permite cambiar esta lista. La lista actual es:</br>
- "application/eot"
- "application/font"
- "application/font-sfnt"
- "application/javascript"
- "application/json"
- "application/opentype"
- "application/otf"
- "application/pkcs7-mime"
- "application/truetype"
- "application/ttf",
- "application/vnd.ms-fontobject"
- "application/xhtml+xml"
- "application/xml"
- "application/xml+rss"
- "application/x-font-opentype"
- "application/x-font-truetype"
- "application/x-font-THF"
- "application/x-httpd-cgi"
- "application/x-mpegurl"
- "application/x-opentype"
- "application/x-otf"
- "application/x-perl"
- "application/x-THF"
- "application/x-javascript"
- "font/eot"
- "font/ttf"
- "font/otf"
- "font/opentype"
- "image/svg+xml"
- "text/css"
- "text/csv"
- "text/html"
- "text/javascript"
- "text/js", "text/plain"
- "text/richtext"
- "text/tab-separated-values"
- "text/xml"
- "text/x-script"
- "text/x-component"
- "text/x-java-source"

Además, el tamaño del archivo debe estar entre 1 KB y 8 MB, ambos inclusive.

Estos perfiles admiten las codificaciones de compresión siguientes:
- [Gzip (GNU zip)](https://en.wikipedia.org/wiki/Gzip)
- [Brotli](https://en.wikipedia.org/wiki/Brotli)

Si una solicitud admite la compresión gzip y Brotli, la compresión Brotli tiene prioridad.</br>
Cuando una solicitud de un recurso especifica la compresión gzip y la solicitud genera un error de caché, Front Door realiza la compresión del recurso directamente en el servidor POP. Después, el archivo comprimido se envía desde la caché. El elemento resultante se devuelve con transfer-encoding: chunked.

## <a name="query-string-behavior"></a>Comportamiento de las cadenas de consulta
Front Door permite controlar cómo se almacenan los archivos en caché para una solicitud web que contiene una cadena de consulta. En una solicitud web con una cadena de consulta, esta última es la parte de la solicitud que hay después del signo de interrogación (?). Una cadena de consulta puede contener uno o más pares clave-valor, en los cuales el nombre de campo y su valor están separados por un signo igual (=). Los pares clave-valor están separados entre ellos por una Y comercial (&). Por ejemplo, `http://www.contoso.com/content.mov?field1=value1&field2=value2`. Si hay más de un par clave-valor en una cadena de consulta de una solicitud, no importa el orden en el que se especifiquen.
- **Pasar por alto las cadenas de consulta**: modo predeterminado. En este modo, Front Door pasa las cadenas de consulta del solicitante al back-end en la primera solicitud y almacena en la memoria caché el recurso. Todas las solicitudes posteriores del recurso que se ofrecen desde el entorno Front Door omiten las cadenas de consulta hasta que expira el recurso en caché.

- **Almacenar en caché todas las URL únicas**: en este modo, cada solicitud con un URL único, incluida la cadena de consulta, se trata como un recurso único con su propia memoria caché. Por ejemplo, la respuesta desde el back-end a una solicitud de `www.example.ashx?q=test1` se almacena en caché en el entorno Front Door y se devuelve en los sucesivos almacenamientos en caché con la misma cadena de consulta. Se almacena en caché una solicitud de `www.example.ashx?q=test2` como un recurso independiente con su propia configuración de período de vida.

## <a name="cache-purge"></a>Purga de la memoria caché
Front Door almacenará los recursos en caché hasta que el período de vida de dichos recursos (TTL) expire. Tras la expiración del TTL del recurso, cuando un cliente solicite el recurso, el entorno de Front Door recuperará una nueva copia actualizada del recurso para atender la solicitud del cliente y almacenar actualizada la memoria caché.
</br>El procedimiento recomendado para asegurarse de que los usuarios siempre obtengan la copia más reciente de los recursos es crear versiones correspondientes a cada actualización y publicarlos como nuevas URL. Front Door recuperará inmediatamente los nuevos recursos en las siguientes solicitudes de los clientes. A veces puede que quiera purgar contenido almacenado en caché de todos los nodos perimetrales y forzarlos todos para recuperar nuevos activos actualizados. Esto puede deberse a actualizaciones de la aplicación web o a actualizaciones rápidas de los activos de actualización que contienen información incorrecta.

</br>Seleccione los activos que quiera purgar de los nodos perimetrales. Si quiere borrar todos los recursos, haga clic en la casilla Purgar todo. De lo contrario, escriba la ruta de acceso completa de cada recurso que quiera purgar en el cuadro de texto Ruta de acceso. Los siguientes formatos se pueden usar en las rutas de acceso.
1. **Purga de ruta de acceso única** Purgue recursos concretos mediante la especificación de la ruta completa del activo (sin el protocolo y dominio), con la extensión de archivo; por ejemplo, /pictures/strasbourg.png;
2. **Purga con carácter comodín**: se puede usar el asterisco (\*) como carácter comodín. Purgue todas las carpetas, subcarpetas y archivos de un punto de conexión con /\* en la ruta de acceso o todas las subcarpetas y archivos de una carpeta concreta especificando la carpeta seguido de /\*; por ejemplo, /pictures/\*.
3. **Purga de dominio raíz**: purgue la raíz del punto de conexión con "/" en la ruta de acceso.

La purga de la memoria caché en Front Door distingue mayúsculas de minúsculas. Además, es independiente de la cadena de consulta, lo que significa que purgar una dirección URL purgará todas sus variaciones de la cadena de consulta. 

## <a name="cache-expiration"></a>Expiración de la caché
El siguiente orden de encabezados se usa para determinar cuánto tiempo se almacenará un elemento en la memoria caché:</br>
1. Cache-Control: s-maxage=\<seconds>
2. Cache-Control: max-age=\<seconds>
3. Expira: \<http-date>

Los encabezados de respuesta Cache-Control que indican que la respuesta no se almacena en caché como Cache-Control: private, Cache-Control: no-cache y Cache-Control: no-store se respetan. Sin embargo, si hay varias solicitudes en lucha en un servidor POP por la misma dirección URL, es posible que compartan la respuesta. Si no hay ningún control de caché presente, el comportamiento predeterminado es que AFD almacenará en caché el recurso durante un tiempo X, donde X se elige aleatoriamente entre 1 y 3 días.

## <a name="request-headers"></a>Encabezados de solicitud

Los siguientes encabezados de solicitud no se reenviarán a un back-end cuando se use el almacenamiento en caché.
- Content-Length
- Transfer-Encoding

## <a name="cache-duration"></a>Duración de la caché

La duración de la caché se puede configurar tanto en el diseñador de Front Door como en el motor de reglas. La duración de la caché establecida en el diseñador de Front Door es la duración mínima de la caché. Esta invalidación no funcionará si el encabezado de control de la caché del origen tiene un valor de TTL mayor que el valor de invalidación. 

La duración de la caché establecida mediante el motor de reglas es una invalidación de caché verdadera, lo que significa que usará el valor de invalidación con independencia del encabezado de respuesta de origen.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
