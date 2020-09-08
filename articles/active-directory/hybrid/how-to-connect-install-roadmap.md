---
title: Guía de instalación de Azure AD Connect y Azure AD Connect Health | Microsoft Docs
description: Este documento proporciona información general de las opciones de instalación y las rutas de acceso disponibles para la instalación de Azure AD Connect y Connect Health.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/18/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: dd3361657406f2ece6a48b74d3ca6ee3d9ed68ca
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277364"
---
# <a name="azure-ad-connect-and-azure-ad-connect-health-installation-roadmap"></a>Guía de instalación de Azure AD Connect y Azure AD Connect Health

## <a name="install-azure-ad-connect"></a>Instalación de Azure AD Connect

> [!IMPORTANT]
> Microsoft no admite la modificación ni el funcionamiento de la sincronización de Azure AD Connect con acciones distintas a estas que se documentan formalmente. Cualquiera de estas acciones pueden provocar un estado incoherente o incompatible de sincronización de Azure AD Connect. Como resultado, Microsoft no ofrece soporte técnico para estas implementaciones.

Puede encontrar la descarga de Azure AD Connect en el [Centro de descarga de Microsoft](https://go.microsoft.com/fwlink/?LinkId=615771).

| Solución | Escenario |
| --- | --- |
| Antes de empezar: [Requisitos previos de Azure AD Connect](how-to-connect-install-prerequisites.md) |<li>Pasos que hay que completar antes de iniciar la instalación de Azure AD Connect.</li> |
| [Configuración rápida](how-to-connect-install-express.md) |<li>Si tiene un único bosque de AD se recomienda que use esta opción.</li> <li>Inicio de sesión de usuario con la misma contraseña mediante la sincronización de contraseñas.</li> |
| [Configuración personalizada](how-to-connect-install-custom.md) |<li>Se usa cuando se tienen varios bosques. Admite muchas [topologías](plan-connect-topologies.md) locales.</li> <li>Personalice la opción de inicio de sesión, como la autenticación de paso, ADFS para la federación o un proveedor de identidades de terceros.</li> <li>Personalice las características de sincronización, como el filtrado y la escritura diferida.</li> |
| [Actualización desde DirSync](how-to-dirsync-upgrade-get-started.md) |<li>Se usa cuando se tiene un servidor de DirSync existente que ya se está ejecutando.</li> |
| [Actualización desde Azure AD Sync o Azure AD Connect](how-to-upgrade-previous-version.md) |<li>Hay varios métodos diferentes entre los que puede elegir en función de sus preferencias.</li> |

[Después de la instalación](how-to-connect-post-installation.md) , debe comprobar que funciona según lo previsto y asignar licencias a los usuarios.

### <a name="next-steps-to-install-azure-ad-connect"></a>Pasos siguientes para instalar Azure AD Connect
|Tema |Vínculo|  
| --- | --- |
|Descarga de Azure AD Connect | [Descarga de Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771)|
|Instalación mediante configuración rápida | [Instalación rápida de Azure AD Connect](./how-to-connect-install-express.md)|
|Instalación mediante configuración personalizada | [Instalación personalizada de Azure AD Connect](./how-to-connect-install-custom.md)|
|Actualización desde DirSync | [Actualización desde la herramienta de sincronización de Azure AD (DirSync)](./how-to-dirsync-upgrade-get-started.md)|
|Después de la instalación | [Comprobación de la instalación y asignación de licencias ](how-to-connect-post-installation.md)|

### <a name="learn-more-about-install-azure-ad-connect"></a>Más información sobre la instalación de Azure AD Connect
También quiere prepararse para los problemas [operativos](./how-to-connect-sync-staging-server.md) . Puede que desee tener un servidor en espera que le permita conmutar por error fácilmente en caso de [desastre](how-to-connect-sync-staging-server.md#disaster-recovery). Si planea realizar cambios frecuentes en la configuración, debe preparar un servidor en [modo provisional](how-to-connect-sync-staging-server.md) .

|Tema |Vínculo|  
| --- | --- |
|Topologías admitidas | [Topologías de Azure AD Connect](plan-connect-topologies.md)|
|Conceptos de diseño | [Conceptos de diseño de Azure AD Connect](plan-connect-design-concepts.md)|
|Cuentas usadas para la instalación | [Más información sobre las credenciales y los permisos de Azure AD Connect](reference-connect-accounts-permissions.md)|
|Planeación operativa | [Sincronización de Azure AD Connect: tareas y consideraciones operativas](./how-to-connect-sync-staging-server.md)|
|Opciones de inicio de sesión de usuario | [Opciones para el inicio de sesión de los usuarios en Azure AD Connect](plan-connect-user-signin.md)|

## <a name="configure-sync-features"></a>Configuración de características de sincronización
Azure AD Connect incluye varias características que puede activar de manera opcional o que están habilitadas de forma predeterminada. En algunas ocasiones, ciertas características pueden requerir configuración adicional en determinados escenarios y topologías.

[filtrado](how-to-connect-sync-configure-filtering.md) se usa cuando se quieren limitar los objetos que se sincronizan con Azure AD. De forma predeterminada todos los usuarios, contactos, grupos y equipos de Windows 10 están sincronizados. Puede cambiar el filtrado basado en dominios, unidades organizativas o atributos.

[Sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md) sincroniza el hash de contraseña de Active Directory con Azure AD. El usuario final puede usar la misma contraseña de forma local y en la nube, pero solo puede administrarla en una ubicación. Como se usa su Active Directory local como autoridad, también podrá usar su propia directiva de contraseñas.

[escritura diferida de contraseñas](../authentication/tutorial-enable-sspr.md) permite a los usuarios cambiar y restablecer sus contraseñas en la nube y aplicar su directiva local de contraseñas.

[Escritura diferida de dispositivo](how-to-connect-device-writeback.md) permite que un dispositivo registrado en Azure AD se reescriba en Active Directory local para que se pueda usar para el acceso condicional.

La característica para [evitar eliminaciones accidentales](how-to-connect-sync-feature-prevent-accidental-deletes.md) está activada de forma predeterminada y protege su directorio en la nube de muchas eliminaciones al mismo tiempo. De forma predeterminada permite 500 eliminaciones por cada ejecución. Puede cambiar esta configuración según el tamaño de la organización.

[actualización automática](how-to-connect-install-automatic-upgrade.md) está habilitada de forma predeterminada para las instalaciones de configuración rápida y garantiza que Azure AD Connect está siempre actualizado con la versión más reciente.

### <a name="next-steps-to-configure-sync-features"></a>Pasos siguientes para configurar características de sincronización
|Tema |Vínculo|  
| --- | --- |
|Configuración del filtrado | [Sincronización de Azure AD Connect: configuración del filtrado](how-to-connect-sync-configure-filtering.md)|
|Sincronización de hash de contraseña | [Sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md)|
|Autenticación de paso a través | [Autenticación de paso a través](how-to-connect-pta.md)
|escritura diferida de contraseñas | [Introducción a la administración de contraseñas](../authentication/tutorial-enable-sspr.md)|
|Escritura diferida de dispositivos | [Habilitación de escritura diferida de dispositivos en Azure AD Connect](how-to-connect-device-writeback.md)|
|evitar eliminaciones accidentales | [Sincronización de Azure AD Connect: evitar eliminaciones involuntarias](how-to-connect-sync-feature-prevent-accidental-deletes.md)|
|Actualización automática | [Azure AD Connect: actualización automática](how-to-connect-install-automatic-upgrade.md)|

## <a name="customize-azure-ad-connect-sync"></a>Personalización de Azure AD Connect Sync
Azure AD Connect Sync incluye una configuración predeterminada diseñada para funcionar en la mayoría de clientes y topologías. Pero siempre hay situaciones en las que la configuración predeterminada no funciona y debe ajustarse. Se permiten los cambios que se documentan en esta sección y en los temas vinculados.

Si no ha trabajado con una topología de sincronización antes, querrá comenzar a comprender los aspectos básicos y los términos usados que se describen en los [conceptos técnicos](how-to-connect-sync-technical-concepts.md). Azure AD Connect es la evolución de MIIS2003, ILM2007 y FIM2010. Aunque algunas cosas son idénticas, muchas también han cambiado.

En la [configuración predeterminada](concept-azure-ad-connect-sync-default-configuration.md) se considera que puede haber más de un bosque. En esas topologías un objeto de usuario se puede representar como un contacto de otro bosque. El usuario puede tener también vinculado un buzón en otro bosque de recursos. El comportamiento de la configuración predeterminada se describe en la sección sobre [usuarios y contactos](concept-azure-ad-connect-sync-user-and-contacts.md).

El modelo de configuración sincronizado se denomina [aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md). Los flujos de atributos avanzados usan [funciones](reference-connect-sync-functions-reference.md) para expresar transformaciones de atributo. Puede ver y examinar toda la configuración con las herramientas que se incluyen en Azure AD Connect. Si tiene que realizar cambios en la configuración, asegúrese de seguir los [procedimientos recomendados](how-to-connect-sync-best-practices-changing-default-configuration.md) para que sea más fácil adoptar nuevas versiones.

### <a name="next-steps-to-customize-azure-ad-connect-sync"></a>Pasos siguientes para personalizar Azure AD Connect Sync
|Tema |Vínculo|  
| --- | --- |
|Todos los artículos de sincronización de Azure AD Connect | [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md)|
|conceptos técnicos | [Sincronización de Azure AD Connect: conceptos técnicos](how-to-connect-sync-technical-concepts.md)|
|Introducción a la configuración predeterminada | [Sincronización de Azure AD Connect: descripción de la configuración predeterminada](concept-azure-ad-connect-sync-default-configuration.md)|
|Descripción de usuarios y contactos | [Sincronización de Azure AD Connect: descripción de usuarios y contactos](concept-azure-ad-connect-sync-user-and-contacts.md)|
|aprovisionamiento declarativo | [Sincronización de Azure AD Connect: descripción de las expresiones de aprovisionamiento declarativas](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)|
|Cambio de la configuración predeterminada | [Prácticas recomendadas de cambio de la configuración predeterminada](how-to-connect-sync-best-practices-changing-default-configuration.md)|

## <a name="configure-federation-features"></a>Configuración de características de federación

Azure AD Connect proporciona varias características que simplifican la federación con Azure AD mediante AD FS y la administración de la confianza de federación. Azure AD Connect es compatible con AD FS en Windows Server 2012R2 o posterior.

[Actualice el certificado TLS/SSL de la granja de AD FS](how-to-connect-fed-ssl-update.md) incluso si no usa Azure AD Connect para administrar la confianza de federación.

[Agregue un servidor de AD FS](how-to-connect-fed-management.md#addadfsserver) a la granja para expandirla según sea necesario.

[Repare la confianza](how-to-connect-fed-management.md#repairthetrust) con Azure AD en unos pocos pasos sencillos.

Se puede configurar AD FS para que admita [varios dominios](how-to-connect-install-multiple-domains.md). Por ejemplo, puede tener varios dominios principales que necesita utilizar para la federación.

Si no se configuró el servidor de AD FS para actualizar automáticamente los certificados de Azure AD o si utiliza una solución que no es AD FS, se le notificará cuando tenga que [actualizar los certificados](how-to-connect-fed-o365-certs.md).

### <a name="next-steps-to-configure-federation-features"></a>Pasos siguientes para configurar características de federación
|Tema |Vínculo|  
| --- | --- |
|Todos los artículos de AD FS | [Azure AD Connect y la federación](how-to-connect-fed-whatis.md)|
|Configuración de AD FS con subdominios | [Compatibilidad con varios dominios para la federación con Azure AD](how-to-connect-install-multiple-domains.md)|
|Administrar una granja de servidores de AD FS | [Administración y personalización de AD FS con Azure AD Connect](how-to-connect-fed-management.md)|
|Actualización manual de certificados de federación | [Renovación de certificados de federación para Office 365 y Azure AD](how-to-connect-fed-o365-certs.md)|


## <a name="get-started-with-azure-ad-connect-health"></a>Introducción a Azure AD Connect Health
Para empezar a usar Azure AD Connect Health, siga estos pasos:

1. [Obtenga Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) o [inicie una prueba](https://azure.microsoft.com/trial/get-started-active-directory/).
2. [Descargue e instale los agentes de Azure AD Connect Health](#download-and-install-azure-ad-connect-health-agent) en los servidores de identidad.
3. Puede ver el panel de Azure AD Connect Health en [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth).

> [!NOTE]
> Recuerde que para poder ver los datos de panel de Azure AD Connect Health, es preciso instalar los agentes de Azure AD Connect Health en los servidores de destino.
>
>

## <a name="download-and-install-azure-ad-connect-health-agent"></a>Descarga e instalación de un agente de Azure AD Connect Health
* Asegúrese de que [cumple los requisitos](how-to-connect-health-agent-install.md#requirements) de Azure AD Connect Health.
* Introducción a Azure AD Connect Health para AD FS
    * [Descargue el agente de Azure AD Connect Health para AD FS.](https://go.microsoft.com/fwlink/?LinkID=518973)
    * [Consulte las instrucciones de instalación](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs).
* Introducción al uso de Azure AD Connect Health para la sincronización
    * [Descargue e instale la versión más reciente de Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771). El agente de Health para la sincronización se instalará como parte de la instalación de Azure AD Connect (versión 1.0.9125.0 o superior).
* Introducción a Azure AD Connect Health para AD DS
    * [Descargue el agente de Azure AD Connect Health para AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * [Consulte las instrucciones de instalación](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-ds).


## <a name="azure-ad-connect-health-portal"></a>Portal de Azure AD Connect Health
El portal de Azure AD Connect Health muestra las vistas de alertas, la supervisión del rendimiento y el análisis de uso. La dirección URL https://aka.ms/aadconnecthealth le lleva a la hoja principal de Azure AD Connect Health. Puede considerar una hoja como una ventana. En la hoja principal, verá **Inicio rápido**, los servicios de Azure AD Connect Health y otras opciones de configuración. Consulte la siguiente captura de pantalla y las explicaciones que se proporciona después de la captura de pantalla. Después de implementar los agentes, el servicio de mantenimiento identifica automáticamente los servicios que Azure AD Connect Health supervisa.

> [!NOTE]
> Para más información sobre licencias, consulte [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md) o la [página de precios de Azure AD](https://aka.ms/aadpricing).
    
![portal de Azure AD Connect Health](./media/whatis-hybrid-identity-health/portalsidebar.png)

* **Inicio rápido**: si se selecciona esta opción, se abre la hoja **Inicio rápido**. Para descargar el agente de Azure AD Connect Health, seleccione **Obtener herramientas**. También puede acceder a la documentación y aportar comentarios.
* **Azure Active Directory Connect (sincronización)** : esta opción muestra los servidores de Azure AD Connect que Azure AD Connect Health supervisa actualmente. La entrada **Errores de sincronización** mostrará los errores de sincronización básicos de su primer servicio de sincronización incorporado por categorías. Al seleccionar la entrada **Servicios de sincronización**, la hoja que se abre muestra información acerca de los servidores de Azure AD Connect. Para más información sobre las funcionalidades, consulte [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md).
* **Servicios de federación de Active Directory (AD FS)** : esta opción muestra todos los servicios de AD FS que Azure AD Connect Health supervisa actualmente. Cuando se selecciona una instancia, la hoja que se abre muestra información acerca de ella. Esta información incluye una descripción general, propiedades, alertas, supervisión y análisis de uso. Para más información sobre las funcionalidades, consulte [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-adfs.md).
* **Active Directory Domain Services**: esta opción muestra todos los bosques de AD DS que Azure AD Connect Health supervisa actualmente. Cuando se selecciona un bosque, la hoja que se abre muestra información acerca de él. Esta información incluye una introducción a la información esencial, el panel de controladores de dominio, el panel de estado de replicación, alertas y supervisión. Para más información sobre las funcionalidades, consulte [Uso de Azure AD Connect Health con AD DS](how-to-connect-health-adds.md).
* **Configurar**: esta sección incluye opciones para activar o desactivar lo siguiente:

   - La **actualización automática** del agente de Azure AD Connect Health a la versión más reciente: el agente de Azure AD Connect Health se actualiza automáticamente cuando hay nuevas versiones disponibles. Esta opción está habilitada de manera predeterminada.
   - El **acceso a los datos** desde la integridad de directorio de Azure AD por parte de Microsoft únicamente para solucionar problemas: si esta opción está habilitada, Microsoft puede acceder a los mismos datos que ve el usuario. Esta información puede ser útil para solucionar problemas y proporcionar la ayuda necesaria. Esta opción está deshabilitada de manera predeterminada.
* **Control de acceso basado en roles (IAM)** es la sección para administrar el acceso a los datos de Connect Health en base a roles. 

## <a name="next-steps"></a>Pasos siguientes

- [Hardware y requisitos previos](how-to-connect-install-prerequisites.md) 
- [Configuración rápida](how-to-connect-install-express.md)
- [Configuración personalizada](how-to-connect-install-custom.md)
- [Sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md)|
- [Autenticación de paso a través](how-to-connect-pta.md)
- [Azure AD Connect y la federación](how-to-connect-fed-whatis.md)
- [Instalación de agentes de Azure AD Connect Health](how-to-connect-health-agent-install.md) 
- [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md)