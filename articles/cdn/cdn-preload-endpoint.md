---
title: Carga previa de recursos en un punto de conexión de Azure CDN | Microsoft Docs
description: Aprenda a cargar previamente contenido almacenado en caché en un punto de conexión de Azure Content Delivery Network. Esta característica está disponible en determinadas versiones del producto.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: 5ea3eba5-1335-413e-9af3-3918ce608a83
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/12/2018
ms.author: allensu
ms.openlocfilehash: 2f0968af5cb52904f6044e130adba8b0ba9a6cd1
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192579"
---
# <a name="pre-load-assets-on-an-azure-cdn-endpoint"></a>Carga previa de activos en un punto de conexión de Azure CDN
[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

De forma predeterminada, los recursos se almacenan en caché solo al solicitarse. Dado que los servidores perimetrales aún no tienen el contenido almacenado en caché y deben reenviar la solicitud al servidor de origen, la primera solicitud de cada región puede tardar más tiempo que las solicitudes posteriores. Para evitar la latencia de primera visita, cargue sus recursos previamente. Además de proporcionar una mejor experiencia de cliente, la precarga de los recursos almacenados en caché puede reducir el tráfico de red en el servidor de origen.

> [!NOTE]
> La precarga de recursos es útil para grandes eventos o contenido que se pone a disposición de muchos usuarios simultáneamente, como el lanzamiento de una nueva película o una actualización de software.
> 
> 

Este tutorial le guiará a través de la precarga de contenido almacenado en la caché en todos los nodos perimetrales de Azure CDN.

## <a name="to-pre-load-assets"></a>Para cargar recursos previamente
1. En [Azure Portal](https://portal.azure.com), examine el perfil de CDN que contiene el punto de conexión que quiere precargar. Se abre el panel del perfil.
    
2. Haga clic en el punto de conexión de la lista. Se abre el panel del punto de conexión.
3. En el panel del punto de conexión de CDN, seleccione **Cargar**.
   
    ![Panel del punto de conexión de CDN](./media/cdn-preload-endpoint/cdn-endpoint-blade.png)
   
    Se abre el panel **Cargar**.
   
    ![Panel Carga de CDN](./media/cdn-preload-endpoint/cdn-load-blade.png)
4. En **Ruta de acceso de contenido**, escriba la ruta de acceso completa que quiera cargar (por ejemplo, `/pictures/kitten.png`).
   
   > [!TIP]
   > Después de empezar a escribir texto, aparecerán más cuadros de texto de **Ruta de acceso de contenido** para permitirle crear una lista de varios recursos. Para eliminar recursos de la lista, seleccione el botón de puntos suspensivos (...) y, a continuación, seleccione **Eliminar**.
   > 
   > Cada ruta de acceso de contenido debe ser una dirección URL relativa que se ajuste a las siguientes [expresiones regulares](/dotnet/standard/base-types/regular-expression-language-quick-reference):  
   > - Carga de una sola ruta de acceso del archivo: `^(?:\/[a-zA-Z0-9-_.%=\u0020]+)+$`  
   > - Carga de un único archivo con cadena de consulta: `^(?:\?[-_a-zA-Z0-9\/%:;=!,.\+'&\u0020]*)?$` 
   > 
   > Dado que cada recurso debe tener su propia ruta de acceso, no hay ninguna funcionalidad de comodín para recursos de carga previa.
   > 
   > 
   
    ![Botón Cargar](./media/cdn-preload-endpoint/cdn-load-paths.png)
5. Cuando haya terminado de escribir rutas de acceso de contenido, seleccione **Cargar**.
   

> [!NOTE]
> Hay un límite de 10 solicitudes de carga por minuto por perfil de CDN y 50 rutas de acceso simultáneas se pueden procesar al mismo tiempo. Cada ruta de acceso tiene un límite de longitud de 1024 caracteres.
> 
> 

## <a name="see-also"></a>Consulte también
* [Purgar un punto de conexión de Azure CDN](cdn-purge-endpoint.md)
* [Referencia de API REST de Azure CDN: cargar previamente contenido en un punto de conexión](https://docs.microsoft.com/rest/api/cdn/endpoints/loadcontent)
* [Referencia de API REST de Azure CDN: purgar contenido desde un punto de conexión](https://docs.microsoft.com/rest/api/cdn/endpoints/purgecontent)

