---
title: Traslado de máquinas virtuales de Azure entre Azure Government y las regiones públicas con Azure Site Recovery
description: Use Azure Site Recovery para trasladar máquinas virtuales de Azure entre Azure Government y las regiones públicas.
author: sideeksh
ms.service: site-recovery
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: sideeksh
ms.custom: MVC
ms.openlocfilehash: a76ebf95b92b6e1251a04daa9ffb48a9abe15b50
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425354"
---
# <a name="move-azure-vms-between-azure-government-and-public-regions"></a>Mover las VM de Azure entre Azure Government y las regiones públicas 

Es posible que quiera mover sus VM de IaaS entre Azure Government y las regiones públicas para aumentar la disponibilidad de las VM ya existentes, para mejorar la capacidad de administración o por razones de gobernanza, tal como se detalla [aquí](azure-to-azure-move-overview.md).

Además de utilizar el servicio [Azure Site Recovery](site-recovery-overview.md) para administrar y coordinar la recuperación ante desastres de máquinas locales y máquinas virtuales de Azure, con el fin de garantizar la continuidad empresarial y la recuperación ante desastres (BCDR), también puede usar Site Recovery para administrar el traslado de máquinas virtuales de Azure a una región secundaria.       

En este tutorial se muestra cómo trasladar las VM de Azure entre Azure Government y las regiones públicas mediante Azure Site Recovery. Lo mismo se puede aplicar a la hora de mover las VM entre pares de regiones que no están dentro del mismo clúster geográfico. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Verificar los requisitos previos
> * Preparación de las máquinas virtuales de origen
> * Preparación de la región de destino
> * Copia de datos en la región de destino
> * Pruebe la configuración.
> * Realización del traslado
> * Descarte de los recursos en la región de origen

