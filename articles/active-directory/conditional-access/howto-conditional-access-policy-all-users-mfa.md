---
title: 'Acceso condicional: exigir autenticación multifactor para todos los usuarios (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para exigir que los usuarios realicen la autenticación multifactor
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: e1e99556b0a0c5b7559cc03b152a19e78aeb58f6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283051"
---
# <a name="conditional-access-require-mfa-for-all-users"></a>Acceso condicional: Exigir autenticación multifactor para todos los usuarios

Como Alex Weinert, el director de seguridad de identidades de Microsoft, menciona en la entrada de su blog [Your Pa$$word doesn't matter](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984) (Tu contraseña no importa):

> La contraseña no es importante, pero la autenticación multifactor sí. Según nuestros estudios, es prácticamente improbable que una cuenta se comprometa si se usa MFA.

Las instrucciones de este artículo le ayudarán a su organización a crear una directiva de MFA equilibrada para su entorno.

## <a name="user-exclusions"></a>Exclusiones de usuarios

Las directivas de acceso condicional son herramientas eficaces, por lo que se recomienda excluir las siguientes cuentas de la directiva:

* Cuentas de **acceso de emergencia** para **evitar**el bloqueo de cuentas en todo el inquilino. En el caso improbable de que todos los administradores estén bloqueados en el inquilino, la cuenta administrativa de acceso de emergencia se puede usar para iniciar sesión en el inquilino y realizar los pasos para recuperar el acceso.
   * Se puede encontrar más información en el artículo [Administración de cuentas de acceso de emergencia en Azure AD](../users-groups-roles/directory-emergency-access.md).
* **Cuentas de servicio** y **entidades de servicio**, como la cuenta de sincronización de Azure AD Connect. Las cuentas de servicio son cuentas no interactivas que no están asociadas a ningún usuario en particular. Los servicios back-end las usan normalmente para permitir el acceso mediante programación a las aplicaciones, pero también se utilizan para iniciar sesión en los sistemas con fines administrativos. Las cuentas de servicio como estas se deben excluir porque MFA no se puede completar mediante programación. El acceso condicional no bloquea las llamadas realizadas por las entidades de servicio.
   * Si su organización usa estas cuentas en scripts o código, piense en la posibilidad de reemplazarlas por [identidades administradas](../managed-identities-azure-resources/overview.md). Como solución temporal, puede excluir estas cuentas específicas de la directiva de línea de base.

## <a name="application-exclusions"></a>Excepción de la aplicación

Las organizaciones pueden tener muchas aplicaciones en la nube en uso. No todas estas aplicaciones pueden requerir una seguridad semejante. Por ejemplo, las aplicaciones de nóminas y de asistencia pueden requerir MFA, pero es probable que una cafetería no. Los administradores pueden optar por excluir aplicaciones específicas de su directiva.

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Los pasos siguientes le ayudarán a crear una directiva de acceso condicional que exija a todos los usuarios realizar la autenticación multifactor.

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**
   1. En **Excluir**, seleccione **Usuarios y grupos** y, luego, elija las cuentas de acceso de emergencia de la organización. 
   1. Seleccione **Listo**.
1. En **Aplicaciones en la nube o acciones** > **Incluir**, seleccione **Todas las aplicaciones en la nube**.
   1. En **Excluir**, seleccione las aplicaciones que no requieren autenticación multifactor.
1. En **Condiciones** > **Aplicaciones cliente (versión preliminar)** , en **Seleccionar aplicaciones cliente a las que se aplicará la directiva** mantenga todos los valores seleccionados y haga clic en **Hecho**.
1. En **Controles de acceso** > **Conceder**, seleccione **Conceder acceso**, **Requerir autenticación multifactor** y **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Activado**.
1. Seleccione **Crear** para crear la directiva.

### <a name="named-locations"></a>Ubicaciones con nombre

Las organizaciones pueden optar por incorporar ubicaciones de red conocidas, denominadas **ubicaciones con nombre**, a sus directivas de acceso condicional. Estas ubicaciones con nombre pueden incluir redes IPv4 de confianza, como las de una ubicación de oficina principal. Para obtener más información sobre la configuración de ubicaciones con nombre, consulte el artículo [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](location-condition.md)

En la directiva de ejemplo anterior, una organización puede optar por no requerir la autenticación multifactor si tiene acceso a una aplicación en la nube desde su red corporativa. En este caso, se podría agregar la siguiente configuración a la directiva:

1. En **Asignaciones**, seleccione **Condiciones** > **Ubicaciones**.
   1. Configure **Sí**.
   1. Incluya **Cualquier ubicación**.
   1. Excluya **Todas las ubicaciones de confianza**.
   1. Seleccione **Listo**.
1. Seleccione **Listo**.
1. **Guarde** sus cambios en la directiva.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-report-only.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
