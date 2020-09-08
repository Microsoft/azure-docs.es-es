---
title: Planear la implementación de Azure Active Directory Application Proxy
description: Una guía completa para planear la implementación de Application Proxy en la organización
services: active-directory
documentationcenter: azure
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: kenwith
ms.openlocfilehash: 09a930778e35897671d10f14a95f3fa48ea5e9eb
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642425"
---
# <a name="plan-an-azure-ad-application-proxy-deployment"></a>Planear una implementación de Azure AD Application Proxy

Azure Active Directory (Azure AD) Application Proxy es una solución de acceso remoto segura y rentable a todas las aplicaciones locales. Proporciona una ruta de acceso de transición inmediata para las organizaciones de tipo "La Nube primero" para administrar el acceso a las aplicaciones locales heredadas que aún no pueden usar protocolos modernos. Para obtener más información introductoria, consulte [¿Qué es Application Proxy?](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy)

Application Proxy se recomienda para dar a los usuarios remotos acceso a recursos internos. Application Proxy reemplaza la necesidad de una VPN o proxy inverso para estos casos de uso de acceso remoto. No está pensado para los de la red corporativa. Estos usuarios que usan Application Proxy para el acceso a la intranet pueden experimentar problemas de rendimiento no deseado.

En este artículo se incluyen los recursos que necesita para planificar, operar y administrar Azure AD Application Proxy.

## <a name="plan-your-implementation"></a>Planeamiento de la implementación

En la siguiente sección se proporciona una amplia visión de los elementos de planeamiento que le prepararán para una experiencia de implementación eficaz.

### <a name="prerequisites"></a>Requisitos previos

Debe cumplir los siguientes requisitos previos antes de iniciar la implementación. Puede ver más información sobre cómo configurar su entorno, incluidos estos requisitos previos, en este [tutorial](application-proxy-add-on-premises-application.md).

* **Conectores**: Los conectores son agentes ligeros que puede implementar en:
   * Hardware físico en el entorno local
   * Una VM hospedada en cualquier solución de hipervisor
   * Una VM hospedada en Azure para habilitar una conexión saliente con el servicio Application Proxy.

