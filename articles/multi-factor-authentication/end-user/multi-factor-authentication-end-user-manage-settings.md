---
title: "Administrar la configuración de verificación en dos pasos | Microsoft Docs"
description: "Administre cómo usar Azure Multi-Factor Authentication, incluida la modificación de la información de contacto o la configuración de los dispositivos."
services: multi-factor-authentication
keywords: "cliente de multi-factor authentication, problema de autenticación, identificador de correlación"
documentationcenter: 
author: barlanmsft
manager: mtillman
ms.reviewer: richagi
ms.assetid: d3372d9a-9ad1-4609-bdcf-2c4ca9679a3b
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/23/2017
ms.author: barlan
ms.custom: end-user
ms.openlocfilehash: 8d84574283aa0c94ce303b0a7e3bde335c0eb2b8
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2017
---
# <a name="manage-your-settings-for-two-step-verification"></a>Administración de la configuración de la comprobación en dos pasos
Este artículo ofrece respuestas a preguntas acerca de cómo actualizar la configuración de actualización en dos pasos o la autenticación multifactor. Si tiene problemas para iniciar sesión en su cuenta, consulte [Problemas con la comprobación en dos pasos](multi-factor-authentication-end-user-troubleshoot.md) para obtener ayuda para la solución de problemas.

## <a name="where-to-find-the-settings-page"></a>Dónde encontrar la página de configuración
Según cómo su compañía configure Azure Multi-Factor Authentication, existen algunos lugares donde puede cambiar los ajustes, como su número de teléfono.

Si su compañía admite el envío de una dirección URL específica o pasos para administrar la verificación en dos pasos, siga las instrucciones. De lo contrario, las instrucciones siguientes deben funcionar para los demás. Si se siguen estos pasos, pero no ve las mismas opciones, significa que su trabajo o escuela personalizó su propio portal. Consulte al administrador para obtener el vínculo a su portal de Azure Multi-Factor Authentication.

1. Inicie sesión en [https://myapps.microsoft.com](https://myapps.microsoft.com)  
2. Seleccione el nombre de cuenta en la parte superior derecha y, después, seleccione **Perfil**.  
3. Seleccione **Comprobación de seguridad adicional**.  

    ![MyApps](./media/multi-factor-authentication-end-user-manage/myapps1.png)
4. La página de comprobación de seguridad adicional se carga con la configuración.

    ![Página de proofup](./media/multi-factor-authentication-end-user-manage/proofup.png)

## <a name="i-want-to-change-my-phone-number-or-add-a-secondary-number"></a>Deseo cambiar mi número de teléfono o agregar un número secundario
Es importante configurar un número de teléfono de autenticación secundario.  Debido a que el número de teléfono principal y la aplicación móvil probablemente se encuentran en el mismo teléfono, el número de teléfono secundario es la única forma que tiene para volver a tener acceso a su cuenta si se le pierde el teléfono o si se lo robaron.

> [!NOTE]
> Si no dispone de acceso a su número de teléfono principal y necesita ayuda para encontrar su cuenta, vea nuestros temas de ayuda [Problemas con la comprobación en dos pasos](multi-factor-authentication-end-user-troubleshoot.md).  

**Para cambiar el número de teléfono principal:**  

1. En la página de comprobación de seguridad adicional, seleccione el cuadro de texto con el número de teléfono actual y modifíquelo con el nuevo número de teléfono.  
2. Seleccione **Guardar**.  
3. Si este es el número que se utiliza para la opción de comprobación preferida, tiene que comprobar el nuevo número para poder guardarlo.  

**Para agregar un número de teléfono secundario:**  

1. En la página de comprobación de seguridad adicional, active la casilla junto al **teléfono de autenticación alternativo.**  
2. En el cuadro de texto, escriba el número de teléfono secundario.  
3. Seleccione **Guardar** y los cambios habrán finalizado.  

## <a name="require-two-step-verification-again-on-a-device-youve-marked-as-trusted"></a>Requisito de nueva verificación en dos pasos en un dispositivo marcado como de confianza

Dependiendo de la configuración de la organización, es posible que tenga una casilla que indica "Don't ask again for **X** days" (No volver a preguntar en X días) al realizar la verificación en dos pasos en el explorador. Si marca este cuadro y pierde el dispositivo o piensa que se ha comprometido la seguridad de su cuenta, debe restaurar la verificación en dos pasos en todos los dispositivos.

1. En la página Comprobación de seguridad adicional, seleccione **Restaurar Multi-Factor Authentication en dispositivos en los que se confió anteriormente**.
2. La próxima vez que inicie sesión en cualquier dispositivo, se le pedirá que realice la verificación en dos pasos.

## <a name="how-do-i-clean-up-microsoft-authenticator-from-my-old-device-and-move-to-a-new-one"></a>¿Cómo puedo eliminar Microsoft Authenticator del dispositivo anterior y moverlo a uno nuevo?
Cuando desinstala la aplicación del dispositivo o restablece el dispositivo, no se elimina la activación en el back-end. Para más información, vea [Microsoft Authenticator](microsoft-authenticator-app-how-to.md).

## <a name="next-steps"></a>Pasos siguientes
* Obtener sugerencias para solucionar problemas y necesita ayuda [Problemas con la comprobación en dos pasos](multi-factor-authentication-end-user-troubleshoot.md)
* Configure [contraseñas de aplicación](multi-factor-authentication-end-user-app-passwords.md) para las aplicaciones que no admiten la comprobación en dos pasos.
