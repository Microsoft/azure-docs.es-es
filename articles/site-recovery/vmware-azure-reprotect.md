---
title: Reprotección de máquinas virtuales de VMware en un entorno local con Azure Site Recovery
description: Obtenga información sobre cómo reproteger las máquinas virtuales de VMware después de una conmutación por error a Azure mediante Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: mayg
ms.openlocfilehash: 8a78ed25be80cbf083467209f764109a26782278
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292779"
---
# <a name="reprotect-from-azure-to-on-premises"></a>Reprotección desde Azure a un entorno local

Tras la [conmutación por error](site-recovery-failover.md) de máquinas virtuales de VMware locales o de servidores físicos a Azure, el primer paso en la conmutación por recuperación en el sitio local es reproteger las máquinas virtuales de Azure que se crearon durante la conmutación por error. En este artículo se describe cómo hacerlo. 

## <a name="before-you-begin"></a>Antes de empezar

1. Siga los pasos descritos en [este artículo](vmware-azure-prepare-failback.md) para prepararse para la reprotección y la conmutación por recuperación. Esto incluye la configuración de un servidor de procesos en Azure y un servidor de destino maestro local, así como la configuración de una VPN de sitio a sitio, o del emparejamiento privado de ExpressRoute para la conmutación por recuperación.
2. Asegúrese de que el servidor de configuración local está en ejecución y conectado a Azure. Durante la conmutación por error a Azure, es posible que no se pueda acceder al sitio local y el servidor de configuración puede estar no disponible o apagado. Durante la conmutación por recuperación, la VM debe encontrarse en la base de datos del servidor de configuración. En caso contrario, la conmutación por recuperación no será correcta.
3. Elimine todas las instantáneas en el servidor de destino maestro local. La reprotección no funcionará si existe alguna instantánea.  Las instantáneas de la VM se combinan automáticamente durante los trabajos de reprotección.
4. Si va a reproteger máquinas virtuales agrupadas en un grupo de replicación para asegurar la coherencia de varias máquinas virtuales, asegúrese de que todas tienen el mismo sistema operativo (Windows o Linux) y de que el servidor de destino maestro que va a implementar tiene el mismo tipo de sistema operativo. Todas las máquinas virtuales de un grupo de replicación deben usar el mismo servidor de destino maestro.
5. Abra [los puertos necesarios](vmware-azure-prepare-failback.md#ports-for-reprotectionfailback) para realizar la conmutación por recuperación.
6. Asegúrese de que vCenter Server esté conectado antes de realizar la conmutación por recuperación. En caso contrario, se producirá un error al desconectar los discos y conectarlos a la máquina virtual.
7. Si una instancia de vCenter Server administra las máquinas virtuales a las que quiere conmutar por recuperación, asegúrese de tener los permisos necesarios. Si realiza la detección de usuarios de solo lectura de vCenter y protege las máquinas virtuales, la protección se ejecutará correctamente y la conmutación por error funcionará. Sin embargo, durante la reprotección, se produce un error de conmutación por error porque no se pueden detectar los almacenes de datos y no aparecen durante la reprotección. Para solucionar este problema, puede actualizar las credenciales de vCenter con la [cuenta o los permisos adecuados](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) y volver a intentar el trabajo. 
8. Si ha usado una plantilla para crear las máquinas virtuales, asegúrese de que cada una tenga su propio UUID para los discos. Si el UUID de la VM local entra en conflicto con el UUID del servidor de destino maestro porque ambos se crearon a partir de la misma plantilla, se produce un error de reprotección. En este caso, implemente desde una plantilla diferente.
9. Si va a conmutar por recuperación en una instancia de vCenter Server alternativa, asegúrese de que se detectan tanto el nuevo vCenter Server como el servidor de destino maestro. Normalmente, si no se detectaron, significa que los almacenes de datos no son accesibles o visibles en **Reproteger**.
10. Compruebe los siguientes escenarios en los que no se puede realizar la conmutación por recuperación:
    - Si utiliza las ediciones gratuitas de ESXi 5.5 o vSphere Hypervisor 6, actualice a una versión diferente.
    - Si tiene un servidor físico de Windows Server 2008 R2 SP1.
    - Las máquinas virtuales de VMware no pueden conmutar por recuperación en Hyper-V.
    - VM que se han migrado.
    - Un a máquina virtual que se ha migrado a otro grupo de recursos.
    - Una máquina virtual de Azure de réplica que se ha eliminado.
    - Una máquina virtual de Azure de réplica que no está protegida (se está replicando en el sitio local).
10. [Revise los tipos de conmutación por recuperación](concepts-types-of-failback.md) que puede usar: recuperación de ubicación original y recuperación de ubicación alternativa.


## <a name="enable-reprotection"></a>Habilitación de la reprotección

Habilite la replicación. Puede reproteger máquinas virtuales específicas o un plan de recuperación:

- Si reprotege un plan de recuperación, tiene que proporcionar los valores para cada máquina protegida.
- Si las máquinas virtuales pertenecen a un grupo de replicación para la coherencia entre varias máquinas virtuales, solo se pueden reproteger mediante un plan de recuperación. Las máquinas virtuales de un grupo de replicación deben usar el mismo servidor de destino maestro.

### <a name="before-you-start"></a>Antes de comenzar

- Después del arranque de una máquina virtual en Azure tras una conmutación por error, el agente tarda algún tiempo en registrarse de nuevo en el servidor de configuración (hasta 15 minutos). Durante este tiempo, no podrá realizar la reprotección y aparece un mensaje de error para indicar que el agente no está instalado. Si esto sucede, espere unos minutos y realice al reprotección.
- Si quiere conmutar por recuperación la máquina virtual de Azure en una máquina virtual local existente, los almacenes de datos de la máquina virtual local deben montarse con acceso de lectura/escritura en el host ESXi del servidor de destino maestro.
- Si quiere conmutar por recuperación en una ubicación alternativa (por ejemplo, si no existe la máquina virtual local), seleccione la unidad de retención y el almacén de datos configurados para el servidor de destino maestro. Cuando se conmuta por recuperación en el sitio local, las máquinas virtuales VMware del plan de protección de conmutación por recuperación usan el mismo almacén de datos que el servidor de destino maestro. Entonces se crea una nueva VM en vCenter.

Para habilitar la reprotección, siga estos pasos:

1. Seleccione **Almacén** > **Elementos replicados**. Haga clic con el botón derecho en la máquina virtual que ha sido objeto de la conmutación por error y seleccione **Reproteger**. O bien, con los botones de comando, seleccione la máquina y luego **Reproteger**.
2. Verifique que la dirección de protección se establece en **De Azure a local**.
3. En **Servidor de destino maestro** y **Servidor de procesos**, seleccione el servidor de destino maestro local y el servidor de procesos.  
4. En **Almacén de datos**, seleccione aquel en el que quiera recuperar los discos en local. Se usa esta opción cuando se elimina la máquina virtual local y es necesario crear discos. Esta opción se omite si los discos ya existen. Todavía debe especificar un valor.
5. Elija la unidad de retención.
6. La directiva de conmutación por recuperación se selecciona automáticamente.
7. Seleccione **Aceptar** para comenzar con la reprotección.

    ![Cuadro de diálogo Reprotección](./media/vmware-azure-reprotect/reprotectinputs.png)
    
8. Comienza un trabajo para replicar la máquina virtual de Azure en el sitio local. Puede realizar el seguimiento del progreso en la pestaña **Trabajos**.
    - Cuando la reprotección se completa satisfactoriamente, la máquina virtual adopta un estado protegido.
    - La máquina virtual local está apagada durante la reprotección. Esto ayuda a garantizar la coherencia de datos durante la replicación.
    - No encienda la máquina virtual local una vez finalizada la reprotección.
   

## <a name="next-steps"></a>Pasos siguientes

- Si tiene algún problema, revise el [artículo de solución de problemas](vmware-azure-troubleshoot-failback-reprotect.md).
- Una vez protegidas las máquinas virtuales de Azure, puede [realizar una conmutación por recuperación](vmware-azure-failback.md). La conmutación por recuperación apaga la máquina virtual de Azure e inicia la máquina virtual local. Debe prever cierto tiempo de inactividad para la aplicación y elegir un tiempo de conmutación por recuperación en consecuencia.


