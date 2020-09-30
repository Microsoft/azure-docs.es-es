---
title: Diagnósticos de arranque de Azure
description: Información general sobre diagnósticos de arranque de Azure y diagnósticos de arranque administrados
services: virtual-machines
ms.service: virtual-machines
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: 0b3e1b3bc296676c44eddf34b35a0d4e06d3b8c4
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90007339"
---
# <a name="azure-boot-diagnostics"></a>Diagnósticos de arranque de Azure

Diagnósticos de arranque es una característica de depuración para máquinas virtuales (VM) de Azure que permite el diagnóstico de errores de arranque de la máquina virtual. Los diagnósticos de arranque permiten a un usuario observar el estado de la máquina virtual cuando está arrancando mediante la recopilación de información de registro serie y capturas de pantallas.

## <a name="boot-diagnostics-storage-account"></a>Cuenta de almacenamiento de información de los diagnósticos de arranque
Al crear una máquina virtual en Azure Portal, los diagnósticos de arranque están habilitados de forma predeterminada. La experiencia de los diagnósticos de arranque recomendada consiste en usar una cuenta de almacenamiento administrada, ya que proporciona importantes mejoras de rendimiento en el momento de crear una máquina virtual de Azure. Esto se debe a que se usará una cuenta de almacenamiento administrada de Azure y se eliminará el tiempo necesario para crear una nueva cuenta de almacenamiento de usuario para almacenar los datos de diagnóstico de arranque.

Una experiencia alternativa de los diagnósticos de arranque consiste en usar una cuenta de almacenamiento administrada por el usuario. Un usuario puede crear una cuenta de almacenamiento o usar una existente.

> [!IMPORTANT]
> A los clientes de Azure no se les cobrará por los costos de almacenamiento asociados a los diagnósticos de arranque que usan una cuenta de almacenamiento administrada hasta octubre de 2020.

## <a name="boot-diagnostics-view"></a>Vista de diagnósticos de arranque
Situada en la hoja de la máquina virtual, la opción Diagnósticos de arranque se encuentra en la sección *Soporte técnico y solución de problemas* de Azure Portal. Al seleccionar los diagnósticos de arranque se mostrará una captura de pantalla e información de registro serie. El registro serie contiene la mensajería del kernel y la captura de pantalla es una instantánea del estado actual de las máquinas virtuales. El aspecto de la captura de pantalla esperada, depende de si la máquina virtual se ejecuta en Windows o Linux. En Windows, los usuarios verán un fondo de escritorio y en Linux, los usuarios verán un mensaje de inicio de sesión.

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-linux.png" alt-text="Captura de pantalla de diagnósticos de arranque de Linux":::
:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-windows.png" alt-text="Captura de pantalla de diagnósticos de arranque de Linux":::


## <a name="limitations"></a>Limitaciones
- Los diagnósticos de arranque solo están disponibles para máquinas virtuales de Azure Resource Manager. 
- Los diagnósticos de arranque no admiten cuentas de almacenamiento Premium. Si se usa una cuenta de almacenamiento Premium para los diagnósticos de arranque, los usuarios recibirán un error `StorageAccountTypeNotSupported` al iniciar la máquina virtual. 
- Las cuentas de almacenamiento administradas se admiten en la API de Resource Manager versión "2020-06-01" y posteriores.
- Actualmente, la consola serie de Azure no es compatible con una cuenta de almacenamiento administrada de los diagnósticos de arranque. Más información sobre la [consola serie de Azure](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/serial-console-overview).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [consola serie de Azure](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/serial-console-overview) y cómo usar los diagnósticos de arranque para [solucionar problemas de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics).
