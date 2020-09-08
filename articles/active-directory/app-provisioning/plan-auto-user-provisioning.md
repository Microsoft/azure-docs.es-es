---
title: Planeamiento de una implementación de aprovisionamiento automático de usuarios para Azure Active Directory
description: Guía para planear y ejecutar aprovisionamiento automático de usuarios
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 10/17/2019
ms.author: kenwith
ms.reviewer: arvindha, celested
ms.openlocfilehash: 24de1b2cfc5e0804039923f23b6a5cd9f6c6fb80
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235712"
---
# <a name="plan-an-automatic-user-provisioning-deployment"></a>Planeamiento de una implementación del aprovisionamiento automático de usuarios

Muchas organizaciones confían en aplicaciones de software como servicio (SaaS) como ServiceNow, Zscaler y Slack para la productividad del usuario final. Históricamente, el personal de TI ha confiado en métodos de aprovisionamiento manual, como la carga de archivos .csv o el uso de scripts personalizados para administrar de manera segura identidades de usuario en cada aplicación SaaS. Estos procesos son propensos a errores, inseguros y difíciles de administrar.

El aprovisionamiento automático de usuarios de Azure Active Directory (Azure AD) simplifica este proceso mediante la automatización segura de la creación, el mantenimiento y la eliminación de las identidades de usuarios en aplicaciones SaaS basadas en reglas de negocio. Esta automatización permite escalar de manera eficaz los sistemas de administración de identidades en entornos híbridos y solo en la nube a medida que se extiende su dependencia de soluciones basadas en la nube.

Consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md) para comprender mejor la funcionalidad.

## <a name="learn"></a>Obtener información

El aprovisionamiento de usuarios crea las bases de una gobernanza continua de identidades y mejora la calidad de los procesos de negocio que se basan en datos de identidades fidedignos.

### <a name="key-benefits"></a>Ventajas principales

Las ventajas principales de habilitar el aprovisionamiento automático de usuarios son:

* **Mayor productividad**. Puede administrar las identidades de usuario en las distintas aplicaciones SaaS con una interfaz de administración de aprovisionamiento de usuarios única. Esta interfaz tiene un solo conjunto de directivas de aprovisionamiento.

* **Administración de los riesgos**. Para aumentar la seguridad, puede automatizar los cambios en función del estado de los empleados o de las pertenencias a grupos que definen los roles o el acceso.

* **Solución para el cumplimiento y la gobernanza**. Azure AD admite registros de auditoría nativos para cada solicitud de aprovisionamiento de usuarios. Las solicitudes se ejecutan en los sistemas de origen y de destino. Esto le permite realizar un seguimiento de quién tiene acceso a las aplicaciones desde una sola pantalla.

* **Reducción del costo**. El aprovisionamiento automático de usuarios reduce los costos al evitar ineficiencias y errores humanos asociados con el aprovisionamiento manual. Disminuye la necesidad de registros de auditoría, scripts y soluciones de aprovisionamiento de usuarios desarrollados de manera personalizada.

### <a name="licensing"></a>Licencias

