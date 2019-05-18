---
title: Crear un grupo de host de vista previa de Escritorio Virtual Windows con una plantilla de Azure Resource Manager - Azure
description: Cómo crear un grupo host en la vista previa de Escritorio Virtual de Windows con una plantilla de Azure Resource Manager.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 04/05/2019
ms.author: helohr
ms.openlocfilehash: 4df8b854a94374346eec6f77d761e65a04ef78a2
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523982"
---
# <a name="create-a-host-pool-with-an-azure-resource-manager-template"></a>Creación de un grupo host con una plantilla de Azure Resource Manager

Los grupos de hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilinos de Windows Virtual Desktop (versión preliminar). Cada grupo de hosts puede contener un grupo de aplicaciones con las que los usuarios pueden interactuar igual que harían en un equipo de escritorio físico.

Siga las instrucciones de esta sección para crear un grupo host para un inquilino de Escritorio Virtual de Windows con una plantilla de Azure Resource Manager proporcionada por Microsoft. Este artículo explica cómo crear un grupo host en el escritorio Virtual de Windows, cree un grupo de recursos con las máquinas virtuales en una suscripción de Azure, unir esas máquinas virtuales al dominio de Active Directory y registrar las máquinas virtuales con Windows de Escritorio Virtual.

## <a name="what-you-need-to-run-the-azure-resource-manager-template"></a>Lo que necesita para ejecutar la plantilla de Azure Resource Manager

Asegúrese de que sabe lo siguiente antes de ejecutar la plantilla de Azure Resource Manager:

- Donde es el origen de la imagen que desea usar. ¿Es desde la Galería de Azure o está personalizada?
- Sus credenciales de unión al dominio.
- Las credenciales de Escritorio Virtual de Windows.

Cuando se crea un grupo de host de Escritorio Virtual de Windows con la plantilla de Azure Resource Manager, puede crear una máquina virtual desde la Galería de Azure, una imagen administrada o una imagen no administrada. Para obtener más información sobre cómo crear imágenes de máquina virtual, consulte [preparar un VHD de Windows o un VHDX para cargar en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/prepare-for-upload-vhd-image) y [crear una imagen administrada de una máquina virtual generalizada en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/capture-image-resource).

## <a name="run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool"></a>Ejecutar la plantilla de Azure Resource Manager para aprovisionar un nuevo grupo host

Para empezar, vaya a [esta dirección URL de GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/Create%20and%20provision%20WVD%20host%20pool).

### <a name="deploy-the-template-to-azure"></a>Implementación de la plantilla en Azure

Si va a implementar en una suscripción Enterprise, desplácese hacia abajo y seleccione **implementar en Azure**, a continuación, omitir anticipada, rellene los parámetros en función del origen de imagen.

Si va a implementar en una suscripción de proveedor de soluciones en la nube, siga estos pasos para implementar en Azure:

1. Desplácese hacia abajo y haga clic en **implementar en Azure**, a continuación, seleccione **ubicación del vínculo de copia**.
2. Abra un editor de texto como el Bloc de notas y pegue el vínculo no existe.
3. Justo después de "https://portal.azure.com/" y el hashtag (##) antes de escribir un signo de arroba (@) seguido del nombre de dominio del inquilino. Este es un ejemplo del formato que debe usar: https://portal.azure.com/@Contoso.onmicrosoft.com#create/.
4. Inicie sesión en el portal de Azure como un usuario con permisos de administrador o colaborador a la suscripción del proveedor de soluciones en la nube.
5. Pegue el vínculo que ha copiado en el editor de texto en la barra de direcciones.

Para obtener instrucciones acerca de los parámetros que debe escribir para su escenario, vea el escritorio Virtual Windows [archivo Léame](https://github.com/Azure/RDS-Templates/blob/master/wvd-templates/Create%20and%20provision%20WVD%20host%20pool/README.md). El archivo Léame se actualiza siempre con los cambios más recientes.

## <a name="assign-users-to-the-desktop-application-group"></a>Asignar a usuarios al grupo de aplicaciones de escritorio

Una vez finalizada la plantilla de GitHub de Azure Resource Manager, asigne acceso de usuario antes de iniciar las pruebas de los escritorios de la sesión completa en sus máquinas virtuales.

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview) que se usará en la sesión de PowerShell.

Para asignar a usuarios al grupo de aplicaciones de escritorio, abra una ventana de PowerShell y ejecute este cmdlet para iniciar sesión en el entorno de Escritorio Virtual de Windows:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

Después de eso, agregar usuarios al grupo de aplicaciones de escritorio con este cmdlet:

```powershell
Add-RdsAppGroupUser <tenantname> <hostpoolname> "Desktop Application Group" -UserPrincipalName <userupn>
```

El UPN del usuario debe coincidir con la identidad del usuario en Azure Active Directory (por ejemplo, user1@contoso.com). Si desea agregar varios usuarios, debe ejecutar este cmdlet para cada usuario.

Después de haber completado estos pasos, los usuarios agregados al grupo de aplicaciones de escritorio pueden iniciar sesión en Windows Virtual Desktop con clientes de Escritorio remoto compatibles y ver un recurso de escritorio de sesión.

>[!IMPORTANT]
>Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host. Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar [acceso de máquina virtual Just-in-Time](https://docs.microsoft.com/azure/security-center/security-center-just-in-time).