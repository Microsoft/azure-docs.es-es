---
title: Optimización de la red CDN de Azure para el tipo de entrega de contenido
description: Obtenga información sobre cómo Azure Content Delivery Network puede optimizar la entrega en función del tipo de contenido. Las prácticas recomendadas de optimización mejoran el rendimiento y la descarga de origen.
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
ms.date: 03/25/2019
ms.author: allensu
ms.openlocfilehash: 0710f2f31510ae299fafe89dc1798f40e325e8b4
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192598"
---
# <a name="optimize-azure-cdn-for-the-type-of-content-delivery"></a>Optimización de la red CDN de Azure para el tipo de entrega de contenido

Cuando realice la entrega de contenido a una audiencia global de gran tamaño, es fundamental garantizar la entrega optimizada del contenido. [Azure Content Delivery Network (CDN)](cdn-overview.md) puede optimizar la experiencia de entrega según el tipo de contenido que tenga. El contenido puede ser un sitio web, una transmisión en directo, un vídeo o un archivo grande para descargarlo. Cuando cree un punto de conexión de CDN, especifique un escenario en la opción **Optimizado para**. Su elección determina qué optimización se aplica al contenido que se va a entregar desde el punto de conexión de la red CDN.

Las opciones de optimización están diseñadas para usar comportamientos de procedimientos recomendados para mejorar el rendimiento de la entrega de contenido y un mejor origen de descarga. Las opciones del escenario afectan el rendimiento mediante la modificación de las configuraciones de almacenamiento parcial en caché, la fragmentación de objetos y la directiva de reintentos de error de origen. 

En este artículo se proporciona una introducción a las diversas características de optimización y cuándo se deben usar. Para más información sobre las características y limitaciones, vea los artículos respectivos en cada tipo de optimización individual.

> [!NOTE]
> Cuando se crea un punto de conexión CDN, las opciones de **Optimizado para** pueden variar en función del tipo de perfil en el que se cree. Los proveedores de Azure CDN aplican la mejora de maneras diferentes, según el escenario. 

## <a name="provider-options"></a>Opciones de proveedor

Los perfiles de **Azure CDN Estándar de Microsoft** admiten las siguientes optimizaciones:

