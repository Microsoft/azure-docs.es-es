---
title: 'Azure Front Door: preguntas más frecuentes'
description: Esta página proporciona respuestas a las preguntas más frecuentes acerca de Azure Front Door.
services: frontdoor
documentationcenter: ''
author: sohamnchatterjee
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/13/2020
ms.author: sohamnc
ms.openlocfilehash: e1b8c432673712b637d51a64ce87b4eb2131be7d
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760420"
---
# <a name="frequently-asked-questions-for-azure-front-door"></a>Preguntas más frecuentes sobre Azure Front Door

En este artículo se responden preguntas comunes sobre la funcionalidad y las características de Azure Front Door. Si no encuentra una respuesta a su pregunta, póngase en contacto con nosotros mediante los siguientes canales (en orden incremental):

1. La sección Comentarios de este artículo.
2. [UserVoice de Azure Front Door](https://feedback.azure.com/forums/217313-networking?category_id=345025).
3. **Soporte técnico de Microsoft:** Para crear una solicitud de soporte técnico, en Azure Portal, vaya a la pestaña **Ayuda**, seleccione el botón **Ayuda y soporte técnico** y elija **Nueva solicitud de soporte técnico**.

## <a name="general"></a>General

### <a name="what-is-azure-front-door"></a>¿Qué es Azure Front Door?

Azure Front Door es una red de entrega de aplicaciones (ADN) como servicio que ofrece diversas funcionalidades de equilibrio de carga de capa 7 para sus aplicaciones. Proporciona aceleración de sitios dinámicos (DSA), junto con equilibrio de carga global con conmutación por error casi en tiempo real. Es un servicio altamente disponible y escalable que está completamente administrado por Azure.

### <a name="what-features-does-azure-front-door-support"></a>¿Qué características admite Azure Front Door?

Azure Front Door admite la aceleración de sitios dinámicos (DSA), la descarga de TLS/SSL y TLS de un extremo a otro, firewall de aplicaciones web, afinidad de sesión basada en cookies, enrutamiento basado en ruta de dirección URL, certificados gratuitos y administración de múltiples dominios, entre otras características. Para obtener una lista completa de las características admitidas, consulte [Overview of Azure Front Door](front-door-overview.md) (Información general de Azure Front Door).

### <a name="what-is-the-difference-between-azure-front-door-and-azure-application-gateway"></a>¿Cuál es la diferencia entre Azure Front Door y Azure Application Gateway?

Si bien tanto Front Door como Application Gateway son equilibradores de carga de capa 7 (HTTP/HTTPS), la principal diferencia es que Front Door es un servicio global, mientras que Application Gateway es un servicio regional. Front Door puede equilibrar la carga entre las diferentes unidades de escalado/clústeres/unidades de marca entre regiones, y Application Gateway, por su parte, le permite equilibrar la carga entre las máquinas virtuales y contenedores, entre otros, que se encuentran dentro de la unidad de escalado.

### <a name="when-should-we-deploy-an-application-gateway-behind-front-door"></a>¿Cuándo se debe implementar una instancia de Application Gateway detrás de Front Door?

Los principales escenarios en que se debería usar Application Gateway detrás de Front Door son los siguientes:

- Front Door puede realizar el equilibrio de carga basado en rutas solo a nivel global, pero si se desea equilibrar la carga del tráfico incluso a nivel de la red virtual, debería utilizarse Application Gateway.
- Puesto que Front Door no funciona en un nivel de máquina virtual o el contenedor, no puede hacer purga de conexiones. Por contra, Application Gateway sí le permite hacer purga de conexiones. 
- Con una instancia de Application Gateway detrás de Front Door, es posible lograr el 100 % de la descarga de TLS/SSL y enrutar solo solicitudes HTTP dentro de su red virtual (VNET).
- Tanto Front Door como Application Gateway admiten la afinidad de la sesión. Mientras que Front Door puede dirigir el tráfico posterior desde una sesión de usuario en el mismo clúster o back-end en una región determinada, Application Gateway puede establecer la afinidad del tráfico al mismo servidor en el clúster.  

### <a name="can-we-deploy-azure-load-balancer-behind-front-door"></a>¿Es posible implementar Azure Load Balancer detrás de Front Door?

Azure Front Door necesita una VIP pública o un nombre DNS disponible públicamente al que enrutar el tráfico. La implementación de Azure Load Balancer detrás de Front Door es un caso de uso común.

### <a name="what-protocols-does-azure-front-door-support"></a>¿Qué protocolos admite Azure Front Door?

Azure Front Door admite HTTP, HTTPS y HTTP/2.

### <a name="how-does-azure-front-door-support-http2"></a>¿Cómo admite HTTP/2 el servicio Azure Front Door?

La compatibilidad con el protocolo HTTP/2 está disponible únicamente para los clientes que se conectan a Azure Front Door. La comunicación con los back-end en el grupo de back-end se produce a través de HTTP/1.1. La compatibilidad con HTTP/2 está habilitada de forma predeterminada.

### <a name="what-resources-are-supported-today-as-part-of-backend-pool"></a>¿Qué recursos son compatibles actualmente como parte del grupo de back-end?

Los grupos de back-end pueden estar compuestos de instancias de Storage, aplicaciones web, Kubernetes o cualquier otro nombre de host personalizado que tenga conectividad pública. Azure Front Door requiere que los back-end se definan a través de una dirección IP pública o un nombre de host DNS que pueda resolverse públicamente. Los miembros de grupos de back-end pueden estar entre zonas, regiones e incluso fuera de Azure, siempre y cuando dispongan de conectividad pública.

### <a name="what-regions-is-the-service-available-in"></a>¿En qué regiones está disponible el servicio?

Azure Front Door es un servicio global y no está asociado a ninguna región de Azure específica. La única ubicación que debe especificar al crear una instancia de Front Door es la ubicación del grupo de recursos, que es básicamente dónde se almacenarán los metadatos para el grupo de recursos. El propio recurso de Front Door se crea como un recurso global, y la configuración se implementa globalmente en todos los puntos de presencia. 

### <a name="what-are-the-pop-locations-for-azure-front-door"></a>¿Cuáles son las ubicaciones de los puntos de presencia para Azure Front Door?

Azure Front Door tiene la misma lista de ubicaciones de puntos de presencia que Azure CDN de Microsoft. Para obtener una lista completa de nuestros puntos de presencia, consulte [Azure CDN POP locations from Microsoft](https://docs.microsoft.com/azure/cdn/cdn-pop-locations) (Ubicaciones de puntos de presencia de Azure CDN de Microsoft).

### <a name="is-azure-front-door-a-dedicated-deployment-for-my-application-or-is-it-shared-across-customers"></a>¿Azure Front Door es una implementación dedicada para mi aplicación o se comparte entre clientes?

Azure Front Door es un servicio multiinquilino distribuido globalmente. Por lo tanto, la infraestructura de Front Door se comparte entre todos sus clientes. Sin embargo, cuando crea un perfil de Front Door, define la configuración específica necesaria para la aplicación y ninguno de los campos realizados a su Front Door afectan a otras configuraciones de Front Door.

### <a name="is-http-https-redirection-supported"></a>¿Se admite el redireccionamiento HTTP->HTTPS?

Sí. De hecho, el servicio Azure Front Door admite la redirección de cadenas de host, ruta de acceso y consulta, así como de parte de la URL. Obtenga más información sobre la [redirección de URL](front-door-url-redirect.md). 

### <a name="in-what-order-are-routing-rules-processed"></a>¿En qué orden se procesan las reglas de enrutamiento?

Las rutas de Front Door no están ordenadas y se selecciona una ruta específica en función de la mejor coincidencia. Obtenga más información sobre [cómo hace coincidir Front Door las solicitudes con una regla de enrutamiento](front-door-route-matching.md).

### <a name="how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door"></a>¿Cómo puedo hacer que Azure Front Door sea el único que tenga acceso a mi back-end?

Para que la aplicación solo acepte tráfico procedente su instancia específica de Front Door, debe configurar listas de control de acceso de IP para el back-end y, a continuación, restringir el tráfico del back-end al valor específico del encabezado "X-Azure-FDID" enviado por Front Door. Estos pasos se detallan a continuación:

- Configure la creación de listas de control de acceso de IP de los back-end para que acepten tráfico únicamente del espacio de direcciones IP de back-end de Azure Front Door y de los servicios de infraestructura de Azure. Consulte los detalles de IP siguientes para la creación de listas de control de acceso en el back-end:
 
    - Consulte la sección *AzureFrontDoor.Backend* en [Etiquetas de servicio e intervalos de direcciones IP de Azure](https://www.microsoft.com/download/details.aspx?id=56519) para el intervalo de direcciones IP del back-end IPv4 de Front Door. También puede usar la etiqueta de servicio *AzureFrontDoor.Backend* en sus[grupos de seguridad en red](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules).
    - El espacio IP de back-end **IPv6** de Front Door, si está cubierto en la etiqueta de servicio, no se incluye en el archivo JSON de intervalos IP de Azure. Si busca un intervalo de direcciones IPv6 explícito, actualmente, está limitado a `2a01:111:2050::/44`
    - [Servicios de infraestructura básica](https://docs.microsoft.com/azure/virtual-network/security-overview#azure-platform-considerations) de Azure través de direcciones IP de host virtualizado: `168.63.129.16` y `169.254.169.254`

    > [!WARNING]
    > El espacio de back-end IP de Front Door puede cambiar más adelante; sin embargo, nos aseguraremos de que antes de que eso suceda nos hayamos integrado con los [intervalos de IP y etiquetas de servicio de Azure](https://www.microsoft.com/download/details.aspx?id=56519). Recomendamos que se suscriba a los [intervalos de IP y etiquetas de servicio de Azure](https://www.microsoft.com/download/details.aspx?id=56519) para conocer los cambios o actualizaciones.

-    Realice una operación GET en Front Door con la versión de API `2020-01-01` o superior. En la llamada API, busque el campo `frontdoorID`. Filtre por el encabezado entrante "**X-Azure-FDID**" que envió Front Door al back-end con el valor del campo `frontdoorID`. Encontrará el valor de `Front Door ID` también en la sección de información general de la página del portal de Front Door. 

### <a name="can-the-anycast-ip-change-over-the-lifetime-of-my-front-door"></a>¿Puede cambiar la dirección IP de difusión por proximidad durante la vigencia de mi instancia de Front Door?

La dirección IP de difusión por proximidad de su instancia de Front Door normalmente no debería cambiar y es posible que permanezca estática durante la vigencia de Front Door. Sin embargo, esto no se puede **garantizar**. No asuma ninguna dependencia directa de la dirección IP.

### <a name="does-azure-front-door-support-static-or-dedicated-ips"></a>¿Azure Front Door admite direcciones IP estáticas o dedicadas?

No, Azure Front Door actualmente no admite direcciones IP de difusión por proximidad de front-end estáticas o dedicadas. 

### <a name="does-azure-front-door-support-x-forwarded-for-headers"></a>¿Azure Front Door admite encabezados x-forwarded-for?

Sí, Azure Front Door admite los encabezados X-Forwarded-For, X-Forwarded-Host y X-Forwarded-Proto. Para X-Forwarded-For, si el encabezado ya estaba presente, Front Door le agrega la dirección IP del socket del cliente. En caso contrario, agrega el encabezado con la dirección IP del socket del cliente como valor. En el caso de For X-Forwarded-Host y X-Forwarded-Proto, el valor se invalida.

Obtenga más información sobre los [encabezados HTTP que admite Front Door](front-door-http-headers-protocol.md).  

### <a name="how-long-does-it-take-to-deploy-an-azure-front-door-does-my-front-door-still-work-when-being-updated"></a>¿Cuánto tiempo se tarda en implementar una instancia de Azure Front Door? ¿Mi Front Door sigue funcionando mientras se actualiza?

La creación de una instancia de Front Door o cualquier actualización a la que se someta una instancia de Front Door tarda entre 3 y 5 minutos en implementarse globalmente. Esto significa que, en un tiempo de entre 3 y 5 minutos, la configuración de Front Door se implementará en sus puntos de presencia de manera global.

Nota: La personalización de las actualizaciones de certificados TLS/SSL tarda unos 30 minutos en implementarse globalmente.

Las actualizaciones de las rutas o los grupos de back-end se realizan sin problemas y no provocarán ningún tiempo de inactividad (si la nueva configuración es correcta). Las actualizaciones de certificados también son atómicas y no provocarán ninguna interrupción, a menos que se cambie de "AFD Managed" (Administrado por AFD) a "Use your own cert" (Usar certificado propio) o viceversa.


## <a name="configuration"></a>Configuración

### <a name="can-azure-front-door-load-balance-or-route-traffic-within-a-virtual-network"></a>¿Puede Azure Front Door equilibrar la carga o enrutar el tráfico dentro de una red virtual?

Azure Front Door (AFD) requiere una dirección IP pública o un nombre DNS que pueda resolverse públicamente para enrutar el tráfico. Por lo tanto, la respuesta es que ninguna instancia de AFD puede enrutar directamente dentro de una red virtual, pero el uso de Application Gateway o Azure Load Balancer en medio resolverá este escenario.

### <a name="what-are-the-various-timeouts-and-limits-for-azure-front-door"></a>¿Cuáles son los distintos tiempos de espera y límites del servicio Azure Front Door?

Conozca todos los [tiempos de espera y límites documentados del servicio Azure Front Door](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-front-door-service-limits).

## <a name="performance"></a>Rendimiento

### <a name="how-does-azure-front-door-support-high-availability-and-scalability"></a>¿Cómo admite Azure Front Door la alta disponibilidad y la escalabilidad?

Azure Front Door es una plataforma multiinquilino distribuida globalmente con grandes volúmenes de capacidad para satisfacer las necesidades de escalabilidad de la aplicación. Ofrecida desde el perímetro de la red global de Microsoft, Front Door proporciona una funcionalidad de equilibrio de carga global que le permite conmutar por error la aplicación entera o incluso microservicios individuales a través de regiones o nubes diferentes.

## <a name="tls-configuration"></a>Configuración de TLS

### <a name="what-tls-versions-are-supported-by-azure-front-door"></a>¿Qué versiones de TLS son compatibles con Azure Front Door?

Todos los perfiles de Front Door creados después de septiembre de 2019 usan TLS 1.2 como valor mínimo predeterminado.

Front Door admite las versiones 1.0, 1.1 y 1.2 de TLS. Aún no se admite TLS 1.3.

### <a name="what-certificates-are-supported-on-azure-front-door"></a>¿Qué certificados se admiten en Azure Front Door?

Para habilitar el protocolo HTTPS para distribuir de forma segura el contenido en un dominio personalizado de Front Door, puede optar por usar un certificado administrado por Azure Front Door o usar su propio certificado.
La opción administrada de Front Door aprovisiona un certificado TLS/SSL estándar mediante Digicert y se almacena en Key Vault de Front Door. Si decide usar su propio certificado, puede incorporar un certificado de una entidad de certificación compatible y puede ser un TLS estándar, un certificado de validación extendido o incluso un certificado comodín. No se admiten certificados autofirmados. Aprenda a [habilitar HTTPS para un dominio personalizado](https://aka.ms/FrontDoorCustomDomainHTTPS).

### <a name="does-front-door-support-autorotation-of-certificates"></a>¿Front Door admite la rotación automática de certificados?

Para la opción de certificados administrados por Front Door, Front Door rota automáticamente los certificados. Si está usando un certificado administrado por Front Door y nota que la fecha de expiración del certificado es inferior a 60 días, cree una incidencia de soporte técnico.
</br>La rotación automática no es compatible con su propio certificado TLS/SSL personalizado. Así como lo configuró la primera vez para un dominio personalizado determinado, tendrá que dirigir Front Door a la versión de certificado correcta en su almacén de claves y asegurarse de que la entidad de servicio para Front Door aún tiene acceso al almacén de claves. Esta operación de lanzamiento de certificados actualizados de Front Door es atómica y no afecta la producción siempre que el nombre de sujeto o SAN para el certificado no cambie.

### <a name="what-are-the-current-cipher-suites-supported-by-azure-front-door"></a>¿Cuáles son los conjuntos de cifrado actuales que admite Azure Front Door?

Para TLS 1.2, se admiten los siguientes conjuntos de cifrado: 

- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256

Al usar dominios personalizados con TLS 1.0/1.1 habilitado, se admiten los siguientes conjuntos de cifrado:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_GCM_SHA384
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA256
- TLS_RSA_WITH_AES_128_CBC_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA
- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384

### <a name="can-i-configure-tls-policy-to-control-tls-protocol-versions"></a>¿Puedo configurar la directiva TLS para controlar las versiones del protocolo TLS?

Puede configurar una versión de TLS mínima en Azure Front Door en la configuración HTTPS del dominio personalizado desde Azure Portal o la [API REST de Azure](https://docs.microsoft.com/rest/api/frontdoorservice/frontdoor/frontdoors/createorupdate#minimumtlsversion). Actualmente, puede elegir entre 1.0 y 1.2.

### <a name="can-i-configure-front-door-to-only-support-specific-cipher-suites"></a>¿Puedo configurar Front Door para admitir únicamente determinados conjuntos de cifrado?

No, no se admite la configuración de Front Door para determinados conjuntos de cifrado. Sin embargo, puede obtener su propio certificado TLS/SSL personalizado de la entidad de certificación (por ejemplo, VeriSign, Entrust o DigiCert) y hacer que se marquen conjuntos de cifrado específicos en el certificado al generarlo. 

### <a name="does-front-door-support-ocsp-stapling"></a>¿Front Door admite la asociación OCSP?

Sí, de forma predeterminada, la asociación OCSP es compatible con Front Door y no se requiere ninguna configuración.

### <a name="does-azure-front-door-also-support-re-encryption-of-traffic-to-the-backend"></a>¿Admite Azure Front Door también volver a cifrar el tráfico dirigido al back-end?

Sí, Azure Front Door admite la descarga de TLS/SSL y TLS de extremo a extremo, lo cual permite volver a cifrar el tráfico dirigido al back-end. De hecho, puesto que las conexiones con el back-end se producen a través de la dirección IP pública, se recomienda que configure Front Door para que use HTTPS como protocolo de reenvío.

### <a name="does-front-door-support-self-signed-certificates-on-the-backend-for-https-connection"></a>¿Admite Front Door los certificados autofirmados en el back-end para la conexión HTTPS?

No, los certificados autofirmados no se admiten en Front Door y la restricción se aplica a ambos:

1. **Back-ends**: No se pueden usar certificados autofirmados cuando se reenvía el tráfico como HTTPS o sondeos de estado HTTPS o se rellena la caché de origen para las reglas de enrutamiento con el almacenamiento en caché habilitado.
2. **Frontend**: No se pueden usar certificados autofirmados al usar su propio certificado TLS/SSL personalizado para habilitar HTTPS en el dominio personalizado.

### <a name="why-is-https-traffic-to-my-backend-failing"></a>¿Por qué se produce un error del tráfico HTTPS al back-end?

Para tener conexiones HTTPS correctas al back-end, ya sea para sondeos de estado o para reenviar solicitudes, hay dos motivos posibles por los que el tráfico HTTPS puede generar un error:

1. **Error de coincidencia de nombre del firmante del certificado**: En el caso de las conexiones HTTPS, Front Door espera que el back-end presente el certificado de una entidad de certificación válida con nombres de firmante que coincidan con el nombre de host del back-end. Por ejemplo, si el nombre de host del back-end se establece en `myapp-centralus.contosonews.net` y el certificado que el back-end presenta durante el protocolo de enlace TLS no tiene `myapp-centralus.contosonews.net` ni `*myapp-centralus*.contosonews.net` en el nombre del firmante, Front Door rechazará la conexión y devolverá un error. 
    1. **Solución**: Aunque no se recomienda desde el punto de vista del cumplimiento, una forma de evitar este error es deshabilitar la comprobación del nombre del firmante del certificado para Front Door. Está presente en Configuración, en Azure Portal, y en BackendPoolsSettings, en la API.
2. **Certificado de hospedaje de back-end de CA no válida**: Solo se pueden usar certificados de [entidades de certificación válidas](/azure/frontdoor/front-door-troubleshoot-allowed-ca) en el back-end con Front Door. No se permiten certificados de entidades de certificación internas ni certificados autofirmados.

## <a name="diagnostics-and-logging"></a>Diagnósticos y registro

### <a name="what-types-of-metrics-and-logs-are-available-with-azure-front-door"></a>¿Qué tipos de registros y métricas están disponibles con Azure Front Door?

Para obtener información sobre los registros y otras funcionalidades de diagnóstico, consulte [Monitoring metrics and logs for Front Door](front-door-diagnostics.md) (Supervisión de métricas y registros para Front Door).

### <a name="what-is-the-retention-policy-on-the-diagnostics-logs"></a>¿Qué es la directiva de retención en los registros de diagnóstico?

Los registros de diagnóstico fluyen hacia la cuenta de almacenamiento de los clientes y estos pueden establecer la directiva de retención según sus preferencias. Los registros de diagnóstico también se pueden enviar a un centro de eventos o a registros de Azure Monitor. Para más información, consulte el artículo sobre [el diagnóstico de Azure Front Door](front-door-diagnostics.md).

### <a name="how-do-i-get-audit-logs-for-azure-front-door"></a>¿Cómo puedo obtener registros de auditoría para Azure Front Door?

Los registros de auditoría están disponibles para Azure Front Door. En el portal, haga clic en **Registro de actividad** en la hoja de menú de su instancia de Front Door para acceder al registro de auditoría. 

### <a name="can-i-set-alerts-with-azure-front-door"></a>¿Puedo establecer alertas con Azure Front Door?

Sí, Azure Front Door admite las alertas. Las alertas se configuran en métricas. 

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
