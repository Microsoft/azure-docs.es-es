---
title: Traslado de máquinas virtuales de IaaS de Azure a otra región con Azure Site Recovery
description: Use Azure Site Recovery para mover máquinas virtuales de IaaS de Azure de una región de Azure a otra.
services: site-recovery
author: Sharmistha-Rai
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: sharrai
ms.custom: MVC
ms.openlocfilehash: e8f14b86678f7d395f445438d7e869168b13e54b
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425932"
---
# <a name="move-azure-vms-to-another-region"></a>Traslado de máquinas virtuales de Azure a otra región

Hay varios escenarios en los que puede desear mover las máquinas virtuales de Azure IaaS existentes de una región a otra. Por ejemplo, si desea mejorar la confiabilidad y disponibilidad de las máquinas virtuales existentes, para mejorar la capacidad de administración, o si quiere moverlas por motivos de gobernanza. Para más información, consulte [Introducción al traslado de máquinas virtuales de Azure](azure-to-azure-move-overview.md). 

Puede usar el servicio [Azure Site Recovery](site-recovery-overview.md) para administrar y coordinar la recuperación ante desastres de máquinas locales y máquinas virtuales de Azure para la continuidad empresarial y recuperación ante desastres (BCDR). También puede usar Site Recovery para administrar el movimiento de máquinas virtuales de Azure a una región secundaria.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> 
> * Comprobar los requisitos previos para el traslado
> * Preparar las máquinas virtuales de origen y la región de destino
> * Copiar los datos y habilitar la replicación
> * Probar la configuración y realizar el traslado
> * Eliminar los recursos en la región de origen
> 
> [!NOTE]
> En este tutorial se muestra cómo trasladar las máquinas virtuales de Azure de una región a otra tal cual. Si tiene que mejorar la disponibilidad moviendo las máquinas virtuales en un conjunto de disponibilidad a máquinas virtuales ancladas a la zona en una región distinta, consulte el [tutorial Traslado de máquinas virtuales de Azure a zonas de disponibilidad](move-azure-vms-avset-azone.md).

## <a name="prerequisites"></a>Prerequisites

