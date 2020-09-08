---
title: Solución de problemas de conectividad de Azure Virtual Network NAT
titleSuffix: Azure Virtual Network
description: Solucione los problemas de Azure Virtual Network NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
Customer intent: As an IT administrator, I want to troubleshoot Virtual Network NAT.
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/20/2020
ms.author: allensu
ms.openlocfilehash: d75f13f6a0621158bdb9a2f1682d0c85eaacb59d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836112"
---
# <a name="troubleshoot-azure-virtual-network-nat-connectivity"></a>Solución de problemas de conectividad de Azure Virtual Network NAT

En este artículo se ayuda a los administradores a diagnosticar y resolver problemas de conectividad cuando se usa Virtual Network NAT.

## <a name="problems"></a>Problemas

* [Agotamiento de SNAT](#snat-exhaustion)
* [Error en el ping de ICMP](#icmp-ping-is-failing)
* [Errores de conectividad](#connectivity-failures)
* [Coexistencia de IPv6](#ipv6-coexistence)
* [La conexión no parte de las direcciones IP de puerta de enlace NAT](#connection-doesnt-originate-from-nat-gateway-ips)

Para resolver estos problemas, siga los pasos de la siguiente sección.

## <a name="resolution"></a>Solución

### <a name="snat-exhaustion"></a>Agotamiento de la arquitectura de redes de sistemas

Un solo [recurso de puerta de enlace de NAT](nat-gateway-resource.md) admite entre 64 000 y 1 millón de flujos simultáneos.  Cada dirección IP proporciona 64 000 puertos de SNAT al inventario disponible. Se pueden usar un máximo de 16 direcciones IP por recurso de puerta de enlace de NAT.  El mecanismo de la arquitectura de redes de sistemas se describe [aquí](nat-gateway-resource.md#source-network-address-translation) de forma más detallada.

Con frecuencia, la causa principal del agotamiento de SNAT es un antipatrón de cómo se establece y administra la conectividad de salida o cómo se cambian los temporizadores configurables de sus valores predeterminados.  Consulte detenidamente esta sección.

#### <a name="steps"></a>Pasos

1. Compruebe si ha modificado el tiempo de espera de inactividad predeterminado en un valor superior a cuatro minutos.
2. Investigue la forma en que la aplicación crea conectividad saliente (por ejemplo la revisión del código o la captura de paquetes). 
3. Determine si esta actividad es el comportamiento esperado o si la aplicación no se comporta correctamente.  Use [métricas](nat-metrics.md) en Azure Monitor para apoyar sus conclusiones. Use la categoría "Error" para la métrica de las conexiones SNAT.
4. Evalúe si se siguen los patrones adecuados.
5. Evalúe si el agotamiento de puertos de SNAT debe mitigarse con la asignación de más direcciones IP a un recurso de puerta de enlace de NAT.

#### <a name="design-patterns"></a>Patrones de diseño

Aproveche la reutilización de las conexiones y la agrupación de conexiones siempre que sea posible.  Estos patrones evitarán problemas de agotamiento de los recursos y el resultado será un comportamiento predecible. Las primitivas de estos patrones se pueden encontrar en muchas bibliotecas y marcos de desarrollo.

_**Solución:**_ Uso de patrones y procedimientos recomendados apropiados

- Los recursos de puerta de enlace NAT tienen un tiempo de espera de inactividad de TCP predeterminado de cuatro minutos.  Si esta configuración cambia a un valor superior, NAT retendrá los flujos por más tiempo y puede causar una [presión innecesaria en el inventario de puertos SNAT](nat-gateway-resource.md#timers).
- Las solicitudes atómicas (una solicitud por conexión) son una mala opción de diseño. Estos límites de antipatrón escalan, reducen el rendimiento y disminuyen la confiabilidad. En su lugar, reutilice las conexiones HTTP/S para reducir el número de conexiones y los puertos SNAT asociados. El escalado de la aplicación aumentará y mejorará el rendimiento debido a la reducción de los protocolos de enlace, a la sobrecarga y al costo de la operación criptográfica al usar TLS.
- DNS puede introducir muchos flujos individuales en el volumen cuando el cliente no almacena en caché el resultado de la resolución DNS. Use el almacenamiento en caché.
- Los flujos UDP (por ejemplo, las búsquedas de DNS) asignan puertos SNAT mientras dure el tiempo de espera de inactividad. Cuanto mayor sea el tiempo de espera de inactividad, mayor será la presión sobre los puertos SNAT. Use un tiempo de espera de inactividad corto (por ejemplo, 4 minutos).
- Use los grupos de conexiones para dar forma al volumen de la conexión.
- Nunca abandone de forma silenciosa un flujo TCP y confíe en temporizadores TCP para limpiar el flujo. Si no permite que TCP cierre explícitamente la conexión, el estado permanecerá asignado en los sistemas y puntos de conexión intermedios y hará que los puertos de SNAT no estén disponibles para otras conexiones. Este patrón puede desencadenar errores en la aplicación y el agotamiento de SNAT. 
- No cambie los valores del temporizador relacionado con el cierre de TCP de nivel de sistema operativo sin el conocimiento experto de impacto. Aunque la pila de TCP se recuperará, el rendimiento de la aplicación puede verse afectado negativamente si los puntos de conexión de una conexión tienen expectativas no coincidentes. El deseo de cambiar los temporizadores suele ser un signo de un problema de diseño subyacente. Revise las siguientes recomendaciones.

El agotamiento de SNAT también se puede amplificar con otros antipatrones en la aplicación subyacente. Revise estos patrones y procedimientos recomendados adicionales para mejorar la escala y la confiabilidad del servicio.

- Explore el impacto de reducir el [tiempo de espera de inactividad de TCP](nat-gateway-resource.md#timers) a valores inferiores, incluido el tiempo de espera predeterminado de inactividad de cuatro minutos para liberar antes el inventario de puertos SNAT.
- Considere la posibilidad de usar [patrones de sondeo asincrónicos](https://docs.microsoft.com/azure/architecture/patterns/async-request-reply) para las operaciones de ejecución prolongada, con el fin de liberar recursos de conexión para otras operaciones.
- Los flujos de larga duración (como las conexiones TCP reutilizadas) deberían usar paquetes keepalive de TCP o de la capa de la aplicación para evitar que los sistemas intermedios superen el tiempo de espera. Aumentar el tiempo de espera de inactividad es un último recurso y es posible que no resuelva la causa principal. Un tiempo de espera prolongado puede causar errores de baja tasa cuando expira el tiempo de espera e introducir retrasos y errores innecesarios.
- Se deben usar [patrones de reintento](https://docs.microsoft.com/azure/architecture/patterns/retry) correctos para evitar reintentos o ráfagas agresivos durante un error transitorio o la recuperación de un error.
La creación de una conexión TCP para cada operación HTTP (también conocidas como "conexiones atómicas") es un antipatrón.  Las conexiones atómicas evitarán que su aplicación se escale de forma correcta y malgastarán recursos.  Canalice siempre varias operaciones a la misma conexión.  Su aplicación aumentará la velocidad de las transacciones y reducirá los costos de los recursos.  Cuando la aplicación usa un cifrado de la capa de transporte (por ejemplo, TLS), se produce un considerable costo asociado con el procesamiento de nuevas conexiones.  Para conocer otros patrones de procedimientos recomendados, consulte [Patrones de diseño en la nube de Azure](https://docs.microsoft.com/azure/architecture/patterns/).

#### <a name="additional-possible-mitigations"></a>Posibles mitigaciones adicionales

_**Solución:**_ Escale la conectividad saliente como se indica a continuación:

| Escenario | Evidencia |Mitigación |
|---|---|---|
| Percibe una contención de los puertos SNAT y un agotamiento de estos en periodos de mucho uso. | La categoría "Error" para la [métrica](nat-metrics.md) de Conexiones SNAT en Azure Monitor muestra los errores transitorios o persistentes a lo largo del tiempo y un volumen de conexión elevado.  | Determine si puede agregar recursos con direcciones IP públicas o recursos con prefijos IP públicos. Esta incorporación permitirá que haya un máximo de 16 direcciones IP en la puerta de enlace de NAT. También proporcionará más inventario para los puertos SNAT disponibles (64 000 por dirección IP) y le permitirá escalar aún más su escenario.|
| Ya se le han proporcionado 16 direcciones IP y sigue sufriendo un agotamiento de puertos SNAT. | Error al intentar agregar una dirección IP adicional. El número total de direcciones IP de los recursos de dirección IP pública o de prefijo de IP pública supera un total de 16. | Distribuya el entorno de aplicaciones entre varias subredes y proporcione un recurso de puerta de enlace de NAT para cada subred.  Vuelva a evaluar los patrones de diseño para optimizar según la [guía](#design-patterns) anterior. |

>[!NOTE]
>Es importante saber por qué se produce el agotamiento de SNAT. Asegúrese de que usa los patrones correctos para escenarios escalables y confiables.  La incorporación de más puertos SNAT a un escenario sin conocer la causa de la demanda sería el último recurso. Si desconoce los motivos por los que su escenario supone una mayor presión para el inventario de puertos SNAT, la incorporación de más puertos SNAT al inventario mediante la adición de más direcciones IP lo único que hará será retrasar el error de agotamiento cuando la aplicación escale.  Es posible que esté ocultando otras ineficacias y antipatrones.

### <a name="icmp-ping-is-failing"></a>Error en el ping de ICMP

[Virtual Network NAT](nat-overview.md) admite los protocolos UDP y TCP de IPv4. ICMP no se admite, por lo que cabe esperar que se produzca un error.  

_**Solución:**_ En su lugar, use pruebas de conexión TCP (por ejemplo, "ping de TCP") y pruebas de la capa de la aplicación específicas de UDP para asegurar la conectividad de un extremo a otro.

La siguiente tabla se puede usar como punto de partida para saber qué herramientas se deben usar para iniciar las pruebas.

| Sistema operativo | Prueba de conexión TCP genérica | Prueba de la capa de la aplicación de TCP | UDP |
|---|---|---|---|
| Linux | nc (prueba de conexión genérica) | curl (prueba de la capa de la aplicación de TCP) | específica de la aplicación |
| Windows | [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping) | [Invoke-WebRequest](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-webrequest) de PowerShell | específica de la aplicación |

### <a name="connectivity-failures"></a>Errores de conectividad

Los problemas de conectividad con [Virtual Network NAT](nat-overview.md) pueden estar causados por varios motivos:

* problemas permanentes debidos a errores de configuración.
* El [agotamiento de SNAT](#snat-exhaustion) transitorio o persistente de la puerta de enlace NAT
* Errores transitorios en la infraestructura de Azure 
* Errores transitorios en la ruta de acceso entre Azure y el destino público de Internet 
* Errores transitorios o persistentes en el destino de Internet público.

Use herramientas como la siguiente para la validación de la conectividad. [No se admite el ping de ICMP](#icmp-ping-is-failing).

| Sistema operativo | Prueba de conexión TCP genérica | Prueba de la capa de la aplicación de TCP | UDP |
|---|---|---|---|
| Linux | nc (prueba de conexión genérica) | curl (prueba de la capa de la aplicación de TCP) | específica de la aplicación |
| Windows | [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping) | [Invoke-WebRequest](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-webrequest) de PowerShell | específica de la aplicación |

#### <a name="configuration"></a>Configuración

Compruebe la configuración:
1. ¿El recurso de puerta de enlace NAT tiene al menos un recurso de IP pública o un recurso de prefijo de dirección IP pública? Debe tener al menos una dirección IP asociada a la puerta de enlace NAT para que pueda proporcionar conectividad de salida.
2. ¿La subred de la red virtual está configurada para usar la puerta de enlace NAT?
3. ¿Está usando UDR (ruta definida por el usuario) y está reemplazando el destino?  Los recursos de puerta de enlace NAT se convierten en la ruta predeterminada (0/0) en subredes configuradas.

#### <a name="snat-exhaustion"></a>Agotamiento de la arquitectura de redes de sistemas

Revise la sección sobre el [agotamiento de SNAT](#snat-exhaustion) en este artículo.

#### <a name="azure-infrastructure"></a>Infraestructura de Azure

Azure supervisa y opera su infraestructura con gran cuidado. Se pueden producir errores transitorios y no hay ninguna garantía de que las transmisiones sean sin pérdida.  Utilice patrones de diseño que permitan las retransmisiones SYN para las aplicaciones TCP. Use tiempos de espera de conexión lo bastante prolongados como para permitir la retransmisión TCP SYN a fin de reducir los efectos transitorios causados por la pérdida de un paquete SYN.

_**Solución:**_

* Compruebe el [agotamiento de SNAT](#snat-exhaustion).
* El parámetro de configuración de una pila TCP que controla el comportamiento de la retransmisión de SYN se denomina RTO ([tiempo de espera de la retransmisión](https://tools.ietf.org/html/rfc793)). El valor de RTO es ajustable, pero normalmente es de 1 segundo o más, de forma predeterminada, con retroceso exponencial.  Si el tiempo de espera de la conexión de la aplicación es demasiado corto (por ejemplo, 1 segundo), es posible que vea tiempos de espera de conexión esporádicos.  Aumente el tiempo de espera de conexión de la aplicación.
* Si observa tiempos de espera más largos e inesperados con comportamientos de aplicación predeterminados, abra un caso de soporte técnico para solucionar el problema.

No se recomienda reducir artificialmente el tiempo de espera de la conexión TCP ni ajustar el parámetro RTO.

#### <a name="public-internet-transit"></a>Tránsito público por Internet

Las posibilidades de errores transitorios aumentan con una ruta de acceso más larga al destino y más sistemas intermedios. Se espera que los errores transitorios puedan aumentar la frecuencia en la [infraestructura de Azure](#azure-infrastructure). 

Siga las mismas instrucciones que en la sección [Infraestructura de Azure](#azure-infrastructure).

#### <a name="internet-endpoint"></a>Punto de conexión de Internet

Se aplican las secciones anteriores, junto con el punto de conexión de Internet con el que se establece la comunicación. Otros factores que pueden afectar al éxito de la conectividad son:

* Administración del tráfico en el lado de destino, incluido
- Limitación de la tasa de API impuesta por el lado de destino
- Formas de tráfico de la capa de transporte o de mitigación de DDoS volumétricas
* Firewall u otros componentes en el destino 

Normalmente, se necesitan capturas de paquetes en el origen y en el destino (si está disponible) para determinar lo que está teniendo lugar.

_**Solución:**_

* Compruebe el [agotamiento de SNAT](#snat-exhaustion). 
* Valide la conectividad a un punto de conexión en la misma región o en otra parte para la comparación.  
* Si va a crear pruebas de gran volumen o de tasa de transacciones, explore si la reducción de la velocidad reduce la aparición de errores.
* Si la tasa de cambio afecta a la tasa de errores, compruebe si es posible que se haya alcanzado el límite de velocidad de la API u otras restricciones en el lado de destino.
* Si la investigación no es concluyente, abra un caso de soporte técnico para solucionar el problema.

#### <a name="tcp-resets-received"></a>Restablecimientos TCP recibidos

La puerta de enlace NAT genera restablecimientos TCP en la máquina virtual de origen para el tráfico que no se reconoce como en curso.

Una posible razón es que la conexión TCP haya agotado el tiempo de espera.  Puede ajustar el tiempo de espera de inactividad entre 4 minutos y un máximo de 120 minutos.

Los restablecimientos TCP no se generan en el lado público de los recursos de puerta de enlace NAT. Los restablecimientos TCP en el destino los genera la máquina virtual de origen, no el recurso de puerta de enlace NAT.

_**Solución:**_

* Revise las recomendaciones de [patrones de diseño](#design-patterns).  
* Abra un caso de soporte técnico para solucionar el problema si es necesario.

### <a name="ipv6-coexistence"></a>Coexistencia de IPv6

[Virtual Network NAT](nat-overview.md) admite los protocolos UDP y TCP de IPv4, y la implementación en una [subred con un prefijo IPv6 no es compatible](nat-overview.md#limitations).

_**Solución:**_ Implemente la puerta de enlace NAT en una subred sin prefijo IPv6.

Puede indicar el interés en funcionalidades adicionales mediante [UserVoice de Virtual Network NAT](https://aka.ms/natuservoice).

### <a name="connection-doesnt-originate-from-nat-gateway-ips"></a>La conexión no parte de las direcciones IP de puerta de enlace NAT

Configure la puerta de enlace NAT, las direcciones IP que se van a usar y qué subred debería usar un recurso de puerta de enlace NAT. Sin embargo, las conexiones de las instancias de máquina virtual que existían antes de que se implementara la puerta de enlace NAT no usan las direcciones IP.  Parece que utilizan direcciones IP que no se usan con el recurso de puerta de enlace NAT.

_**Solución:**_

[Virtual Network NAT](nat-overview.md) reemplaza la conectividad saliente de la subred en que está configurada. Al realizar la transición desde el SNAT predeterminado o el SNAT de salida del equilibrador de carga al uso de puertas de enlace NAT, las nuevas conexiones comenzarán inmediatamente a usar las direcciones IP asociadas con el recurso de puerta de enlace NAT.  Sin embargo, si una máquina virtual todavía tiene una conexión establecida durante el cambio a un recurso de puerta de enlace NAT, la conexión seguirá usando la dirección IP SNAT antigua que se asignó cuando se estableció la conexión.  Asegúrese de que realmente establece una nueva conexión, en lugar de reutilizar una que ya existía porque el sistema operativo o el explorador estaban almacenando en la caché las conexiones en un grupo de conexiones.  Por ejemplo, al usar _curl_ en PowerShell, asegúrese de especificar el parámetro _-DisableKeepalive_ para forzar una conexión nueva.  Si usa un explorador, es posible que las conexiones estén agrupadas.

No es necesario reiniciar una máquina virtual configurando una subred para un recurso de puerta de enlace NAT.  Sin embargo, si se reinicia una máquina virtual, el estado de la conexión se vacía.  Cuando el estado de la conexión se ha vaciado, todas las conexiones empezarán a usar las direcciones IP del recurso de puerta de enlace NAT.  Sin embargo, esto es un efecto secundario de la máquina virtual que se reinicia, no un indicador de que se requiere un reinicio.

Si sigue teniendo problemas, abra un caso de soporte técnico para solucionar el problema.

### <a name="connection-setup-time"></a>Tiempo de configuración de conexión

Como las reglas de salida de Load Balancer asigna de manera estática grupos de puertos SNAT a máquinas virtuales específicas, la creación de flujos de salida nuevos es más rápida que usar Virtual Network NAT. Por lo tanto, al cambiar de las reglas de salida de Load Balancer, puede que vea una mayor latencia al crear una conexión de salida nueva. Como se explicó anteriormente, para maximizar el rendimiento de la aplicación, debe usar flujos de larga duración (por ejemplo, conexiones TCP reutilizadas).

_**Solución:**_

Si está interesado principalmente en la latencia de configuración de instalación mínima, use las reglas de salida de Load Balancer.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [Virtual Network NAT](nat-overview.md).
* Obtenga información sobre los [recursos de puerta de enlace de NAT](nat-gateway-resource.md)
* Obtenga información acerca de las [métricas y alertas de los recursos de puerta de enlace NAT](nat-metrics.md).
* [Indíquenos qué crear a continuación para Virtual Network NAT en UserVoice](https://aka.ms/natuservoice).

