---
title: Guía de referencia de operaciones de gobernanza de Azure Active Directory
description: Esta guía de referencia de operaciones describe las comprobaciones y las acciones que debe realizar para proteger la administración de la gobernanza.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: f420f66e1db6efc6a0aa43cb88f26687839f0d1a
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321521"
---
# <a name="azure-active-directory-governance-operations-reference-guide"></a>Guía de referencia de operaciones de gobernanza de Azure Active Directory

En esta sección de la [guía de referencia de operaciones de Azure AD](active-directory-ops-guide-intro.md) se describen las comprobaciones y las acciones que debe realizar para evaluar y confirmar el acceso concedido a las identidades sin privilegios y con privilegios, así como auditar y controlar los cambios en el entorno.

> [!NOTE]
> Estas recomendaciones están actualizadas hasta la fecha de publicación, pero pueden cambiar con el tiempo. Las organizaciones deben evaluar continuamente sus prácticas de gobernanza a medida que los productos y servicios de Microsoft evolucionen con el tiempo.

## <a name="key-operational-processes"></a>Procesos operativos clave

### <a name="assign-owners-to-key-tasks"></a>Asignación de propietarios a las tareas clave

La administración de Azure Active Directory requiere la ejecución continua de tareas y procesos operativos clave que pueden no formar parte de un proyecto de lanzamiento. Aun así, es importante que configure estas tareas con miras a optimizar su entorno. Entre las tareas clave y sus propietarios recomendados se incluyen:

| Tarea | Propietario |
| :- | :- |
| Archivado de registros de auditoría de Azure AD en el sistema SIEM | Equipo de operaciones de InfoSec |
| Detección de aplicaciones que se administran fuera del cumplimiento | Equipo de operaciones IAM |
| Revisión periódica del acceso a las aplicaciones | Equipo de arquitectura de InfoSec |
| Revisión periódica del acceso a identidades externas | Equipo de arquitectura de InfoSec |
| Revisión periódica de quién tiene roles con privilegios | Equipo de arquitectura de InfoSec |
| Definición de puertas de seguridad para activar los roles con privilegios | Equipo de arquitectura de InfoSec |
| Revisión periódica de las concesiones de consentimiento | Equipo de arquitectura de InfoSec |
| Diseño de catálogos y paquetes de acceso para aplicaciones y recursos basados en empleados de la organización | Propietarios de la aplicación |
| Definición de directivas de seguridad para asignar usuarios a paquetes de acceso | Equipo de InfoSec + propietarios de aplicaciones |
| Si las directivas incluyen flujos de trabajo de aprobación, revisión periódica de las aprobaciones de flujo de trabajo | Propietarios de la aplicación |
| Revisión de excepciones en las directivas de seguridad, como las directivas de acceso condicional, mediante las revisiones de acceso | Equipo de operaciones de InfoSec |

A medida que revise la lista, es posible que tenga que asignar un propietario a las tareas que no tienen uno o ajustar la propiedad de aquellas tareas con propietarios que no coincidan con las recomendaciones anteriores.

#### <a name="owner-recommended-reading"></a>Lecturas recomendadas para propietarios

- [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md)
- [Gobernanza en Azure](../../governance/index.yml)

### <a name="configuration-changes-testing"></a>Pruebas de los cambios de configuración

Hay cambios que requieren consideraciones especiales a la hora de realizar pruebas, desde técnicas sencillas como la implementación de un subconjunto de usuarios de destino para implementar un cambio en un inquilino de prueba paralelo. Si no ha implementado una estrategia de pruebas, debe definir un enfoque de pruebas basado en las directrices de la tabla siguiente:

