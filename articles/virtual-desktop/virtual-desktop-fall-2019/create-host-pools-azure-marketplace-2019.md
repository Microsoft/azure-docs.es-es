---
title: 'Grupo de hosts de Windows Virtual Desktop (clásico) en Azure Marketplace: Azure'
description: Procedimientos para la creación de un grupo de hosts de Windows Virtual Desktop (clásico) mediante Azure Marketplace.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 04184a658ae1efd03afd3e920818eb5317bc7553
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008600"
---
# <a name="tutorial-create-a-host-pool-in-windows-virtual-desktop-classic"></a>Tutorial: Creación de un grupo de hosts en Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido hace referencia a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../create-host-pools-azure-marketplace.md).

En este tutorial, aprenderá cómo crear un grupo de hosts dentro de un inquilino de Windows Virtual Desktop con una oferta de Microsoft Azure Marketplace.

Los grupos hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilino de Windows Virtual Desktop. Cada grupo de hosts puede contener un grupo de aplicaciones con las que los usuarios pueden interactuar igual que harían en un equipo de escritorio físico.

Las tareas de este tutorial incluyen:

> [!div class="checklist"]
>
> * Crear un grupo de hosts en Windows Virtual Desktop.
> * Crear un grupo de recursos con máquinas virtuales en una suscripción de Azure.
> * Unir las máquinas virtuales al dominio de Active Directory.
> * Registrar las máquinas virtuales con Windows Virtual Desktop.

## <a name="prerequisites"></a>Prerrequisitos

* Un inquilino en Virtual Desktop. Un [tutorial](tenant-setup-azure-active-directory.md) anterior crea un inquilino.
* [Módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/)

Cuando tenga este módulo, ejecute el siguiente cmdlet para iniciar sesión en su cuenta:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="run-the-azure-marketplace-offering-to-provision-a-new-host-pool"></a>Ejecución de la oferta de Azure Marketplace para aprovisionar un nuevo grupo de hosts

Para ejecutar la oferta de Azure Marketplace para aprovisionar un nuevo grupo de hosts:

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
1. Escriba **Windows Virtual Desktop** en la ventana de búsqueda de Marketplace.
1. Seleccione **Windows Virtual Desktop - Provision a host pool** (Windows Virtual Desktop: aprovisionar un grupo de hosts) y seleccione **Crear**.

Después, siga las instrucciones de la sección siguiente para escribir la información de las pestañas adecuadas.

### <a name="basics"></a>Aspectos básicos

En la pestaña **Aspectos básicos**, haga lo siguiente:

1. Seleccione una opción en **Suscripción**.
1. En **Grupo de recursos**, seleccione **Crear nuevo** y proporcione un nombre para el nuevo grupo de recursos.
1. Seleccione una **región**.
1. Escriba un nombre para el grupo de hosts que sea un nombre único dentro del inquilino de Windows Virtual Desktop.
1. Seleccione **Desktop type** (Tipo de escritorio). Si selecciona **Personal**, cada usuario que se conecte a este grupo de hosts se asignará permanentemente a una máquina virtual.
1. Especifique los usuarios que pueden iniciar sesión en los clientes de Windows Virtual Desktop y tener acceso a un escritorio. Use una lista separada por comas. Por ejemplo, si quiere asignar acceso a `user1@contoso.com` y `user2@contoso.com`, escriba *`user1@contoso.com,user2@contoso.com`* .
1. En **Service metadata location** (Ubicación de metadatos de servicio), seleccione la misma ubicación que la red virtual que tenga conectividad con el servidor de Active Directory.

   >[!IMPORTANT]
   >Si va a usar una solución pura de Azure Active Directory Domain Services (Azure AD DS) y Azure Active Directory (Azure AD), asegúrese de implementar el grupo host en la misma región que Azure AD DS para evitar los errores de unión al dominio y de credenciales.

1. Seleccione **Siguiente: Configure virtual machines** (Configuración de máquinas virtuales).

### <a name="configure-virtual-machines"></a>Configuración de máquinas virtuales

En la pestaña **Configure virtual machines** (Configuración de las máquinas virtuales):

1. Acepte los valores predeterminados o personalice el número y tamaño de las máquinas virtuales.

    >[!NOTE]
    >Si el tamaño de máquina virtual específico que busca no aparece en el selector, es porque aún no se ha incorporado a la herramienta Azure Marketplace. Para solicitar un tamaño de máquina virtual, cree una solicitud o vote por una existente en el [foro de UserVoice de Windows Virtual Desktop](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).

1. Escriba un prefijo para los nombres de las máquinas virtuales. Por ejemplo, si escribe el nombre *prefijo*, las máquinas virtuales se llamarán **prefijo-0**, **prefijo-1** y así sucesivamente.
1. Seleccione **Siguiente: Configuración de máquina virtual**.

### <a name="virtual-machine-settings"></a>Configuración de máquina virtual

En la pestaña **Configuración de máquina virtual**:

