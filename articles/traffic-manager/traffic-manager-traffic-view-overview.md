---
title: Vista de tráfico en Azure Traffic Manager
description: En esta introducción, aprenderá cómo funciona la vista de tráfico de Traffic Manager.
services: traffic-manager
documentationcenter: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: duau
ms.custom: ''
ms.openlocfilehash: f1a3cc5ccb326d6a198895f9f459e661f7805b1e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401544"
---
# <a name="traffic-manager-traffic-view"></a>Traffic View de Traffic Manager

Traffic Manager proporciona enrutamiento en el nivel de DNS para que los usuarios finales sean dirigidos a puntos de conexión en buen estado en función del método de enrutamiento especificado cuando creó el perfil. Vista de tráfico proporciona a Traffic Manager una visión de las bases de usuarios (en un nivel de granularidad de resolución DNS) y su patrón de tráfico. Cuando se habilita Traffic View, esta información se procesa para proporcionarle detalles sobre lo que actuar. 

Mediante el uso de Traffic View, puede:
- comprender dónde se encuentran las bases de usuarios (hasta una granularidad en el nivel de resolución DNS local).
- ver el volumen de tráfico (en forma de consultas DNS controladas por Azure Traffic Manager) que se origina en estas regiones.
- conocer la latencia representativa que experimentan estos usuarios.
- profundizar en los patrones de tráfico específicos de cada una de estas bases de usuarios para las regiones de Azure en la que tiene puntos de conexión. 

Por ejemplo, puede usar Traffic View para conocer qué regiones tienen una gran cantidad de tráfico pero experimentan latencias más altas. A continuación, puede utilizar esta información para planear la expansión de superficie a nuevas regiones de Azure para que estos usuarios puedan tener una experiencia de latencia inferior.

## <a name="how-traffic-view-works"></a>Cómo funciona Traffic View

Traffic View funciona haciendo que Traffic Manager busque las consultas entrantes recibidas en los últimos siete días con un perfil que tenga esta característica habilitada. De esta información de consultas entrantes, Vista de tráfico extrae la dirección IP de origen de la resolución DNS que se usa como una representación de la ubicación de los usuarios. A continuación, se agrupan en una granularidad en el nivel de resolución DNS para crear regiones de bases de usuarios mediante el uso de la información geográfica de direcciones IP mantenida por Traffic Manager. Traffic Manager, a continuación, busca en las regiones de Azure a las que se enrutó la consulta y construye un mapa de flujo de tráfico para los usuarios de esas regiones.  
En el paso siguiente, Traffic Manager correlaciona la región de la base de usuarios con la asignación de región de Azure con las tablas de información de latencia de red que mantiene para diferentes redes de usuario final para conocer la latencia media que experimentan los usuarios de esas regiones cuando se conectan a regiones de Azure. Todos estos cálculos se combinan en el nivel de dirección IP de resolución DNS local antes de presentarse. Puede utilizar la información de varias maneras.

La frecuencia de actualización de datos de la vista Tráfico depende de varias variables de servicio interno. Sin embargo, los datos normalmente se actualizan cada 24 horas.

>[!NOTE]
>La latencia descrita en Vista de tráfico es una latencia representativa entre el usuario final y las regiones de Azure a las que se habían conectado y no es la latencia de búsqueda DNS. Vista de tráfico hace una estimación de mejor esfuerzo de la latencia entre la resolución DNS local y la región de Azure a la que se enrutó la consulta; si no hay suficientes datos disponibles, la latencia devuelta será nula. 

## <a name="visual-overview"></a>Información general visual

Cuando se desplaza a la sección **Vista de tráfico** en la página de Traffic Manager, se le mostrará un mapa geográfico con una superposición de información detallada de Vista de tráfico. El mapa proporciona información acerca de la base de usuarios y los puntos de conexión para el perfil de Traffic Manager.

![Vista geográfica de vista de tráfico de Traffic Manager][1]

### <a name="user-base-information"></a>Información de base de usuarios

