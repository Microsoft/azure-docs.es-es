---
title: Solución de problemas de disponibilidad de front-end y back-end y de mantenimiento de recursos de Azure Load Balancer
description: Use las métricas disponibles para hacer un diagnóstico de una instancia de Azure Standard Load Balancer que se ha degradado o no está disponible.
services: load-balancer
documentationcenter: na
author: erichrt
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/14/2020
ms.author: errobin
ms.openlocfilehash: 1af3ce7125d30ed0cb9b8ca6b3cb9322dc14c520
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855247"
---
# <a name="troubleshoot-resource-health-frontend-and-backend-availability-issues"></a>Solución de problemas de disponibilidad de front-end y back-end y de mantenimiento de recursos 

Este artículo sirve de guía para investigar los problemas que afectan a la disponibilidad de la IP de front-end y los recursos de back-end del equilibrador de carga. 

## <a name="about-the-metrics-well-use"></a>Acerca de las métricas que se van a usar
Las dos métricas que se van a usar son la *disponibilidad de la ruta de acceso de datos* y el *estado de sondeo de mantenimiento*, y es importante comprender su significado para obtener información correcta. 

## <a name="data-path-availability"></a>Disponibilidad de la ruta de acceso de datos
La métrica de disponibilidad de la ruta de acceso de datos se genera mediante un ping de TCP cada 25 segundos en todos los puertos de front-end que tienen configuradas reglas NAT de entrada y de equilibrio de carga. Después, este ping de TCP se enrutará a cualquiera de las instancias de back-end correctas (comprobadas). Si el servicio recibe una respuesta al ping, se considera un éxito y la suma de la métrica se iterará una vez, en caso de que se produzca un error, no se iterará. El recuento de esta métrica es 1/100 del total de pings de TCP por período de muestra. Por lo tanto, debemos considerar el promedio, que mostrará el promedio de suma/recuento para el período de tiempo. La métrica de disponibilidad de la ruta de acceso de datos agregada por promedio nos ofrece un porcentaje de tasa de éxito para los pings de TCP en la dirección IP:puerto de front-end para cada una de las reglas NAT de entrada y de equilibrio de carga.

## <a name="health-probe-status"></a>Estado del sondeo de mantenimiento
La métrica de estado del sondeo de mantenimiento se genera mediante un ping del protocolo definido en el sondeo de estado. Este ping se envía a cada instancia del grupo de back-end y del puerto definido en el sondeo de estado. En el caso de sondeos HTTP y HTTPS, un ping correcto requiere una respuesta HTTP 200 Correcto, mientras que en los sondeos TCP, cualquier respuesta se considera correcta. Los errores o éxitos consecutivos de cada sondeo determinan si una instancia de back-end es correcta y puede recibir tráfico para las reglas de equilibrio de carga a las que el grupo de back-end está asignado. De manera similar al caso de la disponibilidad de la ruta de acceso de datos, usamos la agregación de promedio, que nos indica el promedio de pings correctos y totales durante el intervalo de muestreo. Este valor del estado de sondeo de mantenimiento indica el estado del back-end en aislamiento del equilibrador de carga, mediante el sondeo de las instancias de back-end sin enviar tráfico a través del front-end.

>[!IMPORTANT]
>El estado de sondeo de mantenimiento se muestra cada minuto. Esto puede dar lugar a fluctuaciones menores en un valor que, en otro caso, sería constante. Por ejemplo, si hay dos instancias de back-end, una sondeada como correcta y otra como incorrecta, el servicio de sondeo de estado puede capturar 7 muestras para la instancia correcta y 6 para la instancia incorrecta. Esto hará que un valor 50, previamente constante, se muestre como 46,15 para un intervalo de un minuto. 

