---
title: ¿Qué es una red de entrega de contenido (CDN)? -Azure | Microsoft Docs
description: Obtenga información acerca de Azure Content Delivery Network (CDN) y de su uso para entregar contenido con alto ancho de banda.
services: cdn
documentationcenter: ''
author: duongau
manager: danielgi
editor: ''
ms.assetid: 866e0c30-1f33-43a5-91f0-d22f033b16c6
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 05/09/2018
ms.author: duau
ms.custom: mvc
ms.openlocfilehash: 44ab6822039ef2a26ffafc2824bed249825e2daa
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783822"
---
# <a name="what-is-a-content-delivery-network-on-azure"></a>¿Qué es una red de entrega de contenido en Azure?
Una red de entrega de contenido (CDN) es una red distribuida de servidores que puede proporcionar contenido web a los usuarios de manera eficaz. Las redes CDN guardan el contenido almacenado en caché de los servidores perimetrales en ubicaciones de punto de presencia (POP) que están cerca de los usuarios finales, para minimizar la latencia. 

Azure Content Delivery Network (CDN) ofrece a los desarrolladores una solución global para la entrega rápida de contenido con alto ancho de banda a los usuarios mediante el almacenamiento en caché del contenido en nodos físicos estratégicamente situados en todo el mundo. Azure CDN también puede acelerar el contenido dinámico, que no puede almacenarse en caché, aprovechando para ello varias optimizaciones de red mediante ubicaciones POP de CDN. Por ejemplo, la optimización de rutas para omitir el protocolo de puerta de enlace de borde (BGP).

Entre las ventajas de usar Azure CDN para entregar los recursos de un sitio web se incluyen:

* Mejor rendimiento y experiencia para los usuarios finales, sobre todo a la hora de utilizar aplicaciones donde son necesarios varios recorridos de ida y vuelta para cargar el contenido.
* Gran escalado para mejorar la administración de cargas instantáneas pesadas, por ejemplo, al comienzo de un evento de lanzamiento de un producto.
* Distribución de las solicitudes de usuario y entrega del contenido directamente desde los servidores perimetrales, de forma que se envía menos tráfico al servidor de origen.

Para ver una lista de las ubicaciones actuales de los nodos de CDN, consulte [Ubicaciones POP de Azure CDN](cdn-pop-locations.md).

## <a name="how-it-works"></a>Funcionamiento
![Información general de la red CDN](./media/cdn-overview/cdn-overview.png)

1. Un usuario (Alice) solicita un archivo (también llamado un recurso) mediante una dirección URL con un nombre de dominio especial, como _&lt;nombre del punto de conexión&gt;_ .azureedge.net. Este nombre puede ser un nombre de host del punto de conexión o un dominio personalizado. El sistema DNS enruta la solicitud hasta la ubicación POP con mejor rendimiento, que normalmente es el punto POP más cercano geográficamente al usuario.
    
2. Si los servidores perimetrales del POP no tienen el archivo en la memoria caché, el punto POP solicita el archivo al servidor de origen. El servidor de origen puede ser una aplicación web de Azure, un servicio en la nube de Azure, una cuenta de Azure Storage o cualquier servidor web accesible públicamente.
   
3. El servidor de origen devuelve el archivo a un servidor perimetral del punto POP.
    
4. Un servidor perimetral del punto POP almacena el archivo en caché y devuelve el archivo al solicitante original (Alice). El archivo permanece en la memoria caché en el servidor perimetral del punto POP hasta que expira el período de vida (TTL) especificado por sus encabezados HTTP. Si el servidor de origen no ha especificado ningún TTL, el valor predeterminado es de siete días.
    
5. Otros usuarios pueden ahora solicitar el mismo archivo utilizando la misma dirección URL que Alicia y también se pueden dirigir al mismo POP.
    
6. Si no ha expirado el período de vida del archivo, el servidor perimetral del punto POP devolverá el archivo directamente desde la memoria caché. Este proceso da lugar a una experiencia de usuario más rápida y una mayor capacidad de respuesta.

## <a name="requirements"></a>Requisitos
Para usar Azure CDN, debe poseer al menos una suscripción de Azure. También debe crear al menos un perfil de CDN, que es una colección de puntos de conexión de CDN. Cada punto de conexión de CDN representa una configuración específica del comportamiento de la entrega de contenido y su acceso. Puede usar varios perfiles para organizar sus puntos de conexión de la red CDN por dominio de Internet, aplicación web o cualquier otro criterio. Dado que los [precios de Azure CDN](https://azure.microsoft.com/pricing/details/cdn/) se aplican en el nivel de perfil de CDN, debe crear varios perfiles de CDN si desea utilizar una combinación de los planes de tarifa. Para obtener información sobre la estructura de la facturación de Azure CDN, vea [Descripción de la facturación de Azure CDN](cdn-billing.md).

### <a name="limitations"></a>Limitaciones
Cada suscripción de Azure tiene límites predeterminados para los siguientes recursos:
 - El número de perfiles de CDN que se pueden crear.
 - El número de puntos de conexión que pueden crearse en un perfil de CDN. 
 - El número de dominios personalizados que pueden asignarse a un punto de conexión.

Para más información acerca de los límites de la suscripción de Azure CDN, consulte [Límites de CDN](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits).
    
## <a name="azure-cdn-features"></a>Características de Azure CDN
Azure CDN ofrece las siguientes características principales:

- [Aceleración de sitios dinámicos](cdn-dynamic-site-acceleration.md)
- [Reglas de almacenamiento en caché de CDN](cdn-caching-rules.md)
- [Compatibilidad con dominios personalizados de HTTPS](cdn-custom-ssl.md)
- [Registros de Azure Diagnostics](cdn-azure-diagnostic-logs.md)
- [Compresión de archivos](cdn-improve-performance.md)
- [Filtrado geográfico](cdn-restrict-access-by-country.md)

Para obtener una lista completa de características que admite cada producto de Azure CDN, consulte [Comparación de características de los productos de Azure CDN](cdn-features.md).

## <a name="next-steps"></a>Pasos siguientes
- Para empezar a trabajar con CDN, consulte [Creación de un perfil y un punto de conexión de Azure CDN](cdn-create-new-endpoint.md).
- Puede administrar los puntos de conexión de CDN con [Microsoft Azure Portal](https://portal.azure.com) o [PowerShell](cdn-manage-powershell.md).
- Aprenda a automatizar Azure CDN con [.NET](cdn-app-dev-net.md) o [Node.js](cdn-app-dev-node.md).
- Para ver Azure CDN en acción, tiene a su disposición los [vídeos de Azure CDN](https://azure.microsoft.com/resources/videos/index/?services=cdn&sort=newest).
- Para obtener información sobre las características más recientes de Azure CDN, vea el [blog de Azure CDN](https://azure.microsoft.com/blog/tag/azure-cdn/).