* Consulte [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md) para obtener una descripción más detallada.

     * Las máquinas del conector [se deben habilitar para TLS 1.2](application-proxy-add-on-premises-application.md) antes de instalar los conectores.

     * Si es posible, implemente los conectores en la [misma red](application-proxy-network-topology.md) y segmento que los servidores de aplicaciones web de back-end. Es mejor implementar los conectores después de completar una detección de aplicaciones.
     * Se recomienda que cada grupo de conectores tenga al menos dos conectores para proporcionar alta disponibilidad y escala. Tener tres conectores es una situación óptima en caso de que necesite realizar tareas de servicio en una máquina en cualquier momento. Revise la [tabla de capacidad de los conectores](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-connectors#capacity-planning) para obtener ayuda para decidir en qué tipo de máquina instalará conectores. Cuanto mayor sea la máquina, más búfer y mayor rendimiento tendrá el conector.

* **Configuración de acceso a la red**: los conectores de Azure AD Application Proxy [se conectan a Azure por HTTPS (puerto TCP 443) y HTTP (puerto TCP 80)](application-proxy-add-on-premises-application.md).

   * La interrupción del tráfico TLS del conector no se admite y evitará que los conectores establezcan un canal seguro con sus respectivos puntos de conexión de Azure App Proxy.

   * Evite todo tipo de inspección insertada en las comunicaciones TLS salientes entre los conectores y Azure. La inspección interna entre un conector y las aplicaciones de back-end es posible, pero podría degradar la experiencia del usuario y, por lo tanto, no se recomienda.

   * Tampoco se admite, ni se necesita, el equilibrio de carga de los propios conectores.

### <a name="important-considerations-before-configuring-azure-ad-application-proxy"></a>Consideraciones importantes antes de configurar Azure AD Application Proxy

Deben cumplirse los siguientes requisitos principales para configurar e implementar Azure AD Application Proxy.

*  **Incorporación de Azure**: Antes de implementar Application Proxy, las identidades de usuario se deben sincronizar desde un directorio local o bien se deben crear directamente desde los inquilinos de Azure AD. La sincronización de identidades permite a Azure AD realizar una autenticación previa de los usuarios antes de concederles acceso a aplicaciones publicadas en App Proxy y tener la información del identificador de usuario necesarios para realizar el inicio de sesión único (SSO).

* **Requisitos de acceso condicional**: No se recomienda el uso de Application Proxy para el acceso de intranet, ya que esto agrega una latencia que afectará a los usuarios. Es recomendable usar Application Proxy con las directivas de autenticación previa y acceso condicional para el acceso remoto desde Internet.  Un enfoque para proporcionar acceso condicional para su uso en la intranet es modernizar las aplicaciones para que se puedan autenticar directamente con AAD. Consulte [Resources for migrating applications to AAD](https://docs.microsoft.com/azure/active-directory/manage-apps/migration-resources) (Recursos para migrar aplicaciones a AAD) para obtener más información.

* **Límites de servicio**: Para protegerse frente al consumo excesivo de recursos por parte de inquilinos individuales, existen límites definidos por aplicación e inquilino. Para ver estos límites, consulte [Restricciones y límites del servicio Azure AD](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-service-limits-restrictions). Estos límites se basan en un banco de pruebas muy superior al volumen de uso típico y proporcionan un búfer suficiente para la mayoría de las implementaciones.

* **Certificado público**: Si usa nombres de dominio personalizados, debe adquirir un certificado TLS/SSL. Según los requisitos de la organización, obtener un certificado puede llevar algún tiempo y se recomienda comenzar el proceso lo antes posible. Azure Application Proxy admite certificados estándar, [comodín](application-proxy-wildcard.md) o basados en SAN. Para más información, consulte [Configuración de dominios personalizados con Azure AD Application Proxy](application-proxy-configure-custom-domain.md).

* **Requisitos de dominio**: El inicio de sesión único en las aplicaciones publicadas con la delegación restringida de Kerberos (KCD) requiere que el servidor que ejecuta el conector y el servidor que ejecuta la aplicación estén unidos a un dominio y formen parte del mismo dominio o los mismos dominios de confianza.
Para obtener más información sobre el tema, consulte [KCD para el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md) con Application Proxy. El servicio de conectores se ejecuta en el contexto del sistema local y no debe configurarse para usar una identidad personalizada.

* **Registros de DNS para direcciones URL**

   * Antes de usar dominios personalizados en Application Proxy, debe crear un registro CNAME en un DNS público, lo que permite a los clientes resolver la dirección URL externa y personalizada definida para la dirección predefinida de Application Proxy. Si no se puede crear un registro CNAME para una aplicación que usa un dominio personalizado, los usuarios remotos no podrán conectarse a la aplicación. Los pasos necesarios para agregar registros CNAME pueden variar de un proveedor de DNS a otro, por lo que es importante aprender a [administrar conjuntos de registros y registros DNS mediante Azure Portal](https://docs.microsoft.com/azure/dns/dns-operations-recordsets-portal).

   * De forma similar, los hosts de conector deben poder resolver la dirección URL interna de las aplicaciones que se publican.

* **Derechos y roles administrativos**

   * La **instalación del conector** requiere derechos de administrador local en el servidor Windows en que se instala. También requiere un mínimo de un rol de *Administrador de la aplicación* para autenticarse y registrar la instancia del conector para su inquilino de Azure AD.

   * La **publicación y administración de aplicaciones** requiere el rol *Administrador de la aplicación*. Los administradores de aplicaciones pueden administrar todas las aplicaciones del directorio, incluidos los registros, configuraciones de SSO, asignaciones y configuración de licencias de usuario y de grupo, configuración del proxy de aplicación y consentimiento. No concede la capacidad de administrar el acceso condicional. El rol *Administrador de aplicaciones en la nube* tiene todas las funcionalidades del administrador de la aplicación, excepto en que no permite la administración de la configuración de Application Proxy.

* **Licencia**: Application Proxy está disponible mediante una suscripción a Azure AD Premium. Consulte la [página precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/) para obtener una lista completa de características y opciones de licencia.

### <a name="application-discovery"></a>Detección de aplicaciones

Para compilar un inventario de todas las aplicaciones del ámbito que se están publicando mediante Application Proxy, recopile la siguiente información:

| Tipo de información| Información que se recopilará |
|---|---|
| Tipo de servicio| Por ejemplo: SharePoint, SAP, CRM, aplicación web personalizada, API |
| Plataforma de aplicaciones | Por ejemplo: Windows IIS, Apache en Linux, Tomcat, NGINX |
| Pertenencia a dominio| Nombre de dominio completo (FQDN) del servidor web |
| Ubicación de la aplicación | Dónde se encuentra el servidor web o la granja en su infraestructura |
| Acceso interno | Dirección URL exacta usada al acceder a la aplicación internamente. <br> En una granja, ¿qué tipo de equilibrio de carga se usa? <br> Si la aplicación dibuja el contenido de orígenes distintos al propio.<br> Determina si la aplicación funciona a través de WebSockets. |
| Acceso externo | Solución de proveedor a que está expuesta la aplicación externamente. <br> Dirección URL que quiere usar para el acceso externo. Si se trata de SharePoint, asegúrese de que se configuren asignaciones de acceso alternativas de acuerdo con [esta guía](https://docs.microsoft.com/SharePoint/administration/configure-alternate-access-mappings). Si no es así, deberá definir direcciones URL externas. |
| Certificado público | Si usa un dominio personalizado, adquiera un certificado con un nombre de firmante correspondiente. Si existe algún certificado, tenga en cuenta el número de serie y la ubicación desde donde se puede obtener. |
| Tipo de autenticación| El tipo de autenticación admitido por la aplicación; por ejemplo, básica, compatibilidad de aplicaciones, como Basic, autenticación de integración de Windows, basada en formularios, basada en encabezados y notificaciones. <br>Si la aplicación está configurada para ejecutarse en una cuenta de dominio específica, anote el nombre de dominio completo (FQDN) de la cuenta de servicio.<br> Si se basa en SAML, las direcciones URL de respuesta y el identificador. <br> Si se basa en el encabezado, la solución de proveedor y el requisito específico para controlar el tipo de autenticación. |
| Nombre del grupo de conectores | Nombre lógico para el grupo de conectores que se designará para proporcionar la canalización y el SSO a esta aplicación de back-end. |
| Acceso de usuarios y grupos | Los usuarios o grupos de usuarios a que se concederá acceso externo a la aplicación. |
| Requisitos adicionales | Tenga en cuenta cualquier acceso remoto o requisitos de seguridad adicionales que se deben tener en cuenta al publicar la aplicación. |

Puede descargar esta [hoja de cálculo de inventario de la aplicación](https://aka.ms/appdiscovery) para hacer el inventario de las aplicaciones.

### <a name="define-organizational-requirements"></a>Definir los requisitos organizativos

Las siguientes son las áreas para las que debe definir los requisitos empresariales de la organización. Cada área contiene ejemplos de requisitos

 **Acceder**

* Los usuarios remotos con uniones a un dominio o con dispositivos unidos a Azure AD pueden tener acceso a las aplicaciones publicadas de forma segura con inicio de sesión único (SSO) de conexión directa.

* Los usuarios remotos con dispositivos personales aprobados pueden acceder de forma segura a las aplicaciones publicadas siempre que estén inscritos en MFA y hayan registrado la aplicación Microsoft Authenticator en su teléfono móvil como método de autenticación.

**Gobernanza**

* Los administradores pueden definir y supervisar el ciclo de vida de las asignaciones de usuario a las aplicaciones publicadas mediante Application Proxy.

**Seguridad**

* Solo los usuarios asignados a las aplicaciones a través de la pertenencia a grupos o individualmente pueden acceder a esas aplicaciones.

**Rendimiento**

* No hay ninguna degradación del rendimiento de la aplicación en comparación con el acceso a la aplicación desde la red interna.

**Experiencia del usuario**

* Los usuarios saben cómo acceder a sus aplicaciones mediante el uso de direcciones URL de empresa familiares en cualquier plataforma de dispositivo.

**Auditoría**
* Los administradores pueden auditar la actividad de acceso del usuario.


### <a name="best-practices-for-a-pilot"></a>Procedimientos recomendados para un piloto

Determine la cantidad de tiempo y esfuerzo necesarios para programar completamente una sola aplicación para el acceso remoto con el inicio de sesión único (SSO). Para hacerlo, ejecute un programa piloto que tenga en cuenta su detección inicial, publicación y pruebas generales. Usar una sencilla aplicación web basada en IIS preconfigurada para la autenticación integrada de Windows (IWA) ayudaría a establecer una línea de base, ya que este programa de instalación requiere un esfuerzo mínimo para acceder correctamente al acceso remoto del piloto y el inicio de sesión único.

Los siguientes elementos de diseño deben aumentar el éxito de su implementación piloto directamente en un inquilino de producción.

**Administración del conector**:

* Los conectores desempeñan un papel clave a la hora de proporcionar un conducto local a las aplicaciones. El uso del grupo de conectores **predeterminado** es adecuado para la prueba piloto inicial de las aplicaciones publicadas antes de autorizarlas para producción. A continuación, las aplicaciones probadas correctamente se pueden mover a grupos de conectores de producción.

**Administración de aplicaciones**:

* Es probable que el personal recuerde que una URL externa es familiar y pertinente. Evite publicar su aplicación mediante nuestros sufijos msappproxy.net o onmicrosoft.com predefinidos. En su lugar, proporcione un dominio verificado familiar de nivel superior, con un prefijo de nombre de host lógico, como *intranet.<dominio_de_clientes>.com*.

* Para restringir la visibilidad del icono de la aplicación piloto a un grupo piloto, oculte su forma de icono de inicio en el portal Mis aplicaciones de Azure. Cuando esté listo para producción, puede restringir la aplicación a su respectivo público de destino en el mismo inquilino de preproducción o publicando la aplicación en el inquilino de producción.

**Configuración del inicio de sesión único** Algunas opciones de SSO tienen dependencias específicas que pueden tardar tiempo en configurarse, así que, para evitar retrasos en el control de cambios, compruebe que las dependencias se abordan con antelación. Esto incluye que los hosts de conector de unión a un dominio realicen el SSO mediante la delegación restringida de Kerberos y que asuman otras actividades que requieren mucho tiempo. Por ejemplo, configurar una instancia de PING Access, si necesita SSO basado en encabezado.

**TLS entre el host de conector y la aplicación de destino**: La seguridad es fundamental, por lo que siempre se debe usar TLS entre el host del conector y las aplicaciones de destino. Especialmente si la aplicación web está configurada para la autenticación basada en formularios (FBA), ya que, en ese caso, las credenciales de usuario se transmiten de forma eficaz en texto no cifrado.

**Implementar de forma incremental y probar cada paso**.
Realice pruebas funcionales básicas después de publicar una aplicación para asegurarse de que se cumplen todos los requisitos empresariales y de usuario siguiendo estas instrucciones:

1. Probar y validar el acceso general a la aplicación web con la autenticación previa deshabilitada.
2. Si se realiza correctamente, habilite la autenticación previa y asigne usuarios y grupos. Pruebe y valide el acceso.
3. A continuación, agregue el método de SSO para la aplicación y vuelva a probarlo para validar el acceso.
4. Aplique el acceso condicional y las directivas de MFA según sea necesario. Pruebe y valide el acceso.

**Herramientas para la solución de problemas**: A la hora de solucionar problemas, empiece siempre por validar el acceso a la aplicación publicada desde el explorador en el host de conexión y confirme que la aplicación funciona según lo previsto. Cuanto más simple sea la configuración, más fácil será determinar la causa principal, de modo que es recomendable intentar reproducir los problemas con una configuración mínima; por ejemplo, un único conector y sin SSO. En algunos casos, las herramientas de depuración web, como Fiddler, de Telerik, pueden resultar indispensables para solucionar problemas de acceso o de contenido en aplicaciones a las que se accede mediante un servidor proxy. Fiddler también puede actuar como un proxy para ayudar a hacer el seguimiento y la depuración del tráfico para plataformas móviles como iOS y Android y prácticamente cualquier cosa que se pueda configurar para redirigirse a través de un servidor proxy. Consulte la [guía de solución de problemas](application-proxy-troubleshoot.md) para obtener más información.

## <a name="implement-your-solution"></a>Implementación de la solución

### <a name="deploy-application-proxy"></a>Implementar Application Proxy

Los pasos para implementar Application Proxy se tratan en este [tutorial para agregar una aplicación local para el acceso remoto](application-proxy-add-on-premises-application.md). Si no se completa correctamente la instalación, seleccione **Solucionar problemas de Application Proxy** en el portal o use la guía de solución de problemas [para los problemas de instalación del conector del agente de Application Proxy](application-proxy-connector-installation-problem.md).

### <a name="publish-applications-via-application-proxy"></a>Publicar aplicaciones mediante Application Proxy

Para la publicación de aplicaciones, se asume que cumple todos los requisitos previos y que tiene varios conectores que se muestran como registrados y activos en la página de Application Proxy.

Para publicar aplicaciones, también puede usar [PowerShell](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview).

A continuación se incluyen algunas prácticas recomendadas para publicar una aplicación:

* **Use grupos de conectores**: asigne un grupo de conectores que se haya designado para la publicación de cada aplicación respectiva. Se recomienda que cada grupo de conectores tenga al menos dos conectores para proporcionar alta disponibilidad y escala. Tener tres conectores es una situación óptima en caso de que necesite realizar tareas de servicio en una máquina en cualquier momento. Además, consulte [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md) para ver cómo puede utilizar también grupos de conectores para segmentar los conectores por red o ubicación.

* **Defina el tiempo de espera de las aplicaciones de back-end**: este parámetro resulta útil en escenarios en que la aplicación puede requerir más de 75 segundos procesar una transacción de cliente. Por ejemplo, cuando un cliente envía una consulta a una aplicación web que actúa como un front-end para una base de datos. El front-end envía esta consulta al servidor de base de datos de back-end y espera una respuesta, pero, cuando la recibe, se agota el tiempo de espera de la conversación del lado cliente. Establecer el tiempo de espera en Largo ofrece 180 segundos para que se completen las transacciones más grandes.

* **Use tipos de cookies adecuados**

   * **Cookie solo HTTP**: Proporciona seguridad adicional, ya que Application Proxy incluye la marca HTTPOnly en los encabezados de respuesta HTTP set-cookie. Este parámetro ayuda a mitigar vulnerabilidades como ataques de scripts de sitios (XSS). Defina esta opción como No para los agentes de usuario o clientes que necesiten acceder a la cookie de sesión. Por ejemplo, un cliente RDP/MTSC que se conecta a una Puerta de enlace de Escritorio remoto mediante App Proxy.

   * **Cookie segura**: Cuando se define una cookie con el atributo Secure, el agente de usuario (aplicación del lado cliente) solo incluirá la cookie en las solicitudes HTTP si la solicitud se transmite a través de un canal TLS seguro. Esta opción ayuda a mitigar el riesgo de que una cookie se ponga en peligro a través de canales de texto no cifrado, por lo que debe estar habilitada.

   * **Cookie persistente**: Permite que la cookie de sesión de Application Proxy persista entre cierres del explorador y siga siendo válida hasta que expire o se elimine. Se usa para escenarios donde una aplicación enriquecida, como Office, accede a un documento dentro de una aplicación web publicada, sin que se vuelva a solicitar al usuario que se autentique. Sin embargo, se debe habilitar con precaución, ya que las cookies persistentes pueden dejar un servicio con el riesgo de un acceso no autorizado si no se usan junto con otros controles de compensación. Esta configuración solo debe usarse para las aplicaciones más antiguas que no pueden compartir cookies entre procesos. Es mejor actualizar la aplicación para controlar el uso compartido de cookies entre procesos, en lugar de usar esta configuración.

* **Traducir las URL en encabezados**: Habilite esta opción para escenarios donde no se puede configurar el DNS interno para que coincida con el espacio de nombres público de la organización (conocido también como DNS dividido). A menos que la aplicación requiera el encabezado de host original en la solicitud de cliente, deje este valor definido como Sí. La alternativa es que el conector use FQDN en la dirección URL interna para el redireccionamiento del tráfico real y el FQDN en la dirección URL externa como encabezado de host. En la mayoría de los casos, esta alternativa debe permitir que la aplicación funcione con normalidad cuando se acceda a ella de forma remota, pero sus usuarios pierden las ventajas de tener una coincidencia dentro y fuera de la dirección URL.

* **Traducir direcciones URL en el cuerpo de la aplicación**: Active la traducción de vínculos del cuerpo de la solicitud de una aplicación cuando quiera que los vínculos de la aplicación se traduzcan en las respuestas que se devuelvan al cliente. Si está habilitada, esta función proporciona un intento de mejor esfuerzo de traducción de todos los vínculos internos que App Proxy encuentra en las respuestas HTML y CSS que se devuelven a los clientes. Resulta útil a la hora de publicar aplicaciones que contienen vínculos de nombre corto NetBIOS o absolutos de codificación rígida en el contenido o aplicaciones con contenido que se vincula a otras aplicaciones locales.

Para los escenarios en que una aplicación publicada vincula a otras aplicaciones publicadas, habilite la traducción de vínculos para cada aplicación de modo que pueda tener control sobre la experiencia del usuario en el nivel de aplicación.

Por ejemplo, imagine que tiene tres aplicaciones publicadas a través del Proxy de aplicación que se vinculan entre sí: Benefits, Expenses y Travel, más una cuarta aplicación, Feedback, que no está publicada mediante Application Proxy.

![Imagen 1](media/App-proxy-deployment-plan/link-translation.png)

Cuando habilita la traducción de vínculos para la aplicación Beneficios, los vínculos a Gastos y Viajes se redirigen a las direcciones URL externas de esas aplicaciones para que los usuarios que acceden a las aplicaciones desde fuera de la red corporativa puedan acceder a ellas. Los vínculos de Gastos y Viajes a Beneficios no funcionan, porque la traducción de vínculos no está habilitada para esas dos aplicaciones. El vínculo a los comentarios no se redirige porque no hay ninguna dirección URL externa, por lo que los usuarios que usan la aplicación Benefits no podrán acceder a la aplicación Feedback desde fuera de la red corporativa. Consulte la información detallada en [Vincular traducción y otras opciones de redireccionamiento](application-proxy-configure-hard-coded-link-translation.md).

### <a name="access-your-application"></a>Acceso a la aplicación

Existen varias opciones para administrar el acceso a los recursos publicados de App Proxy. Elija el más adecuado para el escenario que vaya a usar y sus necesidades de escalabilidad. Los enfoques comunes incluyen el uso de grupos locales sincronizados a través de Azure AD Connect, la creación de grupos dinámicos en Azure AD basada en los atributos del usuario mediante grupos de autoservicio administrados por un propietario del recurso, o bien una combinación de todos estos factores. Consulte en los recursos vinculados sus respectivos beneficios.

La manera más sencilla de asignar acceso a los usuarios para una aplicación es ir a las opciones **Usuarios y grupos** desde el panel izquierdo de la aplicación publicada y asignar directamente grupos o individuos.

![Imagen 24](media/App-proxy-deployment-plan/add-user.png)

También puede permitir que los usuarios accedan mediante el autoservicio a su aplicación asignando un grupo del que no son miembros y configurando las opciones de autoservicio.

![Imagen 25](media/App-proxy-deployment-plan/allow-access.png)

Si está habilitada, los usuarios, a continuación, podrán iniciar sesión en el portal Mis aplicaciones y solicitar acceso. Después, bien se les aprobará automáticamente y se les agregará al grupo de autoservicio permitido o necesitarán la aprobación de un aprobador designado.

También se puede invitar a los usuarios invitados a [acceder a las aplicaciones internas publicadas mediante Application Proxy a través de Azure AD B2B](https://docs.microsoft.com/azure/active-directory/b2b/add-users-information-worker).

Para las aplicaciones locales a las que normalmente se puede acceder de forma anónima y que no requieren ninguna autenticación, puede que prefiera deshabilitar la opción situada en las **Propiedades** de la aplicación.

![Imagen 26](media/App-proxy-deployment-plan/assignment-required.png)


Dejar esta opción definida como No permite a los usuarios acceder a la aplicación local a través de Azure AD App Proxy sin permisos, así que debe usarla con precaución.

Una vez publicada la aplicación, se debe poder acceder a ella escribiendo su URL externa en un explorador o por su icono en [https://myapps.microsoft.com](https://myapps.microsoft.com/).

### <a name="enable-pre-authentication"></a>Habilitar la autenticación previa

Compruebe que se pueda acceder a la aplicación mediante Application Proxy con la dirección URL externa.

1. Vaya a **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones** y elija la aplicación que quiere administrar.

2. Seleccione **Proxy de la aplicación**.

3. En el campo **Autenticación previa**, use la lista desplegable para seleccionar **Azure Active Directory** y seleccione **Guardar**.

Con la autenticación previa habilitada, Azure AD presentará un desafío para los usuarios, en primer lugar para la autenticación y, si el inicio de sesión único está configurado, la aplicación de back-end también comprobará el usuario antes de que se conceda acceso a la aplicación. Cambiar el modo de autenticación previa de Acceso directo a Azure AD también configura la dirección URL externa con HTTPS, por lo que cualquier aplicación configurada inicialmente para HTTP ahora estará protegido con HTTPS.

### <a name="enable-single-sign-on"></a>Habilitar el inicio de sesión único

SSO proporciona la mejor experiencia y seguridad del usuario posibles porque los usuarios solo necesitan iniciar sesión una vez al acceder a Azure AD. Una vez que un usuario se ha autenticado previamente, el conector de Application Proxy realiza el SSO mediante la autenticación en la aplicación local en nombre del usuario. La aplicación de back-end procesa el inicio de sesión como si fuera el propio usuario.

Elegir la opción **Acceso directo** permite a los usuarios acceder a la aplicación publicada sin tener que autenticarse en Azure AD.

Solo se puede realizar el SSO si Azure AD puede identificar al usuario que solicita acceso a un recurso, por lo que la aplicación debe configurarse para la autenticación previa de los usuarios con Azure AD al acceder al SSO. En caso contrario, las opciones de SSO estarán desactivadas.

Lea [Inicio de sesión único en aplicaciones de Azure Active Directory](what-is-single-sign-on.md) para que le ayude a elegir el método de SSO más apropiado al configurar las aplicaciones.

###  <a name="working-with-other-types-of-applications"></a>Trabajar con otros tipos de aplicaciones

Azure AD Application Proxy también puede admitir aplicaciones que se han desarrollado para usar nuestra biblioteca de autenticación de Microsoft ([MSAL](https://docs.microsoft.com/azure/active-directory/develop/v2-overview)). Admite aplicaciones cliente nativas, ya que consume los tokens emitidos recibidos en la información de encabezado de la solicitud del cliente para realizar la autenticación previa en nombre de los usuarios.

Lea [Publicar aplicaciones cliente nativas y móviles](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-native-client) y [Aplicaciones basadas en notificaciones](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-claims-aware-apps) para obtener información sobre las configuraciones disponibles de Application Proxy.

### <a name="use-conditional-access-to-strengthen-security"></a>Use el acceso condicional para reforzar la seguridad

La seguridad de la aplicación requiere un conjunto avanzado de funcionalidades de seguridad que pueden protegerle frente a complejas amenazas locales y en la nube y responder cuando se producen. A menudo, los atacantes acceden a la red corporativa mediante credenciales de usuario no seguras, predeterminadas o robadas.  La seguridad basada en la identidad de Microsoft reduce el uso de credenciales robadas, ya que administra y protege tanto las identidades con privilegios como sin privilegios.

Las siguientes funcionalidades pueden usarse para admitir Azure AD Application Proxy:

* Acceso condicional basado en el usuario y en la ubicación: mantenga protegidos los datos confidenciales limitando el acceso de los usuarios según la geolocalización o una dirección IP con las [directivas de acceso condicional basado en la ubicación](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-locations).

* Acceso condicional basado en dispositivos: asegúrese de que solo los dispositivos inscritos, aprobados y compatibles pueden acceder a datos corporativos con [acceso condicional basado en el dispositivo](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-policy-connected-applications).

* Acceso condicional basado en la aplicación: el trabajo no se tiene que detener cuando un usuario no está en la red corporativa. [Proteja el acceso a aplicaciones locales y en la nube corporativas](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-mam) y mantenga el control con el acceso condicional.

* Acceso condicional basado en el riesgo: Proteja sus datos de hackers malintencionados con una [directiva de acceso condicional basada en el riesgo](https://www.microsoft.com/cloud-platform/conditional-access).

* Aplicaciones de Azure AD: Con el servicio Application Proxy implementado y las aplicaciones publicadas de forma segura, ofrezca a sus usuarios un centro sencillo para detectar y acceder a todas sus aplicaciones. Aumente la productividad con funcionalidades de autoservicio, como la capacidad para solicitar acceso a nuevas aplicaciones y grupos o para administrar el acceso a estos recursos en nombre de otros mediante [Aplicaciones](https://aka.ms/AccessPanelDPDownload).

## <a name="manage-your-implementation"></a>Administrar la implementación

### <a name="required-roles"></a>Roles necesarios

Microsoft defiende el principio de otorgar el mínimo privilegio posible para realizar las tareas necesarias con Azure AD. [Revise los distintos roles de Azure disponibles](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal) y elija el correcto para las necesidades de cada rol. Es posible que algunos roles se tengan que aplicar temporalmente y quitar una vez completada la implementación.

| Rol de negocio| Tareas empresariales| Roles de Azure AD |
|---|---|---|
| Administrador del departamento de soporte técnico | Normalmente, limitado a calificar los problemas que notifica el usuario final y realizar tareas limitadas, como cambiar las contraseñas de los usuarios, invalidar los tokens de actualización y supervisar el estado del servicio. | Administrador del departamento de soporte técnico |
| Administración de identidades| Lea los informes de inicio de sesión de Azure AD y los registros de auditoría para depurar problemas relacionados con App Proxy.| Lector de seguridad |
| Propietario de la aplicación| Cree y administre todos los aspectos de las aplicaciones empresariales, los registros de aplicaciones y la configuración del proxy de aplicación.| Administrador de aplicaciones |
| Administrador de la infraestructura | Propietario de sustitución del certificado | Administrador de aplicaciones |

Minimizar el número de personas que tienen acceso a información o recursos seguros ayuda a reducir la posibilidad de que personas malintencionadas obtengan acceso autorizado a ellos o de que algún usuario autorizado haga algo involuntariamente que pueda afectar a recursos confidenciales.

Sin embargo, los usuarios aún necesitan llevar a cabo operaciones con privilegios de un día para otro, por lo que aplicar directivas de [Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure) basadas en Just-In-Time para proporcionar acceso con privilegios a petición a los recursos de Azure y Azure AD es nuestro enfoque recomendado para administrar de forma eficaz el acceso administrativo y la auditoría.

### <a name="reporting-and-monitoring"></a>Creación de informes y supervisión

Azure AD proporciona información adicional acerca del uso de las aplicaciones de la organización y del estado operativo mediante [registros de auditoría e informes](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). Proxy de aplicación también facilita la supervisión de conectores desde el portal de Azure AD y los registros de eventos de Windows.

#### <a name="application-audit-logs"></a>Registros de auditoría de la aplicación

Estos registros proporcionan información detallada sobre los inicios de sesión a las aplicaciones configuradas con Application Proxy y el dispositivo y el usuario que acceden a la aplicación. [Los registros de auditoría](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) se pueden exportar desde Azure Portal y [Audit API](https://docs.microsoft.com/graph/api/resources/directoryaudit?view=graph-rest-beta). Además, los [informes de uso y de información](../reports-monitoring/concept-usage-insights-report.md?context=azure/active-directory/manage-apps/context/manage-apps-context) también están disponibles para la aplicación.

#### <a name="application-proxy-connector-monitoring"></a>Supervisión de conectores de Application Proxy

Los conectores y el servicio se encargan de todas las tareas de alta disponibilidad. Puede supervisar el estado de los conectores desde la página Proxy de aplicación en Azure AD Portal. Para más información sobre el mantenimiento de los conectores, consulte [Descripción de los conectores del Proxy de aplicación de Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-connectors#maintenance).

![Ejemplo: Conectores de Azure AD Application Proxy](./media/application-proxy-connectors/app-proxy-connectors.png)

#### <a name="windows-event-logs-and-performance-counters"></a>Contadores de rendimiento y registros de eventos de Windows

Los conectores tienen registros de administración y sesión. Los registros de administración incluyen eventos importantes y sus errores. Los registros de sesión incluyen todas las transacciones y sus detalles de procesamiento. Los registros y contadores se encuentran en los registros de eventos de Windows; para más información, consulte [Descripción de los conectores del Proxy de aplicación de Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-connectors#under-the-hood). Siga este [tutorial para configurar orígenes de datos de registro de eventos en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/data-sources-windows-events).

### <a name="troubleshooting-guide-and-steps"></a>Guía y pasos para la solución de problemas

Más información sobre los problemas comunes y cómo resolverlos con nuestra guía para los mensajes de error de [solución de problemas](application-proxy-troubleshoot.md).

Los siguientes artículos cubren escenarios comunes que también se pueden usar para crear guías de solución de problemas para su organización de soporte técnico.

* [Página de aplicación que muestra los problemas](application-proxy-page-appearance-broken-problem.md)
* [La carga de la aplicación es demasiado larga](application-proxy-page-load-speed-problem.md)
* [Los vínculos en la página de aplicación no funcionan](application-proxy-page-links-broken-problem.md)
* [¿Qué puertos se deben abrir para mi aplicación?](application-proxy-add-on-premises-application.md)
* [No hay ningún conector funcionando en un grupo de conectores para mi aplicación](application-proxy-connectivity-no-working-connector.md)
* [Configuración en el portal de administración](application-proxy-config-how-to.md)
* [Configuración del inicio de sesión único en mi aplicación](application-proxy-config-sso-how-to.md)
* [Problema al crear una aplicación en el portal de administración](application-proxy-config-problem.md)
* [Configuración de la delegación limitada de Kerberos](application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
* [Configuración con PingAccess](application-proxy-back-end-ping-access-how-to.md)
* [Error de imposibilidad de acceder a la aplicación corporativa](application-proxy-sign-in-bad-gateway-timeout-error.md)
* [Problema al instalar el conector de agente del proxy de aplicación](application-proxy-connector-installation-problem.md)
* [Problema de inicio de sesión](application-sign-in-problem-on-premises-application-proxy.md)
