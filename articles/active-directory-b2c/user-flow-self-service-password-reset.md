---
title: Restablecimiento de contraseña de autoservicio en Azure Active Directory B2C | Microsoft Docs
description: Se demuestra cómo configurar el autoservicio de restablecimiento de contraseña para los consumidores en Azure Active Directory B2C
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0c9edaf3356ea4c1a521a89f2ec60a4b6ba1a5ef
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87481502"
---
# <a name="set-up-self-service-password-reset-for-your-customers"></a>Configuración del autoservicio de restablecimiento de contraseña para los consumidores

Con la característica de autoservicio de restablecimiento de contraseña, los consumidores que se registraron para obtener cuentas locales pueden restablecer sus contraseñas ellos mismos. De esta manera, se reduce considerablemente la carga del personal de soporte técnico, especialmente si la aplicación tiene millones de consumidores que la usan de forma periódica. Actualmente, solo se admite como método de recuperación el uso de una dirección de correo electrónico verificada.

> [!NOTE]
> Este artículo es aplicable al autoservicio de restablecimiento de contraseña en el contexto de un flujo de usuario de **inicio de sesión** estándar, en el que se utiliza **Inicio de sesión de la cuenta local** como proveedor de identidades. Si necesita invocar flujos de usuario de restablecimiento de contraseña totalmente personalizables desde su aplicación, consulte [este artículo](user-flow-overview.md).
>
>

De forma predeterminada, el directorio no tiene activado el autoservicio de restablecimiento de contraseña. Para activarlo, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de la suscripción. Esta cuenta es la misma cuenta profesional o educativa o la misma cuenta Microsoft que usó para crear el directorio.
2. Abra **Azure Active Directory** (en la barra de navegación del lado izquierdo).
3. Desplácese hacia abajo en la hoja de opciones y seleccione **Restablecimiento de contraseña**.
4. Establezca **Se habilitó el restablecimiento de contraseña del autoservicio.** en **Todo**.
5. Haga clic en **Guardar** en la parte superior de la página. ¡Y ya está!

Para probar, use la característica "Ejecutar ahora" en cualquier flujo de usuario de inicio de sesión que tenga cuentas locales como proveedor de identidades. En la página de inicio de sesión de la cuenta local (donde escribe la dirección de correo electrónico y la contraseña, o bien el nombre de usuario y la contraseña), haga clic en **Can't access your account?** (¿No se puede acceder a la cuenta?) para verificar la experiencia del consumidor.

> [!NOTE]
> Las páginas del autoservicio de restablecimiento de contraseña se pueden personalizar con la [característica de personalización de marca de empresa](../active-directory/fundamentals/customize-branding.md).
>
>

