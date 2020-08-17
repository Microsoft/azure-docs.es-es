---
title: 'Recurso compartido de contenedores de perfiles de FSLogix para Windows Virtual Desktop: Azure'
description: Configuración de un contenedor de perfiles de FSLogix para un grupo de hosts de Windows Virtual Desktop mediante un recurso compartido de archivos basado en máquinas virtuales.
author: Heidilohr
ms.topic: how-to
ms.date: 08/20/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 35b692033cc16f276b48bc6d973b27d994c1082a
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002587"
---
# <a name="create-a-profile-container-for-a-host-pool-using-a-file-share"></a>Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos

El servicio Windows Virtual Desktop ofrece contenedores de perfiles de FSLogix como solución recomendada para los perfiles de usuario. No se recomienda usar la solución Disco de perfil de usuario (UPD), que estará en desuso en futuras versiones de Windows Virtual Desktop.

En este artículo se explica cómo configurar un recurso compartido de contenedor de perfiles de FSLogix para un grupo de hosts mediante un recurso compartido de archivos basado en máquina virtual. Se recomienda encarecidamente usar Azure Files en lugar de recursos compartidos de archivos. Para más información sobre FSLogix, consulte el [sitio web de FSLogix](https://docs.fslogix.com/).

>[!NOTE]
>Si está buscando material de comparación sobre las diferentes opciones de almacenamiento del contenedor de perfiles de FSLogix en Azure, consulte [Opciones de almacenamiento para contenedores de perfiles de FSLogix](store-fslogix-profile.md).

## <a name="create-a-new-virtual-machine-that-will-act-as-a-file-share"></a>Crear una nueva máquina virtual que actúe como recurso compartido de archivos

Al crear la máquina virtual, asegúrese de colocarla en la misma red virtual que las máquinas virtuales del grupo host o en una red virtual que tenga conectividad a las máquinas virtuales del grupo host. Puede crear una máquina virtual de varias maneras:

- [Crear una máquina virtual desde una imagen de la galería de Azure](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [Crear una máquina virtual desde una imagen administrada](../virtual-machines/windows/create-vm-generalized-managed.md)
- [Crear una máquina virtual desde una imagen no administrada](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

Después de crear la máquina virtual, únala al dominio haciendo lo siguiente:

1. [Conéctese a la máquina virtual](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. En la máquina virtual, inicie el **Panel de control** y seleccione **Sistema**.
3. Seleccione **Nombre del equipo**, seleccione **Cambiar configuración** y, luego, seleccione **Cambiar…** .
4. Seleccione **Dominio** y, luego, escriba el dominio de Active Directory en la red virtual.
5. Autentíquese con una cuenta de dominio que tenga privilegios en máquinas unidas a dominio.

## <a name="prepare-the-virtual-machine-to-act-as-a-file-share-for-user-profiles"></a>Preparar la máquina virtual para que actúe como recurso compartido de archivos para los perfiles de usuario

Las siguientes son instrucciones generales sobre cómo preparar una máquina virtual para que actúe como recurso compartido de archivos para perfiles de usuario:

1. Agregue los usuarios de Active Directory de Windows Virtual Desktop a un [grupo de seguridad de Active Directory](/windows/security/identity-protection/access-control/active-directory-security-groups/). Este grupo de seguridad se usará para autenticar a los usuarios de Windows Virtual Desktop en la máquina de virtual del recurso compartido de archivos que acaba de crear.
2. [Conéctese a la máquina virtual del recurso compartido de archivos](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine).
3. En la máquina virtual del recurso compartido de archivos, cree una carpeta en la **unidad C**, que se usará como recurso compartido de perfiles.
4. Haga clic con el botón derecho en la nueva carpeta, seleccione **Propiedades**, seleccione **Uso compartido**, luego, seleccione **Uso compartido avanzado…**
5. Seleccione **Compartir esta carpeta**, seleccione **Permisos…** , luego, seleccione **Agregar…**
6. Busque el grupo de seguridad al que agregó los usuarios de Windows Virtual Desktop, luego, asegúrese de que ese grupo tenga **Control total**.
7. Después de agregar el grupo de seguridad, haga clic con el botón derecho en la carpeta, seleccione **Propiedades**, seleccione **Uso compartido**, luego, copie la **Ruta de acceso de red** para consultar más adelante.

Para más información acerca de los permisos, consulte la [documentación de FSLogix](/fslogix/fslogix-storage-config-ht/).

## <a name="configure-the-fslogix-profile-container"></a>Configurar el contenedor de perfiles de FSLogix

Para configurar las máquinas virtuales con el software de FSLogix, siga este procedimiento en cada máquina registrada en el grupo host:

1. [Conéctese a la máquina virtual](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. Inicie un explorador de Internet y vaya a [este vínculo](https://go.microsoft.com/fwlink/?linkid=2084562) para descargar el agente de FSLogix.
3. Vaya a \\\\Win32\\Release o \\\\X64\\Release en el archivo .zip y ejecute **FSLogixAppsSetup** para instalar el agente de FSLogix.  Para más información sobre cómo instalar FSLogix, consulte [Descarga e instalación de FSLogix](/fslogix/install-ht/).
4. Vaya a **Archivos de programa** > **FSLogix** > **Apps** para confirmar que el agente esté instalado.
5. Desde el menú de Inicio, ejecute **RegEdit** como administrador. Vaya a **Equipo\\HKEY_LOCAL_MACHINE\\software\\FSLogix**.
6. Cree una clave denominada **Profiles**.
7. Cree los siguientes valores para la clave Profiles:

| Nombre                | Tipo               | Datos/valor                        |
|---------------------|--------------------|-----------------------------------|
| habilitado             | DWORD              | 1                                 |
| VHDLocations        | Valor de varias cadenas | "Network path for file share"     |

>[!IMPORTANT]
>Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host. Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar [acceso de máquina virtual Just-in-Time](../security-center/security-center-just-in-time.md).
