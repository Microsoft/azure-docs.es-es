---
title: Preparación de máquinas para la migración con Azure Migrate
description: Aprenda a preparar máquinas locales para la migración con Azure Migrate.
ms.topic: tutorial
ms.date: 06/08/2020
ms.custom: MVC
ms.openlocfilehash: b92a26732f59235dac4c03f4e648d36dadd6c4ac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077981"
---
# <a name="prepare-on-premises-machines-for-migration-to-azure"></a>Preparación de las máquinas locales para la migración a Azure

En este artículo se describe cómo preparar máquinas en el entorno local antes de migrarlas a Azure con la herramienta [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).

En este artículo:
> [!div class="checklist"]
> * Compruebe las limitaciones de la migración.
> * Seleccione un método para migrar las máquinas virtuales VMware.
> * Compruebe los requisitos del hipervisor y del sistema operativo para las máquinas que desea migrar.
> * Revise el acceso a la dirección URL y el puerto en las máquinas que desea migrar.
> * Revise los cambios que debe realizar antes de comenzar la migración.
> * Compruebe los requisitos para máquinas virtuales de Azure de las máquinas migradas.
> * Prepare las máquinas para que pueda conectarse a las máquinas virtuales de Azure después de la migración.



## <a name="verify-migration-limitations"></a>Comprobación de los límites de migración

En la tabla se resumen los límites de detección, evaluación y migración de Azure Migrate. Se recomienda evaluar las máquinas antes de la migración, aunque no es obligatorio.

