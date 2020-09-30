---
title: Protección de un servidor de archivos mediante Azure Site Recovery
description: En este artículo se describe cómo proteger un servidor de archivos mediante Azure Site Recovery
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/31/2019
ms.author: sharrai
ms.custom: mvc
ms.openlocfilehash: 1c88a7e78d37556f0f1f89e16154dc1808f24323
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425524"
---
# <a name="protect-a-file-server-by-using-azure-site-recovery"></a>Protección de un servidor de archivos mediante Azure Site Recovery 

[Azure Site Recovery](site-recovery-overview.md) contribuye a la estrategia de recuperación ante desastres y continuidad empresarial (BCDR) al mantener sus aplicaciones empresariales al día y disponibles durante interrupciones planeadas y no planeadas. Azure Site Recovery administra y orquesta la recuperación ante desastres de máquinas locales y máquinas virtuales de Azure (VM). La recuperación ante desastres incluye replicación, la conmutación por error y la recuperación de varias cargas de trabajo.

En este artículo se describe cómo proteger un servidor de archivos mediante Site Recovery y se dan otras recomendaciones para adaptarse a distintos entornos. 

- [Replicación de las máquinas de servidor de archivos de IaaS de Azure](#disaster-recovery-recommendation-for-azure-iaas-virtual-machines)
- [Replicación de un servidor de archivos local con Site Recovery](#replicate-an-on-premises-file-server-by-using-site-recovery)

## <a name="file-server-architecture"></a>Arquitectura del servidor de archivos
El objetivo de un sistema de uso compartido de archivos distribuido y abierto es proporcionar un entorno en el que un grupo de usuarios geográficamente distribuido pueda colaborar para trabajar de forma eficiente en los archivos y garantizar que se cumplen sus requisitos de integridad. Un ecosistema de servidor de archivos local típico que admite un alto número de usuarios simultáneos y un gran número de elementos de contenido usa la Replicación del sistema de archivos distribuido (DFSR) para la programación de la replicación y el límite de ancho de banda. 

DFSR utiliza un algoritmo de compresión que se conoce como compresión diferencial remota (RDC), que se puede usar para actualizar los archivos de forma eficaz a través de una red con ancho de banda limitado. Detecta las inserciones, eliminaciones y reorganizaciones de datos en los archivos. DFSR está habilitado para replicar solo los bloques de archivo modificados cuando se actualizan los archivos. También hay entornos de servidor de archivos en los que se realizan copias de seguridad diarias en intervalos fuera de horas punta, lo que responde a las necesidades de la recuperación ante desastres. DFSR no está implementado.

El diagrama siguiente muestra el entorno de servidor de archivos con DFSR implementado.
        
![Arquitectura de DFSR](media/site-recovery-file-server/dfsr-architecture.JPG)

En el diagrama anterior, varios servidores de archivos, llamados miembros, participan activamente en la replicación de archivos a través de un grupo de replicación. El contenido de la carpeta replicada estará disponible para todos los clientes que envíen solicitudes a cualquiera de los miembros, incluso en el caso de que uno de los miembros esté desconectado.

## <a name="disaster-recovery-recommendations-for-file-servers"></a>Recomendaciones para la recuperación ante desastres para servidores de archivos

* **Replicación de un servidor de archivos con Site Recovery**: Los servidores de archivos se pueden replicar en Azure mediante Site Recovery. Cuando uno o más servidores de archivos locales son inaccesibles, las máquinas virtuales de recuperación se pueden iniciar en Azure. Las máquinas virtuales, a continuación, pueden atender solicitudes de clientes, de forma local, siempre que haya conectividad VPN de sitio a sitio y Active Directory esté configurado en Azure. Puede utilizar este método en el caso de un entorno configurado para DFSR o en un entorno de servidor de archivos simple sin DFSR. 

* **Extensión de DFSR a una máquina virtual de IaaS de Azure**: En un entorno de servidor de archivos en clúster con DFSR implementado, puede extender la implementación de DFSR local a Azure. A continuación, se habilita una máquina virtual de Azure para realizar el rol de servidor de archivos. 

    * Una vez que se administran las dependencias de Active Directory y la conectividad VPN de sitio a sitio y DFSR está en funcionamiento, cuando uno o varios servidores de archivos locales no sean accesibles, los clientes pueden conectarse a la máquina virtual de Azure, que servirá las solicitudes.

    * Puede usar este enfoque si las máquinas virtuales tienen configuraciones que no son compatibles con Site Recovery. Un ejemplo es un disco de clúster compartido, utilizado normalmente en entornos de servidor de archivos. DFSR también funciona correctamente en entornos de ancho de banda reducido con una tasa de renovación intermedia. Debe considerar el costo adicional de tener una máquina virtual de Azure continuamente en ejecución. 

* **Uso de Azure File Sync para replicar los archivos locales**: Si planea usar la nube o ya utiliza una máquina virtual de Azure, puede usar Azure File Sync. Azure File Sync ofrece la sincronización de recursos compartidos de archivos en la nube totalmente administrados, a los que se puede acceder mediante el protocolo estándar [Bloque de mensajes del servidor](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) (SMB). Los recursos compartidos de Azure Files se pueden montar simultáneamente en implementaciones de Windows, Linux y macOS en la nube o locales. 

El siguiente diagrama le ayuda a determinar qué estrategia utilizar para su entorno de servidor de archivos.

![Árbol de decisión](media/site-recovery-file-server/decisiontree.png)


### <a name="factors-to-consider-in-your-decisions-about-disaster-recovery-to-azure"></a>Factores que hay que tener en cuenta en sus decisiones acerca de la recuperación ante desastres en Azure

|Entorno  |Recomendación  |Puntos que se deben tener en cuenta |
|---------|---------|---------|
|Entorno de servidor de archivos con o sin DFSR|   [Uso de Site Recovery para la replicación](#replicate-an-on-premises-file-server-by-using-site-recovery)   |    Site Recovery no es compatible con los clústeres de discos compartidos ni con el almacenamiento conectado a la red (NAS). Si su entorno usa estas configuraciones, utilice cualquiera de los demás enfoques según corresponda. <br> Site Recovery no es compatible con SMB 3.0. La máquina virtual replicada incorpora los cambios solo cuando se actualizan los cambios realizados en los archivos en la ubicación original de los archivos.<br>  Site Recovery ofrece un proceso de replicación de datos casi sincrónico y, por lo tanto, en caso de que se produzca un escenario de conmutación por error no planeada, podría producirse una pérdida de datos y podrían aparecer problemas de falta de coincidencia de USN.
|Entorno de servidor de archivos con DFSR     |  [Extensión de DFSR a una máquina virtual de IaaS de Azure](#extend-dfsr-to-an-azure-iaas-virtual-machine)  |    DFSR funciona correctamente en entornos con el ancho de banda muy restringido. Este enfoque requiere una máquina virtual de Azure activa y en ejecución todo el tiempo. Debe tener en cuenta el costo de la máquina virtual en la planeación.         |
|Máquina virtual de IaaS de Azure     |     File Sync    |     Si utiliza File Sync en un escenario de recuperación ante desastres, durante la conmutación por error debe realizar acciones manuales para asegurarse de que los recursos compartidos de archivos están accesibles para el equipo cliente de forma transparente. File Sync requiere que el puerto 445 esté abierto en el equipo cliente.     |


### <a name="site-recovery-support"></a>Compatibilidad de Site Recovery
Dado que la replicación de Site Recovery no depende de la aplicación, se espera que estas recomendaciones sirvan también para los escenarios siguientes.

| Source  |En un sitio secundario  |En Azure
|---------|---------|---------|
|Azure|  -|Sí|
|Hyper-V|  Sí  |Sí
|VMware  |Sí|  Sí
|Servidor físico|  Sí  |Sí
 

> [!IMPORTANT]
> Antes de continuar con cualquiera de los tres enfoques siguientes, asegúrese de que se han tenido en cuenta estas dependencias.



**Conectividad de sitio a sitio**: Debe establecerse una conexión directa entre el sitio local y la red de Azure para permitir la comunicación entre servidores. Utilice una conexión VPN de sitio a sitio segura con una red virtual de Azure que se utiliza como el sitio de recuperación ante desastres. Para más información, consulte [Establecimiento de una conexión VPN de sitio a sitio entre un sitio local y una red virtual de Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md).

**Active Directory**: DFSR depende de Active Directory. Esto significa que el bosque de Active Directory con controladores de dominio locales se amplía al sitio de recuperación ante desastres en Azure. Incluso si no está utilizando DFSR, si es necesario garantizar o verificar el acceso de los usuarios, debe realizar estos pasos. Para más información, consulte [Extensión de Active Directory local a Azure](./site-recovery-active-directory.md).

## <a name="disaster-recovery-recommendation-for-azure-iaas-virtual-machines"></a>Recomendación de recuperación ante desastres para máquinas virtuales de IaaS de Azure

Si va a configurar y administrar la recuperación ante desastres de servidores de archivos hospedados en máquinas virtuales de IaaS de Azure, puede elegir entre dos opciones, en función de si desea cambiar a [Azure Files](../storage/files/storage-files-introduction.md):

* [Uso de File Sync ](#use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine)
* [Uso de Site Recovery](#replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery)

## <a name="use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine"></a>Uso de File Sync para replicar archivos hospedados en una máquina virtual de IaaS

Azure Files puede utilizarse para reemplazar totalmente o complementar los servidores de archivos tradicionales locales o en dispositivos NAS. Los recursos compartidos de archivos de Azure se pueden replicar también con File Sync en servidores Windows, locales o en la nube, para obtener un almacenamiento en caché eficiente y distribuido de los datos allí donde se usan. Los pasos siguientes describen la recomendación de la recuperación ante desastres para máquinas virtuales de Azure que realizan la misma funcionalidad como servidores de archivos tradicionales:
* Proteja las máquinas mediante Site Recovery. Siga los pasos descritos en [Replicación de una máquina virtual de Azure en otra región de Azure](azure-to-azure-quickstart.md).
* Use File Sync para replicar los archivos desde la máquina virtual que funciona como servidor de archivos en la nube.
* Utilice la característica [plan de recuperación](site-recovery-create-recovery-plans.md) de Site Recovery para agregar scripts para [montar el recurso compartido de archivos de Azure](../storage/files/storage-how-to-use-files-windows.md) y acceda al recurso compartido en la máquina virtual.

Los pasos siguientes describen brevemente el uso de File Sync:

1. [Cree una cuenta de almacenamiento en Azure](../storage/common/storage-account-create.md?toc=/azure/storage/files/toc.json). Si eligió el almacenamiento con redundancia geográfica con acceso de lectura para las cuentas de almacenamiento, tendrá acceso de lectura a los datos desde la región secundaria en caso de desastre. Para más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../storage/common/storage-disaster-recovery-guidance.md?toc=%2fazure%2fstorage%2ffiless%2ftoc.json).
2. [Cree un recurso compartido de archivos](../storage/files/storage-how-to-create-file-share.md).
3. [Implementación de File Sync](../storage/files/storage-sync-files-deployment-guide.md) en el servidor de archivos de Azure.
4. Crear un grupo de sincronización. Los puntos de conexión dentro de un grupo de sincronización se mantienen sincronizados entre sí. Un grupo de sincronización debe contener al menos un punto de conexión en la nube, que representa un recurso compartido de archivos de Azure. Un grupo de sincronización también debe contener un punto de conexión de servidor, que representa una ruta de acceso en un servidor Windows.
5. Los archivos ahora se mantienen sincronizados entre el recurso compartido de archivos de Azure y el servidor local.
6. Si se produce un desastre en el entorno local, realice una conmutación por error mediante un [plan de recuperación](site-recovery-create-recovery-plans.md). Agregue el script para [montar el recurso compartido de archivos de Azure](../storage/files/storage-how-to-use-files-windows.md) y acceder al recurso compartido en la máquina virtual.

### <a name="replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery"></a>Replicación de una máquina virtual de servidor de archivos de IaaS mediante Site Recovery

Si tiene clientes locales que tienen acceso a la máquina virtual de servidor de archivos de IaaS, realice los siguientes pasos. En caso contrario, vaya al paso 3.

1. Establezca una conexión VPN de sitio a sitio entre el sitio local y la red de Azure.
2. Amplíe Active Directory local.
3. [Configure la recuperación ante desastres](azure-to-azure-tutorial-enable-replication.md) para la máquina del servidor de archivos IaaS en una región secundaria.


Para más información sobre la recuperación ante desastres en una región secundaria, consulte [este artículo](./azure-to-azure-architecture.md).


## <a name="replicate-an-on-premises-file-server-by-using-site-recovery"></a>Replicación de un servidor de archivos local con Site Recovery

Los siguientes pasos describen la replicación de una máquina virtual de VMware. Para conocer los pasos para replicar una máquina virtual de Hyper-V, consulte [este tutorial](./hyper-v-azure-tutorial.md).

1. [Prepare recursos de Azure](tutorial-prepare-azure.md) para la replicación de máquinas locales.
2. Establezca una conexión VPN de sitio a sitio entre el sitio local y la red de Azure. 
3. Amplíe Active Directory local.
4. [Prepare servidores de VMware locales](./vmware-azure-tutorial-prepare-on-premises.md).
5. [Configure la recuperación ante desastres](./vmware-azure-tutorial.md) para máquinas virtuales de VMware locales en Azure.

## <a name="extend-dfsr-to-an-azure-iaas-virtual-machine"></a>Extensión de DFSR a una máquina virtual de IaaS de Azure

1. Establezca una conexión VPN de sitio a sitio entre el sitio local y la red de Azure. 
2. Amplíe Active Directory local.
3. [Cree y aprovisione una máquina virtual de servidor de archivos](../virtual-machines/windows/quick-create-portal.md?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json) en la red virtual de Azure.
Asegúrese de que la máquina virtual esté agregada a la misma red virtual de Azure, la cual dispone de conectividad cruzada con el entorno local. 
4. Instale y [configure DFSR](https://techcommunity.microsoft.com/t5/storage-at-microsoft/dfs-replication-initial-sync-in-windows-server-2012-r2-attack-of/ba-p/424877) en Windows Server.
5. [Implemente un espacio de nombres DFS](/windows-server/storage/dfs-namespaces/deploying-dfs-namespaces).
6. Con el espacio de nombres DFS implementado, la conmutación por error de las carpetas compartidas de producción a los sitios de recuperación ante desastres puede realizarse actualizando los destinos de las carpetas del espacio de nombres DFS. Una vez que estos cambios del espacio de nombres DFS se replican con Active Directory, los usuarios se conectan a los destinos de carpeta apropiados de forma transparente.

## <a name="use-file-sync-to-replicate-your-on-premises-files"></a>Uso de File Sync para replicar los archivos locales
Puede utilizar File Sync para replicar los archivos en la nube. En el caso de que se produzca un desastre o el servidor de archivos local no esté disponible, puede montar las ubicaciones de archivos deseadas desde la nube y continuar con solicitudes de servicio desde los equipos clientes.
Para integrar File Sync con Site Recovery:

* Proteja las máquinas de servidor de archivos mediante Site Recovery. Siga los pasos de [este tutorial](./vmware-azure-tutorial.md).
* Use File Sync para replicar los archivos desde la máquina que funciona como servidor de archivos en la nube.
* Utilice la característica de plan de recuperación de Site Recovery para agregar los scripts para montar el recurso compartido de archivos de Azure en la máquina virtual de servidor de archivos a la que se ha conmutado por error en Azure.

Siga estos pasos para utilizar File Sync:

1. [Cree una cuenta de almacenamiento en Azure](../storage/common/storage-account-create.md?toc=/azure/storage/files/toc.json). Si eligió el almacenamiento con redundancia geográfica con acceso de lectura (recomendado) para las cuentas de almacenamiento, tendrá acceso de lectura a los datos desde la región secundaria en caso de desastre. Para más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../storage/common/storage-disaster-recovery-guidance.md?toc=%2fazure%2fstorage%2ffiless%2ftoc.json).
2. [Cree un recurso compartido de archivos](../storage/files/storage-how-to-create-file-share.md).
3. [Implemente File Sync](../storage/files/storage-sync-files-deployment-guide.md) en el servidor de archivos local.
4. Crear un grupo de sincronización. Los puntos de conexión dentro de un grupo de sincronización se mantienen sincronizados entre sí. Un grupo de sincronización debe contener al menos un punto de conexión en la nube, que representa un recurso compartido de archivos de Azure. Un grupo de sincronización también debe contener un punto de conexión de servidor, que representa una ruta de acceso en el servidor Windows local.
5. Los archivos ahora se mantienen sincronizados entre el recurso compartido de archivos de Azure y el servidor local.
6. Si se produce un desastre en el entorno local, realice una conmutación por error mediante un [plan de recuperación](site-recovery-create-recovery-plans.md). Agregue el script para montar el recurso compartido de archivos de Azure y acceder al recurso compartido en la máquina virtual.

> [!NOTE]
> Asegúrese de que el puerto 445 está abierto. Azure Files usa el protocolo SMB. SMB se comunica a través del puerto TCP 445. Compruebe si el firewall no está bloqueando el puerto TCP 445 desde un equipo cliente.


## <a name="do-a-test-failover"></a>Ejecución de una conmutación por error de prueba

1. Vaya a Azure Portal y seleccione el almacén de Recovery Services.
2. Seleccione el plan de recuperación creado para el entorno del servidor de archivos.
3. Seleccione **Conmutación por error de prueba**.
4. Para iniciar el proceso de conmutación por error de prueba, seleccione el punto de recuperación y la red virtual de Azure.
5. Cuando el entorno secundario esté activo, podrá realizar las validaciones.
6. Después de realizar las validaciones, seleccione **Limpiar conmutación por error de prueba** en el plan de recuperación para limpiar el entorno de la conmutación por error de prueba.

Para más información acerca de la realización de una conmutación por error de prueba, consulte [Conmutación por error de prueba en Site Recovery](site-recovery-test-failover-to-azure.md).

Para obtener instrucciones sobre cómo realizar la conmutación por error de prueba para Active Directory y DNS, consulte las [consideraciones sobre la conmutación por error de prueba para Active Directory y DNS](site-recovery-active-directory.md).

## <a name="do-a-failover"></a>Ejecución de una conmutación por error

1. Vaya a Azure Portal y seleccione el almacén de Recovery Services.
2. Seleccione el plan de recuperación creado para el entorno del servidor de archivos.
3. Seleccione **Failover** (Conmutación por error).
4. Para iniciar el proceso de conmutación por error, seleccione el punto de recuperación.

Para más información acerca de cómo realizar una conmutación por error, consulte [Conmutación por error en Site Recovery](site-recovery-failover.md).