Se muestran en el mapa las resoluciones DNS locales para los que está disponible la información de ubicación. El color de las resoluciones DNS muestra la latencia media experimentada por los usuarios finales que han usado esa resolución DNS para las consultas de Traffic Manager.

Si mantiene el mouse sobre una ubicación de la resolución DNS en el mapa, se muestra:
- la dirección IP de la resolución DNS.
- el volumen de tráfico de consultas DNS visto por Traffic Manager desde él.
- los puntos de conexión a los que se enrutó el tráfico desde la resolución DNS, como una línea entre el punto de conexión y la resolución DNS. 
- la latencia media desde esa ubicación al punto de conexión, representada como el color de la línea que los conecta.

### <a name="endpoint-information"></a>Información de punto de conexión

Las regiones de Azure en las que residen los puntos de conexión se muestran como puntos azules en el mapa. Si el punto de conexión es externo y no tiene una región de Azure asignada, se muestra en la parte superior del mapa. Haga clic en cualquier punto de conexión para ver las diferentes ubicaciones (según la resolución DNS usada) desde donde se dirigió el tráfico a ese punto de conexión. Las conexiones se muestran como una línea entre el punto de conexión y la ubicación de la resolución DNS y están coloreadas de acuerdo con la latencia representativa entre ese par. Además, puede ver el nombre del punto de conexión, la región de Azure en la que se ejecuta y el volumen total de solicitudes que se dirigen a él mediante este perfil de Traffic Manager.


## <a name="tabular-listing-and-raw-data-download"></a>Lista tabular y descarga de datos sin procesar

Puede ver los datos de Vista de tráfico en un formato tabular en Azure Portal. Hay una entrada para cada dirección IP de resolución DNS/par de punto de conexión que muestra la dirección IP de la resolución DNS, el nombre y la ubicación geográfica de la región de Azure en el que se encuentra el punto de conexión (si está disponible), el volumen de solicitudes asociadas con esa resolución DNS a ese punto de conexión y la latencia representativa asociada a los usuarios finales con ese DNS (si está disponible). También puede descargar los datos de Vista de tráfico como un archivo CSV que puede usarse como parte de un flujo de trabajo de análisis de su elección.

## <a name="billing"></a>Facturación

Cuando use Traffic View, se le facturará en función del número de puntos de datos utilizados para crear la información de detalle presentada. Actualmente, el único tipo de punto de datos utilizado son las consultas que recibe el perfil de Traffic Manager. Para más detalles sobre los precios, visite la [página de precios de Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/).

## <a name="faqs"></a>Preguntas más frecuentes

* [¿Cómo funciona Traffic View?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-does-traffic-view-do)

* [¿Cuál es el beneficio de usar Traffic View?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-benefit-from-using-traffic-view)

* [¿En qué se diferencia Traffic View de las métricas de Traffic Manager disponibles mediante Azure Monitor?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-is-traffic-view-different-from-the-traffic-manager-metrics-available-through-azure-monitor)

* [¿Usa Traffic View información de subred de cliente de EDNS?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#does-traffic-view-use-edns-client-subnet-information)

* [¿Cuántos días de datos usa Traffic View?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-many-days-of-data-does-traffic-view-use)

* [¿Cómo controla Traffic View los puntos de conexión externos?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-traffic-view-handle-external-endpoints)

* [¿Es necesario habilitar Traffic View en cada perfil de la suscripción?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#do-i-need-to-enable-traffic-view-for-each-profile-in-my-subscription)

* [¿Cómo se desactiva Traffic View?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-turn-off-traffic-view)

* [¿Cómo funciona la facturación de Traffic View?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-traffic-view-billing-work)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda [cómo funciona el Administrador de tráfico](traffic-manager-overview.md)
- Aprenda más sobre los [métodos de enrutamiento de tráfico](traffic-manager-routing-methods.md) que admite el Administrador de tráfico.
- Aprenda a [crear un perfil del Administrador de tráfico](traffic-manager-create-profile.md)

<!--Image references-->
[1]: ./media/traffic-manager-traffic-view-overview/trafficview.png