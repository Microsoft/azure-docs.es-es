---
title: 'Azure ExpressRoute: Diseño de la recuperación ante desastres'
description: Esta página proporciona recomendaciones de arquitectura para la recuperación ante desastres al usar Azure ExpressRoute.
services: expressroute
author: rambk
ms.service: expressroute
ms.topic: article
ms.date: 05/25/2019
ms.author: rambala
ms.openlocfilehash: 8adfb0ef0d9aa79d1b14127453f76223f035d62a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081175"
---
# <a name="designing-for-disaster-recovery-with-expressroute-private-peering"></a>Diseño para la recuperación ante desastres con el emparejamiento privado de ExpressRoute

ExpressRoute está diseñado para ofrecer una alta disponibilidad y así poder proporcionar conectividad de red privada a nivel de operador a los recursos de Microsoft. En otras palabras, no hay un único punto de error en la ruta de ExpressRoute dentro de la red de Microsoft. En cuanto a las opciones de diseño para maximizar la disponibilidad de un circuito de ExpressRoute, consulte [Designing for high availability with ExpressRoute][HA] (Diseño de para alta disponibilidad con ExpressRoute).

Sin embargo, teniendo en cuenta la popular ley de Murphy: *si algo puede salir mal, saldrá mal*, en este artículo nos centraremos en ofrecer soluciones que vayan más allá de aquellos errores que pueden solucionarse con un solo circuito de ExpressRoute. En otras palabras, en este artículo analizaremos las opciones que ofrece la arquitectura de red para crear una sólida conectividad de red de back-end para la recuperación ante desastres; para ello, usaremos circuitos de ExpressRoute con redundancia geográfica.

>[!NOTE]
>Los conceptos descritos en este artículo se aplican de la misma forma cuando se crea un circuito ExpressRoute en una red WAN virtual o fuera de él.
>

## <a name="need-for-redundant-connectivity-solution"></a>Necesidad de una solución de conectividad redundante

Existen posibilidades e instancias donde un servicio regional completo (ya sea de Microsoft, proveedores de servicios de red, clientes u otros proveedores de servicios en la nube) se degrada. La causa fundamental de esta degradación del servicio a nivel regional incluye los desastres naturales. Por lo tanto, para asegurar la continuidad del negocio y las aplicaciones críticas, es importante planificar la recuperación ante desastres.   

Independientemente de si ejecuta sus aplicaciones críticas en una región de Azure, en una región local o en cualquier otro lugar, puede usar otra región de Azure como sitio de conmutación por error. En los siguientes artículos se explica la recuperación ante desastres desde las aplicaciones y las perspectivas de acceso de front-end:

- [Recuperación ante desastres de escala empresarial][Enterprise DR]
- [Recuperación ante desastres SMB con Azure Site Recovery][SMB DR]

Si confía en la conectividad de ExpressRoute entre su red local y Microsoft para realizar operaciones críticas, su plan de recuperación ante desastres también debe incluir conectividad de red con redundancia geográfica. 

## <a name="challenges-of-using-multiple-expressroute-circuits"></a>Desafíos de usar varios circuitos de ExpressRoute

Cuando interconecta el mismo conjunto de redes con más de una conexión, debe introducir rutas de acceso paralelas entre las redes. Las rutas de acceso paralelas, cuando no están correctamente diseñadas, podrían conducir a un enrutamiento asimétrico. Si tiene entidades con estado (por ejemplo, NAT, firewall) en la ruta de acceso, el enrutamiento asimétrico podría bloquear el flujo de tráfico.  Por lo general, si usa la ruta de emparejamiento privado de ExpressRoute no se encontrará con entidades con estado como NAT o los firewall. Por lo tanto, el enrutamiento asimétrico sobre el emparejamiento privado de ExpressRoute no bloquea necesariamente el flujo de tráfico.
 
