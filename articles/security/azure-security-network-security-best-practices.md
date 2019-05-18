---
title: Procedimientos recomendados de seguridad de red - Microsoft Azure
description: Este artículo proporciona un conjunto de procedimientos recomendados de seguridad de la red con funcionalidades de Azure integradas.
services: security
author: TerryLanfear
manager: barbkess
editor: TomShinder
ms.assetid: 7f6aa45f-138f-4fde-a611-aaf7e8fe56d1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/05/2019
ms.author: TomSh
ms.openlocfilehash: f36658d57fb514ff279c39e9cb1644cf56341ec4
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761827"
---
# <a name="azure-best-practices-for-network-security"></a>Procedimientos recomendados de Azure para la seguridad de red
Este artículo describe una colección de procedimientos recomendados de Azure para mejorar la seguridad de red. Estos procedimientos recomendados se derivan de nuestra experiencia con las redes en Azure, y las experiencias de clientes como usted.

Para cada procedimiento recomendado, en este artículo se explica:

* Qué es el procedimiento recomendado
* Por qué le conviene habilitar este procedimiento recomendado
* Cuál podría ser el resultado si no habilita el procedimiento recomendado
* Alternativas posibles al procedimiento recomendado
* Cómo aprender a habilitar el procedimiento recomendado

Estas recomendaciones se basan en un consenso capacidades de la plataforma Azure y los conjuntos de características, tal y como aparecen en el momento en que se escribió este artículo. Las opiniones y las tecnologías cambian con el tiempo, por lo que se actualizará de forma periódica para reflejar esos cambios.

