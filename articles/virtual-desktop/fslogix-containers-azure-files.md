---
title: 'Archivos de contenedores de perfiles de FSLogix para Windows Virtual Desktop: Azure'
description: Este artículo describe los contenedores de perfiles de FSLogix dentro de Windows Virtual Desktop y Azure Files.
author: Heidilohr
ms.topic: conceptual
ms.date: 08/07/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 669f4baa723b78b8933f3a75fc361c468f9e2df9
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002385"
---
# <a name="fslogix-profile-containers-and-azure-files"></a>Contenedores de perfiles de FSLogix y archivos de Azure

El servicio Windows Virtual Desktop recomienda contenedores de perfiles de FSLogix como solución para los perfiles de usuario. FSLogix está diseñado para itinerar perfiles en entornos informáticos remotos, como Windows Virtual Desktop. Almacena un perfil de usuario completo en un único contenedor. Al iniciar sesión, este contenedor se adjunta dinámicamente al entorno informático mediante el disco duro virtual (VHD) compatible de forma nativa y el disco duro virtual de Hyper-V (VHDX). El perfil de usuario está disponible inmediatamente y aparece en el sistema exactamente como un perfil de usuario nativo. En este artículo se describe cómo se usan los contenedores de perfiles de FSLogix con la función de Azure Files en Windows Virtual Desktop.

>[!NOTE]
>Si está buscando material de comparación sobre las diferentes opciones de almacenamiento del contenedor de perfiles de FSLogix en Azure, consulte [Opciones de almacenamiento para contenedores de perfiles de FSLogix](store-fslogix-profile.md).

## <a name="user-profiles"></a>Perfiles de usuario

Un perfil de usuario contiene elementos de datos sobre una persona, incluida información de configuración, como la configuración del escritorio, las conexiones de red persistente y la configuración de la aplicación. De forma predeterminada, Windows crea un perfil de usuario local que se integra estrechamente con el sistema operativo.

Un perfil de usuario remoto proporciona una partición entre el sistema operativo y los datos de usuario. Esto permite reemplazar o cambiar el sistema operativo sin afectar a los datos de usuario. En el host de sesión de Escritorio remoto (RDSH) y las infraestructuras de escritorio virtual (VDI), el sistema operativo puede reemplazarse por las razones siguientes:

- Una actualización del sistema operativo.
- Una sustitución de una máquina virtual existente.
- Un usuario que forma parte de un entorno de RDSH o VDI agrupado (no persistente).

Los productos de Microsoft funcionan con varias tecnologías para perfiles de usuario remoto, incluidas las siguientes:
- Perfiles de usuario móvil (RUP)
- Discos de perfil de usuario (UPD)
- Enterprise State Roaming (ESR)

UPD y RUP son las tecnologías más usadas para perfiles de usuario en entornos de host de sesión de Escritorio remoto (RDSH) y disco duro virtual (VHD).

### <a name="challenges-with-previous-user-profile-technologies"></a>Desafíos con tecnologías de perfil de usuario anteriores

Las soluciones de Microsoft para perfiles de usuario existentes y obsoletas presentaban varios desafíos. Ninguna solución anterior satisfacía todas las necesidades asociadas a un entorno de RDSH o VDI. Por ejemplo, UPD no puede controlar archivos OST de gran tamaño y RUP no conservar configuraciones modernas.

#### <a name="functionality"></a>Funcionalidad

La siguiente tabla muestra las ventajas y limitaciones de las tecnologías de perfiles de usuario anteriores.

| Technology | Configuración moderna | Configuración de Win32 | Configuración del SO | Datos de usuario | Compatible con SKU del servidor | Almacenamiento de back-end en Azure | Almacenamiento de back-end local | Compatibilidad con versiones | Tiempo de inicio de sesión posterior |Notas|
| ---------- | :-------------: | :------------: | :---------: | --------: | :---------------------: | :-----------------------: | :--------------------------: | :-------------: | :---------------------: |-----|
| **Discos de perfil de usuario (UPD)** | Sí | Sí | Sí | Sí | Sí | No | Sí | Win 7+ | Sí | |
| **Perfil de usuario móvil (RUP), modo de mantenimiento** | No | Sí | Sí | Sí | Sí| No | Sí | Win 7+ | No | |
| **Enterprise State Roaming (ESR)** | Sí | No | Sí | No | Vea las notas | Sí | No | Windows 10 | No | Funciones en la SKU del servidor, pero sin interfaz de usuario de soporte |
| **User Experience Virtualization (UE-V)** | Sí | Sí | Sí | No | Sí | No | Sí | Win 7+ | No |  |
| **Archivos en la nube de OneDrive** | No | No | No | Sí | Vea las notas | Vea las notas  | Vea las notas | Win 10 RS3 | No | No probado en la SKU del servidor. El almacenamiento de back-end en Azure depende del cliente de sincronización. El almacenamiento de back-end local necesita un cliente de sincronización. |

#### <a name="performance"></a>Rendimiento