Sin embargo, si equilibra la carga del tráfico a través de rutas paralelas con redundancia geográfica, independientemente de si tiene entidades con estado o no, experimentará un rendimiento de red incoherente. En este artículo discutiremos cómo abordar estos desafíos.

## <a name="small-to-medium-on-premises-network-considerations"></a>Opciones de la red local pequeña a mediana

Usaremos el ejemplo de red ilustrado en el siguiente diagrama. En este ejemplo, la conectividad de ExpressRoute con redundancia geográfica se establece entre la ubicación local de Contoso y la red virtual de Contoso en una región de Azure. En el diagrama, la línea verde continua indica la ruta de acceso preferida (a través de ExpressRoute 1) y la línea de puntos representa la ruta de acceso en espera (a través de ExpressRoute 2).

[![1]][1]

Cuando diseñe la conectividad de ExpressRoute para la recuperación ante desastres, debe tener en cuenta lo siguiente:

- debe usar circuitos de ExpressRoute con redundancia geográfica.
- debe usar diversas redes de proveedores de servicios para diferentes circuitos de ExpressRoute.
- debe diseñar cada uno de los circuitos de ExpressRoute para la [alta disponibilidad][HA].
- debe terminar el circuito de ExpressRoute diferente en una ubicación diferente que se encuentre en la red del cliente.

De manera predeterminada, si anuncia rutas de forma idéntica en todas las rutas de ExpressRoute, Azure equilibrará la carga del tráfico enlazado local en todas las rutas de ExpressRoute mediante el enrutamiento de varias rutas de igual costo (ECMP).

Sin embargo, con los circuitos de ExpressRoute con redundancia geográfica debemos tener en cuenta los diferentes rendimientos de la red que cuenta con diferentes rutas de acceso de red (especialmente para la latencia de la red). Para obtener un rendimiento de red más consistente durante una operación normal, puede usar el circuito de ExpressRoute que ofrece la latencia mínima.

Asimismo, puede influir en Azure para que use un circuito de ExpressRoute en vez de otro, mediante una de las siguientes técnicas (enumeradas en el orden de efectividad):

- indicar una ruta de acceso más específica a través del circuito de ExpressRoute que prefiera, en comparación con otros circuitos de ExpressRoute.
- configurar el peso más alto en aquella conexión que conecta la red virtual con el circuito de ExpressRoute preferido.
- indicar las rutas a través del circuito de ExpressRoute que menos prefiera con una ruta de acceso AS más larga (anteponiendo la ruta de acceso de AS).

### <a name="more-specific-route"></a>Una ruta más especifica

En el siguiente diagrama se indica cómo influir en la selección de rutas de acceso de ExpressRoute con un anuncio de ruta más específico. En el ejemplo ilustrado, el rango de IP local /24 de Contoso se indica como rangos de dos direcciones /25 a través de la ruta de acceso preferida (ExpressRoute 1) y como /24 a través de la ruta de acceso en espera (ExpressRoute 2).

[![2]][2]

Como /25 es más específico en comparación con /24, Azure enviará el tráfico destinado a 10.1.11.0/24 a través de ExpressRoute 1 en un estado normal. Si ambas conexiones de ExpressRoute 1 se interrumpen, la red virtual verá el anuncio de la ruta 10.1.11.0/24 solo a través de ExpressRoute 2; por lo tanto, el circuito de espera se usará en este estado de error.

### <a name="connection-weight"></a>Peso de la conexión

En la siguiente captura de pantalla se indica la configuración del peso de una conexión de ExpressRoute a través de Azure Portal.

[![3]][3]

En el siguiente diagrama se ilustra cómo influir en la selección de la ruta de acceso de ExpressRoute con el peso de la conexión. El peso de conexión predeterminado es 0. En el siguiente ejemplo, el peso de la conexión para ExpressRoute 1 se configura como 100. Cuando una red virtual recibe un prefijo de ruta anunciado a través de más de un circuito de ExpressRoute, la red virtual usará la conexión con el mayor peso.

[![4]][4]