> [!IMPORTANT]
> En este tutorial se muestra cómo trasladar las VM de Azure entre Azure Government y las regiones públicas, o entre pares de regiones que no son compatibles con la solución de recuperación ante desastres de las VM de Azure. En caso de que sus pares de regiones de origen y destino se [admitan](./azure-to-azure-support-matrix.md#region-support), consulte este [documento](azure-to-azure-tutorial-migrate.md) para realizar el traslado. Si necesita mejorar la disponibilidad al trasladar las VM de un conjunto de disponibilidad a las VM ancladas en la zona de otra región, consulte el tutorial que encontrará [aquí](move-azure-VMs-AVset-Azone.md).

> [!IMPORTANT]
> No es recomendable usar este método para configurar DR entre pares de regiones no compatibles, ya que los pares se definen teniendo en cuenta la latencia de los datos, lo que es fundamental para un escenario de DR.

## <a name="verify-prerequisites"></a>Verificar los requisitos previos

> [!NOTE]
> Asegúrese de que conoce a fondo [la arquitectura y los componentes](physical-azure-architecture.md) de este escenario. Esta arquitectura se usará para mover las VM de Azure, **tratándolas como si fueran servidores físicos** .

- Revise los [requisitos de compatibilidad](vmware-physical-secondary-support-matrix.md) de todos los componentes.
- Asegúrese de que los servidores que quiere replicar cumplen los [requisitos de VM de Azure](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
- Prepare una cuenta para la instalación automática de Mobility Service en cada servidor que quiera replicar.

- Después de una conmutación por error en la región de destino de Azure, no podrá realizar directamente una conmutación por recuperación en la región de origen. Debido a ello, tendrá que volver a configurar la replicación en el destino.

### <a name="verify-azure-account-permissions"></a>Comprobar los permisos de cuenta de Azure

Asegúrese de que la cuenta de Azure tiene permisos para la replicación de máquinas virtuales en Azure.

- Revise los [permisos](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) que necesita para replicar máquinas en Azure.
- Compruebe y modifique los permisos de [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md). 

### <a name="set-up-an-azure-network"></a>Configurar una red de Azure

Configure una [red de Azure](../virtual-network/quick-create-portal.md) de destino.

- Las VM de Azure se colocarán en esta red cuando se creen después de la conmutación por error.
- La red debe estar en la misma región que el almacén de Recovery Services.


### <a name="set-up-an-azure-storage-account"></a>Configurar una cuenta de Azure Storage

Configure una [cuenta de almacenamiento de Azure](../storage/common/storage-account-create.md).

- Site Recovery replica máquinas virtuales locales en Azure Storage. Las máquinas virtuales de Azure se crean en el almacenamiento después de producirse la conmutación por error.
- La cuenta de almacenamiento debe estar en la misma región que el almacén de Recovery Services.


## <a name="prepare-the-source-vms"></a>Preparación de las máquinas virtuales de origen

### <a name="prepare-an-account-for-mobility-service-installation"></a>Preparación de una cuenta para la instalación de Mobility Service

Tiene que instalar el Servicio de movilidad en cada servidor que quiera replicar. Site Recovery instala este servicio automáticamente cuando se habilita la replicación del servidor. Para instalar automáticamente, debe preparar una cuenta que Site Recovery vaya a usar para acceder al servidor.

- Puede usar una cuenta local o de dominio.
- En el caso de máquinas virtuales de Windows, si no usa una cuenta de dominio, deshabilite el control de acceso de usuarios remotos en el equipo local. Para hacerlo, en el Registro, en **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, agregue la entrada DWORD **LocalAccountTokenFilterPolicy** con un valor de 1.
- Para agregar la entrada del Registro para deshabilitar el valor desde una CLI, escriba: ``REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1.``
- Para Linux, la cuenta debe ser una raíz en el servidor Linux de origen.


## <a name="prepare-the-target-region"></a>Preparación de la región de destino

1. Compruebe que su suscripción de Azure permite crear máquinas virtuales en la región de destino que se usa para la recuperación ante desastres. Para habilitar la cuota necesaria, póngase en contacto con el soporte técnico.

2. Asegúrese de que su suscripción tiene suficientes recursos para admitir máquinas virtuales con tamaños que se correspondan con las máquinas virtuales de origen. Si está usando Site Recovery para copiar datos en el destino, elija el mismo tamaño para la máquina virtual de destino o el más cercano posible.

3. Asegúrese de que crea un recurso de destino para cada componente identificado en el diseño de la red de origen. Esto es importante para asegurarse de que, tras la migración a la región de destino, las VM tengan toda las funcionalidades y características que tenían en el origen.

    > [!NOTE]
    > Azure Site Recovery automáticamente detecta y crea una red virtual cuando habilita la replicación para la máquina virtual de origen, o también puede crear previamente una red y asignarla a la máquina virtual del flujo de usuario para habilitar la replicación. Pero para cualquier otro recurso, como se menciona a continuación, es necesario crearlo manualmente en la región de destino.

     Consulte los siguientes documentos para crear los recursos de red más utilizados y que considere más relevantes, en función de la configuración de la máquina virtual de origen.

    - [Grupos de seguridad de red](../virtual-network/manage-network-security-group.md)
    - [Equilibradores de carga](../load-balancer/index.yml)
    - [Dirección IP pública](../virtual-network/virtual-network-public-ip-address.md)
    
    Para cualquier otro componente de red, consulte la [documentación](../index.yml?pivot=products&panel=network) de red.

4. Si desea probar la configuración antes de realizar la migración final a la región de destino, [cree manualmente una red sin producción](../virtual-network/quick-create-portal.md) en la región de destino. Esto creará una interferencia mínima con producción y, por este motivo, se recomienda.

## <a name="copy-data-to-the-target-region"></a>Copia de datos en la región de destino
En los pasos siguientes aprenderá a utilizar Azure Site Recovery para copiar datos en la región de destino.

### <a name="create-the-vault-in-any-region-except-the-source-region"></a>Cree el almacén en cualquier región, excepto en la de origen.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) > **Recovery Services**.
2. Seleccione **Crear un recurso** > **Herramientas de administración** > **Backup and Site Recovery**.
3. En el apartado **Nombre**, especifique el nombre descriptivo **ContosoVMVault**. Si tiene más de una suscripción, seleccione la más adecuada.
4. Cree un grupo de recursos denominado **ContosoRG**.
5. Especifique una región de Azure. Para comprobar las regiones admitidas, consulte la disponibilidad geográfica en [Detalles de precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. En los almacenes de Recovery Services, haga clic en **Overview (Información general)**  > **ContosoVMVault** >  **+Replicate** (+Replicar).
7. Seleccione **To Azure (En Azure)**  > **Not virtualized/Other** (No virtualizado/Otro).

### <a name="set-up-the-configuration-server-to-discover-vms"></a>Configure el servidor de configuración para descubrir VM.


Configure el servidor de configuración, regístrelo en el almacén y detecte máquinas virtuales.

1. Haga clic en **Site Recovery** > **Preparar la infraestructura** > **Origen**.
2. Si no tiene un servidor de configuración, haga clic en **+Servidor de configuración**.
3. En **Agregar servidor**, compruebe que aparezca **Servidor de configuración** en **Tipo de servidor**.
4. Descargue el archivo de instalación unificada de Site Recovery.
5. Descargue la clave de registro del almacén. Se le solicitará cuando ejecute la instalación unificada. La clave será válida durante cinco días a partir del momento en que se genera.

   ![Configurar origen](./media/physical-azure-disaster-recovery/source-environment.png)


### <a name="register-the-configuration-server-in-the-vault"></a>Registro del servidor de configuración en el almacén

Antes de empezar, haga lo siguiente: 

#### <a name="verify-time-accuracy"></a>Verificación de la precisión de tiempo
En la máquina del servidor de configuración, asegúrese de que el reloj del sistema está sincronizado con un [servidor horario](/windows-server/networking/windows-time-service/windows-time-service-top). Deben ser iguales. Si hay una diferencia de 15 minutos, antes o después, se podría producir un error en la instalación.

#### <a name="verify-connectivity"></a>Verificación de la conectividad
Asegúrese de que la máquina puede acceder a estas direcciones URL según el entorno: 

[!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]  

Las reglas de firewall basadas en direcciones IP deben permitir la comunicación a todas las direcciones URL de Azure que se indican a continuación a través del puerto HTTPS (443). Para simplificar y limitar los intervalos de direcciones IP, se recomienda que se realiza el filtrado de URL.

- **Direcciones IP comerciales**: permita los [intervalos de direcciones IP de Azure Datacenter](https://www.microsoft.com/download/confirmation.aspx?id=41653) y el puerto HTTPS (443). Permita los intervalos de direcciones IP para la región de Azure de su suscripción de modo que se admitan las direcciones URL de almacenamiento, AAD, copia de seguridad y replicación.  
- **Las direcciones IP de administraciones públicas**: permita que los [intervalos de direcciones IP del centro de datos de Azure Government](https://www.microsoft.com/en-us/download/details.aspx?id=57063) y el puerto HTTPS (443) para todas las regiones de USGov (Virginia, Texas, Arizona e Iowa) admitan las direcciones URL de almacenamiento, copia de seguridad, replicación y AAD.  

#### <a name="run-setup"></a>Ejecución de la configuración
Ejecute la instalación unificada como administrador Local para instalar el servidor de configuración. El servidor de procesos y el servidor de destino maestro también se instalan de forma predeterminada en el servidor de configuración.

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

Una vez terminado el registro, el servidor de configuración se muestra en la página **Configuración** > **Servidores** del almacén.

### <a name="configure-target-settings-for-replication"></a>Configuración de los ajustes de destino para la replicación

Seleccione y compruebe los recursos de destino.

1. Haga clic en **Preparar infraestructura** > **Destino** y seleccione la suscripción de Azure que quiere usar.
2. Especifique el modelo de implementación de destino.
3. Site Recovery comprueba que tiene una o más redes y cuentas de Azure Storage compatibles.

   ![Destino](./media/physical-azure-disaster-recovery/network-storage.png)


### <a name="create-a-replication-policy"></a>Creación de una directiva de replicación

1. Para crear una directiva de replicación, haga clic en **Site Recovery Infrastructure (Infraestructura de Site Recovery)**  > **Directivas de replicación** >  **+Directiva de replicación**.
2. En **Crear directiva de replicación**, especifique un nombre de directiva.
3. En **Umbral de RPO**, especifique el límite del objetivo de punto de recuperación (RPO). Este valor especifica la frecuencia con que se crean puntos de recuperación de datos. Se genera una alerta cuando la replicación continua supera este límite.
4. En **Retención de punto de recuperación**, especifique la duración (en horas) del período de retención de cada punto de recuperación. Las máquinas virtuales replicadas se pueden recuperar a cualquier momento de un período. Se admite una retención de hasta 24 horas para máquinas replicadas en Premium Storage y 72 horas para almacenamiento estándar.
5. En **Frecuencia de instantánea coherente con la aplicación**especifique la frecuencia (en minutos) con la que se crearán puntos de recuperación que contengan las instantáneas coherentes con la aplicación. Haga clic en **Aceptar** para crear la directiva.

    ![Directiva de replicación](./media/physical-azure-disaster-recovery/replication-policy.png)


La directiva se asocia automáticamente al servidor de configuración. De forma predeterminada, se crea automáticamente una directiva correspondiente para la conmutación por recuperación. Por ejemplo, si la directiva de replicación es **rep-policy**, se crea una directiva de conmutación por recuperación **rep-policy-failback**. Esta directiva no se usa hasta que se inicie una conmutación por recuperación desde Azure.

### <a name="enable-replication"></a>Habilitar replicación

- Site Recovery instala Mobility Service cuando se habilita la replicación.
- Cuando se habilita la replicación para un servidor, los cambios pueden tardar 15 minutos o más en aplicarse y aparecer en el portal.

1. Haga clic en **Replicar la aplicación** > **Origen**.
2. En **Origen**, seleccione el servidor de configuración.
3. En **Tipo de máquina**, seleccione **Máquinas físicas**.
4. Seleccione el servidor de procesos (el servidor de configuración). A continuación, haga clic en **Aceptar**.
5. En **Destino**, seleccione la suscripción y el grupo de recursos donde quiere crear las máquinas virtuales de Azure después de la conmutación por error. Elija el modelo de implementación que quiere usar en Azure (clásico o administración de recursos).
6. Seleccione la cuenta de Azure Storage que desea usar para los datos de replicación. 
7. Seleccione la red y la subred de Azure a la que se conectarán las máquinas virtuales de Azure cuando se creen después de la conmutación por error.
8. Seleccione la opción **Configurar ahora para las máquinas seleccionadas** con el fin de aplicar la configuración de red a todas las máquinas que seleccione para su protección. Seleccione **Configurar más tarde** para seleccionar la red de Azure por máquina. 
9. En **Máquinas físicas**, haga clic en **+ Máquina física**. Especifique el nombre y la dirección IP. Seleccione el sistema operativo del equipo que quiere replicar. Los servidores tardan unos minutos en detectarse y mostrarse. 

   > [!WARNING]
   > Debe especificar la dirección IP de la VM de Azure que va a mover.

10. En **Propiedades** > **Configurar propiedades**, seleccione la cuenta que va a usar el servidor de procesos para instalar automáticamente Mobility Service en el equipo.
11. En **Configuración de la replicación** > **Establecer configuración de replicación**, compruebe que se ha seleccionado la directiva de replicación correcta. 
12. Haga clic en **Enable Replication**. Puede hacer un seguimiento del progreso del trabajo **Habilitar protección** en **Configuración** > **Trabajos** > **Trabajos de Site Recovery**. La máquina estará preparada para la conmutación por error después de que finalice el trabajo **Finalizar la protección** .


Para supervisar los servidores que agregue, puede comprobar la última hora de detección en **Servidores de configuración** > **Último contacto**. Para agregar equipos sin esperar a una hora de detección programada, resalte el servidor de configuración (sin hacer clic en él) y haga clic en **Actualizar**.

## <a name="test-the-configuration"></a>Pruebe la configuración.


1. Vaya al almacén, en **Configuración** > **Elementos replicados**, haga clic en la máquina virtual que desea trasladar a la región de destino y, finalmente, haga clic en el icono **+Conmutación por error de prueba**.
2. En **Conmutación por error**, seleccione el punto de recuperación que usar para la conmutación por error:

   - **Procesado más recientemente**: error de la VM en el último punto de recuperación procesado por el servicio Site Recovery. Se muestra la marca de tiempo. Con esta opción, no se emplea tiempo en el procesamiento de datos, por lo que se proporciona un objetivo de tiempo de recuperación (RTO) bajo.
   - **Más reciente coherente con la aplicación**: esta opción conmuta por error todas las VM en el punto de recuperación más reciente coherente con la aplicación. Se muestra la marca de tiempo.
   - **Personalizado**: seleccione un punto de recuperación.

3. Seleccione la red virtual de Azure de destino a la que desea trasladar las máquinas virtuales de Azure para probar la configuración. 

   > [!IMPORTANT]
   > Le recomendamos que utilice una red de VM de Azure independiente para la conmutación por error de prueba y no la red de producción a la que quiere trasladar las VM y que se configuró cuando habilitó la replicación.

4. Para comenzar a probar el traslado, haga clic en **Aceptar**. Para realizar el seguimiento del progreso, haga clic en la máquina virtual para abrir sus propiedades. También puede hacer clic en el trabajo **Conmutación por error de prueba** en el nombre del almacén > **Configuración** > **Trabajos** > **Trabajos de Site Recovery**.
5. Una vez finalizada la conmutación por error, la VM de Azure de réplica aparece en Azure Portal > **Virtual Machines**. Asegúrese de que la máquina virtual está en funcionamiento, tiene el tamaño adecuado y está conectada a la red apropiada.
6. Si desea eliminar la máquina virtual creada como parte de la prueba del traslado, haga clic en **Limpiar conmutación por error de prueba** en el elemento replicado. En **Notas**, registre y guarde las observaciones asociadas a la prueba.

## <a name="perform-the-move-to-the-target-region-and-confirm"></a>Realice el traslado a la región de destino y confirme.

1. Vaya al almacén, en **Configuración** > **Elementos replicados**, haga clic en la máquina virtual y, después, en **Conmutación por error**.
2. En **Conmutación por error**, seleccione **Más reciente**. 
3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error**. A continuación, Site Recovery intentará apagar la máquina virtual de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**. 
4. Una vez finalizado el trabajo, compruebe que la máquina virtual aparece en la región de Azure de destino como se esperaba.
5. En **Elementos replicados**, haga clic con el botón derecho en la máquina virtual > **Confirmar**. Esto finaliza el proceso de traslado a la región de destino. Espere hasta que finalice el trabajo de confirmación.

## <a name="discard-the-resource-in-the-source-region"></a>Descarte de los recursos en la región de origen 

- Vaya a la máquina virtual.  Haga clic en **Deshabilitar replicación**.  Esto detiene el proceso de copiar los datos para la máquina virtual.  

   > [!IMPORTANT]
   > Es importante realizar este paso para evitar cargos por replicación de ASR.

En caso de que no tenga planes de volver a utilizar ninguno de los recursos de la fuente, continúe con los siguientes pasos.

1. ¿Desea eliminar todos los recursos de red pertinente en la región de origen que se enumeró como parte del paso 4 en [Preparar las máquinas virtuales de origen](#prepare-the-source-vms) 
2. Elimine la cuenta de almacenamiento correspondiente en la región de origen.



## <a name="next-steps"></a>Pasos siguientes

Siguiendo este tutorial ha trasladado una máquina virtual de Azure a otra región de Azure. Ya puede configurar la opción de recuperación ante desastres para la máquina virtual que ha trasladado.

> [!div class="nextstepaction"]
> [Configurar la recuperación ante desastres después de la migración](azure-to-azure-quickstart.md)
