---
title: Desuso de característica de cifrado de datos de Azure Site Recovery | Microsoft Docs
description: Detalles sobre la característica de cifrado de datos de Azure Site Recovery
services: site-recovery
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/15/2019
ms.author: sharrai
ms.openlocfilehash: 5860928d71c0e7431190908d5df5d7496e8ffb17
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426323"
---
# <a name="deprecation-of-site-recovery-data-encryption-feature"></a>Desuso de la característica de cifrado de datos de Site Recovery

En este documento se describen los detalles de desuso y la acción correctiva que debe realizar si usa la característica de cifrado de datos de Site Recovery al configurar la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure. 

## <a name="deprecation-information"></a>Información sobre el desuso


El cifrado de datos de Site Recovery estaba disponible para los clientes que protegen máquinas virtuales de Hyper-V para asegurarse de que los datos replicados estaban protegidos contra amenazas de seguridad. Esta característica caerá en desuso el **30 de abril de 2022**. Se está reemplazando por la característica más avanzada [Cifrado en reposo](https://azure.microsoft.com/blog/azure-site-recovery-encryption-at-rest/), que usa [Storage Service Encryption](../storage/common/storage-service-encryption.md) (SSE). Con SSE, los datos se cifran antes de guardarse en el almacenamiento y se descifran en la recuperación y, tras la conmutación por error en Azure, las máquinas virtuales se ejecutarán desde las cuentas de almacenamiento cifradas, lo que permite un mejor objetivo de tiempo de recuperación (RTO).

Tenga en cuenta que si es un cliente que usa esta característica, ya habrá recibido información con los detalles de desuso y los pasos de corrección. 


## <a name="what-are-the-implications"></a>¿Qué implica este cambio?

Después del **30 de abril de 2022**, cualquier VM que siga usando la característica de cifrado retirada no podrá realizar la conmutación por error. 

## <a name="required-action"></a>Acción necesaria
Para continuar con las operaciones de conmutación por error correctas y las replicaciones, siga los pasos que se mencionan a continuación:

Siga estos pasos para cada máquina virtual: 
1.  [Deshabilite la replicación](./site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario).
2.  [Cree una directiva de replicación nueva](./hyper-v-azure-tutorial.md#set-up-a-replication-policy).
3.  [Habilite la replicación](./hyper-v-vmm-azure-tutorial.md#enable-replication) y seleccione una cuenta de almacenamiento con SSE habilitado.

Después de completar la replicación inicial en las cuentas de almacenamiento con SSE habilitado, las máquinas virtuales utilizarán el cifrado en reposo con Azure Site Recovery.


## <a name="next-steps"></a>Pasos siguientes
Planee la realización de los pasos de corrección y ejecútelos lo más pronto posible. En caso de que tenga consultas relacionadas, póngase en contacto con Soporte técnico de Microsoft. Para más información sobre el escenario de Hyper-V a Azure, visite [este sitio](hyper-v-vmm-architecture.md).