Si ambas conexiones de ExpressRoute 1 se interrumpen, la red virtual verá el anuncio de la ruta 10.1.11.0/24 solo a través de ExpressRoute 2; por lo tanto, el circuito de espera se usará en este estado de error.

### <a name="as-path-prepend"></a>Anteponer la ruta de acceso AS

En el siguiente diagrama se ilustra cómo influir en la selección de la ruta de acceso de ExpressRoute anteponiendo la ruta de acceso AS. En el diagrama, el anuncio de ruta sobre ExpressRoute 1 indica el comportamiento predeterminado de eBGP. En el anuncio de ruta sobre ExpressRoute 2, el ASN de la red local se antepone adicionalmente a la ruta de acceso AS de la ruta en cuestión. Cuando se recibe la misma ruta a través de varios circuitos de ExpressRoute, según el proceso de selección de ruta de eBGP, la red virtual usará la ruta con la ruta de acceso AS más corta. 

[![5]][5]

Si las conexiones de ExpressRoute 1 se interrumpen, entonces la red virtual verá el anuncio de ruta 10.1.11.0/24 solo a través de ExpressRoute 2. Por consiguiente, la ruta de acceso de AS más larga se vuelve irrelevante. Así pues, el circuito de espera se usa en este estado de error.

Al usar cualquiera de esas técnicas, si influye en Azure para que prefiera una de sus rutas de ExpressRoute sobre otras, también debe asegurarse de que la red local también prefiera la misma ruta de ExpressRoute para el tráfico con destino a Azure y así evitar flujos asimétricos. Normalmente, el valor de preferencia local se usa para influir en la red local y que así esta prefiera un circuito de ExpressRoute sobre otros. La preferencia local es una métrica interna de BGP (iBGP). Se prefiere la ruta de BGP que tenga el valor de preferencia local más alto.

> [!IMPORTANT]
> Cuando use ciertos circuitos de ExpressRoute como modo de espera, debe administrarlos activamente y probar periódicamente la operación de conmutación por error. 
> 

## <a name="large-distributed-enterprise-network"></a>Red de empresa distribuida de gran tamaño

Cuando tiene una red de empresa distribuida de gran tamaño, es probable que tenga varios circuitos de ExpressRoute. En esta sección veremos cómo diseñar la recuperación ante desastres mediante los circuitos de ExpressRoute activo-activo, sin necesidad de usar circuitos adicionales en espera. 

Usaremos el ejemplo ilustrado en el siguiente diagrama. En el ejemplo, Contoso tiene dos ubicaciones locales conectadas a dos implementaciones de IaaS de Contoso en dos regiones de Azure diferentes a través de circuitos ExpressRoute y que están en dos ubicaciones de emparejamiento diferentes. 

[![6]][6]

La forma en que diseñemos la recuperación ante desastres afectará a la forma en que se enruta el tráfico de una región a otra (región1/región2 a ubicación2/ubicación1). Debemos tener en cuenta dos arquitecturas diferentes de desastres que enruten el tráfico de ubicación de región de manera diferente.

### <a name="scenario-1"></a>Escenario 1

En el primer escenario, diseñaremos la recuperación ante desastres de manera que todo el tráfico entre una región de Azure y la red local fluya a través del circuito de ExpressRoute local que tiene un estado estable. Si el circuito local de ExpressRoute produce un error, entonces el circuito remoto de ExpressRoute se usará para todos los flujos de tráfico entre Azure y la red local.

El escenario 1 se ilustra en el diagrama siguiente. En el diagrama, las líneas verdes indican las rutas de acceso del flujo de tráfico entre VNet1 y las redes locales. Las líneas azules indican las rutas de acceso del flujo de tráfico entre VNet2 y las redes locales. Las líneas continuas indican la ruta de acceso que se quiere usar y que tiene un estado estable; las líneas discontinuas, por su parte, indican la ruta de acceso del tráfico en el error del circuito de ExpressRoute correspondiente que transporta el flujo de tráfico en estado estable. 

