---
title: Aprovisionamiento automatizado de usuarios para aplicaciones SaaS en Azure AD | Microsoft Docs
description: Una introducción sobre cómo puede usar Azure AD para el aprovisionamiento, el desaprovisionamiento y la actualización continua de cuentas de usuario de manera automática en varias aplicaciones SaaS de terceros.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/02/2019
ms.author: mimart
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0bdae98f19be11cb960fe53c444c34a0b521f90e
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824239"
---
# <a name="automate-user-provisioning-and-deprovisioning-to-saas-applications-with-azure-active-directory"></a>Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory

## <a name="what-is-automated-user-provisioning-for-saas-apps"></a>¿Qué es el aprovisionamiento automático de usuarios para aplicaciones SaaS?
Azure Active Directory (Azure AD) le permite automatizar la creación, mantenimiento y eliminación de identidades de usuario en la nube ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) aplicaciones como Dropbox, Salesforce, ServiceNow y mucho más.

> [!VIDEO https://www.youtube.com/embed/_ZjARPpI6NI]

**Esta característica le permite:**

- Crear cuentas automáticamente en los sistemas adecuados para los usuarios nuevos cuando se unen a su equipo u organización.
- Desactivar las cuentas automáticamente en los sistemas adecuados cuando los usuarios dejan el equipo o la organización.
- Asegurarse de que las identidades de las aplicaciones y sistemas se mantienen actualizadas con los cambios del directorio o el sistema de recursos humanos.
- Aprovisionar los objetos que no sean de los usuarios, como los grupos, para las aplicaciones que los admitan.

**Aprovisionamiento automático de usuarios, también incluye esta funcionalidad:**

- La capacidad de hacer coincidir las identidades existentes entre los sistemas de origen y de destino.
- Asignaciones de atributos personalizables que definen qué datos del usuario deben fluir del sistema de origen al sistema de destino.
- Alertas de correo electrónico opcionales para errores de aprovisionamiento.
- Informes y registros de actividades para facilitar la supervisión y solución de problemas.

## <a name="why-use-automated-provisioning"></a>¿Por qué usar el aprovisionamiento automático?

Las razones habituales por las que se debe usar esta característica son:

- Evitar los costos, las ineficiencias y los errores humanos asociados con los procesos de aprovisionamiento manuales.
- Evitar los costos asociados al hospedaje y el mantenimiento de scripts y soluciones de aprovisionamiento desarrollados de forma personalizada.
- Protección de su organización mediante la eliminación al instante las identidades de usuarios de aplicaciones SaaS claves cuando abandonan la organización.
- Importar fácilmente un gran número de usuarios en una determinada aplicación o sistema SaaS.
- Tener un único conjunto de directivas para determinar quién se aprovisiona y quién puede iniciar sesión en una aplicación.

## <a name="how-does-automatic-provisioning-work"></a>¿Cómo funciona el aprovisionamiento automático?
    
El **el servicio de aprovisionamiento de Azure AD** aprovisiona usuarios para aplicaciones SaaS y otros sistemas mediante la conexión a puntos de conexión de API de administración de usuario proporcionadas por cada proveedor de la aplicación. Estos puntos de conexión de la API de administración de usuarios permiten a Azure AD crear, actualizar y quitar usuarios mediante programación. Para las aplicaciones seleccionadas, el servicio de aprovisionamiento también puede crear, actualizar y quitar objetos adicionales relacionados con la identidad, como los grupos y roles. 

![Aprovisionamiento](./media/user-provisioning/provisioning0.PNG)
*Figura 1: Servicio de aprovisionamiento de Azure AD*

![Aprovisionamiento de salida](./media/user-provisioning/provisioning1.PNG)
*Figura 2: Flujo de trabajo "saliente" del aprovisionamiento de usuarios desde Azure AD a aplicaciones SaaS populares*

![Aprovisionamiento de entrada](./media/user-provisioning/provisioning2.PNG)
*Figura 3: Flujo de trabajo "entrante" del aprovisionamiento de usuarios desde aplicaciones populares de administración del capital humano (HCM) a Azure Active Directory y Windows Server Active Directory*


## <a name="what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning"></a>¿Qué aplicaciones y sistemas puedo usar con el aprovisionamiento automático de usuarios de Azure AD?

Características de Azure AD integrado previamente para muchos populares aplicaciones y recursos humanos sistemas SaaS y la compatibilidad con genérico para las aplicaciones que implementan determinadas partes del estándar SCIM 2.0.

### <a name="pre-integrated-applications"></a>Aplicaciones preintegradas

Para ver una lista de todas las aplicaciones en las que Azure AD admite un conector de aprovisionamiento previamente integrado, consulte la [lista de tutoriales de aplicaciones de aprovisionamiento de usuarios](../saas-apps/tutorial-list.md).

Si desea ponerse en contacto con el equipo de ingeniería de Azure AD para solicitar soporte técnico para el aprovisionamiento de aplicaciones adicionales, envíe un mensaje a través del [foro de comentarios de Azure Active Directory](https://feedback.azure.com/forums/374982-azure-active-directory-application-requests/filters/new?category_id=172035).

> [!NOTE]
> Para que una aplicación admita el aprovisionamiento automático de usuarios, primero debe proporcionar las API de administración de usuarios necesarias que permitan a los programas externos automatizar la creación, el mantenimiento y la eliminación de usuarios. Por lo tanto, no todas las aplicaciones SaaS son compatibles con esta característica. Para las aplicaciones que admiten las API de administración de usuario, el equipo de ingeniería de Azure AD, a continuación, puede crear un conector de aprovisionamiento para esas aplicaciones, y este trabajo es ordenada por las necesidades de los clientes actuales y potenciales. 

### <a name="connecting-applications-that-support-scim-20"></a>Conexión de aplicaciones que admiten SCIM 2.0

Para obtener información sobre cómo conectar aplicaciones de forma general que implementen API de administración de usuario basado en SCIM 2.0, consulte [Uso de SCIM para aprovisionar automáticamente a los usuarios y grupos de Azure Active Directory para aplicaciones](use-scim-to-provision-users-and-groups.md).

    
## <a name="how-do-i-set-up-automatic-provisioning-to-an-application"></a>¿Cómo configuro el aprovisionamiento automático para una aplicación?

> [!VIDEO https://www.youtube.com/embed/pKzyts6kfrw]

Use el portal de Azure Active Directory para configurar el servicio de una aplicación seleccionada de aprovisionamiento de Azure AD.

1. Abra el  **[portal de Azure Active Directory](https://aad.portal.azure.com)**.

1. Seleccione **aplicaciones empresariales** en el panel izquierdo. Una lista de todas las aplicaciones configuradas es mostrar.

1. Elija **+ nueva aplicación** para agregar una aplicación. Agregue cualquiera de las siguientes según el escenario:

   - El **agregar su propia aplicación** opción es compatible con las integraciones de SCIM personalizada.

   - Todas las aplicaciones de la **agregar desde la galería** > **aplicaciones destacadas** sección admiten el aprovisionamiento automático. Consulte la [lista de tutoriales de aplicaciones para el aprovisionamiento de usuarios](../saas-apps/tutorial-list.md) para ver más.

1. Proporcione los detalles y seleccione **agregar**. La nueva aplicación se agrega a la lista de aplicaciones empresariales y se abre en su pantalla de administración de aplicaciones.

1. Seleccione **aprovisionamiento** para administrar la configuración de la aplicación de aprovisionamiento de cuentas de usuario.

   ![Configuración](./media/user-provisioning/provisioning_settings0.PNG)

1. Seleccione la opción automática para el **modo de aprovisionamiento** para especificar la configuración de credenciales de administrador, asignaciones, iniciar y detener y la sincronización.

   - Expanda **las credenciales de administrador** que escriba las credenciales necesarias para que Azure AD para conectarse a la API de administración de usuario de la aplicación. Esta sección también le permite habilitar notificaciones por correo electrónico si las credenciales producen error o el trabajo de aprovisionamiento entra en [cuarentena](#quarantine).

   - Expanda **asignaciones** para ver y editar los atributos de usuario que fluyen entre Azure AD y la aplicación de destino cuando se aprovisiona o se actualizan las cuentas de usuario. Si la aplicación de destino lo admite, esta sección le permite configurar, opcionalmente, el aprovisionamiento de grupos y cuentas de usuario. Seleccione una asignación de la tabla para abrir el editor de asignación a la derecha, donde puede ver y personalizar los atributos de usuario.
   
     **Los filtros de ámbito** indicar al servicio de aprovisionamiento qué usuarios y grupos en el sistema de origen se deben aprovisionar o desaprovisionar para el sistema de destino. En el **asignación de atributos** panel, seleccione **ámbito de objeto de origen** para filtrar por valores de atributo concreto. Por ejemplo, puede especificar que solo los usuarios con el atributo "Department" (Departamento) de "Sales" (Ventas) deben estar en el ámbito del aprovisionamiento. Para más información, consulte [Uso de filtros de ámbito](define-conditional-rules-for-provisioning-user-accounts.md).
    
     Para obtener más información, vea [Personalización de la asignación de atributos](customize-application-attributes.md).

   - **Configuración** controlan el funcionamiento del servicio de aprovisionamiento para una aplicación, incluso si se está ejecutando actualmente. El **ámbito** menú le permite especificar si solo usuarios y grupos asignados deben incluirse en el ámbito de aprovisionamiento, o si se deben aprovisionar todos los usuarios en el directorio de Azure AD. Para obtener más información sobre la "asignación" de usuarios y grupos, consulte [Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory](assign-user-or-group-access-portal.md).

En la pantalla de administración de aplicaciones, seleccione **registros de auditoría** para ver los registros de cada operación de ejecutan el Azure AD provisioning service. Para obtener más información, consulte la [guía de informes de aprovisionamiento](check-status-user-account-provisioning.md).

![Configuración](./media/user-provisioning/audit_logs.PNG)

> [!NOTE]
> El servicio de aprovisionamiento de usuarios de Azure AD también se puede configurar y administrar mediante [Microsoft Graph API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview).


## <a name="what-happens-during-provisioning"></a>¿Qué ocurre durante el aprovisionamiento?

Si Azure AD es el sistema de origen, el servicio de aprovisionamiento usa la [característica de consulta diferencial de Azure AD Graph API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-differential-query) para supervisar usuarios y grupos. El servicio de aprovisionamiento ejecuta una sincronización con el sistema de origen y el sistema de destino, seguida de sincronizaciones incrementales periódicas. 

### <a name="initial-sync"></a>Sincronización inicial

Cuando se inicia el servicio de aprovisionamiento, la primera sincronización que nunca se ejecute lo siguiente:

1. Se consultan todos los usuarios y grupos del sistema de origen y se recuperan todos los atributos definidos en la [asignación de atributos](customize-application-attributes.md).
2. Se filtran los usuarios y grupos devueltos mediante cualquier [asignación](assign-user-or-group-access-portal.md) o [filtro de ámbito basado en atributos](define-conditional-rules-for-provisioning-user-accounts.md) configurados.
3. Cuando un usuario está asignado o en el ámbito de aprovisionamiento, el servicio consulta el sistema de destino para un usuario coincidente mediante especificado [atributos coincidentes](customize-application-attributes.md#understanding-attribute-mapping-properties). Ejemplo: Si el nombre de userPrincipal en el sistema de origen es el atributo coincidente y se asigna a userName en el sistema de destino, el servicio de aprovisionamiento consulta el sistema de destino en busca de valores userName que coincidan con los valores de nombre del sistema de origen.
4. Si un usuario coincidente no se encuentra en el sistema de destino, se crea mediante los atributos devueltos por el sistema de origen. Una vez creada la cuenta de usuario, el servicio de aprovisionamiento detecta y se almacena en caché el identificador del sistema de destino para el nuevo usuario, que se usa para ejecutar todas las futuras operaciones en los que el usuario.
5. Si se encuentra un usuario coincidente, se actualiza con los atributos proporcionados por el sistema de origen. Después de que coincide con la cuenta de usuario, el servicio de aprovisionamiento detecta y almacena en caché el identificador del sistema de destino para el nuevo usuario, que se usa para ejecutar todas las futuras operaciones sobre ese usuario.
6. Si las asignaciones de atributos contienen atributos de "referencia", el servicio realiza actualizaciones adicionales en el sistema de destino para crear y vincular los objetos que se hace referencia. Por ejemplo, un usuario puede tener un atributo "Administrador" en el sistema de destino, que está vinculado a otro usuario creado en el sistema de destino.
7. Conservar una marca de agua al final de la sincronización inicial, que proporciona el punto de partida para las sincronizaciones incrementales más adelante.

Algunas aplicaciones como la compatibilidad con ServiceNow, G Suite y Box no solo el aprovisionamiento de usuarios, sino también aprovisionamiento de grupos y sus miembros. En esos casos, si el aprovisionamiento de grupos está habilitado en el [asignaciones](customize-application-attributes.md), el servicio de aprovisionamiento sincroniza los usuarios y los grupos y, a continuación, más adelante se sincroniza la pertenencia a grupos. 

### <a name="incremental-syncs"></a>Sincronizaciones incrementales

Después de la sincronización inicial, todas las otras sincronizaciones hará lo siguiente:

1. Se consulta el sistema de origen de los usuarios y grupos que se actualizaron desde la última marca de agua almacenada.
2. Se filtran los usuarios y grupos devueltos mediante cualquier [asignación](assign-user-or-group-access-portal.md) o [filtro de ámbito basado en atributos](define-conditional-rules-for-provisioning-user-accounts.md) configurados.
3. Cuando un usuario está asignado o en el ámbito de aprovisionamiento, el servicio consulta el sistema de destino para un usuario coincidente mediante especificado [atributos coincidentes](customize-application-attributes.md#understanding-attribute-mapping-properties).
4. Si un usuario coincidente no se encuentra en el sistema de destino, se crea mediante los atributos devueltos por el sistema de origen. Una vez creada la cuenta de usuario, el servicio de aprovisionamiento detecta y se almacena en caché el identificador del sistema de destino para el nuevo usuario, que se usa para ejecutar todas las futuras operaciones en los que el usuario.
5. Si se encuentra un usuario coincidente, se actualiza con los atributos proporcionados por el sistema de origen. Si es una cuenta recién asignada que coincide, el servicio de aprovisionamiento detecta y almacena en caché el identificador del sistema de destino para el nuevo usuario, que se usa para ejecutar todas las futuras operaciones sobre ese usuario.
6. Si las asignaciones de atributos contienen atributos de "referencia", el servicio realiza actualizaciones adicionales en el sistema de destino para crear y vincular los objetos que se hace referencia. Por ejemplo, un usuario puede tener un atributo "Administrador" en el sistema de destino, que está vinculado a otro usuario creado en el sistema de destino.
7. Si un usuario que estaba anteriormente en ámbito del aprovisionamiento se quita del ámbito (lo que incluye estar sin asignar), el servicio deshabilita al usuario en el sistema de destino mediante una actualización.
8. Si un usuario que estaba anteriormente en ámbito del aprovisionamiento se deshabilita o se elimina temporalmente en el sistema de origen, el servicio deshabilita al usuario en el sistema de destino mediante una actualización.
9. Si un usuario que estaba anteriormente en ámbito del aprovisionamiento se elimina permanentemente en el sistema de origen, el servicio elimina al usuario en el sistema de destino. En Azure AD, los usuarios son eliminan permanentemente 30 días después de que están eliminados temporalmente.
10. Conservar una marca de agua nueva al final de la sincronización incremental, que proporciona el punto de partida para las sincronizaciones incrementales más adelante.

>[!NOTE]
> Opcionalmente, puede deshabilitar el **crear**, **actualización**, o **eliminar** las operaciones mediante la **acciones del objeto de destino** casillas de verificación en la [Asignaciones](customize-application-attributes.md) sección. La lógica para deshabilitar un usuario durante una actualización también se controla mediante una asignación de atributos desde un campo como "accountEnabled".

El servicio de aprovisionamiento sigue ejecutando sincronizaciones incrementales opuestas de forma indefinida, a intervalos definidos en el [tutorial específica para cada aplicación](../saas-apps/tutorial-list.md), hasta que se produce uno de los siguientes eventos:

- El servicio se detiene manualmente mediante Azure Portal o por medio del comando de Graph API adecuado. 
- Se desencadena una nueva sincronización inicial mediante la opción **Clear state and restart** (Borrar estado y reiniciar) de Azure Portal o por medio del comando de Graph API adecuado. Esta acción borra cualquier marca de agua almacenada y hace que todos los objetos de origen se evalúen de nuevo.
- Se desencadena una nueva sincronización inicial debido a un cambio en las asignaciones de atributos o filtros de ámbito. Esta acción también borra cualquier marca de agua almacenada y hace que todos los objetos de origen se evalúen de nuevo.
- El proceso de aprovisionamiento entra en cuarentena (ver abajo) debido a una alta tasa de errores y permanece en cuarentena durante más de cuatro semanas. En este caso, el servicio se deshabilita automáticamente.

### <a name="errors-and-retries"></a>Errores y reintentos

Si un usuario individual no se agrega, actualiza o elimina en el sistema de destino debido a un error en el sistema de destino, la operación se reintenta en el siguiente ciclo de sincronización. Si el error continúa, los reintentos comienzan a producirse según una frecuencia reducida y disminuyen gradualmente hasta un solo intento al día. Para resolver el error, los administradores deben comprobar la [registros de auditoría](check-status-user-account-provisioning.md) de "custodia de proceso" eventos para determinar la raíz de la causa y realizar la acción apropiada. Algunos errores comunes son:

- Usuarios que no tienen relleno un atributo en el sistema de origen que es necesario en el sistema de destino
- Los usuarios que tienen un valor de atributo en el sistema de origen para el que hay una restricción unique en el sistema de destino y el mismo valor está presentes en otro registro de usuario

Estos errores pueden resolverse mediante el ajuste de los valores de atributo del usuario afectado en el sistema de origen, o por medio del ajuste de las asignaciones de atributos para no provocar conflictos.   

### <a name="quarantine"></a>Cuarentena

Si la mayoría o todas las llamadas realizadas en el sistema de destino de forma coherente de producir un error debido a un error (por ejemplo, las credenciales de administrador no válidas), a continuación, el trabajo de aprovisionamiento entra en un estado de "cuarentena". Este estado se indica en la [informe resumen de aprovisionamiento](check-status-user-account-provisioning.md) y por correo electrónico si se han configurado las notificaciones de correo electrónico en el portal de Azure. 

En cuarentena, la frecuencia de las sincronizaciones incrementales se reduce gradualmente a una vez al día. 

El trabajo de aprovisionamiento se quitará de la cuarentena después de que se ha solucionado todos los errores causantes y se inicia el siguiente ciclo de sincronización. Si el trabajo de aprovisionamiento permanece en cuarentena durante más de cuatro semanas, se deshabilita.


## <a name="how-long-will-it-take-to-provision-users"></a>¿Cuánto tiempo se tarda en aprovisionar usuarios?

El rendimiento depende de si ejecuta el trabajo de aprovisionamiento de una sincronización inicial o una sincronización incremental.

Para **inicial se sincroniza**, el tiempo de trabajo depende de muchos factores, incluido el número de usuarios y grupos en el ámbito de aprovisionamiento y el número total de usuarios y grupos en el sistema de origen. En esta sección se indica más adelante una lista completa de factores que afectan al rendimiento inicial de la sincronización.

En cuanto a las **sincronizaciones incrementales**, el tiempo de trabajo depende del número de cambios detectados en ese ciclo de sincronización. Si hay menos de 5000 cambios de pertenencia de un grupo o usuario, el trabajo puede finalizar dentro de un solo ciclo de sincronización incremental. 

En la siguiente tabla se resumen los tiempos de sincronización de escenarios comunes de aprovisionamiento. En estos escenarios, el sistema de origen es Azure AD y el sistema de destino es una aplicación SaaS. Los tiempos de sincronización se derivan de un análisis estadístico de los trabajos de sincronización para las aplicaciones de SaaS Workplace, ServiceNow, Salesforce y G Suite.


| Configuración de ámbito | Usuarios, grupos y miembros del ámbito | Hora de sincronización inicial | Hora de sincronización incremental |
| -------- | -------- | -------- | -------- |
| Sincronizar solo los usuarios y grupos asignados |  < 1000 |  < 30 minutos | < 30 minutos |
| Sincronizar solo los usuarios y grupos asignados |  1000 - 10.000 | 142 - 708 minutos | < 30 minutos |
| Sincronizar solo los usuarios y grupos asignados |   10.000 - 100.000 | 1170 - 2340 minutos | < 30 minutos |
| Sincronizar todos los usuarios y grupos en Azure AD |  < 1000 | < 30 minutos  | < 30 minutos |
| Sincronizar todos los usuarios y grupos en Azure AD |  1000 - 10.000 | < 30 - 120 minutos | < 30 minutos |
| Sincronizar todos los usuarios y grupos en Azure AD |  10.000 - 100.000  | 713 - 1425 minutos | < 30 minutos |
| Sincronizar todos usuarios en Azure AD|  < 1000  | < 30 minutos | < 30 minutos |
| Sincronizar todos usuarios en Azure AD | 1000 - 10.000  | 43 - 86 minutos | < 30 minutos |


Para la configuración dedicada a **sincronizar únicamente los usuarios y grupos asignados**, puede usar las siguientes fórmulas para determinar aproximadamente las veces que se realiza la **sincronización inicial** mínima y máxima esperada:

    Minimum minutes =  0.01 x [Number of assigned users, groups, and group members]
    Maximum minutes = 0.08 x [Number of assigned users, groups, and group members] 
    
Resumen de los factores que influyen en el tiempo que lleva completar una **sincronización inicial**:

- El número total de usuarios y grupos del ámbito para el aprovisionamiento.

- La cantidad total de usuarios, grupos y miembros del grupo presentes en el sistema de origen (Azure AD).

- Si los usuarios en el ámbito de aprovisionamiento se vinculan con los usuarios existentes en la aplicación de destino, o deben crear por primera vez. Trabajos de sincronización para que todos los usuarios se crean por primera vez tardar aproximadamente *doble de tiempo* como para el que se buscan todos los usuarios a los usuarios existentes de trabajos de sincronización.

- Número de errores en los [registros de auditoría](check-status-user-account-provisioning.md). El rendimiento es también menor si hay muchos errores y si el servicio de aprovisionamiento ha quedado en un estado de "cuarentena".    

- Solicitar los límites de velocidad y otras limitaciones que haya implementado el sistema de destino. Algunos sistemas de destino implementan límites de frecuencia de solicitud y la limitación, que puede afectar al rendimiento durante las operaciones de sincronización de gran tamaño. En estos casos, una aplicación que recibe demasiadas solicitudes demasiado rápido puede ralentizar su velocidad de respuesta o cerrar la conexión. Para mejorar el rendimiento, el conector debe ajustarse; para ello, no debe enviar las solicitudes de la aplicación más rápido de lo que la aplicación puede procesarlas. Los conectores de aprovisionamiento que ha compilado Microsoft hacen este ajuste. 

- La cantidad y el tamaño de los grupos asignados. Sincronizar grupos asignados lleva más tiempo que sincronizar usuarios. Tanto el número como los tamaños de los grupos asignados afectan al rendimiento. Si una aplicación tiene [asignaciones habilitadas para la sincronización de objetos de grupo](customize-application-attributes.md#editing-group-attribute-mappings), las propiedades del grupo, como los nombres y la pertenencia, se sincronizan además de los usuarios. Estas sincronizaciones adicionales tomarán más tiempo que solo la sincronización de objetos de usuario.


## <a name="how-can-i-tell-if-users-are-being-provisioned-properly"></a>¿Cómo puedo saber si los usuarios se aprovisionan correctamente?

Todas las operaciones que se ejecuta el servicio de aprovisionamiento de usuario se registran en Azure AD los registros de auditoría. Esto incluye todas las operaciones realizadas a los sistemas de origen y de destino y los datos de usuario que se leen o escritos durante cada operación de lectura y escritura.

Para obtener información sobre cómo leer los registros de auditoría en el portal de Azure, consulte el [Guía de informes de aprovisionamiento](check-status-user-account-provisioning.md).


## <a name="how-do-i-troubleshoot-issues-with-user-provisioning"></a>¿Cómo resolver problemas con el aprovisionamiento de usuarios?

Para ver instrucciones basadas en escenarios sobre cómo solucionar problemas de aprovisionamiento automático de usuarios, consulte [Problemas al configurar y aprovisionar usuarios en una aplicación](application-provisioning-config-problem.md).


## <a name="what-are-the-best-practices-for-rolling-out-automatic-user-provisioning"></a>¿Cuáles son los procedimientos recomendados para la implementación del aprovisionamiento automático de usuarios?

> [!VIDEO https://www.youtube.com/embed/MAy8s5WSe3A]

Para obtener un ejemplo del plan de implementación detallado para el aprovisionamiento de usuarios saliente a una aplicación, consulte la [guía de implementación de identidad para el aprovisionamiento de usuarios](https://aka.ms/userprovisioningdeploymentplan).

## <a name="more-frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-b2b-users-in-azure-ad"></a>¿Funciona el aprovisionamiento automático de usuarios para aplicaciones SaaS con usuarios de B2B en Azure AD?

Sí, es posible usar el aprovisionamiento de usuarios de servicio para aprovisionar B2B (o invitados) de Azure AD a aplicaciones SaaS de usuario de Azure AD.

Sin embargo, para que los usuarios de B2B iniciar sesión en la aplicación SaaS con Azure AD, la aplicación SaaS debe tener su basado en SAML único inicio de sesión capacidad configurada de forma específica. Para obtener más información sobre cómo configurar aplicaciones de SaaS para admitir inicios de sesión de los usuarios de B2B, consulte [aplicaciones de SaaS configurar para la colaboración B2B]( https://docs.microsoft.com/azure/active-directory/b2b/configure-saas-apps).

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-dynamic-groups-in-azure-ad"></a>¿Funciona el aprovisionamiento automático de usuarios para aplicaciones SaaS con grupos dinámicos en Azure AD?

Sí. Cuando se configura en "Sincronizar solo asignada los usuarios y grupos", puede aprovisionar el servicio de aprovisionamiento de usuario de Azure AD o los usuarios de desaprovisionamiento de una aplicación SaaS en función de si se convierten en miembros de un [grupo dinámico](../users-groups-roles/groups-create-rule.md). Los grupos dinámicos también funcionan con la opción "sincronizar todos los usuarios y grupos".

Sin embargo, el uso de grupos dinámicos puede afectar al rendimiento general del aprovisionamiento de usuarios de un extremo a otro desde Azure AD a las aplicaciones SaaS. Al usar grupos dinámicos, tenga en cuenta estas observaciones y recomendaciones:

- La velocidad con la se aprovisiona o desaprovisiona un usuario de un grupo dinámico en una aplicación SaaS depende de la rapidez con la que el grupo dinámico pueda evaluar los cambios de pertenencia. Para obtener información acerca de cómo comprobar el estado de procesamiento de un grupo dinámico, consulte [Comprobación del estado de procesamiento de una regla de pertenencia](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-create-rule).

- Al usar grupos dinámicos, se deben considerar cuidadosamente las reglas de aprovisionamiento y Desaprovisionamiento de usuarios en mente, como una pérdida de los resultados de la pertenencia en un evento de cancelación del aprovisionamiento.

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-nested-groups-in-azure-ad"></a>¿Funciona el aprovisionamiento automático de usuarios para aplicaciones SaaS con grupos anidados en Azure AD?

No. Cuando se configura en "Sincronizar solo asignada los usuarios y grupos", el servicio de aprovisionamiento de usuario de Azure AD no puede leer ni aprovisionar los usuarios que están en grupos anidados. Solo es capaz de leer y aprovisionamiento de usuarios que son miembros inmediatos del grupo asignado explícitamente.

Esta es una limitación de las "asignaciones basadas en grupos a aplicaciones" que también afecta al inicio de sesión único y se describe en [Uso de un grupo para administrar el acceso a aplicaciones SaaS](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-saasapps ).

Como alternativa, debe asignar explícitamente (o de otro modo [definir el ámbito en](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts)) los grupos que contienen los usuarios que necesiten aprovisionarse.

### <a name="is-provisioning-between-azure-ad-and-a-target-application-using-an-encrypted-channel"></a>¿Se está aprovisionando entre Azure AD y una aplicación de destino mediante un canal cifrado?

Sí. Se usa el cifrado SSL de HTTPS para el destino de servidor. 

## <a name="related-articles"></a>Artículos relacionados

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)
- [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](customize-application-attributes.md)
- [Escritura de expresiones para asignaciones de atributos](functions-for-customizing-application-data.md)
- [Filtros de ámbito para el aprovisionamiento de usuario](define-conditional-rules-for-provisioning-user-accounts.md)
- [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](use-scim-to-provision-users-and-groups.md)
- [Introducción a la API de sincronización de Azure AD](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)