Azure AD ofrece la integración de autoservicio de cualquier aplicación mediante las plantillas que se proporcionan en el menú de la galería de aplicaciones. Para una lista completa de los requisitos de licencia, consulte la [página de licencias de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

#### <a name="application-licensing"></a>Licencias de aplicaciones

Necesitará las licencias adecuadas para las aplicaciones que quiera aprovisionar de manera automática. Hable con los propietarios de la aplicación para saber los usuarios asignados a la aplicación tienen las licencias adecuadas para sus roles de aplicación. Si Azure AD administra el aprovisionamiento automático basado en roles, los roles asignados en Azure AD deben alinearse con las licencias de aplicación. Las licencias incorrectas que se poseen en la aplicación pueden dar lugar a errores durante el aprovisionamiento o la actualización de un usuario.

### <a name="terms"></a>Términos

En este artículo se usan los términos siguientes:

* Operaciones CRUD: acciones realizadas en las cuentas de usuario: Crear, leer, actualizar, eliminar.

* Inicio de sesión único (SSO): la capacidad de un usuario de iniciar sesión una vez y acceder a todas las aplicaciones habilitadas para SSO. En el contexto del aprovisionamiento de usuarios, SSO es el resultado de que los usuarios tengan una cuenta única para acceder a todos los sistemas que usan el aprovisionamiento automático de usuarios.

* Sistema de origen: el repositorio de los usuarios desde donde se aprovisiona Azure AD. Azure AD es el sistema de origen para la mayoría de los conectores de aprovisionamiento integrados previamente. Sin embargo, hay algunas excepciones para las aplicaciones en la nube, como SAP, WorkDay y AWS. Por ejemplo, consulte el [aprovisionamiento de usuarios de WorkDay a AD](../saas-apps/workday-inbound-tutorial.md).

* Sistema de destino: el repositorio de los usuarios hacia donde se aprovisiona Azure AD. Por lo general, el sistema de destino es una aplicación SaaS como ServiceNow, Zscaler y Slack. El sistema de destino también puede ser un sistema local, como AD.

* [Sistema para la administración de identidades entre dominios (SCIM)](https://aka.ms/scimoverview): un estándar abierto que permite la automatización del aprovisionamiento de usuarios. SCIM comunica datos de identidades de usuario entre proveedores de identidades, como Microsoft, y proveedores de servicios como Salesforce u otras aplicaciones SaaS que requieren información de identidad de usuario.

### <a name="training-resources"></a>Recursos de aprendizaje

| Recursos| Vínculo y descripción |
| - | - |
| Seminarios web a petición| [Administración de aplicaciones empresariales con Azure AD](https://info.microsoft.com/CO-AZUREPLAT-WBNR-FY18-03Mar-06-ManageYourEnterpriseApplicationsOption1-MCW0004438_02OnDemandRegistration-ForminBody.html)<br>‎Obtenga información sobre cómo Azure AD puede ayudarlo a lograr SSO a las aplicaciones SaaS empresariales y procedimientos recomendados para controlar el acceso. |
| Vídeos| [¿Qué es el aprovisionamiento de usuarios en Active Azure Directory?](https://youtu.be/_ZjARPpI6NI) <br> [¿Cómo implementar el aprovisionamiento de usuarios en Azure Active Directory?](https://youtu.be/pKzyts6kfrw) <br> [Integración de Salesforce con Azure AD: automatización del aprovisionamiento de usuarios](https://azure.microsoft.com/resources/videos/integrating-salesforce-with-azure-ad-how-to-automate-user-provisioning/) |
| Cursos en línea| SkillUp Online:  [Administración de identidades](https://skillup.online/courses/course-v1:Microsoft+AZ-100.5+2018_T3/about) <br> Obtenga información sobre cómo integrar Azure AD con muchas aplicaciones SaaS y proteger el acceso del usuario a esas aplicaciones. |
| Libros| [Modern Authentication with Azure Active Directory for Web Applications (Developer Reference)](https://www.amazon.com/Authentication-Directory-Applications-Developer-Reference/dp/0735696942/ref=sr_1_fkmr0_1?keywords=Azure+multifactor+authentication&qid=1550168894&s=gateway&sr=8-1-fkmr0) (Autenticación remota con Azure Active Directory para aplicaciones web [referencia para desarrolladores]), primera edición.  <br> ‎Se trata de una guía autoritativa y profunda sobre cómo compilar soluciones de autenticación de Active Directory para estos entornos nuevos. |
| Tutoriales| Consulte la [lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure AD](../saas-apps/tutorial-list.md). |
| Preguntas más frecuentes| [Preguntas más frecuentes](../app-provisioning/user-provisioning.md) sobre el aprovisionamiento automatizado de usuarios |

### <a name="solution-architectures"></a>Arquitecturas de las soluciones

El servicio de aprovisionamiento de Azure AD aprovisiona usuarios para las aplicaciones SaaS y otros sistemas mediante la conexión de los puntos de conexión de la API de administración de usuarios que proporciona el proveedor de cada aplicación. Estos puntos de conexión de la API de administración de usuarios permiten a Azure AD crear, actualizar y quitar usuarios mediante programación.

#### <a name="automatic-user-provisioning-for-hybrid-enterprises"></a>Aprovisionamiento automático de usuarios para empresas híbridas

En este ejemplo, los usuarios y los grupos se crean en una base de datos de recursos humanos conectada a un directorio local. El servicio de aprovisionamiento de Azure AD administra el aprovisionamiento automático de usuarios en las aplicaciones SaaS de destino.

 ![aprovisionamiento de usuarios](./media/plan-auto-user-provisioning/hybridprovisioning.png)

**Descripción del flujo de trabajo:**

1. Los usuarios o grupos se crean en un sistema o aplicación de recursos humanos local, como SAP. 

1. El **agente de Azure AD Connect** ejecuta sincronizaciones programadas de identidades (usuarios y grupos) desde la instancia de AD local a Azure AD.

1. El **servicio de aprovisionamiento de Azure AD** comienza un [ciclo inicial](../app-provisioning/user-provisioning.md) en el sistema de origen y el sistema de destino. 

1. El **servicio de aprovisionamiento de Azure AD** consulta el sistema de origen para saber si algún usuario o grupo cambió desde el ciclo inicial y envía los cambios en [ciclos incrementales](../app-provisioning/user-provisioning.md).

#### <a name="automatic-user-provisioning-for-cloud-only-enterprises"></a>Aprovisionamiento automático de usuarios para empresas solo en la nube

En este ejemplo, la creación de usuarios se produce en Azure AD y el servicio de aprovisionamiento de Azure AD administra el aprovisionamiento automático de usuarios en las aplicaciones de destino (SaaS).

![Imagen 2](./media/plan-auto-user-provisioning/cloudprovisioning.png)

**Descripción del flujo de trabajo:**

1. Los usuarios o grupos se crean en Azure AD.

1. El **servicio de aprovisionamiento de Azure AD** comienza un [ciclo inicial](../app-provisioning/user-provisioning.md) en el sistema de origen y el sistema de destino. 

1. El **servicio de aprovisionamiento de Azure AD** consulta el sistema de origen para saber si algún usuario o grupo se actualizó desde el ciclo inicial y realiza cualquier [ciclo incremental](../app-provisioning/user-provisioning.md).

#### <a name="automatic-user-provisioning-for-cloud-hr-applications"></a>Aprovisionamiento automático de usuarios para aplicaciones de recursos humanos en la nube 

En este ejemplo, los usuarios o grupos se crean en una aplicación de recursos humanos en la nube, como Workday y SuccessFactors. El servicio de aprovisionamiento de Azure AD y el agente de aprovisionamiento de Azure AD Connect aprovisionan los datos de los usuarios desde el inquilino de la aplicación de recursos humanos en la nube en AD. Una vez que se actualizan las cuentas en AD, se realiza la sincronización con Azure AD a través de Azure AD Connect, y los atributos de dirección de correo electrónico y nombre de usuario se pueden volver a escribir en el inquilino de la aplicación de recursos humanos en la nube.

![Imagen 2](./media/plan-auto-user-provisioning/workdayprovisioning.png)

1.  El **equipo de recursos humanos** realiza las transacciones en el inquilino de la aplicación de RR. HH. en la nube.
2.  **Servicio de aprovisionamiento de Azure AD** ejecuta los ciclos programados desde el inquilino de la aplicación de RR. HH. en la nube e identifica los cambios que deben procesarse para la sincronización con AD.
3.  El **servicio de aprovisionamiento de Azure AD** invoca al agente de aprovisionamiento de Azure AD Connect con una carga de solicitud que contiene las operaciones de creación, actualización, habilitación o deshabilitación de las cuentas de AD.
4.  El **agente de aprovisionamiento de Azure AD Connect** usa una cuenta de servicio para administrar los datos de la cuenta de AD.
5.  **Azure AD Connect** ejecuta una sincronización diferencial para extraer las actualizaciones de AD.
6.  Las actualizaciones de **AD** se sincronizan con Azure AD. 
7.  El **servicio de aprovisionamiento de Azure AD**  reescribe el atributo de correo electrónico y el nombre de usuario de Azure AD en el inquilino de la aplicación de RR. HH. en la nube.

## <a name="plan-the-deployment-project"></a>Planeamiento del proyecto de implementación

Tenga en cuenta las necesidades de su organización para determinar la estrategia de implementación del aprovisionamiento de usuarios en el entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de involucrar a las partes interesadas correctas](https://aka.ms/deploymentplans) y que los roles de las partes interesadas en el proyecto se entiendan bien; para ello, documente las partes interesadas y sus aportes y responsabilidades en el proyecto.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Comunique de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia, y cómo obtener soporte técnico si tienen cualquier problema.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Se recomienda que la configuración inicial del aprovisionamiento automático de usuarios esté en un entorno de prueba con un subconjunto de usuarios pequeño antes de escalarla a todos los usuarios de producción. Vea los [procedimientos recomendados](../fundamentals/active-directory-deployment-plans.md#best-practices-for-a-pilot) para ejecutar un piloto.

#### <a name="best-practices-for-a-pilot"></a>Procedimientos recomendados para un piloto  

Un piloto le permite probar con un grupo pequeño antes de implementar una funcionalidad para todos. Asegúrese de que, como parte de sus pruebas, cada caso de uso de su organización se prueba de forma exhaustiva.

En su primera oleada, TI de destino, facilidad de uso y otros usuarios adecuados que pueden probar y proporcionar comentarios. Use estos comentarios para desarrollar aún más las comunicaciones e instrucciones que envía a sus usuarios y proporcionarles información sobre los tipos de problemas que puede ver su personal de soporte técnico.

Amplíe la implementación en grupos de usuarios más grandes mediante el aumento del ámbito de los grupos objetivo. Esto puede hacerse a través de la [pertenencia dinámica a grupos](../users-groups-roles/groups-dynamic-membership.md) o agregando usuarios manualmente a los grupos objetivo.

## <a name="plan-application-connections-and-administration"></a>Planeamiento de la administración y conexiones de aplicación

Use el portal de Azure AD para ver y administrar todas las aplicaciones que admiten el aprovisionamiento. Consulte [Búsqueda de aplicaciones en el portal](../app-provisioning/configure-automatic-user-provisioning-portal.md).

### <a name="determine-the-type-of-connector-to-use"></a>Determine el tipo de conector que se va a usar

Los pasos que son necesarios para habilitar y configurar el aprovisionamiento automático varían según la aplicación. Si la aplicación que quiere aprovisionar automáticamente aparece en la [galería de aplicaciones SaaS de Azure AD](../saas-apps/tutorial-list.md), debe seleccionar el [tutorial de integración específico para la aplicación](../saas-apps/tutorial-list.md) para configurar el conector de aprovisionamiento de usuarios integrado previamente.

Si no es así, siga estos pasos:

1. [Cree una solicitud](../azuread-dev/howto-app-gallery-listing.md) para un conector de aprovisionamiento de usuarios integrado previamente. Nuestro equipo trabajará con usted y con el desarrollador de la aplicación para incorporar la aplicación a nuestra plataforma si es compatible con SCIM.

1. Use la compatibilidad con el aprovisionamiento genérico de usuarios [BYOA SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md) de la aplicación. Este es un requisito de Azure AD para aprovisionar usuarios en la aplicación sin un conector de aprovisionamiento integrado previamente.

1. Si la aplicación puede usar el conector BYOA SCIM, consulte el [tutorial de integración de BYOA SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md) para configurar el conector para la aplicación.

Para más información, consulte [¿Qué aplicaciones y sistemas puedo usar con el aprovisionamiento automático de usuarios de Azure AD?](../app-provisioning/user-provisioning.md)

### <a name="collect-information-to-authorize-application-access"></a>Recopilación de información para autorizar el acceso a la aplicación

La configuración del aprovisionamiento automático de usuarios es un proceso por aplicación. Para cada aplicación, necesita proporcionar [credenciales de administrador](../app-provisioning/configure-automatic-user-provisioning-portal.md) para conectarse al punto de conexión del usuario del sistema de destino.

En la imagen siguiente muestra una versión de las credenciales de administración necesarias:

![Pantalla Aprovisionamiento para administrar la configuración del aprovisionamiento de cuentas de usuario](./media/plan-auto-user-provisioning/userprovisioning-admincredentials.png)

Si bien algunas aplicaciones requieren la contraseña y el nombre de usuario del administrador, es posible que otras necesiten un token de portador.

## <a name="plan-user-and-group-provisioning"></a>Planeamiento del aprovisionamiento de usuarios y grupos

Si habilita el aprovisionamiento de usuarios para aplicaciones empresariales, [Azure Portal](https://portal.azure.com/) controla sus valores de atributos a través de la asignación de atributos.

### <a name="determine-operations-for-each-saas-app"></a>Determinación de las operaciones para cada aplicación SaaS

Cada aplicación puede tener atributos de usuario o de grupo únicos que deben asignarse a los atributos de Azure AD. La aplicación puede tener solo un subconjunto de las operaciones CRUD disponibles.

Para cada aplicación, documente la información siguiente:

* Las operaciones CRUD de aprovisionamiento que se realizarán en los objetos de usuario o grupo para los sistemas de destino. Por ejemplo, es posible que cada propietario de empresa de una aplicación SaaS no quiera todas las operaciones posibles.

* Atributos disponibles en el sistema de origen

* Atributos disponibles en el sistema de destino

* Asignación de los atributos entre sistemas.

### <a name="choose-which-users-and-groups-to-provision"></a>Elección de usuarios y grupos que se van a aprovisionar

Antes de implementar el aprovisionamiento automático de usuarios, debe determinar los usuarios y grupos que se aprovisionarán en la aplicación.

* Use [filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md) para definir reglas basadas en atributos que determinen qué usuarios se aprovisionan en una aplicación.

* A continuación, use [asignaciones de usuarios y grupos](../manage-apps/assign-user-or-group-access-portal.md) según sea necesario para el filtrado adicional.

### <a name="define-user-and-group-attribute-mapping"></a>Definición de la asignación de atributos de usuario y grupo

Para implementar el aprovisionamiento automático de usuarios, debe definir los atributos de usuario y grupo necesarios para la aplicación. Hay un conjunto preconfigurado de atributos y [asignaciones de atributos](../app-provisioning/configure-automatic-user-provisioning-portal.md) entre los objetos de usuario de Azure AD y los objetos de usuario de cada aplicación SaaS. No todas las aplicaciones SaaS habilitan atributos de grupo.

Azure AD admite la asignación de atributo a atributo directa, lo que proporciona valores constantes o permite [escribir expresiones para asignaciones de atributos](../app-provisioning/functions-for-customizing-application-data.md). Esta flexibilidad le brinda un control preciso de lo que se rellenará en el atributo del sistema de destino. Puede usar [Microsoft Graph API](../app-provisioning/export-import-provisioning-configuration.md) y Probador de Graph para exportar el esquema y las asignaciones de atributos de aprovisionamiento de usuarios a un archivo JSON e importarlos de nuevo en Azure AD.

Para más información, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](../app-provisioning/customize-application-attributes.md).

### <a name="special-considerations-for-user-provisioning"></a>Consideraciones especiales para el aprovisionamiento de usuarios

Tenga en cuenta lo siguiente para disminuir los problemas posteriores a la implementación:

* Asegúrese de que los atributos usados para asignar objetos de usuario o grupo entre las aplicaciones de origen y de destino sean resistentes. No deberían hacer que los usuarios o grupos se aprovisionen de manera incorrecta si los atributos cambian (por ejemplo, un usuario se mueve a otra parte de la empresa).

* Las aplicaciones pueden tener restricciones o requisitos específicos que deben cumplirse para que el aprovisionamiento de usuarios funcione correctamente. Por ejemplo, Slack trunca los valores de ciertos atributos. Consulte los [tutoriales de aprovisionamiento automático de usuarios](../saas-apps/tutorial-list.md) específicos para cada aplicación.

* Confirme la coherencia del esquema entre los sistemas de origen y de destino. Entre los problemas comunes se incluyen atributos como UPN o correo que no coinciden. Por ejemplo, UPN en Azure AD se establece como *john_smith@contoso.com* y en la aplicación, es *jsmith@contoso.com* . Para más información, consulte la [referencia del esquema de usuario y grupo](../app-provisioning/use-scim-to-provision-users-and-groups.md).

## <a name="plan-testing-and-security"></a>Planeamiento de pruebas y seguridad

En cada fase de la implementación, asegúrese de que está probando que los resultados son los esperados y auditando los ciclos de aprovisionamiento.

### <a name="plan-testing"></a>Planeamiento de pruebas

Una vez que haya configurado el aprovisionamiento automático de usuarios para la aplicación, ejecutará casos de prueba para comprobar que esta solución cumple con los requisitos de su organización.

| Escenarios| Resultados esperados |
| - | - |
| El usuario se agrega a un grupo asignado al sistema de destino | El objeto de usuario se aprovisiona en el sistema de destino. <br>El usuario puede iniciar sesión en el sistema de destino y realizar las acciones deseadas. |
| El usuario se quita de un grupo asignado al sistema de destino | El objeto de usuario se desaprovisiona en el sistema de destino.<br>El usuario no puede iniciar sesión en el sistema de destino. |
| La información de usuario se actualiza en Azure AD con cualquier método | Los atributos de usuario actualizados se reflejan en el sistema de destino después de un ciclo incremental. |
| El usuario está fuera del ámbito | El objeto de usuario está deshabilitado o eliminado. <br>Nota: Este comportamiento se invalida para el [aprovisionamiento de Workday](skip-out-of-scope-deletions.md). |

### <a name="plan-security"></a>Planeamiento de seguridad

Es habitual que se requiera una revisión de seguridad como parte de una implementación. Si necesita una revisión de seguridad, consulte las muchas [notas del producto](https://www.microsoft.com/download/details.aspx?id=36391) de Azure AD que proporcionan información general sobre la identidad como servicio.

### <a name="plan-rollback"></a>Planeamiento de la reversión

Si la implementación del aprovisionamiento automático de usuarios no funciona como se desea en el entorno de producción, los pasos de reversión siguientes pueden ayudarlo a revertir a un estado correcto conocido anterior:

1. Revise el [informe de resumen de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md) y los [registros de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md#provisioning-logs-preview) para determine qué operaciones incorrectas se produjeron en los usuarios o grupos afectados.

1. Use los registros de auditoría del aprovisionamiento para determinar el último estado correcto conocido de los usuarios o grupos afectados. Revise también los sistemas de origen (Azure AD o AD).

1. Trabaje con el propietario de la aplicación para actualizar los usuarios o grupos afectados directamente en la aplicación con los últimos valores de estado correctos conocidos.

## <a name="deploy-automatic-user-provisioning-service"></a>Implementación del servicio de aprovisionamiento automático de usuarios

Elija los pasos que se ajusten a los requisitos de la solución.

### <a name="prepare-for-the-initial-cycle"></a>Preparación del ciclo inicial

Cuando el servicio de aprovisionamiento de Azure AD se ejecuta por primera vez, el ciclo inicial en el sistema de origen y el sistema de destino crea una instantánea de todos los objetos de usuario para cada sistema de destino.

Al habilitar el aprovisionamiento automático para una aplicación, el ciclo inicial puede tardar desde 20 minutos a varias horas. La duración depende del tamaño del directorio de Azure AD y el número de usuarios en el ámbito del aprovisionamiento.

El servicio de aprovisionamiento almacena el estado de ambos sistemas después del ciclo inicial, lo que permite mejorar el rendimiento de los ciclos incrementales siguientes.

### <a name="configure-automatic-user-provisioning"></a>Configuración del aprovisionamiento automático de usuarios

Use [Azure Portal](https://portal.azure.com/) para administrar el aprovisionamiento y el desaprovisionamiento automáticos de cuentas de usuario en aplicaciones que lo admitan. Siga los pasos que aparecen en [¿Cómo configuro el aprovisionamiento automático para una aplicación?](../app-provisioning/user-provisioning.md)

El servicio de aprovisionamiento de usuarios de Azure AD también se puede configurar y administrar mediante [Microsoft Graph API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview).

## <a name="manage-automatic-user-provisioning"></a>Administración del aprovisionamiento automático de usuarios

Ahora que realizó la implementación, es necesario administrar la solución.

### <a name="monitor-user-provisioning-operation-health"></a>Supervisión del estado de las operaciones de aprovisionamiento de usuarios

Después de un [ciclo inicial](../app-provisioning/user-provisioning.md) exitoso, el servicio de aprovisionamiento de Azure AD ejecutará actualizaciones incrementales de manera indefinida, a intervalos específicos para cada aplicación, hasta que se produce alguno de estos eventos:

* El servicio se detiene de manera manual y se desencadena un ciclo inicial nuevo mediante [Azure Portal](https://portal.azure.com/) o con el comando adecuado de [Microsoft Graph API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview).

* Se desencadena un nuevo ciclo inicial debido a un cambio en las asignaciones de atributos o los filtros de ámbito.

* El proceso de aprovisionamiento entra en cuarentena debido a una alta tasa de errores y permanece en ese estado durante más de cuatro semanas, cuando se deshabilitará automáticamente.

Para revisar estos eventos y todas las demás actividades realizadas por el servicio de aprovisionamiento, consulte los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) de Azure AD.

Para saber cuánto tiempo tardan los ciclos de aprovisionamiento y supervisar el progreso del trabajo de aprovisionamiento, puede [comprobar el estado de aprovisionamiento de usuarios](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md).

### <a name="gain-insights-from-reports"></a>Obtención de información a partir de los datos

Azure AD puede proporcionar [información adicional](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) sobre el uso del aprovisionamiento de usuarios y el estado operativo mediante registros de auditoría e informes.

Los administradores deben comprobar el informe de resumen del aprovisionamiento para supervisar el estado operativo del trabajo de aprovisionamiento. Todas las actividades realizadas por el servicio de aprovisionamiento se registran en los registros de aprovisionamiento de Azure AD. Consulte [Tutorial: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

Se recomienda asumir la propiedad de estos informes y consumirlos según una cadencia que cumpla con los requisitos de la organización. Azure AD conserva la mayoría de los datos de auditoría durante 30 días.

### <a name="troubleshoot"></a>Solución de problemas

Consulte los vínculos siguientes para solucionar los problemas que pueden surgir durante el aprovisionamiento:

* [Problema al configurar el aprovisionamiento de usuarios para una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem.md)

* [Sincronización de un atributo de Active Directory local con Azure AD para el aprovisionamiento en una aplicación](../app-provisioning/user-provisioning-sync-attributes-for-mapping.md)

* [Problema al guardar las credenciales de administrador al configurar el aprovisionamiento de usuarios em una aplicación de galería de Azure Active Directory](../app-provisioning/application-provisioning-config-problem-storage-limit.md)

* [No se aprovisionan usuarios en una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md)

* [Aprovisionamiento de un conjunto de usuarios incorrecto en una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem-wrong-users-provisioned.md)

### <a name="helpful-documentation"></a>Documentación útil

* [Escritura de expresiones para la asignación de atributos](../app-provisioning/functions-for-customizing-application-data.md)

* [Introducción a la API de sincronización de Azure AD](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)

* [Omisión de la eliminación de cuentas de usuario que están fuera de ámbito](skip-out-of-scope-deletions.md)

* [Agente de aprovisionamiento de Azure AD Connect: historial de versiones](provisioning-agent-release-version-history.md)

#### <a name="resources"></a>Recursos

* [Proporcionar comentarios sobre el producto](https://feedback.azure.com/forums/169401-azure-active-directory)

* [Manténgase al día con las novedades de Azure AD](https://azure.microsoft.com/updates/?product=active-directory)

* [Foro de Azure AD sobre el desbordamiento de pila](https://stackoverflow.com/questions/tagged/azure-active-directory)

## <a name="next-steps"></a>Pasos siguientes
* [Configuración del aprovisionamiento automático de usuarios](../app-provisioning/configure-automatic-user-provisioning-portal.md)

* [Exportación o importación de la configuración de aprovisionamiento mediante Microsoft Graph API](../app-provisioning/export-import-provisioning-configuration.md)

* [Escritura de expresiones para la asignación de atributos en Azure Active Directory](../app-provisioning/functions-for-customizing-application-data.md)