| Escenario| Recomendación |
|-|-|
|Cambio del tipo de autenticación de federado a PHS/PTA o viceversa| Use la [implementación por fases](../hybrid/how-to-connect-staged-rollout.md) para probar la repercusión del cambio del tipo de autenticación.|
|Implementación de una nueva directiva de acceso condicional o una directiva de protección de identidad|Cree una nueva directiva de acceso condicional y asígnela a usuarios de prueba.|
|Incorporación de un entorno de prueba de una aplicación|Agregue la aplicación a un entorno de producción, ocúltela en el panel Mis aplicaciones y asígnela a usuarios de prueba durante la fase de control de calidad.|
|Cambio de las reglas de sincronización|Realice los cambios en una instancia de Azure AD Connect de prueba con la misma configuración que está actualmente en producción, también conocido como modo de ensayo, y analice los resultados de CSExport. Si está satisfecho, pase a producción cuando esté listo.|
|Cambio de marca|Pruebe en un inquilino de prueba independiente.|
|Implementación de una nueva característica|Si la característica admite la implementación en un conjunto de usuarios de destino, asigne unos usuarios piloto y prosiga con la implementación. Por ejemplo, el autoservicio de restablecimiento de contraseña y la autenticación multifactor pueden dirigirse a usuarios o grupos específicos.|
|Traslado de una aplicación de un proveedor de identidades local (IdP), como Active Directory, a Azure AD|Si la aplicación admite varias configuraciones de IdP, por ejemplo, Salesforce, configure y pruebe Azure AD durante una ventana de cambio (en caso de que la aplicación introduzca la página de HRD). Si la aplicación no admite varios IdP, programe las pruebas durante una ventana de control de cambios y el tiempo de inactividad del programa.|
|Actualización de reglas de grupo dinámico|Cree un grupo dinámico paralelo con la nueva regla. Compare con el resultado calculado, por ejemplo, ejecute PowerShell con la misma condición.<br>Si la prueba es satisfactoria, intercambie los lugares donde se usó el grupo anterior (si es factible).|
|Migración de licencias de productos|Consulte [Cambio de la licencia de un solo usuario de un grupo con licencia en Azure Active Directory](../users-groups-roles/licensing-groups-change-licenses.md).|
|Cambio de reglas de AD FS como autorización, emisión, MFA|Use las notificaciones de grupo para dirigirse a un subconjunto de usuarios.|
|Cambio de la experiencia de autenticación de AD FS o cambios similares en toda la granja de servidores|Cree una granja de servidores paralela con el mismo nombre de host, implemente cambios de configuración y realice pruebas en clientes mediante el archivo HOSTS, las reglas de enrutamiento de NLB o un enrutamiento similar.<br>Si la plataforma de destino no admite archivos HOSTS (por ejemplo, dispositivos móviles), controle el cambio.|

## <a name="access-reviews"></a>Revisiones de acceso

### <a name="access-reviews-to-applications"></a>Revisiones de acceso a las aplicaciones

Con el tiempo, los usuarios pueden acumular acceso a recursos a medida que van rotando entre distintos equipos y puestos. Es importante que los propietarios de los recursos revisen el acceso a las aplicaciones de forma regular y quiten los privilegios que ya no se necesiten durante todo el ciclo de vida de los usuarios. Las [revisiones de acceso](../governance/access-reviews-overview.md) de Azure AD permiten a las organizaciones administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. Los propietarios de los recursos deben revisar el acceso de los usuarios de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. Idealmente, debería considerar el uso de revisiones de acceso de Azure AD para esta tarea.

![Página de inicio de revisiones de acceso](./media/active-directory-ops-guide/active-directory-ops-img15.png)

> [!NOTE]
> Cada usuario que interactúa con las revisiones de acceso debe tener una licencia de pago de Azure AD Premium P2.

### <a name="access-reviews-to-external-identities"></a>Revisiones de acceso a identidades externas

Es fundamental mantener el acceso a las identidades externas restringido solo a los recursos necesarios, durante el tiempo necesario. Establezca un proceso de revisión de acceso automatizado periódico para todas las identidades externas y el acceso a las aplicaciones mediante [revisiones de acceso](../governance/access-reviews-overview.md) de Azure AD. Si ya existe un proceso local, considere la posibilidad de usar revisiones de acceso de Azure AD. Una vez que una aplicación se retire o se deje de usar, quite todas las identidades externas que tuvieran acceso a la aplicación.

> [!NOTE]
> Cada usuario que interactúa con las revisiones de acceso debe tener una licencia de pago de Azure AD Premium P2.

## <a name="privileged-account-management"></a>Administración de cuentas con privilegios

### <a name="privileged-account-usage"></a>Uso de cuenta con privilegios

Los hackers a menudo tienen como objetivo las cuentas de administrador y otros elementos con acceso privilegiado para obtener acceso rápido a datos confidenciales y a sistemas. Dado que con el tiempo tienden a acumularse los usuarios con roles con privilegios, es importante revisar y administrar el acceso de administrador de forma periódica y proporcionar acceso con privilegios cuando sea necesario a Azure AD y los recursos de Azure.

