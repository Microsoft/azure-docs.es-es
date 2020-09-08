---
title: Diagnósticos de arranque de Azure
description: Información general sobre diagnósticos de arranque de Azure y diagnósticos de arranque administrados
services: virtual-machines
ms.service: virtual-machines
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: e2ba5d909a3aa43921f52295d2f7216aac76bc32
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067093"
---
# <a name="azure-boot-diagnostics"></a>Diagnósticos de arranque de Azure

Diagnósticos de arranque es una característica de depuración para máquinas virtuales (VM) de Azure que permite el diagnóstico de errores de arranque de la máquina virtual. Los diagnósticos de arranque permiten a un usuario observar el estado de la máquina virtual cuando está arrancando mediante la recopilación de información de registro serie y capturas de pantallas.

## <a name="boot-diagnostics-view"></a>Vista de diagnósticos de arranque
Situada en la hoja de la máquina virtual, la opción Diagnósticos de arranque se encuentra en la sección *Soporte técnico y solución de problemas* de Azure Portal. Al seleccionar los diagnósticos de arranque se mostrará una captura de pantalla e información de registro serie. El registro serie contiene la mensajería del kernel y la captura de pantalla es una instantánea del estado actual de las máquinas virtuales. El aspecto de la captura de pantalla esperada, depende de si la máquina virtual se ejecuta en Windows o Linux. En Windows, los usuarios verán un fondo de escritorio y en Linux, los usuarios verán un mensaje de inicio de sesión.

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-linux.png" alt-text="Captura de pantalla de diagnósticos de arranque de Linux":::
:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-windows.png" alt-text="Captura de pantalla de diagnósticos de arranque de Windows":::


## <a name="limitations"></a>Limitaciones
- Los diagnósticos de arranque solo están disponibles para máquinas virtuales de Azure Resource Manager. 
- Los diagnósticos de arranque no admiten cuentas de almacenamiento Premium. Si se usa una cuenta de almacenamiento Premium para los diagnósticos de arranque, los usuarios recibirán un error `StorageAccountTypeNotSupported` al iniciar la máquina virtual. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [consola serie de Azure](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/serial-console-overview) y cómo usar los diagnósticos de arranque para [solucionar problemas de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics).
