---
title: 'Acceso condicional: Bloqueo del acceso por ubicación (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para bloquear el acceso a los recursos por ubicación de IP.
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
ms.openlocfilehash: a531692264a768e4f6cb8e6475807789df049d03
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049153"
---
# <a name="conditional-access-block-access-by-location"></a>Acceso condicional: Bloqueo del acceso por ubicación

Gracias a la condición de ubicación del acceso condicional, podrá controlar el acceso a las aplicaciones en la nube en función de la ubicación de red de un usuario. La condición de ubicación se usa normalmente para bloquear el acceso desde países o regiones de los que la organización sabe que no debe provenir el tráfico.

## <a name="define-locations"></a>Definición de ubicaciones

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Ubicaciones con nombre**.
1. Elija **Nueva ubicación**.
1. Asigne un nombre a la ubicación.
1. Elija **Intervalos IP** si conoce los intervalos de direcciones IPv4 específicos accesibles externamente que constituyen esa ubicación, o seleccione **Países o regiones**.
   1. Proporcione los valores de **Intervalos IP** o seleccione el valor de **Países o regiones** de la ubicación especificada.
      * Si elige Países o regiones, puede optar por incluir áreas desconocidas.
1. Elija **Guardar**.

Para más información sobre la condición de ubicación en el acceso condicional, consulte el artículo [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](location-condition.md)

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
1. En **Aplicaciones en la nube o acciones** > **Incluir**, seleccione **Todas las aplicaciones en la nube**.
1. En **Condiciones** > **Ubicación**:
   1. Establezca **Configurar** en **Sí**.
   1. En **Incluir**, seleccione **Ubicaciones seleccionadas**.
   1. Seleccione la ubicación bloqueada que ha creado para su organización.
   1. Haga clic en **Seleccionar**.
1. En **Controles de acceso** > seleccione **Bloquear acceso** y seleccione **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Activado**.
1. Seleccione **Crear** para crear una directiva de acceso condicional.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
