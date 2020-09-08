---
title: Traslado de máquinas virtuales de Azure a otra región con Azure Site Recovery
description: Use Azure Site Recovery para mover máquinas virtuales de IaaS de Azure de una región de Azure a otra.
services: site-recovery
author: sideeksh
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: sideeksh
ms.custom: MVC
ms.openlocfilehash: 11767e7369648ad2f4dec4480fbad0f6218446fb
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425422"
---
# <a name="move-azure-vms-to-another-azure-region"></a>Traslado de máquinas virtuales de Azure a otra región de Azure

Puede que quiera mover máquinas virtuales de infraestructura como servicio (IaaS) de Azure de una región a otra para mejorar la confiabilidad, la disponibilidad, la administración o la gobernanza. En este tutorial se muestra cómo mover las máquinas virtuales de Azure a otra región con Azure Site Recovery. Aprenderá a:

> [!div class="checklist"]
> * Verificar los requisitos previos
> * Preparación de las máquinas virtuales de origen
> * Preparación de la región de destino
> * Copia de datos en la región de destino
> * Pruebe la configuración.
> * Realización del traslado
> * Descartar los recursos de la región de origen


> [!IMPORTANT]
> En este artículo se describe cómo mover máquinas virtuales de Azure de una región a otra *tal cual*. Si su objetivo es mover las máquinas virtuales a las zonas de disponibilidad para mejorar la disponibilidad de la infraestructura, consulte [Traslado de máquinas virtuales de Azure a zonas de disponibilidad](move-azure-vms-avset-azone.md).

