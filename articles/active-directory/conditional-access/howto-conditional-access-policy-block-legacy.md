---
title: 'Acceso condicional: Bloquear la autenticación heredada (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para bloquear los protocolos de autenticación heredados.
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
ms.openlocfilehash: 15e3c5ba72ff7b97e03cd7aa2b68eeffb2aa7882
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283017"
---
# <a name="conditional-access-block-legacy-authentication"></a>Acceso condicional: Bloquear la autenticación heredada

Debido al aumento del riesgo asociado a los protocolos de autenticación heredados, Microsoft recomienda que las organizaciones bloqueen las solicitudes de autenticación que usan estos protocolos y que exijan la autenticación moderna.

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Los pasos siguientes le ayudarán a crear una directiva de acceso condicional para bloquear las solicitudes de autenticación heredadas. Esta directiva se coloca en [modo de solo informe](howto-conditional-access-report-only.md) para comenzar, de modo que los administradores puedan establecer el impacto que tendrán en los usuarios existentes. Cuando los administradores se sientan cómodos con que la directiva se aplique según lo previsto, pueden cambiar a **Activado** o ensayar la implementación agregando grupos específicos y excluyendo otros.

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
   1. En **Excluir**, seleccione **Usuarios y grupos** y elija las cuentas que deben mantener la capacidad para usar la autenticación heredada. Excluya al menos una cuenta para evitar que se le bloquee. Si no se excluye ninguna cuenta, no podrá crear esta directiva.
   1. Seleccione **Listo**.
1. En **Aplicaciones o acciones en la nube**, seleccione **Todas las aplicaciones en la nube**.
   1. Seleccione **Listo**.
1. En **Condiciones** > **Aplicaciones cliente (versión preliminar)** , en **Seleccionar aplicaciones cliente a las que se aplicará la directiva**:
   1. Active solo las casillas **Aplicaciones móviles y clientes de escritorio** > **Otros clientes**.
   1. Seleccione **Listo**.
1. En **Controles de acceso** > **Conceder**, seleccione **Bloquear acceso**.
   1. Elija **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Solo informe**.
1. Seleccione **Crear** para crear la directiva.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-report-only.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)

[Cómo configurar una aplicación o dispositivo multifunción para enviar correos electrónicos mediante Office 365 y Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-office-3)
