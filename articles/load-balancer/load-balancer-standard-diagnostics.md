---
title: Diagnóstico de Azure Standard Load Balancer con métricas, alertas y estado de los recursos
description: Use la información de las métricas, las alertas y el estado de los recursos disponible para el diagnóstico de Azure Standard Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/14/2019
ms.author: allensu
ms.openlocfilehash: 034a49793d3a3e416f307741e49446979eb33bb3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090457"
---
# <a name="standard-load-balancer-diagnostics-with-metrics-alerts-and-resource-health"></a>Diagnóstico de Standard Load Balancer con métricas, alertas y estado de los recursos

Azure Standard Load Balancer proporciona las siguientes funcionalidades de diagnóstico:

* **Métricas y alertas multidimensionales**: Proporciona funcionalidades de diagnóstico multidimensionales para configuraciones de Standard Load Balancer mediante [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview). Puede supervisar, administrar y solucionar problemas con los recursos del equilibrador de carga estándar.

* **Estado de los recursos**: La página Load Balancer en Azure Portal y la página Resource Health (en Monitor) exponen la sección Resource Health de Standard Load Balancer. 

En este artículo se proporciona una introducción a estas funcionalidades y maneras de usarlas con Load Balancer Estándar. 

## <a name="multi-dimensional-metrics"></a><a name = "MultiDimensionalMetrics"></a>Métricas multidimensionales

Azure Load Balancer proporciona métricas multidimensionales en la página Métricas de Azure de Azure Portal y le ayuda a obtener información detallada de diagnóstico en tiempo real sobre los recursos del equilibrador de carga. 

Las distintas configuraciones de Load Balancer Estándar proporcionan las siguientes métricas:

| Métrica | Tipo de recurso | Descripción | Agregación recomendada |
| --- | --- | --- | --- |
| Disponibilidad de la ruta de acceso de datos | Equilibrador de carga interno y público | Load Balancer Estándar usa continuamente la ruta de acceso a los datos desde una región hasta el servidor front-end del equilibrador de carga y, finalmente, hasta la pila de SDN que respalda la máquina virtual. Siempre que permanezcan las instancias correctas, la medida sigue la misma ruta de acceso que el tráfico con equilibrio de carga de las aplicaciones. También se valida la ruta de acceso a los datos que usan los clientes. La medida es invisible para la aplicación y no interfiere con otras operaciones.| Average |
| Estado del sondeo de mantenimiento | Equilibrador de carga interno y público | Load Balancer Estándar usa un servicio de sondeo de mantenimiento distribuido que supervisa el mantenimiento del punto de conexión de la aplicación de acuerdo con la configuración. Esta métrica proporciona una vista agregada o filtrada por punto de conexión de cada punto de conexión de instancia del grupo del equilibrador de carga. Puede ver cómo Load Balancer observa el estado de su aplicación según se indica en la configuración de sondeo de estado. |  Average |
| Paquetes SYN (sincronizar) | Equilibrador de carga interno y público | Load Balancer Estándar no finaliza las conexiones de Protocolo de control de transmisión (TCP) ni interactúa con los flujos de paquetes TCP o UDP. Los flujos y los protocolos de enlace son siempre entre el origen y la instancia de máquina virtual. Para solucionar mejor los escenarios de protocolo TCP, puede hacer uso de estos contadores de paquetes SYN para saber el número de intentos de conexión TCP realizados. La métrica indica el número de paquetes TCP SYN recibidos.| Average |
| Conexiones SNAT | Equilibrador de carga público |Load Balancer Estándar informa del número de flujos salientes enmascarados en el servidor front-end de dirección IP pública. Los puertos de traducción de direcciones de red de origen (SNAT) son un recurso agotable. Esta métrica puede proporcionar una indicación de la dependencia que su aplicación tiene de SNAT en los flujos salientes originados. Los contadores de los flujos de salida de SNAT que se realizaron con éxito y los que tuvieron algún error se notifican y se pueden utilizar para solucionar problemas y comprender el estado de los flujos de salida.| Average |
| Puertos SNAT asignados | Equilibrador de carga público | Standard Load Balancer informa del número de puertos SNAT asignados por instancia de back-end. | Average |
| Puertos SNAT usados | Equilibrador de carga público | Standard Load Balancer informa del número de puertos SNAT usados por instancia de back-end. | Average | 
| Contadores de bytes |  Equilibrador de carga interno y público | Load Balancer Estándar informa de los datos procesados por front-end. Es posible que observe que los bytes no se distribuyen equitativamente entre las instancias de back-end. Se espera que el algoritmo de Azure Load Balancer se base en flujos. | Average |
| Contadores de paquetes |  Equilibrador de carga interno y público | Load Balancer Estándar informa de los paquetes procesados por front-end.| Average |

  >[!NOTE]
  >Cuando se usa la distribución del tráfico de un equilibrador de carga interno a través de una NVA o un paquete SYN de firewall, las métricas del contador de bytes y del contador de paquetes no están disponibles y se mostrarán como cero. 
  