**Escenario** | **Proyecto** | **Detección y evaluación** | **Migración**
--- | --- | --- | ---
**Máquinas virtuales de VMware** | Detecte y evalúe hasta 35 000 máquinas virtuales en un solo proyecto de Azure Migrate. | Detecte hasta 10 000 máquinas virtuales VMware con un solo [dispositivo Azure Migrate](common-questions-appliance.md) para VMware. | **Migración sin agente**: puede replicar simultáneamente un máximo de 300 máquinas virtuales. Para obtener el mejor rendimiento, se recomienda crear varios lotes de máquinas virtuales si tiene más de 50.<br/><br/> **Migración basada en agente**: puede [escalar horizontalmente](./agent-based-migration-architecture.md#performance-and-scaling) el [dispositivo de replicación](migrate-replication-appliance.md) para replicar un gran número de máquinas virtuales.<br/><br/> En el portal puede seleccionar hasta 10 máquinas virtuales a la vez para la replicación. Para replicar más máquinas, agregue lotes de 10.
**Máquinas virtuales de Hyper-V** | Detecte y evalúe hasta 35 000 máquinas virtuales en un solo proyecto de Azure Migrate. | Detecte hasta 5 000 máquinas virtuales Hyper-V con un único dispositivo Azure Migrate. | No se utiliza un dispositivo para la migración de Hyper-V. En su lugar, el proveedor de replicación de Hyper-V se ejecuta en cada host de Hyper-V.<br/><br/> La capacidad de replicación se ve afectada por factores de rendimiento como la renovación de máquinas virtuales y la carga de ancho de banda de los datos de replicación.<br/><br/> En el portal puede seleccionar hasta 10 máquinas virtuales a la vez para la replicación. Para replicar más máquinas, agregue lotes de 10.
**máquinas físicas** | Detecte y evalúe hasta 35 000 máquinas en un solo proyecto de Azure Migrate. | Detecte hasta 250 servidores físicos con un único dispositivo Azure Migrate para servidores físicos. | Puede [escalar horizontalmente](./agent-based-migration-architecture.md#performance-and-scaling) el [dispositivo de replicación](migrate-replication-appliance.md) para replicar un gran número de servidores.<br/><br/> En el portal puede seleccionar hasta 10 máquinas virtuales a la vez para la replicación. Para replicar más máquinas, agregue lotes de 10.

## <a name="select-a-vmware-migration-method"></a>Selección de un método de migración de VMware

Si va a migrar máquinas virtuales VMware a Azure, [compare](server-migrate-overview.md#compare-migration-methods) los métodos de migración basados en agente y sin agente para decidir el más adecuado.

## <a name="verify-hypervisor-requirements"></a>Comprobación de los requisitos del hipervisor

- Compruebe los requisitos de la migración [sin agente de VMware](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) o [basada en agente de VMware](migrate-support-matrix-vmware-migration.md#vmware-requirements-agent-based).
- Compruebe los requisitos del [host de Hyper-V](migrate-support-matrix-hyper-v-migration.md#hyper-v-host-requirements).


## <a name="verify-operating-system-requirements"></a>Comprobación de los requisitos de sistema operativo

Compruebe los sistemas operativos compatibles para la migración:

- Si va a migrar máquinas virtuales VMware o máquinas virtuales Hyper-V, compruebe los requisitos de máquinas virtuales VMware para la migración [sin agente](migrate-support-matrix-vmware-migration.md#vm-requirements-agentless) y [basada en agente](migrate-support-matrix-vmware-migration.md#vm-requirements-agent-based), así como los requisitos de las [máquinas virtuales Hyper-V](migrate-support-matrix-hyper-v-migration.md#hyper-v-vms).
- Compruebe que los [sistemas operativos Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines) se admiten en Azure.
- Compruebe que las [distribuciones de Linux](../virtual-machines/linux/endorsed-distros.md) se admiten en Azure.

## <a name="review-url-and-port-access"></a>Revisión del acceso a la dirección URL y al puerto

Revise las direcciones URL y los puertos a los que se tiene acceso durante la migración.

**Escenario** | **Detalles** |  **URLs** | **Puertos**
--- | --- | --- | ---
**Migración sin agente de VMware** | Utiliza el [dispositivo Azure Migrate](migrate-appliance-architecture.md) para la migración. No se instala nada en las máquinas virtuales VMware. | Revise las [direcciones URL](migrate-appliance.md#url-access) de la nube pública y las del gobierno necesarias para la detección, evaluación y migración con el dispositivo. | [Revise](migrate-support-matrix-vmware-migration.md#port-requirements-agentless) los requisitos de puertos para la migración sin agente.
**Migración basada en agente de VMware** | Usa el [dispositivo de replicación](migrate-replication-appliance.md) para la migración. El agente del servicio Mobility se instala en las máquinas virtuales. | Revise las direcciones URL de la [nube pública](migrate-replication-appliance.md#url-access) y de [Azure Government](migrate-replication-appliance.md#azure-government-url-access) a las que necesita acceder el dispositivo de replicación. | [Revise](migrate-replication-appliance.md#port-access) los puertos utilizados durante la migración basada en agente.
**Migración de Hyper-V** | Usa un proveedor instalado en los hosts de Hyper-V para la migración. No se instala nada en las máquinas virtuales Hyper-V. | Revise las direcciones URL de la [nube pública](migrate-support-matrix-hyper-v-migration.md#url-access-public-cloud) y de [Azure Government](migrate-support-matrix-hyper-v-migration.md#url-access-azure-government) a las que necesita acceder el proveedor de replicación que se ejecuta en los hosts. | El proveedor de replicación del host de Hyper-V usa conexiones salientes en el puerto HTTPS 443 para enviar los datos de replicación de la máquina virtual.
**máquinas físicas** | Usa el [dispositivo de replicación](migrate-replication-appliance.md) para la migración. El agente del servicio Mobility se instala en las máquinas físicas. | Revise las direcciones URL de la [nube pública](migrate-replication-appliance.md#url-access) y de [Azure Government](migrate-replication-appliance.md#azure-government-url-access) a las que necesita acceder el dispositivo de replicación. | [Revise](migrate-replication-appliance.md#port-access) los puertos utilizados durante la migración física.

## <a name="verify-required-changes-before-migrating"></a>Comprobación de los cambios necesarios antes de la migración

Hay algunos cambios necesarios en las máquinas virtuales antes de migrarlas a Azure.

- En el caso de algunos sistemas operativos, Azure Migrate realiza cambios automáticamente durante el proceso de replicación o migración.
- En el caso de otros sistemas operativos, debe configurar manualmente las opciones.
- Es importante configurar las opciones manualmente antes de comenzar la migración. Si migra la máquina virtual antes de realizar el cambio, es posible que la máquina virtual no arranque en Azure.

Revise las tablas para identificar los cambios que debe realizar.

### <a name="windows-machines"></a>Máquinas de Windows

En la tabla se resumen los cambios necesarios.

**Acción** | **VMware (migración sin agente)** | **VMware (basada en agente)/máquinas físicas** | **Windows en Hyper-V** 
--- | --- | --- | ---
**Configurar la directiva de SAN como Todo en línea**<br/><br/> Esto asegura que los volúmenes de Windows en la máquina virtual de Azure usan las mismas asignaciones de letra de unidad que la máquina virtual local. | Se establece automáticamente para máquinas que ejecutan Windows Server 2008 R2 o posterior.<br/><br/> Configure la opción manualmente para sistemas operativos anteriores. | Se establece automáticamente en la mayoría de los casos. | Configure de forma manual.
**Instalar la integración de invitado de Hyper-V** | [Instale manualmente](prepare-windows-server-2003-migration.md#install-on-vmware-vms) en máquinas que ejecutan Windows Server 2003. | [Instale manualmente](prepare-windows-server-2003-migration.md#install-on-vmware-vms) en máquinas que ejecutan Windows Server 2003. | [Instale manualmente](prepare-windows-server-2003-migration.md#install-on-hyper-v-vms) en máquinas que ejecutan Windows Server 2003.
**Habilitar la consola serie de Azure**.<br/><br/>[Habilite la consola](../virtual-machines/troubleshooting/serial-console-windows.md) en las máquinas virtuales de Azure para ayudar en la solución de problemas. No es necesario reiniciar la máquina virtual. La máquina virtual de Azure se iniciará mediante la imagen de disco. El arranque desde la imagen de disco equivale a un reinicio de la nueva máquina virtual. | Habilitar manualmente | Habilitar manualmente | Habilitar manualmente
**Conectarse después de la migración**<br/><br/> Para conectarse después de la migración, hay que realizar una serie de pasos antes de la migración. | [Configure](#prepare-to-connect-to-azure-windows-vms) manualmente. | [Configure](#prepare-to-connect-to-azure-windows-vms) manualmente. | [Configure](#prepare-to-connect-to-azure-windows-vms) manualmente.


#### <a name="configure-san-policy"></a>Configuración de la directiva de SAN

De forma predeterminada, las máquinas virtuales de Azure tienen asignada la unidad D para su uso como almacenamiento temporal.

- Esta asignación de unidad hace que todas las demás asignaciones de unidad de almacenamiento asociadas aumenten en una letra.
- Por ejemplo, si la instalación local usa un disco de datos que se asigna a la unidad D para instalaciones de aplicaciones, la asignación de esta unidad se incrementa a la unidad E después de haber migrado la máquina virtual a Azure. 
- Para evitar esta asignación automática y para asegurarse de que Azure asigna la siguiente letra de unidad libre a su volumen temporal, establezca la directiva de red de área de almacenamiento (SAN) en **OnlineAll**:

Configure esta opción manualmente de la siguiente manera:

1. En la máquina local (no en el servidor host), abra un símbolo del sistema con privilegios elevados.
2. Escriba **diskpart**.
3. Escriba **SAN**. Si no se mantiene la letra de unidad del sistema operativo invitado, se devuelven **Offline All** u **Offline Shared**.
4. En la petición de **DISKPART**, escriba **SAN Policy=OnlineAll**. Esta configuración garantiza que los discos se ponen en línea y que se puede leer y escribir en ambos discos.
5. Durante la migración de prueba, puede comprobar que se conserven las letras de unidad.


### <a name="linux-machines"></a>Equipos con Linux

Azure Migrate completa automáticamente estas acciones para estas versiones

- Red Hat Enterprise Linux 7.0+, 6.5+
- CentOS 7.0+, 6.5+
- SUSE Linux Enterprise Server 12 SP1+
- Ubuntu 18.04LTS, 16.04LTS, 14.04LTS
- Debian 8, 7

Para otras versiones, prepare las máquinas como se resume en la tabla.  


**Acción** | **Detalles** | **Versión de Linux**
--- | --- | ---
**Instalar los servicios de integración de Linux de Hyper-V** | Recompile la imagen init de Linux para que contenga los controladores de Hyper-V necesarios. La recompilación de la imagen init garantiza que la máquina virtual se iniciará en Azure. | La mayoría de las nuevas versiones de las distribuciones de Linux lo incluyen de forma predeterminada.<br/><br/> Si no se ha incluido, instálelo manualmente para todas las versiones excepto las mencionadas anteriormente.
**Habilitar el registro de la consola serie de Azure** | La habilitación del registro de la consola ayuda a solucionar problemas. No es necesario reiniciar la máquina virtual. La máquina virtual de Azure se iniciará mediante la imagen de disco. El arranque desde la imagen de disco equivale a un reinicio de la nueva máquina virtual.<br/><br/> Siga [estas instrucciones](../virtual-machines/troubleshooting/serial-console-linux.md) para habilitarlo.
**Actualizar el archivo de asignación de dispositivos** | Actualice el archivo de asignación de dispositivos que contiene las asociaciones de nombre de dispositivo y volumen para usar identificadores de dispositivo persistentes. | Instálelo manualmente para todas las versiones excepto las mencionadas anteriormente. (Solo se aplica en el escenario de VMware basado en agente)
**Actualizar las entradas de fstab** |  Actualice las entradas para utilizar identificadores de volumen persistentes.    | Actualícelo manualmente para todas las versiones excepto las mencionadas anteriormente.
**Eliminar regla udev** | Elimine las reglas udev que reserven los nombres de interfaz en función de la dirección MAC, etc. | Elimínelas de forma manual para todas las versiones excepto las mencionadas anteriormente.
**Actualizar las interfaces de red** | Actualice las interfaces de red para recibir direcciones IP basadas en DHCP. | Actualícelo manualmente para todas las versiones excepto las mencionadas anteriormente.
**Habilitar SSH** | Asegúrese de que SSH está habilitado y de que el servicio sshd está configurado para iniciarse automáticamente al reiniciar.<br/><br/> Asegúrese de que las solicitudes de conexión SSH entrantes no estén bloqueadas por el firewall del sistema operativo o las reglas que admiten scripts.| Habilite la opción manualmente para todas las versiones excepto las mencionadas anteriormente.

En la tabla siguiente se resumen los pasos que se realizan automáticamente para los sistemas operativos enumerados anteriormente.

| Acción                                      | Migración de \-VMware basada en el agente | Migración de VMware sin agente | Hyper\-V   |
|---------------------------------------------|-------------------------------|----------------------------|------------|
| Instalación de los servicios de integración de Linux de Hyper-V\- | Sí                           | Sí                        | No es necesario |
| Habilitación del registro de la consola serie de Azure         | Sí                           | Sí                        | No         |
| Actualizar el archivo de asignación de dispositivos                      | Sí                           | No                         | No         |
| Actualizar las entradas de fstab                        | Sí                           | Sí                        | No         |
| Eliminar reglas udev                            | Sí                           | Sí                        | No         |
| Actualizar las interfaces de red                   | Sí                           | Sí                        | No         |
| Habilitar SSH                                  | No                            | No                         | No         |

Más información sobre los pasos para [ejecutar una máquina virtual Linux en Azure](../virtual-machines/linux/create-upload-generic.md) y obtenga instrucciones para algunas de las distribuciones de Linux más populares.


## <a name="check-azure-vm-requirements"></a>Comprobación de los requisitos de la máquina virtual de Azure

Las máquinas locales que se replican en Azure deben cumplir los requisitos de las máquinas virtuales de Azure para el sistema operativo y la arquitectura, los discos, la configuración de red y la nomenclatura de la máquina virtual.

Antes de realizar la migración, revise los requisitos de las máquinas virtuales de Azure para la migración de [VMware](migrate-support-matrix-vmware-migration.md#azure-vm-requirements), de [Hyper-V](migrate-support-matrix-hyper-v-migration.md#azure-vm-requirements) y de [servidor físico](migrate-support-matrix-physical-migration.md#azure-vm-requirements).



## <a name="prepare-to-connect-after-migration"></a>Preparación para la conexión después de la migración

Las máquinas virtuales de Azure se crean durante la migración a Azure. Después de la migración, debe ser capaz de conectarse a las nuevas máquinas virtuales de Azure. Se requieren varios pasos para conectarse correctamente.

### <a name="prepare-to-connect-to-azure-windows-vms"></a>Preparación de la conexión a las máquinas virtuales Windows de Azure

En máquinas Windows locales:

1. Configure Windows. La configuración incluye la eliminación de cualquier ruta o proxy WinHTTP persistente estático.
2. Asegúrese de que se están ejecutando los [servicios necesarios](../virtual-machines/windows/prepare-for-upload-vhd-image.md#check-the-windows-services).
3. Habilite el protocolo de escritorio remoto (RDP) para permitir las conexiones remotas a la máquina local. Aprenda a [usar PowerShell para habilitar RDP](../virtual-machines/windows/prepare-for-upload-vhd-image.md#update-remote-desktop-registry-settings).
4. Para acceder a una máquina virtual de Azure mediante Internet tras la migración, en el firewall de Windows de la máquina local, permita TCP y UDP en el perfil público y establezca el protocolo de escritorio remoto como aplicación permitida para todos los perfiles.
5. Si quiere acceder a una máquina virtual de Azure mediante una conexión VPN de sitio a sitio, en el firewall de Windows de la máquina local, permita el protocolo de escritorio remoto para los perfiles de dominio y privado. Aprenda a [permitir el tráfico RDP](../virtual-machines/windows/prepare-for-upload-vhd-image.md#configure-windows-firewall-rules). 
6. Asegúrese de que no haya actualizaciones de Windows pendientes en la máquina virtual local al migrarla. Si es el caso, las actualizaciones podrían empezar a instalarse en la máquina virtual de Azure después de la migración y no podrá iniciar sesión en ella hasta que finalicen.


### <a name="prepare-to-connect-with-linux-azure-vms"></a>Preparación para la conexión a las máquinas virtuales Linux en Azure

En máquinas Linux locales:

1. Asegúrese de que el servicio de Secure Shell está configurado para iniciarse automáticamente al arrancar el sistema.
2. Compruebe que las reglas de firewall permiten una conexión SSH.

### <a name="configure-azure-vms-after-migration"></a>Configuración de máquinas virtuales de Azure después de la migración

Después de la migración, lleve a cabo estos pasos en las máquinas virtuales de Azure que se creen:

1. Para conectarse a la máquina virtual desde Internet, asígnele una dirección IP pública. Debe usar una dirección IP pública diferente para la máquina virtual de Azure de la que usó para la máquina local. [Más información](../virtual-network/virtual-network-public-ip-address.md).
2. Compruebe que las reglas del grupo de seguridad de red (NSG) en la máquina virtual permiten las conexiones entrantes al puerto RDP o SSH.
3. Compruebe los [diagnósticos de arranque](../virtual-machines/troubleshooting/boot-diagnostics.md#enable-boot-diagnostics-on-existing-virtual-machine) para ver la máquina virtual.


## <a name="next-steps"></a>Pasos siguientes

Decida qué método desea usar para la [migración de las máquinas virtuales de VMware](server-migrate-overview.md) a Azure o comience la migración de [máquinas virtuales de Hyper-V](tutorial-migrate-hyper-v.md) o de [servidores físicos o virtualizados o máquinas virtuales en la nube](tutorial-migrate-physical-virtual-machines.md).

## <a name="see-whats-supported"></a>¿Qué se admite?

En máquinas virtuales de VMware, Server Migration admite la [migración con agente y sin agente](server-migrate-overview.md).

- **VM de VMware**: compruebe los [requisitos y el soporte técnico necesarios para la migración](migrate-support-matrix-vmware-migration.md) de las máquinas virtuales de VMware.
- **VM de Hyper-V**: Compruebe los [requisitos y el soporte técnico necesarios para la migración](migrate-support-matrix-hyper-v-migration.md) de las máquinas virtuales de Hyper-V.
- **Máquinas físicas**: compruebe los [requisitos y el soporte técnico necesarios para la migración](migrate-support-matrix-physical-migration.md) de máquinas físicas locales y otros servidores virtualizados. 
