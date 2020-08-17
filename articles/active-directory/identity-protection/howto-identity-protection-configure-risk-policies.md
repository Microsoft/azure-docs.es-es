---
title: 'Directivas de riesgo: Azure Active Directory Identity Protection'
description: Habilitación y configuración de las directivas de riesgo en Azure Active Directory Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 54a6c455f3702433398035a2eb6398c39b609ce9
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337456"
---
# <a name="how-to-configure-and-enable-risk-policies"></a>Instrucciones: Configuración y habilitación de directivas de riesgo

Como vimos en el artículo anterior, [Directivas de Identity Protection](concept-identity-protection-policies.md), tenemos dos directivas de riesgo que podemos habilitar en nuestro directorio. 

- Directiva de riesgo de inicio de sesión
- Directiva de riesgo de usuario

![Página de información general de seguridad para habilitar las directivas de riesgo de usuario e inicio de sesión](./media/howto-identity-protection-configure-risk-policies/identity-protection-security-overview.png)

Ambas directivas se ocupan de automatizar la respuesta a las detecciones de riesgo en su entorno y permitir que los usuarios corrijan por sí mismos los posibles riesgos cuando estos se detecten. 

> [!VIDEO https://www.youtube.com/embed/zEsbbik-BTE]

## <a name="prerequisites"></a>Prerrequisitos 

Si su organización desea permitir que los usuarios corrijan por sí mismos los riesgos cuando se detecten, los usuarios deben estar registrados para el autoservicio de restablecimiento de contraseña y Azure Multi-Factor Authentication. Se recomienda [habilitar la experiencia de registro de información de seguridad combinada](../authentication/howto-registration-mfa-sspr-combined.md) para obtener la mejor experiencia. Al permitir a los usuarios que se ocupen por sí mismos de solucionar los problemas, se les otorga un estado productivo en el que no es necesaria la intervención del administrador. Los administradores todavía pueden ver estos eventos e investigarlos después del hecho. 

## <a name="choosing-acceptable-risk-levels"></a>Elección de niveles de riesgo aceptables

Las organizaciones deben decidir el nivel de riesgo que están dispuestos a aceptar para equilibrar la experiencia del usuario y la postura de seguridad. 

La recomendación de Microsoft es establecer el umbral de la directiva de riesgo de usuario en **Alto** y la directiva de riesgo de inicio de sesión en **Medio y superior**.

Elegir un umbral **Alto** reduce el número de veces que una directiva se desencadena y minimiza el impacto en los usuarios. Sin embargo, excluye de la directiva las detecciones de riesgo **Bajo** y **Medio**, por lo que es posible que no impida que un atacante aproveche una identidad en peligro. Seleccionar un umbral **Bajo** presenta interrupciones adicionales para el usuario, pero aumenta el control de la seguridad.

## <a name="exclusions"></a>Exclusiones

Todas las directivas permiten excluir a usuarios, como las [cuentas de administrador de acceso de emergencia](../users-groups-roles/directory-emergency-access.md). Las organizaciones pueden determinar la necesidad de excluir otras cuentas de algunas directivas específicas en función de la forma en que se utilizan las cuentas. Todas las exclusiones deben revisarse periódicamente para ver si siguen siendo aplicables.

Identity Protection usa las [ubicaciones de red](../conditional-access/location-condition.md) de confianza configuradas en algunas detecciones de riesgos para reducir los falsos positivos.

## <a name="enable-policies"></a>Habilitación de directivas

Para habilitar las directivas de riesgo de usuario e inicio de sesión, complete los pasos siguientes.

1. Acceda a [Azure Portal](https://portal.azure.com).
1. Vaya a **Azure Active Directory** > **Seguridad** > **Identity Protection** > **Información general**.
1. Seleccione **Directiva de riesgo de usuario**.
   1. En **Asignaciones**
      1. **Usuarios**: elija **Todos los usuarios** o **Seleccionar individuos y grupos** si limita el lanzamiento.
         1. Opcionalmente, puede optar por excluir usuarios de la directiva.
      1. **Condiciones** - **Riesgo de usuario**: la recomendación de Microsoft es establecer esta opción en **Alto**.
   1. En **Controles**
      1. **Acceso**: la recomendación de Microsoft es **Permitir el acceso** y **Requerir cambio de contraseña**.
   1. **Aplicar directiva** - **Activado**
   1. **Guardar**: esta acción le devolverá a la página **Información general**.
1. Seleccione **Directiva de riesgo de inicio de sesión**.
   1. En **Asignaciones**
      1. **Usuarios**: elija **Todos los usuarios** o **Seleccionar individuos y grupos** si limita el lanzamiento.
         1. Opcionalmente, puede optar por excluir usuarios de la directiva.
      1. **Condiciones** - **Riesgo de inicio de sesión**: la recomendación de Microsoft es establecer esta opción en **Medio y superior**.
   1. En **Controles**
      1. **Acceso**: la recomendación de Microsoft es **Permitir el acceso** y **Requerir autenticación multifactor**.
   1. **Aplicar directiva** - **Activado**
   1. **Guardar**

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación de la directiva de registro de autenticación multifactor de Azure](howto-identity-protection-configure-mfa-policy.md)

- [¿Qué es el riesgo?](concept-identity-protection-risks.md)

- [Investigación de detecciones de riesgos](howto-identity-protection-investigate-risk.md)

- [Simulación de detecciones de riesgos](howto-identity-protection-simulate-risk.md)
