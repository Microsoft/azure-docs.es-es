---
title: 'Acceso condicional: información de seguridad combinada (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para el registro de la información de seguridad.
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
ms.openlocfilehash: da68e21aa279ea2503a21ce35eee52f8e49d1434
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049101"
---
# <a name="conditional-access-securing-security-info-registration"></a>Acceso condicional: Protección del registro de información de seguridad

Proteger cuándo y cómo se registran los usuarios para Azure Multi-Factor Authentication y el restablecimiento de contraseñas de autoservicio ya es posible con las acciones del usuario en la directiva de acceso condicional. Esta característica de versión preliminar está disponible para organizaciones que han habilitado la [vista previa del registro combinado](../authentication/concept-registration-mfa-sspr-combined.md). Esta funcionalidad se puede habilitar en las organizaciones que quieren usar condiciones como la ubicación de red de confianza para restringir el acceso al registro en Azure Multi-Factor Authentication y autoservicio de restablecimiento de contraseña (SSPR). Para obtener más información sobre las condiciones que se pueden usar, consulte el artículo [Acceso condicional: Condiciones](concept-conditional-access-conditions.md).

## <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Crear una directiva para requerir el registro desde una ubicación de confianza

La siguiente directiva se aplica a todos los usuarios seleccionados, que intentan registrarse con la experiencia de registro combinado, y bloquea su acceso a menos que se conecten desde una ubicación marcada como red de confianza.

1. En **Azure Portal**, vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. En Nombre, escriba un nombre para la directiva. Por ejemplo, **Registro de información de seguridad combinada en redes de confianza**.
1. En **Asignaciones**, seleccione **Usuarios y grupos** y seleccione los usuarios y grupos a los que quiera aplicar esta directiva.

   > [!WARNING]
   > Los usuarios deben estar habilitados para el [registro combinado](../authentication/howto-registration-mfa-sspr-combined.md).

1. En **Aplicaciones en la nube o acciones**, seleccione **Acciones del usuario** y active la casilla **Registro de la información de seguridad**.
1. En **Condiciones** > **Ubicaciones**.
   1. Configure **Sí**.
   1. Incluya **Cualquier ubicación**.
   1. Excluya **Todas las ubicaciones de confianza**.
   1. Seleccione **Listo** en la hoja Ubicaciones.
   1. Seleccione **Listo** en la hoja Condiciones.
1. En **Condiciones** > **Aplicaciones cliente (versión preliminar)** , establezca **Configurar** en **Sí**, y seleccione **Listo**.
1. En **Controles de acceso** > **Conceder**.
   1. Seleccione **Block access** (Bloquear acceso).
   1. Después, haga clic en **Seleccionar**.
1. Establezca **Habilitar directiva** en **Activado**.
1. Después, seleccione **Guardar**.

En el paso 6 de esta directiva, las organizaciones pueden tomar elecciones. La directiva anterior requiere el registro de una ubicación de red de confianza. Las organizaciones pueden optar por emplear cualquier condición disponible en lugar de **Ubicaciones**. Recuerde que esta directiva es una directiva de bloqueo, por lo que se bloquea todo lo que se incluye, y se permite todo que no coincida con la inclusión. 

Algunos pueden optar por usar el estado del dispositivo en lugar de la ubicación en el paso 6 anterior:

6. En **Condiciones** > **Estado del dispositivo (versión preliminar)** .
   1. Configure **Sí**.
   1. Incluya **Todos los estados de dispositivo**.
   1. Excluya **Unido a Azure AD híbrido de dispositivo** o **Dispositivo marcado como compatible**.
   1. Seleccione **Listo** en la hoja Ubicaciones.
   1. Seleccione **Listo** en la hoja Condiciones.

> [!WARNING]
> Si usa el estado del dispositivo como condición en la directiva, esto puede afectar a los usuarios invitados en el directorio. El [Modo de solo informe](concept-conditional-access-report-only.md) puede ayudar a determinar el impacto de las decisiones de la directiva.
> Tenga en cuenta que el modo de solo informe no es aplicable a las directivas de CA con el ámbito "acciones del usuario".

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