UPD requiere [espacios de almacenamiento directo (S2D)](/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment/) para satisfacer los requisitos de rendimiento. UPD usa el protocolo de bloque de mensajes del servidor (SMB). Copia el perfil a la máquina virtual en la que se está registrando el usuario. UPD con S2D es la solución que se recomienda para Windows Virtual Desktop.

#### <a name="cost"></a>Coste

Aunque los clústeres de S2D logran el rendimiento necesario, el costo es caro para los clientes empresariales, pero especialmente costosa para los clientes de pequeñas y medianas empresas (PYMES). Para esta solución, las empresas pagan por los discos de almacenamiento, junto con el costo de las máquinas virtuales que usan los discos para un recurso compartido.

#### <a name="administrative-overhead"></a>Sobrecarga administrativa

Los clústeres de S2D requieren un sistema operativo revisado, actualizado y mantenido en un estado seguro. Estos procesos y la complejidad de la configuración de la recuperación ante desastres de S2D hacen que S2D solo sea viable para empresas con personal de TI dedicado.

## <a name="fslogix-profile-containers"></a>Contenedores de perfiles de FSLogix

El 19 de noviembre de 2018 [Microsoft adquirió FSLogix](https://blogs.microsoft.com/blog/2018/11/19/microsoft-acquires-fslogix-to-enhance-the-office-365-virtualization-experience/). FSLogix aborda muchos desafíos de contenedores de perfiles. Los siguientes son los principales:

- **Rendimiento:** Los [contenedores de perfiles de FSLogix](/fslogix/configure-profile-container-tutorial/) son de alto rendimiento y resuelven problemas de rendimiento que históricamente bloqueaban el modo caché de Exchange.
- **OneDrive:** Sin contenedores de perfiles de FSLogix, OneDrive para la Empresa no se admite en entornos de RDSH o VDI no persistentes. Los [procedimientos recomendados de OneDrive para la Empresa y FSLogix](/fslogix/overview/) describen cómo interactúan. Para más información, consulte [Uso del cliente de sincronización en escritorios virtuales](/deployoffice/rds-onedrive-business-vdi/).
- **Carpetas adicionales:** FSLogix proporciona la capacidad de ampliar los perfiles de usuario para incluir carpetas adicionales.

Desde su adquisición, Microsoft comenzó a reemplazar las soluciones de perfiles de usuario existentes, como UPD, por contenedores de perfiles de FSLogix.

## <a name="azure-files-integration-with-azure-active-directory-domain-service"></a>Integración de Azure Files con Azure Active Directory Domain Service

El rendimiento y las características de los contenedores de perfiles de FSLogix aprovechan las ventajas de la nube. El 7 de agosto de 2019, Microsoft Azure Files anunció la disponibilidad general de la autenticación de [Azure files con Azure Active Directory Domain Services (AD DS)](../storage/files/storage-files-active-directory-overview.md). Al resolver tanto la sobrecarga administrativa como la de costos, Azure Files con autenticación de Azure AD DS es una solución premium para perfiles de usuario en el nuevo servicio Windows Virtual Desktop.

## <a name="best-practices-for-windows-virtual-desktop"></a>Procedimientos recomendados para Windows Virtual Desktop

Windows Virtual Desktop ofrece un control total sobre el tamaño, el tipo y el número de máquinas virtuales que utilizan los clientes. Para más información, consulte [¿Qué es Windows Virtual Desktop?](overview.md).

Para asegurarse de que su entorno de Windows Virtual Desktop sigue los procedimientos recomendados:

- La cuenta de almacenamiento de Azure Files debe estar en la misma región que las máquinas virtuales del host de sesión.
- Los permisos de Azure Files deben coincidir con los permisos descritos en [Requisitos - Contenedores de perfiles](/fslogix/fslogix-storage-config-ht).
- Cada grupo host debe estar creado con una máquina virtual del mismo tipo y tamaño basada en la misma imagen maestra.
- Cada máquina virtual de grupo host debe encontrarse en el mismo grupo de recursos para facilitar la administración, el escalado y la actualización.
- Para obtener un rendimiento óptimo, la solución de almacenamiento y el contenedor de perfiles de FSLogix deben estar en la misma ubicación del centro de datos.
- La cuenta de almacenamiento que contiene la imagen maestra debe estar en la misma región y suscripción donde se aprovisionan las máquinas virtuales.

## <a name="next-steps"></a>Pasos siguientes

Utilice las siguientes instrucciones para configurar un entorno de Windows Virtual Desktop.

- Para empezar a compilar su solución de virtualización de escritorio, consulte [Creación de un inquilino en Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).
- Para crear un grupo host dentro de su inquilino de Windows Virtual Desktop, consulte [Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace.md).
- Para configurar recursos compartidos de archivos totalmente administrados en la nube, consulte [Configuración de recurso compartido de Azure Files](/azure/storage/files/storage-files-active-directory-enable/).
- Para configurar contenedores de perfiles de FSLogix, consulte [Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos](create-host-pools-user-profile.md).
- Para asignar usuarios a un grupo host, consulte [Administración de grupos de aplicaciones de Windows Virtual Desktop](manage-app-groups.md).
- Para acceder a los recursos de Windows Virtual Desktop desde un explorador web, consulte [Conexión a Windows Virtual Desktop](connect-web.md).
