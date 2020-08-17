---
title: Información sobre la delegación de roles de administración en Azure Active Directory | Microsoft Docs
description: Modelos de delegación, ejemplos y seguridad de los roles en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: conceptual
ms.date: 04/29/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: c26951d5ab7552dc6c63288ce92cdb8134bd58ed
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498232"
---
# <a name="delegate-administration-in-azure-active-directory"></a>Delegación de administración en Azure Active Directory

El crecimiento de una organización conlleva más complejidad. Una respuesta común a esta complejidad es reducir la carga de trabajo de administración de acceso con los roles de administrador de Azure Active Directory (AD). Puede asignar los privilegios mínimos a los usuarios para acceder a sus aplicaciones y realizar sus tareas. Incluso si no asigna el rol de administrador global a cada propietario de aplicación, está otorgando responsabilidades de administración de aplicaciones a los administradores globales existentes. Hay muchas razones a favor de que una organización cambie a un modelo de administración más descentralizado. Este artículo puede ayudarle a planear la delegación en su organización.

<!--What about reporting? Who has which role and how do I audit?-->

## <a name="centralized-versus-delegated-permissions"></a>Centralización y permisos delegados

A medida que una organización crece, puede resultar difícil controlar qué usuarios tienen roles de administrador específicos. Si un empleado tiene derechos de administrador que no debería tener, la organización puede ser susceptible a las infracciones de seguridad. Por lo general, el número de administradores que se admiten y el nivel de pormenorización de sus permisos dependen del tamaño y complejidad de la implementación.

* En las implementaciones pequeñas o de prueba de concepto, uno o unos pocos administradores lo hacen todo, no hay delegaciones. En este caso, cree cada administrador con el rol de administrador global.
* En implementaciones más grandes con más máquinas, aplicaciones y escritorios, se necesita un nivel de delegación mayor. Varios administradores pueden tener responsabilidades funcionales más específicas (roles). Por ejemplo, algunos podrían ser los administradores de Privileged Identity, mientras que otros serían los administradores de aplicación. Además, un administrador puede administrar únicamente determinados grupos de objetos, como dispositivos.
* Las implementaciones más grandes podrían requerir incluso permisos más pormenorizados, además, estos se podrían otorgar posiblemente a administradores con roles poco convencionales o híbridos.

En el portal de Azure AD, puede [ver todos los miembros de cualquier rol](directory-manage-roles-portal.md), esto puede ayudarle a comprobar rápidamente la implementación y delegar permisos.

