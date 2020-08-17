---
title: 'Error en la solicitud de Azure Digital Twins con el estado: 403 (Prohibido)'
description: 'Causas y soluciones de "Service request failed. Estado: 403 (Forbidden) (Error en la solicitud del servicio. Estado: 403 [prohibido]) en Azure Digital Twins.'
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 7/20/2020
ms.openlocfilehash: a971291dd423894e4d04158abe873a7222f9802c
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405618"
---
# <a name="service-request-failed-status-403-forbidden"></a>Error en la solicitud del servicio. Estado: 403 (Prohibido)

En este artículo se describen las causas y los pasos de resolución al recibir un error 403 de las solicitudes de servicio a Azure Digital Twins. 

## <a name="symptoms"></a>Síntomas

Este error puede producirse en muchos tipos de solicitudes de servicio que requieren autenticación. El efecto es que se produce un error en la solicitud de API y se devuelve un estado de error `403 (Forbidden)`.

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

A menudo este error indica que los permisos de control de acceso basado en rol (RBAC) para el servicio no están configurados correctamente. Muchas acciones de Azure Digital Twins requieren el rol de *propietario de Azure Digital Twins (versión preliminar)* **en la instancia que intente administrar**. 

### <a name="cause-2"></a>Causa 2

Si usa una aplicación cliente para comunicarse con Azure Digital Twins, este error puede producirse porque el registro de la aplicación en [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) no tenga permisos configurados para el servicio Azure Digital Twins.

Este registro de la aplicación debe tener configurados los permisos de acceso a las API de Azure Digital Twins. Después, cuando la aplicación cliente se autentique durante el registro de la aplicación, se le concederán los permisos que se hayan configurado en este.

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

La primera solución es comprobar que el usuario de Azure tenga el rol de _**propietario de Azure Digital Twins (versión preliminar)**_ en la instancia que intente administrar. Si no tiene este rol, establézcalo.

Tenga en cuenta que este rol es diferente del...
* rol de *propietario* de toda la suscripción de Azure El rol de *propietario de Azure Digital Twins (versión preliminar)* es un rol dentro de Azure Digital Twins y se limita a esa instancia en concreto.
* rol de *propietario* en Azure Digital Twins. Son dos roles de administración distintos de Azure Digital Twins; *propietario de Azure Digital Twins (versión preliminar)* es el rol que debe usar para la administración durante la versión preliminar.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

#### <a name="fix-issues"></a>Corrección de problemas 

Si no tiene esta asignación de roles, alguien con un rol de propietario en la **suscripción de Azure** debe ejecutar el siguiente comando para proporcionar al usuario de Azure el rol de*propietario de Azure Digital Twins (versión preliminar)* en la **instancia de Azure Digital Twins**. 

Si es propietario de la suscripción, puede ejecutar este comando usted mismo. Si no es así, póngase en contacto con un propietario para que ejecute este comando por usted.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<your-Azure-AD-email>" --role "Azure Digital Twins Owner (Preview)"
```

Para más información sobre este requisito de rol y el proceso de asignación, consulte la [*sección de Configuración de los permisos de acceso del usuario en el artículo*](how-to-set-up-instance-CLI.md#set-up-user-access-permissions) de *Procedimiento de configuración de una instancia y de la autenticación (mediante la CLI o el portal)* .

Si ya tiene esta asignación de roles y el problema 403 persiste, pase a la siguiente solución.

### <a name="solution-2"></a>Solución 2

La segunda solución consiste en comprobar que el registro de la aplicación en Azure AD tiene los permisos configurados para el servicio Azure Digital Twins. Si no es el caso, establézcalos.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

[!INCLUDE [digital-twins-setup-verify-app-registration-1.md](../../includes/digital-twins-setup-verify-app-registration-1.md)]

En primer lugar, compruebe que la configuración de permisos de Azure Digital Twins se estableció correctamente en el registro. Para ello, seleccione *Manifiesto* en la barra de menús para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque estos campos en `requiredResourceAccess`. Los valores deben coincidir con los de la siguiente captura de pantalla:

[!INCLUDE [digital-twins-setup-verify-app-registration-2.md](../../includes/digital-twins-setup-verify-app-registration-2.md)]

#### <a name="fix-issues"></a>Corrección de problemas

Si algo no se parece a lo que se describe aquí, siga las instrucciones sobre cómo configurar el registro de una aplicación en la [*sección Configuración de permisos de acceso para aplicaciones cliente*](how-to-set-up-instance-cli.md#set-up-access-permissions-for-client-applications) del artículo de *Procedimiento de configuración de una instancia y de la autenticación (mediante la CLI o el portal)* .

## <a name="next-steps"></a>Pasos siguientes

Lea los pasos de configuración para crear y autenticar una nueva instancia de Azure Digital Twins:
* [*Procedimiento: Configuración de una instancia y de la autenticación (CLI)* ](how-to-set-up-instance-cli.md)

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [*Conceptos: Seguridad para las soluciones de Azure Digital Twins*](concepts-security.md)