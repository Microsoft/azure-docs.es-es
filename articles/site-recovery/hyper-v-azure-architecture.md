---
title: Arquitectura de recuperación ante desastres de máquinas virtuales de Hyper-V en Azure Site Recovery
description: En este artículo se proporciona información general sobre los componentes y la arquitectura usados al implementar la recuperación ante desastres de máquinas virtuales de Hyper-V locales (sin VMM) en Azure con el servicio Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: af387b063a3c07d8b6b6c544814565e2a5ebdd46
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495733"
---
# <a name="hyper-v-to-azure-disaster-recovery-architecture"></a>Arquitectura de recuperación ante desastres de Hyper-V en Azure


Este artículo describe la arquitectura y los procesos que se utilizan al replicar, conmutar por error y recuperar máquinas virtuales de Hyper-V entre los hosts de Hyper-V local y Azure, mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

Los hosts de Hyper-V pueden administrarse en nubes privadas de System Center Virtual Machine Manager (VMM).



## <a name="architectural-components---hyper-v-without-vmm"></a>Componentes de la arquitectura: Hyper-V sin VMM

La tabla y el gráfico siguientes proporcionan una visión general de los componentes utilizados para la replicación de Hyper-V en Azure, cuando los hosts de Hyper-V no están administrados por VMM.

**Componente** | **Requisito** | **Detalles**
--- | --- | ---
**Azure** | Suscripción de Azure, cuenta de Azure Storage y red de Azure. | Los datos replicados desde las cargas de trabajo de las máquinas virtuales locales se almacenan en la cuenta de almacenamiento. Las máquinas virtuales de Azure se crean con los datos de la carga de trabajo replicados cuando se produce la conmutación por error desde el sitio local.<br/><br/> Las máquinas virtuales de Azure se conectan a la red virtual de Azure cuando se crean.
**Hyper-V** | Durante la implementación de Site Recovery, se recopilan los clústeres y los hosts de Hyper-V en los sitios de Hyper-V. Se instala el agente de Recovery Services y el Proveedor de Azure Site Recovery en cada host de Hyper-V independiente y en cada nodo de clúster de Hyper-V. | El proveedor organiza la replicación con Site Recovery a través de Internet. El agente de Recovery Services se encarga de la replicación de datos.<br/><br/> Las comunicaciones del proveedor y el agente son seguras y cifradas. También se cifran los datos replicados en el almacenamiento de Azure.
**Máquinas virtuales de Hyper-V** | Una o varias máquinas virtuales en ejecución en Hyper-V. | No es necesario instalar nada explícitamente en las máquinas virtuales.


**Arquitectura de Hyper-V a Azure (sin VMM)**

![Diagrama que muestra el sitio de Hyper-V local en la arquitectura de Azure sin VMM.](./media/hyper-v-azure-architecture/arch-onprem-azure-hypervsite.png)


## <a name="architectural-components---hyper-v-with-vmm"></a>Componentes de la arquitectura: Hyper-V con VMM

La tabla y gráfico siguientes proporcionan una visión general de los componentes utilizados para la replicación de Hyper-V en Azure, cuando los hosts Hyper-V se administran en VMM.

**Componente** | **Requisito** | **Detalles**
--- | --- | ---
**Azure** | Suscripción de Azure, cuenta de Azure Storage y red de Azure. | Los datos replicados desde las cargas de trabajo de las máquinas virtuales locales se almacenan en la cuenta de almacenamiento. Las máquinas virtuales de Azure se crean con los datos replicados cuando se produce la conmutación por error desde el sitio local.<br/><br/> Las máquinas virtuales de Azure se conectan a la red virtual de Azure cuando se crean.
**Servidor VMM** | El servidor de VMM tiene una o varias nubes que contienen hosts de Hyper-V. | Se instala el Proveedor de Site Recovery en el servidor VMM para orquestar la replicación con Site Recovery, y se registra el servidor en el almacén de Recovery Services.
**Host de Hyper-V** | Uno o más hosts o clústeres de Hyper-V administrados por VMM. |  Se instala el agente de Recovery Services en cada nodo del clúster o host de Hyper-V.
**Máquinas virtuales de Hyper-V** | Una o varias máquinas virtuales en un servidor host de Hyper-V. | No es necesario instalar nada explícitamente en las máquinas virtuales.
**Redes** | Redes lógicas y de máquinas virtuales configuradas en el servidor de VMM. La red de máquinas virtuales debe estar vinculada a una red lógica asociada con la nube. | Las redes de máquinas virtuales se asignan a las redes virtuales de Azure. Cuando se crean máquinas virtuales de Azure después de la conmutación por error, se agregan a la red de Azure que está asignada a la red de máquinas virtuales.

**Arquitectura de Hyper-V a Azure (con VMM)**

![Diagrama que muestra el sitio de Hyper-V local en la arquitectura de Azure con VMM.](./media/hyper-v-azure-architecture/arch-onprem-onprem-azure-vmm.png)

## <a name="set-up-outbound-network-connectivity"></a>Configuración de la conectividad de red saliente

