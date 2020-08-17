---
title: Publicación de aplicaciones locales con Azure AD Application Proxy
description: Entienda el uso de Application Proxy para publicar aplicaciones web locales de forma externa para usuarios remotos. Obtenga información sobre la arquitectura, los conectores, los métodos de autenticación y las ventajas de seguridad de Application Proxy.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 05/31/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 68ccf3518a7011c0c39003d8e447a4e1d233fd91
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88164587"
---
# <a name="using-azure-ad-application-proxy-to-publish-on-premises-apps-for-remote-users"></a>Uso de Azure AD Application Proxy para publicar aplicaciones locales para usuarios remotos

Azure Active Directory (Azure AD) ofrece muchas funcionalidades para proteger a los usuarios, las aplicaciones y los datos en la nube y en el entorno local. En particular, la característica Azure AD Application Proxy la pueden implementar profesionales de TI que quieran publicar aplicaciones web locales de forma externa. Los usuarios remotos que necesitan acceso a aplicaciones internas pueden acceder a ellas de forma segura.

La capacidad de acceder de forma segura a las aplicaciones internas desde fuera de la red se vuelve aún más importante en el área de trabajo moderna. Con escenarios como BYOD (Bring Your Own Device) y dispositivos móviles, los profesionales de TI se enfrentan al desafío de lograr dos objetivos:

* Capacitar a los usuarios finales para que sean productivos en cualquier momento y en cualquier lugar
* Proteger los recursos corporativos en todo momento

Muchas organizaciones creen que tienen el control y que están protegidas cuando los recursos se encuentran dentro de los límites de las redes corporativas. Pero en el área de trabajo digital de hoy en día, esa frontera se ha ampliado con dispositivos móviles administrados, así como con recursos y servicios en la nube. Ahora necesita administrar la complejidad de proteger las identidades de los usuarios y los datos almacenados en los dispositivos y aplicaciones.

Quizás ya esté utilizando Azure AD para administrar usuarios en la nube que necesitan acceder a Office 365 y otras aplicaciones SaaS, así como aplicaciones web hospedadas en el entorno local. Si ya tiene Azure AD, puede utilizarlo como un plano de control para permitir un acceso perfecto y seguro a las aplicaciones locales. O, tal vez todavía está contemplando pasarse a la nube. Si es así, puede comenzar su viaje a la nube al implementar Application Proxy y dar el primer paso hacia la construcción de una sólida base de identidad.

Aunque no de manera exhaustiva, en la lista siguiente se muestran algunas de las cosas que puede habilitar al implementar Application Proxy en un escenario de coexistencia híbrida:

* Publicar aplicaciones web locales externamente de forma simplificada sin una red perimetral
* Admitir el inicio de sesión único (SSO) en todos los dispositivos, recursos y aplicaciones en la nube y locales
* Admitir la autenticación multifactor para las aplicaciones en la nube y locales
* Aprovechar rápidamente las características de la nube con la seguridad de Microsoft Cloud
* Centralizar la administración de las cuentas de usuario
* Centralizar el control de la identidad y seguridad
* Agregar o quitar automáticamente el acceso del usuario a las aplicaciones en función de la pertenencia a grupos

En este artículo se explica cómo Azure AD y Application Proxy ofrecen a los usuarios remotos una experiencia de inicio de sesión único (SSO). Los usuarios se conectan de forma segura a las aplicaciones locales sin una VPN ni servidores de doble conexión ni reglas de firewall. Este artículo le ayudará a comprender cómo Application Proxy lleva las funcionalidades y ventajas de seguridad de la nube a las aplicaciones web locales. También se describe la arquitectura y las topologías que son posibles.

## <a name="remote-access-in-the-past"></a>Acceso remoto en el pasado

Anteriormente, el plano de control para proteger los recursos internos de los atacantes y facilitar el acceso a los usuarios remotos se encontraba en la red perimetral. Pero las soluciones VPN y de proxy inverso implementadas en la red perimetral que utilizan los clientes externos para acceder a los recursos corporativos no son adecuadas para el mundo de la nube. Normalmente, tienen las siguientes desventajas:

