---
title: Planeamiento de la aplicación de RR. HH. en la nube para el aprovisionamiento de usuarios de Azure Active Directory
description: En este artículo se describe el proceso de implementación de la integración de sistemas de RR. HH. en la nube, como Workday y SuccessFactors, con Azure Active Directory. La integración de Azure AD con el sistema de RR. HH. en la nube tiene como resultado un sistema completo de administración del ciclo de vida de la identidad.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: kenwith
ms.reviewer: arvindha, celested
ms.openlocfilehash: b96b679e967fd898f072b4b1ae195e3dd1061c04
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235706"
---
# <a name="plan-cloud-hr-application-to-azure-active-directory-user-provisioning"></a>Planeamiento de la aplicación de RR. HH. en la nube para el aprovisionamiento de usuarios de Azure Active Directory

Históricamente, el personal del departamento de TI ha confiado en métodos manuales para crear, actualizar y eliminar a empleados. Ha usado métodos como la carga de archivos CSV o scripts personalizados para sincronizar los datos de los empleados. Estos procesos de aprovisionamiento son propensos a errores, inseguros y difíciles de administrar.

Para administrar los ciclos de vida de la identidad de los empleados, proveedores o trabajadores temporales, el [servicio de aprovisionamiento de usuarios de Azure Active Directory (Azure AD)](../app-provisioning/user-provisioning.md) ofrece integración con aplicaciones de recursos humanos (RR. HH.) basadas en la nube. Entre los ejemplos de aplicaciones se incluyen Workday o SuccessFactors.

Azure AD usa esta integración para habilitar los siguientes flujos de trabajo de aplicación de RR. HH. en la nube:

- **Aprovisionar a los usuarios en Active Directory:** aprovisione conjuntos de usuarios seleccionados de una aplicación de RR. HH. en la nube en uno o varios dominios de Active Directory.
- **Aprovisionar a los usuarios solo en la nube en Azure AD:** en escenarios en los que no se usa Active Directory, aprovisione a los usuarios directamente desde la aplicación de RR. HH. en la nube en Azure AD.
- **Reescribir en la aplicación de RR. HH. en la nube:** reescriba los atributos de nombre de usuario y dirección de correo electrónico de Azure AD en la aplicación de RR. HH. en la nube.

