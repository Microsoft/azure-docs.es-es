---
title: 'Herramienta de administración de Windows Virtual Desktop (clásico): Azure'
description: Solución de problemas con la herramienta de administración de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6f03284103b4a2aa6900bf1ba5c50a4688c50b0e
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005489"
---
# <a name="troubleshoot-the-windows-virtual-desktop-classic-management-tool"></a>Solución de problemas de la herramienta Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

En este artículo se describen los problemas que pueden producirse durante la implementación de la herramienta de Windows Virtual Desktop y cómo corregirlos.

## <a name="error-management-tool-services-configured-but-automated-setup-fails"></a>Error: Se configuran los servicios de la herramienta de administración, pero la instalación automatizada produce un error

Cuando se configuran correctamente los servicios de la herramienta de administración, pero se produce un error en la instalación automatizada, verá este mensaje de error:

```console
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n "status": "Failed",\r\n "error": {\r\n "code": "ResourceDeploymentFailure",\r\n "message": "The resource operation completed with terminal provisioning state 'Failed'."\r\n }\r\n}"}]}
```

Esto suele significar dos cosas:

- El usuario tiene permisos de propietario en su suscripción y de administrador global en el nivel de inquilino, pero no puede iniciar sesión en Azure.
- La configuración de la cuenta del usuario tiene habilitada la autenticación multifactor.

Para solucionar este error:

1. Asegúrese de que el usuario que ha creado con el nombre principal de usuario de Azure Active Directory tenga el nivel de suscripción de "colaborador".
2. Inicie sesión en <portal.azure.com> con la cuenta de UPN para comprobar la configuración de la cuenta y asegúrese de que la autenticación multifactor no esté activada. Si está activada, desactívela.
3. Visite la página de consentimiento de Windows Virtual Desktop y asegúrese de que las aplicaciones cliente y servidor tienen el consentimiento.
4. Si el problema continúa, revise el tutorial [Implementación de una herramienta de administración](manage-resources-using-ui.md) y vuelva a implementar la herramienta.

## <a name="error-job-with-specified-id-already-exists"></a>Error: El trabajo con el identificador especificado ya existe

Si el usuario ve el mensaje de error "	Ya existe un trabajo con el id. especificado", es porque no proporcionó un nombre único en el parámetro "Nombre de la aplicación" al implementar la plantilla.

Para corregir este error, vuelva a implementar la herramienta de administración con el parámetro "Nombre de la aplicación" rellenado.

## <a name="delayed-consent-prompt-when-opening-management-tool"></a>Petición de consentimiento retrasada al abrir la herramienta de administración

Al implementar la herramienta de administración, es posible que la petición de consentimiento no se abra de inmediato. Esto significa que el servicio de aplicaciones web de Azure está tardando más de lo habitual en cargarse. El mensaje debe aparecer después de que haya terminado de cargarse.

## <a name="the-user-cant-deploy-the-management-tool-in-the-east-us-region"></a>El usuario no puede implementar la herramienta de administración en la región este de EE. UU.

Si un cliente establece la región en este de EE. UU., no podrá implementar la herramienta de administración.

Para solucionarlo, implemente la herramienta de administración en otra región. La nueva implementación de la herramienta en otra región no debe afectar a la experiencia del usuario.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los seguimientos de escalación en [Información general sobre solución de problemas, comentarios y soporte técnico](troubleshoot-set-up-overview-2019.md).
- Aprenda a notificar problemas con las herramientas de Windows Virtual Desktop en [Plantillas de ARM para Servicios de Escritorio remoto](https://github.com/Azure/RDS-Templates/blob/master/README.md).
- Para aprender a implementar la herramienta de administración, consulte [Implementación de una herramienta de administración](manage-resources-using-ui.md).