* Costos de hardware
* Mantenimiento de la seguridad (aplicación de revisiones, supervisión de puertos, etc.)
* Autenticación de los usuarios en el perímetro
* Autenticación de los usuarios a los servidores web en la red perimetral
* Mantenimiento del acceso de VPN para usuarios remotos con la distribución y configuración del software cliente VPN. Además, el mantenimiento de los servidores unidos al dominio en la red perimetral, que pueden ser vulnerables a ataques del exterior.

En la nube de hoy en día, Azure AD es el más adecuado para controlar quién y qué entra en su red. Azure AD Application Proxy se integra con las tecnologías modernas de autenticación y basadas en la nube, como las aplicaciones SaaS y los proveedores de identidades. Esta integración permite a los usuarios el acceso a las aplicaciones desde cualquier lugar. Application Proxy no solo es más adecuado para el área de trabajo digital actual, sino que es más seguro que las soluciones VPN y de proxy inverso, y más fácil de implementar. Los usuarios remotos pueden tener acceso a sus aplicaciones locales del mismo modo que tienen acceso a Office 365 y otras aplicaciones de SaaS integradas con Azure AD. No es necesario cambiar o actualizar las aplicaciones para trabajar con el Proxy de aplicación. Es más, Application Proxy no necesita que se abran conexiones entrantes a través del firewall. Con Application Proxy, simplemente se configura y se olvida.

## <a name="the-future-of-remote-access"></a>El futuro de acceso remoto