### <a name="view-your-load-balancer-metrics-in-the-azure-portal"></a>Visualización de las métricas del equilibrador de carga en Azure Portal

Azure Portal expone las métricas del equilibrador de carga en la página Métricas, que está disponible en la página de recursos del equilibrador de carga de un recurso específico y también en la página de Azure Monitor. 

Para ver las métricas de los recursos de Load Balancer Estándar:
1. Vaya a la página Métricas y realice una de las siguientes acciones:
   * En la página de recursos de equilibrador de carga, seleccione el tipo de métrica en la lista desplegable.
   * En la página de Azure Monitor, seleccione el recurso del equilibrador de carga.
2. Configure el tipo de agregación de métricas adecuado.
3. Opcionalmente, configure el filtrado y la agrupación necesarios.
4. Opcionalmente, configure el intervalo y la agregación de tiempo. La hora se muestra en UTC de forma predeterminada.

  >[!NOTE] 
  >La agregación de tiempo es importante cuando se interpretan ciertas métricas, ya que los datos se muestrean una vez cada minuto. Si la agregación de tiempo se establece en cinco minutos y se usa el tipo de agregación de métricas Suma en métricas como la de asignación de SNAT, el gráfico mostrará cinco veces el total de puertos SNAT asignados. 

![Métricas para Standard Load Balancer](./media/load-balancer-standard-diagnostics/lbmetrics1anew.png)

*Ilustración: Métrica de disponibilidad de la ruta de acceso de datos para Standard Load Balancer*

### <a name="retrieve-multi-dimensional-metrics-programmatically-via-apis"></a>Recuperación de las métricas multidimensionales mediante programación con API

