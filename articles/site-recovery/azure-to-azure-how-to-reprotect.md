---
title: Volver a proteger las máquinas virtuales de Azure en una región primaria con Azure Site Recovery | Microsoft Docs
description: Se describe cómo volver a proteger las máquinas virtuales de Azure después de la conmutación por error de una región secundaria a una primaria con Azure Site Recovery.
services: site-recovery
author: Rajeswari-Mamilla
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 11/27/2018
ms.author: ramamill
ms.openlocfilehash: c3e4b1c97c5fcc86a863313c2d6f54d93f26390e
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055048"
---
# <a name="reprotect-failed-over-azure-vms-to-the-primary-region"></a>Reprotección de máquinas virtuales de Azure conmutadas por error en la región principal

Al [conmutar por error](site-recovery-failover.md) las máquinas virtuales de Azure desde una región a otra con [Azure Site Recovery](site-recovery-overview.md), las máquinas virtuales se inician en la región secundaria, con un estado **desprotegido**. Si quiere conmutar por recuperación las máquinas virtuales en la región primaria, realice las tareas siguientes:

1. Vuelva a proteger las máquinas virtuales en la región secundaria, para que empiecen a replicarse en la región primaria.
1. Una vez completada la reprotección y después de que las máquinas virtuales se estén replicando, puede realizar una conmutación por error desde la región secundaria a la primaria.

## <a name="prerequisites"></a>Prerrequisitos

- La conmutación por error de la máquina virtual de la región principal a la secundaria se debe confirmar.
- El sitio de destino principal debe estar disponible y, además, debe tener la posibilidad de acceder o crear recursos en dicha región.

## <a name="reprotect-a-vm"></a>Reprotección de una máquina virtual

1. En **Almacén** > **Elementos replicados**, haga clic con el botón derecho en la máquina virtual que se conmutó por error y, luego, seleccione **Volver a proteger**. La dirección de reprotección debe mostrarse desde la ubicación secundaria a la principal.

   ![Reprotección](./media/site-recovery-how-to-reprotect-azure-to-azure/reprotect.png)

1. Revise el grupo de recursos, la red, el almacenamiento y los conjuntos de disponibilidad. A continuación, haga clic en **Aceptar**. Si hay algún recurso marcado como nuevo, se crea como parte del proceso de reprotección.
1. Este trabajo de reprotección inicializa el sitio de destino con los datos más recientes. Una vez finalizado el proceso, se produce la replicación diferencial. Después, puede conmutar por recuperación al sitio principal. Puede seleccionar la cuenta de almacenamiento o la red que desea usar durante la reprotección mediante la opción de personalización.

   ![Opción de personalización](./media/site-recovery-how-to-reprotect-azure-to-azure/customize.png)

### <a name="customize-reprotect-settings"></a>Personalización de la configuración de reprotección

Puede personalizar las siguientes propiedades de la máquina virtual de destino durante la reprotección.

![Personalizar](./media/site-recovery-how-to-reprotect-azure-to-azure/customizeblade.png)

|Propiedad |Notas  |
|---------|---------|
|Grupo de recursos de destino | Modifique el grupo de recursos de destino en que se crea la máquina virtual. Como parte de la reprotección, se elimina la máquina virtual de destino. Puede elegir un grupo de recursos nuevo en el que crear la máquina virtual tras la conmutación por error. |
|Red virtual de destino | No se puede cambiar la red de destino durante el trabajo de reprotección. Para cambiar la red, vuelva a hacer la asignación de red. |
|Almacenamiento de destino (la máquina virtual secundaria no utiliza discos administrados) | Puede cambiar la cuenta de almacenamiento que la máquina virtual usa tras la conmutación por error. |
|Discos administrados de réplica (la VM secundaria utiliza discos administrados) | Site Recovery crea discos administrado de réplica en la región principal para crear el reflejo de discos administrados de la VM secundaria. |
|Almacenamiento en caché | Puede especificar una cuenta de almacenamiento en caché para usarla durante la replicación. De forma predeterminada, se crea una cuenta de almacenamiento en caché en caso de que no exista. |
|Conjunto de disponibilidad | Si la máquina virtual de la región secundaria forma parte de un conjunto de disponibilidad, puede elegir un conjunto de disponibilidad para la máquina virtual de destino en la región principal. De forma predeterminada, Site Recovery intenta encontrar el conjunto de disponibilidad existente en la región principal para usarlo. Durante la personalización, puede especificar un nuevo conjunto de disponibilidad. |

