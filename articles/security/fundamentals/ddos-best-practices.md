---
title: Diseño de soluciones resistentes con Azure DDoS Protection
description: Aprenda a usar datos de registro para obtener un conocimiento más profundo sobre su aplicación.
services: security
author: terrylanfear
manager: RKarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/18/2018
ms.author: terrylan
ms.openlocfilehash: 3b1a0d646c05be0de83837b33e77e34969c9254c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287801"
---
# <a name="azure-ddos-protection---designing-resilient-solutions"></a>Azure DDoS Protection: diseño de soluciones resistentes

Este artículo está destinado a los responsables de decisiones de TI y al personal de seguridad. Se presupone que está familiarizado con las redes y la seguridad de Azure.
DDoS es un tipo de ataque que intenta agotar los recursos de la aplicación. El objetivo es afectar a la disponibilidad de la aplicación y a su capacidad para administrar solicitudes legítimas. Los ataques son cada vez más sofisticados y mayor en tamaño e impacto. Los ataques DDoS pueden ir dirigidos a cualquier punto de conexión que sea públicamente accesible a través de Internet. Diseñar para crear resistencia frente a los ataques de denegación de servicio distribuidos (DDoS) requiere planear y diseñar para una amplia variedad de modos de error. Azure proporciona protección continua contra los ataques de DDoS. Esta protección se integra en la plataforma Azure de forma predeterminada y sin costo adicional.