Para obtener orientación de API para recuperar las definiciones y los valores de las métricas multidimensionales, consulte el [tutorial sobre la API REST de supervisión de Azure](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough#retrieve-metric-definitions-multi-dimensional-api). Estas métricas se pueden escribir en una cuenta de almacenamiento solo a través de la opción "Todas las métricas". 

### <a name="configure-alerts-for-multi-dimensional-metrics"></a>Configuración de alertas para métricas multidimensionales ###

Azure Standard Load Balancer admite alertas configurables fácilmente para métricas multidimensionales. Configure umbrales personalizados para que métricas específicas desencadenen alertas con distintos niveles de gravedad para habilitar una experiencia de supervisión de recursos sin contacto.

Para configurar alertas:
1. Vaya a la subhoja de la alerta del equilibrador de carga.
1. Creación de una nueva regla de alertas
    1.  Configuración de la condición de alerta
    1.  (Opcional) Adición de un grupo de acciones para la reparación automatizada
    1.  Asigne la gravedad de la alerta, el nombre y la descripción que habilitan una reacción intuitiva.

  >[!NOTE]
  >La ventana de configuración de la condición de alerta mostrará la serie temporal para el historial de señales. Hay una opción para filtrar esta serie temporal por dimensiones, como la dirección IP de back-end. Esto filtrará el gráfico de la serie temporal, pero **no** la propia alerta. No puede configurar alertas para direcciones IP de back-end específicas.

### <a name="common-diagnostic-scenarios-and-recommended-views"></a><a name = "DiagnosticScenarios"></a>Escenarios comunes de diagnóstico y vistas recomendadas

#### <a name="is-the-data-path-up-and-available-for-my-load-balancer-frontend"></a>¿Está lista y disponible la ruta de acceso de datos para mi front-end de Load Balancer?
<details><summary>Expanda</summary>

La métrica de disponibilidad de la ruta de acceso de datos describe el mantenimiento de la ruta de acceso de los datos dentro de la región para el host de proceso donde se encuentran las máquinas virtuales. Es un reflejo del mantenimiento de la infraestructura de Azure. Puede usar esta métrica para:
- Supervisar la disponibilidad externa del servicio
- Profundizar y decidir si la plataforma en la que se implementa el servicio es correcta o si el sistema operativo invitado o la instancia de la aplicación son correctas.
- Determinar si un evento está relacionado con el servicio o con el plano de datos subyacente. Esta métrica no debe confundirse con el estado de sondeo de mantenimiento ("disponibilidad de instancia de back-end").

Para obtener la disponibilidad de la ruta de acceso de los datos de los recursos de Standard Load Balancer:
1. Asegúrese de que está seleccionado el recurso del equilibrador de carga correcto. 
2. En la lista desplegable **Métrica**, seleccione **Disponibilidad de la ruta de acceso de datos**. 
3. En la lista desplegable **Agregación**, seleccione **Promedio**. 
4. Además, agregue un filtro en la dirección VIP de Frontend o el puerto de Frontend como dimensión con la dirección IP de front-end o puerto de front-end requerido, y agrúpelos por la dimensión seleccionada.

![Sondeo de VIP](./media/load-balancer-standard-diagnostics/LBMetrics-VIPProbing.png)

*Ilustración: Detalles de sondeo del front-end del equilibrador de carga*

La métrica se genera mediante una medición activa en la banda. Un servicio de sondeo dentro de la región origina el tráfico para la medida. El servicio se activa tan pronto como cree una implementación con un front-end público, y continúa hasta que quite el front-end. 

Se genera periódicamente un paquete que coincide con el front-end y la regla de la implementación. Recorre la región desde el origen hasta el host, donde se encuentra una máquina virtual en el grupo back-end. La infraestructura del equilibrador de carga llevará a cabo las mismas operaciones de traducción y equilibrio de carga que con el tráfico restante. Este sondeo está en la banda en el punto de conexión de la carga equilibrada. Una vez que el sondeo llega al host Compute donde se encuentra una máquina virtual correcta en el grupo de servidores back-end, el host Compute genera una respuesta al servicio de sondeo. La máquina virtual no ve este tráfico.

Se produce un error en la disponibilidad de la ruta de acceso de datos por los motivos siguientes:
- Para la implementación no queda ninguna máquina virtual correcta en el grupo de servidores back-end. 
- Se ha producido una interrupción de la infraestructura.

Puede usar la [métrica de disponibilidad de ruta de acceso a datos junto con el estado de sondeo de mantenimiento](#vipavailabilityandhealthprobes) con fines de diagnóstico.

Use **Average** (Promedio) como agregación para la mayoría de los escenarios.
</details>

#### <a name="are-the-backend-instances-for-my-load-balancer-responding-to-probes"></a>¿Responden las instancias de back-end de mi Load Balancer a los sondeos?
<details>
  <summary>Expanda</summary>
La métrica de estado de sondeo de mantenimiento describe el mantenimiento de la implementación de la aplicación según la configurara al configurar el sondeo de mantenimiento del equilibrador de carga. El equilibrador de carga usa el estado de sondeo de mantenimiento para determinar dónde se deben enviar flujos nuevos. Los sondeos de mantenimiento se originan en una dirección de infraestructura de Azure y se ven en el sistema operativo invitado de la máquina virtual.

Para obtener el estado de sondeo de mantenimiento para los recursos de Standard Load Balancer:
1. Seleccione la métrica **Estado de sondeo de mantenimiento** con el tipo de agregación **Promedio**. 
2. Aplique un filtro en el puerto o la dirección IP de Frontend (o ambos) necesarios.

Se producirá un error en el sondeo de mantenimiento por los motivos siguientes:
- Se configura un sondeo de mantenimiento en un puerto que no está escuchando o no responde, o que usa un protocolo incorrecto. Si el servicio utiliza reglas DSR (IP flotante), debe asegurarse de que el servicio está escuchando en la dirección IP de la configuración de IP de la NIC y no solo en el bucle invertido configurado con la dirección IP de front-end.
- El grupo de seguridad de red, el firewall del sistema operativo invitado de la máquina virtual o los filtros de nivel de aplicación no permiten el sondeo.

Use **Average** (Promedio) como agregación para la mayoría de los escenarios.
</details>

#### <a name="how-do-i-check-my-outbound-connection-statistics"></a>¿Cómo se comprueban las estadísticas de conexión saliente? 
<details>
  <summary>Expanda</summary>
La métrica de conexiones SNAT describe el volumen de conexiones correctas y con errores para los [flujos salientes](https://aka.ms/lboutbound).

Un volumen de conexión con errores mayor que cero indica el agotamiento de los puertos SNAT. Debe investigar en profundidad para determinar las posibles causas de estos errores. El agotamiento de los puertos SNAT se manifiesta como error al establecer un [flujo saliente](https://aka.ms/lboutbound). Revise el artículo sobre las conexiones salientes para comprender los escenarios y los mecanismos en el trabajo, y aprender a mitigar o diseñar para evitar el agotamiento de los puertos SNAT. 

Para obtener las estadísticas de conexión SNAT:
1. Seleccione el tipo de métrica **Conexiones SNAT** y **Suma** como agregación. 
2. Agrupe por **Estado de conexión** para el recuento de conexiones SNAT correctas y con errores representadas por diferentes líneas. 

![Conexión SNAT](./media/load-balancer-standard-diagnostics/LBMetrics-SNATConnection.png)

*Ilustración: Recuento de conexiones SNAT para Load Balancer*
</details>


#### <a name="how-do-i-check-my-snat-port-usage-and-allocation"></a>¿Cómo se comprueba el uso y asignación de puertos SNAT?
<details>
  <summary>Expanda</summary>
La métrica de uso de SNAT indica cuántos flujos únicos se establecen entre un origen de Internet y una máquina virtual back-end o un conjunto de escalado de máquinas virtuales que está detrás de un equilibrador de carga y carece de dirección IP pública. Al compararla con la métrica de asignación de SNAT, se puede determinar si el servicio está en fase o en riesgo de agotamiento de SNAT, lo que podría derivar en un error en el flujo saliente. 

Si las métricas señalan que hay riesgo de error del [flujo saliente](https://aka.ms/lboutbound), vea el artículo y tome medidas para mitigar este problema para poder garantizar el estado del servicio.

Para ver el uso y asignación de puertos SNAT:
1. Establezca la agregación de tiempo del gráfico en 1 minuto para procurar que se muestren los datos deseados.
1. Seleccione **Uso de SNAT** y/o **Asignación de SNAT** como tipo de métrica y **Promedio** como agregación.
    * De forma predeterminada, este es el promedio de puertos SNAT asignados a (o usados por) cada máquina virtual o conjunto de máquinas virtuales de back-end, y se corresponde con todas las direcciones IP de front-end públicas asignadas al equilibrador de carga, agregadas a través de TCP y UDP.
    * Para ver los puertos SNAT totales usados por (o asignados a) el equilibrador de carga, use la agregación de métricas **Suma**.
1. Filtre por un **Tipo de protocolo** específico, un conjunto de **IP de back-end** y/o **IP de front-end**.
1. Para supervisar el estado por instancia de front-end o back-end, use una división. 
    * Cabe decir que la división solo permite que se muestre una sola métrica cada vez. 
1. Por ejemplo, para supervisar el uso de SNAT en los flujos TCP por equipo, agregue **Promedio**, divida entre **IP de back-end** y filtre por **Tipo de protocolo**. 

![Uso y asignación de SNAT](./media/load-balancer-standard-diagnostics/snat-usage-and-allocation.png)

*Ilustración: Promedio de uso y asignación de puertos TCP SNAT de un conjunto de máquinas virtuales de back-end*

![Uso de SNAT por instancia de back-end](./media/load-balancer-standard-diagnostics/snat-usage-split.png)

*Ilustración: Uso de puertos TCP SNAT por instancia de back-end*
</details>

#### <a name="how-do-i-check-inboundoutbound-connection-attempts-for-my-service"></a>¿Cómo se comprueban los intentos de conexión entrantes y salientes de mi servicio?
<details>
  <summary>Expanda</summary>
La métrica de paquetes SYN describe el volumen de los paquetes TCP SYN, que han llegado o que se enviaron (para [flujos salientes](https://aka.ms/lboutbound)), asociado con un front-end determinado. Esta métrica se puede utilizar para entender los intentos de conexión TCP al servicio.

Use **Total** como agregación para la mayoría de los escenarios.

![Conexión SYN](./media/load-balancer-standard-diagnostics/LBMetrics-SYNCount.png)

*Ilustración: Recuento de SYN para Load Balancer*
</details>


#### <a name="how-do-i-check-my-network-bandwidth-consumption"></a>¿Cómo se comprueba el consumo de ancho de banda de la red? 
<details>
  <summary>Expanda</summary>
La métrica de contadores de bytes/paquetes describe el volumen de bytes y paquetes enviados o recibidos por el servicio por front-end.

Use **Total** como agregación para la mayoría de los escenarios.

Para obtener las estadísticas de recuento de bytes o paquetes:
1. Seleccione el tipo de métrica **Recuento de bytes** o **Recuento de paquetes** con **Promedio** como agregación. 
2. Realice cualquiera de las siguientes acciones:
   * Aplique un filtro en una IP de front-end específica, un puerto front-end, una IP de back-end o un puerto de back-end.
   * Obtenga estadísticas generales para los recursos del equilibrador de carga sin filtrado.

![Recuento de bytes](./media/load-balancer-standard-diagnostics/LBMetrics-ByteCount.png)

*Ilustración: Recuento de bytes para Load Balancer*
</details>

#### <a name="how-do-i-diagnose-my-load-balancer-deployment"></a><a name = "vipavailabilityandhealthprobes"></a>¿Cómo se diagnostica la implementación de Load Balancer?
<details>
  <summary>Expanda</summary>
El uso de una combinación de las métricas de disponibilidad de la ruta de acceso de datos y de estado de sondeo de mantenimiento en un único gráfico permite identificar dónde buscar el problema para resolverlo. Puede estar seguro de que Azure funciona correctamente y basarse en esta certeza para determinar de forma concluyente la configuración o la aplicación con la causa principal.

Puede utilizar las métricas de sondeo de mantenimiento para entender cómo Azure ve el mantenimiento de la implementación según la configuración que ha proporcionado. Examinar los sondeos de mantenimiento siempre es un primer paso importante para supervisar o determinar una causa.

Puede dar otro paso y usar la métrica de disponibilidad de la ruta de acceso de datos para comprender mejor cómo Azure ve el mantenimiento del plano de datos subyacente responsable de la implementación específica. Cuando se combinan ambas métricas, se puede determinar el lugar del error, como se ilustra en este ejemplo:

![Combinación de las métricas de disponibilidad de ruta de acceso a datos y de estado de sondeo de mantenimiento](./media/load-balancer-standard-diagnostics/lbmetrics-dipnvipavailability-2bnew.png)

*Ilustración: Combinación de las métricas de disponibilidad de ruta de acceso a datos y de estado de sondeo de mantenimiento*

En este gráfico se muestra la siguiente información:
- La infraestructura que hospeda las máquinas virtuales no estaba disponible y estaba en el 0 por ciento al principio del gráfico. Después, la infraestructura era correcta y las máquinas virtuales eran alcanzables, y más de una máquina virtual se colocó en el back-end. Esta información se indica mediante el seguimiento azul para la disponibilidad de la ruta de acceso de datos, que se muestra al 100 %. 
- El estado de sondeo de mantenimiento, indicado por el seguimiento de color púrpura, es del 0 % al principio del gráfico. El área en un círculo verde resalta dónde el estado de sondeo de mantenimiento cambió a correcto y en qué momento la implementación del cliente pudo aceptar nuevos flujos.

El gráfico permite a los clientes solucionar los problemas de la implementación por sí mismos, sin necesidad de adivinar o solicitar soporte técnico si se produjeron otros problemas. El servicio no estaba disponible porque los sondeos de mantenimiento producían errores debidos a una configuración incorrecta o errores en la aplicación.
</details>

## <a name="resource-health-status"></a><a name = "ResourceHealth"></a>Estado de mantenimiento de los recursos

El estado de mantenimiento de los recursos de Load Balancer Estándar se expone en **Mantenimiento de los recursos**, en **Supervisar > Estado del servicio**.

Para ver el mantenimiento de los recursos públicos de Load Balancer Estándar:
1. Seleccione **Monitor** > **Service Health**.

   ![Página Supervisar](./media/load-balancer-standard-diagnostics/LBHealth1.png)

   *Ilustración: Vínculo de Service Health en Azure Monitor*

2. Seleccione **Resource Health** y asegúrese de que **Id. de suscripción** y **Resource Type = Load Balancer** (Tipo de recurso = Load Balancer) están seleccionados.

   ![Estado de mantenimiento de los recursos](./media/load-balancer-standard-diagnostics/LBHealth3.png)

   *Ilustración: Selección de un recurso para ver su mantenimiento*

3. Haga clic en el recurso de Load Balancer de la lista para ver sus datos históricos de estado de mantenimiento.

    ![Estado de mantenimiento de Load Balancer](./media/load-balancer-standard-diagnostics/LBHealth4.png)

   *Ilustración: Vista del mantenimiento de los recursos de Load Balancer*
 
En la tabla siguiente se enumeran los estados de mantenimiento de varios recursos y su descripción. 

| Estado de mantenimiento de los recursos | Descripción |
| --- | --- |
| Disponible | El recurso de Standard Load Balancer está listo y disponible. |
| No disponible | El recurso de Standard Load Balancer público no es correcto. Diagnostique el estado seleccionando **Azure Monitor** > **Métricas**.<br>(El estado *No disponible* también puede significar que el recurso no está conectado a Standard Load Balancer). |
| Desconocido | El estado de mantenimiento del recurso para Standard Load Balancer aún no se ha actualizado.<br>(El estado *Desconocido* también puede significar que el recurso no está conectado a Standard Load Balancer).  |

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Load Balancer Estándar](load-balancer-standard-overview.md).
- Más información sobre la [conectividad saliente de Load Balancer](https://aka.ms/lboutbound).
- Más información acerca de [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview).
- Obtenga información sobre la [API REST de Azure Monitor](https://docs.microsoft.com/rest/api/monitor/) y [cómo recuperar las métricas a través de la API REST](/rest/api/monitor/metrics/list).