- Asegúrese de tener las máquinas virtuales de Azure en la región de Azure desde la que va a realizar el traslado.
- Compruebe si [se admite la combinación de región de origen y región de destino](./azure-to-azure-support-matrix.md#region-support) que ha elegido y tome una decisión informada sobre la región de destino.
- Asegúrese de entender la [arquitectura y los componentes del escenario](azure-to-azure-architecture.md).
- Revise las [limitaciones y los requisitos de compatibilidad](azure-to-azure-support-matrix.md).
- Compruebe los permisos de la cuenta. Si ha creado su cuenta de Azure gratis, ya es el administrador de la suscripción. Si no es el administrador de la suscripción, solicite al administrador que le asigne los permisos que necesita. Para habilitar la replicación para una máquina virtual y esencialmente copiar los datos mediante Azure Site Recovery, tiene que tener:

    - Permisos para crear una máquina virtual en recursos de Azure. El rol integrado "Colaborador de la máquina virtual" tiene estos permisos, que incluyen:
    - Permiso para crear una máquina virtual en el grupo de recursos seleccionado.
    - Permiso para crear una máquina virtual en la red virtual seleccionada
    - Permiso para escribir en la cuenta de almacenamiento seleccionada
    
    - Permisos para administrar las operaciones de Azure Site Recovery. El rol "Colaborador de Site Recovery" tiene todos los permisos necesarios para administrar las operaciones de Site Recovery en un almacén de Recovery Services.

- Asegúrese de que todos los certificados raíz más recientes están en las máquinas virtuales de Azure que quiera trasladar. Si los certificados raíz más recientes no están presentes en la máquina virtual, las restricciones de seguridad no permitirán la copia de datos en la región de destino.

- Para las máquinas virtuales de Windows, instale las actualizaciones de Windows más recientes en la máquina virtual, de modo que todos los certificados raíz de confianza estén en ella. En un entorno desconectado, siga los procesos estándar de actualización de certificados y de Windows Update en su organización.
    
- En las máquinas virtuales Linux, para obtener los últimos certificados raíz de confianza y la lista de revocación de certificados en la máquina virtual, siga las instrucciones proporcionadas por su distribuidor de Linux.
- Asegúrese de que no utiliza un proxy de autenticación para controlar la conectividad de red de las máquinas virtuales que quiere trasladar.

- Si la máquina virtual que está intentando trasladar no tiene acceso a Internet, o utiliza un proxy de firewall para controlar el acceso de salida, [compruebe los requisitos](azure-to-azure-tutorial-enable-replication.md#set-up-outbound-network-connectivity-for-vms).

- Identifique el diseño de red de origen y todos los recursos que está usando actualmente. Esto incluye, pero no se limita a los equilibradores de carga, grupos de seguridad de red (NSG), e IP públicas.

- Compruebe que su suscripción de Azure permite crear máquinas virtuales en la región de destino que se usa para la recuperación ante desastres. Para habilitar la cuota necesaria, póngase en contacto con el soporte técnico.

- Asegúrese de que su suscripción tiene suficientes recursos para admitir máquinas virtuales con tamaños que se correspondan con las máquinas virtuales de origen. Si está usando Site Recovery para copiar datos en el destino, Site Recovery elije el mismo tamaño para la máquina virtual de destino o el más cercano posible.

- Asegúrese de que crea un recurso de destino para cada componente identificado en el diseño de la red de origen. Este paso es importante para asegurarse de que las máquinas virtuales tengan en la región de destino toda la funcionalidad y características que tenían en la región de origen.

     > [!NOTE] 
     > Azure Site Recovery automáticamente detecta y crea una red virtual al habilitar la replicación para la máquina virtual de origen. También puede crear previamente una red y asignarla a la máquina virtual en el flujo de usuario para habilitar la replicación. Como se menciona a continuación, es necesario crear manualmente cualquier otro recurso en la región de destino.

    Para crear los recursos de red más utilizados que considere apropiados, en función de la configuración de la máquina virtual de origen, consulte la siguiente documentación:
    - [Grupos de seguridad de red](../virtual-network/manage-network-security-group.md)
    - [Equilibradores de carga](../load-balancer/index.yml)
    -  [Dirección IP pública](../virtual-network/virtual-network-public-ip-address.md)
    - Para cualquier otro componente de red, consulte la [documentación de red](../index.yml?pivot=products&panel=network).



## <a name="prepare"></a>Preparación
En los pasos siguientes se muestra cómo preparar la máquina virtual para el movimiento mediante Azure Site Recovery como una solución. 

### <a name="create-the-vault-in-any-region-except-the-source-region"></a>Cree el almacén en cualquier región, excepto en la de origen

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. En Buscar, escriba Recovery Services y haga clic en Almacenes de Recovery Services.
1. En el menú Almacenes de Recovery Services, haga clic en +Agregar.
1. En el apartado **Nombre**, especifique el nombre descriptivo **ContosoVMVault**. Si tiene más de una suscripción, seleccione la apropiada.
1. Cree un grupo de recursos denominado **ContosoRG**.
1. Especifique una región de Azure. Para comprobar las regiones admitidas, consulte la disponibilidad geográfica en [Detalles de precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
1. En los **almacenes de Recovery Services**, seleccione **Información general** > **ContosoVMVault** > **Elementos replicados** >  **+Replicar**.
1. En **Origen**, seleccione **Azure**.
1. En **Ubicación de origen**, seleccione la región de Azure de origen donde se ejecutan actualmente sus máquinas virtuales.
1. Seleccione el modelo de implementación de Resource Manager. A continuación, seleccione la **suscripción de origen** y el **grupo de recursos de origen**.
1. Seleccione **Aceptar** para guardar la configuración.

### <a name="enable-replication-for-azure-vms-and-start-copying-the-data"></a>Habilite la replicación de máquinas virtuales de Azure y comience a copiar los datos

Site Recovery recupera una lista de las máquinas virtuales asociadas a la suscripción y el grupo de recursos.

1. En el paso siguiente, seleccione la máquina virtual que desee mover y, después, seleccione **Aceptar**.
1. En **Configuración**, haga clic en **Recuperación ante desastres**.
1. En **Configurar la recuperación ante desastres** > **Región de destino**, seleccione la región de destino en la que quiere realizar la replicación.
1. Para este tutorial, acepte los valores predeterminados.
1. Seleccione **Habilitar replicación**. Este paso inicia un trabajo para habilitar la replicación de la máquina virtual.

    ![Habilitar replicación](media/tutorial-migrate-azure-to-azure/settings.png)

## <a name="move"></a>Move

En los pasos siguientes se muestra cómo realizar el traslado a la región de destino.

1. Vaya al almacén. En **Configuración** > **Elementos replicados**, seleccione la máquina virtual y luego seleccione **Conmutación por error**.
2. En **Conmutación por error**, seleccione **Más reciente**.
3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error**. A continuación, Site Recovery intentará apagar la máquina virtual de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
4. Una vez finalizado el trabajo, compruebe que la máquina virtual aparece en la región de Azure de destino como se esperaba.


## <a name="discard"></a>Discard (Descartar) 

En caso de que haya comprobado la máquina virtual que se ha trasladado y tenga que cambiarla al punto de conmutación por error o desee volver a un punto anterior, en los **Elementos replicados**, haga clic con el botón de derecho para seleccionar la máquina virtual y **cambie el punto de recuperación**. Este paso le ofrece la opción de especificar un punto de recuperación diferente y conmutar por error a ese mismo. 


## <a name="commit"></a>Commit 

Una vez que haya comprobado la máquina virtual que se ha trasladado y esté preparado para confirmar el cambio **, en los**elementos replicados, seleccione con el botón derecho la máquina virtual y **confirme**. Esto finaliza el proceso de traslado a la región de destino. Espere hasta que el trabajo de confirmación finalice.

## <a name="clean-up"></a>Limpieza

Los pasos siguientes le guiarán por el proceso de limpieza de la región de origen, así como de los recursos relacionados que se usaron para el traslado.

Para todos los recursos que se usaron para el traslado:

- Vaya a la máquina virtual. Seleccione **Deshabilitar replicación**. Este paso detiene el proceso de copiar los datos para la máquina virtual.

   > [!IMPORTANT]
   > Es importante realizar este paso para evitar cargos por replicación de Azure Site Recovery.

Si no piensa volver a usar ninguno de los recursos de origen, siga estos pasos adicionales:

1. Elimine todos los recursos de red pertinentes en la región de origen que identificó en los [requisitos previos](#prerequisites).
1. Elimine la cuenta de almacenamiento correspondiente en la región de origen.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha trasladado una máquina virtual de Azure a otra región de Azure. Ahora puede configurar la opción de recuperación ante desastres para la máquina virtual que ha trasladado.

> [!div class="nextstepaction"]
> [Configurar la recuperación ante desastres después de la migración](azure-to-azure-quickstart.md)
