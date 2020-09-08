---
title: Problemas comunes durante la creación de discos duros virtuales (P+F)
description: Preguntas más frecuentes sobre problemas comunes al crear un disco duro virtual (VHD)
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: guide
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 04/09/2020
ms.openlocfilehash: f4d30cdb931d6523eba3aac003caeee38a8c024d
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653535"
---
# <a name="common-issues-during-vhd-creation"></a>Problemas comunes durante la creación de discos duros virtuales

Estas preguntas más frecuentes (P+F) abordan problemas comunes que pueden surgir al crear un disco duro virtual (VHD) para la oferta de máquinas virtuales de Azure.

## <a name="how-do-i-create-a-vm-from-the-azure-portal-using-a-vhd-in-premium-storage"></a>¿Cómo se crea una máquina virtual desde Azure Portal mediante un disco duro virtual en Premium Storage?

En la actualidad, Azure Marketplace no admite la creación de ofertas de máquina virtual a partir de imágenes del almacenamiento administrado o desde Azure Premium Storage. Para información detallada, vea [Introducción a Azure Managed Disks](../../virtual-machines/managed-disks-overview.md).

## <a name="can-i-use-generation-2-vms-for-offers"></a>¿Se pueden usar máquinas virtuales de generación 2 para las ofertas?

No, solo se admiten los discos duros virtuales de generación 1. Pero, en la actualidad, estamos trabajando con el equipo de la plataforma de Microsoft Azure para investigar la compatibilidad con las máquinas virtuales de generación 2. Para información detallada, vea [¿Debería crear una máquina virtual de generación 1 o 2 en Hyper-V?](https://docs.microsoft.com/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)

## <a name="how-do-i-change-the-name-of-the-host"></a>¿Cómo se cambia el nombre del host?

No puede. Una vez creada la máquina virtual, los usuarios (incluidos los propietarios) no pueden actualizar el nombre de host.

## <a name="how-do-i-reset-the-remote-desktop-service-or-its-sign-in-password"></a>¿Cómo se restablece el servicio Escritorio remoto o su contraseña de inicio de sesión?

En estos artículos se explica cómo realizar el restablecimiento de RDS para máquinas virtuales basadas en Windows y Linux:

* [Restablecimiento del servicio Escritorio remoto o la contraseña de inicio de sesión en una máquina virtual Windows](/azure/virtual-machines/troubleshooting/reset-rdp)
* [Cómo restablecer una contraseña de máquina virtual Linux o clave SSH, corregir la configuración de SSH y comprobar la coherencia de disco con la extensión VMAccess](/azure/virtual-machines/extensions/vmaccess)

## <a name="how-do-i-generate-new-ssh-certificates"></a>¿Cómo se generan certificados SSH nuevos?

La generación de certificados se explica en [Certificación de imágenes de máquina virtual de Azure](https://aks.ms/CertifyVMimage).

## <a name="how-do-i-configure-a-virtual-private-network-vpn-to-work-with-my-vms"></a>¿Cómo se configura una red privada virtual (VPN) para trabajar con las máquinas virtuales?

Si usa el modelo de implementación de Azure Resource Manager, tiene tres opciones:

* [Creación de una instancia de VPN Gateway basada en rutas mediante Azure Portal](../../vpn-gateway/create-routebased-vpn-gateway-portal.md)
* [Creación de una instancia de VPN Gateway basada en rutas mediante Azure PowerShell](../../vpn-gateway/create-routebased-vpn-gateway-powershell.md)
* [Creación de una instancia de VPN Gateway basada en rutas mediante la CLI](../../vpn-gateway/create-routebased-vpn-gateway-cli.md)

## <a name="what-are-microsoft-support-policies-for-running-microsoft-server-software-on-azure-based-vms"></a>¿Cuáles son las directivas de soporte técnico de Microsoft para ejecutar software de servidor de Microsoft en máquinas virtuales basadas en Azure?

Puede encontrar información detallada en [Soporte técnico del software de servidor de Microsoft para máquinas virtuales de Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

## <a name="do-virtual-machines-have-unique-identifiers-associated-with-them"></a>¿Las máquinas virtuales tienen identificadores únicos asociados?

Sí, si se hospedan en Azure. Azure asigna un identificador único, denominado [identificador único de máquina virtual de Azure](https://blogs.msdn.microsoft.com/wasimbloch/2016/10/20/azure-virtual-machine-unique-id/), a cada recurso de máquina virtual creado. Para información detallada, vea Identificador único de máquina virtual de Azure. También puede obtener este identificador mediante [List API](https://docs.microsoft.com/rest/api/compute/virtualmachines/list).

## <a name="in-a-vm-how-do-i-manage-the-custom-script-extension-in-the-startup-task"></a>¿Cómo se puede administrar la extensión de script personalizada en la tarea de inicio en una máquina virtual?

Para obtener información detallada sobre cómo usar la extensión de script personalizado mediante el módulo de Azure PowerShell y plantillas de Azure Resource Manager, y también se detallan los pasos para solucionar problemas en los sistemas Windows, vea [Extensión de script personalizado para Windows](/azure/virtual-machines/extensions/custom-script-windows).

## <a name="are-32-bit-applications-or-services-supported-in-azure-marketplace"></a>¿Se admiten en Azure Marketplace las aplicaciones o los servicios de 32 bits?

En general, no. Todos los sistemas operativos compatibles y los servicios estándar para las máquinas virtuales de Azure son de 64 bits. Aunque la mayoría de los sistemas operativos de 64 bits admiten versiones de las aplicaciones de 32 bits para mantener la compatibilidad con versiones anteriores; el uso de aplicaciones de 32 bits como parte de la solución de máquina virtual no se admite y no se recomienda. Vuelva a crear la aplicación como un proyecto de 64 bits.

Para obtener más información, consulte estos artículos:

* [Running 32-bit applications](https://docs.microsoft.com/windows/desktop/WinProg64/running-32-bit-applications) (Ejecución de aplicaciones de 32 bits)
* [Soporte para sistemas operativos de 32 bits en máquinas virtuales de Azure](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines)
* [Soporte técnico del software de servidor de Microsoft para máquinas virtuales de Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)

## <a name="error-vhd-is-already-registered-with-image-repository-as-the-resource"></a>Error: El disco duro virtual ya está registrado con el repositorio de imágenes como un recurso.

Cada vez que trato de crear una imagen desde el VHD, se produce el error "VHD ya está registrado con el repositorio de imágenes como un recurso" en Azure PowerShell. No he creado ninguna imagen antes ni he encontrado ninguna imagen con este nombre en Azure. ¿Cómo se resuelve este problema?

Este problema suele aparecer si ha creado una máquina virtual desde un disco duro virtual que tiene un bloqueo. Asegúrese de que no hay ninguna máquina virtual asignada desde este disco duro virtual y, después, vuelva a intentar la operación. Si el problema continúa, abra una incidencia de soporte técnico. Vea [Soporte técnico para el Centro de partners](support.md).
