---
title: Planeamiento de la implementación de Aplicaciones de Azure Active Directory
description: Guía de implementación de Aplicaciones de Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 09/27/2019
ms.author: kenwith
ms.openlocfilehash: b7ddbff2643086f1875ca190b67f521edb115c3e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930543"
---
# <a name="plan-an-azure-active-directory-my-apps-deployment"></a>Planeamiento de la implementación de Aplicaciones de Azure Active Directory

Aplicaciones de Azure Active Directory (Azure AD) es un portal basado en web que le permite reducir los costos de soporte técnico, aumentar la productividad y la seguridad, y reducir la frustración de los usuarios. El sistema incluye informes detallados de seguimiento de su acceso al sistema y envía notificaciones de alerta a los administradores ante posibles abusos o usos indebidos.

Con Aplicaciones de Azure AD, puede:

* Detectar y acceder a todos los recursos de su empresa conectados a Azure AD como, por ejemplo, a las aplicaciones
* Solicitar acceso a las nuevas aplicaciones y grupos
* Administrar el acceso a estos recursos para otros usuarios
* Administrar el autoservicio de restablecimiento de contraseña y la configuración de Azure Multi-Factor Authentication
* Administrar sus dispositivos

También permite a los administradores administrar:

* Los términos del servicio
* Las organizaciones
* Revisiones de acceso


## <a name="benefits-of-azure-ad-my-apps-integration"></a>Ventajas de la integración de Aplicaciones de Azure AD

Aplicaciones de Azure AD beneficia a las empresas de la forma siguiente:

**Proporciona una experiencia de usuario intuitiva**: Aplicaciones proporciona una plataforma única para todas las aplicaciones conectadas al inicio de sesión único (SSO) de Azure. Tiene un portal unificado para encontrar la configuración actual y nuevas funcionalidades, similar a la administración de grupos y al autoservicio de restablecimiento de contraseña, a medida que se agregan. La experiencia intuitiva permite a los usuarios trabajar más rápido y ser más productivos, a la vez que se reduce su frustración.

**Aumenta la productividad**: Todas las aplicaciones de usuario de Aplicaciones tienen habilitado el inicio de sesión único. Habilitar el inicio de sesión único en las aplicaciones empresariales y en Microsoft 365 ofrece una experiencia de inicio de sesión superior con la reducción o eliminación de los mensajes de inicio de sesión adicionales. Aplicaciones usa la pertenencia dinámica y de autoservicio y mejora la seguridad global del sistema de identidades. Para ello, garantiza que las personas adecuadas administran el acceso a las aplicaciones. Aplicaciones sirve como una página de aterrizaje coherente para encontrar rápidamente los recursos y continuar con las tareas del trabajo.

**Administra los costos**: la habilitación de Aplicaciones con Azure AD puede ayudar a retirar las inversiones en infraestructuras locales. Reduce los costos de soporte técnico al proporcionar un portal coherente para encontrar todas las aplicaciones, solicitar acceso a los recursos y administrar las cuentas.

**Aumenta la flexibilidad y la seguridad**: Aplicaciones le permite acceder a la seguridad y la flexibilidad que proporciona una plataforma en la nube. Los administradores pueden cambiar fácilmente la configuración de las aplicaciones y los recursos, y adaptarse a los nuevos requisitos de seguridad sin que ello afecte a los usuarios.

**Permite una eficaz auditoría y seguimiento del uso**: la auditoría y el seguimiento de uso de todas las funcionalidades del usuario final le permiten saber cuándo los usuarios usan sus recursos y le garantizan la evaluación de la seguridad.

### <a name="licensing-considerations"></a>Consideraciones acerca de las licencias

Aplicaciones es gratuito y no requiere ninguna licencia para usarse en un nivel básico. Sin embargo, es posible que el número de objetos en el directorio y las características adicionales que desee implementar requieran licencias adicionales. Entre los escenarios de Azure AD más habituales que tienen requisitos de licencia se incluyen las siguientes características de seguridad:

* [Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-how-it-works)