## <a name="diagnose-degraded-and-unavailable-load-balancers"></a>Diagnóstico de equilibradores de carga degradados y no disponibles
Como se describe en el [artículo sobre el mantenimiento de los recursos](load-balancer-standard-diagnostics.md#resource-health-status), un equilibrador de carga degradado es aquel que muestra entre el 25 % y el 90 % de disponibilidad de la ruta de acceso de datos, y un equilibrador de carga no disponible es el que tiene menos del 25 % de disponibilidad de la ruta de acceso de datos, durante un período de dos minutos. Se pueden seguir estos mismos pasos para investigar los errores observados en cualquier estado de sondeo de mantenimiento o alertas de disponibilidad de ruta de acceso de datos que haya configurado. Exploraremos el caso en el que comprobamos el mantenimiento de los recursos y encontramos que el equilibrador de carga tiene una disponibilidad de la ruta de acceso de datos del 0 %; es decir, nuestro servicio está inactivo.

En primer lugar, vamos a la vista de métricas detalladas de nuestra hoja de información del equilibrador de carga. Puede hacerlo a través de la hoja de recursos del equilibrador de carga o el vínculo del mensaje de mantenimiento de los recursos.  Después, vaya a la pestaña de disponibilidad de front-end y back-end y revise una ventana de treinta minutos del período de tiempo en el que se produjo el estado de degradación o de no disponibilidad. Si vemos que la disponibilidad de la ruta de acceso a los datos ha sido del 0 %, sabemos que hay un problema que impide el tráfico para todas nuestras reglas NAT de entrada y de equilibrio de carga, y se puede ver cuánto tiempo ha durado este impacto. 

Lo siguiente que necesitamos mirar es nuestra métrica de estado de sondeo de mantenimiento para determinar si la ruta de acceso a los datos no está disponible porque no tenemos instancias de back-end correctas para atender el tráfico. Si tenemos al menos una instancia de back-end correcta para todas nuestras reglas de equilibrio de carga y de entrada, sabemos que no es nuestra configuración la causa de que las rutas de acceso de datos no estén disponibles. Este escenario puede indicar un problema de la plataforma Azure, aunque es poco frecuente, pero no debe preocuparse, ya que se envía una alerta automatizada a nuestro equipo para resolver rápidamente todos los problemas de la plataforma.

## <a name="diagnose-health-probe-failures"></a>Diagnóstico de errores de sondeo de estado
Supongamos que comprobamos el estado de sondeo de mantenimiento y encontramos que todas las instancias aparecen como incorrectas. Esto explica por qué nuestra ruta de acceso a datos no está disponible, ya que el tráfico no tiene ninguna instancia a la que ir. Debemos recorrer la siguiente lista de comprobación para descartar los errores de configuración comunes:
* Compruebe el uso de CPU de los recursos para comprobar si las instancias son realmente correctas.
  * Puede comprobar esto. 
* Si se usa una comprobación de sondeo HTTP o HTTPS, si la aplicación es correcta y responde.
  * Valide si es funcional accediendo directamente a las aplicaciones a través de la dirección IP privada o la dirección IP pública de nivel de instancia asociada a la instancia de back-end.
* Revise los grupos de seguridad de red que se aplican a nuestros recursos de back-end. Asegúrese de que no hay ninguna regla con una prioridad mayor que AllowAzureLoadBalancerInBound, lo que bloquearía el sondeo de estado.
  * Para ello, visite la hoja de redes de las máquinas virtuales de back-end o Virtual Machine Scale Sets
  * Si existe este problema de NSG, mueva la regla de permiso existente o cree una nueva regla de prioridad alta para permitir el tráfico de AzureLoadBalancer.
* Compruebe el sistema operativo. Asegúrese de que las máquinas virtuales están escuchando en el puerto de sondeo y revise las reglas de firewall del sistema operativo para asegurarse de que no estén bloqueando el tráfico de sondeo procedente de la dirección IP 168.63.129.16.
  * Puede comprobar los puertos de escucha ejecutando netstat -a en el símbolo del sistema de Windows o netstat-l en un terminal de Linux.
* No coloque una máquina virtual NVA de firewall en el grupo de back-end del equilibrador de carga, use [rutas definidas por el usuario](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#user-defined) para enrutar el tráfico a las instancias de back-end a través del firewall.
* Asegúrese de que usa el protocolo correcto, si usa HTTP para sondear un puerto que escucha una aplicación que no es HTTP, se producirá un error en el sondeo.

Si ha recorrido esta lista de comprobación y todavía se producen errores de sondeo de estado, puede haber problemas de plataforma poco frecuentes que afecten al servicio de sondeo para las instancias. En este caso, Azure le respaldará. Se envía una alerta automatizada a nuestro equipo para resolver rápidamente todos los problemas de la plataforma.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los sondeos de estado de Azure Load Balancer](load-balancer-custom-probe-overview.md)
* [Más información sobre las métricas de Azure Load Balancer](load-balancer-standard-diagnostics.md)


