---
title: 'Grupo de hosts de Windows Virtual Desktop (clásico) mediante Azure Resource Manager: Azure'
description: Creación de un grupo de hosts en Windows Virtual Desktop (clásico) con una plantilla de Azure Resource Manager.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: b9999ae92840d79bb19464216c0f28504011b3f3
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008328"
---
# <a name="create-a-host-pool-in-windows-virtual-desktop-classic-with-an-azure-resource-manager-template"></a>Creación de un grupo de hosts en Windows Virtual Desktop (clásico) con una plantilla de Azure Resource Manager

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

Los grupos hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilino de Windows Virtual Desktop. Cada grupo de hosts puede contener un grupo de aplicaciones con las que los usuarios pueden interactuar igual que harían en un equipo de escritorio físico.

Siga las instrucciones de esta sección para crear un grupo de hosts de un inquilino de Windows Virtual Desktop con una plantilla de Azure Resource Manager proporcionada por Microsoft. En este artículo se explica cómo crear un grupo de hosts en Windows Virtual Desktop, cómo crear un grupo de recursos con máquinas virtuales en una suscripción de Azure, cómo unir esas máquinas virtuales al dominio de AD y cómo registrar las máquinas virtuales con Windows Virtual Desktop.

## <a name="what-you-need-to-run-the-azure-resource-manager-template"></a>¿Qué se necesita para ejecutar la plantilla de Azure Resource Manager?

Asegúrese de que conoce los siguientes datos antes de ejecutar la plantilla de Azure Resource Manager:

- Dónde está el origen de la imagen que quiere usar. ¿Está en Azure Gallery o es personalizada?
- Sus credenciales de unión un dominio.
- Sus credenciales de Windows Virtual Desktop.

Cuando se crea un grupo de hosts de Windows Virtual Desktop con la plantilla de Azure Resource Manager, se puede crear una máquina virtual desde una imagen de Azure Gallery, una imagen administrada o una imagen no administrada. Para obtener más información sobre cómo crear imágenes de máquina virtual, vea [Preparación de un VHD o un VHDX de Windows para cargar en Azure](../../virtual-machines/windows/prepare-for-upload-vhd-image.md) y [Creación de una imagen administrada de una máquina virtual generalizada en Azure](../../virtual-machines/windows/capture-image-resource.md).

## <a name="run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool"></a>Ejecutar la plantilla de Azure Resource Manager para el aprovisionamiento de un nuevo grupo de hosts

Para empezar, vaya a [esta dirección URL de GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/Create%20and%20provision%20WVD%20host%20pool).

### <a name="deploy-the-template-to-azure"></a>Implementación de la plantilla en Azure

Si va a realizar la implementación en una suscripción de Enterprise, desplácese hacia abajo, seleccione **Implementar en Azure** y, luego, vaya directamente a rellenar los parámetros según el origen de la imagen.

Si va a realizar la implementación en una suscripción de proveedor de soluciones en la nube, haga lo siguiente para implementar en Azure:

1. Desplácese hacia abajo, haga clic con el botón derecho en **Implementar en Azure** y, después, seleccione **Copy Link Location** (Copiar ubicación del vínculo).
2. Abra un editor de texto como el Bloc de notas y pegue el vínculo ahí.
3. Justo después de "https://portal.azure.com/" y antes de la almohadilla (#), escriba una arroba (@) seguida del nombre de dominio del inquilino. Este es un ejemplo del formato que debe usar: `https://portal.azure.com/@Contoso.onmicrosoft.com#create/`.
4. Inicie sesión en el Azure Portal como un usuario con permisos de administrador o colaborador en la suscripción del proveedor de soluciones en la nube.
5. Pegue el vínculo que ha copiado en el editor de texto en la barra de direcciones.

Para obtener instrucciones sobre los parámetros que debe especificar en su caso, vea el [archivo Léame](https://github.com/Azure/RDS-Templates/blob/master/wvd-templates/Create%20and%20provision%20WVD%20host%20pool/README.md)de Windows Virtual Desktop. El archivo Léame siempre está actualizado con los cambios más recientes.

## <a name="assign-users-to-the-desktop-application-group"></a>Asignar usuarios al grupo de aplicaciones de escritorio

Cuando la plantilla de Azure Resource Manager de GitHub finalice, asigne accesos de usuario antes de empezar a comprobar los escritorios de sesión completos en sus máquinas virtuales.

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell.

Para asignar usuarios al grupo de aplicaciones de escritorio, abra una ventana de PowerShell y ejecute este cmdlet para iniciar sesión en el entorno de Windows Virtual Desktop:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

Tras esto, agregue usuarios al grupo de aplicaciones de escritorio con este cmdlet:

```powershell
Add-RdsAppGroupUser <tenantname> <hostpoolname> "Desktop Application Group" -UserPrincipalName <userupn>
```

El nombre principal de usuario debe coincidir con la identidad del usuario en Azure Active Directory (por ejemplo, user1@contoso.com). Si desea agregar varios usuarios, debe ejecutar este cmdlet para cada usuario.

Después de haber completado estos pasos, los usuarios agregados al grupo de aplicaciones de escritorio pueden iniciar sesión en Windows Virtual Desktop con clientes de Escritorio remoto compatibles y ver un recurso de escritorio de sesión.

>[!IMPORTANT]
>Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host. Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar [acceso de máquina virtual Just-in-Time](../../security-center/security-center-just-in-time.md).