* [Pertenencia basada en grupos](https://docs.microsoft.com/azure/active-directory/active-directory-manage-groups)

* [Restablecimiento de la contraseña de autoservicio](https://docs.microsoft.com/azure/active-directory/authentication/quickstart-sspr)

* [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)

Consulte la [guía de licencias completa para Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="prerequisites-for-deploying-azure-ad-my-apps"></a>Requisitos previos para la implementación de Aplicaciones de Azure AD

Cumpla los siguientes requisitos previos antes de empezar este proyecto:

* [Integre el inicio de sesión único de las aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/plan-sso-deployment)

* [Administre la infraestructura de usuarios y grupos de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-manage-groups)

## <a name="plan-azure-ad-my-apps-deployment"></a>Planeamiento de la implementación de Aplicaciones de Azure AD

En la tabla siguiente se describen los casos de uso principales de una implementación de Aplicaciones:

| Área| Descripción |
| - | - |
| Acceso| El portal de Aplicaciones es accesible desde dispositivos corporativos y personales dentro de la red corporativa. |
|Acceso | El portal de Aplicaciones es accesible desde dispositivos corporativos fuera de la red corporativa. |
| Auditoría| Los datos de uso se descargan en sistemas corporativos al menos cada 29 días. |
| Gobernanza| El ciclo de vida de las asignaciones de usuarios a aplicaciones y grupos conectados de Azure AD se define y se supervisa. |
| Seguridad| El acceso a los recursos se controla a través de asignaciones de usuarios y grupos. Solo los usuarios autorizados pueden administrar el acceso a los recursos. |
| Rendimiento| Las escalas de tiempo de propagación de asignación de acceso se documentan y supervisan. |
| Experiencia del usuario| Los usuarios son conscientes de las funcionalidades de Aplicaciones y de cómo usarlas.|
| Experiencia del usuario| Los usuarios pueden administrar el acceso a las aplicaciones y los grupos.|
| Experiencia del usuario| Los usuarios pueden administrar sus cuentas. |
| Experiencia del usuario| Los usuarios son conscientes de la compatibilidad con el explorador. |
| Soporte técnico| Los usuarios pueden encontrar soporte técnico para los problemas de Aplicaciones. |

### <a name="best-practices-for-deploying-azure-ad-my-apps"></a>Procedimientos recomendados para la implementación de Aplicaciones de Azure AD

La funcionalidad de Aplicaciones se puede habilitar gradualmente. Se recomienda el siguiente orden de implementación:

1. Mis aplicaciones
   * Iniciador de aplicaciones
   * Autoservicio de administración de aplicaciones
   * Integración de Microsoft 365

1. Autoservicio de detección de aplicaciones
   * Restablecimiento de la contraseña de autoservicio
   * Configuración de Multi-Factor Authentication
   * Administración de dispositivos
   * Términos de uso
   * Administración de organizaciones

1. Mis grupos
   * Self-service group management (Administración de grupos de autoservicio)
1. Revisiones de acceso
   * Administración de la revisión de acceso

Empezando en Aplicaciones, se presenta a los usuarios el portal como lugar común para el acceso a los recursos. La adición del autoservicio de detección de aplicaciones se crea a partir de la experiencia de Aplicaciones. Mis grupos y las revisiones de acceso se crean en las funcionalidades de autoservicio.

### <a name="plan-configurations-for-azure-my-apps"></a>Planeamiento de la configuración de Aplicaciones de Azure

En la tabla siguiente se enumeran varias configuraciones importantes de Aplicaciones y los valores típicos que se pueden usar:

| Configuración| Valores típicos |
| - | - |
| Determinar los grupos piloto| Identificar el grupo de seguridad de Azure AD que se va a utilizar y asegurarse de que todos los miembros piloto forman parte del grupo |
| Determinar los grupos que se van a habilitar para producción| Identificar los grupos de seguridad de Azure AD o los grupos de Active Directory sincronizados con Azure AD, que se van a usar. Asegúrese de que todos los miembros piloto forman parte del grupo. |
| Permitir a los usuarios usar el inicio de sesión único para determinados tipos de aplicaciones| Inicio de sesión único federado, OAuth, inicio de sesión único con contraseña, proxy de aplicación |
| Permitir a los usuarios el uso del autoservicio de restablecimiento de contraseña | Sí |
| Permitir a los usuarios usar Multi-Factor Authentication| Sí |
| Permitir a los usuarios usar la administración de grupos de autoservicio para ciertos tipos de grupos| Grupos de seguridad y grupos de Microsoft 365 |
| Permitir a los usuarios usar la administración de aplicaciones de autoservicio| Sí |
| Permitir a los usuarios usar revisiones de acceso| Sí |

### <a name="plan-consent-strategy"></a>Planeación de la estrategia de consentimiento

Los usuarios o administradores deben dar su consentimiento a los términos de uso y a las directivas de privacidad de todas las aplicaciones. Si sus reglas de negocio lo permiten, use el consentimiento del administrador, ya que ofrece a los usuarios una mejor experiencia.

Para usar el consentimiento del administrador, debe ser un administrador global de la organización y las aplicaciones deben:

* Registrarse en su organización

* Registrarse en otro inquilino de Azure AD y haber recibido previamente el consentimiento, como mínimo, de un usuario

Para más información, consulte [Configuración del modo en que los usuarios finales dan su consentimiento a una aplicación en Azure Active Directory](configure-user-consent.md).

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que involucra a las partes interesadas adecuadas](../fundamentals/active-directory-deployment-plans.md) y que estas conocen bien sus roles.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Informe de forma proactiva a los usuarios sobre cómo y cuándo cambiará su experiencia y cómo obtener soporte técnico si es necesario.

Aunque Aplicaciones no suele dar problemas al usuario, es importante estar preparado. Cree guías y una lista de todos los recursos para el personal de soporte técnico antes de su lanzamiento.

#### <a name="communications-templates"></a>Plantillas de comunicaciones

Microsoft proporciona [plantillas personalizables para correos electrónicos y otras comunicaciones](https://aka.ms/APTemplates) de Aplicaciones. Puede adaptar estos recursos para su uso en otros canales de comunicación, como resulte adecuado para su cultura corporativa.

## <a name="plan-your-sso-configuration"></a>Planeación de la configuración del inicio de sesión único

Cuando un usuario inicia sesión en una aplicación, pasa por un proceso de autenticación en el que debe demostrar que es quien dice ser. Sin el inicio de sesión único, se almacena una contraseña en la aplicación y el usuario debe conocer esta contraseña. Con el inicio de sesión único, las credenciales de usuario se pasan a la aplicación por lo que no es necesario que vuelvan a escribirlas para cada aplicación.

Para iniciar aplicaciones en Aplicaciones, el inicio de sesión único debe estar habilitado. Azure AD admite varias opciones de inicio de sesión único. Para más información al respecto, consulte [Opciones de inicio de sesión único en Azure AD](sso-options.md).

> [!NOTE]
> Para obtener más información sobre cómo usar Azure AD como proveedor de identidades de una aplicación, consulte la [serie de inicios rápidos sobre la administración de aplicaciones](view-applications-portal.md).

Para disfrutar de la mejor experiencia con la página Aplicaciones, empiece por la integración de las aplicaciones en la nube disponibles para el inicio de sesión único federado. El inicio de sesión único federado permite a los usuarios tener una experiencia coherente de un solo clic en las superficies de inicio de la aplicación y tiende a ser más eficaz en el control de la configuración.

Use el inicio de sesión único federado con Azure AD (OpenID Connect o SAML) cuando una aplicación lo admita, en lugar del inicio de sesión único basado en contraseña y ADFS.

Para más información sobre cómo implementar y configurar aplicaciones SaaS, consulte el [plan de implementación del inicio de sesión único de SaaS](https://aka.ms/deploymentplans/sso).

#### <a name="plan-to-deploy-the-my-apps-browser-extension"></a>Planeación de la implementación de la extensión del explorador de Aplicaciones

Cuando los usuarios inician sesión en las aplicaciones de inicio de sesión único basadas en contraseña, deben instalar y usar la extensión de inicio de sesión seguro en Aplicaciones. La extensión ejecuta un script que transmite la contraseña en el formulario de inicio de sesión de la aplicación. Se solicita a los usuarios que instalen la extensión la primera vez que inicien la aplicación de inicio de sesión único basado en contraseñas. Puede encontrar más información sobre la extensión en nuestra documentación sobre la [instalación de la extensión del explorador de Aplicaciones](access-panel-extension-problem-installing.md).

Si debe integrar aplicaciones de inicio de sesión único basado en contraseñas, debería definir un mecanismo para implementar la extensión a escala con [exploradores compatibles](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction). Las opciones incluyen:

* [Directiva de grupo para Internet Explorer](https://azure.microsoft.com/documentation/articles/active-directory-saas-ie-group-policy/)

* [Configuration Manager para Internet Explorer](https://docs.microsoft.com/configmgr/core/clients/deploy/deploy-clients-to-windows-computers)

* [Descarga y configuración de Chrome, Firefox, Microsoft Edge o IE controladas por el usuario](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)

Los usuarios que no usan aplicaciones de inicio de sesión único basado en contraseñas también se benefician de la extensión. Entre las ventajas se incluyen la posibilidad de iniciar cualquier aplicación desde su barra de búsqueda, encontrar acceso a las aplicaciones usadas recientemente y tener un vínculo a la página Aplicaciones.

#### <a name="plan-for-mobile-access"></a>Planeación del acceso móvil

Se necesita un explorador protegido con una directiva de Intune (Microsoft Edge o Intune Managed Browser) para que los usuarios móviles inicien aplicaciones de inicio de sesión único basado en contraseñas. Un explorador protegido por directivas permite la transferencia de la contraseña guardada para la aplicación. Microsoft Edge o Managed Browser proporcionan un conjunto de características de protección de datos web. También puede usar Microsoft Edge para escenarios empresariales en dispositivos iOS y Android. Microsoft Edge admite los mismos escenarios de administración que Intune Managed Browser y mejora la experiencia del usuario final. Más información: [Administración del acceso web mediante un explorador protegido por directivas de Microsoft Intune](https://docs.microsoft.com/intune/app-configuration-managed-browser).

## <a name="plan-your-my-apps-deployment"></a>Planeación de la implementación de la página Aplicaciones

La base de Aplicaciones es el portal del iniciador de aplicaciones, al que los usuarios acceden en [https://myapps.microsoft.com](https://myapps.microsoft.com/). La página Aplicaciones proporciona a los usuarios un único lugar desde el que iniciar el trabajo y utilizar las aplicaciones necesarias. En ella, los usuarios encontrarán una lista de todas las aplicaciones a las que tienen acceso de inicio de sesión único. 

> [!NOTE]
> Las mismas aplicaciones se mostrarán en el iniciador de aplicaciones de Microsoft 365.

Planee el orden en el que va a agregar las aplicaciones al iniciador de la página Aplicaciones y decida si las va a distribuir gradualmente o todas a la vez. Para ello, cree un inventario de aplicaciones que muestre el tipo de autenticación y cualquier integración existente de inicio de sesión único para cada aplicación.

#### <a name="add-applications-to-the-my-apps-panel"></a>Adición de aplicaciones al panel Aplicaciones

Se puede agregar cualquier aplicación habilitada para el inicio de sesión único de Azure AD al panel Aplicaciones. Otras aplicaciones se agregan mediante la opción de inicio de sesión único vinculado. Puede configurar un icono de la aplicación que se vincule a la dirección URL de la aplicación web existente. El inicio de sesión único vinculado le permite empezar a dirigir a los usuarios al portal Aplicaciones sin migrar todas las aplicaciones al inicio de sesión único de Azure AD. Puede moverse gradualmente a aplicaciones configuradas con inicio de sesión único de Azure AD sin interrumpir la experiencia de los usuarios.

#### <a name="use-my-apps-collections"></a>Empleo de colecciones de Aplicaciones

De forma predeterminada, todas las aplicaciones se muestran juntas en una sola página. Sin embargo, puede usar colecciones para agrupar aplicaciones relacionadas y presentarlas en una pestaña independiente, lo que facilita su búsqueda. Por ejemplo, puede usar colecciones para crear agrupaciones lógicas de aplicaciones para roles de trabajo específicos, tareas, proyectos, etc. Para obtener información, vea [Cómo usar colecciones de Aplicaciones](access-panel-collections.md). 

#### <a name="plan-whether-to-use-my-apps-or-an-existing-portal"></a>Decidir si utilizar Aplicaciones o un portal ya existente

Es posible que los usuarios ya dispongan de una aplicación o portal distinto de Aplicaciones. Si es así, decida si desea admitir ambos portales o si solo va a utilizar uno.

Si ya se está usando un portal existente como punto de partida para los usuarios, puede integrar la funcionalidad Aplicaciones mediante direcciones URL de acceso de usuarios. Los usuarios acceden a la función de direcciones URL como vínculos directos a las aplicaciones disponibles en el portal Aplicaciones. Estas direcciones URL se pueden insertar en cualquier sitio web existente. Cuando un usuario selecciona el vínculo, este inicia la aplicación desde el portal Aplicaciones.

Puede encontrar la propiedad URL de acceso de usuario en el área **Propiedades** de la aplicación en Azure Portal.

![Captura de pantalla del panel de aplicaciones](media/access-panel-deployment-plan/ap-dp-user-access-url.png)


## <a name="plan-self-service-application-discovery-and-access"></a>Planeación de la detección y el acceso a aplicaciones de autoservicio

Una vez que se implementa un conjunto básico de aplicaciones en la página Aplicaciones de un usuario, se recomienda habilitar las características de autoservicio para la administración de aplicaciones. El autoservicio de detección de aplicaciones permite a los usuarios:

* Buscar nuevas aplicaciones para agregarlas a Aplicaciones. 
* Agregar aplicaciones opcionales que no sabía que necesitaban durante la configuración.

Hay flujos de trabajo de aprobación disponibles para la aprobación explícita para acceder a las aplicaciones. Los usuarios que sean aprobadores recibirán notificaciones en el portal Aplicaciones cuando haya solicitudes pendientes de acceso a la aplicación.

## <a name="plan-self-service-group-membership"></a>Planeación de la pertenencia a grupos de autoservicio 

Puede permitir que los usuarios creen y administren sus propios grupos de seguridad o grupos de Microsoft 365 en Azure AD. El propietario del grupo puede aprobar o rechazar solicitudes de pertenencia y delegar el control de la pertenencia a grupos. Las características de administración de grupos de autoservicio no están disponibles para grupos de seguridad habilitados para correo electrónico o listas de distribución.

Para planear la pertenencia a grupos de autoservicio, determine si va a permitir que todos los usuarios de su organización creen y administren los grupos, o solo un subconjunto de usuarios. Si solo lo va a permitir a un subconjunto de usuarios, deberá configurar un grupo al que se agreguen esas personas. Consulte [Configuración de la administración de grupos de autoservicio en Azure Active Directory](../users-groups-roles/groups-self-service-management.md) para más información sobre cómo habilitar estos escenarios.

## <a name="plan-reporting-and-auditing"></a>Planeación de informes y auditoría

Azure AD proporciona [informes que ofrecen datos técnicos y empresariales](../reports-monitoring/overview-reports.md). Colabore con los propietarios de las aplicaciones empresariales y técnicas para que asuman la propiedad de los informes y los usen de forma periódica. En la tabla siguiente se proporcionan algunos ejemplos de escenarios de informes típicos.

| Ejemplo | Administración de los riesgos.| Aumento de la productividad| Gobernanza y cumplimiento |
|  - |- | - | - |
| Tipos de informes|  Permisos y uso de la aplicación| Actividad de aprovisionamiento de cuentas| Revise quién accede a las aplicaciones |
| Acciones posibles| Auditar el acceso; revocar los permisos| Corregir los errores de aprovisionamiento| Revocar el acceso |

Azure AD mantiene la mayoría de los datos de auditoría durante 30 días. Los datos están disponibles a través del portal de administración de Azure o de la API para que pueda descargarlos en los sistemas de análisis.

#### <a name="auditing"></a>Auditoría

Los registros de auditoría para el acceso a las aplicaciones estarán disponibles durante 30 días. Si la auditoría de seguridad de su empresa requiere una retención más prolongada, los registros deberán exportarse en una herramienta de Administración de eventos e información de seguridad (SIEM), como Splunk o ArcSight.

Para las copias de seguridad de auditoría, informes y recuperación ante desastres, documente la frecuencia necesaria de descarga, cuál es el sistema de destino y quién es el responsable de administrar cada copia de seguridad. Es posible que no necesite copias de seguridad independientes de auditoría e informes. La copia de seguridad de recuperación ante desastres debe ser una entidad independiente.

## <a name="deploy-applications-to-users-my-apps-panel"></a>Implementación de aplicaciones en el panel Aplicaciones de los usuarios

Después de configurar una aplicación para el inicio de sesión único, se asigna acceso a los grupos. Los usuarios de los grupos asignados tendrán acceso y verán la aplicación en su portal Aplicaciones y en el iniciador de aplicaciones de Microsoft 365.

Consulte [Asignación de usuarios y grupos en una aplicación de Active Directory](methods-for-assigning-users-and-groups.md).

Si durante las pruebas o la implementación quiere agregar los grupos, pero no permitir todavía que las aplicaciones aparezcan en Aplicaciones, consulte [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](hide-application-from-user-portal.md).

### <a name="deploy-microsoft-microsoft-365-applications-to-my-apps"></a>Implementación de aplicaciones de Microsoft 365 en Aplicaciones

En el caso de aplicaciones de Microsoft 365, los usuarios recibirán una copia de Office basada en las licencias que se les hayan asignado. Un requisito previo para el acceso a las aplicaciones de Office que se asigne a los usuarios las licencias correctas asociadas a las aplicaciones de Office. Al asignar a un usuario la licencia, verá automáticamente las aplicaciones asociadas a ella en la página Aplicaciones y en el iniciador de aplicaciones de Microsoft 365.

Si quiere ocultar un conjunto de aplicaciones de Office a los usuarios, hay una opción para ocultar aplicaciones en el portal Aplicaciones y, al mismo tiempo, permitir el acceso desde el portal de Microsoft 365. Más información: [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](hide-application-from-user-portal.md)

### <a name="deploy-application-self-service-capabilities"></a>Implementación de funcionalidades de autoservicio de aplicaciones

El acceso de autoservicio a las aplicaciones permite a los usuarios la detección de las aplicaciones y la solicitud del acceso. Los usuarios tienen libertad para acceder a las aplicaciones que necesitan sin tener que pasar siempre por un equipo de TI para solicitar acceso. Cuando un usuario solicita acceso y obtiene la aprobación de forma automática o manual por parte de un propietario de la aplicación, el usuario se agrega a un grupo en el back-end. Hay informes disponibles para saber quién ha solicitado, aprobado o eliminado el acceso, y estos le proporcionan control sobre la administración de los roles asignados.

Puede delegar la aprobación de las solicitudes de acceso a las aplicaciones a los aprobadores de la empresa. El aprobador de la empresa puede establecer las contraseñas de acceso a las aplicaciones desde su página Aplicaciones.

Más información: [Uso del acceso de autoservicio a las aplicaciones](access-panel-manage-self-service-access.md).

## <a name="validate-your-deployment"></a>Validación de la implementación

Asegúrese de que la implementación de Aplicaciones se ha probado exhaustivamente y de que hay un plan de reversión en vigor.

Las siguientes pruebas se deben llevar a cabo con dispositivos de propiedad corporativa y con dispositivos personales. Estos casos de prueba también deben reflejar los casos de uso empresarial. A continuación, se muestran algunos casos basados en los requisitos empresariales de ejemplo de este documento y en escenarios técnicos habituales. Agregue otros que sean específicos a sus necesidades.

#### <a name="application-sso-access-test-case-examples"></a>Ejemplos de casos de prueba de acceso con el inicio de sesión único de aplicación:


| Oportunidad de negocio| Resultado esperado |
| - | -|
| El usuario inicia sesión en el portal Aplicaciones.| El usuario puede iniciar sesión y ver sus aplicaciones. |
| El usuario inicia una aplicación de inicio de sesión único federado.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación de inicio de sesión único con contraseña por primera vez.| El usuario debe instalar la extensión Aplicaciones. |
| El usuario inicia una aplicación de inicio de sesión único con contraseña las veces posteriores.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación desde el portal de Microsoft 365.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación desde Managed Browser.| El usuario inicia sesión automáticamente en la aplicación. |


#### <a name="application-self-service-capabilities-test-case-examples"></a>Casos de prueba de ejemplo de las funcionalidades de autoservicio de aplicaciones


| Oportunidad de negocio| Resultado esperado |
| - | - |
| El usuario puede administrar la pertenencia a la aplicación.| El usuario puede agregar o quitar miembros con acceso a la aplicación. |
| El usuario puede editar la aplicación.| El usuario puede editar la descripción y las credenciales de la aplicación en el caso de aplicaciones de inicio de sesión único con contraseña. |

### <a name="rollback-steps"></a>Pasos de reversión

Es importante planear qué hacer si la implementación no funciona según lo previsto. Si se produce un error en la configuración del inicio de sesión único durante la implementación, debe saber cómo [solucionar los problemas de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/hybrid/tshoot-connect-sso) y reducir el impacto para los usuarios. En circunstancias extremas, puede que necesite [revertir el inicio de sesión único](../manage-apps/plan-sso-deployment.md#rollback-process).


## <a name="manage-your-implementation"></a>Administrar la implementación

Debe usar el rol con menos privilegios para realizar una tarea necesaria en Azure Active Directory. [Revise los distintos roles disponibles](../users-groups-roles/directory-assign-admin-roles.md) y elija el correcto para resolver las necesidades para cada rol de esta aplicación. Es posible que algunos roles se tengan que aplicar temporalmente y quitar una vez completada la implementación.

| Personas| Roles| Rol de Azure AD  |
| - | -| -|
| Administrador del departamento de soporte técnico| Soporte técnico de nivel 1| None |
| Administración de identidades| Configurar y depurar cuando haya problemas que afectan a Azure AD| Administrador global |
| Administrador de aplicaciones| Atestación de usuarios en la aplicación, configuración en usuarios con permisos| None |
| Administradores de infraestructura| Propietario de la sustitución de certificados| Administrador global |
| Propietario de la empresa o parte interesada| Atestación de usuarios en la aplicación, configuración en usuarios con permisos| None |

Puede usar [Privileged Identity Management](../privileged-identity-management/pim-configure.md) para administrar los roles con el fin de permitir la auditoría, control y revisión de acceso adicionales para los usuarios con permisos de directorio.

### <a name="troubleshoot-my-apps-issues"></a>Solución de problemas de Aplicaciones

Cree guías de solución de problemas para su equipo de soporte técnico con escenarios comunes y con vínculos a la documentación de Microsoft sobre sus soluciones. Puede ser conveniente crear guías que dividan el soporte técnico en los distintos niveles que usa su organización.

Consulte las siguientes guías de solución de problemas como referencia:

[Aplicaciones que no aparecen](access-panel-troubleshoot-application-not-appearing.md)

[Aplicaciones inesperadas que aparecen](access-panel-troubleshoot-unexpected-application.md)

[El usuario no puede iniciar sesión en Aplicaciones](access-panel-troubleshoot-web-sign-in-problem.md)

[Problemas al usar el acceso de autoservicio a las aplicaciones](access-panel-troubleshoot-self-service-access.md)

[Problemas con la extensión del explorador](manage-access-panel-browser-extension.md)

## <a name="next-steps"></a>Pasos siguientes

[Planeamiento de una implementación de Azure Multi-Factor Authentication](https://aka.ms/deploymentplans/mfa)
