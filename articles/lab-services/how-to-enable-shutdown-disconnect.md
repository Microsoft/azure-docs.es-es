---
title: Configuración del apagado automático de las máquinas virtuales de un laboratorio en Azure Lab Services
description: Obtenga información sobre cómo habilitar o deshabilitar el apagado automático de las máquinas virtuales cuando se desconecta una conexión a escritorio remoto.
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: 0df3fdcaea99c00461caac37a3b655d152a0e527
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798489"
---
# <a name="configure-automatic-shutdown-of-vms-for-a-lab"></a>Configuración del apagado automático de las máquinas virtuales de un laboratorio

En este artículo se muestra cómo configurar el apagado automático de las máquinas virtuales de un laboratorio.

Cuando las máquinas virtuales no se están usando, se pueden habilitar varias características de control de costos para el apagado automático, a fin de impedir costos adicionales. La combinación de las tres características siguientes de apagado y desconexión automáticos detecta la mayoría de los casos en los que los usuarios dejan funcionando sin querer sus máquinas virtuales:
 
* Desconectar automáticamente a los usuarios de las máquinas virtuales que el sistema operativo considere inactivas (solo Windows).
* Apagar automáticamente las máquinas virtuales cuando los usuarios se desconectan (Windows y Linux)
* Apagar automáticamente las máquinas virtuales que se han iniciado, pero a las que los usuarios no se conectan.

Puede encontrar más detalles sobre las características de apagado automático en la sección [Maximización del control de costos con la configuración de apagado automático](cost-management-guide.md#automatic-shutdown-settings-for-cost-control).

Un administrador de cuentas de laboratorio puede configurar esta opción para la cuenta de laboratorio en la que crea laboratorios. Para más información, consulte [Configuración del apagado automático de las máquinas virtuales de una cuenta de laboratorio](how-to-configure-lab-accounts.md). Como propietario de un laboratorio, puede invalidar la configuración al crear un laboratorio o después de su creación. 

## <a name="configure-for-the-lab-level"></a>Configuración en el nivel de laboratorio

Se puede configurar la opción de apagado automático en [Azure Lab Services](https://labs.azure.com/).

* al crear un laboratorio (en **Directivas de laboratorio**), o
* una vez creado el laboratorio (en **Configuración**).

> [!div class="mx-imgBorder"]
> ![Configuración al crear el laboratorio](./media/how-to-enable-shutdown-disconnect/configure-lab-creation.png)

Asegúrese de revisar los detalles sobre el apagado automático en la sección [Maximización del control de costos con la configuración de apagado automático](cost-management-guide.md#automatic-shutdown-settings-for-cost-control).

> [!WARNING]
> Si apaga el sistema operativo Windows (SO) en una máquina virtual antes de desconectar en ella una sesión RDP, la característica de apagado automático no funcionará correctamente.  

## <a name="next-steps"></a>Pasos siguientes

[Panel para laboratorios de clase](use-dashboard.md)