En el área de trabajo digital actual, los usuarios trabajan en cualquier lugar con múltiples dispositivos y aplicaciones. Lo único constante es la identidad del usuario. Es por eso que el primer paso hacia una red segura hoy en día es utilizar las funcionalidades de [administración de identidades de Azure AD](https://docs.microsoft.com/azure/security/security-identity-management-overview) como su plano de control de seguridad. Este modelo que utiliza la identidad como su plano de control consta normalmente de los siguientes componentes:

* Un proveedor de identidades para realizar el seguimiento de los usuarios y de la información relacionada con el usuario.
* Un directorio de dispositivos para mantener una lista de los dispositivos que tienen acceso a los recursos corporativos. En este directorio se incluye la información del dispositivo correspondiente (por ejemplo, tipo de dispositivo, integridad, etc.).
* Un servicio de evaluación de directivas para determinar si el usuario y el dispositivo cumplen la directiva establecida por los administradores de seguridad.
* La capacidad de conceder o denegar el acceso a los recursos de la organización.

Con Application Proxy, Azure AD realiza el seguimiento de los usuarios que tienen que acceder a las aplicaciones web publicadas en el entorno local y en la nube. Proporciona un punto de administración central para esas aplicaciones. Aunque no es necesario, se recomienda habilitar también el acceso condicional de Azure AD. Al definir las condiciones de autenticación y acceso de los usuarios, se garantiza que las personas adecuadas tengan acceso a las aplicaciones.

**Nota:** Es importante entender que Azure AD Application Proxy está pensado como una VPN o un reemplazo del proxy inverso para usuarios móviles (o remotos) que necesitan acceso a los recursos internos. No está pensado para los usuarios internos de la red corporativa. Los usuarios internos que usan innecesariamente Application Proxy pueden presentar problemas de rendimiento inesperados y no deseados.

![Azure Active Directory y todas las aplicaciones](media/what-is-application-proxy/azure-ad-and-all-your-apps.png)

### <a name="an-overview-of-how-app-proxy-works"></a>Introducción al funcionamiento de Application Proxy

Application Proxy es un servicio de Azure AD que se configura en Azure Portal. Le permite publicar un punto de conexión de dirección URL de HTTP/HTTPS público externo en Azure Cloud, que se conecta a la dirección URL de un servidor de aplicaciones interno de la organización. Estas aplicaciones web locales pueden integrarse con Azure AD para admitir el inicio de sesión único. Después, los usuarios finales pueden tener acceso a las aplicaciones web locales del mismo modo que tienen acceso a Office 365 y otras aplicaciones SaaS.

Los componentes de esta característica incluyen el servicio Application Proxy, que se ejecuta en la nube, el conector de Application Proxy, que es un agente ligero que se ejecuta en un servidor local, y Azure AD, que es el proveedor de identidades. Los tres componentes trabajan juntos para proporcionar al usuario una experiencia de inicio de sesión única para acceder a las aplicaciones web locales.

Después de iniciar sesión, los usuarios externos pueden acceder a las aplicaciones web locales mediante una dirección URL conocida o el [panel de acceso de MyApps](https://docs.microsoft.com/azure/active-directory/user-help/my-apps-portal-end-user-access) desde su escritorio o dispositivos iOS o MAC. Por ejemplo, Application Proxy puede proporcionar acceso remoto e inicio de sesión único para Escritorio remoto, sitios de SharePoint, Tableau, Qlik y Outlook en la web, así como aplicaciones de línea de negocio (LOB).

![Arquitectura de Azure AD Application Proxy](media/what-is-application-proxy/azure-ad-application-proxy-architecture.png)

### <a name="authentication"></a>Authentication

Existen varias formas de configurar una aplicación para el inicio de sesión único y el método que seleccione depende de la autenticación que utilice la aplicación. Application Proxy admite los siguientes tipos de aplicaciones:

* Aplicaciones web
* API web que desea exponer a aplicaciones sofisticadas de diferentes dispositivos
* Aplicaciones que se encuentran detrás de una puerta de enlace de escritorio remoto
* Aplicaciones cliente enriquecidas que se integran con la [biblioteca de autenticación de Microsoft (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/v2-overview).

Application Proxy funciona con aplicaciones que utilizan el siguiente protocolo de autenticación nativo:

* **[Autenticación integrada de Windows (IWA)](application-proxy-configure-single-sign-on-with-kcd.md).** Para la autenticación integrada de Windows, los conectores de Application Proxy utilizan la delegación restringida de Kerberos (KCD) para autenticar a los usuarios en la aplicación Kerberos.

Application Proxy también admite los siguientes protocolos de autenticación con integración de terceros o en escenarios de configuración específicos:

* [**Autenticación basada en encabezados**](application-proxy-configure-single-sign-on-with-ping-access.md). Este método de inicio de sesión utiliza un servicio de autenticación de terceros llamado PingAccess y se utiliza cuando la aplicación utiliza encabezados para la autenticación. En este escenario, la autenticación se controla mediante PingAccess.
* [**Autenticación basada en contraseña o en formularios**](application-proxy-configure-single-sign-on-password-vaulting.md). Con este método de autenticación, los usuarios inician sesión en la aplicación con un nombre de usuario y una contraseña la primera vez que acceden a ella. Después del primer inicio de sesión, Azure AD proporciona el nombre de usuario y la contraseña a la aplicación. En este escenario, la autenticación se controla mediante Azure AD.
* [**Autenticación SAML**](application-proxy-configure-single-sign-on-on-premises-apps.md). El inicio de sesión único basado en SAML es compatible con aplicaciones que usan los protocolos SAML 2.0 o WS-Federation. Con el inicio de sesión único de SAML, Azure AD se autentica en la aplicación mediante el uso de la cuenta de Azure AD del usuario.

Para más información sobre los métodos admitidos, consulte [Elección de un método de inicio de sesión único](sso-options.md#choosing-a-single-sign-on-method).

### <a name="security-benefits"></a>Ventajas de seguridad

La solución de acceso remoto ofrecida por Application Proxy y Azure AD admite varias ventajas de seguridad que los clientes pueden aprovechar, entre ellas:

* **Acceso autenticado**. Application Proxy es el más adecuado para publicar aplicaciones con [autenticación previa](application-proxy-security.md#authenticated-access) para garantizar que solo las conexiones autenticadas lleguen a la red. En el caso de las aplicaciones publicadas con autenticación previa, no se permite el paso de ningún tráfico a través del servicio Application Proxy a su entorno local sin un token válido. La autenticación previa, por su propia naturaleza, bloquea un número significativo de ataques dirigidos, ya que las identidades autenticadas son las únicas que pueden acceder a la aplicación de back-end.
* **Acceso condicional**. Se pueden aplicar controles de directiva más completos antes de que se establezcan las conexiones a la red. Con el acceso condicional, puede definir las restricciones sobre el tráfico que se permite que tenga acceso a las aplicaciones de back-end. Cree directivas que restrinjan los inicios de sesión en función de la ubicación, el nivel de autenticación y el perfil de riesgo del usuario. A medida que evoluciona el acceso condicional, se agregan más controles para proporcionar seguridad adicional, como la integración con Microsoft Cloud App Security (MCAS). La integración con MCAS le permite configurar una aplicación local para la [supervisión en tiempo real](application-proxy-integrate-with-microsoft-cloud-application-security.md) al aprovechar el acceso condicional para supervisar y controlar sesiones en tiempo real basadas en directivas de acceso condicional.
* **Terminación del tráfico**. Todo el tráfico a la aplicación de back-end se termina en el servicio Application Proxy en la nube mientras que la sesión se restablece con el servidor back-end. Esta estrategia de conexión significa que los servidores de back-end no se exponen al tráfico directo de HTTP. Están mejor protegidos contra ataques por denegación de servicio (DoS) dirigidos porque el firewall no se está atacando.
* **Todo el acceso es de salida**. Los conectores de Application Proxy solo utilizan conexiones de salida al servicio Application Proxy en la nube a través de los puertos 80 y 443. Sin conexiones de entrada, no hay necesidad de abrir puertos de firewall para conexiones o componentes de entrada en la red perimetral. Todas las conexiones son de salida y a través de un canal seguro.
* **Inteligencia basada en análisis de seguridad y Machine Learning (ML)** . Dado que es parte de Azure Active Directory, Application Proxy puede aprovecharse de [Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/overview) (requiere [licencias Premium P2](https://azure.microsoft.com/pricing/details/active-directory/)). Azure AD Identity Protection combina la inteligencia de seguridad de aprendizaje automático con las fuentes de distribución de datos procedentes de [Digital Crimes Unit](https://news.microsoft.com/stories/cybercrime/index.html) y [Microsoft Security Response Center](https://www.microsoft.com/msrc) de Microsoft para identificar de forma proactiva las cuentas comprometidas. Identity Protection ofrece protección en tiempo real de inicios de sesión de alto riesgo. Tiene en cuenta factores como los accesos desde dispositivos infectados, a través de redes anónimas o desde ubicaciones atípicas e improbables para aumentar el perfil de riesgo de una sesión. Este perfil de riesgo se usa para la protección en tiempo real. Muchos de estos informes y eventos ya están disponibles a través de una API para la integración con los sistemas SIEM.

* **Acceso remoto en forma de servicio**. No tiene que preocuparse sobre cómo mantener y aplicar revisiones a servidores locales para habilitar el acceso remoto. Application Proxy es un servicio de escala de Internet que es propiedad de Microsoft, por lo que siempre obtendrá las actualizaciones y revisiones de seguridad más recientes. El software al que no se aplican revisiones sigue siendo objeto de un gran número de ataques. Según el Departamento de Seguridad Nacional, hasta el [85 por ciento de los ataques selectivos son evitables](https://www.us-cert.gov/ncas/alerts/TA15-119A). Con este modelo de servicio, ya no tiene que llevar la pesada carga de administrar los servidores periféricos ni tener que luchar para aplicarles revisiones cuando sea necesario.

* **Integración con Intune**. Con Intune, el tráfico corporativo se enruta por separado del tráfico de personal. Application Proxy garantiza que se autentica el tráfico corporativo. [Application Proxy y la funcionalidad Intune Managed Browser](https://docs.microsoft.com/intune/app-configuration-managed-browser#how-to-configure-application-proxy-settings-for-protected-browsers) también se pueden usar juntos para permitir a los usuarios remotos acceder de forma segura a sitios web internos desde dispositivos iOS y Android.

### <a name="roadmap-to-the-cloud"></a>Hoja de ruta de la nube

Otra ventaja importante de la implementación de Application Proxy es la extensión de Azure AD al entorno local. De hecho, la implementación de Application Proxy es un paso clave para trasladar la organización y las aplicaciones a la nube. Al trasladarse a la nube y alejarse de la autenticación en el entorno local, se reduce la huella local y se utilizan las funcionalidades de administración de identidades de Azure AD como el plano de control. Con actualizaciones mínimas o nulas de las aplicaciones existentes, tiene acceso a las funcionalidades en la nube, como el inicio de sesión único, la autenticación multifactor y la administración centralizada. La instalación de los componentes necesarios en Application Proxy es un proceso sencillo para establecer un marco de acceso remoto. Además, al pasar a la nube, tendrá acceso a las últimas características, actualizaciones y funcionalidades de Azure AD, como la alta disponibilidad y la recuperación ante desastres.

Para más información sobre la migración de las aplicaciones a Azure AD, consulte las notas del producto [Migrating Your Applications to Azure Active Directory](https://aka.ms/migrateapps/whitepaper) (Migración de las aplicaciones a Azure Active Directory).

## <a name="architecture"></a>Architecture

En el siguiente diagrama se ilustra de manera general cómo los servicios de autenticación de Azure AD y Application Proxy trabajan juntos para proporcionar un inicio de sesión único a las aplicaciones locales para los usuarios finales.

![Flujo de autenticación de Azure AD Application Proxy](media/what-is-application-proxy/azure-ad-application-proxy-authentication-flow.png)

1. Una vez que el usuario acceda a la aplicación a través de un punto de conexión, se le redirigirá a la página de inicio de sesión de Azure AD. Si ha configurado directivas de acceso condicional, en este momento se comprueban las condiciones específicas para garantizar que cumple con los requisitos de seguridad de la organización.
2. Después de un inicio de sesión correcto, Azure AD envía un token al dispositivo cliente del usuario.
3. El cliente envía el token al servicio Application Proxy, que recupera el nombre principal de usuario (UPN) y el nombre de entidad de seguridad (SPN) del token.
4. Application Proxy reenvía la solicitud, que recoge el [conector](application-proxy-connectors.md) Application Proxy.
5. El conector realiza cualquier autenticación adicional necesaria en nombre del usuario (*opcional según el método de autenticación*), solicita el punto de conexión interno del servidor de aplicaciones y envía la solicitud a la aplicación local.
6. La respuesta se envía del servidor de aplicaciones a través del conector al servicio Application Proxy.
7. La respuesta se envía del servicio Application Proxy al usuario.

|**Componente**|**Descripción**|
|:-|:-|
|Punto de conexión|El punto de conexión es una dirección URL o un  [portal de usuario final](end-user-experiences.md). Los usuarios pueden acceder a aplicaciones fuera de la red mediante el acceso a una dirección URL externa. Los usuarios dentro de la red pueden acceder a la aplicación a través de una dirección URL o un portal del usuario final. Cuando los usuarios usan uno de estos puntos de conexión, se autentican en Azure AD y, luego, se enrutan a través del conector a la aplicación local.|
|Azure AD|Azure AD realiza la autenticación mediante el directorio de inquilino almacenado en la nube.|
|Servicio Application Proxy|Este servicio se ejecuta en la nube como parte de Azure AD. Lo que hace es pasar el token de inicio de sesión del usuario al conector Application Proxy. Application Proxy reenvía los encabezados accesibles en la solicitud a la dirección IP del cliente y los establece según su protocolo en esta dirección. Si la solicitud entrante al proxy ya tiene ese encabezado, la dirección IP del cliente se agrega al final de la lista separada por comas que es el valor del encabezado.|
|Conector de Application Proxy|El conector es un agente ligero que se ejecuta en un servidor de Windows dentro de la red. El conector administra la comunicación entre el servicio Application Proxy en la nube y la aplicación local. El conector solo usa conexiones salientes, por lo que no es necesario abrir ningún puerto de entrada ni colocar nada en la red perimetral. Los conectores no tienen estado y extraen la información de la nube según sea necesario. Para más información sobre los conectores, por ejemplo cómo equilibran la carga y cómo la autentican, consulte  [Descripción de los conectores de Azure AD Application Proxy](application-proxy-connectors.md).|
|Active Directory (AD)|Active Directory se ejecuta en el entorno local para realizar la autenticación de las cuentas de dominio. Cuando se configura el inicio de sesión único, el conector se comunica con AD para realizar cualquier autenticación adicional necesaria.|
|Aplicación local|Por último, el usuario puede acceder a una aplicación local.|

Azure AD Application Proxy consiste en el servicio Application Proxy basado en la nube y un conector local. El conector atiende las solicitudes del servicio Application Proxy y controla las conexiones con las aplicaciones internas. Es importante tener en cuenta que todas las comunicaciones se producen a través de TLS y siempre se originan en el conector del servicio Application Proxy. Es decir, las comunicaciones son exclusivamente de salida. El conector usa un certificado de cliente para autenticarse en el servicio Application Proxy para todas las llamadas. La única excepción de la seguridad de la conexión es el momento de la instalación inicial, donde se establece el certificado de cliente. Consulte la sección [En segundo plano](application-proxy-security.md#under-the-hood) de Application Proxy para más información.

### <a name="application-proxy-connectors"></a>Conectores de Application Proxy

Los [conectores de Application Proxy](application-proxy-connectors.md) son agentes ligeros implementados en el entorno local que facilitan la conexión saliente del servicio Application Proxy en la nube. Los conectores deben instalarse en un servidor de Windows Server con acceso a la aplicación de back-end. Los usuarios se conectan al servicio en la nube de Application Proxy que enruta el tráfico a las aplicaciones a través de los conectores, como se muestra a continuación.

![Conexiones de red de Azure AD Application Proxy](media/what-is-application-proxy/azure-ad-application-proxy-network-connections.png)

La configuración y el registro entre un conector y el servicio Application Proxy se realiza de la siguiente manera:

1. El administrador de TI abre los puertos 80 y 443 al tráfico de salida y permite el acceso a varias direcciones URL que son necesarias para el conector, el servicio Application Proxy y Azure AD.
2. El administrador inicia sesión en Azure Portal y ejecuta un archivo ejecutable para instalar el conector en un servidor Windows local.
3. El conector comienza a "escuchar" el servicio Application Proxy.
4. El administrador agrega la aplicación local a Azure AD y configura parámetros como las direcciones URL que los usuarios necesitan para conectarse a las aplicaciones.

Para más información, consulte [Planear una implementación de Azure AD Application Proxy](application-proxy-deployment-plan.md).

Se recomienda implementar siempre varios conectores para obtener redundancia y escalabilidad. Los conectores, junto con el servicio, se encargan de todas las tareas de alta disponibilidad y pueden agregarse o eliminarse dinámicamente. Cada vez que llega una solicitud nueva, esta se enruta a uno de los conectores que esté disponible. Cuando hay un conector en ejecución, este permanece activo tal y como se conecta al servicio. Si un conector no está disponible temporalmente, no responde a este tráfico. Los conectores que no se utilizan se etiquetan como inactivos y se quitan tras 10 días de inactividad.

Los conectores también sondean al servidor para averiguar si hay una versión más reciente del conector. Aunque puede realizar una actualización manual, los conectores se actualizarán automáticamente mientras se esté ejecutando el servicio Application Proxy Connector Updater. Para los inquilinos con varios conectores, las actualizaciones automáticas se dirigen a un conector cada vez en cada grupo para evitar tiempos de inactividad en su entorno.

**Nota**: Para supervisar la [página del historial de versiones](application-proxy-release-version-history.md) de Application Proxy para que se le notifique cuando se hayan publicado actualizaciones, suscríbase a la fuente RSS.

Cada conector Application Proxy se asigna a un [grupo de conectores](application-proxy-connector-groups.md). Los conectores del mismo grupo funcionan como una única unidad para una alta disponibilidad y equilibrio de carga. Puede crear nuevos grupos, asignarles conectores en Azure Portal y, a continuación, asignarles conectores específicos para que presten servicio a aplicaciones específicas. Se recomienda tener al menos dos conectores en cada grupo de conectores para lograr una alta disponibilidad.

Los grupos de conectores son útiles cuando se necesita admitir los escenarios siguientes:

* Publicación de aplicaciones geográficas
* Aislamiento o segmentación de aplicaciones
* Publicación de aplicaciones web que se ejecutan en la nube o en el entorno local

Para más información sobre cómo elegir dónde instalar los conectores, consulte [Consideraciones sobre la topología de red al utilizar Azure Active Directory Application Proxy](application-proxy-network-topology.md).

## <a name="other-use-cases"></a>Otros casos de uso

Hasta ahora, nos hemos centrado en el uso de Application Proxy para publicar aplicaciones locales de forma externa, a la vez que permitimos el inicio de sesión único en todas las aplicaciones en la nube y locales. Sin embargo, hay otros casos de uso para Application Proxy que merece la pena mencionar. Incluyen:

* **Publicación de forma segura de API REST**. Si tiene lógica de negocio o API que se ejecutan localmente o se hospedan en máquinas virtuales en la nube, Application Proxy proporciona un punto de conexión público para el acceso a la API. El acceso al punto de conexión de la API le permite controlar la autenticación y la autorización sin necesidad de puertos de entrada. Proporciona seguridad adicional mediante las características de Azure AD Premium, como la autenticación multifactor y el acceso condicional basado en dispositivos para equipos de sobremesa, iOS, MAC y dispositivos Android que utilizan Intune. Para más información, consulte [How to enable native client applications to interact with proxy applications](application-proxy-configure-native-client-application.md) (Cómo habilitar las aplicaciones cliente nativas para que interactúen con las aplicaciones proxy) y [Protección de una API mediante OAuth 2.0 con Azure Active Directory API Management](https://docs.microsoft.com/azure/api-management/api-management-howto-protect-backend-with-aad).
* **Servicios de Escritorio remoto** **(RDS)** . Las implementaciones de RDS estándar requieren conexiones de entrada abiertas. Sin embargo, la implementación de [RDS con Application Proxy](application-proxy-integrate-with-remote-desktop-services.md) tiene una conexión de salida permanente desde el servidor que ejecuta el servicio de conexión. De esta forma, puede ofrecer más aplicaciones a los usuarios finales mediante la publicación de aplicaciones locales mediante Servicios de Escritorio remoto. También puede reducir la superficie de ataque de la implementación con un conjunto limitado de controles de verificación en dos pasos y de acceso condicional a RDS.
* **Publicación de aplicaciones que se conectan mediante WebSockets**. La compatibilidad con [Qlik Sense](application-proxy-qlik.md) se encuentra en versión preliminar pública y se expandirá a otras aplicaciones en el futuro.
* **Habilitación de las aplicaciones de cliente nativas para interactuar con el proxy de aplicaciones**. Puede usar Azure AD Application Proxy para publicar aplicaciones web, pero también se puede utilizarse para publicar las [aplicaciones cliente nativas](application-proxy-configure-native-client-application.md) que se configuran con la Biblioteca de autenticación de Azure AD (ADAL). Las aplicaciones cliente nativas son distintas de las aplicaciones web porque se instalan en un dispositivo, mientras a las aplicaciones web se accede mediante un explorador.

## <a name="conclusion"></a>Conclusión

La forma en que trabajamos y las herramientas que utilizamos están cambiando rápidamente. Con más empleados que traen sus propios dispositivos al trabajo y el uso generalizado de aplicaciones de software como servicio (SaaS), la forma en que las organizaciones administran y protegen los datos también debe evolucionar. Las empresas ya no operan únicamente dentro de sus propios muros, protegidas por un foso que rodea su frontera. Los datos viajan a más lugares que nunca, tanto en entornos locales como en entornos en la nube. Esta evolución ha ayudado a aumentar la productividad y la capacidad de colaboración de los usuarios, pero también hace que la protección de datos confidenciales sea más difícil.

Ya sea que actualmente esté usando Azure AD para administrar usuarios en un escenario de coexistencia híbrida o esté interesado en comenzar su viaje a la nube, la implementación de Azure AD Application Proxy puede ayudar a reducir el tamaño de su huella local al proporcionar acceso remoto como un servicio.

Las organizaciones deben comenzar a sacar partido a Application Proxy hoy mismo para aprovecharse de las ventajas siguientes:

* Publicación de las aplicaciones locales externamente sin la sobrecarga asociada con el mantenimiento de la VPN tradicional u otras soluciones de publicación web locales y el enfoque de red perimetral
* Inicio de sesión único para todas las aplicaciones, ya sean Office 365 u otras aplicaciones SaaS, incluidas las aplicaciones locales
* Seguridad a escala de nube donde Azure AD aprovecha la telemetría de Office 365 para evitar el acceso no autorizado
* Integración con Intune para garantizar que el tráfico corporativo se ha autenticado
* Centralización de la administración de las cuentas de usuario
* Actualizaciones automáticas para asegurarse de que dispone de las últimas revisiones de seguridad
* Nuevas características a medida que se publican; la más reciente es la compatibilidad con el inicio de sesión único de SAML y una administración más pormenorizada de las cookies de aplicación

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre el planeamiento, el funcionamiento y la administración de Azure AD Application Proxy, consulte [Planear una implementación de Azure AD Application Proxy](application-proxy-deployment-plan.md).
* Para programar una demostración en vivo u obtener una evaluación gratuita de 90 días para su evaluación, consulte [Introducción a Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-trial).
