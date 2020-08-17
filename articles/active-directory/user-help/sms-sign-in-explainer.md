---
title: 'Experiencia del usuario de inicio de sesión mediante SMS con el número de teléfono (versión preliminar): Azure AD'
description: Más información sobre la experiencia del usuario de inicio de sesión mediante SMS con números de teléfono nuevos o existentes
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 04/14/2020
ms.author: curtand
ms.reviewer: kasimpso
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: 864225238c10ab2fda96e95448790201cc8a16ae
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423004"
---
# <a name="use-your-phone-number-as-a-user-name-preview"></a>Uso del número de teléfono como nombre de usuario (versión preliminar)

El registro de un dispositivo permite que el teléfono acceda a los servicios de la organización, pero no que la organización acceda a su teléfono. Si es administrador, puede encontrar más información en [Configuración y habilitación de los usuarios para la autenticación basada en SMS](../authentication/howto-authentication-sms-signin.md).

Si la organización no ha habilitado el inicio de sesión mediante SMS, no verá la opción correspondiente al registrar un teléfono con su cuenta.  

## <a name="when-you-have-a-new-phone-number"></a>Si tiene un nuevo número de teléfono

Si recibe un nuevo teléfono o un nuevo número y lo registra en una organización para la que el inicio de sesión mediante SMS está disponible, seguirá el proceso de registro de teléfono normal:

1. Seleccione **Agregar método**.
1. Seleccione **Teléfono**.
1. Escriba el número de teléfono y seleccione **Enviarme un código por mensaje de texto**.
1. Después de escribir el código, seleccione **Siguiente**.
1. Verá un mensaje que indica "SMS verificado. El teléfono se ha registrado correctamente."

> [!Important]
> Debido a un problema conocido en la versión preliminar, por un breve período la incorporación de un número de teléfono no lo registrará para el inicio de sesión mediante SMS. Tendrá que iniciar sesión con el número agregado y, a continuación, seguir las indicaciones para registrar el número para el inicio de sesión mediante SMS.

### <a name="when-the-phone-number-is-in-use"></a>Cuando el número de teléfono está en uso

Si intenta usar un número de teléfono que ya está usando otro usuario de la organización, verá el siguiente mensaje:

![Mensaje de error que aparece cuando ya se está usando el número de teléfono](media/sms-sign-in-explainer/sms-sign-in-error.png)

Póngase en contacto con su administrador para corregir el problema.

## <a name="when-you-have-an-existing-number"></a>Si tiene un número existente

Si ya usa un número de teléfono con una organización y pasa a estar disponible la opción de usar el número de teléfono como nombre de usuario, los pasos siguientes pueden ayudarle a iniciar sesión.

1. Cuando el inicio de sesión mediante SMS está disponible, se muestra un banner en el que se le pregunta si desea habilitar el número de teléfono para el inicio de sesión mediante SMS:

    [![Banner para habilitar el inicio de sesión mediante SMS en un número de teléfono.](media/sms-sign-in-explainer/sms-sign-in-banner.png)](media/sms-sign-in-explainer/sms-sign-in-banner.png#lightbox)

1. Además, aparece un botón **Habilitar** si selecciona el símbolo en el icono del método de teléfono:

    [![Banner para habilitar el inicio de sesión mediante SMS en un número de teléfono.](media/sms-sign-in-explainer/sms-sign-in-phone-method.png)](media/sms-sign-in-explainer/sms-sign-in-phone-method.png#lightbox)

1. Para habilitar el método, seleccione **Habilitar**. Se le pedirá que confirme la acción:

    ![Cuadro de diálogo de confirmación para habilitar el inicio de sesión mediante SMS en un número de teléfono](media/sms-sign-in-explainer/sms-sign-in-confirmation.png)

1. Seleccione **Habilitar**.

## <a name="when-you-remove-your-phone-number"></a>Si quita el número de teléfono

1. Para eliminar el número de teléfono, seleccione el botón de eliminación en el icono del método de teléfono de inicio de sesión mediante SMS.

    [![Banner para eliminar el inicio de sesión mediante SMS en un número de teléfono.](media/sms-sign-in-explainer/sms-sign-in-delete-method.png)](media/sms-sign-in-explainer/sms-sign-in-delete-method.png#lightbox)

2. Cuando se le pida que confirme la acción, seleccione **Aceptar**.

No se puede quitar un número de teléfono que esté en uso como método de inicio de sesión predeterminado. Para quitar el número, tendría que cambiar el método de inicio de sesión predeterminado y, a continuación, quitar el número de teléfono.