Además de la protección contra DDoS en la plataforma, [Azure DDoS Protection Standard](https://azure.microsoft.com/services/ddos-protection/) proporciona funcionalidades avanzadas de mitigación contra DDoS ante ataques en la red. Se ajusta automáticamente para proteger los recursos específicos de Azure. La protección se puede habilitar fácilmente durante la creación de nuevas redes virtuales. También puede realizarse después de la creación y no requiere ningún cambio de aplicación o recurso.

![Función de Azure DDoS Protection a la hora de proteger a los clientes y a una red virtual de un atacante](./media/ddos-best-practices/image1.png)


## <a name="fundamental-best-practices"></a>Procedimientos recomendados fundamentales

En las siguientes secciones se proporcionan instrucciones preceptivas para crear servicios resistentes a los ataques de DDoS en Azure.

### <a name="design-for-security"></a>Diseño para seguridad

Asegúrese de que la seguridad es prioritaria durante todo el ciclo de vida de una aplicación, desde su diseño e implementación hasta la implementación y las operaciones. Las aplicaciones pueden tener errores que permiten que un volumen relativamente bajo de solicitudes usen una cantidad excesiva de recursos y produzcan una interrupción del servicio. 

Para proteger un servicio que se ejecuta en Microsoft Azure, debe conocer bien la arquitectura de su aplicación y centrarse en los [cinco pilares de la calidad del software](/azure/architecture/guide/pillars).
Debe conocer los volúmenes de tráfico típicos, el modelo de conectividad entre la aplicación y otras aplicaciones, y los puntos de conexión del servicio expuestos a la red pública de Internet.

Es de vital importancia garantizar que una aplicación sea lo suficientemente resistente para tratar con un ataque de denegación de servicio dirigido a la propia aplicación. La seguridad y la privacidad están integradas en la plataforma Azure, comenzando por el  [ciclo de vida del desarrollo de la seguridad (SDL)](https://www.microsoft.com/sdl/default.aspx). El SDL aborda la seguridad en cada fase de desarrollo y se asegura de que Azure se actualice continuamente para que sea aún más seguro.

### <a name="design-for-scalability"></a>Diseño para escalabilidad

La escalabilidad representa el grado en que un sistema puede controlar el aumento de la carga. Debe diseñar sus aplicaciones de modo que se puedan [escalar horizontalmente](/azure/architecture/guide/design-principles/scale-out) para satisfacer la demanda de una carga mayor, específicamente en caso de un ataque de DDoS. Si la aplicación depende de una única instancia de un servicio, crea un único punto de error. El aprovisionamiento de varias instancias hace que el sistema sea más resistente y más escalable.

Para [Azure App Service](/azure/app-service/app-service-value-prop-what-is), seleccione un [Plan de App Service](/azure/app-service/overview-hosting-plans) que ofrezca varias instancias. Para Azure Cloud Services, configure cada uno de los roles para utilizar [varias instancias](/azure/cloud-services/cloud-services-choose-me). En el caso de [Azure Virtual Machines](../../virtual-machines/index.yml), asegúrese de que la arquitectura de las máquinas virtuales incluya más de una máquina virtual y que cada una de ellas se incluya en un [conjunto de disponibilidad](../../virtual-machines/windows/tutorial-availability-sets.md). Se recomienda usar [conjuntos de escalado de máquinas virtuales](../../virtual-machine-scale-sets/overview.md) para contar con funcionalidades de escalado automático.

### <a name="defense-in-depth"></a>Defensa en profundidad

La idea que subyace a la defensa en profundidad es administrar los riesgos con diversas estrategias defensivas. Disponer en niveles la defensa de la seguridad en una aplicación reduce las probabilidades de éxito de un ataque. Se recomienda que implemente diseños seguros para sus aplicaciones con las funcionalidades integradas de la plataforma Azure.

Por ejemplo, el riesgo de ataques aumenta con el tamaño (*área expuesta*) de la aplicación. Puede reducir el área expuesta mediante la creación de listas blancas para cerrar el espacio de direcciones IP expuesto y los puertos de escucha que no sean necesarios en los equilibradores de carga ([Azure Load Balancer](/azure/load-balancer/load-balancer-get-started-internet-portal) y [Azure Application Gateway](/azure/application-gateway/application-gateway-create-probe-portal)). Los [grupos de seguridad de red (NSG)](/azure/virtual-network/security-overview) constituyen otra manera de reducir el área expuesta a ataques.
Puede usar [etiquetas de servicio](/azure/virtual-network/security-overview#service-tags) y [grupos de seguridad de la aplicación](/azure/virtual-network/security-overview#application-security-groups) para minimizar la complejidad de la creación de reglas de seguridad y configurar la seguridad de la red como una extensión natural de la estructura de una aplicación.

Debe implementar los servicios de Azure en una [red virtual](/azure/virtual-network/virtual-networks-overview) siempre que sea posible. Este procedimiento permite que los recursos del servicio se comuniquen mediante direcciones IP privadas. De forma predeterminada, el tráfico de los servicios Azure desde una red virtual usa direcciones IP públicas como direcciones IP de origen. Con los [puntos de conexión de servicio](/azure/virtual-network/virtual-network-service-endpoints-overview), el tráfico del servicio cambia para usar direcciones privadas de red virtual como direcciones IP de origen al acceder al servicio de Azure desde una red virtual.

Con frecuencia vemos ataques a los recursos locales de un cliente, además de a los recursos en Azure. Si conecta un entorno local a Azure, se recomienda que reduzca al mínimo la exposición de los recursos locales a la red pública de Internet. Para usar las funcionalidades de escalado y protección contra DDoS de Azure puede implementar entidades públicas bien conocidas en Azure. Como estas entidades de acceso público suelen ser destinatarias de ataques de DDoS, al colocarlas en Azure se reduce el impacto en los recursos locales.

## <a name="azure-offerings-for-ddos-protection"></a>Ofertas de Azure para protección contra DDoS

Azure tiene dos ofertas de servicio de DDoS que proporcionan protección frente a ataques de red (niveles 3 y 4): DDoS Protection Basic y DDoS Protection Standard. 

### <a name="ddos-protection-basic"></a>DDoS Protection Basic

DDoS Protection Basic se integra en Azure de forma predeterminada y sin costo adicional. El tamaño y la capacidad de la red de implementación global de Azure proporciona una defensa contra los ataques al nivel de red más comunes mediante la supervisión constante del tráfico y la mitigación en tiempo real. DDoS Protection Basic no requiere ningún cambio en la configuración de usuarios o aplicaciones. DDoS Protection Basic ayuda a proteger todos los servicios de Azure, incluidos servicios de PaaS como Azure DNS.

![Asigne la representación de la red de Azure con el texto "Presencia global de mitigación de DDoS" y "Capacidad principal de mitigación de DDoS"](./media/ddos-best-practices/image3.png)

Azure DDoS Protection Basic consta de componentes de hardware y software. El plano de control de software decide cuándo, dónde y qué tipo de tráfico debe derivarse a los dispositivos de hardware que analizan y quitan el tráfico de ataque. El plano de control toma esta decisión basándose en una *directiva* de protección contra DDoS para toda la infraestructura. Esta directiva se establece estáticamente y se aplica universalmente a todos los clientes de Azure.

Por ejemplo, la directiva de protección contra DDoS especifica con qué volumen de tráfico se debe *desencadenar* la protección. (Es decir, el tráfico del inquilino se debe enrutar a través de los dispositivos de limpieza). La directiva, a continuación, especifica cómo los dispositivos de limpieza deben *mitigar* el ataque.

El objetivo del servicio Azure DDoS Protection Basic es proteger la infraestructura y la plataforma Azure. Reduce el tráfico cuando supera una tasa que es tan significativa que podría afectar a varios clientes en un entorno multiinquilino. No proporciona alertas ni directivas personalizadas por cliente.

### <a name="ddos-protection-standard"></a>DDoS Protection Standard

La protección estándar proporciona características mejoradas de mitigación de DDoS. Se ajusta automáticamente para proteger los recursos específicos de Azure de una red virtual. La protección se puede habilitar fácilmente en cualquier red virtual nueva o existente y no requiere cambios en las aplicaciones ni los recursos. Tiene varias ventajas sobre el servicio básico, incluidos el registro, las alertas y la telemetría. Las siguientes secciones describen las características clave del servicio Azure DDoS Protection Standard.

#### <a name="adaptive-real-time-tuning"></a>Ajuste adaptable en tiempo real

El servicio Azure DDoS Protection Basic le ayuda a proteger a los clientes y a evitar que los ataques afecten a otros clientes. Por ejemplo, si un servicio se aprovisiona para un volumen normal de tráfico de entrada legítimo que es menor que la *tasa desencadenante* de la directiva de DDoS Protection para toda la infraestructura, un ataque de DDoS en los recursos de ese cliente podría pasar desapercibido. Por lo general, la complejidad de los ataques recientes (por ejemplo, DDoS multivector), así como los comportamientos de los inquilinos, específicos de cada aplicación, hacen necesarias directivas de protección personalizadas para cada cliente. El servicio consigue esta personalización basándose en dos tipos de información:

- Aprendizaje automático de los patrones de tráfico de los niveles 3 y 4 (por IP) de cada cliente.

- Reducción de los falsos positivos ya que el escalado de Azure le permite absorber una cantidad importante de tráfico.

![Diagrama de funcionamiento de DDoS Protection Standard, con "Generación de directiva" rodeada por un círculo](./media/ddos-best-practices/image5.png)

#### <a name="ddos-protection-telemetry-monitoring-and-alerting"></a>Telemetría, alertas y supervisión de DDoS Protection

DDoS Protection Standard expone datos de telemetría detallados mediante [Azure Monitor](/azure/azure-monitor/overview) mientras dura un ataque de DDoS. También puede configurar alertas para todas las métricas de Azure Monitor que usa DDoS Protection. El registro se puede integrar con Splunk (Azure Event Hubs), registros de Azure Monitor y Azure Storage para realizar análisis avanzados con la interfaz de Azure Monitor Diagnostics.

##### <a name="ddos-mitigation-policies"></a>Directivas de mitigación de DDoS

En Azure Portal, seleccione **Monitor** > **Métricas**. En el panel **Métricas**, seleccione el grupo de recursos, un tipo de recurso de **Dirección IP pública** y la dirección IP pública de Azure. Las métricas de DDoS estarán visibles en el panel **Métricas disponibles**.

DDoS Protection Standard aplica tres directivas de mitigación de ajuste automático (TCP SYN, TCP y UDP) a cada dirección IP pública del recurso protegido, en la red virtual que tiene habilitado DDoS. Para ver los umbrales de la directiva, seleccione la métrica "**Inbound packets to trigger DDoS mitigation**" (Paquetes de entrada para desencadenar la mitigación de DDoS).

![Gráficos de métricas y de métricas disponibles](./media/ddos-best-practices/image7.png)

Los umbrales de las directivas se configuran automáticamente con el sistema de generación de perfiles de tráfico de red basado en aprendizaje automático. La mitigación de DDoS se produce para una dirección IP que está siendo atacada solo cuando se supera el umbral de la directiva.

##### <a name="metric-for-an-ip-address-under-ddos-attack"></a>Métricas de una dirección IP sometida a un ataque de DDoS

Si la dirección IP pública está siendo atacada, el valor de la métrica **Under DDoS attack or not** (Bajo ataque de DDoS o no) cambia a 1 cuando aplicamos la mitigación al tráfico de ataque.

![Métrica y gráfico "Under DDoS attack or not" (Bajo ataque de DDoS o no)](./media/ddos-best-practices/image8.png)

Se recomienda configurar una alerta en esta métrica. Si lo hace, recibirá una notificación cuando se esté aplicando una mitigación de DDoS en su dirección IP pública.

Para más información, consulte [Administración de Azure DDoS Protection Standard mediante Azure Portal](/azure/virtual-network/ddos-protection-manage-portal).

#### <a name="web-application-firewall-for-resource-attacks"></a>Firewall de aplicaciones web para ataques a recursos

En el caso de los ataques a recursos en el nivel de aplicación, los clientes deben configurar un firewall de aplicaciones web (WAF) para ayudar a proteger las aplicaciones web. Un WAF inspecciona el tráfico web entrante para bloquear las inyecciones de SQL, los scripts entre sitios, DDoS y otros ataques al nivel 7. Azure ofrece [WAF como una característica de Application Gateway](/azure/application-gateway/application-gateway-web-application-firewall-overview) para ofrecer una protección centralizada de las aplicaciones web contra las vulnerabilidades de seguridad comunes. En [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?search=WAF&page=1) encontrará ofertas de WAF de asociados de Azure que podrían ser más adecuadas para sus necesidades.

Incluso los firewalls de aplicaciones web son susceptibles de sufrir ataques de agotamiento del estado y volumétricos. Es muy recomendable activar DDoS Protection Standard en la red virtual del WAF para impedir los ataques a protocolos y volumétricos. Para más información, consulte la sección [Arquitecturas de referencia de DDoS Protection](#ddos-protection-reference-architectures).

### <a name="protection-planning"></a>Planeamiento de la protección

El planeamiento y la preparación son cruciales para entender cómo se comportará un sistema durante un ataque de DDoS. Diseñar un plan de respuesta de administración de incidentes forma parte de este esfuerzo.

Si dispone de DDoS Protection Standard, asegúrese de que esté habilitado en la red virtual de los puntos de conexión accesibles desde Internet. La configuración de alertas de DDoS le ayuda a vigilar constantemente los posibles ataques contra su infraestructura. 

Supervise las aplicaciones de forma independiente. Debe conocer el comportamiento normal de una aplicación. Debe estar preparado para actuar si la aplicación no se comporta como está previsto durante un ataque de DDoS.

#### <a name="testing-through-simulations"></a>Pruebas mediante simulaciones

El procedimiento recomendado es realizar simulaciones periódicas para probar las suposiciones acerca de cómo responderán los servicios a un ataque. Durante las pruebas, compruebe que los servicios y las aplicaciones continúan funcionando según lo previsto y que no hay ninguna interrupción en la experiencia del usuario. Identifique las carencias desde la perspectiva de la tecnología y de los procesos, e incorpórelas en la estrategia de respuesta a DDoS. Es recomendable realizar dichas pruebas en entornos de ensayo o durante las horas de poca actividad para reducir el impacto en el entorno de producción.

Nos hemos asociado con [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) para crear una interfaz en la que los clientes de Azure puedan generar tráfico destinado a los puntos de conexión públicos que tengan habilitado el servicio DDoS Protection con fines de simulación. Puede usar la simulación [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) para:

- Comprobar en qué medida Azure DDoS Protection protege sus recursos de Azure frente a ataques de DDoS.

- Optimizar el proceso de respuesta a incidentes durante el ataque de DDoS.

- Documentar el cumplimiento normativo de DDoS.

- Enseñar a los equipos de seguridad de red.

La ciberseguridad requiere una innovación constante en materia de defensa. Azure DDoS Protection Standard es una innovadora oferta que proporciona una solución eficaz para mitigar los ataques de DDoS cada vez más complejos.

## <a name="components-of-a-ddos-response-strategy"></a>Componentes de una estrategia de respuesta a DDoS

Un ataque DDoS destinado a recursos de Azure normalmente requiere una intervención mínima desde la perspectiva del usuario. No obstante, incorporar la mitigación de DDoS como parte de la estrategia de respuesta a incidentes ayuda a minimizar el impacto en la continuidad del negocio.

### <a name="microsoft-threat-intelligence"></a>Información sobre amenazas de Microsoft

Microsoft cuenta con una extensa red de información sobre amenazas. Esta red usa los conocimientos colectivos de una amplia comunidad de seguridad para dar soporte a los servicios en línea de Microsoft, los asociados de Microsoft y las relaciones dentro de la comunidad de seguridad de Internet. 

Como importante proveedor de infraestructura, Microsoft recibe alertas tempranas acerca de las amenazas. Microsoft recopila la información sobre amenazas obtenida de otros servicios en línea y de la base internacional de clientes. Microsoft incorpora toda esta información sobre amenazas a los productos de Azure DDoS Protection.

Además, Microsoft Digital Crimes Unit (DCU) adopta estrategias ofensivas frente a las redes de robots. Las redes de robots son una causa común de comando y control de los ataques de DDoS.

### <a name="risk-evaluation-of-your-azure-resources"></a>Evaluación del riesgo de los recursos de Azure

Es fundamental conocer el alcance del riesgo de un ataque de DDoS de forma continuada. Periódicamente, pregúntese lo siguiente:

- ¿Qué nuevos recursos de Azure que están públicamente disponibles necesitan protección?

- ¿Hay un único punto de error en el servicio? 

- ¿Cómo pueden aislarse los servicios para limitar el impacto de un ataque, manteniendo al mismo tiempo los servicios disponibles para los clientes legítimos?

- ¿Hay redes virtuales en las que DDoS Protection Standard debería estar habilitado y no lo está? 

- ¿Estás mis servicios activos o activos con conmutación por error en varias regiones?

### <a name="customer-ddos-response-team"></a>Equipo de respuesta a DDoS del cliente

Crear un equipo de respuesta a DDoS es un paso clave para garantizar una respuesta rápida y eficaz a un ataque. Debe identificar los contactos de la organización que supervisarán el planeamiento y la ejecución. Este equipo de respuesta de DDoS debe conocer perfectamente el servicio Azure DDoS Protection Standard. Asegúrese de que el equipo puede identificar y mitigar un ataque en coordinación con clientes internos y externos, incluido el equipo de soporte técnico de Microsoft.

Para el equipo de respuesta frente a ataques de DDoS, es recomendable que utilice ejercicios de simulación como parte normal de la disponibilidad del servicio y el planeamiento de la continuidad. Estos ejercicios deben incluir pruebas de escalado.

### <a name="alerts-during-an-attack"></a>Alertas durante un ataque

Azure DDoS Protection Standard identifica y mitiga los ataques de DDoS sin intervención del usuario. Para recibir una notificación cuando haya una mitigación activa en una dirección IP pública protegida, también puede [configurar una alerta](/azure/virtual-network/ddos-protection-manage-portal) en la métrica **Under DDoS attack or not** (Bajo ataque de DDoS o no). Si es necesario, puede crear alertas para otras métricas de DDoS para entender la magnitud del ataque, el tráfico que se va a quitar, etc.

#### <a name="when-to-contact-microsoft-support"></a>Cuándo ponerse en contacto con el soporte técnico de Microsoft

- Si, durante un ataque de DDoS, encuentra que la degradación del rendimiento del recurso protegido es grave o el recurso no está disponible.

- Si cree que el servicio DDoS Protection no se comporta según lo previsto. 

  El servicio DDoS Protection comienza la mitigación solo si el valor de la métrica **Policy to trigger DDoS mitigation (TCP/TCP SYN/UDP)** [Directiva para desencadenar la mitigación de DDoS (TCP/TCP SYN/UDP)] es menor que el tráfico recibido en el recurso de IP pública protegido.

- Planea un evento viral que aumentará significativamente el tráfico.

- Si alguien ha amenazado con iniciar un ataque de DDoS contra sus recursos.

- Si necesita incluir en la lista de permitidos una dirección IP o un intervalo IP de Azure DDoS Protection Estándar. Un escenario común consiste en incluir en la lista de permitidos una dirección IP si el tráfico se enruta desde una nube externa WAF hasta Azure. 

En caso de ataques que afectan a aspectos críticos de su empresa, cree una [incidencia de soporte](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) de gravedad A.

### <a name="post-attack-steps"></a>Pasos posteriores al ataque

Siempre es una buena estrategia hacer una autopsia después de un ataque y ajustar la estrategia de respuesta a DDoS según sea necesario. Puntos que se deben tener en cuenta:

- ¿Se produjo una interrupción en el servicio o en la experiencia del usuario debido a la falta de una arquitectura escalable?

- ¿Qué aplicaciones o servicios sufrieron más?

- ¿Fue eficaz la estrategia de respuesta a DDoS y cómo se podría mejorar aún más?

Si sospecha que está sufriendo un ataque de DDoS, notifíquelo a través de los canales de soporte técnico de Azure habituales.

## <a name="ddos-protection-reference-architectures"></a>Arquitecturas de referencia de DDoS Protection

DDoS Protection Standard se ha diseñado [para los servicios que se implementan en una red virtual](/azure/virtual-network/virtual-network-for-azure-services). Para otros servicios se aplica el servicio predeterminado DDoS Protection Basic. Las siguientes arquitecturas de referencia se organizan por escenarios, y los patrones de la arquitectura se agrupan juntos.

### <a name="virtual-machine-windowslinux-workloads"></a>Cargas de trabajo de máquina virtual (Windows o Linux)

#### <a name="application-running-on-load-balanced-vms"></a>Aplicación que se ejecuta en máquinas virtuales con equilibrio de carga

En esta arquitectura de referencia se muestra un conjunto de prácticas demostradas para ejecutar varias máquinas virtuales con Windows en un conjunto de escalado detrás de un equilibrador de carga, para mejorar la disponibilidad y escalabilidad. Esta arquitectura puede usarse para cargas de trabajo sin estado, como un servidor web.

![Diagrama de la arquitectura de referencia para una aplicación que se ejecuta en máquinas virtuales con equilibrio de carga](./media/ddos-best-practices/image9.png)

En esta arquitectura, una carga de trabajo se distribuye entre varias instancias de máquina virtual. Hay una única dirección IP pública, y el tráfico de Internet se distribuye a las máquinas virtuales a través de un equilibrador de carga. DDoS Protection Standard está habilitado en la red virtual del equilibrador de carga de Azure (Internet) que tiene asociada la dirección IP pública.

El equilibrador de carga distribuye las solicitudes entrantes de Internet a las instancias de máquina virtual. Los conjuntos de escalado de máquinas virtuales permiten reducir o escalar horizontalmente el número de máquinas virtuales de forma manual, o bien automáticamente en función de reglas predefinidas. Esto es importante si el recurso está sufriendo un ataque de DDoS. Consulte [este artículo](/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm) para más información acerca de esta arquitectura de referencia.

#### <a name="application-running-on-windows-n-tier"></a>Aplicación que se ejecuta en una arquitectura de n niveles de Windows

Hay muchas maneras de implementar una arquitectura de n niveles. El siguiente diagrama muestra una aplicación web típica de tres niveles. Esta arquitectura se basa en el artículo [Ejecución de máquinas virtuales de carga equilibrada para escalabilidad y disponibilidad](/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm). Los niveles Web y Business usan máquinas virtuales de carga equilibrada.

![Diagrama de la arquitectura de referencia para una aplicación que se ejecuta en una arquitectura de n niveles de Windows](./media/ddos-best-practices/image10.png)

En esta arquitectura, DDoS Protection Standard está habilitado en la red virtual. Todas las direcciones IP públicas de la red virtual obtendrán protección contra DDoS en los niveles 3 y 4. Para la protección del nivel 7, implemente Application Gateway en la SKU de WAF. Consulte [este artículo](/azure/architecture/reference-architectures/virtual-machines-windows/n-tier) para más información acerca de esta arquitectura de referencia.

#### <a name="paas-web-application"></a>Aplicación web PaaS

Esta arquitectura de referencia muestra la ejecución de una aplicación de Azure App Service en una única región. Esta arquitectura muestra un conjunto de prácticas demostradas para una aplicación web que usa  [Azure App Service](https://azure.microsoft.com/documentation/services/app-service/) y [Azure SQL Database](https://azure.microsoft.com/documentation/services/sql-database/).
Se configura una región en espera para escenarios de conmutación por error.

![Diagrama de la arquitectura de referencia para una aplicación web PaaS](./media/ddos-best-practices/image11.png)

Azure Traffic Manager enruta las solicitudes entrantes a Application Gateway en una de las regiones. Durante las operaciones normales, enruta las solicitudes a Application Gateway en la región activa. Si esa región deja de estar disponible, Traffic Manager conmuta por error a Application Gateway en la región en espera.

Todo el tráfico de Internet con destino a la aplicación web se enruta a la [dirección IP pública de Application Gateway](/azure/application-gateway/application-gateway-web-app-overview) a través de Traffic Manager. En este escenario, App Service (Web Apps) no es directamente accesible desde el exterior y está protegido por Application Gateway. 

Se recomienda configurar la SKU de WAF de Application Gateway (modo de prevención) para protegerse contra ataques de nivel 7 (WebSocket, HTTP o HTTPS). Además, las aplicaciones web están configuradas para [aceptar tráfico solo desde la dirección IP de Application Gateway](https://azure.microsoft.com/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/).

Consulte [este artículo](/azure/architecture/reference-architectures/app-service-web-app/multi-region) para más información acerca de esta arquitectura de referencia.

### <a name="mitigation-for-non-web-paas-services"></a>Mitigación para servicios de PaaS que no son web

#### <a name="hdinsight-on-azure"></a>HDInsight en Azure

Esta arquitectura de referencia muestra cómo configurar DDoS Protection Standard para un [clúster de Azure HDInsight](/azure/hdinsight/). Asegúrese de que el clúster de HDInsight esté vinculado a una red virtual y que DDoS Protection esté habilitado en esa red virtual.

![Paneles "HDInsight" y "Configuración avanzada", con la configuración de la red virtual](./media/ddos-best-practices/image12.png)

![Selección para habilitar DDoS Protection](./media/ddos-best-practices/image13.png)

En esta arquitectura, el tráfico de Internet con destino al clúster de HDInsight se enruta a la dirección IP pública asociada con el equilibrador de carga de la puerta de enlace de HDInsight. El equilibrador de carga de la puerta de enlace envía el tráfico directamente a los nodos principales o los nodos de trabajo. Como DDoS Protection Standard está habilitado en la red virtual de HDInsight, todas las direcciones IP públicas de la red virtual obtienen protección contra DDoS en los niveles 3 y 4. Esta arquitectura de referencia puede combinarse con arquitecturas de referencia de n niveles o múltiples regiones.

Para más información acerca de esta arquitectura de referencia, consulte [Extensión de HDInsight con Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network?toc=%2fazure%2fvirtual-network%2ftoc.json).


> [!NOTE]
> Azure App Service Environment para PowerApps o API Management en redes con direcciones IP públicas no se admiten de forma nativa.

## <a name="next-steps"></a>Pasos siguientes

* [Responsabilidad compartida en la nube](shared-responsibility.md)
* [Página del producto Azure DDoS Protection](https://azure.microsoft.com/services/ddos-protection/)
* [Documentación de Azure DDoS Protection](/azure/virtual-network/ddos-protection-overview)
