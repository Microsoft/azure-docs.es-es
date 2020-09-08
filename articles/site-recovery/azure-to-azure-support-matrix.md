---
title: Matriz de soporte para la recuperación ante desastres de máquinas virtuales de Azure con Azure Site Recovery
description: Resume la compatibilidad con la recuperación ante desastres de máquinas virtuales de Azure en una región secundaria con Azure Site Recovery.
ms.topic: article
ms.date: 07/14/2020
ms.author: raynew
ms.openlocfilehash: 3006522f75ed732c08e453a266e660cf4c577917
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815375"
---
# <a name="support-matrix-for-azure-vm-disaster-recovery-between-azure-regions"></a>Matriz de soporte para la recuperación ante desastres de máquinas virtuales de Azure entre regiones de Azure

En este artículo se resumen los requisitos previos y la compatibilidad para la recuperación ante desastres de máquinas virtuales de Azure de una región de Azure a otra mediante el servicio [Azure Site Recovery](site-recovery-overview.md).


## <a name="deployment-method-support"></a>Compatibilidad de método de implementación

**Implementación** |  **Soporte técnico**
--- | ---
**Azure Portal** | Compatible.
**PowerShell** | Compatible. [Más información](azure-to-azure-powershell.md)
**REST API** | Compatible.
**CLI** | No se admite actualmente.


## <a name="resource-support"></a>Compatibilidad de recursos

**Acción de recursos** | **Detalles**
--- | ---
**Mover almacenes entre grupos de recursos** | No compatible
**Mover recursos de proceso, almacenamiento y red entre grupos de recursos** | No compatible.<br/><br/> Si mueve una máquina virtual o los componentes asociados, como el almacenamiento o la red, después de la replicación de esta, tendrá que deshabilitar la replicación y volver a habilitarla para la máquina virtual.
**Replicar máquinas virtuales de Azure de una suscripción a otra para la recuperación ante desastres** | Se admite en el mismo inquilino de Azure Active Directory.
**Migración de máquinas virtuales entre regiones dentro de los clústeres geográficos admitidos (dentro y entre suscripciones)** | Se admite en el mismo inquilino de Azure Active Directory.
**Migrar máquinas virtuales dentro de la misma región** | No compatible.

## <a name="region-support"></a>Regiones admitidas

Puede replicar y recuperar máquinas virtuales entre dos regiones cualesquiera dentro del mismo clúster geográfico. Los clústeres geográficos se definen teniendo presente la latencia y la soberanía de datos.


**Clúster geográfico** | **Regiones de Azure**
-- | --
America | Centro de Canadá y Este de Canadá, Centro-sur de EE. UU.., centro-oeste de EE. UU., este de EE. UU., este de EE. UU. 2, oeste de EE. UU., oeste de EE. UU. 2 centro de EE. UU., centro-norte de EE. UU.
Europa | Oeste de Reino Unido, Sur de Reino Unido, Norte de Europa, Oeste de Europa, Oeste de Sudáfrica, Norte de Sudáfrica, Este de Noruega, Oeste de Noruega, Centro de Francia
Asia | Sur de la India, Centro de la India, India occidental, Sudeste de Asia, Este de Asia, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Sur de Corea del Sur
Australia    | Este de Australia, Sudeste de Australia, Centro de Australia, Centro de Australia 2
Azure Government    | Virginia Gob. EE. UU., Iowa Gob. EE. UU., US Gov de Arizona, US Gov de Texas, US DOD este, US DOD central
Alemania    | Centro de Alemania y Nordeste de Alemania
China | Este de China, Norte de China, Norte de China 2 y Este de China 2
Regiones restringidas reservadas para la recuperación ante desastres dentro de un país |La región Norte de Alemania está reservada para Centro-oeste de Alemania, la región Oeste de Suiza está reservada para Norte de Suiza, la región Sur de Francia está reservada para Centro de Francia y la región Centro de Emiratos Árabes Unidos está restringida para los clientes de Norte de Emiratos Árabes Unidos.

