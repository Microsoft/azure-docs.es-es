---
title: 'Azure Front Door: métodos de enrutamiento del tráfico | Microsoft Docs'
description: Este artículo le ayudará a entender los distintos métodos de enrutamiento de tráfico usados por Front Door
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: d12eb67abbc216afb241fa6c5a9ef9c66e65040c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399317"
---
# <a name="front-door-routing-methods"></a>Métodos de enrutamiento de Front Door

Azure Front Door admite varios métodos de enrutamiento del tráfico para determinar cómo enrutar el tráfico HTTP/HTTPS a los diversos puntos de conexión de servicio. Con cada una de las solicitudes de cliente que lleguen a Front Door, se aplica el método de enrutamiento configurado para asegurarse de que las solicitudes se reenvían a la mejor instancia de back-end. 

Hay cuatro conceptos básicos para el enrutamiento del tráfico disponible a Front Door:

* **[Latencia](#latency):** el enrutamiento basado en la latencia garantiza que las solicitudes se envían a los back-ends con menor latencia aceptables dentro de un intervalo de confidencialidad. Básicamente, las solicitudes de usuario se envían al conjunto de servidores back-end "más cercanos" con respecto a la latencia de red.
* **[Prioridad](#priority):** puede asignar prioridades a los diferentes back-end cuando quiera usar un back-end del servicio principal para todo el tráfico y proporcionar copias de seguridad en caso de que el back-end principal o de copia de seguridad no estén disponibles.
* **[Ponderado](#weighted):** puede asignar pesos a sus diferentes back-end cuando desee distribuir el tráfico entre un conjunto de back-end, ya sea de manera uniforme o según los coeficientes de peso.
* **[Afinidad de sesión](#affinity)**: puede configurar la afinidad de sesión para los hosts de front-end o dominios cuando desee que las solicitudes posteriores de un usuario se envíen al mismo back-end siempre y cuando la sesión del usuario esté todavía activa y la instancia de back-end aparezca como correcta en los sondeos de estado de mantenimiento. 

Todas las configuraciones de Front Door incluyen la supervisión del estado de mantenimiento de los servidores back-end y la conmutación por error global instantánea automatizada. Para más información, consulte [Front Door Backend Monitoring](front-door-health-probes.md) (Supervisión de servidores back-end en Front Door). Front Door se puede configurar para trabajar en función de un método de enrutamiento único y, dependiendo de las necesidades de su aplicación, puede usar varios o todos estos métodos de enrutamientos combinados para crear una topología de enrutamiento óptima.

## <a name="lowest-latencies-based-traffic-routing"></a><a name = "latency"></a>Enrutamiento del tráfico basado en la latencia más baja

La capacidad de respuesta de muchas aplicaciones se puede mejorar con la implementación de servidores back-end en dos o más ubicaciones del planeta y el enrutamiento del tráfico a la ubicación más "cercana" a los usuarios finales. El método de enrutamiento de tráfico predeterminado para la configuración de Front Door reenvía las solicitudes de los usuarios finales al servidor back-end más cercano al entorno de Front Door que recibió la solicitud. En combinación con la arquitectura Anycast de Azure Front Door, este enfoque garantiza que cada uno de los usuarios finales obtendrá el máximo rendimiento personalizado según su ubicación.

El punto de conexión "más cercano" no es necesariamente el más cercano en cuanto a la distancia geográfica. En su lugar, Front Door mide la latencia de red para determinar cuál es el servidor back-end más cercano. Obtenga más información sobre la [arquitectura de enrutamiento de Front Door](front-door-routing-architecture.md). 

El siguiente es el flujo de decisión general:

| Servidores back-end disponibles | Priority | Señal de latencia (basada en sondeo de estado) | Pesos |
|-------------| ----------- | ----------- | ----------- |
| En primer lugar, seleccione todos los servidores back-end que estén habilitados y devuelvan un estado correcto (200 OK) en el sondeo de estado de mantenimiento. Supongamos que hay seis servidores back-end, A, B, C, D, E y F y, entre ellos, C está en mal estado y E está deshabilitado. Por lo tanto, la lista de servidores back-end disponible es A, B, D y F.  | Después, se seleccionan los servidores back-end de mayor prioridad entre los que estén disponibles. Por ejemplo, los servidores back-end A, B y D tienen prioridad 1 y el servidor back-end F tiene prioridad 2. Por lo tanto, los servidores back-end seleccionados serán A, B y D.| Seleccione los servidores back-end con un intervalo de latencia (menos latencia menor sensibilidad a la latencia, expresada en milisegundos). Por ejemplo, si A está a 15 ms, B a 30 ms y D a 60 ms del entorno de Front Door al que llegó la solicitud, y sensibilidad a la latencia es 30 ms, el grupo de menor latencia está formado por los servidores back-end A y B, puesto que D está a más de 30 ms del servidor back-end más cercano, que es A. | Por último, Front Door realizará una distribución round robin del tráfico entre los servidores back-end del grupo seleccionado, en la proporción especificada por su peso. Por ejemplo, si un servidor back-end tiene un peso de 5 y el servidor back-end B tiene un peso de 8, el tráfico se distribuirá en la proporción 5:8 entre el los servidores back-end A y B. |

>[!NOTE]
> De forma predeterminada, la propiedad de sensibilidad a la latencia se establece en 0 ms; es decir, la solicitud siempre se reenvía al servidor back-end más rápido disponible.


## <a name="priority-based-traffic-routing"></a><a name = "priority"></a>Enrutamiento de tráfico basado en la prioridad

Habitualmente, las organizaciones desean ofrecer confiabilidad en sus servicios y, para ello, implementan uno o varios servicios de reserva en caso de que su servicio principal se vuelva inactivo. En la industria, esta topología también se conoce como topología de implementación de activo/pasivo o activo/en espera. El método de enrutamiento de tráfico de "Prioridad" permite que los clientes de Azure implementen fácilmente este patrón de conmutación por error.

La instancia de Front Door predeterminada contiene una lista de servidores back-end con la misma prioridad. De forma predeterminada, Front Door envía tráfico solo a los servidores back-end de prioridad superior (valor de prioridad más bajo); es decir, el conjunto principal de servidores back-end. Si los servidores back-end principales no están disponibles, Front Door enruta el tráfico al conjunto secundario de servidores back-end (segundo valor de prioridad más bajo). Si los servidores back-end principal y secundario no están disponibles, el tráfico pasa al tercero, y así sucesivamente. Disponibilidad de los servidores back-end se basa en el estado configurado (habilitado o deshabilitado) y en el estado de mantenimiento actual del servidor back-end determinado por los sondeos de estado.

### <a name="configuring-priority-for-backends"></a>Configuración de la prioridad de los servidores back-end

Cada uno de los servidores back-end del grupo dentro de la configuración de Front Door tiene una propiedad llamada "Priority", que puede ser un número entre 1 y 5. Con Azure Front Door, la prioridad de cada servidor de back-end se configura explícitamente con esta propiedad. Esta propiedad es un valor comprendido entre 1 y 5. Los valores más bajos representan una prioridad más alta. Los servidores back-ends pueden compartir los valores de prioridad.

## <a name="weighted-traffic-routing-method"></a><a name = "weighted"></a>Método de enrutamiento de tráfico Ponderado
El método de enrutamiento de tráfico "Ponderado" le permite distribuir el tráfico uniformemente o utilizar una ponderación predefinida.

En el método de enrutamiento de tráfico ponderado, se asigna un peso a cada servidor back-end en la configuración del grupo de servidores back-end en Front Door. El peso es un entero comprendido entre 1 y 1000. Este parámetro utiliza un peso predeterminado de "50".

Entre la lista de servidores back-end disponibles que están dentro de la sensibilidad a la latencia aceptado (tal y como se especifica), el tráfico se distribuye con un mecanismo round robin en la proporción especificada por el peso. Si la sensibilidad a la latencia se establece en 0 milisegundos, esta propiedad no surtirá efecto a menos que haya dos servidores back-end con la misma latencia de red. 

El método ponderado permite algunos escenarios útiles:

* **Actualización gradual de aplicaciones**: asigne un porcentaje del tráfico que se redirigirá a un nuevo servidor back-end y aumente el tráfico gradualmente hasta que esté a la par con otros servidores back-end.
* **Migración de aplicaciones a Azure**: cree un perfil con servidores back-end de Azure y externos. Ajuste el peso de los servidores back-end para dar preferencia a los nuevos servidores back-end. Para configurar esto gradualmente, se pueden deshabilitar los nuevos servidores back-end y, a continuación, se les asigna el menor peso para ir aumentándolo lentamente a niveles en los que tomarán la mayor parte del tráfico. Por último, se deshabilitan los servidores back-end menos preferidos y se quitan del grupo.  
* **Expansión de la nube para conseguir capacidad adicional**: expanda rápidamente una implementación local en la nube colocándola detrás de Front Door. Cuando necesite capacidad adicional en la nube, puede agregar o habilitar más servidores back-end y especificar la porción de tráfico que va a cada uno.

## <a name="session-affinity"></a><a name = "affinity"></a>Afinidad de sesión
De forma predeterminada, cuando no hay afinidad de sesión, Front Door reenvía las solicitudes procedentes del mismo cliente a distintos servidores de back-end según la configuración del equilibrio de carga, especialmente cuando cambian las latencias de los servidores de back-end o si las diferentes solicitudes del mismo usuario llegan a un entorno de Front Door diferente. Sin embargo, algunas aplicaciones con estado o en algunos otros escenarios, prefiere que las solicitudes posteriores del mismo usuario lleguen al mismo servidor back-end que procesó la solicitud inicial. La característica de afinidad de sesión basada en cookies es útil cuando se quiere mantener una sesión de usuario en el mismo servidor back-end. Con las cookies administradas por Front Door, Azure Front Door puede dirigir el tráfico posterior de una sesión de usuario al mismo back-end para su procesamiento, siempre que este esté en buen estado y la sesión de usuario no haya expirado. 

La afinidad de sesión puede habilitarse en el nivel de host de front-end para cada uno de los dominios configurados (o subdominios). Una vez habilitada, Front Door agrega una cookie a la sesión del usuario. La afinidad de sesión basada en cookies permite a Front Door identificar usuarios diferentes, aunque estén detrás de la misma dirección IP lo que, a su vez, permite realizar una distribución más uniforme del tráfico entre los diferentes servidores back-end.

La duración de la cookie es la misma que la sesión del usuario, porque Front Door actualmente solo admite cookies de sesión. 

> [!NOTE]
> Los servidores proxy públicos pueden interferir con la afinidad de sesión. El motivo es que, para establecer una sesión, Front Door tiene que agregar una cookie de afinidad de sesión a la respuesta, lo que no se puede realizar si la respuesta se puede almacenar en caché porque interrumpiría las cookies de otros clientes que soliciten el mismo recurso. Para evitar este problema, la afinidad de sesión **no** se establecerá si, al intentarlo, el servidor back-end envía una respuesta almacenable en caché. Si la sesión ya se ha establecido, no importa si la respuesta del back-end se puede almacenar en caché.
> La afinidad de sesión se establecerá en las siguientes circunstancias, **a menos que** la respuesta tenga un código de estado HTTP 304:
> - La respuesta tiene establecidos valores específicos para el encabezado ```Cache-Control```, que impiden el almacenamiento en caché, por ejemplo, "private" o "no-store".
> - La respuesta contiene un encabezado ```Authorization``` que no ha expirado.
> - La respuesta tiene un código de estado HTTP 302.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
