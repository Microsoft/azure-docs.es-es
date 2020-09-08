---
title: 'Cuatro pasos para una base de identidad sólida: Azure AD'
description: En este tema se describen cuatro pasos que pueden seguir los clientes de identidad híbrida para establecer las bases de una identidad segura.
services: active-directory
author: martincoetzer
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/20/2019
ms.subservice: hybrid
ms.author: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b7e394c4227be9ead491fe77457b466b1f62468
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279336"
---
# <a name="four-steps-to-a-strong-identity-foundation-with-azure-active-directory"></a>Cuatro pasos para establecer las bases de una identidad segura con Azure Active Directory

La administración del acceso a datos y aplicaciones ya no puede basarse en las tradicionales estrategias de los límites de seguridad de la red, como firewalls y redes perimetrales, dado el rápido movimiento de las aplicaciones a la nube. Ahora, las organizaciones deben confiar en su solución de identidad para controlar quién y qué tiene acceso a los datos y las aplicaciones de la organización. Cada vez más organizaciones permiten a los empleados traer sus propios dispositivos para trabajar y usar sus dispositivos desde cualquier lugar desde el que puedan conectarse a Internet. Garantizar que esos dispositivos son seguros y cumplen las normas se ha convertido en un importante aspecto a tener en cuenta en la solución de identidad que una organización elige implementar. En el lugar de trabajo actual, la [identidad es el plano de control principal](https://www.microsoft.com/security/technology/identity-access-management?rtc=1) de cualquier organización que se mueve a la nube.

A la hora de adoptar una solución de identidad híbrida de Azure Active Directory (Azure AD), las organizaciones obtienen acceso a características premium que liberan la productividad gracias a las funcionalidades de automatización, delegación, autoservicio e inicio de sesión único. Los trabajadores pueden acceder a los recursos de la empresa desde cualquier parte, y el equipo de TI puede controlar ese acceso y garantizar que las personas adecuadas tienen el acceso adecuado a los recursos correctos para establecer una productividad segura.

Según lo que sabemos, esta lista de comprobación de procedimientos recomendados le ayudará a implementar rápidamente las acciones recomendadas para sentar las bases de la identidad *segura* en su organización:

* Conexión fácil a las aplicaciones
* Establecimiento de una identidad para cada usuario automáticamente
* Capacitación a los usuarios de forma segura
* Operacionalización de las conclusiones

## <a name="step-1---connect-to-apps-easily"></a>Paso 1: Conexión fácil a las aplicaciones

Al conectar las aplicaciones con Azure AD, puede mejorar la productividad y la seguridad del usuario final al permitir el inicio de sesión único (SSO) y realizar el aprovisionamiento de usuarios. Como las aplicaciones se administran en un solo lugar, Azure AD, puede minimizar la sobrecarga administrativa y lograr un único punto de control para sus directivas de seguridad y cumplimiento.

En esta sección se tratan las opciones para administrar el acceso de los usuarios a las aplicaciones, de forma que es posible el acceso remoto seguro a aplicaciones internas, y las ventajas de migrar sus aplicaciones a Azure AD.

### <a name="make-apps-available-to-your-users-seamlessly"></a>Permitir que las aplicaciones estén disponibles para usuarios íntegramente

Azure AD permite a los administradores [agregar aplicaciones](../manage-apps/add-application-portal.md) a la galería de aplicaciones empresariales en [Azure Portal](https://portal.azure.com/). Agregar aplicaciones a la galería de aplicaciones empresariales facilita la configuración de aplicaciones que usan Azure AD como proveedor de identidades. También permite administrar el acceso de los usuarios a las aplicaciones con directivas de acceso condicional y configurar el inicio de sesión único (SSO) en las aplicaciones, de forma que los usuarios no tengan que escribir sus contraseñas varias veces e inicien sesión automáticamente tanto en aplicaciones locales como basadas en la nube.

Una vez que se agregan aplicaciones a la galería de Azure AD, los usuarios pueden ver aquellas que tienen asignadas y buscar y solicitar otras aplicaciones según sea necesario. Azure AD proporciona [varios métodos](../manage-apps/end-user-experiences.md) para que los usuarios accedan a sus aplicaciones:

* Panel de acceso/Aplicaciones
* Iniciador de aplicaciones de Office 365
* Inicio de sesión directo en aplicaciones federadas
* Vínculos directos de inicio de sesión

Para más información sobre el acceso de los usuarios a las aplicaciones, consulte el **Paso 3: Capacitación a los usuarios** en este artículo.

### <a name="migrate-apps-from-active-directory-federation-services-to-azure-ad"></a>Migración de aplicaciones desde los Servicios de federación de Active Directory (AD FS) a Azure AD

La migración de la configuración de inicio de sesión único desde los Servicios de federación de Active Directory (AD FS) hasta Azure AD permite funcionalidades adicionales de seguridad, una capacidad de administración más coherente y la colaboración. Para obtener resultados óptimos, se recomienda migrar las aplicaciones de AD FS a Azure AD. Incluir la autenticación y la autorización de las aplicaciones en Azure AD proporciona las siguientes ventajas:

* Administración de costos
* Administración de riesgos
* Aumento de la productividad
* Satisfacción de los requisitos de gobierno y cumplimiento

Para más información, consulte las notas del producto [Migrating Your Applications to Azure Active Directory](https://aka.ms/migrateapps/whitepaper) (Migración de las aplicaciones a Azure Active Directory).

### <a name="enable-secure-remote-access-to-apps"></a>Habilitación del acceso remoto seguro a las aplicaciones

[Azure AD Application Proxy](../manage-apps/what-is-application-proxy.md) ofrece a las organizaciones una solución sencilla para publicar aplicaciones locales en la nube destinadas a usuarios remotos que necesitan tener acceso a aplicaciones internas de forma segura. Después de un inicio de sesión único en Azure AD, los usuarios pueden acceder a aplicaciones locales y en la nube mediante direcciones URL externas o un portal de aplicaciones interno.

Azure AD Application Proxy ofrece las siguientes ventajas:

* Extensión de Azure AD para recursos locales
  * Seguridad y protección de escala en la nube
  * Características como acceso condicional y autenticación multifactor que son fáciles de habilitar
* No hay componentes en la red perimetral, como VPN y soluciones tradicionales de proxy inverso
* No se requieren conexiones entrantes
* Inicio de sesión único (SSO) en todos los dispositivos, recursos y aplicaciones en la nube y locales
* Capacita a los usuarios finales para ser productivos en cualquier momento y lugar

### <a name="discover-shadow-it-with-microsoft-cloud-app-security"></a>Detectar TI en la sombra con Microsoft Cloud App Security

En las empresas modernas, los departamentos de TI no son a menudo conscientes de todas las aplicaciones en la nube que usan los usuarios para hacer su trabajo. Cuando a los administradores de TI se les pregunta por el número de aplicaciones que creen que usan sus empleados, como media dicen 30 o 40. En realidad, los empleados de la organización usan más de 1000 aplicaciones diferentes. El 80 % de los empleados usan aplicaciones no autorizadas que nadie ha revisado y que puede que no sean compatibles con las directivas de seguridad y cumplimiento.

[Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) (MCAS) puede ayudarle a identificar aplicaciones útiles que son populares entre los usuarios, y que es posible que TI no autorice, y agregarlas a la galería de aplicaciones empresariales para que los usuarios puedan beneficiarse de funcionalidades como inicio de sesión único y acceso condicional.

<em>"**Cloud App Security** nos ayuda a garantizar que nuestro personal usa correctamente las aplicaciones SaaS y en la nube de formas tales que respalden las directivas de seguridad fundamentales que ayudan a proteger Accenture".</em> --- [John Blasi, director general de seguridad de la información en Accenture](https://customers.microsoft.com/story/accenture-professional-services-cloud-app-security)

Además de detectar TI en la sombra, MCAS también puede determinar el nivel de riesgo de las aplicaciones, impedir el acceso no autorizado a los datos corporativos, evitar una posible pérdida de datos y otros riesgos de seguridad inherentes a las aplicaciones.

## <a name="step-2---establish-one-identity-for-every-user-automatically"></a>Paso 2: Establecimiento de una identidad para cada usuario automáticamente

Reunir directorios locales y basados en la nube en una solución de identidad híbrida de Azure AD le permitirá reutilizar su inversión existente en Active Directory local mediante el aprovisionamiento de las identidades existentes en la nube. La solución sincroniza las identidades locales con Azure AD, mientras el personal de TI sigue ejecutando Active Directory local con cualquier solución de gobierno existente como origen principal de veracidad de las identidades. Las soluciones de identidad de Azure AD de Microsoft abarcan funcionalidades locales y basadas en la nube, de forma que se crea una identidad de usuario común para la autenticación y la autorización en todos los recursos, sin importar su ubicación.

Cuando los directorios locales se integran con Azure AD, aumenta la productividad de los usuarios dado que al proporcionarse una identidad común para acceder a los recursos locales y en la nube, los usuarios no necesitan usar varias cuentas con las distintas aplicaciones y servicios. El uso de varias cuentas es un problema para los usuarios finales y para el personal de TI por igual. Desde la perspectiva del usuario final, tener varias cuentas significa tener que recordar varias contraseñas. Para evitar esto, muchos usuarios reutilizan la misma contraseña con cada cuenta, lo que no es bueno para la seguridad. Desde la perspectiva del personal de TI, la reutilización conduce generalmente a más restablecimientos de contraseñas y costos de soporte técnico, junto con quejas del usuario final.

Azure AD Connect es la herramienta que se usa para sincronizar las identidades locales con Azure AD, que luego se pueden emplear para acceder a las aplicaciones en la nube. Una vez que las identidades están en Azure AD, pueden aprovisionarse con aplicaciones SaaS como Salesforce o Concur.

En esta sección, se enumeran las recomendaciones para proporcionar alta disponibilidad y autenticación moderna para la nube, y para reducir la superficie local.

> [!NOTE]
> Para más información sobre Azure AD Connect, consulte [¿Qué es Azure AD Connect Sync?](./how-to-connect-sync-whatis.md)

### <a name="set-up-a-staging-server-for-azure-ad-connect-and-keep-it-up-to-date"></a>Configurar un servidor de ensayo de Azure AD Connect y mantenerlo actualizado

Azure AD Connect desempeña un papel clave en el proceso de aprovisionamiento. Si por algún motivo el servidor de sincronización se queda sin conexión, los cambios en el entorno local no se actualizarán en la nube y se producirán problemas de acceso para los usuarios. Es importante definir una estrategia de conmutación por error que permita a los administradores reanudar rápidamente la sincronización después de que el servidor de sincronización se queda sin conexión.

Para proporcionar alta disponibilidad en caso de que el servidor principal de Azure AD Connect se quede sin conexión, se recomienda implementar otro [servidor de ensayo](./how-to-connect-sync-staging-server.md) para Azure AD Connect. La implementación de un servidor permite al administrador "promover" el servidor de ensayo a producción mediante un sencillo cambio de configuración. Tener un servidor en espera configurado en modo de ensayo también le permite probar e implementar nuevos cambios de configuración e introducir un nuevo servidor si retira el antiguo.

> [!TIP]
> Azure AD Connect se actualiza de forma periódica. Por lo tanto, se recomienda mantener actualizado el servidor de ensayo para aprovechar las mejoras de rendimiento, corregir errores y hacer uso de las nuevas funcionalidades que proporciona cada nueva versión.

### <a name="enable-cloud-authentication"></a>Habilitar la autenticación en la nube

Las organizaciones con una instancia local de Active Directory deben extender su directorio a Azure AD con Azure AD Connect y configurar el método de autenticación adecuado. La [elección del método de autenticación correcto](./choose-ad-authn.md) para su organización es el primer paso en su viaje para mover las aplicaciones a la nube. Este componente es esencial puesto que controla el acceso a todos los datos y recursos de la nube.

El método más sencillo y recomendado para permitir la autenticación en la nube para los objetos de directorio local en Azure AD es habilitar la [sincronización de hash de contraseña](./how-to-connect-password-hash-synchronization.md) (PHS). Como alternativa, algunas organizaciones pueden considerar la posibilidad de habilitar la [autenticación de paso a través](./how-to-connect-pta-quick-start.md) (PTA).

Tanto si elige PHS o PTA, no olvide habilitar el [inicio de sesión único de conexión directa](./how-to-connect-sso.md) para permitir que los usuarios accedan a las aplicaciones en la nube sin escribir constantemente su nombre de usuario y contraseña cuando usan dispositivos Windows 7 y 8 en la red de la empresa. Sin el inicio de sesión único, los usuarios deben recordar cada contraseña específica de la aplicación e iniciar sesión en cada aplicación. Igualmente, el personal de TI debe crear y actualizar cuentas de usuario para cada aplicación como, por ejemplo, Office 365, Box y Salesforce. Los usuarios tienen que recordar sus contraseñas, además de dedicar tiempo a iniciar sesión en cada aplicación. Proporcionar un mecanismo estándar de inicio de sesión único para toda la empresa es fundamental para mejorar la experiencia del usuario, reducir el riesgo, generar informes y facilitar el gobierno.

En el caso de las organizaciones que ya usan AD FS u otro proveedor de autenticación local, el hecho de pasarse a Azure AD como proveedor de identidades puede reducir la complejidad y mejorar la disponibilidad. A menos que tenga escenarios específicos para el uso de la federación, se recomienda migrar de la autenticación federada a PHS y SSO de conexión directa o a PTA y SSO de conexión directa para disfrutar de las ventajas de tener una superficie local reducida y la flexibilidad que ofrece la nube con las experiencias de usuario mejoradas. Para más información, consulte [Migración de la federación a la sincronización de hash de contraseña para Azure Active Directory](./plan-migrate-adfs-password-hash-sync.md).

### <a name="enable-automatic-deprovisioning-of-accounts"></a>Habilitación del desaprovisionamiento automático de cuentas

La mejor estrategia para regular el ciclo de vida de las identidades entre numerosos sistemas es habilitar el aprovisionamiento y desaprovisionamiento automáticos para las aplicaciones. Azure AD admite el [aprovisionamiento y desaprovisionamiento automáticos basados en directivas](../app-provisioning/configure-automatic-user-provisioning-portal.md) de cuentas de usuario en una variedad de aplicaciones SaaS conocidas, como ServiceNow y Salesforce, además de otras que implementan el [protocolo SCIM 2.0](../app-provisioning/use-scim-to-provision-users-and-groups.md). A diferencia de las soluciones de aprovisionamiento tradicionales, que requieren código personalizado o la carga manual de archivos CSV, el servicio de aprovisionamiento se hospeda en la nube y presenta conectores ya integrados que se pueden configurar y administran mediante Azure Portal. Una de las principales ventajas del desaprovisionamiento automático es que ayuda a proteger su organización al eliminar al instante las identidades de los usuarios de las aplicaciones SaaS cuando abandonan la organización.

Para más información sobre el aprovisionamiento automático de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="step-3---empower-your-users-securely"></a>Paso 3: Capacitación a los usuarios de forma segura

En el lugar de trabajo digital de hoy en día, es importante encontrar un equilibrio entre seguridad y productividad. Sin embargo, los usuarios finales a menudo rechazan las medidas de seguridad que ralentizar su productividad y el acceso a las aplicaciones en la nube. Para ayudar a solucionar este problema, Azure AD proporciona funcionalidades de autoservicio que permiten a los usuarios seguir siendo productivos, al tiempo que reducen la sobrecarga administrativa.

En esta sección se enumeran las recomendaciones para evitar fricciones en su organización y que sea posible capacitar a los usuarios pero al mismo tiempo estar alerta.

### <a name="enable-self-service-password-reset-for-all-users"></a>Habilitación del autoservicio de restablecimiento de contraseña para todos los usuarios

El [autoservicio de restablecimiento de contraseña (SSPR)](../authentication/tutorial-enable-sspr.md) de Azure ofrece a los administradores de TI un medio simple para permitir a los usuarios restablecer y desbloquear sus contraseñas o cuentas sin la intervención del administrador. El sistema incluye informes detallados del seguimiento de acceso de los usuarios al sistema, además de notificaciones de alerta de posibles abusos o usos indebidos.

De forma predeterminada, Azure AD desbloquea las cuentas cuando se realiza un restablecimiento de contraseña. Sin embargo, cuando habilite la [integración local](../authentication/concept-sspr-howitworks.md#on-premises-integration) de Azure AD Connect, también tendrá la opción de separar esas dos operaciones, de forma que los usuarios desbloqueen su cuenta sin tener que restablecer la contraseña.

### <a name="ensure-all-users-are-registered-for-mfa-and-sspr"></a>Garantizar que todos los usuarios están registrados en MFA y SSPR

Azure proporciona informes que puede usarlos usted y su organización para asegurarse de que los usuarios se registren en MFA y SSPR. Puede que los usuarios que aún no se hayan registrado necesiten instrucciones para realizar el proceso.

El [informe de inicios de sesión](../authentication/howto-mfa-reporting.md) de MFS incluye información sobre el uso de MFA y le ofrece conclusiones sobre el funcionamiento de MFA en su organización. El acceso a la actividad de inicio de sesión (así como a las auditorías y las detecciones de riesgo) de Azure AD es fundamental para la solución de problemas, el análisis de uso y las investigaciones forenses.

Del mismo modo, el [informe de autoservicio de administración de contraseñas](../authentication/howto-sspr-reporting.md) puede usarse para determinar quién se ha registrado (o no) en SSPR.

### <a name="self-service-app-management"></a>Autoservicio de administración de aplicaciones

Para que los usuarios puedan detectar automáticamente aplicaciones en su panel de acceso, debe habilitar el [autoservicio de acceso a las aplicaciones](../manage-apps/access-panel-manage-self-service-access.md) para las aplicaciones que quiera que los usuarios detecten automáticamente y puedan solicitar acceso. El autoservicio de acceso a las aplicaciones es una excelente manera de permitir que los usuarios detecten automáticamente las aplicaciones y, si lo desea, permitir que el grupo de negocios apruebe el acceso a esas aplicaciones. Puede permitir que el grupo de negocios administre las credenciales asignadas a esos usuarios para que puedan realizar un [inicio de sesión único con contraseña en las aplicaciones](../manage-apps/application-sign-in-problem-password-sso-gallery.md#configure-password-sso-for-an-azure-ad-gallery-app) directamente desde sus paneles de acceso.

### <a name="self-service-group-management"></a>Self-service group management (Administración de grupos de autoservicio)

La asignación de usuarios a las aplicaciones se realiza mejor cuando se usan grupos, porque permiten una gran flexibilidad y la posibilidad de administrarlos a escala:

* Basada en atributos con pertenencia dinámica a grupos
* Delegación a propietarios de la aplicación

Azure AD le proporciona la capacidad de administrar el acceso a los recursos mediante grupos de seguridad y grupos de Office 365. Estos grupos los puede administrar el propietario de un grupo que pueda aprobar o rechazar solicitudes de pertenencia y delegar el control de la pertenencia a grupos. Esta característica, conocida como [administración de grupos de autoservicio](../users-groups-roles/groups-self-service-management.md), ahorra tiempo al permitir a los propietarios de los grupos que no tienen asignado un rol administrativo crear y administrar los grupos sin depender de los administradores para gestionar sus solicitudes.

## <a name="step-4---operationalize-your-insights"></a>Paso 4: Operacionalización de las conclusiones

La auditoría y el registro de eventos relacionados con la seguridad y las alertas relacionadas son componentes esenciales de una estrategia eficaz para garantizar que los usuarios son productivos y la organización es segura. Los registros e informes de seguridad pueden ayudarle a responder a preguntas como:

* ¿Está usando por lo que paga?
* ¿Está sucediendo algo sospechoso o malintencionado en mi inquilino?
* ¿Qué resultó afectado durante un incidente de seguridad?

Los registros e informes de seguridad proporcionan un registro electrónico de actividades sospechosas y le ayudan a detectar patrones que puedan indicar un acceso externo a la red, así como ataques internos. Puede usar la auditoría para supervisar la actividad del usuario y el cumplimiento normativo de documentos, realizar análisis forenses y mucho más. Además, las alertas le proporcionarán notificaciones acerca de eventos de seguridad.

### <a name="assign-least-privileged-admin-roles-for-operations"></a>Asignación de roles de administrador con menos privilegios para las operaciones

Cuando piense en su enfoque sobre las operaciones, es necesario tener en cuenta dos niveles de administración. El primer nivel coloca la carga de administración sobre los administradores globales. Usar siempre el rol de administrador global podría ser adecuado para pequeñas empresas. Sin embargo, en el caso de organizaciones más grandes con personal de departamento de soporte técnico y administradores responsables de tareas específicas, asignar el rol de administrador global puede ser un riesgo de seguridad, ya que proporciona a esos individuos la posibilidad de administrar tareas que están por encima de sus capacidades.

En este caso, debería considerar el siguiente nivel de administración. Con Azure AD, puede designar a los usuarios finales como "administradores limitados" que pueden administrar tareas en roles con menos privilegios. Por ejemplo, podría asignar al personal del departamento de soporte técnico el rol [lector de seguridad](../users-groups-roles/directory-assign-admin-roles.md#security-reader) con el fin de proporcionarles la capacidad de administrar las características relacionadas con la seguridad con acceso de solo lectura. O, quizás, tenga sentido asignar el rol de [administrador de autenticación](../users-groups-roles/directory-assign-admin-roles.md#authentication-administrator) a individuos para ofrecerles la posibilidad de restablecer credenciales sin contraseña o de leer y configurar Azure Service Health.

Para más información, consulte [Permisos de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).

### <a name="monitor-hybrid-components-azure-ad-connect-sync-ad-fs-using-azure-ad-connect-health"></a>Supervisión de componentes híbridos (Azure AD Connect Sync, AD FS) con Azure AD Connect Health

Azure AD Connect y AD FS son componentes esenciales que pueden afectar a la administración y la autenticación del ciclo de vida y, en última instancia, provocar interrupciones. Por lo tanto, debe implementar Azure AD Connect Health para la supervisión y la notificación de información sobre estos componentes.

Para más información, lea [Supervisión de AD FS mediante Azure AD Connect Health](./how-to-connect-health-adfs.md).

### <a name="use-azure-monitor-to-collect-data-logs-for-analytics"></a>Uso de Azure Monitor para recopilar registros de datos de análisis

[Azure Monitor](../../azure-monitor/overview.md) es un portal de supervisión unificado de todos los registros de Azure AD, que proporciona conclusiones detalladas, análisis avanzados y aprendizaje automático inteligente. Con Azure Monitor, puede consumir métricas y registros en el portal y mediante las API para obtener una mayor visibilidad sobre el estado y el rendimiento de sus recursos. Proporciona una experiencia de panel único dentro del portal y al mismo tiempo ofrece la posibilidad de integrar una amplia variedad de productos mediante API y opciones de exportación de datos que admiten sistemas tradicionales de SIEM de terceros. Azure Monitor también ofrece la posibilidad de configurar reglas de alerta para recibir notificaciones o de realizar acciones automatizadas sobre problemas que afectan a los recursos.

![Azure Monitor](./media/four-steps/image1.png)

### <a name="create-custom-dashboards-for-your-leadership-and-your-day-to-day"></a>Crear paneles personalizados para el personal directivo y sus ocupaciones diarias

Las organizaciones que no tienen una solución SIEM pueden descargar el [paquete de contenido de Power BI](../reports-monitoring/howto-use-azure-monitor-workbooks.md) para Azure AD. El paquete de contenido de Power BI incluye informes pregenerados para ayudarle a comprender cómo los usuarios adoptan y usan características de Azure AD, lo que permite extraer conclusiones sobre todas las actividades dentro del directorio. También puede crear su propio [panel personalizado](/power-bi/service-dashboards) y compartirlo con su equipo de dirección para informar sobre las actividades diarias. Los paneles son una excelente manera de supervisar el negocio y ver todas las métricas más importantes de un vistazo. Las visualizaciones de un panel pueden proceder de uno o varios conjuntos de datos subyacentes o de uno o varios informes subyacentes. En un panel se combinan datos locales y en la nube, de forma que se obtiene una vista consolidada con independencia de donde residan los datos.

![Panel personalizado de Power BI](./media/four-steps/image2.png)

### <a name="understand-your-support-call-drivers"></a>Comprender los factores desencadenantes de las llamadas de soporte técnico

Al implementar una solución de identidad híbrida como se describe en este artículo, debería observar al final una reducción en las llamadas al servicio de soporte técnico. Problemas comunes, como contraseñas olvidadas y bloqueos de cuentas se reducen mediante la implementación del autoservicio de restablecimiento de contraseña de Azure, mientras que la habilitación del autoservicio de acceso a las aplicaciones permite a los usuarios la detección automática y la solicitud de acceso a las aplicaciones sin depender del personal de TI.

Si no observa una reducción de las llamadas de soporte técnico, se recomienda analizar los factores que las desencadenan a fin de confirmar si SSPR o el autoservicio de acceso a las aplicaciones se han configurado correctamente o si hay algún otro problema que se pueda solucionar sistemáticamente.

*"En nuestro recorrido por la transformación digital, necesitábamos un proveedor confiable de administración de identidades y acceso para facilitar la integración completa pero segura entre nosotros, los asociados y los proveedores de servicios en la nube, con la finalidad de obtener un ecosistema eficaz; Azure AD fue la mejor opción ya que nos ofrecía las funcionalidades y la visibilidad necesarias que nos permitían detectar los riesgos y responder a ellos".* --- [Aramex Yazan Almasri, director de seguridad de la información global en Aramex](https://customers.microsoft.com/story/aramex-azure-active-directory-travel-transportation-united-arab-emirates-en)

### <a name="monitor-your-usage-of-apps-to-drive-insights"></a>Supervisar el uso de aplicaciones para extraer conclusiones

Además de detectar la TI en la sombra, la supervisión del uso de las aplicaciones en la organización mediante [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) puede ayudarles a medida que se mueve a la nube para aprovechar la promesa de las aplicaciones de esta plataforma. También, le puede permitir tener el control de sus recursos gracias a la visibilidad mejorada sobre la actividad y al aumento de la protección de datos críticos en todas las aplicaciones en la nube. Supervisar el uso de las aplicaciones en su organización mediante MCAS puede ayudarle a responder a las preguntas siguientes:

* ¿Qué aplicaciones no autorizadas usan los empleados para almacenar los datos?
* ¿Dónde y cuándo se almacenan los datos confidenciales en la nube?
* ¿Quién tiene acceso a datos confidenciales en la nube?

*"Con Cloud App Security, podemos detectar rápidamente anomalías y tomar medidas".* --- [Eric LePenske, directivo superior de seguridad de la información en Accenture](https://customers.microsoft.com/story/accenture-professional-services-cloud-app-security)

## <a name="summary"></a>Resumen

Hay muchos aspectos que se deben tener en cuenta al implementar una solución de identidad híbrida, pero esta lista de comprobación de cuatro pasos le ayudará a lograr rápidamente una infraestructura de identidad que permita a los usuarios ser más productivos y seguros.

* Conexión fácil a las aplicaciones
* Establecimiento de una identidad para cada usuario automáticamente
* Capacitación a los usuarios de forma segura
* Operacionalización de las conclusiones

Esperamos que este documento sea una guía útil para establecer una buena base para la identidad en su organización.

## <a name="identity-checklist"></a>Lista de comprobación de identidad

Se recomienda imprimir la lista de comprobación siguiente como referencia cuando comience su viaje hacia una base de identidad más sólida en su organización.

### <a name="today"></a>Hoy

|¿Ha acabado?|Elemento|
|:-|:-|
||Poner a prueba el autoservicio de restablecimiento de contraseña (SSPR) para un grupo|
||Supervisar los componentes híbridos mediante Azure AD Connect Health|
||Asignar roles de administrador con menos privilegios para las operaciones|
||Detectar TI en la sombra con Microsoft Cloud App Security|
||Usar Azure Monitor para recopilar registros de datos para el análisis|

### <a name="next-two-weeks"></a>Próximas dos semanas

|¿Ha acabado?|Elemento|
|:-|:-|
||Poner una aplicación a disposición de los usuarios|
||Poner a prueba el aprovisionamiento de Azure AD para una aplicación SaaS de su elección|
||Configurar un servidor de ensayo para Azure AD Connect y mantenerlo actualizado|
||Iniciar la migración de aplicaciones de ADFS a Azure AD|
||Crear paneles personalizados para el personal directivo y sus ocupaciones diarias|

### <a name="next-month"></a>Próximo mes

|¿Ha acabado?|Elemento|
|:-|:-|
||Supervisar el uso de aplicaciones para extraer conclusiones|
||Poner aprueba el acceso remoto seguro a las aplicaciones|
||Garantizar que todos los usuarios están registrados en MFA y SSPR|
||Habilitar la autenticación en la nube|

### <a name="next-three-months"></a>Tres próximos meses

|¿Ha acabado?|Elemento|
|:-|:-|
||Habilitar el autoservicio de administración de aplicaciones|
||Habilitar la administración de grupos de autoservicio|
||Supervisar el uso de aplicaciones para extraer conclusiones|
||Comprender los factores desencadenantes de las llamadas de soporte técnico|

## <a name="next-steps"></a>Pasos siguientes

Descubra cómo puede mejorar su posición de seguridad mediante las funcionalidades de Azure Active Directory y esta lista de comprobación de cinco pasos: [Cinco pasos para asegurar su infraestructura de identidad](https://aka.ms/securitysteps).

Conozca cómo las características de identidad de Azure AD pueden ayudarle a acelerar la transición a la administración regulada por la nube al proporcionar las soluciones y funcionalidades que permiten a las organizaciones adoptarla rápidamente y mover una mayor parte de administración de identidades desde sistemas tradicionales locales hasta Azure AD: [Cómo Azure AD ofrece administración regulada por la nube para cargas de trabajo locales](https://aka.ms/cloudgoverned).