### <a name="what-happens-during-reprotection"></a>¿Qué ocurre durante la reprotección?

De forma predeterminada, sucede lo siguiente:

1. Se crea una cuenta de almacenamiento en caché en la región donde se está ejecutando la máquina virtual conmutada por error.
1. Si la cuenta de almacenamiento de destino (la cuenta de almacenamiento original de la región principal) no existe, se crea una. El nombre asignado para la cuenta de almacenamiento se corresponde con el nombre de la cuenta de almacenamiento que usa la máquina virtual secundaria, con el sufijo `asr`.
1. Si la máquina virtual utiliza discos administrados, los discos administrados de réplica se crean en la región principal para almacenar los datos replicados de discos de la máquina virtual secundaria.
1. Si el conjunto de disponibilidad de destino no existe, se crea uno como parte del trabajo de reprotección, en caso de que sea necesario. Si personalizó la configuración de reprotección, en este caso se usa el conjunto seleccionado.

Cuando se desencadena un trabajo de reprotección y la máquina virtual de destino ya existe, ocurre lo siguiente:

1. La máquina virtual de destino se apaga en caso de que esté en ejecución.
1. Si la máquina virtual está usando discos administrados, se crea una copia del disco original con el sufijo `-ASRReplica`. Los discos originales se eliminan. Las copias de `-ASRReplica` se usan para la replicación.
1. Si la máquina virtual usa discos no administrados, los discos de datos de la máquina virtual de destino se separan y se usan para la replicación. Se crea una copia del disco del sistema operativo y se adjunta a la máquina virtual. El disco del sistema operativo original se separa y se usa para la replicación.
1. Solo se sincronizan los cambios entre el disco de origen y el disco de destino. Las copias de seguridad diferenciales se calculan comparando los discos y, a continuación, se transfieren. Consulte a continuación para averiguar el tiempo estimado para completar la reprotección.
1. Cuando finaliza el trabajo de sincronización, comienza la replicación diferencial y se crea un punto de recuperación según la directiva de replicación.

Cuando se desencadena un trabajo de reprotección y la máquina virtual y los discos de destino no existen, ocurre lo siguiente:

1. Si usa discos administrados en la máquina virtual, los discos de réplica se crean con el sufijo `-ASRReplica`. Las copias de `-ASRReplica` se usan para la replicación.
1. Si usa discos no administrados en la máquina virtual, los discos de réplica se crean en la cuenta de almacenamiento de destino.
1. Todos los discos se copian de la región que ha conmutado por error a la nueva región de destino.
1. Cuando finaliza el trabajo de sincronización, comienza la replicación diferencial y se crea un punto de recuperación según la directiva de replicación.

#### <a name="estimated-time-to-do-the-reprotection"></a>Tiempo estimado para realizar la reprotección

En la mayoría de los casos, Azure Site Recovery no replica los datos completos en la región de origen.
Las siguientes condiciones determinan la cantidad de datos que se replican:

1. Si los datos de la máquina virtual de origen se eliminan, están dañados o no son accesibles por algún motivo, como el cambio o la eliminación del grupo de recursos, durante la reprotección se produce una replicación inicial completa porque no hay datos disponibles para usar en la región de origen.
1. Si los datos de la máquina virtual de origen son accesibles, solo se calculan los diferenciales al comparar ambos discos y luego se transfieren. Consulte la tabla siguiente para obtener el tiempo estimado.

