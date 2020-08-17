---
title: Configuración de los requisitos de complejidad de las contraseñas
titleSuffix: Azure AD B2C
description: Configuración de los requisitos de complejidad de contraseñas proporcionados por los consumidores en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9c7f0ead3a2f93d7f390106b8e7471d58b709023
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87481570"
---
# <a name="configure-complexity-requirements-for-passwords-in-azure-active-directory-b2c"></a>Configuración de los requisitos de complejidad de contraseñas de Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) admite el cambio de los requisitos de complejidad de las contraseñas proporcionadas por un usuario final al crear una cuenta. De forma predeterminada, Azure AD B2C utiliza contraseñas `Strong`. Azure AD B2C también admite opciones de configuración para controlar la complejidad de las contraseñas que los clientes pueden usar.

## <a name="password-rule-enforcement"></a>Aplicación de las reglas de contraseña

Al realizar el registro o el restablecimiento de contraseña, un usuario final tiene que proporcionar una contraseña que cumpla las reglas de complejidad. Las reglas de complejidad de contraseña se aplican por flujo de usuario. Es posible tener un flujo de usuario que exija un pin de cuatro dígitos durante el registro mientras otro flujo de usuario requiere una cadena de ocho caracteres al realizar el registro. Por ejemplo, puede utilizar un flujo de usuario con complejidad de contraseña diferente para adultos y para niños.

La complejidad de la contraseña nunca se aplica durante el inicio de sesión. Nunca se pide a los usuarios durante el inicio de sesión que cambien su contraseña porque no cumple los requisitos de complejidad actuales.

La complejidad de la contraseña puede configurarse en los siguientes tipos de flujos de usuario:

- Flujo de usuario de registro o de inicio de sesión
- Flujo de usuario de restablecimiento de contraseña

Si usa directivas personalizadas, puede ([configurar la complejidad de la contraseña en una directiva personalizada](custom-policy-password-complexity.md)).

## <a name="configure-password-complexity"></a>Configurar la complejidad de la contraseña

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
3. En Azure Portal, busque y seleccione **Azure AD B2C**.
4. Seleccione **Flujos de usuario**.
2. Seleccione un flujo de usuario y haga clic en **Propiedades**.
3. En **Complejidad de la contraseña**, cambie la complejidad de contraseña de este flujo de usuario a **Simple**, **Segura** o **Personalizada**.

### <a name="comparison-chart"></a>Gráfico de comparación

| Complejidad | Descripción |
| --- | --- |
| Simple | Una contraseña que tenga al menos de 8 a 64 caracteres. |
| Alta | Una contraseña que tenga al menos de 8 a 64 caracteres. Requiere 3 de 4 de minúsculas, mayúsculas, números o símbolos. |
| Personalizado | Esta opción proporciona el máximo control sobre las reglas de complejidad de contraseña.  Permite configurar una longitud personalizada.  También permite aceptar contraseñas compuestas solo por números (PIN). |

## <a name="custom-options"></a>opciones personalizadas

### <a name="character-set"></a>Juego de caracteres

Permite aceptar solo dígitos (PIN) o el juego completo de caracteres.

- **Solo números** permite solamente dígitos (0-9) al escribir una contraseña.
- **Todos** permite cualquier letra, número o símbolo.

### <a name="length"></a>Length

Permite controlar los requisitos de longitud de la contraseña.

- **Longitud mínima** tiene que ser 4 como mínimo.
- **Longitud máxima** tiene que ser mayor o igual que la longitud mínima y como máximo puede ser 64 caracteres.

### <a name="character-classes"></a>Clase de caracteres

Permite controlar los distintos tipos de caracteres utilizados en la contraseña.

- **2 de 4: carácter en minúscula, carácter en mayúscula, número (0-9), símbolo** garantiza que la contraseña contiene al menos dos tipos de caracteres. Por ejemplo, un número y un carácter en minúscula.
- **3 de 4: carácter en minúscula, carácter en mayúscula, número (0-9), símbolo** garantiza que la contraseña contiene al menos tres tipos de caracteres. Por ejemplo, un número, un carácter en minúscula y un carácter en mayúscula.
- **4 de 4: carácter en minúscula, carácter en mayúscula, número (0-9), símbolo** garantiza que la contraseña contiene al menos todos tipos de caracteres.

    > [!NOTE]
    > Solicitar una contraseña de tipo **4 de 4** puede dar lugar a la frustración del usuario final. Algunos estudios han demostrado que este requisito no mejorar la entropía de contraseña. Consulte las [directrices para contraseñas del NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#appA)