## <a name="use-strong-network-controls"></a>Use controles de red segura
Puede conectar las [máquinas virtuales y los dispositivos de Azure](https://azure.microsoft.com/services/virtual-machines/) a otros dispositivos en red, colocándolos en [redes virtuales de Azure](https://docs.microsoft.com/azure/virtual-network/). Esto es, puede conectar tarjetas de interfaz de red virtual a una red virtual para posibilitar las comunicaciones basadas en TCP/IP entre dispositivos habilitados para la red. Las máquinas virtuales conectadas a una red virtual de Azure pueden conectarse a dispositivos en la misma red virtual, en distintas redes virtuales, en Internet o, incluso, en sus propias redes locales.

Al planificar la red y la seguridad de la red, se recomienda que centralice:

- Administración de funciones de red principales, como ExpressRoute, la red virtual y subred aprovisionamiento y asignación de direcciones IP.
- Gobierno de los elementos de seguridad de red, como las funciones de aplicación virtual de red como ExpressRoute, la red virtual y subred aprovisionamiento y asignación de direcciones IP.

Si usa un conjunto común de herramientas de administración para supervisar la red y la seguridad de la red, obtenga una visibilidad clara en ambos. Una estrategia de seguridad sencilla y unificada reduce los errores porque aumenta descripción humana y la confiabilidad de la automatización.

## <a name="logically-segment-subnets"></a>Segmentación lógica de subredes
Redes virtuales de Azure son similares a las redes LAN de la red local. La idea detrás de una red virtual de Azure es que se crea una red, en función de un sola privado espacio de direcciones IP, en el que puede colocar todas las máquinas virtuales de Azure. Los espacios de direcciones IP privados están en los intervalos de clase A (10.0.0.0/8), B (172.16.0.0/12) y C (192.168.0.0/16).

Los procedimientos recomendados para segmentar lógicamente las subredes son:

**Procedimiento recomendado**: No asigne reglas con intervalos amplia de permiso (por ejemplo, permitir 0.0.0.0 y 255.255.255.255).  
**Detalles**: Asegúrese de procedimientos de solución de problemas no bajo ninguna circunstancia o prohibir la configuración de estos tipos de reglas. Estos permiten responsable de las reglas para una falsa sensación de seguridad y son con frecuencia se encuentra y aprovechar los equipos rojo.

**Procedimiento recomendado**: segmentar el mayor espacio de direcciones en las subredes.   
**Detalles**: use los principios de subred basado en [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) para crear las subredes.

**Procedimiento recomendado**: crear controles de acceso de red entre subredes. El enrutamiento entre subredes se realizará automáticamente y no es necesario configurar manualmente las tablas de enrutamiento. De forma predeterminada, no hay ningún control de acceso de red entre las subredes que se crean en una red virtual de Azure.   
**Detalles**: Use un [grupo de seguridad de red](../virtual-network/virtual-networks-nsg.md) para protegerse contra el tráfico no solicitado en subredes de Azure. Grupos de seguridad de red son dispositivos de inspección de paquetes con estado, simple que use el enfoque de 5-tupla (IP de origen, puerto de origen, IP de destino, puerto de destino y protocolo de nivel 4) para crear reglas de permitir o denegar el tráfico de red. Puede permitir o denegar el tráfico hacia y desde una sola dirección IP, hacia y desde varias direcciones IP o, incluso, hacia y desde subredes enteras.

Cuando use grupos de seguridad de red para el control de acceso de red entre subredes, puede colocar los recursos que pertenecen a la misma zona de seguridad o el rol en sus propias subredes.

**Procedimiento recomendado**: Evitar pequeñas redes virtuales y subredes para garantizar la simplicidad y flexibilidad.   
**Detalles**: La mayoría de las organizaciones agregan más recursos que inicialmente lo planeado, y volver a asignar direcciones es laboriosa. Uso de subredes pequeño agrega el valor de seguridad limitada y asignar un grupo de seguridad de red a cada subred se sobrecarga. Definir subredes ampliamente para asegurarse de que dispone de flexibilidad para el crecimiento.

**Procedimiento recomendado**: Simplificar la administración de reglas de grupo de seguridad de red mediante la definición de [grupos de seguridad de la aplicación](https://docs.microsoft.com/rest/api/virtualnetwork/applicationsecuritygroups).  
**Detalles**: Definir un grupo de seguridad de aplicaciones para las listas de direcciones IP que piensa que podrían cambiar en el futuro o utilizarse en varios grupos de seguridad de red. Asegúrese de nombre de la seguridad de la aplicación grupos claramente por lo que otros pueden comprender su contenido y el propósito.

## <a name="adopt-a-zero-trust-approach"></a>Adopte un enfoque de confiar en cero
Las redes basadas en el perímetro funcionan en la suposición de que se pueden confiar en todos los sistemas dentro de una red. Pero los empleados de hoy en día, tener acceso a los recursos de su organización desde cualquier lugar en una gran variedad de dispositivos y aplicaciones, lo que hace perimetral controles de seguridad pertinente. Directivas de Access control que se centran solo en quién pueden acceder a un recurso no son suficientes. Para dominar el equilibrio entre seguridad y productividad, los administradores de seguridad también deben tener en cuenta *cómo* se tiene acceso a un recurso.

Las redes deben ir más allá del tradicionales defensas porque las redes pueden ser vulnerables a las infracciones: un atacante puede poner en peligro un único punto de conexión dentro del límite de confianza y, a continuación, expanda rápidamente un punto de apoyo en toda la red. [Cero confianza](https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/) redes eliminan el concepto de confianza según la ubicación de red dentro de un perímetro. En su lugar, las arquitecturas de confianza cero usar notificaciones de confianza de usuario y dispositivo para obtener acceso a datos de la organización y los recursos. Nuevas iniciativas, adoptar enfoques de confianza de cero que validación la confianza en el momento de acceso.

Procedimientos recomendados son:

**Procedimiento recomendado**: Conceder acceso condicional a recursos en función del dispositivo, identidad, garantía, ubicación de red y mucho más.  
**Detalles**: [Acceso condicional de Azure AD](../active-directory/conditional-access/overview.md) permite aplicar los controles de acceso adecuado mediante la implementación de las decisiones de control de acceso automatizado según las condiciones necesarias. Para obtener más información, consulte [administrar el acceso a la administración de Azure con el acceso condicional](../role-based-access-control/conditional-access-azure-management.md).

**Procedimiento recomendado**: Habilitar el acceso de puerto solo tras la aprobación de flujo de trabajo.  
**Detalles**: Puede usar [acceso de máquina virtual just-in-time en Azure Security Center](../security-center/security-center-just-in-time.md) para bloquear el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques al tiempo que proporciona un acceso sencillo a conectarse a máquinas virtuales cuando sea necesario.

**Procedimiento recomendado**: Conceder permisos temporales para realizar tareas con privilegios, lo que impide que usuarios no autorizados o malintencionados obtengan acceso después de que los permisos han expirado. El acceso se concede solo cuando los usuarios lo necesitan.  
**Detalles**: Usar acceso just-in-time en Azure AD Privileged Identity Management o en una solución de terceros para conceder permisos para realizar tareas con privilegios.

Cero confianza es la siguiente evolución en seguridad de red. El estado de los ciberataques unidades a las organizaciones sacar la mentalidad de "presunción de brecha", pero este enfoque no debería estar limitando. Cero redes de confianza protegen los datos corporativos y recursos garantizando que las organizaciones pueden crear un área de trabajo moderno mediante el uso de tecnologías que permiten a los empleados sean productivos en cualquier momento y lugar, de ninguna manera.

## <a name="control-routing-behavior"></a>Control del comportamiento de enrutamiento
Cuando coloca una máquina virtual en una instancia de Azure Virtual Network, observará que la máquina virtual puede conectarse a cualquier otra máquina virtual de la misma red virtual, incluso si las otras máquinas virtuales están en subredes diferentes. Esto es posible porque hay una colección de rutas del sistema que están habilitadas de forma predeterminada y que permiten este tipo de comunicación. Estas rutas predeterminadas permiten que las máquinas virtuales de la misma red virtual inicien conexiones entre sí y con Internet (solo para comunicaciones salientes a Internet).

Si bien las rutas del sistema predeterminadas son útiles para muchos escenarios de implementación, habrá veces en las que preferirá personalizar la configuración de enrutamiento para las implementaciones. Puede configurar la dirección del próximo salto para que acceda a destinos específicos.

Igualmente le recomendamos que configure las [rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md) al implementar un dispositivo de seguridad para una red virtual. Trataremos este punto más adelante en la sección dedicada a [proteger los recursos de servicio de Azure críticos únicamente para las redes virtuales](azure-security-network-security-best-practices.md#secure-your-critical-azure-service-resources-to-only-your-virtual-networks).

> [!NOTE]
> Las rutas definidas por el usuario no son necesarias, y las rutas del sistema predeterminadas funcionan en la mayoría de lo casos.
>
>

## <a name="use-virtual-network-appliances"></a>Uso de aplicaciones de red virtual
Grupos de seguridad de red y enrutamiento definido por el usuario pueden proporcionar un cierto grado de seguridad de red en las capas de red y transporte de la [modelo OSI](https://en.wikipedia.org/wiki/OSI_model). Aún así, es posible que en algunas situaciones quiera o necesite habilitar la seguridad en los niveles altos de la pila. En tales situaciones, se recomienda implementar aplicaciones de seguridad de la red virtual proporcionadas por asociados de Azure.

Las aplicaciones de seguridad de la red de Azure pueden proporcionar niveles de seguridad mejorados que los que proporcionan los controles de nivel de red. Las funcionalidades de seguridad de red correspondientes a los dispositivos de seguridad de la red virtual incluyen:


* Firewalls
* Detección y prevención de intrusiones
* Administración de vulnerabilidades
* Control de aplicaciones
* Detección de anomalías basadas en la red
* Filtrado de web
* Antivirus
* Protección de redes de robots (botnets)

Para buscar dispositivos de seguridad de redes virtuales de Azure, vaya a [Azure Marketplace](https://azure.microsoft.com/marketplace/) y busque mediante las palabras clave "seguridad" y "seguridad de red".

## <a name="deploy-perimeter-networks-for-security-zones"></a>Implementar redes perimetrales para las zonas de seguridad
Una [red perimetral](https://docs.microsoft.com/azure/best-practices-network-security) (también conocida como DMZ) es un segmento de red físico o lógico que está diseñado para proporcionar un nivel de seguridad adicional entre los recursos e Internet. Los dispositivos de control de acceso de red especializados que se encuentran en el borde de una red perimetral solo permiten el tráfico deseado en la red virtual.

Las redes perimetrales son útiles porque permiten centrar la administración, supervisión, registro y generación de informes sobre los dispositivos del control de acceso a la red en el borde de la instancia de Azure Virtual Network. Una red perimetral es donde normalmente permiten denegación distribuida de prevención de servicio (DDoS), sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS), las reglas de firewall y directivas, filtrado web, antimalware de la red y mucho más. Los dispositivos de seguridad de la red se sitúan entre Internet y la instancia de Azure Virtual Network, y tienen una interfaz en ambas redes.

Aunque se trata el diseño básico de una red perimetral, hay muchos diseños diferentes, como opuesto, triple alojamiento y múltiple.

Según el concepto de confiar en cero que se ha mencionado anteriormente, se recomienda que considere la posibilidad de usar una red perimetral para todas las implementaciones de alta seguridad para mejorar el nivel de control de acceso y seguridad de red para los recursos de Azure. Puede usar Azure o una solución de terceros para proporcionar una capa adicional de seguridad entre internet y los recursos:

- Controles nativos de Azure. [Firewall de Azure](../firewall/overview.md) y [firewall de aplicaciones web en Application Gateway](../application-gateway/overview.md#web-application-firewall) ofrecen seguridad básica con un firewall con estado totalmente como servicio, alta disponibilidad integrada, escalabilidad sin restricciones en la nube, filtrado de FQDN , compatibilidad con OWASP core rule Set y sencillo programa de instalación y configuración.
- Ofertas de terceros. Búsqueda del [Azure Marketplace](https://azuremarketplace.microsoft.com/) de firewall de próxima generación (NGFW) y otras ofertas de terceros que proporcionan herramientas de seguridad conocidas y mucho mejores niveles de seguridad de red. Configuración podría ser más compleja, pero una oferta de terceros es posible que le permiten usar conjuntos de habilidades y capacidades existentes.

## <a name="avoid-exposure-to-the-internet-with-dedicated-wan-links"></a>Evitar la exposición a internet con vínculos WAN dedicados
Muchas organizaciones han elegido la ruta de TI híbrida. Con la TI híbrida, algunos de los activos de información de la empresa están en Azure y otros siguen siendo locales. En muchos casos, algunos componentes de un servicio se ejecutan en Azure, mientras que otros componentes siguen siendo locales.

En un escenario de TI híbrida, suele haber algún tipo de conectividad entre locales. Esta conectividad entre locales permite a la empresa conectar sus redes locales con las redes virtuales de Azure. Hay dos soluciones de conectividad entre locales:

* [VPN de sitio a sitio](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md). es una tecnología de confianza y bien establecida, pero que realiza la conexión a través de Internet. Ancho de banda está limitado a un máximo de aproximadamente 1,25 Gbps. VPN de sitio a sitio es una opción conveniente en algunos escenarios.
* **Azure ExpressRoute**. se recomienda que use [ExpressRoute](../expressroute/expressroute-introduction.md) para la conectividad entre locales. ExpressRoute le permite ampliar sus redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad. Con ExpressRoute, se pueden establecer conexiones a servicios de nube de Microsoft como Azure, Office 365 y Dynamics 365. ExpressRoute es una WAN dedicado de vínculo entre su ubicación local o un proveedor de hospedaje de Microsoft Exchange. Se trata de una conexión de telecomunicaciones, los datos no viajan a través de internet, por lo que no se expone a los posibles riesgos de las comunicaciones de internet.

La ubicación de la conexión de ExpressRoute puede afectar a la visibilidad de tráfico de red, confiabilidad, escalabilidad y capacidad de firewall. Deberá identificar dónde terminar de ExpressRoute en las redes existentes (local). Puede:

- Finalizar fuera del firewall (el paradigma de la red perimetral) si necesita visibilidad sobre el tráfico, si tiene que seguir una práctica de aislar los centros de datos existente, o si solamente va a colocar los recursos de extranet en Azure.
- Finalizar dentro del firewall (el paradigma de extensión de red). Se trata de la recomendación predeterminada. En todos los demás casos, se recomienda tratar Azure como un centro de datos enésimo.

## <a name="optimize-uptime-and-performance"></a>Optimización del rendimiento y el tiempo de actividad
Si un servicio está inactivo, no puede accederse a la información. Si el rendimiento es tan bajo que no se pueden utilizar los datos, podemos considerar que los datos son inaccesibles. Por lo tanto, desde una perspectiva de seguridad, necesitamos hacer todo lo posible para asegurarnos de que nuestros servicios tienen un rendimiento y un tiempo de actividad óptimos.

Un método popular y eficaz para mejorar la disponibilidad y el rendimiento es usar el equilibrio de carga. El equilibrio de carga es un método para distribuir el tráfico de la red entre los servidores que forman parte de un servicio. Por ejemplo, si tiene servidores web front-end que forman parte de su servicio, puede usar el equilibrio de carga para distribuir el tráfico entre ellos.

Esta distribución del tráfico aumenta la disponibilidad, ya que si uno de los servidores web deja de estar disponible, el equilibrio de carga deja de enviarle tráfico y lo redirige a los servidores que aún están en línea. El equilibrio de carga también mejora el rendimiento, ya que la sobrecarga del procesador, la red y la memoria para atender a las solicitudes se distribuye entre todos los servidores con carga equilibrada.

Se recomienda usar el equilibrio de carga siempre que se pueda y, según sea adecuado para los servicios. Estos son los escenarios en el nivel de Azure Virtual Network y el nivel global, junto con las opciones de equilibrio de carga para cada uno.

**Escenario**: ahora tiene una aplicación que:

- Requiere solicitudes de la misma sesión de usuario o cliente para llegar a la misma máquina virtual de back-end. Ejemplos de esto serían las aplicaciones del carro de la compra y los servidores de correo web.
- Como solo acepta una conexión segura, la comunicación sin cifrar con los servidores no es una opción aceptable.
- Es necesario que varias solicitudes HTTP en la misma conexión TCP de ejecución prolongada se enruten a servidores de back-end diferentes o su carga se equilibre entre estos.

**Opción de equilibrio de carga**: use [Azure Application Gateway](../application-gateway/application-gateway-introduction.md), que es un equilibrador de carga del tráfico de web HTTP. Application Gateway admite el cifrado SSL de un extremo a otro y la [terminación SSL](../application-gateway/application-gateway-introduction.md) en la puerta de enlace. A continuación, los servidores web pueden librarse de la sobrecarga de cifrado y descifrado y del tráfico que fluye sin encriptar a los servidores de back-end.

**Escenario**: es necesario equilibrar la carga de las conexiones entrantes de Internet entre los servidores situados en una instancia de Azure Virtual Network. Los escenarios surgen cuando:

- Tiene aplicaciones sin estado que acepten solicitudes entrantes de Internet.
- No se requieren sesiones temporales o descargas de SSL. Las sesiones temporales son un método que se usa con el equilibrio de carga de la aplicación, para lograr la afinidad del servidor.

**Opción de equilibrio de carga**: use Azure Portal para [crear un equilibrador de carga externo](../load-balancer/quickstart-create-basic-load-balancer-portal.md) que distribuya las solicitudes entrantes entre varias máquinas virtuales para proporcionar un mayor nivel de disponibilidad.

**Escenario**: tendrá que equilibrar la carga de las conexiones de las máquinas virtuales que no estén en Internet. En la mayoría de los casos, los dispositivos se encargan de iniciar en una instancia de Azure Virtual Network las conexiones que se aceptan para el equilibrio de carga, como instancias de SQL Server o servidores web internos.   
**Opción de equilibrio de carga**: use Azure Portal para [crear un equilibrador de carga interno](../load-balancer/quickstart-create-basic-load-balancer-powershell.md) que distribuya las solicitudes entrantes entre varias máquinas virtuales para proporcionar un mayor nivel de disponibilidad.

**Escenario**: necesita conseguir un equilibrio de carga global ya que:

- Tiene una solución en la nube que se distribuye ampliamente en varias regiones y requiere el nivel más alto de tiempo de actividad (o disponibilidad) posible.
- Necesita el nivel más alto de tiempo de actividad para asegurarse de que el servicio está disponible incluso si todo un centro de datos deja de funcionar.

**Opción de equilibrio de carga**: use Azure Traffic Manager. Traffic Manager le permite equilibrar la carga de las conexiones a los servicios, en función de la ubicación del usuario.

Por ejemplo, si el usuario realiza una solicitud a su servicio desde la Unión Europea, la conexión se dirige a los servicios situados en un centro de datos de la Unión Europea. Esta parte del equilibrio de carga global del Administrador de tráfico ayuda a mejorar el rendimiento, ya que la conexión al centro de datos más cercano es más rápida que a los centros de datos que están lejos.

## <a name="disable-rdpssh-access-to-virtual-machines"></a>Deshabilitar el acceso RDP/SSH a las máquinas virtuales
Es posible obtener acceso a Azure Virtual Machines mediante los protocolos de [Escritorio remoto](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) (RDP) y [Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (SSH). Estos protocolos le permiten administrar máquinas virtuales desde ubicaciones remotas y son los protocolos estándar que se usan en el procesamiento de los centros de datos.

El posible problema de seguridad al usar estos protocolos a través de Internet, es que los atacantes pueden utilizar diversas técnicas [de fuerza bruta](https://en.wikipedia.org/wiki/Brute-force_attack) para obtener acceso a Azure Virtual Machines. Una vez que los atacantes obtienen acceso, pueden utilizar la máquina virtual como punto de inicio para poner en peligro otros equipos de la red virtual o incluso atacar dispositivos en red fuera de Azure.

Por este motivo, se recomienda deshabilitar el acceso directo de RDP y SSH a Azure Virtual Machines desde Internet. Cuando se deshabilita el acceso directo de RDP y SSH desde Internet, tiene otras opciones que puede utilizar para acceder a estas máquinas virtuales y así administrarlas de forma remota.

**Escenario**: habilite una conexión de un solo usuario a una instancia de Azure Virtual Network a través de Internet.   
**Opción**: [VPN de punto a sitio](../vpn-gateway/vpn-gateway-point-to-site-create.md) es otro término para una conexión cliente/servidor de VPN con acceso remoto. Una vez establecida la conexión de punto a sitio, el usuario podrá usar RDP o SSH para conectarse a cualquier máquina virtual situada en la instancia de Azure Virtual Network a la que el usuario se conectó mediante la VPN de punto a sitio. Con esto, se supone que el usuario tiene permiso para obtener acceso a dichas máquinas virtuales.

La VPN de punto a sitio es más segura que las conexiones de RDP o SSH directas, ya que el usuario tiene que autenticarse dos veces antes de conectarse a una máquina virtual. En primer lugar, el usuario debe autenticarse (y ser autorizado) para poder establecer la conexión VPN de punto a sitio. En segundo lugar, el usuario debe autenticarse (y ser autorizado) para poder establecer la sesión RDP o SSH.

**Escenario**: permitir que los usuarios de la red local se conecten a máquinas virtuales en Azure Virtual Network.   
**Opción**: una [VPN de sitio a sitio](../vpn-gateway/vpn-gateway-site-to-site-create.md) conecta toda una red a otra a través de Internet. Puede usar una VPN de sitio a sitio para conectar su red local a una instancia de Azure Virtual Network. Los usuarios de su red local se pueden conectar mediante el protocolo RDP o SSH, a través de la conexión VPN de sitio a sitio. No debe permitir el acceso directo de RDP o SSH a través de Internet.

**Escenario**: use un vínculo WAN dedicado para ofrecer una funcionalidad similar a la VPN de sitio a sitio.   
**Opción**: use [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/). Proporciona funcionalidades similares a la VPN de sitio a sitio. Las principales diferencias son:

- El vínculo WAN dedicado no recorre Internet.
- Los vínculos WAN dedicados suelen ser más estables y eficaces.

## <a name="secure-your-critical-azure-service-resources-to-only-your-virtual-networks"></a>Proteja los recursos de servicio de Azure críticos únicamente en las redes virtuales
Use los puntos de conexión de servicio de red virtual para extender el espacio de direcciones privadas de la red virtual y la identidad de la red virtual a los servicios de Azure, a través de una conexión directa. Los puntos de conexión permiten proteger los recursos de servicio de Azure críticos únicamente para las redes virtuales. El tráfico desde la red virtual al servicio de Azure siempre permanece en la red troncal de Microsoft Azure.

Los puntos de conexión de servicio proporcionan las siguientes ventajas:

- **Seguridad mejorada para los recursos de servicio de Azure**: Con los puntos de conexión de servicio, se pueden proteger los recursos del servicio de Azure para la red virtual. Si protege los recursos del servicio en una red virtual, mejorará la seguridad al quitar totalmente el acceso público a través de Internet a estos recursos y al permitir el tráfico solo desde la red virtual.
- **Enrutamiento óptimo para el tráfico del servicio de Azure desde la red virtual**: las rutas de la red virtual que fuerzan el tráfico de Internet a las aplicaciones virtuales o locales, conocidas como tunelización forzada, también fuerzan el tráfico del servicio de Azure para realizar la misma ruta que el tráfico de Internet. Los puntos de conexión de servicio proporcionan un enrutamiento óptimo al tráfico de Azure.

  Los puntos de conexión siempre toman el tráfico del servicio directamente de la red virtual al servicio en la red troncal de Azure. Si mantiene el tráfico en la red troncal de Azure, podrá seguir auditando y supervisando el tráfico saliente de Internet desde las redes virtuales, a través de la tunelización forzada, sin que ello afecte al tráfico del servicio. Obtenga más información sobre las [rutas definidas por el usuario y la tunelización forzada](../virtual-network/virtual-networks-udr-overview.md).

- **Fácil de configurar con menos sobrecarga de administración**: ya no necesita direcciones IP públicas y reservadas en sus redes virtuales para proteger los recursos de Azure a través de una dirección IP del firewall. No hay ningún dispositivo NAT o de puerta de enlace necesario para configurar los puntos de conexión de servicio. Los puntos de conexión de servicio se pueden configurar con un simple clic en una subred. No hay sobrecarga adicional para mantener los puntos de conexión.

Para obtener más información sobre los puntos de conexión de servicio y sobre los servicios de Azure y las regiones en las que están disponibles los puntos de conexión de servicio, consulte [Puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md).

## <a name="next-steps"></a>Pasos siguientes
Vea [Patrones y procedimientos recomendados de seguridad en Azure](security-best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.