## <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de tener las máquinas virtuales de Azure en la región de Azure de origen *desde* donde se va a realizar el traslado.
- Compruebe que se admite su opción de [combinación de región de origen y región de destino](./azure-to-azure-support-matrix.md#region-support) y elija con cuidado la región de destino.
- Asegúrese de entender la [arquitectura y los componentes del escenario](azure-to-azure-architecture.md).
- Revise las [limitaciones y los requisitos de compatibilidad](azure-to-azure-support-matrix.md).
- Compruebe los permisos de la cuenta. Si acaba de crear su cuenta de Azure gratis, *ya es* el administrador de la suscripción. Si no es el administrador, trabaje con el administrador para obtener los permisos que necesita:
  -  Para permitir la replicación de una máquina virtual y copiar los datos en el destino mediante Site Recovery, debe tener permisos para crear una máquina virtual en los recursos de Azure. El rol integrado "Colaborador de la máquina virtual" tiene estos permisos. Con los permisos, puede:
        - Crear una máquina virtual en el grupo de recursos seleccionado.
        - Crear una máquina virtual en la red virtual seleccionada.
        - Escribir en la cuenta de almacenamiento seleccionada.

  - También necesita permiso para administrar las operaciones de Site Recovery. El rol Colaborador de Site Recovery tiene todos los permisos necesarios para administrar las operaciones de Site Recovery en un almacén de Azure Recovery Services.

## <a name="prepare-the-source-vms"></a>Preparación de las máquinas virtuales de origen

1. Compruebe que las máquinas virtuales de Azure que va a mover tienen los certificados raíz más recientes. Si no es así, no puede habilitar la copia de datos en la región de destino debido a las restricciones de seguridad.

    - En máquinas virtuales Windows, instale las actualizaciones de Windows más recientes de modo que todos los certificados raíz de confianza estén en ella. En un entorno desconectado, siga los procesos estándar de actualización de certificados y de Windows Update en su organización.
    - En las máquinas virtuales Linux, siga las instrucciones proporcionadas por su distribuidor de Linux para obtener los certificados raíz de confianza y la lista de revocación de certificados más recientes.
2. Asegúrese de que no usa un proxy de autenticación para controlar la conectividad de red de las máquinas virtuales que quiere mover.
3. Si la máquina virtual que intenta mover no tiene acceso a Internet, o usa un proxy de firewall para controlar el acceso de salida, compruebe los [requisitos](azure-to-azure-tutorial-enable-replication.md#set-up-outbound-network-connectivity-for-vms).
4. Documente el diseño de la red de origen y todos los recursos que usa actualmente, incluidos (entre otros) equilibradores de carga, grupos de seguridad de red y dirección IP pública, para comprobarlos.

## <a name="prepare-the-target-region"></a>Preparación de la región de destino

1. En su suscripción de Azure, compruebe que puede crear máquinas virtuales en la región de destino que se usa para la recuperación ante desastres. Si es preciso, póngase en contacto con el soporte técnico para habilitar la cuota necesaria.

2. Asegúrese de que su suscripción tenga suficientes recursos para admitir las máquinas virtuales de origen. Si usa Site Recovery para copiar datos en el destino, elija para la máquina virtual de destino el mismo tamaño o el que más se aproxime.

3. Asegúrese de que crea un recurso de destino para cada componente identificado en el diseño de la red de origen. De esta forma se garantiza que las máquinas virtuales tengan toda la funcionalidad y características de la región de destino que tenían en la región de origen.

   Azure Site Recovery detecta y crea automáticamente una red virtual y una cuenta de almacenamiento al habilitar la replicación para la máquina virtual de origen. También puede crear previamente estos recursos y asignarlos a la máquina virtual como parte del paso para habilitar la replicación. Sin embargo, debe crear manualmente cualquier otro recurso en la región de destino. Consulte los siguientes documentos para crear los recursos de red usados más comúnmente en función de la configuración de la máquina virtual de origen:

   - [Grupos de seguridad de red](../virtual-network/manage-network-security-group.md)
   - [Equilibradores de carga](../load-balancer/index.yml)
   - [Dirección IP pública](../virtual-network/virtual-network-public-ip-address.md)
    
   Para cualquier otro componente de red, consulte la [documentación de red de Azure](../index.yml?pivot=products&panel=network). 

4. Para probar la configuración antes de realizar el traslado, [cree una red que no sea de producción](../virtual-network/quick-create-portal.md) manualmente en la región de destino. Al probar la configuración se crea una interferencia mínima con el entorno de producción y, por este motivo, se recomienda.
    
## <a name="copy-data-to-the-target-region"></a>Copia de datos en la región de destino
En los pasos siguientes, usará Azure Site Recovery para copiar datos en la región de destino.

### <a name="create-the-vault-in-any-region-except-the-source"></a>Creación del almacén en cualquier región, excepto en la de origen

1. Inicie sesión en [Azure Portal](https://portal.azure.com) > **Recovery Services**.
2. Seleccione **Crear un recurso** > **Herramientas de administración** > **Backup and Site Recovery**.
3. En **Nombre**, especifique el nombre descriptivo **ContosoVMVault**. Si tiene más de una suscripción, seleccione la apropiada.
4. Cree un grupo de recursos denominado **ContosoRG**.
5. Especifique una región de Azure. Para ver las regiones admitidas, consulte [Detalles de precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. En los almacenes de Recovery Services, haga clic en **Información general** > **ContosoVMVault** >  **+Replicar**.
7. En **Origen**, seleccione **Azure**.
8. En **Ubicación de origen**, seleccione la región de Azure de origen donde se ejecutan actualmente sus máquinas virtuales.
9. Seleccione el modelo de implementación de Azure Resource Manager. A continuación, seleccione la **suscripción de origen** y el **grupo de recursos de origen**.
10. Seleccione **Aceptar** para guardar la configuración.

### <a name="enable-replication-for-azure-vms-and-start-copying-the-data"></a>Habilite la replicación de máquinas virtuales de Azure y comience a copiar los datos

Site Recovery recupera una lista de las máquinas virtuales asociadas a la suscripción y el grupo de recursos.

1. Seleccione la máquina virtual que quiere mover y, después, seleccione **Aceptar**.
2. En **Configuración**, haga clic en **Recuperación ante desastres**.
3. En **Configurar recuperación ante desastres** > **Región de destino**, seleccione la región de destino en la que quiere realizar la replicación.
4. Elija usar los recursos de destino predeterminados o los que ha creado previamente.
5. Seleccione **Habilitar la replicación** para iniciar el trabajo.

   ![Habilitar replicación](media/tutorial-migrate-azure-to-azure/settings.png)

 

## <a name="test-the-configuration"></a>Pruebe la configuración.


1. Vaya al almacén. En **Configuración** > **Elementos replicados**, seleccione la máquina virtual que quiere mover a la región de destino. Luego, seleccione **Conmutación por error de prueba**.
2. En **Conmutación por error**, seleccione el punto de recuperación que usará para la conmutación por error:

   - **Procesado más recientemente**: error de la VM en el último punto de recuperación procesado por el servicio Site Recovery. Se muestra la marca de tiempo. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, por lo que no se dedica tiempo al procesamiento de los datos.
   - **Más reciente coherente con la aplicación**: esta opción permite conmutar por error todas las máquinas virtuales al punto de recuperación coherente con la aplicación más reciente. Se muestra la marca de tiempo.
   - **Personalizado**: seleccione un punto de recuperación.

3. Seleccione la red virtual de Azure de destino a la que desea trasladar las máquinas virtuales de Azure para probar la configuración.

   > [!IMPORTANT]
   > Se recomienda usar una red de máquinas virtuales de Azure independiente para la conmutación por error de prueba y no la red de producción de la región de destino.

4. Para comenzar el traslado de prueba, seleccione **Aceptar**. Para seguir el progreso, seleccione la máquina virtual para abrir sus **propiedades**. O bien, seleccione el trabajo **Conmutación por error de prueba** en el almacén. A continuación, seleccione **configuración** > **Trabajos** > **Trabajos de Site Recovery**.
5. Una vez finalizada la conmutación por error, la VM de Azure de réplica aparece en Azure Portal > **Virtual Machines**. Asegúrese de que la máquina virtual está en funcionamiento, tiene el tamaño adecuado y está conectada a la red apropiada.
6. Para eliminar la máquina virtual que ha creado para las pruebas, seleccione **Limpiar conmutación por error de prueba** en el elemento replicado. En **Notas**, registre y guarde las observaciones relacionadas con la prueba.

## <a name="perform-the-move-and-confirm"></a>Realización del traslado y confirmación

1. Vaya al almacén, en **Configuración** > **Elementos replicados**, seleccione la máquina virtual y, después, en **Conmutación por error**.
1. En **Conmutación por error**, seleccione **Más reciente**. 
2. Seleccione **Apague la máquina antes de comenzar con la conmutación por error**. Site Recovery intentará apagar la máquina virtual de origen antes de desencadenar la conmutación por error. Pero la conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
3. Una vez finalizado el trabajo, compruebe que la máquina virtual aparece en la región de Azure de destino como se esperaba.
4. En **Elementos replicados**, haga clic con el botón derecho en la máquina virtual y seleccione **Confirmar**. Ahora ha finalizado el traslado. Espere hasta que el trabajo de confirmación finalice.

## <a name="discard-the-resources-from-the-source-region"></a>Descartar los recursos de la región de origen

- Vaya a la máquina virtual y seleccione **Deshabilitar replicación**. Esto detiene el proceso de copiar los datos para la máquina virtual.

  > [!IMPORTANT]
  > Realice este paso para evitar cargos por la replicación de Site Recovery tras el traslado.

Si no tiene pensado volver a usar ninguno de los recursos de origen, siga estos pasos:

1. Elimine todos los recursos de red pertinentes de la región de origen que se enumeraron en el paso 4 en [Preparación de las máquinas virtuales de origen](#prepare-the-source-vms).
2. Elimine la cuenta de almacenamiento correspondiente en la región de origen.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a mover máquinas virtuales de Azure a otra región de Azure. Ya puede configurar la opción de recuperación ante desastres para esas máquinas virtuales.

> [!div class="nextstepaction"]
> [Configurar la recuperación ante desastres después de la migración](azure-to-azure-quickstart.md)