Si no hay ningún proceso en su organización para administrar cuentas con privilegios o si actualmente tiene administradores que usan sus cuentas de usuario habituales para administrar servicios y recursos, debe empezar a usar cuentas independientes, por ejemplo, una para las actividades habituales del día a día y otra para el acceso privilegiado configurada con MFA. Mejor aún, si su organización tiene una suscripción Azure AD Premium P2, debe implementar inmediatamente [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md#license-requirements) (PIM). En el mismo token, también debe revisar esas cuentas con privilegios y [asignar roles con menos privilegios](../users-groups-roles/directory-admin-roles-secure.md), si procede.

Otro aspecto de la administración de cuentas con privilegios que se debe implementar es la definición de [revisiones de acceso](../governance/access-reviews-overview.md) para esas cuentas, ya sea de forma manual o [automatizada a través de PIM](../privileged-identity-management/pim-how-to-perform-security-review.md).

#### <a name="privileged-account-management-recommended-reading"></a>Lectura recomendada para la administración de cuentas con privilegios

- [Roles en Privileged Identity Management de Azure AD](../privileged-identity-management/pim-roles.md)

### <a name="emergency-access-accounts"></a>Cuentas de acceso de emergencia

Las organizaciones deben crear [cuentas de emergencia](../users-groups-roles/directory-emergency-access.md) a fin de prepararse para administrar Azure AD en casos de interrupciones de autenticación como los siguientes:

- Interrupción de los componentes de las infraestructuras de autenticación (AD FS, AD local, servicio MFA)
- Rotación del personal administrativo

Para evitar el bloqueo accidental del inquilino por no poder iniciar sesión o activar una cuenta de usuario individual existente como administrador, debe crear dos o más cuentas de emergencia y asegurarse de que estén implementadas y alineadas con los [procedimientos recomendados de Microsoft](../users-groups-roles/directory-admin-roles-secure.md) y los [procedimientos de emergencia](../users-groups-roles/directory-admin-roles-secure.md#break-glass-what-to-do-in-an-emergency).

### <a name="privileged-access-to-azure-ea-portal"></a>Acceso con privilegios al portal del Contrato Enterprise de Azure

El [portal de Contrato Enterprise de Azure (Azure EA)](https://azure.microsoft.com/blog/create-enterprise-subscription-experience-in-azure-portal-public-preview/) le permite crear suscripciones de Azure en el marco de un Contrato Enterprise, que es un rol importante dentro de la empresa. Es habitual comenzar la creación de este portal antes incluso de que Azure AD esté vigente, por lo que es necesario usar identidades de Azure AD para bloquearlo, quitar cuentas personales del portal, asegurarse de que está implantada la delegación habitual y mitigar el riesgo de bloqueo.

En aras de la claridad, si el nivel de autorización del portal de EA está configurado actualmente en "modo mixto", debe eliminar cualquier [cuenta Microsoft](https://support.skype.com/en/faq/FA12059/what-is-a-microsoft-account) de todos los accesos con privilegios del portal de EA y configurar el portal de EA para usar solo cuentas de Azure AD. Si los roles delegados del portal de EA no están configurados, también debe buscar e implementar roles delegados para departamentos y cuentas.

#### <a name="privileged-access-recommended-reading"></a>Lectura recomendada para el acceso con privilegios

- [Permisos de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md)

## <a name="entitlement-management"></a>Administración de derechos

La [administración de derechos (EM)](../governance/entitlement-management-overview.md) permite a los propietarios de aplicaciones agrupar recursos y asignarlos a roles específicos de la organización (tanto internos como externos). Además, posibilita la delegación y el registro de autoservicio a los propietarios empresariales, a la vez que mantiene las directivas de gobernanza para conceder acceso, establecer duraciones de acceso y permitir flujos de trabajo de aprobación. 

> [!NOTE]
> La administración de derechos de Azure AD requiere licencias de Azure AD Premium P2.

## <a name="summary"></a>Resumen

Una gobernanza de identidad segura está conformada por ocho aspectos. Esta lista le ayudará a identificar las acciones que debe realizar para evaluar y confirmar el acceso concedido a las identidades sin privilegios y con privilegios, así como auditar y controlar los cambios en el entorno.

- Asignar propietarios a las tareas principales.
- Implementar una estrategia de pruebas.
- Usar las revisiones de acceso de Azure AD para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles.
- Establecer un proceso de revisión de acceso automatizado periódico para todos los tipos de identidades externas y acceso a las aplicaciones.
- Establecer un proceso de revisión de acceso para revisar y administrar el acceso de administrador de forma periódica y proporcionar acceso con privilegios cuando es necesario a Azure AD y los recursos de Azure.
- Aprovisionar cuentas de emergencia para que estén preparadas para administrar Azure AD ante interrupciones inesperadas.
- Bloquear el acceso al portal de Azure EA.
- Implementar la administración de derechos para proporcionar acceso controlado a una colección de recursos.

## <a name="next-steps"></a>Pasos siguientes

Comience a trabajar con las [comprobaciones y las acciones operativas de Azure AD](active-directory-ops-guide-ops.md).