[![7]][7]

Puede diseñar el escenario usando el peso de la conexión para influir en las redes virtuales para que prefieran la conexión a la ubicación de emparejamiento local de ExpressRoute para el tráfico enlazado de red local. Para completar la solución, debe garantizar un flujo de tráfico inverso simétrico. Puede usar la preferencia local en la sesión de iBGP entre sus enrutadores de BGP (en los cuales los circuitos de ExpressRoute finalizan en el lado del entorno local) para elegir un circuito de ExpressRoute. La solución se ilustra en el siguiente diagrama. 

[![8]][8]

### <a name="scenario-2"></a>Escenario 2

El escenario 2 se ilustra en el diagrama siguiente. En el diagrama, las líneas verdes indican las rutas de acceso del flujo de tráfico entre VNet1 y las redes locales. Las líneas azules indican las rutas de acceso del flujo de tráfico entre VNet2 y las redes locales. En el estado estable (líneas continuas en el diagrama), todo el tráfico entre redes virtuales y ubicaciones locales fluye a través de la red troncal de Microsoft en su mayor parte; además, también fluye a través de la interconexión entre ubicaciones locales solo en el estado de error (líneas punteadas en el diagrama) de una ruta de ExpressRoute.

[![9]][9]

La solución se ilustra en el siguiente diagrama. Tal como se ilustra, puede diseñar el escenario con una ruta más específica (Opción 1) o con la anteposición de la ruta de acceso AS (Opción 2) para influir en la selección de la ruta de acceso de la red virtual. Para influir en la selección de la ruta de la red local para el tráfico enlazado a Azure, debe configurar la interconexión entre la ubicación local como la menos preferible. Sin embargo, la configuración del vínculo de interconexión depende del protocolo de enrutamiento utilizado en la red local. Puede usar la preferencia local con iBGP o la métrica con IGP (OSPF o IS-IS).

[![10]][10]


## <a name="next-steps"></a>Pasos siguientes

En este artículo discutiremos cómo diseñar la recuperación ante desastres de una conectividad de emparejamiento privado de un circuito de ExpressRoute. En los siguientes artículos se explica la recuperación ante desastres desde las aplicaciones y las perspectivas de acceso de front-end:

- [Recuperación ante desastres de escala empresarial][Enterprise DR]
- [Recuperación ante desastres SMB con Azure Site Recovery][SMB DR]

<!--Image References-->
[1]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/one-region.png "opciones de la red local de tamaño pequeño a mediano"
[2 ]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/specificroute.png "influir en la selección de la ruta de acceso usando rutas más específicas"
[3]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/configure-weight.png "configurar el peso de la conexión a través de Azure Portal"
[4]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/connectionweight.png "influir en la selección de la ruta de acceso con el peso de la conexión"
[5]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/aspath.png "influir en la selección de la ruta de acceso anteponiendo la ruta de acceso AS "
[6]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region.png "opciones de red local distribuida de gran tamaño"
[7]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-arch1.png "escenario 1"
[8]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-sol1.png "solución de circuitos de ExpressRoute activo-activo 1"
[9]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-arch2.png "escenario 2"
[10]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-sol2.png "solución de circuitos ExpressRoute activo-activo 2"

<!--Link References-->
[HA]: https://docs.microsoft.com/azure/expressroute/designing-for-high-availability-with-expressroute
[Enterprise DR]: https://azure.microsoft.com/solutions/architecture/disaster-recovery-enterprise-scale-dr/
[SMB DR]: https://azure.microsoft.com/solutions/architecture/disaster-recovery-smb-azure-site-recovery/
[con wgt]: https://docs.microsoft.com/azure/expressroute/expressroute-optimize-routing#solution-assign-a-high-weight-to-local-connection
[AS Path Pre]: https://docs.microsoft.com/azure/expressroute/expressroute-optimize-routing#solution-use-as-path-prepending