Si lo que le interesa es delegar el acceso en los recursos de Azure en lugar del acceso administrativo de Azure AD, consulte este artículo sobre la [asignación de un rol de Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="delegation-planning"></a>Planeamiento de delegaciones

Esta tarea implica desarrollar un modelo de delegación que se adapte a sus necesidades. Desarrollar un modelo de delegación es un proceso de diseño iterativo y le sugerimos que siga estos pasos:

* Definición de los roles que necesita
* Delegación de la administración de aplicaciones
* Concesión de la capacidad para registrar aplicaciones
* Delegación de la propiedad de las aplicaciones
* Desarrollo de un plan de seguridad
* Establecimiento de cuentas de emergencia
* Protección de los roles de administrador
* Elevación de privilegios de forma temporal

## <a name="define-roles"></a>Definición de roles

Determine las tareas de Active Directory que llevan a cabo los administradores y cómo dichas tareas se corresponden con roles. Puede [ver descripciones detalladas de los roles](directory-manage-roles-portal.md) en Azure Portal.

Cada tarea se debe evaluar de acuerdo a su frecuencia, importancia y dificultad. Estos son criterios esenciales en la definición de una tarea, porque controlan si un permiso debe o no delegarse:

* Las tareas que realiza de forma rutinaria, que tienen un riesgo limitado y cuya conclusión es trivial son idóneas para la delegación.
* Las tareas que no se realizan casi nunca pero tienen una gran repercusión en toda la organización y requieren niveles altos de habilidad deben pensarse con mucho cuidado antes de delegarlas. En su lugar, puede [elevar temporalmente una cuenta al rol necesario](../active-directory-privileged-identity-management-configure.md) o volver a asignar la tarea.

## <a name="delegate-app-administration"></a>Delegación de la administración de aplicaciones

La proliferación de aplicaciones dentro de su organización puede forzar el modelo de delegación de la misma. Si este modelo coloca la carga de administración de acceso a aplicaciones en el administrador global, es probable que vaya sobrecargándose a medida que pasa el tiempo. Si ha concedido a algunas personas el rol de administrador global para cosas como la configuración de aplicaciones empresariales, ahora puede descargar a estos usuarios concediéndoles los siguientes roles con menos privilegios. Con ello, ayuda a mejorar su posición de seguridad y reduce la posibilidad de errores indeseables. Los roles de administración de aplicación con más privilegios son:

* El rol de **administrador de aplicación**, que concede la capacidad para administrar todas las aplicaciones del directorio, incluidos los registros, configuraciones de inicio de sesión único, asignaciones y configuración de licencias de usuario y de grupo, configuración del proxy de aplicación y consentimiento. No concede la capacidad de administrar el acceso condicional.
* El rol de **administrador de la aplicación en la nube**, que concede todas las capacidades del administrador de aplicación, salvo el acceso a la configuración del proxy de aplicación (porque no tiene ningún permiso en el entorno local).

## <a name="delegate-app-registration"></a>Delegación del registro de aplicaciones

De forma predeterminada, todos los usuarios pueden crear registros de aplicación. Para conceder de forma selectiva la capacidad de crear registros de aplicación:

* Establezca **Los usuarios pueden registrar aplicaciones** en No desde la **Configuración de usuario**.
* Asigne al usuario el rol de desarrollador de aplicaciones.

Para conceder de forma selectiva la capacidad de dar su consentimiento para permitir que una aplicación acceda a los datos:

* Establezca **Los usuarios pueden permitir que las aplicaciones accedan a los datos de la compañía en su nombre** en No desde la **Configuración de usuario**.
* Asigne al usuario el rol de desarrollador de aplicaciones.

Cuando un desarrollador de aplicaciones crea un nuevo registro de aplicaciones, se agrega automáticamente como primer propietario.

## <a name="delegate-app-ownership"></a>Delegación de la propiedad de las aplicaciones

Para detallar aún más la delegación de acceso a las aplicaciones, puede asignar propiedad a las aplicaciones empresariales individuales. Esto complementa la compatibilidad ya existente para la asignación de propietarios de registro de aplicaciones. La propiedad se asigna en forma de aplicación por empresa en la hoja aplicaciones empresariales. La ventaja es que los propietarios pueden administrar solo las aplicaciones empresariales de las que son propietarios. Por ejemplo, puede asignar un propietario a la aplicación de Salesforce, que puede administrar el acceso y la configuración solamente de Salesforce y de ninguna otra aplicación. Una aplicación empresarial puede tener muchos propietarios, y un usuario puede ser el propietario de muchas aplicaciones empresariales. Hay dos roles de propietario de aplicación:

* El rol de **propietario de aplicación empresarial** que concede la capacidad para administrar las aplicaciones empresariales que son propiedad del usuario, incluida la configuración de inicio de sesión único, las asignaciones de usuario y de grupo y la incorporación de propietarios adicionales. No concede la capacidad para administrar la configuración del proxy de aplicación o de acceso condicional.
* El rol de **propietario del registro de aplicación** que concede la capacidad de administrar los registros de aplicación para la aplicación que es propiedad del usuario, incluido el manifiesto de aplicación y la incorporación de propietarios adicionales.

## <a name="develop-a-security-plan"></a>Desarrollo de un plan de seguridad

Azure AD proporciona una extensa guía para planear y ejecutar un plan de seguridad en los roles de administrador de Azure AD, [protegiendo el acceso con privilegios para las implementaciones híbridas y en la nube](directory-admin-roles-secure.md).

## <a name="establish-emergency-accounts"></a>Establecimiento de cuentas de emergencia

Para mantener el acceso a su almacén de administración de identidades cuando se produce el problema, prepare cuentas de acceso de emergencia de acuerdo a la [Creación de cuentas administrativas de acceso de emergencia](directory-emergency-access.md).

## <a name="secure-your-administrator-roles"></a>Protección de los roles de administrador

Si un atacante toma el control de cuentas con privilegios, puede hacer un daño enorme, por lo que en primer lugar tiene que proteger estas cuentas utilizando la [directiva de acceso de base de referencia](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/22/baseline-security-policy-for-azure-ad-admin-accounts-in-public-preview/) que está disponible de forma predeterminada para todas las organizaciones de Azure AD (en versión preliminar pública). La directiva exige la autenticación multifactor en cuentas con privilegios de Azure AD. Los siguientes roles de Azure AD están cubiertos por la directiva de base de referencia de Azure AD:

* Administrador global
* Administrador de SharePoint
* Administrador de Exchange
* Administrador de acceso condicional
* Administrador de seguridad

## <a name="elevate-privilege-temporarily"></a>Elevación temporal de privilegios

Para la mayoría de las actividades diarias, no todos los usuarios necesitan derechos de administrador global y no todos ellos deben tener asignado de forma permanente el rol de administrador global. Cuando un usuario necesita los permisos de un administrador global, debe activar la asignación de roles en Azure AD [Privileged Identity Management](../active-directory-privileged-identity-management-configure.md) en su propia cuenta o en una cuenta administrativa alternativa.

## <a name="next-steps"></a>Pasos siguientes

Para una referencia a las descripciones de rol de Azure AD, consulte [Asignación de roles de administrador en Azure AD](directory-assign-admin-roles.md)
