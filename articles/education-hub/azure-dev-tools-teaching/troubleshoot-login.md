---
title: Resolución del error de inicio de sesión de Azure Dev Tools for Teaching
description: Se describen las acciones que debe realizar un alumno si reciben un mensaje de error al iniciar sesión en Azure Dev Tools for Teaching.
author: rymend
ms.author: rymend
ms.topic: troubleshooting
ms.service: azure-education
ms.subservice: education-hub
ms.date: 06/30/2020
ms.openlocfilehash: 276ab2cc1d79235c00ce1bb50ff9b648bb1de6ea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095110"
---
# <a name="troubleshooting-student-login-issues"></a>Solución de problemas de inicio de sesión de alumnos
El acceso a Azure Dev Tools for Teaching requiere que un usuario tenga una cuenta de Microsoft (MSA). Los alumnos se dirigirán automáticamente a la página para crear una cuenta MSA si la suya no lo es o no está vinculada a una. Si el dominio está asociado a Active Directory, todas las cuentas de ese dominio ya se consideran MSA.

Si un alumno intenta iniciar sesión y recibe el mensaje de error siguiente, use una de las soluciones que se describen a continuación.

:::image type="content" source="media/troubleshoot-login/error-login-message.png" alt-text="Mensaje de error de inicio de sesión." border="false":::

Hay dos soluciones: crear una nueva cuenta de Microsoft o usar una cuenta de Microsoft existente.

## <a name="create-a-new-microsoft-account"></a>Creación de una cuenta de Microsoft
### <a name="use-a-university-email-address"></a>Uso de una dirección de correo electrónico de la universidad
Si inicia sesión con una dirección de correo electrónico de la universidad (por ejemplo, `John.Smith@university.edu`), deberá crear una cuenta de Microsoft con esa dirección de correo electrónico. La creación de una cuenta es gratuita y solo requiere unos minutos. Tener un cuenta de Microsoft le permitirá iniciar sesión automáticamente en todos los productos de Microsoft con un nombre de usuario y una contraseña únicos.

### <a name="use-a-personal-email-address"></a>Uso una dirección de correo electrónico personal
Si inicia sesión con una dirección de correo electrónico personal (por ejemplo, `John.Smith@email.com`), pero también tiene una dirección de correo electrónico de la universidad, pruebe a usar la dirección de correo electrónico de la universidad. Si anteriormente inició sesión en la tienda web de su institución con una dirección de correo electrónico personal o no tiene ninguna dirección de correo electrónico de la universidad, deberá crear o vincular un cuenta de Microsoft con su dirección de correo electrónico personal.

## <a name="use-an-existing-microsoft-account"></a>Uso de un cuenta de Microsoft existente
Si un alumno tiene un cuenta de Microsoft existente (por ejemplo, Xbox), puede conectar esa cuenta a una cuenta de Azure Dev Tools.

1. Ir a https://account.microsoft.com.
1. Inicie sesión con las credenciales de su cuenta de Microsoft.
1. Seleccione **Tu información** en el menú superior de la cinta de opciones.

1. Haga clic en **Administrar cómo inicia sesión en Microsoft**. Se le pedirá que compruebe su identidad. Se le enviará un código de seguridad por correo electrónico.

    :::image type="content" source="media/troubleshoot-accounts/manage-sign-in.png" alt-text="Administrar el inicio de sesión." border="false":::

1. Escriba el código de seguridad.

    :::image type="content" source="media/troubleshoot-accounts/sign-in-enter-code.png" alt-text="Escribir el código de inicio de sesión." border="false":::

1. Haga clic en **Agregar correo electrónico** a su cuenta y escriba la dirección de correo electrónico de la universidad.
La próxima vez que inicie sesión, puede usar su dirección de correo electrónico de la universidad para acceder a las Azure Dev Tools for Teaching.

    :::image type="content" source="media/troubleshoot-accounts/manage-sign-in-add-email.png" alt-text="Administrar cómo inicia sesión en Microsoft." border="false":::

## <a name="next-steps"></a>Pasos siguientes
- [P+F](program-faq.md)

- [Opciones de soporte técnico](program-support.md)
