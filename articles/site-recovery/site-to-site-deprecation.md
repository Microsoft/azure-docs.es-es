---
title: Desuso de la recuperación ante desastres entre sitios administrados por el cliente (con VMM) mediante Azure Site Recovery | Microsoft Docs
description: Detalles sobre el próximo desuso de la recuperación ante desastres entre los sitios propiedad del cliente con Hyper-V y entre sitios administrados por SCVMM en Azure y opciones alternativas
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 02/25/2020
ms.author: sharrai
ms.openlocfilehash: 9ffe7a3158b1de6828350947dcf81ef41d08708d
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421848"
---
# <a name="deprecation-of-disaster-recovery-between-customer-managed-sites-with-vmm-using-azure-site-recovery"></a>Desuso de la recuperación ante desastres entre sitios administrados por el cliente (con VMM) mediante Azure Site Recovery

En este artículo se describe el plan para el próximo desuso, las implicaciones correspondientes y las opciones alternativas disponibles para los clientes en los escenarios siguientes:

Recuperación ante desastres entre los sitios propiedad del cliente administrados por System Center Virtual Machine Manager (SCVMM) mediante Site Recovery

> [!IMPORTANT]
> Se recomienda que los clientes tomen las medidas de corrección cuanto antes para evitar cualquier interrupción del entorno. 

## <a name="what-changes-should-you-expect"></a>¿Qué cambios debe esperar?

- A partir de marzo de 2020 recibirá notificaciones de Azure Portal y comunicaciones de correo electrónico en que se le informará del próximo desuso de la replicación de sitio a sitio de las máquinas virtuales de Hyper-V. Está planeado que las máquinas entren en desuso en marzo de 2023.

- Si tiene una configuración, no resultará afectada.

- Una vez que los escenarios estén en desuso, a menos que el cliente siga los enfoques alternativos, es posible que se interrumpan las replicaciones existentes. Los clientes no podrán ver, administrar ni realizar ninguna operación relacionada con la recuperación ante desastres a través de la experiencia de Azure Site Recovery en Azure Portal.
 
## <a name="alternatives"></a>Alternativas 

A continuación, se muestran las alternativas entre las que puede elegir el cliente para garantizar que su estrategia de recuperación ante desastres no se vea afectada una vez que el escenario esté en desuso. 

- Opción 1 (recomendada): Elija [empezar a usar Azure como destino de recuperación ante desastres](hyper-v-vmm-azure-tutorial.md).


- Opción 2: Elija continuar con la replicación de sitio a sitio con la [solución de réplica de Hyper-V](/windows-server/virtualization/hyper-v/manage/set-up-hyper-v-replica) subyacente, aunque no podrá administrar las configuraciones de recuperación ante desastres con Azure Site Recovery en Azure Portal. 


## <a name="remediation-steps"></a>Pasos de corrección

Si elige usar la opción 1, ejecute estos pasos:

1. [Deshabilite la protección de todas las máquinas virtuales asociadas con los VMM](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario). Use la opción **Deshabilitar la replicación y quitar** o ejecute los scripts mencionados para garantizar que la configuración de replicación en el entorno local se elimina. 

2. [Anule el registro de todos los servidores VMM](site-recovery-manage-registration-and-protection.md#unregister-a-vmm-server) de la configuración de replicación de sitio a sitio.

3. [Prepare los recursos de Azure](tutorial-prepare-azure-for-hyperv.md) para la habilitación de la replicación de las VM.
4. [Preparación de los servidores Hyper-V locales](hyper-v-prepare-on-premises-tutorial.md)
5. [Configuración de la replicación de las máquinas virtuales en la nube de VMM](hyper-v-vmm-azure-tutorial.md)
6. Opcional, pero recomendable: [Ejecución de exploración de la recuperación ante desastres](tutorial-dr-drill-azure.md)

Si elige usar la opción 2 para usar una réplica Hyper-V, ejecute estos pasos:

1. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
2. En **Deshabilitar replicación**, seleccione **Quitar**.

    Esto quita el elemento replicado de Azure Site Recovery (se detiene la facturación). La configuración de replicación de la máquina virtual local **no** se va a limpiar. 

## <a name="next-steps"></a>Pasos siguientes
Planee el desuso y elija una opción alternativa que sea más adecuada para su infraestructura y negocios. Si tiene alguna duda al respecto, póngase en contacto con Soporte técnico de Microsoft

