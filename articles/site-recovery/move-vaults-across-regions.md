---
title: Traslado de un almacén de Azure Site Recovery a otra región
description: Describe cómo trasladar un almacén de Recovery Services (Azure Site Recovery) a otra región de Azure
services: site-recovery
author: Sharmistha-Rai
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/31/2019
ms.author: sharrai
ms.custom: MVC
ms.openlocfilehash: a8cb5ec782b5932c13e321b2ba2d6513597fef52
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422647"
---
# <a name="move-a-recovery-services-vault-and-azure-site-recovery-configuration-to-another-azure-region"></a>Traslado del almacén de Recovery Services y la configuración de Azure Site Recovery a otra región de Azure

Hay varios escenarios en los que puede desear mover los recursos de Azure existentes de una región a otra. Por ejemplo, por capacidad de administración, por motivos de gobernanza o quizá a causa de las fusiones y adquisiciones de la empresa. Uno de los recursos relacionados que puede querer trasladar al mover las máquinas virtuales de Azure es la configuración de recuperación ante desastres. 

No existe el método perfecto para trasladar una configuración de recuperación ante desastres existente de una región a otra. Esto se debe a que la región de destino se configura en función de la región de la máquina virtual de origen. Si decide cambiar la región de origen, no podrá volver a usar la configuración anterior de la región de destino y deberá restablecerla. En este artículo se define el proceso paso a paso para volver a establecer la configuración de recuperación ante desastres y moverla a otra región.

En este documento se hará lo siguiente:

> [!div class="checklist"]
> * Comprobación de los requisitos previos para el traslado
> * Identificación de los recursos usados en Azure Site Recovery
> * Deshabilitación de la replicación
> * Eliminación de los recursos
> * Configuración de Site Recovery en función de la nueva región de origen para las máquinas virtuales

> [!IMPORTANT]
> Actualmente no existe el método perfecto para mover un almacén de Recovery Services y la configuración de recuperación ante desastres tal como está a una región diferente. Este artículo le guía por el proceso de deshabilitación de la replicación y su configuración en la nueva región.

## <a name="prerequisites"></a>Prerequisites

- Asegúrese de quitar y eliminar la configuración de recuperación ante desastres antes de intentar trasladar las máquinas virtuales de Azure a otra región. 

  > [!NOTE]
  > Si la nueva región de destino para la máquina virtual de Azure es la misma que la región de destino de recuperación ante desastres, puede usar la configuración de replicación existente y moverla. Siga los pasos de [Traslado de máquinas virtuales de Azure a otra región](azure-to-azure-tutorial-migrate.md).

- Asegúrese de que toma la decisión con conocimiento de causa y de que las partes interesadas están informadas. La máquina virtual no estará protegida contra desastres hasta que se complete su traslado.

## <a name="identify-the-resources-that-were-used-by-azure-site-recovery"></a>Identificar los recursos que usaba Azure Site Recovery
Se recomienda llevar a cabo este paso antes de continuar con el siguiente. Es más fácil identificar los recursos pertinentes mientras se replican las máquinas virtuales.

Para cada máquina virtual de Azure que se esté replicando, vaya a **Elementos protegidos** > **Elementos replicados** > **Propiedades** e identifique los siguientes recursos:

- Grupo de recursos de destino
- Cuenta de almacenamiento en caché
- Cuenta de almacenamiento de destino (en el caso de una máquina virtual de Azure basada en disco no administrado) 
- Red de destino


## <a name="disable-the-existing-disaster-recovery-configuration"></a>Deshabilitación de la configuración de recuperación ante desastres existente

1. Vaya al almacén de Recovery Services.
2. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina y seleccione **Deshabilitar replicación**.
3. Repita este paso para todas las máquinas virtuales que desee trasladar.

> [!NOTE]
> Mobility Service no se desinstalará de los servidores protegidos. Debe desinstalarlo manualmente. Si va a proteger el servidor de nuevo, puede omitir la desinstalación del servicio de movilidad.

## <a name="delete-the-resources"></a>Eliminación de los recursos

1. Vaya al almacén de Recovery Services.
2. Seleccione **Eliminar**.
3. Elimine todos los demás recursos que [identificó anteriormente](#identify-the-resources-that-were-used-by-azure-site-recovery).
 
## <a name="move-azure-vms-to-the-new-target-region"></a>Traslado de máquinas virtuales de Azure a la nueva región de destino

Siga los pasos que se indican en los artículos siguientes, en función de sus necesidades para trasladar máquinas virtuales de Azure a la región de destino:

- [Traslado de máquinas virtuales de Azure a otra región](azure-to-azure-tutorial-migrate.md)
- [Traslado de máquinas virtuales de Azure a zonas de disponibilidad](move-azure-VMs-AVset-Azone.md)

## <a name="set-up-site-recovery-based-on-the-new-source-region-for-the-vms"></a>Configuración de Site Recovery en función de la nueva región de origen para las máquinas virtuales

Configure la recuperación ante desastres para las máquinas virtuales de Azure que se movieron a la nueva región siguiendo los pasos descritos en [Configuración de la recuperación ante desastres de máquinas virtuales de Azure](azure-to-azure-tutorial-enable-replication.md).