Para que Site Recovery funcione de la forma esperada, debe modificar la conectividad de red saliente para permitir la replicación de su entorno.

> [!NOTE]
> Site Recovery no admite el uso de un proxy de autenticación para controlar la conectividad de la red.

### <a name="outbound-connectivity-for-urls"></a>Conectividad de salida para las direcciones URL

Si usa un proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe permitir el acceso a ellas:

| **Nombre**                  | **Comercial**                               | **Gobierno**                                 | **Descripción** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Storage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net`              | Permite que los datos se puedan escribir desde la máquina virtual a la cuenta de almacenamiento de caché en la región de origen. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Proporciona autorización y autenticación de las direcciones URL del servicio Site Recovery. |
| Replicación               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permite que la máquina virtual se comunique con el servicio Site Recovery. |
| Azure Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permite que la máquina virtual escriba los datos de diagnóstico y supervisión de Site Recovery. |


## <a name="replication-process"></a>Proceso de replicación

![Diagrama que muestra el proceso de replicación de Hyper-V en Azure](./media/hyper-v-azure-architecture/arch-hyperv-azure-workflow.png)

**Proceso de replicación y recuperación**


### <a name="enable-protection"></a>Habilitar protección

1. Después de habilitar la protección de una máquina virtual de Hyper-V, en el Azure Portal o de forma local, la opción **Habilitar la protección** se inicia.
2. El trabajo comprueba que la máquina cumpla los requisitos previos e invoca [CreateReplicationRelationship](/windows/win32/hyperv_v2/createreplicationrelationship-msvm-replicationservice) para que configure la replicación con su configuración.
3. El trabajo comienza la replicación inicial con la invocación del método [StartReplication](/windows/win32/hyperv_v2/startreplication-msvm-replicationservice) para inicializar una replicación completa de la máquina virtual y enviar los discos virtuales de la máquina virtual a Azure.
4. Puede supervisar el trabajo en la pestaña **Trabajos**.      ![Captura de pantalla de la lista de trabajos en la pestaña Trabajos.](media/hyper-v-azure-architecture/image1.png) ![Captura de pantalla de la pantalla Habilitar protección con más detalles.](media/hyper-v-azure-architecture/image2.png)


### <a name="initial-data-replication"></a>Replicación inicial de datos

1. Cuando se desencadena la replicación inicial, se toma una [instantánea de la máquina virtual de Hyper-V](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560637(v=ws.10)).
2. Los discos duros virtuales en la máquina virtual se replican uno a uno hasta que se copian todos en Azure. Esto puede tardar unos minutos, en función del tamaño de la máquina virtual y del ancho de banda de la red. [Aprenda a](https://support.microsoft.com/kb/3056159) aumentar el ancho de banda de red.
3. Si se producen cambios en el disco con la replicación inicial en curso, el seguimiento de replicaciones de Réplica de Hyper-V realiza un seguimiento de esos cambios en los registros de replicación de Hyper-V (.hrl). Estos archivos de registro se encuentran en la misma carpeta que los discos. Cada disco tiene un archivo .hrl asociado que se envía al almacenamiento secundario. Los archivos de instantáneas y de registro consumen recursos de disco mientras la replicación inicial está en curso.
4. Cuando finalice la replicación inicial, se elimina la instantánea de la máquina virtual.
5. Los cambios diferenciales en el registro del disco se sincronizan y se combinan en el disco primario.


### <a name="finalize-protection-process"></a>Proceso de finalización de la protección

1. Una vez finalizada la replicación inicial, se ejecuta el trabajo **Finalizar la protección en la máquina virtual**. Configura la red y otras opciones posteriores a la replicación, para que la máquina virtual esté protegida.
2. En esta fase puede comprobar la configuración de la máquina virtual para asegurarse de que está lista para la conmutación por error. Puede ejecutar una exploración de la recuperación ante desastres (conmutación por error de prueba) para la máquina virtual, a fin de comprobar que la conmutación se realiza según lo previsto. 


## <a name="delta-replication"></a>Replicación diferencial

1. Después de la replicación inicial, comienza la diferencial, según la directiva de replicación.
2. La herramienta de seguimiento de la replicación de Réplica de Hyper-V supervisa los cambios realizados en un disco duro virtual como archivos .hrl. Cada disco configurado para la replicación tiene un archivo .hrl asociado.
3. El registro se envía a la cuenta de almacenamiento del cliente. Cuando un registro se encuentra en tránsito hacia Azure, los cambios en el disco principal se siguen en otro archivo de registro de la misma carpeta.
4. Durante la replicación inicial y diferencial, puede supervisar la máquina virtual en Azure Portal.

### <a name="resynchronization-process"></a>Proceso de resincronización

1. Si se produce un error en la replicación diferencial y una replicación completa sería costosa en términos de ancho de banda o de tiempo, se marca una máquina virtual para la resincronización.
    - Por ejemplo, si los archivos .hrl alcanzan el 50 % del tamaño del disco, la máquina virtual se marcará para repetir la sincronización.
    -  De forma predeterminada, la resincronización está programada para ejecutarse automáticamente fuera del horario de oficina.
1.  La resincronización solo envía los datos de diferencias.
    - Minimiza la cantidad de datos que se envían calculando sumas de comprobación de las máquinas virtuales de origen y de destino.
    - Utiliza un algoritmo de fragmentación de bloques fijos donde los archivos de origen y destino se dividen en fragmentos fijos.
    - Se generan sumas de comprobación para cada fragmento. Estas se comparan para determinar qué bloques del origen se deben aplicar al destino.
2. Una vez finalizada la resincronización, se debe reanudar la replicación diferencial normal.
3. Si no desea esperar para volver a sincronizar fuera del horario de forma predeterminada, puede volver a sincronizar una máquina virtual manualmente. Por ejemplo, si se produce una interrupción. Para ello, en Azure Portal, seleccione la máquina virtual > **Volver a sincronizar**.

    ![Captura de pantalla que muestra la opción Volver a sincronizar.](./media/hyper-v-azure-architecture/image4-site.png)


### <a name="retry-process"></a>Reintento del proceso

Si se produce un error de replicación, se realiza un reintento de forma predefinida. El reintento se clasifica como se describe en la tabla.

**Categoría** | **Detalles**
--- | ---
**Errores irrecuperables** | No se realiza ningún reintento. El estado de la máquina virtual será **Crítico** y se requiere la intervención del administrador.<br/><br/> Entre los ejemplos de estos errores se incluyen la interrupción de la cadena de discos duros virtuales, un estado no válido de la máquina virtual de réplica, errores de autenticación de red, errores de autorización, y errores que indican que no se encuentra la máquina virtual (para servidores Hyper-V independientes).
**Errores recuperables** | Los reintentos se producen con cada intervalo de replicación mediante el uso de la interrupción exponencial, lo que aumenta el intervalo de reintentos desde el inicio del primer intento en 1, 2, 4, 8 y 10 minutos. Si el error persiste, reintente cada 30 minutos. Algunos ejemplos son los errores de red, los errores de espacio insuficiente en disco y los errores de memoria insuficiente.



## <a name="failover-and-failback-process"></a>Proceso de conmutación por error y conmutación por recuperación

1. Puede ejecutar una conmutación por error, planeada o no, desde máquinas virtuales de Hyper-V locales en Azure. Si ejecuta una conmutación por error planeada, las máquinas virtuales de origen se apagan para garantizar que no se pierdan datos. Ejecute una conmutación por error no planeada, si el sitio primario no está accesible.
2. Puede conmutar por error una única máquina o crear planes de recuperación para organizar la conmutación por error de varias máquinas.
3. Ejecuta una conmutación por error. Una vez que se completa la primera fase de la conmutación por error, debería poder ver las máquinas virtuales de la réplica creadas en Azure. Puede asignar una dirección IP pública a la máquina virtual si es necesario.
4. Después, confirme la conmutación por error, para iniciar el acceso a la carga de trabajo desde la máquina virtual de Azure de la réplica.

Una vez que la infraestructura local está activa y ejecutándose, puede conmutar por recuperación. La conmutación por recuperación se produce en tres fases:

1. Se inicia una conmutación por error planeada desde Azure en el sitio local:
    - **Minimize downtime** (Minimizar tiempo de inactividad): si usa esta opción, Site Recovery sincroniza los datos antes de la conmutación por error. Comprueba si hay bloques de datos que han cambiado y los descarga en el sitio local, mientras la máquina virtual de Azure se sigue ejecutando, lo que minimiza el tiempo de inactividad. Cuando especifica manualmente que la conmutación por error se debe completar, la máquina virtual de Azure se apaga, los cambios de deferencias finales se copian y se inicia la conmutación por error.
    - **Full download** (Descarga completa): con esta opción, los datos se sincronizan durante la conmutación por error. Esta opción descarga todo el disco. Es más rápido porque no se calculan las sumas de comprobación, pero hay más tiempo de inactividad. Utilice esta opción si ha ejecutado la réplica de máquinas virtuales de Azure durante algún tiempo, o si se ha eliminado la máquina virtual local.
    - **Create VM** (Crear VM): Puede seleccionar conmutar por recuperación en la misma máquina virtual o en otra alternativa. Puede especificar que Site Recovery deba crear la máquina virtual si aún no existe.

2. Una vez finalizada la sincronización inicial, seleccione esta opción para completar la conmutación por error. Una vez que se completa, puede iniciar sesión en la máquina virtual local para comprobar que todo funciona según lo previsto. En Azure Portal, puede ver que las máquinas virtuales de Azure se han detenido.
3.  Después, confirme la conmutación por error para terminar y comience a acceder de nuevo a la carga de trabajo desde la máquina virtual local.
4. Una vez que las cargas de trabajo han conmutado por recuperación, habilite la replicación inversa, para que las máquinas virtuales locales se repliquen en Azure de nuevo.



## <a name="next-steps"></a>Pasos siguientes


Siga [este tutorial](tutorial-prepare-azure.md) para empezar con la replicación de Hyper-V en Azure.