|Situación de ejemplo | Tiempo necesario para reproteger |
|---|---|
|La región de origen tiene una máquina virtual con un disco estándar de 1 TB.<br/>Solo se usan 127 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es estándar con un rendimiento de 60 MBps.<br/>Ningún cambio de datos después de la conmutación por error.| Tiempo aproximado: De 60 a 90 minutos.<br/> Durante la reprotección, Site Recovery rellenará la suma de comprobación de todos los datos. Esto funciona a 45 MBps, por lo que el tiempo total que tardará es de 127 GB/45 MBps, aproximadamente 45 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos. |
|La región de origen tiene una máquina virtual con un disco estándar de 1 TB.<br/>Solo se usan 127 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es estándar con un rendimiento de 60 MBps.<br/>45 GB de datos cambian después de la conmutación por error.| Tiempo aproximado: De 2,5 a 3 horas.<br/> Durante la reprotección, Site Recovery rellenará la suma de comprobación de todos los datos. Esto funciona a 45 MBps, por lo que el tiempo total que tardará es de 127 GB/45 MBps, aproximadamente 45 minutos.<br/>La velocidad de transferencia es aproximadamente un 16 % del rendimiento; lo que supone 9,6 MBps. Por consiguiente, el tiempo de transferencia para aplicar los cambios es de 45 GB, es decir, 45 GB/9,6 MBps, aproximadamente 80 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos. |
|La región de origen tiene una máquina virtual con un disco estándar de 1 TB.<br/>Solo se usan 20 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es estándar con un rendimiento de 60 MBps.<br/>Los datos iniciales del disco inmediatamente después de la conmutación por error eran 15 GB. Hubo un cambio de 5 GB de datos después de la conmutación por error. Por consiguiente, el total de datos rellenados es de 20 GB.| Tiempo aproximado: De 1 a 1,5 horas.<br/>Dado que los datos rellenados en el disco son menos del 10 % del tamaño del disco, realizamos una replicación inicial completa.<br/> La velocidad de transferencia es aproximadamente un 16 % del rendimiento, es decir, 9,6 MBps. Por consiguiente, el tiempo de transferencia para aplicar los cambios es de 20 GB, es decir, 20 GB/9,6 MBps, aproximadamente 36 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos. |
|La región de origen tiene una máquina virtual con un disco Premium de 1 TB.<br/>Solo se usan 127 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es Premium con un rendimiento de 200 MBps.<br/>Ningún cambio de datos después de la conmutación por error.| Tiempo aproximado: 2 horas.<br/>Durante la reprotección, Site Recovery rellenará la suma de comprobación de todos los datos. Esto funciona a 25 MBps (hasta el 16 % del rendimiento del disco), por lo que el tiempo total que tardará es de 127 GB/25 MBps, aproximadamente 87 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos. |
|La región de origen tiene una máquina virtual con un disco Premium de 1 TB.<br/>Solo se usan 127 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es Premium con un rendimiento de 200 MBps.<br/>45 GB de datos cambian después de la conmutación por error.| Tiempo aproximado: De 2,5 a 3 horas.<br/>Durante la reprotección, Site Recovery rellenará la suma de comprobación de todos los datos. Esto funciona a 25 MBps (hasta el 16 % del rendimiento del disco), por lo que el tiempo total que tardará es de 127 GB/25 MBps, aproximadamente 87 minutos.</br>La velocidad de transferencia es aproximadamente un 16 % del rendimiento, es decir, 32 MBps. Por consiguiente, el tiempo de transferencia para aplicar los cambios es de 45 GB, es decir, 45 GB/32 MBps, aproximadamente 24 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos. |
|La región de origen tiene una máquina virtual con un disco Premium de 1 TB.<br/>Solo se usan 20 GB de datos y el resto del disco está vacío.<br/>El tipo de disco es Premium con un rendimiento de 200 MBps.<br/>Los datos iniciales del disco inmediatamente después de la conmutación por error eran 15 GB. Hubo un cambio de 5 GB de datos después de la conmutación por error. Por consiguiente, el total de datos rellenados es de 20 GB.| Tiempo aproximado: De 30 a 45 minutos.<br/>Dado que los datos rellenados en el disco son menos del 10 % del tamaño del disco, realizamos una replicación inicial completa.<br/>La velocidad de transferencia es aproximadamente un 16 % del rendimiento, es decir, 32 MBps. Por consiguiente, el tiempo de transferencia para aplicar los cambios es de 20 GB, es decir, 20 GB/32 MBps, aproximadamente 11 minutos.<br/>Se necesita algún tiempo de sobrecarga para que Site Recovery realice el escalado automático, aproximadamente entre 20 y 30 minutos |

Cuando se vuelve a proteger la VM después de la conmutación por recuperación en la región primaria (es decir, si la VM se vuelve a proteger desde la región primaria a la región de DR), se eliminan la VM de destino y las NIC asociadas.

Cuando la máquina virtual se vuelve a proteger desde la región de recuperación ante desastres a la región primaria, no se elimina la máquina virtual principal anterior ni las NIC asociadas.

## <a name="next-steps"></a>Pasos siguientes

Una vez protegida la máquina virtual, se puede iniciar una conmutación por error. La conmutación por error apaga la máquina virtual en la región secundaria y se crea y arranca la máquina virtual en la región primaria, con un breve tiempo de inactividad durante el proceso. Se recomienda elegir una hora adecuada para el proceso y ejecutar una prueba de conmutación por error antes de iniciar una conmutación por error completa al sitio principal. [Más información](site-recovery-failover.md) sobre la conmutación por error de Azure Site Recovery