1. En **Origen de la imagen**, seleccione el origen y escriba la información adecuada para encontrarla y almacenarla. Las opciones difieren en el almacenamiento de blobs, la imagen administrada y la galería.

   Si decide no usar discos administrados, seleccione la cuenta de almacenamiento que contiene el archivo *.vhd*.
1. Escriba el nombre principal de usuario y la contraseña. Esta cuenta debe ser la cuenta de dominio que unirá las máquinas virtuales al dominio de Active Directory. Este mismo nombre de usuario y contraseña se creará en las máquinas virtuales como una cuenta local. Puede restablecer estas cuentas locales más adelante.

   >[!NOTE]
   > Si va a unir sus máquinas virtuales a un entorno de Azure AD DS, asegúrese de que su usuario de unión a un dominio también es miembro del [grupo de administradores de controlador de dominio de AAD](../../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group).
   >
   > La cuenta también forma parte del dominio administrado de Azure AD DS o del inquilino de Azure AD. Las cuentas de directorios externos asociadas al inquilino de Azure AD no se pueden autenticar correctamente durante el proceso de unión al dominio.

1. Seleccione la **red virtual** que tenga conectividad con el servidor de Active Directory y, después, elija una subred donde se hospedarán las máquinas virtuales.
1. Seleccione **Siguiente: Información de Windows Virtual Desktop**.

### <a name="windows-virtual-desktop-tenant-information"></a>Información del inquilino de Windows Virtual Desktop

En la pestaña **Windows Virtual Desktop tenant information** (Información del inquilino de Windows Virtual Desktop):

1. En **Nombre de grupo de inquilinos de Windows Virtual Desktop**, escriba el nombre del grupo de inquilinos que contiene su inquilino. A menos que se le haya proporcionado un nombre de grupo de inquilinos específico, deje el valor predeterminado.
1. En **Nombre de inquilino de Windows Virtual Desktop**, escriba el nombre del inquilino donde se creará este grupo de hosts.
1. Especificar el tipo de credenciales que desea usar para autenticarse como propietario de RDS del inquilino de Windows Virtual Desktop. Escriba el nombre principal de usuario o la entidad de servicio y una contraseña.

   Si completó el [tutorial Creación de entidades de servicio y asignaciones de roles con PowerShell](create-service-principal-role-powershell.md), seleccione **serviceprincipal**.

1. En **Entidad de servicio**, para **Id. de inquilino de Azure AD**, escriba la cuenta de administrador de inquilinos para la instancia de Azure AD que contiene la entidad de servicio. Solo se admiten entidades de servicio con credenciales de contraseña.
1. Seleccione **Siguiente: Review + create** (Revisar y crear).

## <a name="complete-setup-and-create-the-virtual-machine"></a>Finalización de la instalación y creación de la máquina virtual

En **Revisar y crear**, revise la información de configuración. Si necesita cambiar algo, vuelva y realice los cambios. Cuando esté listo, seleccione **Crear** para implementar el grupo de hosts.

Según cuántas máquinas virtuales esté creando, este proceso puede tardar 30 minutos o más en completarse.

>[!IMPORTANT]
> Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host.
>
> Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar el acceso de máquina virtual Just-in-Time. Para más información, consulte [Protección de los puertos de administración con acceso Just-in-Time](../../security-center/security-center-just-in-time.md).

## <a name="optional-assign-additional-users-to-the-desktop-application-group"></a>(Opcional) Asignación de usuarios adicionales al grupo de aplicaciones de escritorio

Una vez que Azure Marketplace termine de crear el grupo, puede asignar más usuarios al grupo de aplicaciones de escritorio. Si no desea agregar más, omita esta sección.

Para asignar usuarios al grupo de aplicaciones de escritorio:

1. Abra una ventana de PowerShell.

1. Ejecute el siguiente comando para iniciar sesión en el entorno de Windows Virtual Desktop.

   ```powershell
   Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
   ```

1. Agregue usuarios al grupo de aplicaciones de escritorio con este comando:

   ```powershell
   Add-RdsAppGroupUser <tenantname> <hostpoolname> "Desktop Application Group" -UserPrincipalName <userupn>
   ```

   El nombre principal de usuario debe coincidir con la identidad del usuario en Azure AD, por ejemplo, *user1@contoso.com* . Si desea agregar varios usuarios, ejecute el comando para cada usuario.

Los usuarios que ha agregado al grupo de aplicaciones de escritorio pueden iniciar sesión en Windows Virtual Desktop con clientes de Escritorio remoto compatibles y ver un recurso de escritorio de sesión.

Estos son los clientes compatibles actualmente:

* [Cliente de Escritorio remoto para Windows 7 y Windows 10](connect-windows-7-10-2019.md)
* [Cliente web de Windows Virtual Desktop](connect-web-2019.md)

## <a name="next-steps"></a>Pasos siguientes

Ha realizado un grupo de host y ha asignado usuarios para tener acceso al escritorio. Puede rellenar el grupo de host con programas RemoteApp. Para más información sobre cómo administrar las aplicaciones en Windows Virtual Desktop, consulte este tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Administración de grupos de aplicaciones](manage-app-groups-2019.md)