>[!NOTE]
>
> - En la región **Sur de Brasil**, puede replicar y conmutar por error en las siguientes regiones: Centro-sur de EE. UU., Centro-oeste de EE. UU., Este de EE. UU., Este de EE. UU. 2, Oeste de EE. UU., Oeste de EE. UU. 2 y Centro-norte de EE. UU.
> - La región Sur de Brasil solo puede utilizarse como la región de origen desde la que las máquinas virtuales replican mediante Site Recovery. No puede ser una región de destino. Esto se debe a problemas de latencia provocados por las distancias geográficas. Tenga en cuenta que si realiza la conmutación por error desde Sur de Brasil como región de origen a un destino, se admite la conmutación por recuperación a Sur de Brasil desde la región de destino.
> - Puede trabajar dentro de las regiones para las que tenga un acceso adecuado.
> - Si la región donde quiere crear un almacén no aparece, asegúrese de que su suscripción tiene acceso para crear recursos en esa región.
> - Si no ve una región dentro de un clúster geográfico al habilitar la replicación, asegúrese de que su suscripción tiene permisos para crear máquinas virtuales en dicha región.



## <a name="cache-storage"></a>Almacenamiento en caché

Esta tabla resume la compatibilidad con la cuenta de almacenamiento de la memoria caché que usa Site Recovery durante la replicación.

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
Cuentas de almacenamiento de uso general V2 (capas de acceso frecuente y esporádico) | Compatible | Se desaconseja usar GPv2, ya que los costes de las transacciones de V2 son significativamente más altos que los de las cuentas de almacenamiento de V1.
Premium Storage | No compatible | Las cuentas de almacenamiento estándar se usan para el almacenamiento en caché, para ayudar a optimizar los costos.
Firewalls de Azure Storage para redes virtuales  | Compatible | Si usa una cuenta de almacenamiento en caché o de destino con firewall habilitado, asegúrese de seleccionar ["Permitir servicios de Microsoft de confianza"](../storage/common/storage-network-security.md#exceptions).<br></br>Además, asegúrese de permitir el acceso a al menos una subred de Vnet de origen.


## <a name="replicated-machine-operating-systems"></a>Sistemas operativos de máquinas replicadas

Site Recovery admite la replicación de máquinas virtuales de Azure que ejecutan los sistemas operativos enumerados en esta sección. Tenga en cuenta que si una máquina que ya se está replicando se actualiza posteriormente (o se cambia a una versión anterior) a un kernel principal diferente, deberá deshabilitar la replicación y volverla a habilitar después de la actualización.

### <a name="windows"></a>Windows


**Sistema operativo** | **Detalles**
--- | ---
Windows Server 2019 | Se admite para Server Core y Server con Experiencia de escritorio.
Windows Server 2016  | Se admite para Server Core y Server con Experiencia de escritorio
Windows Server 2012 R2 | Compatible.
Windows Server 2012 | Compatible.
Windows Server 2008 R2 con SP1/SP2 | Compatible.<br/><br/> Desde la versión [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de la extensión de servicio de Mobility Service para máquinas virtuales de Azure, necesita instalar una [actualización de la pila de servicio (SSU)](https://support.microsoft.com/help/4490628) de Windows y la [actualización de SHA-2](https://support.microsoft.com/help/4474419) en las máquinas que ejecutan Windows Server 2008 R2 SP1/SP2.  SHA-1 no se admite desde septiembre de 2019 y, si la firma de código SHA-2 no está habilitada, la extensión del agente no se instalará ni actualizará según lo previsto. Más información sobre los [requisitos y la actualización de SHA-2](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Compatible.
Windows 8.1 (x64) | Compatible.
Windows 8 (x64) | Compatible.
Windows 7 (x64) con SP1 en adelante | Desde la versión [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de la extensión de servicio de Mobility Service para máquinas virtuales de Azure, necesita instalar una [actualización de la pila de servicio (SSU)](https://support.microsoft.com/help/4490628) de Windows y la [actualización de SHA-2](https://support.microsoft.com/help/4474419) en las máquinas que ejecutan Windows 7 con SP1.  SHA-1 no se admite desde septiembre de 2019 y, si la firma de código SHA-2 no está habilitada, la extensión del agente no se instalará ni actualizará según lo previsto. Más información sobre los [requisitos y la actualización de SHA-2](https://aka.ms/SHA-2KB).



#### <a name="linux"></a>Linux

**Sistema operativo** | **Detalles**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6,[7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4564347/), [7.9](https://support.microsoft.com/help/4578241/), [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1, [8.2](https://support.microsoft.com/help/4570609/)
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, [7.8](https://support.microsoft.com/help/4564347/), [7.9](https://support.microsoft.com/help/4578241/), 8.0, 8.1, [8.2](https://support.microsoft.com/en-us/help/4570609)
Servidor Ubuntu 14.04 LTS | [Versiones de kernel admitidas](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Servidor Ubuntu 16.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Los servidores Ubuntu que usan un inicio de sesión y una autenticación mediante contraseña y que usan el paquete cloud-init para configurar máquinas virtuales en la nube pueden tener el inicio de sesión mediante contraseña deshabilitado tras una conmutación por error (en función de la configuración de cloudinit). Para volver a habilitar el inicio de sesión mediante contraseña en la máquina virtual, restablezca la contraseña desde el menú Soporte > Solución de problemas > Configuración (de la máquina virtual conmutada por error en Azure Portal).
Servidor Ubuntu 18.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions-for-azure-virtual-machines) </br> Ubuntu 18.4.03 (Kernel v5.4) se admite desde [9.36](https://support.microsoft.com/help/4578241/)|
Servidor Ubuntu 20.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Debian 7 | Incluye compatibilidad para todas las versiones 7. *x*). [Versiones de kernel admitidas](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | Incluye compatibilidad para todas las versiones 8. *x*). [Versiones de kernel admitidas](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4, SP5 [(versiones de kernel admitidas)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines) </br> (**SP5 está disponible a través de la última revisión de 9.33 de Azure para el escenario de recuperación ante desastres de Azure).
SUSE Linux Enterprise Server 15 | 15 y 15 SP1. [(Versiones de kernel admitidas)](#supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | SP3<br/><br/> No se admite la actualización de máquinas de replicación de SP3 a SP4. Si se ha actualizado una máquina replicada, deberá deshabilitar la replicación y volverla a habilitar después de la actualización.
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4573888/), [8.0](https://support.microsoft.com/help/4573888/)  <br/> Ejecución del kernel compatible de Red Hat o Unbreakable Enterprise Kernel Release 3, 4 y 5 (UEK3, UEK4, UEK5)<br/><br/>8.1<br/>La ejecución en todos los kernels de UEK y RedHat kernel <= 3.10.0-1062.* se admiten en [9.35](https://support.microsoft.com/help/4573888/). La compatibilidad con el resto de los kernels de RedHat está disponible en [9.36](https://support.microsoft.com/help/4578241/).

**Nota: Para admitir los kernels de Linux más recientes en un plazo de 30 días desde su publicación, Azure Site Recovery implementa la revisión de corrección urgente sobre la versión más reciente del agente de movilidad. Esta corrección se implementa entre dos versiones principales. Para actualizar a la versión más reciente del agente de movilidad (incluida la revisión de corrección urgente), siga los pasos que se mencionan en [este artículo](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). Esta revisión está implementada actualmente para los agentes de movilidad que se usan en Azure para el escenario de recuperación ante desastres de Azure.

#### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel de Ubuntu admitidas para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
14.04 LTS | 9.32, [9.33](https://support.microsoft.com/help/4564347/),[9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/)| 3.13.0-24-generic a 3.13.0-170-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-148-generic,<br/>4.15.0-1023-azure a 4.15.0-1045-azure |
|||
16.04 LTS | [9.36](https://support.microsoft.com/help/4578241/)| 4.4.0-21-generic a 4.4.0-187-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-112-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1092-azure |
16.04 LTS | [9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/) | 4.4.0-21-generic a 4.4.0-184-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-106-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1089-azure</br> 4.15.0-107-generic, 4.4.0-185-generic & 4.15.0-1091-azure through 9.35 hot fix patch** |
16.04 LTS | [9.33](https://support.microsoft.com/help/4564347/) | 4.4.0-21-generic a 4.4.0-178-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-99-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1082-azure </br> 4.15.0-101-generic y 4.4.0-179-generic a la revisión de corrección urgente 9.33**|
16.04 LTS | 9.32 | 4.4.0-21-generic a 4.4.0-171-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-74-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1066-azure|
|||
18.04 LTS | [9.36](https://support.microsoft.com/help/4578241/) | 4.15.0-20-generic a 4.15.0-112-generic </br> De 4.18.0-13-generic a 4.18.0-25-generic </br> 5.0.0-15-generic a 5.0.0-58-generic </br> 5.3.0-19-generic a 5.3.0-65-generic </br> 5.4.0-37-generic a 5.4.0-42-generic</br> 4.15.0-1009-azure a 4.15.0-1092-azure </br> De 4.18.0-1006-azure a 4.18.0-1025-azure </br> 5.0.0-1012-azure a 5.0.0-1036-azure </br> 5.3.0-1007-azure a 5.3.0-1032-azure </br> 5.4.0-1020-azure a 5.4.0-1022-azure|
18.04 LTS | [9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/) | 4.15.0-20-generic a 4.15.0-108-generic </br> De 4.18.0-13-generic a 4.18.0-25-generic </br> 5.0.0-15-generic a 5.0.0-52-generic </br> 5.3.0-19-generic a 5.3.0-61-generic </br> 4.15.0-1009-azure a 4.15.0-1089-azure </br> De 4.18.0-1006-azure a 4.18.0-1025-azure </br> 5.0.0-1012-azure a 5.0.0-1036-azure </br> 5.3.0-1007-azure a 5.3.0-1031-azure </br> 4.15.0-109-generic, 5.0.0-53-generic, 5.3.0-62-generic, 4.15.0-1091-azure & 5.3.0-1032-azure through 9.35 hot fix patch**|
18.04 LTS | [9.33](https://support.microsoft.com/help/4564347/) | 4.15.0-20-generic a 4.15.0-99-generic </br> De 4.18.0-13-generic a 4.18.0-25-generic </br> 5.0.0-15-generic a 5.0.0-47-generic </br> 5.3.0-19-generic a 5.3.0-51-generic </br> 4.15.0-1009-azure a 4.15.0-1082-azure </br> De 4.18.0-1006-azure a 4.18.0-1025-azure </br> 5.0.0-1012-azure a 5.0.0-1036-azure </br> 5.3.0-1007-azure a 5.3.0-1020-azure </br> 4.15.0-101-generic, 5.0.0-48-generic, 5.3.0-1022-azure y 5.3.0-53-generic a la revisión de corrección urgente 9.33**|
18.04 LTS | 9.32| 4.15.0-20-generic a 4.15.0-74-generic </br> De 4.18.0-13-generic a 4.18.0-25-generic </br> 5.0.0-15-generic a 5.0.0-37-generic </br> 5.3.0-19-generic a 5.3.0-24-generic </br> De 4.15.0-1009-azure a 4.15.0-1037-azure </br> De 4.18.0-1006-azure a 4.18.0-1025-azure </br> 5.0.0-1012-azure a 5.0.0-1028-azure </br> 5.3.0-1007-azure a 5.3.0-1009-azure|
|||
20.04 LTS |[9.36](https://support.microsoft.com/help/4578241/) | 5.4.0-26-generic a 5.4.0-42 </br> -generic 5.4.0-1010-azure a 5.4.0-1022-azure

**Nota: Para admitir los kernels de Linux más recientes en un plazo de 30 días desde su publicación, Azure Site Recovery implementa la revisión de corrección urgente sobre la versión más reciente del agente de movilidad. Esta corrección se implementa entre dos versiones principales. Para actualizar a la versión más reciente del agente de movilidad (incluida la revisión de corrección urgente), siga los pasos que se mencionan en [este artículo](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). Esta revisión está implementada actualmente para los agentes de movilidad que se usan en Azure para el escenario de recuperación ante desastres de Azure.

#### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel de Debian admitidas para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
Debian 7 | 9.32, [9.33](https://support.microsoft.com/help/4564347/), [9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/) | 3.2.0-4-amd64 a 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | [9.35](https://support.microsoft.com/help/4573888/, ), [9.36](https://support.microsoft.com/help/4578241/) | 3.16.0-4-amd64 a 3.16.0-11-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.11-amd64 |
Debian 8 | 9.32, [9.33](https://support.microsoft.com/help/4564347/), [9.34](https://support.microsoft.com/help/4570609) | 3.16.0-4-amd64 a 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.11-amd64 |

#### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel admitidas de SUSE Linux Enterprise Server 12 para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.36](https://support.microsoft.com/help/4578241/) | Se admiten todos los [kernels de SUSE 12 SP1, SP2, SP3 y SP4](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.4.138-4.7-azure a 4.4.180-4.31-azure,</br>4.12.14-6.3-azure a 4.12.14-6.43-azure </br> 4.12.14-16.7-azure a 4.12.14-16.22-azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/) | Se admiten todos los [kernels de SUSE 12 SP1, SP2, SP3 y SP4](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.4.138-4.7-azure a 4.4.180-4.31-azure,</br>4.12.14-6.3-azure a 4.12.14-6.43-azure </br> 4.12.14-16.7-azure a 4.12.14-16.19-azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9.32, [9.33](https://support.microsoft.com/help/4564347/), [9.35](https://support.microsoft.com/help/4573888/) | Se admiten todos los [kernels de SUSE 12 SP1, SP2, SP3 y SP4](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.4.138-4.7-azure a 4.4.180-4.31-azure,</br>4.12.14-6.3-azure a 4.12.14-6.34-azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | Revisión de corrección urgente 9.33 | Se admiten todos los [kernels de SUSE 12 SP1, SP2, SP3 y SP4](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.4.138-4.7-azure a 4.4.180-4.31-azure,</br>4.12.14-6.3-azure a 4.12.14-6.34-azure </br> 4.12.14-16.7-azure a 4.12.14-16.13-azure  |

#### <a name="supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel admitidas de SUSE Linux Enterprise Server 15 para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
SUSE Linux Enterprise Server 15 y 15 SP1 | [9.36](https://support.microsoft.com/help/4578241/)  | De forma predeterminada, se admiten todos los [kernels incluidos en SUSE 15 y 15](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.12.14-5.5-azure a 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure a 4.12.14-8.38-azure
SUSE Linux Enterprise Server 15 y 15 SP1 | [9.34](https://support.microsoft.com/help/4570609), [9.35](https://support.microsoft.com/help/4573888/)  | De forma predeterminada, se admiten todos los [kernels incluidos en SUSE 15 y 15](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.12.14-5.5-azure a 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure a 4.12.14-8.33-azure 
|SUSE Linux Enterprise Server 15 y 15 SP1 | [9.33](https://support.microsoft.com/help/4564347/) | De forma predeterminada, se admiten todos los [kernels incluidos en SUSE 15 y 15](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.12.14-5.5-azure a 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure a 4.12.14-8.30-azure |
SUSE Linux Enterprise Server 15 y 15 SP1 | 9.32 | De forma predeterminada, se admiten todos los [kernels incluidos en SUSE 15 y 15](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.12.14-5.5-azure a 4.12.14-8.22-azure |


## <a name="replicated-machines---linux-file-systemguest-storage"></a>Máquinas replicadas: Sistema de archivos o almacenamiento de invitado de Linux

* Sistemas de archivos: ext3, ext4, XFS y BTRFS
* Administrador de volúmenes: LVM2

> [!NOTE]
> No se admite software de múltiples rutas. 


## <a name="replicated-machines---compute-settings"></a>Máquinas replicadas: Configuración de proceso

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
Size | Cualquier tamaño de máquina virtual de Azure con al menos 2 núcleos de CPU y 1 GB de RAM | Compruebe los [tamaños de máquina virtual de Azure](../virtual-machines/sizes.md).
Conjuntos de disponibilidad | Compatible | Si habilita la replicación de una máquina virtual de Azure con las opciones predeterminadas, se creará automáticamente un conjunto de disponibilidad en función de la configuración de la región de origen. Puede modificar estos valores.
Zonas de disponibilidad | Compatible |
Ventaja de uso híbrido (HUB) | Compatible | Si la máquina virtual de origen tiene habilitada una licencia HUB, en la conmutación por error de prueba o la máquina virtual de conmutación por error también se usa la licencia HUB.
Conjuntos de escalado de máquinas virtuales | No compatible |
Imágenes de la galería de Azure (publicadas por Microsoft) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes de la Galería de Azure (publicadas por terceros) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes personalizadas (publicadas de terceros) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Máquinas virtuales migradas con Site Recovery | Compatible | Si se migró una máquina virtual de VMware o una física a Azure mediante Site Recovery, deberá desinstalar la versión antigua de Mobility Service que se ejecuta en la máquina y reiniciarla antes de replicarla en otra región de Azure.
Directivas RBAC | No compatible | Las directivas de control de acceso basado en rol (RBAC) de las máquinas virtuales no se replican en la máquina virtual de conmutación por error de la región de destino.
Extensiones | No compatible | Las extensiones no se replican en la máquina virtual de conmutación por error en la región de destino. Deben instalarse manualmente después de la conmutación por error.
Grupos con ubicación por proximidad | Compatible | Las máquinas virtuales que se encuentran dentro de un grupo con ubicación por proximidad se pueden proteger mediante Site Recovery.


## <a name="replicated-machines---disk-actions"></a>Máquinas replicadas: acciones de disco

**Acción** | **Detalles**
-- | ---
Cambiar el tamaño de disco en una máquina virtual replicada | Se admite en la máquina virtual de origen antes de la conmutación por error. No es necesario deshabilitar o volver a habilitar la replicación.<br/><br/> Si cambia la máquina virtual de origen después de la conmutación por error, los cambios no se capturan.<br/><br/> Si cambia el tamaño del disco en la máquina virtual de Azure después de la conmutación por error, Site Recovery no capturará los cambios y la conmutación por recuperación se realizará al tamaño original de la máquina virtual.
Agregar un disco a una máquina virtual replicada | Compatible
Cambios sin conexión en discos protegidos | Para desconectar discos y realizar modificaciones sin conexión en ellos debe desencadenar una resincronización completa.

## <a name="replicated-machines---storage"></a>Máquinas replicadas: almacenamiento

Esta tabla resume la compatibilidad con el disco del sistema operativo, el disco de datos y el disco temporal de Azure VM.

- Es importante respetar los límites y destinos de los discos de máquinas virtuales [Linux](../virtual-machines/linux/disk-scalability-targets.md) y [Windows](../virtual-machines/windows/disk-scalability-targets.md) para evitar cualquier problema de rendimiento.
- Si implementa con la configuración predeterminada, Site Recovery creará automáticamente las cuentas de almacenamiento y los discos necesarios en función de la configuración de origen.
- Si desea personalizar, asegúrese de que sigue las directrices.

**Componente** | **Soporte técnico** | **Detalles**
--- | --- | ---
Tamaño máximo del disco de sistema operativo | 2048 GB | [Más información](../virtual-machines/managed-disks-overview.md) sobre discos de máquina virtual.
Disco temporal | No compatible | El disco temporal se excluye de la replicación siempre.<br/><br/> No almacene los datos persistentes en el disco temporal. [Más información](../virtual-machines/managed-disks-overview.md).
Tamaño máximo del disco de datos | 8192 GB para discos administrados<br></br>4095 GB para discos no administrados|
Tamaño mínimo del disco de datos | Sin restricción de discos no administrados. 2 GB en discos administrados |
Número máximo de discos de datos | Hasta 64, según la compatibilidad con un tamaño específico de máquina virtual de Azure | [Más información](../virtual-machines/sizes.md) sobre tamaños de máquina virtual.
Tasa de cambio de disco de datos | Máximo de 20 MBps por disco para el almacenamiento Premium. Máximo de 2 MBps por disco para el almacenamiento Estándar. | Si la tasa media de cambio de los datos en el disco sobrepasa el máximo continuamente, la replicación no mantendrá el ritmo.<br/><br/>  Sin embargo, si se supera el máximo esporádicamente, la replicación podrá mantenerlo, aunque podría ver puntos de recuperación ligeramente retrasados.
Disco de datos: cuenta de almacenamiento Estándar | Compatible |
Disco de datos: cuenta de almacenamiento Premium | Compatible | Si una máquina virtual tiene discos repartidos entre cuentas de almacenamiento Estándar y Premium, puede seleccionar una cuenta de almacenamiento de destino diferente para cada disco a fin de garantizar que tenga la misma configuración de almacenamiento en la región de destino.
Disco administrado: estándar | Se admite en regiones de Azure en las que se pueda usar Azure Site Recovery. |
Disk administrado: premium | Se admite en regiones de Azure en las que se pueda usar Azure Site Recovery. |
SSD estándar | Compatible |
Redundancia | Se admiten LRS y GRS.<br/><br/> No se admite ZRS.
Almacenamiento esporádico y almacenamiento frecuente | No compatible | Los discos de máquina virtual no admiten el almacenamiento esporádico ni el frecuente
Espacios de almacenamiento | Compatible |
Cifrado en reposo (SSE) | Compatible | SSE es la configuración predeterminada en las cuentas de almacenamiento.
Cifrado en reposo (CMK) | Compatible | Para los discos administrados se admiten tanto claves de software como HSM
Cifrado en reposo doble | Compatible | Más información sobre las regiones admitidas para [Windows](../virtual-machines/windows/disk-encryption.md) y [Linux](../virtual-machines/linux/disk-encryption.md)
Azure Disk Encryption (ADE) para sistemas operativos Windows | Compatible con máquinas virtuales con discos administrados. | No se admiten las máquinas virtuales que usan discos no administrados. <br/><br/> No se admiten las claves protegidas con HSM. <br/><br/> No se admite el cifrado de volúmenes individuales en un solo disco. |
Azure Disk Encryption (ADE) para sistemas operativos Linux | Compatible con máquinas virtuales con discos administrados. | No se admiten las máquinas virtuales que usan discos no administrados. <br/><br/> No se admiten las claves protegidas con HSM. <br/><br/> No se admite el cifrado de volúmenes individuales en un solo disco. <br><br> Problema conocido en la habilitación de la replicación. [Más información.](./azure-to-azure-troubleshoot-errors.md#enable-protection-failed-as-the-installer-is-unable-to-find-the-root-disk-error-code-151137) |
Rotación de claves SAS | No compatible | Si se gira la clave SAS de las cuentas de almacenamiento, el cliente debe deshabilitar y volver a habilitar la replicación. |
Adición en caliente    | Compatible | La replicación de un disco de datos que se agrega a una máquina virtual de Azure replicada se puede habilitar en máquinas virtuales que usan discos administrados. <br/><br/> Solo se puede agregar un disco a una máquina virtual de Azure a la vez. No se admite la adición paralela de varios discos. |
Eliminación de disco en caliente    | No compatible | Si quita un disco de datos en la máquina virtual, deberá deshabilitar la replicación y habilitarla de nuevo en la máquina virtual.
Excluir el disco | Compatible. Debe usar [PowerShell](azure-to-azure-exclude-disks.md) para configurarlo. |    Los discos temporales se excluyen de forma predeterminada.
Espacios de almacenamiento directo  | Compatible con puntos de recuperación coherentes con los bloqueos. Los puntos de recuperación coherentes de la aplicación no son compatibles. |
Servidor de archivos de escalado horizontal  | Compatible con puntos de recuperación coherentes con los bloqueos. Los puntos de recuperación coherentes de la aplicación no son compatibles. |
DRBD | No se admiten los discos que forman parte de una instalación de DRBD. |
LRS | Compatible |
GRS | Compatible |
RA-GRS | Compatible |
ZRS | No compatible |
Almacenamiento en frío y en caliente | No compatible | Los discos de máquina virtual no admiten el almacenamiento temporal y permanente.
Firewalls de Azure Storage para redes virtuales  | Compatible | Si restringe el acceso de red virtual a las cuentas de almacenamiento, habilite [Permitir servicios de Microsoft de confianza](../storage/common/storage-network-security.md#exceptions).
Cuentas de almacenamiento de uso general V2 (capas de acceso frecuente y esporádico) | Compatible | Los costos de transacción aumentan considerablemente en comparación con las cuentas de almacenamiento de uso general V1
Generación 2 (Arranque UEFI) | Compatible
Discos NVMe | No compatible
Discos compartidos de Azure | No compatible
Opción de transferencia segura | Compatible

>[!IMPORTANT]
> Para evitar cualquier problema de rendimiento, asegúrese de respetar los objetivos de escalabilidad y rendimiento del disco de la máquina virtual de máquinas virtuales [Linux](../virtual-machines/linux/disk-scalability-targets.md) o [Windows](../virtual-machines/windows/disk-scalability-targets.md). Si usa la configuración predeterminada, Site Recovery crea las cuentas de almacenamiento y los discos necesarios en función de la configuración de origen. Si personaliza y selecciona su propia configuración, respete los objetivos de escalabilidad y rendimiento del disco de las máquinas virtuales de origen.

## <a name="limits-and-data-change-rates"></a>Límites y tasas de cambio de datos

En la tabla siguiente se resumen los límites de Site Recovery.

- Estos límites se basan en nuestras pruebas, pero evidentemente no pueden cubrir todas las combinaciones de E/S posibles de la aplicación.
- Los resultados reales pueden variar en función de la combinación de E/S de la aplicación.
- Hay dos límites que deben tenerse en cuenta: renovación de datos por disco y por máquina virtual.
- El límite actual para la actividad de datos por máquina virtual es de 54 MB/s, independientemente del tamaño.

**Destino de almacenamiento** | **Promedio de E/S de disco de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos de disco de origen total por día**
---|---|---|---
Standard Storage | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |    336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más |20 MB/s | 1684 GB por disco

## <a name="replicated-machines---networking"></a>Máquinas replicadas: redes
**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
NIC | Número máximo admitido para un tamaño específico de máquina virtual de Azure | Las NIC se crean cuando se crea la máquina virtual durante la conmutación por error.<br/><br/> El número de tarjetas NIC en la máquina virtual de conmutación por error viene determinado por el número de tarjetas NIC que haya en la máquina virtual de origen en el momento de habilitar la replicación. Si agrega o quita una tarjeta NIC después de habilitar la replicación, esto no influirá en el número de NIC de la máquina virtual replicada después de la conmutación por error. <br/><br/> No existe garantía de que el orden de lo adaptadores de red después de la conmutación sea el mismo que el orden original. <br/><br/> Puede cambiar el nombre de los adaptadores de red en la región de destino en función de las convenciones de nomenclatura de su organización. El cambio de nombre de adaptador de red es compatible con PowerShell.
Equilibrador de carga de Internet | Compatible | Asocie el equilibrador de carga configurado previamente con un script de Azure Automation de un plan de recuperación.
Equilibrador de carga interno | Compatible | Asocie el equilibrador de carga configurado previamente con un script de Azure Automation de un plan de recuperación.
Dirección IP pública | Compatible | Asocie una dirección IP pública existente a la NIC. O bien, cree una dirección IP pública y asóciela con la NIC mediante un script de Azure Automation de un plan de recuperación.
Grupo de seguridad de red en una tarjeta NIC | Compatible | Asocie el grupo de seguridad de red a la NIC con un script de Azure Automation de un plan de recuperación.
Grupo de seguridad de red en la subred | Compatible | Asocie el grupo de seguridad de red a la subred con un script de Azure Automation de un plan de recuperación.
Dirección IP (estática) reservada | Compatible | Si la NIC de la máquina virtual de origen tiene una dirección IP estática y la subred de destino tiene la misma dirección IP disponible, esta se asigna a la máquina virtual de conmutación por error.<br/><br/> Si la subred de destino no tiene la misma dirección IP disponible, una de las direcciones IP disponibles de la subred se reserva para esta máquina virtual.<br/><br/> También puede especificar una dirección IP fija y una subred en **Elementos replicados** > **Configuración** > **Proceso y red** > **Interfaces de red**.
Dirección IP dinámica | Compatible | Si la NIC de origen tiene una dirección IP dinámica, la NIC de la máquina virtual de conmutación por error será también dinámica de forma predeterminada.<br/><br/> Puede modificar esta a una dirección IP fija si es necesario.
Varias direcciones IP | No compatible | Cuando se conmuta por error una máquina virtual que tiene una NIC con varias direcciones IP, solamente se mantiene la dirección IP principal de la NIC en la región de origen. Para asignar varias direcciones IP, puede agregar máquinas virtuales a un [plan de recuperación](recovery-plan-overview.md) y adjuntar un script para asignar direcciones IP adicionales al plan, o bien puede realizar el cambio manualmente o con un script después de la conmutación por error.
Traffic Manager     | Compatible | Puede configurar previamente Traffic Manager de forma que el tráfico se dirija regularmente al punto de conexión de la región de origen y al punto de conexión de la región de destino en caso de conmutación por error.
Azure DNS | Compatible |
DNS personalizado    | Compatible |
Proxy no autenticado | Compatible | [Más información](./azure-to-azure-about-networking.md)
Proxy autenticado | No compatible | Si la máquina virtual usa un proxy autenticado para la conectividad saliente, no se puede replicar mediante Azure Site Recovery.
Conexión VPN de sitio a sitio en el entorno local<br/><br/>(con o sin ExpressRoute)| Compatible | Asegúrese de que los UDR y NSG estén configurados de manera que el tráfico de Site Recovery no se dirija al entorno local. [Más información](./azure-to-azure-about-networking.md)
Conexión de red virtual a red virtual    | Compatible | [Más información](./azure-to-azure-about-networking.md)
Puntos de conexión de servicio de red virtual | Compatible | Si está restringiendo el acceso de red virtual a las cuentas de almacenamiento, asegúrese de que los servicios de Microsoft de confianza pueden acceder a la cuenta de almacenamiento.
Redes aceleradas | Compatible | Se deben habilitar las redes aceleradas en una máquina virtual de origen. [Más información](azure-vm-disaster-recovery-with-accelerated-networking.md).
Dispositivo de Palo Alto Networks | No compatible | Con los dispositivos de terceros, a menudo hay restricciones impuestas por el proveedor dentro de la máquina virtual. Azure Site Recovery necesita que el agente, las extensiones y la conectividad saliente estén disponibles. Pero el dispositivo no permite que se configure ninguna actividad saliente dentro de la máquina virtual.
IPv6  | No compatible | Tampoco se admiten configuraciones mixtas que incluyan IPv4 e IPv6. Libere la subred del intervalo IPv6 antes de cualquier operación de Site Recovery.
Acceso de vínculo privado al servicio Site Recovery | Compatible | [Más información](azure-to-azure-how-to-enable-replication-private-endpoints.md)



## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información en las [instrucciones sobre redes](./azure-to-azure-about-networking.md) para replicar máquinas virtuales de Azure.
- Implemente la recuperación ante desastres mediante [la replicación de máquinas virtuales de Azure](./azure-to-azure-quickstart.md).
