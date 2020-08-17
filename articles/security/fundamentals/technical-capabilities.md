---
title: Capacidades técnicas de seguridad en Azure - Microsoft Azure
description: Introducción a los servicios de seguridad de Azure que ayudan a proteger los datos, recursos y aplicaciones en la nube.
services: security
author: terrylanfear
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.date: 07/13/2020
ms.author: terrylan
ms.openlocfilehash: d861388c8c7a5ff64a17607736d4c8a292343dec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071589"
---
# <a name="azure-security-technical-capabilities"></a>Funcionalidades técnicas de seguridad de Azure
Este artículo sirve de introducción a los servicios de seguridad de Azure que ayudan a proteger los datos, recursos y aplicaciones en la nube, así como a satisfacer las necesidades de seguridad de su negocio.

## <a name="azure-platform"></a>Plataforma Azure

[Microsoft Azure](https://azure.microsoft.com/overview/what-is-azure/) es una plataforma en la nube compuesta de servicios de infraestructura y aplicaciones, con servicios de datos y análisis avanzado integrados, y herramientas y servicios para desarrolladores, que se hospeda dentro de los centros de datos de la nube pública de Microsoft. Los clientes pueden usar Azure para muchos tipos de capacidades y escenarios, que van desde procesos, redes y almacenamiento básicos a servicios móviles y de aplicaciones web, o a escenarios completamente en la nube como Internet de las cosas, que se pueden usar con tecnologías de código abierto e implementarse en nubes híbridas u hospedarse en el centro de datos de un cliente. Azure proporciona tecnología de nube como bloques de creación para ayudar a las empresas a ahorrar costos, innovar con rapidez y a administrar los sistemas de forma proactiva. Al crear o migrar recursos de TI a un proveedor de nube, confía en las capacidades de la organización para proteger las aplicaciones y los datos con los servicios y los controles que facilitan para administrar la seguridad de sus recursos en la nube.

Microsoft Azure es el único proveedor de informática en la nube que ofrece una plataforma de aplicaciones seguras y coherente y una infraestructura como servicio para que los equipos trabajen según sus diferentes habilidades en la nube y niveles de complejidad del proyecto, con servicios de datos y análisis integrados que descubran inteligencia de datos en cualquier parte, a través de plataformas de Microsoft o de terceros, abra marcos y herramientas, proporcionando alternativas para la integración en la nube con aplicaciones locales así como la implementación de servicios en la nube de Azure en centros de datos locales. Como parte de Microsoft Trusted Cloud, los clientes confían en Azure a la hora de conseguir una seguridad, confiabilidad, cumplimiento y privacidad líderes en el mercado, así como en la enorme red de usuarios, asociados y procesos existentes para ayudar a las organizaciones en la nube.

Con Microsoft Azure, puede:

- Acelerar la innovación con la nube.

- Impulsar aplicaciones y decisiones empresariales con información.

- Compilar con libertad e implementar en cualquier parte.

- Proteger su negocio.

## <a name="security-technical-capabilities-to-fulfil-your-responsibility"></a>Funcionalidades técnicas de seguridad para cumplir con su responsabilidad

Microsoft Azure proporciona servicios que pueden ayudarle a satisfacer las necesidades de seguridad, privacidad y cumplimiento. La imagen siguiente nos ayuda a explicarle los diversos servicios de Azure disponibles para que pueda crear una infraestructura de aplicaciones segura y compatible con los estándares del sector.

![Funcionalidades técnicas de seguridad disponibles. Panorama general](./media/technical-capabilities/azure-security-technical-capabilities-fig1.png)

## <a name="manage-and-control-identity-and-user-access"></a>Administración y control de identidades y del acceso de usuarios

Azure le ayuda a proteger información empresarial y personal permitiéndole administrar las identidades y credenciales del usuario y controlar los accesos.

### <a name="azure-active-directory"></a>Azure Active Directory

Las soluciones de administración de identidades y acceso de Microsoft ayudan al departamento de TI a proteger el acceso a las aplicaciones y los recursos en el centro de datos corporativo y en la nube, para lo que se habilitan más niveles de validación, tales como las directivas de autenticación multifactor y de acceso condicional. La supervisión de actividades sospechosas mediante auditorías, alertas e informes de seguridad avanzados contribuye a minimizar los posibles problemas de seguridad. [Azure Active Directory Premium](../../active-directory/active-directory-whatis.md) proporciona un inicio de sesión único para miles de aplicaciones en la nube y acceso a aplicaciones web que se ejecutan de forma local.

Entre las ventajas en materia de seguridad de Azure Active Directory (Azure AD) se incluye la capacidad de:

- Crear y administrar una identidad única para cada usuario en toda la empresa híbrida, lo que mantiene los usuarios, grupos y dispositivos sincronizados.

- Proporcionar un acceso de inicio de sesión único a las aplicaciones, incluidas miles de aplicaciones SaaS preintegradas.

- Habilitar la seguridad del acceso de las aplicaciones mediante la aplicación de Multi-Factor Authentication basado en reglas para las aplicaciones locales y en la nube.

- Proporcionar un acceso remoto seguro a las aplicaciones web locales a través del proxy de la aplicación de Azure AD.

El [portal de Azure Active Directory](https://aad.portal.azure.com/) está disponible como parte de Azure Portal. En este panel, puede obtener una visión general del estado de su organización y administrar fácilmente el directorio, los usuarios o el acceso a las aplicaciones.

![Azure Active Directory](./media/technical-capabilities/azure-security-technical-capabilities-fig2.png)

Las siguientes son las principales funcionalidades de administración de identidades de Azure:

- Inicio de sesión único

- Multi-Factor Authentication

- Supervisión de seguridad, alertas e informes basados en aprendizaje automático

- Administración de identidades y acceso de consumidores

- Registro de dispositivos

- Privileged Identity Management

- Protección de identidad

#### <a name="single-sign-on"></a>Inicio de sesión único

[Inicio de sesión único (SSO)](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/) significa poder tener acceso a todas las aplicaciones y los recursos que necesita para hacer negocios, iniciando sesión una sola vez con una única cuenta de usuario. Una vez que ha iniciado sesión, puede tener acceso a todas las aplicaciones que necesite sin tener que autenticarse (por ejemplo, escribiendo una contraseña) una segunda vez.

Muchas organizaciones confían en el modelo de software como servicio (SaaS), como Office 365, Box y Salesforce para la productividad del usuario final. Tradicionalmente, el personal de TI tenía que crear y actualizar individualmente cuentas de usuario en cada aplicación SaaS y los usuarios tenían que recordar una contraseña para cada aplicación SaaS.

[Azure AD extiende Active Directory local a la nube](../../active-directory/manage-apps/what-is-single-sign-on.md), permitiendo a los usuarios usar su cuenta profesional principal no solo para iniciar sesión en sus dispositivos unidos a dominios y recursos de la empresa, sino también en todas las aplicaciones web y SaaS necesarias para su trabajo.

Los usuarios no solo no tienen que administrar varios conjuntos de nombres de usuario y contraseñas, sino que es posible aprovisionar o cancelar el aprovisionamiento del acceso a las aplicaciones automáticamente en función de los grupos de la organización y de su estado de empleado. [Azure AD introduce controles de seguridad y de gobernanza del acceso](../../active-directory/active-directory-enterprise-apps-manage-sso.md) que permiten administrar de forma centralizada el acceso de los usuarios a través de aplicaciones SaaS.

#### <a name="multi-factor-authentication"></a>Multi-Factor Authentication

[Azure Multi-factor Authentication (MFA)](../../active-directory/authentication/multi-factor-authentication.md) es un método de autenticación que requiere el uso de más de un método de verificación, y agrega un segundo nivel de seguridad crítico a las transacciones e inicios de sesión del usuario. [MFA ayuda a proteger](../../active-directory/authentication/concept-mfa-howitworks.md) el acceso a los datos y las aplicaciones, al tiempo que satisface la demanda de los usuarios de un proceso de inicio de sesión simple. Proporciona autenticación sólida mediante una gran variedad de opciones de verificación, como llamadas telefónicas, mensajes de texto, notificaciones de aplicaciones móviles, códigos de verificación y tokens OAuth de terceros.

#### <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>Supervisión de seguridad, alertas e informes basados en aprendizaje automático

La supervisión y las alertas de seguridad, así como los informes basados en el aprendizaje automático que identifican patrones de acceso incoherentes, puede ayudarle a proteger su negocio. Puede usar los informes de acceso y uso de Active Directory de Azure para proporcionar visibilidad sobre la integridad y la seguridad del directorio de su organización. Con esta información, un administrador de directorios puede determinar mejor dónde puede haber posibles riesgos de seguridad de modo que pueda planear adecuadamente la mitigación de estos riesgos.

En Azure Portal o a través del [portal de Azure Active Directory](https://aad.portal.azure.com/), los [informes](../../active-directory/active-directory-reporting-azure-portal.md) se clasifican de la manera siguiente:

- Informes de anomalías: contienen eventos de inicio de sesión que se consideran anómalos. Nuestro objetivo es que sea consciente de dicha actividad y que pueda tomar una decisión sobre si un evento es sospechoso.

- Informes de aplicaciones integradas: ofrecen información sobre cómo se usan las aplicaciones en la nube en la organización. Azure Active Directory ofrece integración con miles de aplicaciones en la nube.

- Informes de errores: indican errores que se pueden producir al aprovisionar cuentas a aplicaciones externas.

- Informes específicos del usuario: muestran los datos de actividad de dispositivo o de inicio de sesión de un usuario concreto.

- Registros de actividad: contienen un registro de todos los eventos auditados en las últimas 24 horas, los últimos 7 días o los últimos 30 días, así como los cambios en la actividad del grupo y la actividad de registro y de restablecimiento de contraseña.

#### <a name="consumer-identity-and-access-management"></a>Administración de identidades y acceso de consumidores

[Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) es un servicio de administración de identidades global y de alta disponibilidad para aplicaciones orientadas al consumidor que se puede utilizar con cientos de millones de identidades. Se puede integrar en plataformas móviles y web. Los consumidores pueden iniciar sesión en todas sus aplicaciones con una experiencia totalmente personalizable, usando sus cuentas de las redes sociales o mediante credenciales nuevas.

En el pasado, los desarrolladores de aplicaciones que deseaban [registrar e iniciar la sesión de los consumidores](../../active-directory-b2c/overview.md) en sus aplicaciones tenían que escribir su propio código. Y usaban bases de datos o sistemas locales para almacenar nombres de usuario y contraseñas. Azure Active Directory B2C ofrece a su organización una manera mejor de integrar la administración de identidades de consumidor en las aplicaciones gracias a la ayuda de una plataforma segura basada en estándares y un amplio conjunto de directivas extensibles.

El uso de Azure Active Directory B2C permite a los consumidores registrarse en las aplicaciones con sus cuentas de redes sociales existentes (Facebook, Google, Amazon, LinkedIn) o creando nuevas credenciales (dirección de correo electrónico y contraseña o nombre de usuario y contraseña).

#### <a name="device-registration"></a>Registro de dispositivos

El [Registro de dispositivos de Azure AD](../../active-directory/devices/overview.md) es la base de los escenarios de [acceso condicional](../../active-directory/devices/overview.md) basado en dispositivos. Cuando se registra un dispositivo, el Registro de dispositivos de Azure AD le proporciona una identidad que se utiliza para autenticar el dispositivo cuando el usuario inicia sesión. El dispositivo autenticado y los atributos del dispositivo pueden utilizarse para aplicar directivas de acceso condicional a las aplicaciones que se hospedan en la nube y en el entorno local.

Cuando se combina con una solución de [administración de dispositivos móviles (MDM)](https://www.microsoft.com/itshowcase/Article/Content/588/Mobile-device-management-at-Microsoft) como Intune, los atributos del dispositivo en Azure Active Directory se actualizan con información adicional sobre este. Esto le permite crear reglas de acceso condicional que obligan a que el acceso desde dispositivos cumpla con las normas de seguridad y cumplimiento.

#### <a name="privileged-identity-management"></a>Privileged Identity Management

[Privileged Identity Management de Azure Active Directory (AD)](../../active-directory/privileged-identity-management/pim-configure.md) le permite administrar, controlar y supervisar las identidades con privilegios y el acceso a los recursos en Azure AD y en otros servicios en línea de Microsoft, como Office 365 o Microsoft Intune.

A veces, los usuarios necesitan llevar a cabo operaciones con privilegios en recursos de Azure u Office 365 o en otras aplicaciones SaaS. Esto significa a menudo que las organizaciones tienen que concederles acceso con privilegios permanentes en Azure AD. Esto representa un riesgo de seguridad creciente para los recursos hospedados en la nube porque las organizaciones no pueden supervisar suficientemente qué hacen esos usuarios con sus privilegios administrativos. Además, si se pone en peligro una cuenta de usuario que tiene acceso con privilegios, esta situación podría afectar a su seguridad en la nube global. Administración de identidades con privilegios de Azure AD le ayuda a resolver este riesgo.

Azure AD Privileged Identity Management le permite:

- Ver qué usuarios son administradores de Azure AD

- Habilitar el acceso administrativo a petición y "justo a tiempo" en servicios de Microsoft Online Services, como Office 365 e Intune

- Obtener informes sobre el historial de acceso de administrador y sobre los cambios en las asignaciones de administrador

- Obtener alertas sobre el acceso a un rol con privilegios

#### <a name="identity-protection"></a>Protección de identidad

[Azure AD Identity Protection](../../active-directory/identity-protection/overview.md) es un servicio de seguridad que proporciona una vista consolidada de detecciones de riesgo y posibles vulnerabilidades que afectan a las identidades de su organización. Identity Protection usa las funcionalidades de detección de anomalías existentes de Azure Active Directory (disponibles mediante informes de actividad anómala de Azure AD) e introduce nuevos tipos de detección de riesgo que pueden detectar anomalías en tiempo real.

## <a name="secure-resource-access"></a>Protección del acceso a los recursos

El control de acceso de Azure se inicia desde una perspectiva de facturación. El propietario de una cuenta de Azure, a la que se accede desde el [Centro de cuentas de Azure](https://account.windowsazure.com/subscriptions), es el administrador de cuenta (AA). Las suscripciones son un contenedor para la facturación, pero también sirven de límite de seguridad: cada suscripción tiene un administrador de servicios (SA) que puede agregar, quitar y modificar recursos de Azure en esa suscripción mediante Azure Portal. El administrador de servicios predeterminado de una suscripción nueva es el administrador de cuenta, pero este puede cambiar el administrador de servicios en el Centro de cuentas de Azure.

![Acceso protegido a los recursos de Azure](./media/technical-capabilities/azure-security-technical-capabilities-fig3.png)

Las suscripciones también tienen una asociación con un directorio. El directorio define un conjunto de usuarios. Pueden ser usuarios de una cuenta profesional o educativa que crearon el directorio, o bien pueden ser usuarios externos (es decir, Cuentas Microsoft). Las suscripciones son accesibles por un subconjunto de esos usuarios del directorio que se han asignado como administrador de servicios (SA) o coadministrador (CA); la única excepción es que, por razones heredadas, las Cuentas de Microsoft (antes Windows Live ID) pueden asignarse como SA o CA sin estar presentes en el directorio.

Las empresas de seguridad deben centrarse en conceder a los empleados los permisos exactos que necesiten. Un número elevado de permisos puede provocar que la cuenta esté expuesta a los atacantes. Si se conceden muy pocos, los empleados no podrán realizar su trabajo de manera eficaz. El [control de acceso basado en rol (RBAC) de Azure](../../role-based-access-control/overview.md) ayuda a abordar este problema al ofrecer administración avanzada del acceso para Azure.

![Acceso protegido a los recursos](./media/technical-capabilities/azure-security-technical-capabilities-fig4.png)

También podrá repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. En lugar de proporcionar a todos los empleados permisos no restringidos en los recursos o la suscripción de Azure, puede permitir solo determinadas acciones. Por ejemplo, utilice RBAC para dejar que un empleado administre máquinas virtuales en una suscripción, y que otro pueda administrar bases de datos SQL en la misma suscripción.

![Acceso protegido a los recursos de Azure (RBAC)](./media/technical-capabilities/azure-security-technical-capabilities-fig5.png)

## <a name="data-security-and-encryption"></a>Seguridad y cifrado de datos

Uno de los elementos clave para la protección de datos en la nube consiste en tener en cuenta los posibles estados en que se pueden producir datos y qué controles hay disponibles para ese estado. Como parte de los procedimientos recomendados de cifrado y seguridad de datos en Azure, se ofrecen recomendaciones relacionadas con los estados de datos siguientes.

- En reposo: Esto incluye información sobre todos los objetos de almacenamiento, los contenedores y los tipos que existen de forma estática en medios físicos, ya sean discos magnéticos u ópticos.
- En tránsito: Se considera que los datos están en movimiento cuando se transfieren entre componentes, ubicaciones o programas. Por ejemplo, a través de la red o un bus de servicio (desde una ubicación local hacia la nube, y viceversa, incluidas las conexiones híbridas como ExpressRoute) o durante el proceso de entrada y salida.

### <a name="encryption-at-rest"></a>Cifrado en reposo

El cifrado en reposo se describe con más detalle en [Cifrado en reposo de datos de Azure](encryption-atrest.md).

### <a name="encryption-in-transit"></a>Cifrado en tránsito

Proteger los datos en tránsito debe ser una parte esencial de su estrategia de protección de datos. Puesto que los datos se desplazan entre muchas ubicaciones, la recomendación general es utilizar siempre los protocolos SSL/TLS para intercambiar datos entre diferentes ubicaciones. En algunas circunstancias, podría aislar todo el canal de comunicación entre su infraestructura local y la nube mediante una red privada virtual (VPN).

Para los datos que se desplazan entre la infraestructura local y Azure, debe plantearse usar medidas de seguridad apropiadas, como HTTPS o VPN.

Para las organizaciones que necesitan proteger el acceso a Azure desde varias estaciones de trabajo locales, use una [VPN de sitio a sitio de Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md).

Si la organización necesita proteger el acceso a Azure desde una estación de trabajo local, use una [VPN de punto a sitio](../../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md).

Los conjuntos de datos más grandes se pueden mover por medio de un vínculo WAN dedicado de alta velocidad como [ExpressRoute](https://azure.microsoft.com/services/expressroute/). Si decide usar ExpressRoute, también puede cifrar los datos en el nivel de aplicación mediante [SSL/TLS](https://support.microsoft.com/kb/257591) u otros protocolos para una mayor protección.

Si interactúa con Azure Storage a través de Azure Portal, todas las transacciones se realizan a través de HTTPS. También se puede usar la [API REST de almacenamiento](https://msdn.microsoft.com/library/azure/dd179355.aspx) a través de HTTPS para interactuar con [Azure Storage](https://azure.microsoft.com/services/storage/) y [Azure SQL Database](https://azure.microsoft.com/services/sql-database/).

Las organizaciones que no protegen los datos en tránsito son más susceptibles a los [ataques del tipo "Man in the middle"](https://technet.microsoft.com/library/gg195821.aspx), a la [interceptación](https://technet.microsoft.com/library/gg195641.aspx) y al secuestro de sesión. Estos ataques pueden ser el primer paso para obtener acceso a datos confidenciales.

Para más información sobre la opción de VPN de Azure, lea el artículo [Planeamiento y diseño de VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="enforce-file-level-data-encryption"></a>Aplicación del cifrado de datos a nivel de archivos

[Azure RMS](https://technet.microsoft.com/library/jj585026.aspx) usa directivas de autorización, identidad y cifrado para ayudar a proteger los archivos y el correo electrónico. Azure RMS funciona en varios dispositivos (teléfonos, tabletas y PC) y los protege tanto dentro como fuera de su organización. Esta funcionalidad es posible porque Azure RMS agrega una capa de protección que acompaña a los datos, incluso cuando salen de los límites de su organización.

Cuando usa Azure RMS para proteger los archivos, usa criptografía estándar del sector que es totalmente compatible con [FIPS 140-2](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf). Cuando aprovecha Azure RMS para proteger los datos, tiene la garantía de que la protección permanece con el archivo, incluso si se copia en almacenamiento que no esté bajo el control de TI, como un servicio de almacenamiento en la nube. Lo mismo ocurre con los archivos compartidos por correo electrónico: se protege el archivo adjunto al mensaje de correo electrónico y se incluyen instrucciones para abrirlo.
Durante el planeamiento para adoptar Azure RMS, se recomienda lo siguiente:

- Instale la [aplicación RMS sharing](https://technet.microsoft.com/library/dn339006.aspx). Esta aplicación se integra con las aplicaciones de Office con la instalación de un complemento de Office con el que los usuarios pueden proteger los archivos directamente y de forma sencilla.

- Configure las aplicaciones y los servicios para que admitan Azure RMS.

- Cree [plantillas personalizadas](https://technet.microsoft.com/library/dn642472.aspx) que reflejen sus requisitos empresariales. Por ejemplo: una plantilla para los datos más delicados que se deba aplicar a todos los correos electrónicos de mayor confidencialidad.

Las organizaciones con puntos débiles en la [clasificación de datos](https://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf) y la protección de archivos pueden ser más susceptibles a la fuga de datos. Sin una protección adecuada de los archivos, las organizaciones no pueden obtener perspectivas empresariales, supervisar el abuso ni evitar el acceso malicioso a archivos.

> [!Note]
> Puede obtener más información sobre Azure RMS en el artículo [Requisitos de Azure Information Protection](https://technet.microsoft.com/library/jj585016.aspx).

## <a name="secure-your-application"></a>Protección de la aplicación
Aunque Azure se encarga de proteger la infraestructura y la plataforma en las que se ejecuta su aplicación, es responsabilidad suya proteger su propia aplicación. Es decir, debe desarrollar, implementar y administrar el código y el contenido de la aplicación de forma segura. De lo contrario, el código o el contenido de la aplicación pueden seguir siendo vulnerables frente a amenazas.

### <a name="web-application-firewall"></a>Firewall de aplicaciones web
[Firewall de aplicaciones web (WAF)](../../application-gateway/waf-overview.md) es una característica de [Application Gateway](../../application-gateway/overview.md) que proporciona a las aplicaciones una protección centralizada contra vulnerabilidades de seguridad comunes.

Firewall de aplicaciones web se basa en las reglas contenidas en [OWASP Core Rule Set](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 o 2.2.9. Las aplicaciones web son cada vez más los objetivos de ataques malintencionados que aprovechan vulnerabilidades comunes conocidas, como ataques por inyección de código SQL o ataques de scripts de sitios, por nombrar unos pocos. Impedir tales ataques en el código de aplicación puede ser un verdadero desafío y requerir tareas rigurosas de mantenimiento, aplicación de revisiones y supervisión en varias capas de la topología de aplicación. Un firewall de aplicaciones web centralizado facilita enormemente la administración de la seguridad y proporciona mayor protección a los administradores de la aplicación frente a amenazas o intrusiones. Las soluciones de WAF también pueden reaccionar más rápido ante una amenaza de la seguridad aplicando revisiones que aborden una vulnerabilidad conocida en una ubicación central en lugar de proteger cada una de las aplicaciones web por separado. Las puertas de enlace de aplicaciones existentes pueden transformarse rápidamente en puertas de enlace con un firewall de aplicaciones web habilitado.

Entre las vulnerabilidades web más habituales frente a las que protege el firewall de aplicaciones web se incluyen:

- Protección contra la inyección de código SQL

- Protección contra scripts entre sitios

- Protección contra ataques web comunes, como inyección de comandos, contrabando de solicitudes HTTP, división de respuestas HTTP y ataque remoto de inclusión de archivos

- Protección contra infracciones del protocolo HTTP

- Protección contra anomalías del protocolo HTTP, como la falta de agentes de usuario de host y encabezados de aceptación

- Prevención contra bots, rastreadores y escáneres

- Detección de errores de configuración comunes en aplicaciones (es decir, Apache, IIS, etc.)

> [!Note]
> Para una lista más detallada de las reglas y los mecanismos de protección, consulte el siguiente apartado sobre [conjuntos de reglas principales](../../application-gateway/waf-overview.md):

Azure dispone también de varias características fáciles de usar que contribuyen a proteger el tráfico entrante y saliente de la aplicación. Azure ayuda a los clientes a proteger el código de su aplicación con funcionalidad externa para detectar vulnerabilidades en la aplicación web.

- [Configurar la autenticación de Azure Active Directory en la aplicación](https://azure.microsoft.com/blog/azure-websites-authentication-authorization/)

- [Proteger el tráfico a la aplicación mediante la habilitación de Seguridad de la capa de transporte (TLS/SSL) - HTTPS](../../app-service/configure-ssl-bindings.md)

  - [Forzar todo el tráfico entrante a través de la conexión HTTPS](http://microsoftazurewebsitescheatsheet.info/)

  - [Habilitar Seguridad de transporte estricto (HSTS)](http://microsoftazurewebsitescheatsheet.info/#enable-http-strict-transport-security-hsts)

- [Restringir el acceso a la aplicación mediante la dirección IP del cliente](http://microsoftazurewebsitescheatsheet.info/#filtering-traffic-by-ip)

- [Restringir el acceso a la aplicación según el comportamiento del cliente: frecuencia de solicitud y simultaneidad](http://microsoftazurewebsitescheatsheet.info/#dynamic-ip-restrictions)

- [Examinar el código de la aplicación web en búsqueda de vulnerabilidades mediante Tinfoil Security Scanning](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/)

- [Configurar la autenticación mutua de TLS para requerir certificados de cliente para conectarse a la aplicación web](../../app-service/app-service-web-configure-tls-mutual-auth.md)

- [Configurar un certificado de cliente para usarlo desde la aplicación para conectarse de forma segura a recursos externos](https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/)

- [Quitar encabezados de servidor estándar para evitar que las herramientas creen huellas digitales en la aplicación](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/)

- [Conectar de forma segura la aplicación con los recursos de una red privada mediante VPN de punto a sitio](../../app-service/web-sites-integrate-with-vnet.md)

- [Conectar de forma segura la aplicación con los recursos de una red privada mediante conexiones híbridas](../../app-service/app-service-hybrid-connections.md)

Azure App Service usa la misma solución antimalware que usan Azure Cloud Services y Virtual Machines. Para obtener más información al respecto, consulte nuestra [documentación sobre antimalware](antimalware.md)

## <a name="secure-your-network"></a>Protección de la red
Microsoft Azure incluye una sólida infraestructura de red que respalda sus requisitos de conectividad de aplicaciones y servicios. Es posible la conectividad de red entre recursos ubicados en Azure, entre recursos locales y hospedados en Azure y entre Internet y Azure.

La [infraestructura de red de Azure](../../virtual-machines/windows/infrastructure-example.md) le permite conectar de forma segura los recursos de Azure entre sí por medio de [redes virtuales](../../virtual-network/virtual-networks-overview.md). Una red virtual es una representación de su propia red en la nube. Una red virtual es un aislamiento lógico de la red de nube de Azure dedicada a su suscripción. Puede conectar redes virtuales a las redes locales.

![Protección de la red (protección)](./media/technical-capabilities/azure-security-technical-capabilities-fig6.png)

Si necesita un control de acceso de nivel de red básico (basado en la dirección IP y los protocolos TCP o UDP), puede usar [grupos de seguridad de red](../../virtual-network/virtual-network-vnet-plan-design-arm.md). Un grupo de seguridad de red (NSG) es un firewall de filtrado de paquetes básico con estado que le permite controlar el acceso basado en una [5-tupla](https://www.techopedia.com/definition/28190/5-tuple).

Las redes de Azure ofrecen la posibilidad de personalizar el comportamiento de enrutamiento del tráfico de red en Azure Virtual Network. Para ello, puede configurar [rutas definidas por el usuario](../../virtual-network/virtual-networks-udr-overview.md) en Azure.

[tunelización forzada](https://www.petri.com/azure-forced-tunneling) es un mecanismo que puede usar para tener la seguridad de que no se permite que sus servicios inicien una conexión con dispositivos en Internet.

Azure es compatible con la conectividad de vínculo WAN dedicado a su red local y a una instancia de Azure Virtual Network con [ExpressRoute](../../expressroute/expressroute-introduction.md). El vínculo entre Azure y su sitio utiliza una conexión dedicada que no va por la red pública de Internet. Si su aplicación de Azure se va a ejecutar en varios centros de datos, puede utilizar [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) para enrutar las solicitudes de los usuarios de forma inteligente entre instancias de la aplicación. También puede enrutar tráfico a servicios que no se ejecuten en Azure si se puede obtener acceso a ellos desde Internet.

## <a name="virtual-machine-security"></a>Seguridad de máquina virtual

[Azure Virtual Machines](../../virtual-machines/index.yml) permite implementar una amplia gama de soluciones informáticas con agilidad. Gracias a la compatibilidad con Microsoft Windows, Linux, Microsoft SQL Server, Oracle, IBM, SAP y Azure BizTalk Services, puede implementar cualquier carga de trabajo y cualquier idioma en casi cualquier sistema operativo.

Con Azure, puede usar [software antimalware](antimalware.md) de proveedores de seguridad como Microsoft, Symantec, Trend Micro y Kaspersky, para proteger las máquinas virtuales de archivos malintencionados, adware y otras amenazas.

Microsoft Antimalware para Azure Cloud Services y Virtual Machines es una funcionalidad de protección en tiempo real que permite identificar y eliminar virus, spyware y otro software malintencionado. Microsoft Antimalware activa alertas configurables cuando software no deseado o malintencionado intenta instalarse o ejecutarse en los sistemas de Azure.

[Azure Backup](../../backup/backup-overview.md) es una solución escalable que protege los datos de su aplicación sin necesidad de realizar ninguna inversión y afrontando unos costos operativos mínimos. Los errores de una aplicación pueden dañar los datos, y los errores humanos pueden crear errores en las aplicaciones. Con Azure Backup, las máquinas virtuales que ejecutan Windows y Linux están protegidas.

[Azure Site Recovery](../../site-recovery/site-recovery-overview.md) ayuda a coordinar la replicación, la conmutación por error y la recuperación de aplicaciones y cargas de trabajo para que estén disponibles desde una ubicación secundaria si la ubicación principal deja de funcionar.

## <a name="ensure-compliance-cloud-services-due-diligence-checklist"></a>Asegurar el cumplimiento: Lista de comprobación de diligencia debida para Cloud Services

Microsoft ha desarrollado [la lista de comprobación de diligencia debida para Cloud Services](https://aka.ms/cloudchecklist.download) para ayudar a las organizaciones a actuar con la debida diligencia cuando están pensando trasladarse a la nube. Proporciona una estructura que ayuda a las organizaciones de cualquier tipo y tamaño, desde empresas privadas a organizaciones públicas, incluidos organismos gubernamentales de todos los niveles y organizaciones sin ánimo de lucro, a identificar su propio rendimiento, servicio, administración de datos y objetivos y requisitos de gobernanza. Esto les permite comparar las ofertas de los distintos proveedores de servicios en la nube que, en última instancia, forman la base para un contrato de este tipo de servicios.

Esta lista de comprobación proporciona un marco que se adecua cláusula por clausula a un nuevo estándar internacional de contratos de servicios en la nube, la norma ISO/IEC 19086. Esta norma ofrece un conjunto unificado de consideraciones que ayuda a las organizaciones a la hora de tomar decisiones acerca de la adopción de la nube y permiten crear una base para la comparación entre las ofertas de servicios en la nube.

La lista de comprobación promueve un movimiento muy estudiado en favor de la nube, proporciona una guía estructurada y un enfoque coherente y repetible para elegir un proveedor de servicios en la nube.

La adopción de la nube ya no es solo una decisión tecnológica. Dado que los requisitos de la lista de comprobación abarcan todos los aspectos de una organización, sirven para reunir a todos los encargados de la toma de decisiones: desde el CIO y el CISO a los profesionales de los departamentos legales, de administración de riesgos y de cumplimiento normativo. Esto aumenta la eficacia del proceso de toma de decisiones y la toma de decisiones fundamentadas en sólidos razonamientos, lo que reduce la probabilidad de obstáculos imprevistos en el proceso de adopción.

Además, la lista de comprobación:

- Muestra temas de discusión clave para los encargados de tomar decisiones al principio del proceso de adopción de la nube.

- Fomenta debates exhaustivos en la empresa acerca de la normativa y los objetivos de la organización en cuanto a privacidad, información de identificación personal (PII) y seguridad de los datos.

- Ayuda a las organizaciones a identificar posibles problemas que pudieran afectar a un proyecto en la nube.

- Proporciona un conjunto coherente de preguntas con los mismos términos, definiciones, métricas y resultados para cada proveedor, para simplificar el proceso de comparación entre las ofertas de los distintos proveedores de servicios en la nube.

## <a name="azure-infrastructure-and-application-security-validation"></a>Validación de la seguridad de la infraestructura y aplicaciones de Azure

Con [seguridad operativa de Azure](operational-security.md), se hace referencia a los servicios, los controles y las características disponibles para los usuarios para proteger sus datos, aplicaciones y otros recursos en Microsoft Azure.

![validación de seguridad (detección)](./media/technical-capabilities/azure-security-technical-capabilities-fig7.png)

La seguridad operativa de Azure se basa en una plataforma que incorpora el conocimiento adquirido a través de diversas funcionalidades exclusivas de Microsoft, incluido el ciclo de vida de desarrollo de seguridad (SDL) de Microsoft, el programa Microsoft Security Response Center y un conocimiento en profundidad del panorama de amenazas de ciberseguridad.

### <a name="microsoft-azure-monitor"></a>Microsoft Azure Monitor

[Azure Monitor](../../azure-monitor/index.yml) es la solución de administración de TI para la nube híbrida. Si se usa de forma independiente o para extender la implementación existente de System Center, los registros de Azure Monitor ofrecen la máxima flexibilidad y control para la administración basada en la nube de su infraestructura.

![Azure Monitor](./media/technical-capabilities/azure-security-technical-capabilities-fig8.png)

Con Azure Monitor, puede administrar cualquier instancia en cualquier nube, incluidas instancias en entornos locales, Azure, AWS, Windows Server, Linux, VMware y OpenStack, a un costo menor que el de las soluciones de la competencia. Diseñado para el mundo que da prioridad a la nube, Azure Monitor ofrece un nuevo método de administración empresarial que es la forma más rápida y rentable de enfrentarse a los nuevos retos empresariales y adaptarse a nuevas cargas de trabajo, aplicaciones y entornos de nube.

### <a name="azure-monitor-logs"></a>Registros de Azure Monitor

Los [registros de Azure Monitor](https://azure.microsoft.com/documentation/services/log-analytics) proporcionan servicios de supervisión al recopilar datos de los recursos administrados en un repositorio central. Estos datos podrían incluir eventos, datos de rendimiento o datos personalizados proporcionados a través de la API. Una vez recopilados, los datos están disponibles para las alertas, el análisis y la exportación.

![Registros de Azure Monitor](./media/technical-capabilities/azure-security-technical-capabilities-fig9.png)

Este método permite consolidar datos de varios orígenes, por lo que puede combinar datos de los servicios de Azure con el entorno local existente. También separa claramente la recopilación de los datos de la acción realizada en los datos, para que todas las acciones estén disponibles para todos los tipos de datos.

### <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](../../security-center/security-center-intro.md) ayuda a evitar, detectar y responder a amenazas con más visibilidad y control sobre la seguridad de sus recursos de Azure. Proporciona administración de directivas y supervisión de la seguridad integrada en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con un amplio ecosistema de soluciones de seguridad.

El Centro de seguridad analiza el estado de seguridad de los recursos de Azure para identificar posibles vulnerabilidades de seguridad. Una lista de recomendaciones le guiará por el proceso de configuración de los controles necesarios.

Algunos ejemplos son:

- Aprovisionamiento de antimalware para ayudar a identificar y eliminar el software malintencionado.

- Configuración de reglas y grupos de seguridad de red para controlar el tráfico a las VM

- Aprovisionamiento de firewalls de aplicaciones web para ayudar a defenderse contra ataques dirigidos a las aplicaciones web.

- Implementación de actualizaciones del sistema que faltan.

- Resolución de las configuraciones de sistema operativo que no coinciden con las líneas base recomendadas.

El Centro de seguridad recopila, analiza e integra automáticamente los datos de registro de los recursos de Azure, la red y soluciones de asociados como firewalls y programas antimalware. Cuando se detecten amenazas, se creará una alerta de seguridad. Como ejemplos se incluye la detección de:

- VM en peligro que se comunican con direcciones IP malintencionadas conocidas

- Malware avanzado detectado mediante la generación de informes de errores de Windows.

- Ataques de fuerza bruta contra VM

- Alertas de seguridad de programas antimalware y firewalls integrados

### <a name="azure-monitor"></a>Azure Monitor

[Azure Monitor](../../azure-monitor/overview.md) proporciona punteros a la información sobre determinados tipos de recursos. Ofrece funciones de visualización, consulta, enrutamiento, alertas, escalado automático y automatización de los datos tanto de la infraestructura de Azure (registro de actividad) como de cada recurso individual de Azure (registros de diagnóstico).

Las aplicaciones de nube son complejas y tienen muchas partes móviles. La supervisión proporciona datos para garantizar que la aplicación permanece en funcionamiento en un estado correcto. También ayuda a evitar posibles problemas o a solucionar los existentes.

![Azure Monitor](./media/technical-capabilities/azure-security-technical-capabilities-fig10.png) Además, puede usar datos de supervisión para obtener un conocimiento más profundo sobre su aplicación. Este conocimiento puede ayudarle a mejorar el rendimiento o mantenimiento de la aplicación, o a automatizar acciones que de lo contrario requerirían intervención manual.

La auditoría de la seguridad de la red es fundamental para detectar vulnerabilidades de red y asegurar el cumplimiento de la seguridad de TI y el modelo de gobernanza reglamentario. Con la vista Grupo de seguridad, puede recuperar las reglas de seguridad y de grupo de seguridad de red configuradas, así como las reglas de seguridad efectivas. Con la lista de reglas que se aplican, puede determinar los puertos que están abiertos y evaluar la vulnerabilidad de la red.

### <a name="network-watcher"></a>Network Watcher

[Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) es un servicio regional que permite supervisar y diagnosticar problemas en un nivel de red en Azure. Las herramientas de visualización y diagnóstico de red que incluye Network Watcher le ayudan a conocer, diagnosticar y obtener información acerca de cualquier red de Azure. Este servicio incluye captura de paquetes, próximo salto, comprobación de flujo de IP, vista de grupos de seguridad y registros de flujo de NSG. La supervisión en el nivel de escenario, ofrece una visión global de los recursos de la red que contrasta con la supervisión de recursos de red individuales.

### <a name="storage-analytics"></a>Storage Analytics

[Storage Analytics](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics) puede almacenar métricas que incluyen estadísticas de las transacciones y datos de capacidad agregados sobre las solicitudes realizadas a un servicio de almacenamiento. Las transacciones se notifican tanto en el nivel de operación de API como en el nivel de servicio de almacenamiento, y la capacidad se notifica en el nivel de servicio de almacenamiento. Los datos de las métricas se pueden utilizar para analizar el uso del servicio de almacenamiento, diagnosticar problemas con las solicitudes realizadas en el mismo y mejorar el rendimiento de las aplicaciones que utilizan un servicio.

### <a name="application-insights"></a>Application Insights

[Application Insights](../../azure-monitor/app/app-insights-overview.md) es un servicio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas. Úselo para supervisar la aplicación web en directo. Se detectarán automáticamente las anomalías de rendimiento. Incluye herramientas de análisis eficaces que le ayudan a diagnosticar problemas y comprender lo que hacen realmente los usuarios con la aplicación. Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso. Funciona con diversas aplicaciones y en una amplia variedad de plataformas, como .NET, Node.js o Java EE, tanto hospedadas localmente como en la nube. Se integra con el proceso de devOps y tiene puntos de conexión para diversas herramientas de desarrollo.

Supervisa:

- **Tasas de solicitud, tiempos de respuesta y tasas de error** - Averigüe qué páginas que son las más conocidas, en qué momento del día y dónde están los usuarios. Vea qué páginas presentan mejor rendimiento. Si los tiempos de respuesta y las tasas de error aumentan cuando hay más solicitudes, quizás tiene un problema de recursos.

- **Tasas de dependencia, tiempos de respuesta y tasa de error** - Averigüe si los servicios externos le ralentizan.

- **Excepciones**: - Analice las estadísticas agregadas o seleccione instancias concretas y profundice en el seguimiento de la pila y las solicitudes relacionadas. Se notifican tanto las excepciones de servidor como las de explorador.

- **Vistas de página y rendimiento de carga** - Notificados por los exploradores de los usuarios.

- **Llamadas AJAX desde páginas web**: Tasas, tiempos de respuesta y tasas de error.

- **Número de usuarios y sesiones**.

- **Contadores de rendimiento** de las máquinas de servidor de Windows o Linux, como CPU, memoria y uso de la red.

- **Diagnóstico de host** de Docker o Azure.

- **Registros de seguimiento de diagnóstico** de la aplicación - De esta forma puede correlacionar eventos de seguimiento con las solicitudes.

- **Métricas y eventos personalizados** que usted mismo escribe en el código de cliente o servidor para realizar un seguimiento de eventos empresariales, como artículos vendidos o partidas ganadas.

La infraestructura de la aplicación está constituida normalmente por varios componentes: quizás una máquina virtual, una cuenta de almacenamiento y una red virtual, o una aplicación web, una base de datos, un servidor de bases de datos y servicios de terceros. Estos componentes no se ven como entidades independientes, sino como partes de una sola entidad relacionadas e interdependientes. Desea implementarlos, administrarlos y supervisarlos como grupo. [Azure Resource Manager](../../azure-resource-manager/management/overview.md) permite trabajar con los recursos de la solución como grupo.

Todos los recursos de la solución se pueden implementar, actualizar o eliminar en una sola operación coordinada. Para realizar la implementación se usa una plantilla, que puede funcionar en distintos entornos, como producción, pruebas y ensayo. Administrador de recursos proporciona funciones de seguridad, auditoría y etiquetado que le ayudan a administrar los recursos después de la implementación.

**Ventajas de usar Resource Manager**

Administrador de recursos ofrece varias ventajas:

- Puede implementar, administrar y supervisar todos los recursos de la solución en grupo, en lugar de controlarlos individualmente.

- Puede implementar la solución repetidamente a lo largo del ciclo de vida del desarrollo y tener la seguridad de que los recursos se implementan de forma coherente.

- Puede administrar la infraestructura mediante plantillas declarativas en lugar de scripts.

- Puede definir las dependencias entre recursos de modo que se implementen en el orden correcto.

- Puede aplicar control de acceso a todos los servicios del grupo de recursos al integrarse de forma nativa Control de acceso basado en rol (RBAC) en la plataforma de administración.

- Puede aplicar etiquetas a los recursos para organizar de manera lógica todos los recursos de la suscripción.

- Puede aclarar la facturación de su organización viendo los costos de un grupo de recursos que compartan la misma etiqueta.

> [!Note]
> Resource Manager ofrece una nueva manera de implementar y administrar las soluciones. Si usó el anterior modelo de implementación y desea obtener más información sobre los cambios, consulte [Descripción de la implementación de Resource Manager y la implementación clásica](../../azure-resource-manager/management/deployment-models.md).

## <a name="next-steps"></a>Pasos siguientes

Obtener más información acerca de la seguridad con la lectura de algunos de nuestros detallados temas de seguridad:

- [Auditoría y registro](https://www.microsoft.com/en-us/trustcenter/security/auditingandlogging)

- [Delitos informáticos](https://www.microsoft.com/en-us/trustcenter/security/cybercrime)

- [Diseño y seguridad operativa](https://www.microsoft.com/en-us/trustcenter/security/designopsecurity)

- [Cifrado](https://www.microsoft.com/en-us/trustcenter/security/encryption)

- [Administración de identidades y acceso](https://www.microsoft.com/en-us/trustcenter/security/identity)

- [Seguridad de las redes](https://www.microsoft.com/en-us/trustcenter/security/networksecurity)

- [Administración de amenazas](https://www.microsoft.com/en-us/trustcenter/security/threatmanagement)