> [!NOTE]
> En este plan de implementación se muestra cómo implementar los flujos de trabajo de aplicaciones de RR. HH. en la nube con el aprovisionamiento de usuarios de Azure AD. Para obtener información sobre cómo implementar el aprovisionamiento automático de usuarios en aplicaciones de software como servicio (SaaS), vea [Planeamiento de una implementación del aprovisionamiento automático de usuarios](https://aka.ms/deploymentplans/provisioning).

## <a name="enabled-hr-scenarios"></a>Escenarios de RR. HH. habilitados

El servicio de aprovisionamiento de usuarios de Azure AD permite la automatización de los siguientes escenarios de administración del ciclo de vida de la identidad basados en RR. HH.:

- **Contratación de nuevos empleados:** cuando se agrega a un nuevo empleado a la aplicación de RR. HH. en la nube, se crea automáticamente una cuenta de usuario en Active Directory y Azure AD con la opción de reescribir los atributos de nombre de usuario y dirección de correo electrónico en la aplicación de RR. HH. en la nube.
- **Actualizaciones de los perfiles y los atributos de los empleados:** cuando se actualiza el registro de un empleado (por ejemplo, el nombre, el puesto o el jefe) en la aplicación de RR. HH. en la nube, su cuenta de usuario se actualiza automáticamente en Active Directory y Azure AD.
- **Bajas de empleados:** cuando se da de baja a un empleado en la aplicación de RR. HH. en la nube, su cuenta de usuario se deshabilita automáticamente en Active Directory y Azure AD.
- **Recontrataciones de empleados:** cuando se vuelve a contratar a un empleado en la aplicación de RR. HH. en la nube, su cuenta anterior se puede volver a activar o aprovisionar automáticamente en Active Directory y Azure AD.

## <a name="who-is-this-integration-best-suited-for"></a>¿Para quién es idónea esta integración?

La integración de aplicaciones de RR. HH. en la nube con el aprovisionamiento de usuarios de Azure AD es ideal para organizaciones que:

- Buscan una solución basada en la nube y precompilada para el aprovisionamiento de usuarios de RR. HH. en la nube.
- Necesitan un aprovisionamiento de usuarios directos de la aplicación de RR. HH. en la nube en Active Directory o Azure AD.
- Necesitan que los usuarios se aprovisionen con los datos obtenidos de la aplicación de RR. HH. en la nube.
- Necesitan combinar y mover usuarios, y dejar que se sincronicen con uno o varios bosques, dominios o unidades organizativas de Active Directory en función de únicamente la información de cambio detectada en la aplicación de RR. HH. en la nube.
- Usan Office 365 para el correo electrónico.

## <a name="learn"></a>Obtener información

El aprovisionamiento de usuarios crea una base en la gobernanza de identidades en curso. Mejora la calidad de los procesos empresariales que se basan en datos de identidad autoritativos.

### <a name="terms"></a>Términos

En este artículo se usan los términos siguientes:

- **Sistema de origen**: repositorio de usuarios desde el que se aprovisiona Azure AD. Un ejemplo es una aplicación de RR. HH. en la nube, como Workday o SuccessFactors.
- **Sistema de destino**: repositorio de usuarios en el que aprovisiona Azure AD. Algunos ejemplos son Active Directory, Azure AD, Office 365 u otras aplicaciones SaaS.
- **Proceso de incorporaciones, cambios y bajas**: término que se usa para nuevas contrataciones, transferencias y bajas mediante una aplicación de RR. HH. en la nube como sistema de registros. El proceso se completa cuando el servicio aprovisiona correctamente los atributos necesarios en el sistema de destino.

### <a name="key-benefits"></a>Ventajas principales

Esta función de aprovisionamiento de TI controlada por RR. HH. ofrece las siguientes ventajas empresariales importantes:

- **Aumento de la productividad:** ahora puede automatizar la asignación de las cuentas de usuario y las licencias de Office 365, así como proporcionar acceso a los grupos clave. La automatización de las asignaciones proporciona a las nuevas contrataciones acceso inmediato a sus herramientas de trabajo y aumenta la productividad.
- **Administración de riesgos:** para aumentar la seguridad, puede automatizar los cambios en función del estado de los empleados o de las pertenencias a grupos con datos que fluyen desde la aplicación de RR. HH. en la nube. La automatización de los cambios garantiza que las identidades de los usuarios y el acceso a las aplicaciones principales se actualizan automáticamente cuando los usuarios se mueven o salen de la organización.
- **Control de cumplimiento y gobernanza:** Azure AD admite registros de auditoría nativos de las solicitudes de aprovisionamiento de usuarios que hacen las aplicaciones de los sistemas de origen y de destino. Con la auditoría, puede hacer un seguimiento de quién tiene acceso a las aplicaciones desde una sola pantalla.
- **Administración del costo:** el aprovisionamiento automático reduce los costos al evitar ineficiencias y errores humanos asociados con el aprovisionamiento manual. Reduce la necesidad de desarrollar soluciones personalizadas de aprovisionamiento de usuarios a lo largo del tiempo mediante plataformas heredadas y obsoletas.

### <a name="licensing"></a>Licencias

Para configurar la integración del aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Azure AD, necesita una [licencia válida de Azure AD Premium](https://azure.microsoft.com/pricing/details/active-directory/) y una licencia de la aplicación de RR. HH. en la nube, como Workday o SuccessFactors.

También necesita una licencia válida de suscripción a Azure AD Premium P1 o superior para cada usuario que se obtenga de la aplicación de RR. HH. en la nube y que se aprovisionará en Active Directory o en Azure AD. Un número incorrecto de las licencias que se poseen en la aplicación de RR. HH. en la nube puede dar lugar a errores durante el aprovisionamiento de usuarios.

### <a name="prerequisites"></a>Requisitos previos

- Rol de [administrador de identidades híbridas](../users-groups-roles/directory-assign-admin-roles.md#hybrid-identity-administrator) de Azure AD para configurar el agente de aprovisionamiento de Azure AD Connect.
- Rol de [administrador de aplicaciones](../users-groups-roles/directory-assign-admin-roles.md#application-administrator) de Azure AD para configurar la aplicación de aprovisionamiento en Azure Portal.
- Una instancia de prueba y producción de la aplicación de RR. HH. en la nube.
- Permisos de administrador en la aplicación de RR. HH. en la nube para crear un usuario de integración del sistema y efectuar cambios para probar los datos de los empleados con fines de pruebas.
- Para el aprovisionamiento de usuarios en Active Directory, se necesita un servidor que ejecute Windows Server 2012 (o una versión posterior) con el entorno de ejecución .NET 4.7.1 (o posterior) a fin de hospedar el agente de aprovisionamiento de Azure AD Connect.
- [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md) para sincronizar usuarios entre Active Directory y Azure AD.

### <a name="training-resources"></a>Recursos de aprendizaje

| **Recursos** | **Vínculo y descripción** |
|:-|:-|
| Vídeos | [¿Qué es el aprovisionamiento de usuarios en Active Azure Directory?](https://youtu.be/_ZjARPpI6NI) |
| | [Cómo implementar el aprovisionamiento de usuarios en Azure Active Directory](https://youtu.be/pKzyts6kfrw) |
| Tutoriales | [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure AD](../saas-apps/tutorial-list.md) |
| | [Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios](../saas-apps/workday-inbound-tutorial.md#frequently-asked-questions-faq) |
| Preguntas más frecuentes | [Aprovisionamiento automático de usuarios](../app-provisioning/user-provisioning.md#what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning). |
| | [Aprovisionamiento de WorkDay a Azure AD](../saas-apps/workday-inbound-tutorial.md#frequently-asked-questions-faq) |

### <a name="solution-architecture"></a>Arquitectura de la solución

En el ejemplo siguiente se describe la arquitectura de la solución de aprovisionamiento de usuarios de un extremo a otro para entornos híbridos comunes, e incluye lo siguiente:

- **Flujo de datos de RR. HH. autoritativo desde la aplicación de RR. HH. en la nube a Active Directory.** En este flujo, el evento de RR. HH. (proceso de incorporaciones, cambios y bajas) se inicia en el inquilino de la aplicación de RR. HH. en la nube. El servicio de aprovisionamiento de Azure AD y el agente de aprovisionamiento de Azure AD Connect aprovisionan los datos de los usuarios desde el inquilino de la aplicación de RR. HH. en la nube en Active Directory. En función del evento, puede dar lugar a operaciones de creación, actualización, habilitación y deshabilitación en Active Directory.
- **Sincronice con Azure AD y reescriba el correo electrónico y el nombre de usuario desde la instancia local de Active Directory en la aplicación de RR. HH. en la nube.** Una vez actualizadas las cuentas en Active Directory, se sincronizan con Azure AD mediante Azure AD Connect. Se pueden reescribir los atributos de nombre de usuario y dirección de correo electrónico en el inquilino de la aplicación de RR. HH. en la nube.

![Diagrama de flujo de trabajo](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img1.png)

#### <a name="description-of-workflow"></a>Descripción del flujo de trabajo

En el diagrama se indican los siguientes pasos clave:  

1. El **equipo de recursos humanos** realiza las transacciones en el inquilino de la aplicación de RR. HH. en la nube.
2. El **servicio de aprovisionamiento de Azure AD** ejecuta los ciclos programados desde el inquilino de la aplicación de RR. HH. en la nube e identifica los cambios que deben procesarse para la sincronización con Active Directory.
3. El **servicio de aprovisionamiento de Azure AD** invoca al agente de aprovisionamiento de Azure AD Connect con una carga de solicitud que contiene las operaciones de creación, actualización, habilitación y deshabilitación de las cuentas de Active Directory.
4. El **agente de aprovisionamiento de Azure AD Connect** usa una cuenta de servicio para administrar los datos de la cuenta de Active Directory.
5. **Azure AD Connect** ejecuta una [sincronización](../hybrid/how-to-connect-sync-whatis.md) diferencial para extraer las actualizaciones de Active Directory.
6. Las actualizaciones de **Active Directory** se sincronizan con Azure AD.
7. El **servicio de aprovisionamiento de Azure AD** reescribe el atributo de correo electrónico y el nombre de usuario de Azure AD en el inquilino de la aplicación de RR. HH. en la nube.

## <a name="plan-the-deployment-project"></a>Planeamiento del proyecto de implementación

Tenga en cuenta las necesidades de su organización al determinar la estrategia de esta implementación en su entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que interactúa con las partes interesadas adecuadas](https://aka.ms/deploymentplans). Asegúrese también de que los roles de las partes interesadas del proyecto estén bien entendidos. Documente las partes interesadas, así como sus aportaciones y responsabilidades en el proyecto.

Incluya a un representante de la organización de RR. HH. que pueda proporcionar aportaciones sobre los procesos empresariales de RR. HH. y los requisitos de procesamiento de datos laborales e identidad de los trabajadores existentes.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Comuníquese de forma proactiva con los usuarios sobre cuándo y cómo cambiará su experiencia. Indíqueles cómo obtener soporte técnico si tienen problemas.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Integrar procesos de negocio de RR. HH. y flujos de trabajo de identidad desde la aplicación de RR. HH. en la nube en los sistemas de destino necesita una gran cantidad de validación, transformación y limpieza de datos, así como pruebas de un extremo a otro antes de poder implementar la solución en producción.

Ejecute la configuración inicial en un [entorno piloto](../fundamentals/active-directory-deployment-plans.md#best-practices-for-a-pilot) antes de escalarla a todos los usuarios de producción.

## <a name="select-cloud-hr-provisioning-connector-apps"></a>Selección de aplicaciones de conectores de aprovisionamiento de RR. HH. en la nube

Para facilitar los flujos de trabajo de aprovisionamiento de Azure AD entre la aplicación de RR. HH. en la nube y Active Directory, puede agregar varias aplicaciones de conectores de aprovisionamiento desde la galería de aplicaciones de Azure AD:

- **Aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Active Directory**: esta aplicación de conector de aprovisionamiento facilita el aprovisionamiento de cuentas de usuario de la aplicación de RR. HH. en la nube en un solo dominio de Active Directory. Si tiene varios dominios, puede agregar una instancia de esta aplicación desde la galería de aplicaciones de Azure AD para cada dominio de Active Directory en el que deba efectuar un aprovisionamiento.
- **Aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Azure AD**: si bien Azure AD Connect es la herramienta que se debe usar para sincronizar los usuarios de Active Directory con Azure AD, esta aplicación de conector de aprovisionamiento se puede usar para facilitar el aprovisionamiento de usuarios que solo están en la nube desde la aplicación de RR. HH. en la nube en un inquilino de Azure AD.
- **Reescritura de la aplicación de RR. HH. en la nube**: esta aplicación de conector de aprovisionamiento facilita la reescritura de las direcciones de correo electrónico de los usuarios de Azure AD en la aplicación de RR. HH. en la nube.

Por ejemplo, en la imagen siguiente se enumeran las aplicaciones de conector de Workday que están disponibles en la galería de aplicaciones de Azure AD.

![Galería de aplicaciones del portal de Azure Active Directory](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img2.png)

### <a name="decision-flow-chart"></a>Diagrama de flujo de decisión

Use el siguiente diagrama de decisiones para identificar qué aplicaciones de aprovisionamiento de RR. HH. en la nube son pertinentes en su escenario.

![Diagrama de flujo de decisión](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img3.png)

## <a name="design-the-azure-ad-connect-provisioning-agent-deployment-topology"></a>Diseño de la topología de implementación del agente de aprovisionamiento de Azure AD Connect

La integración de aprovisionamiento entre la aplicación de RR. HH. en la nube y Active Directory necesita cuatro componentes:

- Inquilino de la aplicación de RR. HH. en la nube
- Aplicación de conector de aprovisionamiento
- Agente de aprovisionamiento de Azure AD Connect
- Dominio de Active Directory

La topología de implementación del agente de aprovisionamiento de Azure AD Connect depende del número de inquilinos de la aplicación de RR. HH. en la nube y de los dominios secundarios de Active Directory que se van a integrar. Si tiene varios dominios de Active Directory, dependerá de si estos dominios son contiguos o [no contiguos](/windows-server/identity/ad-ds/plan/disjoint-namespace).

En función de su decisión, elija uno de los escenarios de implementación siguientes:

- Inquilino de una aplicación de RR. HH. en la nube único -> dirigido a uno o varios dominios secundarios de Active Directory en un bosque de confianza.
- Inquilino de una aplicación de RR. HH. en la nube único -> dirigido a varios dominios secundarios en un bosque de Active Directory no contiguo.

### <a name="single-cloud-hr-app-tenant---target-single-or-multiple-active-directory-child-domains-in-a-trusted-forest"></a>Inquilino de una aplicación de RR. HH. en la nube único -> dirigido a uno o varios dominios secundarios de Active Directory en un bosque de confianza.

Se recomienda la siguiente configuración de producción:

|Requisito|Recomendación|
|:-|:-|
|Número de agentes de aprovisionamiento de Azure AD Connect que se van a implementar|Dos (para la alta disponibilidad y la conmutación por error)
|Número de aplicaciones de conectores de aprovisionamiento que se van a configurar|Una aplicación por dominio secundario|
|Host de servidor para el agente de aprovisionamiento de Azure AD Connect|Windows 2012 R2+ con campo visual a los controladores de dominio de Active Directory ubicados geográficamente</br>Puede coexistir con el servicio de Azure AD Connect|

![Flujo a los agentes locales](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img4.png)

### <a name="single-cloud-hr-app-tenant---target-multiple-child-domains-in-a-disjoint-active-directory-forest"></a>Inquilino de una aplicación de RR. HH. en la nube único -> dirigido a varios dominios secundarios en un bosque de Active Directory no contiguo.

Este escenario implica aprovisionar a los usuarios desde la aplicación de RR. HH. en dominios de bosques de Active Directory no contiguos.

Se recomienda la siguiente configuración de producción:

|Requisito|Recomendación|
|:-|:-|
|Número de agentes de aprovisionamiento de Azure AD Connect que se van a implementar localmente|Dos por bosque de Active Directory no contiguo|
|Número de aplicaciones de conectores de aprovisionamiento que se van a configurar|Una aplicación por dominio secundario|
|Host de servidor para el agente de aprovisionamiento de Azure AD Connect|Windows 2012 R2+ con campo visual a los controladores de dominio de Active Directory ubicados geográficamente</br>Puede coexistir con el servicio de Azure AD Connect|

![Bosque único de Active Directory no contiguo del inquilino de la aplicación de RR. HH. en la nube](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img5.png)

### <a name="azure-ad-connect-provisioning-agent-requirements"></a>Requisitos del agente de aprovisionamiento de Azure AD Connect

Para la solución de aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Active Directory es necesario implementar uno o varios agentes de aprovisionamiento de Azure AD Connect en servidores que ejecutan Windows 2012 R2 o versiones posteriores. Los servidores deben tener un mínimo de 4 GB de RAM y un entorno de ejecución .NET 4.7.1+. Asegúrese de que el servidor host tiene acceso de red al dominio de Active Directory de destino.

Para preparar el entorno local, el asistente para la configuración del agente de aprovisionamiento de Azure AD Connect registra el agente en el inquilino de Azure AD, [abre los puertos](../manage-apps/application-proxy-add-on-premises-application.md#open-ports), [permite el acceso a las direcciones URL](../manage-apps/application-proxy-add-on-premises-application.md#allow-access-to-urls) y admite la [configuración de proxy HTTPS saliente](../saas-apps/workday-inbound-tutorial.md#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication).

El agente de aprovisionamiento usa una cuenta de servicio para comunicarse con los dominios de Active Directory. Antes de instalar el agente, cree una cuenta de servicio en Usuarios y equipos de Active Directory que cumpla los siguientes requisitos:

- Una contraseña que no expire
- Permisos de control delegado para leer, crear, eliminar y administrar cuentas de usuario

Puede seleccionar los controladores de dominio que deben controlar las solicitudes de aprovisionamiento. Si tiene varios controladores de dominio distribuidos geográficamente, instale al agente de aprovisionamiento en el mismo sitio que los controladores de dominio preferidos. Este posicionamiento mejora la confiabilidad y el rendimiento de la solución integral.

Para lograr una alta disponibilidad, puede implementar más de un agente de aprovisionamiento de Azure AD Connect. Registre el agente para controlar el mismo conjunto de dominios locales de Active Directory.

## <a name="plan-scoping-filters-and-attribute-mapping"></a>Planeamiento de los filtros de ámbito y la asignación de atributos

Al habilitar el aprovisionamiento desde la aplicación de RR. HH. en la nube en Active Directory o Azure AD, Azure Portal controla los valores de atributo mediante la asignación de atributos.

### <a name="define-scoping-filters"></a>Definición de filtros de ámbito

Use [filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md) para definir las reglas basadas en atributos que determinan qué usuarios se deben aprovisionar desde la aplicación de RR. HH. en la nube en Active Directory o Azure AD.

Al iniciar el proceso de las incorporaciones, reúna los siguientes requisitos:

- ¿La aplicación de RR. HH. en la nube se usa para incluir tanto a los empleados como a los trabajadores temporales?
- ¿Planea usar la aplicación de RR. HH. en la nube para el aprovisionamiento de usuarios de Azure AD para administrar tanto a los empleados como a los trabajadores temporales?
- ¿Tiene pensado implementar el aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Azure AD solo en un subconjunto de los usuarios de la aplicación de RR. HH. en la nube? Un ejemplo podría ser solo los empleados.

En función de sus requisitos, al configurar las asignaciones de atributos, puede establecer el campo **Ámbito de objeto de origen** para seleccionar los conjuntos de usuarios de la aplicación de RR. HH. en la nube que deben incluirse en el aprovisionamiento en Active Directory. Para más información, vea el tutorial de la aplicación de RR. HH. en la nube, donde podrá ver los filtros de ámbito que más se usan.

### <a name="determine-matching-attributes"></a>Determinación de los atributos coincidentes

Con el aprovisionamiento, obtiene la capacidad de hacer coincidir las cuentas existentes entre el sistema de origen y el de destino. Al integrar la aplicación de RR. HH. en la nube con el servicio de aprovisionamiento de Azure AD, puede [configurar la asignación de atributos](../app-provisioning/configure-automatic-user-provisioning-portal.md#mappings) para determinar qué datos de los usuarios deben ir de la aplicación de RR. HH. en la nube a Active Directory o Azure AD.

Al iniciar el proceso de las incorporaciones, reúna los siguientes requisitos:

- ¿Cuál es el identificador único de esta aplicación de RR. HH. en la nube que se usa para identificar a cada usuario?
- Desde la perspectiva del ciclo de vida de las identidades, ¿cómo se administran las recontrataciones? En las recontrataciones, ¿se conserva el identificador anterior de los empleados?
- ¿Procesa las contrataciones con fecha futura y crea cuentas de Active Directory para estas de antemano?
- Desde la perspectiva del ciclo de vida de las identidades, ¿cómo controla la conversión de empleado a trabajador temporal o a la inversa?
- ¿Los usuarios convertidos conservan sus cuentas anteriores de Active Directory u obtienen otras nuevas?

En función de sus requisitos, Azure AD admite la asignación directa de atributo a atributo al proporcionar valores constantes o [escribir expresiones de asignaciones de atributos](../app-provisioning/functions-for-customizing-application-data.md). Esta flexibilidad le brinda un mayor control de lo que se rellena en el atributo de la aplicación de destino. Puede usar [Microsoft Graph API](../app-provisioning/export-import-provisioning-configuration.md) y Probador de Graph para exportar el esquema y las asignaciones de atributos de aprovisionamiento de usuarios en un archivo JSON e importarlos de nuevo en Azure AD.

De forma predeterminada, el atributo de la aplicación de RR. HH. en la nube que representa el identificador de empleado único se usa como atributo coincidente *asignado al atributo único en Active Directory*. Por ejemplo, en el escenario de la aplicación Workday, el atributo **WorkerID** de **Workday** está asignado al atributo **employeeID** de Active Directory.

Puede establecer varios atributos coincidentes y asignar una precedencia de coincidencia. Se evalúan según la precedencia de la coincidencia. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

También puede [personalizar las asignaciones de atributos predeterminados](../app-provisioning/customize-application-attributes.md#understanding-attribute-mapping-types), como cambiar o eliminar asignaciones de atributos existentes. También puede crear asignaciones de atributos según sus necesidades empresariales. Para más información, vea el tutorial de la aplicación de RR. HH. en la nube (por ejemplo, [Workday](../saas-apps/workday-inbound-tutorial.md#managing-your-configuration)), en donde podrá consultar una lista de los atributos personalizados que se pueden asignar.

### <a name="determine-user-account-status"></a>Determinar el estado de la cuenta de usuario

De forma predeterminada, la aplicación de conector de aprovisionamiento asigna el estado de perfil de usuario de RR. HH. al estado de la cuenta de usuario en Active Directory o Azure AD para determinar si se debe habilitar o deshabilitar la cuenta de usuario.

Al iniciar el proceso de las incorporaciones o bajas, reúna los siguientes requisitos.

| Proceso | Requisitos |
| - | - |
| **Incorporaciones** | Desde la perspectiva del ciclo de vida de las identidades, ¿cómo se administran las recontrataciones? En las recontrataciones, ¿se conserva el identificador anterior de los empleados? |
| | ¿Procesa las contrataciones con fecha futura y crea cuentas de Active Directory para estas de antemano? ¿Estas cuentas se crean con el estado habilitado o deshabilitado? |
| | Desde la perspectiva del ciclo de vida de las identidades, ¿cómo controla la conversión de empleado a trabajador temporal o a la inversa? |
| | ¿Los usuarios convertidos conservan sus cuentas anteriores de Active Directory u obtienen otras nuevas? |
| **Bajas** | ¿Las bajas de los empleados y los trabajadores temporales se administran de forma diferente en Active Directory? |
| | ¿Qué fechas vigentes se tienen en cuenta para procesar la baja del usuario? |
| | ¿Cómo afectan las conversiones de empleados y trabajadores temporales a las cuentas de Active Directory existentes? |
| | ¿Cómo procesa la operación de rescisión en Active Directory? Las operaciones de rescisión deben controlarse si se crean contrataciones futuras en Active Directory como parte del proceso de incorporación. |

En función de sus requisitos, puede personalizar la lógica de asignación mediante [expresiones de Azure AD](../app-provisioning/functions-for-customizing-application-data.md) para que la cuenta de Active Directory esté habilitada o deshabilitada según una combinación de puntos de datos.

### <a name="map-cloud-hr-app-to-active-directory-user-attributes"></a>Asignación de atributos de usuario de la aplicación de RR. HH. en la nube en Active Directory

Cada aplicación de RR. HH. en la nube se incluye con asignaciones predeterminadas de la aplicación de RR. HH. en la nube en Active Directory.

Al iniciar el proceso de incorporaciones, cambios o bajas, reúna los siguientes requisitos.

| Proceso | Requisitos |
| - | - |
| **Incorporaciones** | ¿El proceso de creación de cuentas de Active Directory es manual, automatizado o parcialmente automatizado? |
| | ¿Tiene pensado propagar los atributos personalizados de la aplicación de RR. HH. en la nube a Active Directory? |
| **Cambios** | ¿Qué atributos quiere procesar cada vez que se realice una operación de cambio en la aplicación de RR. HH. en la nube? |
| | ¿Se realizan validaciones de atributo específicas en el momento de realizar las actualizaciones de usuarios? Si la respuesta es que sí, proporcione detalles. |
| **Bajas** | ¿Las bajas de los empleados y los trabajadores temporales se administran de forma diferente en Active Directory? |
| | ¿Qué fechas vigentes se tienen en cuenta para procesar la baja del usuario? |
| | ¿Cómo afectan las conversiones de empleados y trabajadores temporales a las cuentas de Active Directory existentes? |

En función de sus requisitos, puede modificar las asignaciones para satisfacer sus objetivos de integración. Para más información, vea el tutorial de la aplicación específica de RR. HH. en la nube (por ejemplo, [Workday](../saas-apps/workday-inbound-tutorial.md#part-4-configure-attribute-mappings)), en donde podrá consultar una lista de los atributos personalizados que se pueden asignar.

### <a name="generate-a-unique-attribute-value"></a>Generación de un valor de atributo único

Al iniciar el proceso de incorporaciones, es posible que tenga que generar valores de atributo únicos cuando establezca atributos como CN, samAccountName y UPN que tengan restricciones únicas.

La función de Azure AD [SelectUniqueValues](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue) evalúa cada regla y, después, comprueba la unicidad del valor generado en el sistema de destino. Si quiere consultar un ejemplo, vea [Generación de un valor único para el atributo userPrincipalName (UPN)](../app-provisioning/functions-for-customizing-application-data.md#generate-unique-value-for-userprincipalname-upn-attribute).

> [!NOTE]
> Actualmente, esta función solo se admite en el aprovisionamiento de usuarios de Workday en Active Directory y no se puede usar con otras aplicaciones de aprovisionamiento.

### <a name="configure-active-directory-ou-container-assignment"></a>Configuración de la asignación de contenedores de unidades organizativas de Active Directory

Un requisito común es colocar las cuentas de usuario de Active Directory en contenedores en función de las unidades, las ubicaciones y los departamentos de la empresa. Cuando se inicia un proceso de cambio, y si hay un cambio en la organización de supervisión, es posible que tenga que mover al usuario de una unidad organizativa a otra en Active Directory.

Use la función [Switch()](../app-provisioning/functions-for-customizing-application-data.md#switch) para configurar la lógica de negocios de la asignación de la unidad organizativa y asígnela al atributo **parentDistinguishedName** de Active Directory.

Por ejemplo, si quiere crear usuarios en la unidad organizativa según el atributo de RR. HH. **Municipio**, puede usar la siguiente expresión:

`
Switch([Municipality], "OU=Default,OU=Users,DC=contoso,DC=com", "Dallas", "OU=Dallas,OU=Users,DC=contoso,DC=com", "Austin", "OU=Austin,OU=Users,DC=contoso,DC=com", "Seattle", "OU=Seattle,OU=Users,DC=contoso,DC=com", "London", "OU=London,OU=Users,DC=contoso,DC=com")
`

Con esta expresión, si el valor de Municipio es Dallas, Austin, Seattle o Londres, la cuenta de usuario se creará en la unidad organizativa correspondiente. Si no hay ninguna coincidencia, la cuenta se creará en la unidad organizativa predeterminada.

## <a name="plan-for-password-delivery-of-new-user-accounts"></a>Planeamiento de la entrega de contraseñas de nuevas cuentas de usuario

Al iniciar el proceso de incorporaciones, debe establecer y proporcionar una contraseña temporal a las nuevas cuentas de usuario. Con el aprovisionamiento de usuarios de RR. HH. en la nube en Azure AD, puede implementar la función de [autoservicio de restablecimiento de contraseña](../authentication/tutorial-enable-sspr.md) (SSPR) de Azure AD para el usuario desde el primer día.

SSPR es un medio sencillo con el que los administradores de TI pueden permitir que los usuarios restablezcan sus contraseñas o desbloqueen sus cuentas. Puede aprovisionar el atributo **Número de móvil** desde la aplicación de RR. HH. en la nube en Active Directory y sincronizarlo con Azure AD. Una vez que el atributo **Número de móvil** esté en Azure AD, puede habilitar la SSPR en la cuenta del usuario. El primer día, el nuevo usuario puede usar el número de móvil registrado y verificado para autenticarse.

## <a name="plan-for-initial-cycle"></a>Planeamiento del ciclo inicial

Cuando el servicio de aprovisionamiento de Azure AD se ejecuta por primera vez, se ejecuta un [ciclo inicial](../app-provisioning/how-provisioning-works.md#initial-cycle) en la aplicación de RR. HH. en la nube para crear una instantánea de todos los objetos de usuario en la aplicación de RR. HH. en la nube. El tiempo que tardan los ciclos iniciales depende directamente del número de usuarios que haya en el sistema de origen. El ciclo inicial de algunos inquilinos de aplicaciones de RR. HH. en la nube con más de 100 000 usuarios puede llevar mucho tiempo.

**En el caso de los inquilinos de aplicaciones de RR. HH. en la nube de gran tamaño (>30 000 usuarios)** , ejecute el ciclo inicial en fases progresivas. Inicie las actualizaciones incrementales después de validar que se han establecido en Active Directory los atributos correctos de distintos escenarios de aprovisionamiento de usuarios. Siga el orden que se indica aquí.

1. Ejecute el ciclo inicial solo para un conjunto limitado de usuarios estableciendo el [filtro de ámbito](#plan-scoping-filters-and-attribute-mapping).
2. Compruebe el aprovisionamiento de cuentas de Active Directory y los valores de atributo establecidos para los usuarios seleccionados en la primera ejecución. Si el resultado cumple con sus expectativas, expanda el filtro de ámbito para que incluya más usuarios de forma progresiva y compruebe los resultados de la segunda ejecución.

Cuando esté conforme con los resultados del ciclo inicial de los usuarios de prueba, inicie las [actualizaciones incrementales](../app-provisioning/how-provisioning-works.md#incremental-cycles).

## <a name="plan-testing-and-security"></a>Planeamiento de pruebas y seguridad

En cada fase de la implementación, desde el piloto inicial hasta la habilitación del aprovisionamiento de usuarios, asegúrese de que está probando que los resultados son los esperados y auditando los ciclos de aprovisionamiento.

### <a name="plan-testing"></a>Planeamiento de pruebas

Después de configurar el aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube en Azure AD, ejecute casos de prueba para comprobar si esta solución cumple con los requisitos de su organización.

|Escenarios|Resultados esperados|
|:-|:-|
|Se contrata al nuevo empleado en la aplicación de RR. HH. en la nube.| - Se aprovisiona la cuenta de usuario en Active Directory.</br>- El usuario puede iniciar sesión en las aplicaciones del dominio de Active Directory y realizar las acciones que quiera.</br>- Si se ha configurado la sincronización de Azure AD Connect, la cuenta de usuario también se crea en Azure AD.
|Se da de baja al usuario en la aplicación de RR. HH. en la nube.|- Se deshabilita la cuenta de usuario en Active Directory.</br>- El usuario no puede iniciar sesión en ninguna de las aplicaciones empresariales protegidas por Active Directory.
|Se actualiza la organización de supervisión de usuarios en la aplicación de RR. HH. en la nube.|En función de la asignación de atributos, la cuenta de usuario se mueve de una unidad organizativa a otra en Active Directory.|
|El departamento de RR. HH. actualiza al administrador del usuario en la aplicación de RR. HH. en la nube.|Se actualiza el campo de administrador de Active Directory para que refleje el nombre del nuevo administrador.|
|RR. HH. recontrata a un empleado en un nuevo rol.|El comportamiento depende de cómo se haya configurado la aplicación de RR. HH. en la nube para generar los identificadores de los empleados:</br>- Si se vuelve a usar el identificador anterior del empleado en el caso de que se le vuelva a contratar, el conector habilita la cuenta existente de Active Directory del usuario.</br>- Si el empleado al que se vuelve a contratar obtiene un nuevo identificador, el conector crea otra cuenta de Active Directory para el usuario.|
|El departamento de RR. HH. convierte al empleado en trabajador temporal o viceversa.|Se crea otra cuenta de Active Directory para el nuevo rol y la cuenta anterior se deshabilita en la fecha de aplicación de la conversión.|

Use los resultados anteriores para determinar cómo realizar la transición de la implementación automática del aprovisionamiento de usuarios en producción en función de las escalas de tiempo establecidas.

> [!TIP]
> Use técnicas como la reducción y la limpieza de datos al actualizar el entorno de prueba con los datos de producción para quitar o enmascarar datos personales confidenciales a fin de cumplir con los estándares de privacidad y seguridad. 

### <a name="plan-security"></a>Planeamiento de seguridad

Es habitual que se necesite una revisión de seguridad como parte de la implementación de un nuevo servicio. Si se necesita una revisión de seguridad o no se ha realizado, vea las numerosas [notas del producto](https://www.microsoft.com/download/details.aspx?id=36391) de Azure AD, en las que se proporciona información general sobre la identidad como servicio.

### <a name="plan-rollback"></a>Planeamiento de la reversión

La implementación del aprovisionamiento de usuarios de RR. HH. en la nube podría no funcionar según lo previsto en el entorno de producción. Si es así, los siguientes pasos de reversión pueden ayudarle a volver a un estado anterior que sepa que funciona.

1. Vea el [informe de resumen de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md#getting-provisioning-reports-from-the-azure-portal) y los [registros de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md#provisioning-logs-preview) para determinar qué operaciones incorrectas se han producido en los usuarios o grupos afectados. Para más información sobre el informe de resumen y los registros de aprovisionamiento, vea [Administración del aprovisionamiento de usuarios de la aplicación de RR. HH. en la nube](#manage-your-configuration).
2. El último estado válido conocido de los usuarios o grupos afectados puede determinarse mediante los registros de auditoría de aprovisionamiento o al revisar los sistemas de destino (Azure AD o Active Directory).
3. Trabaje con el propietario de la aplicación para actualizar los usuarios o grupos afectados directamente en la aplicación con los últimos valores de estado válido conocidos.

## <a name="deploy-the-cloud-hr-app"></a>Implementación de la aplicación de RR. HH. en la nube

Elija la aplicación de RR. HH. en la nube que se ajuste a los requisitos de la solución.

**Workday**: para importar perfiles de trabajo de Workday en Active Directory y Azure AD, vea [Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios](../saas-apps/workday-inbound-tutorial.md#planning-your-deployment). Si quiere, puede reescribir la dirección de correo electrónico, el nombre de usuario y el número de teléfono en Workday.

**SAP SuccessFactors**: para importar perfiles de trabajo de SuccessFactors en Active Directory y Azure AD, vea [Tutorial: Configuración del aprovisionamiento de usuarios de SAP SuccessFactors a Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md). Opcionalmente, puede reescribir la dirección de correo electrónico y el nombre de usuario en SuccessFactors.

## <a name="manage-your-configuration"></a>Administración de la configuración

Azure AD puede proporcionar información adicional sobre el uso del aprovisionamiento de usuarios y el estado operativo mediante registros de auditoría e informes.

### <a name="gain-insights-from-reports-and-logs"></a>Obtención de información a partir de informes y registros

Después de un [ciclo inicial](../app-provisioning/how-provisioning-works.md#initial-cycle) correcto, el servicio de aprovisionamiento de Azure AD sigue ejecutando actualizaciones incrementales de manera indefinida, a intervalos definidos en los tutoriales específicos de cada aplicación, hasta que se produzca alguno de los siguientes eventos:

- El servicio se detiene de forma manual. Se desencadena un nuevo ciclo inicial mediante [Azure Portal](https://portal.azure.com/) o el comando adecuado de [Microsoft Graph API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview).
- Se desencadena un nuevo ciclo inicial debido a un cambio en las asignaciones de atributos o los filtros de ámbito.
- El proceso de aprovisionamiento entra en cuarentena debido a una alta tasa de errores. Permanece en cuarentena durante más de cuatro semanas, momento en el que se deshabilita de forma automática.

Para revisar estos eventos y todas las demás actividades que ha realizado el servicio de aprovisionamiento, [aprenda a consultar los registros y obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md).

#### <a name="azure-monitor-logs"></a>Registros de Azure Monitor

Todas las actividades realizadas por el servicio de aprovisionamiento se registran en los registros de aprovisionamiento de Azure AD. Puede enrutar los registros de auditoría de Azure AD a registros de Azure Monitor para su posterior análisis. Con los registros de Azure Monitor (que también se conocen como área de trabajo de Log Analytics), puede consultar los datos para buscar eventos, analizar tendencias y correlacionar varios orígenes de datos. Vea el siguiente [vídeo](https://youtu.be/MP5IaCTwkQg) para conocer las ventajas del uso de registros de Azure Monitor para registros de Azure AD en escenarios de usuario prácticos.

Instale las [vistas de Log Analytics para los registros de actividad de Azure AD](../reports-monitoring/howto-install-use-log-analytics-views.md) a fin de acceder a [informes pregenerados](https://github.com/AzureAD/Deployment-Plans/tree/master/Log%20Analytics%20Views) sobre eventos de aprovisionamiento del entorno.

Para más información, vea [Análisis de registros de actividad de Azure AD con registros de Azure Monitor](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md).

### <a name="manage-personal-data"></a>Administración de datos personales

El agente de aprovisionamiento de Azure AD Connect instalado en Windows Server crea registros en el registro de eventos de Windows que pueden contener datos personales en función de las asignaciones de atributos de la aplicación de RR. HH. en la nube en Active Directory. Para cumplir con las obligaciones de privacidad de los usuarios, configure una tarea programada de Windows que borre el registro de eventos y asegúrese de que no se conserve ningún dato durante más de 48 horas.

El servicio de aprovisionamiento de Azure AD no genera informes, realiza análisis ni proporciona información más allá de 30 días, ya que el servicio no almacena, procesa ni conserva datos que tengan más de 30 días.

### <a name="troubleshoot"></a>Solución de problemas

Para solucionar los problemas que pueden surgir durante el aprovisionamiento, vea los siguientes artículos:

- [Problema al configurar el aprovisionamiento de usuarios para una aplicación de la galería de Azure AD](application-provisioning-config-problem.md)
- [Sincronización de un atributo de Active Directory local con Azure AD para el aprovisionamiento en una aplicación](user-provisioning-sync-attributes-for-mapping.md)
- [Problema al guardar las credenciales de administrador al configurar el aprovisionamiento de usuarios em una aplicación de galería de Azure Active Directory](application-provisioning-config-problem-storage-limit.md)
- [No se aprovisionan usuarios en una aplicación de la galería de Azure AD](application-provisioning-config-problem-no-users-provisioned.md)
- [Aprovisionamiento de un conjunto de usuarios incorrecto en una aplicación de la galería de Azure AD](application-provisioning-config-problem-wrong-users-provisioned.md)
- [Configuración del Visor de eventos de Windows para solucionar problemas del agente](../saas-apps/workday-inbound-tutorial.md#setting-up-windows-event-viewer-for-agent-troubleshooting)
- [Configuración de registros de auditoría de Azure Portal para solucionar problemas con el servicio](../saas-apps/workday-inbound-tutorial.md#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
- [Información sobre registros para las operaciones de creación de cuentas de usuario de AD](../saas-apps/workday-inbound-tutorial.md#understanding-logs-for-ad-user-account-create-operations)
- [Información sobre registros para las operaciones de actualización del administrador](../saas-apps/workday-inbound-tutorial.md#understanding-logs-for-manager-update-operations)
- [Resolución de errores comunes](../saas-apps/workday-inbound-tutorial.md#resolving-commonly-encountered-errors)

### <a name="next-steps"></a>Pasos siguientes

- [Escritura de expresiones para la asignación de atributos](functions-for-customizing-application-data.md)
- [Introducción a la API de sincronización de Azure AD](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)
- [Omisión de la eliminación de cuentas de usuario que están fuera de ámbito](skip-out-of-scope-deletions.md)
- [Agente de aprovisionamiento de Azure AD Connect: historial de versiones](provisioning-agent-release-version-history.md)