* [Entrega web general](#general-web-delivery). Esta optimización se usa también para el streaming multimedia y la descarga de archivos de gran tamaño.

> [!NOTE]
> La aceleración de sitios dinámicos de Microsoft se ofrece a través de [Azure Front Door Service](https://docs.microsoft.com/azure/frontdoor/front-door-overview).

Los perfiles de **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon** admiten las siguientes optimizaciones:

* [Entrega web general](#general-web-delivery). Esta optimización se usa también para el streaming multimedia y la descarga de archivos de gran tamaño.

* [Aceleración de sitios dinámicos](#dynamic-site-acceleration) 


Los perfiles de **Azure CDN Estándar de Akamai** admiten las siguientes optimizaciones:

* [Entrega web general](#general-web-delivery) 

* [Streaming multimedia general](#general-media-streaming)

* [Streaming multimedia de vídeo a petición](#video-on-demand-media-streaming)

* [Descarga de archivos de gran tamaño](#large-file-download)

* [Aceleración de sitios dinámicos](#dynamic-site-acceleration) 

Microsoft recomienda que pruebe las variaciones en el rendimiento entre diferentes proveedores para seleccionar el proveedor óptimo para su entrega.

## <a name="select-and-configure-optimization-types"></a>Seleccionar y configurar los tipos de optimización

Para crear un punto de conexión de CDN, seleccione el tipo de optimización que mejor coincida con el escenario y el tipo de contenido que quiere que el punto de conexión entregue. **Entrega web general** es la selección predeterminada. Para los puntos de conexión existentes de **Azure CDN Estándar de Akamai**, puede actualizar la opción de optimización en cualquier momento. Este cambio no interrumpe la entrega desde Azure CDN. 

1. Dentro de un perfil de **Azure CDN Estándar de Akamai**, seleccione un punto de conexión.

    ![Selección de punto de conexión](./media/cdn-optimization-overview/01_Akamai.png)

2. En CONFIGURACIÓN, seleccione **Optimización**. Después, seleccione un tipo en la lista desplegable **Optimizado para**.

    ![Selección de la optimización y el tipo.](./media/cdn-optimization-overview/02_Select.png)

## <a name="optimization-for-specific-scenarios"></a>Optimización de escenarios específicos

Puede optimizar el punto de conexión de la red CDN para uno de estos escenarios. 

### <a name="general-web-delivery"></a>Entrega web general

Entrega web general es la opción de optimización más común. Está diseñada para la optimización de contenido web general, como páginas web y aplicaciones web. Esta optimización puede usarse también para la descarga de archivos y vídeos.

Un sitio web típico incluye contenido estático y dinámico. El contenido estático incluye las imágenes, las bibliotecas de JavaScript y las hojas de estilos, que pueden almacenarse en la memoria caché y se entregan a distintos usuarios. El contenido dinámico es personalizado para un usuario específico, como elementos de noticias adaptados a un perfil de usuario. El contenido dinámico, como el contenido de los carros de la compra, no se almacena en la memoria caché porque es único para cada usuario. La entrega web general puede optimizar todo el sitio web. 

> [!NOTE]
> Si usa un perfil de **Azure CDN Estándar de Akamai**, seleccione este tipo de optimización cuando el tamaño medio de los archivos sea inferior a 10 MB. De lo contrario, si el tamaño promedio de los archivos es superior a 10 MB, seleccione **Descarga de archivos grandes** en la lista desplegable **Optimizado para**.

### <a name="general-media-streaming"></a>Streaming multimedia general

Si tiene que usar el punto de conexión para streaming en vivo y de vídeo bajo demanda, seleccione el tipo de optimización de streaming multimedia general.

El streaming multimedia depende del tiempo, porque los paquetes que llegan tarde al cliente, como el almacenamiento en búfer frecuente del contenido de vídeo, pueden provocar una experiencia de visualización deteriorada. La optimización del streaming multimedia reduce la latencia de entrega de contenido multimedia y proporciona una experiencia de streaming con velocidad de transmisión adaptable para los usuarios. 

Es un escenario común para los clientes de Azure Media Services. Cuando usa Azure Media Services, obtiene un solo punto de conexión de streaming que puede usarse para streaming en vivo y bajo demanda. Con este escenario, los clientes no tienen que cambiar a otro punto de conexión cuando pasan de streaming en vivo a streaming bajo demanda. La optimización de streaming multimedia general admite este tipo de escenario.

Para **Azure CDN Estándar de Microsoft**, **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**, use el tipo de optimización de entrega web general para entregar streaming de contenido multimedia general.

Para más información acerca de la optimización del streaming multimedia, consulte [Optimización del streaming multimedia](cdn-media-streaming-optimization.md).

### <a name="video-on-demand-media-streaming"></a>Streaming multimedia de vídeo a petición

La optimización del streaming multimedia de vídeo a petición mejora el contenido que se transmite a petición. Use esta opción, si usa un punto de conexión para el streaming de vídeo bajo demanda.

Para los perfiles de **Azure CDN Estándar de Microsoft**, **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**, use el tipo de optimización de entrega web general para entregar streaming de contenido multimedia de vídeo bajo demanda.

Para más información acerca de la optimización del streaming multimedia, consulte [Optimización del streaming multimedia](cdn-media-streaming-optimization.md).

> [!NOTE]
> Si el punto de conexión de CDN sirve principalmente contenido de vídeo bajo demanda, use este tipo de optimización. La principal diferencia entre esta optimización y la de streaming de multimedia general es el tiempo de espera del reintento de conexión. El tiempo de expiración es mucho más breve para trabajar con escenarios de streaming en vivo.
>

### <a name="large-file-download"></a>Descarga de archivos de gran tamaño

Con los perfiles de **Azure CDN Estándar de Akamai**, se optimizan las descargas de archivos de gran tamaño de contenido de más de 10 MB. Si el tamaño medio de los archivos es menor que 10 MB, use la entrega web general. Si los tamaños promedios de los archivos son constantemente superiores a 10 MB, es posible que sea más eficaz crear un punto de conexión independiente para los archivos grandes. Por ejemplo, las actualizaciones de firmware o software normalmente son archivos grandes. Para entregar archivos de más de 1,8 GB, se requiere la optimización de descarga de archivos grandes.

Para los perfiles de **Azure CDN Estándar de Microsoft**, **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**, use el tipo de optimización de entrega web general para entregar contenido de descarga de archivos de gran tamaño. No hay ninguna limitación en el tamaño de la descarga de archivos.

Para más información acerca de la optimización de archivos grandes, consulte [Optimización de archivos grandes](cdn-large-file-optimization.md).

### <a name="dynamic-site-acceleration"></a>Aceleración de sitios dinámicos

 La aceleración de sitios dinámicos (DSA) está disponible para los perfiles de **Azure CDN Estándar de Akamai**, **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**. El uso de esta optimización implica el pago de una cuota adicional; para más información, consulte [Precios de Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

> [!NOTE]
> La aceleración de sitios dinámicos de Microsoft se ofrece a través de [Azure Front Door Service](https://docs.microsoft.com/azure/frontdoor/front-door-overview), que es un servicio global de [difusión por proximidad](https://en.wikipedia.org/wiki/Anycast) que aprovecha la red privada de global de Microsoft para distribuir sus cargas de trabajo de la aplicación.

DSA incluye diversas técnicas que benefician a la latencia y el rendimiento del contenido dinámico. Las técnicas incluyen la optimización de rutas y redes, la optimización de TCP y mucho más. 

Esta optimización se puede usar para acelerar una aplicación web que incluye muchas respuestas que no se pueden almacenar en caché. Algunos ejemplos son los resultados de búsquedas, transacciones de finalización de la compra o datos en tiempo real. Puede seguir usando las funcionalidades de almacenamiento en caché principales de Azure CDN para los datos estáticos. 

Para más información acerca de la aceleración de sitios dinámicos, consulte [Aceleración de sitios dinámicos](cdn-dynamic-site-acceleration.md).



