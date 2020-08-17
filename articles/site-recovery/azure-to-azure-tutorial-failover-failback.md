---
title: Conmutación por error y reprotección de las máquinas virtuales de Azure replicadas en una región secundaria de Azure para la recuperación ante desastres con el servicio Azure Site Recovery.
description: Obtenga información sobre la conmutación por error y la reprotección de las máquinas virtuales de Azure replicadas en una región secundaria de Azure para la recuperación ante desastres con el servicio Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 08/05/2019
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 8d38aa513b0829c2626fcd4a92c40faabff1f83e
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502399"
---
# <a name="fail-over-and-reprotect-azure-vms-between-regions"></a>Conmutación por error y reprotección de máquinas virtuales de Azure entre regiones

En este tutorial se describe cómo conmutar por error una máquina virtual (VM) de Azure en una región secundaria de Azure con el servicio [Azure Site Recovery](site-recovery-overview.md). Una vez realizada la conmutación por error, volverá a proteger la máquina virtual. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conmutar por error la máquina virtual de Azure
> * Volver a proteger la máquina virtual de Azure secundaria, de modo que se replique en la región principal.

> [!NOTE]
> Este tutorial describe la ruta más sencilla, con las opciones predeterminadas y la mínima personalización. Para escenarios más complejos, use los artículos de Guías de procedimientos para máquinas virtuales de Azure.


## <a name="prerequisites"></a>Prerrequisitos

- Antes de empezar, repase las [preguntas más frecuentes](site-recovery-faq.md#failover) sobre la conmutación por error.
- Asegúrese de que ha completado una [exploración de la recuperación ante desastres](azure-to-azure-tutorial-dr-drill.md) para comprobar que todo funciona según lo previsto.
- Compruebe las propiedades de la máquina virtual antes de ejecutar la conmutación por error de prueba. La máquina virtual debe cumplir los [requisitos de Azure](azure-to-azure-support-matrix.md#replicated-machine-operating-systems).

## <a name="run-a-failover-to-the-secondary-region"></a>Ejecución de una conmutación por error en la región secundaria

1. En **Elementos replicados**, seleccione la máquina virtual que desea conmutar por error > **Conmutar por error**

   ![Captura de pantalla que muestra las opciones de Conmutación por error de una máquina virtual.](./media/azure-to-azure-tutorial-failover-failback/failover.png)

2. En **Conmutación por error**, seleccione un **Punto de recuperación** en el que realizar la conmutación por error. Puede seleccionar una de las siguientes opciones:

   * **Latest** (Más reciente) (valor predeterminado): procesa todos los datos en el servicio Site Recovery y proporciona el objetivo de punto de recuperación (RPO) más bajo.
   * **Procesado más recientemente**: revierte la máquina virtual al punto de recuperación más reciente que el servicio Site Recovery haya procesado.
   * **Personalizado**: conmuta por error a un punto de recuperación concreto. Esta opción es útil para realizar una conmutación por error de prueba.

3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error** si desea que Site Recovery intente apagar las máquinas virtuales de origen antes de desencadenar la conmutación por error. La operación de apagado ayuda a garantizar que no se pierden datos. La conmutación por error continúa aunque se produzca un error de cierre. Site Recovery no es un origen limpio después de la conmutación por error.

4. Siga el progreso de la conmutación por error en la página **Trabajos**.

5. Después de la conmutación por error, valide la máquina virtual iniciando sesión en ella. Si desea volver a otro punto de recuperación para la máquina virtual, puede usar la opción **Cambiar punto de recuperación**.

6. Realice la acción **Confirmar** una vez que quede satisfecho con la conmutación por error de la máquina virtual.
   Al confirmar, se eliminan todos los puntos de recuperación disponibles con el servicio. Ahora no podrá cambiar el punto de recuperación.

> [!NOTE]
> Cuando se conmuta por error una máquina virtual a la que agrega un disco después de habilitar la replicación para la máquina virtual, los puntos de la replicación mostrarán los discos que están disponibles para la recuperación. Por ejemplo, si una máquina virtual tiene un único disco y agrega uno nuevo, los puntos de replicación que se crearon antes de agregar el disco mostrarán que el punto de replicación se compone de "1 de 2 discos".

![Captura de pantalla que muestra la conmutación por error con un disco agregado.](./media/azure-to-azure-tutorial-failover-failback/failover-added.png)

## <a name="reprotect-the-secondary-vm"></a>Volver a proteger la máquina virtual secundaria

Después de la conmutación por error de la máquina virtual, debe volver a protegerla para que se replique de nuevo en la región principal.

1. Asegúrese de que la máquina virtual está en el estado **Conmutación por error confirmada** y compruebe que la región principal está disponible, y puede crear y tener acceso a los recursos nuevos en ella.
2. En **Almacén** > **Elementos replicados**, haga clic con el botón derecho en la máquina virtual que ha sido objeto de la conmutación por error y seleccione **Volver a proteger**.

   ![Captura de pantalla de la opción Reprotección de una máquina virtual.](./media/azure-to-azure-tutorial-failover-failback/reprotect.png)

2. Compruebe que la dirección de la protección, de la región secundaria a la primaria, ya está seleccionada.
3. Revise la información de **Grupo de recursos, red, almacenamiento y conjuntos de disponibilidad**. Los recursos marcados se crean como parte de la operación de reprotección.
4. Haga clic en **Aceptar** para desencadenar un trabajo de repetición de la protección. Este trabajo inicializa el sitio de destino con los datos más recientes. A continuación, replica las diferencias en la región principal. La máquina virtual está ahora en un estado protegido.

## <a name="next-steps"></a>Pasos siguientes
- Después de la reprotección, [obtenga información](azure-to-azure-tutorial-failback.md) sobre cómo conmutar por recuperación a la región primaria cuando esté disponible.
- [Obtenga más información](azure-to-azure-how-to-reprotect.md#what-happens-during-reprotection) sobre el flujo de reprotección.
