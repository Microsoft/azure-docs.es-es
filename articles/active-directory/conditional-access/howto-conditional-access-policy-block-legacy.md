---
title: 'Acceso condicional: Bloquear la autenticación heredada (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para bloquear los protocolos de autenticación heredados.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 08/07/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c10f0ab4ff7fd8461ef81da67e50f7e5e174104
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068633"
---
# <a name="conditional-access-block-legacy-authentication"></a>Acceso condicional: Bloquear la autenticación heredada

Debido al aumento del riesgo asociado a los protocolos de autenticación heredados, Microsoft recomienda que las organizaciones bloqueen las solicitudes de autenticación que usan estos protocolos y que exijan la autenticación moderna.

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Los pasos siguientes le ayudarán a crear una directiva de acceso condicional para bloquear las solicitudes de autenticación heredadas. Esta directiva se coloca en [modo de solo informe](howto-conditional-access-insights-reporting.md) para comenzar, de modo que los administradores puedan establecer el impacto que tendrán en los usuarios existentes. Cuando los administradores se sientan cómodos con que la directiva se aplique según lo previsto, pueden cambiar a **Activado** o ensayar la implementación agregando grupos específicos y excluyendo otros.

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
1. En **Condiciones** > **Aplicaciones cliente**, establezca **Configurar** en **Sí**.
   1. Active solo las casillas **Clientes de Exchange ActiveSync** y **Otros clientes**. Para implementar la directiva de acceso condicional de Exchange ActiveSync en Azure, el usuario también debe ser administrador global.
   1. Seleccione **Listo**.
1. En **Controles de acceso** > **Conceder**, seleccione **Bloquear acceso**.
   1. Elija **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Solo informe**.
1. Seleccione **Crear** para crear la directiva.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)

[Cómo configurar una aplicación o dispositivo multifunción para enviar correos electrónicos mediante Office 365 y Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-office-3)
