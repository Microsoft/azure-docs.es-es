---
title: Preparación de Azure para la recuperación ante desastres en el entorno local con Azure Site Recovery
description: Obtenga información sobre cómo preparar Azure para la recuperación ante desastres de máquinas locales mediante Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/09/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 0d697f2100360732666c9f5ea8050800dad154b4
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653824"
---
# <a name="prepare-azure-for-on-premises-disaster-recovery-to-azure"></a>Preparación de Azure para la recuperación ante desastres en el entorno local en Azure

En este artículo se describe cómo preparar los recursos y componentes de Azure para que pueda configurar la recuperación ante desastres de máquinas virtuales de VMware, máquinas virtuales de Hyper-V o servidores físicos Windows/Linux en Azure, mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

Este artículo es el primer tutorial de una serie que muestra cómo configurar la recuperación ante desastres para máquinas virtuales locales. 


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar que la cuenta de Azure tiene permisos de replicación.
> * Cree un almacén de Recovery Services. Un almacén contiene metadatos e información de configuración de las máquinas virtuales y otros componentes de replicación.
> * Configurar una red virtual de Azure Virtual Network. Cuando se crean máquinas virtuales de Azure después de la conmutación por error, se unen a esta red.

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Usan opciones predeterminadas siempre que es posible y no muestran todos los valores y las rutas de acceso posibles. Para obtener instrucciones detalladas, consulte el artículo de la sección de procedimientos de la tabla de contenido de Site Recovery.

## <a name="before-you-start"></a>Antes de comenzar

- Revise la arquitectura de recuperación ante desastres de [VMware](vmware-azure-architecture.md), [Hyper-V](hyper-v-azure-architecture.md) y el [servidor físico](physical-azure-architecture.md).
- Lea las preguntas habituales de [VMware](vmware-azure-common-questions.md) e [Hyper-V](hyper-v-azure-common-questions.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar. Luego, inicie sesión en el [Portal de Azure](https://portal.azure.com).


## <a name="verify-account-permissions"></a>Comprobar los permisos de la cuenta

Si acaba de crear una cuenta de Azure gratis, es el administrador de la suscripción y tiene los permisos que necesita. Si no es el administrador de la suscripción, solicite al administrador que le asigne los permisos que necesita. Para habilitar la replicación de una nueva máquina virtual, debe tener permiso para:

- Crear una máquina virtual en el grupo de recursos seleccionado.
- Crear una máquina virtual en la red virtual seleccionada.
- Escribir en una cuenta de Azure Storage.
- Escribir en un disco administrado de Azure.

Para completar estas tareas su cuenta debe tener asignado el rol integrado de colaborador de la máquina virtual. Además, para administrar las operaciones de Site Recovery en un almacén, su cuenta debe tener asignado el rol integrado de colaborador de Site Recovery.


## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

1. En Azure Portal, seleccione **Crear un recurso** y, en Marketplace, busque **Recuperación**.
2. Seleccione **Backup and Site Recovery** en los resultados de la búsqueda y, una vez en esta página, haga clic en **Crear**. 
3. En la página **Crear almacén de Recovery Services**, seleccione la **Suscripción**. Vamos a usar la **suscripción de Contoso**.
4. En **Grupo de recursos**, seleccione un grupo de recursos existente o cree uno nuevo. En este tutorial se va a usar **contosoRG**.
5. En **Nombre del almacén**, escriba un nombre descriptivo para identificar el almacén. Para este conjunto de tutoriales se usa **ContosoVMVault**.
6. En **Región**, seleccione la región en la que debería estar el almacén. se va a usar **Oeste de Europa**.
7. Seleccione **Revisar + crear**.

   ![Captura de pantalla de la página Create Recovery Services vault (Crear almacén de Recovery Services).](./media/tutorial-prepare-azure/new-vault-settings.png)

   El nuevo almacén aparecerá ahora en **Panel** > **Todos los recursos** y en la página principal de **Almacenes de Recovery Services**.

## <a name="set-up-an-azure-network"></a>Configurar una red de Azure

Las máquinas locales se replican en los discos administrados de Azure. Cuando se produce la conmutación por error, se crean máquinas virtuales de Azure a partir de estos discos administrados y se unen a la red de Azure que especifique en este procedimiento.

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Redes** > **Red virtual**.
2. Deje **Resource Manager** seleccionado como modelo de implementación.
3. En **Nombre**, escriba un nombre de red. El nombre debe ser único dentro del grupo de recursos de Azure. Se va a usar **ContosoASRnet** en este tutorial.
4. En **Espacio de direcciones**, escriba el intervalo de direcciones de la red virtual en notación CIDR. Usamos **10.1.0.0/24**.
5. En **Suscripción**, seleccione la suscripción en la que se creará la red.
6. Especifique el **grupo de recursos** en el que se creará la red. Se va a usar el grupo de recursos existente **contosoRG**.
7. En **Ubicación**, seleccione la región que en que se creó el almacén de Recovery Services. En este tutorial es **Oeste de Europa**. La red debe estar en la misma región que el almacén.
8. En **Intervalo de direcciones**, escriba el intervalo de la red. Usamos **10.1.0.0/24**, pero no usamos ninguna subred.
9. Se van a dejar las opciones predeterminadas de protección básica contra DDoS, con ningún punto de conexión de servicio ni firewall en la red.
9. Seleccione **Crear**.

   ![Captura de pantalla de las opciones de Crear red virtual.](media/tutorial-prepare-azure/create-network.png)

La red virtual tarda unos segundos en crearse. Una vez creada, la verá en el panel de Azure Portal.




## <a name="next-steps"></a>Pasos siguientes

- Para la recuperación ante desastres de VMware, [prepare la infraestructura de VMware local](./vmware-azure-tutorial-prepare-on-premises.md).
- Para la recuperación ante desastres de Hyper-V, [prepare los servidores de Hiper-V locales](hyper-v-prepare-on-premises-tutorial.md).
- Para la recuperación ante desastres de servidores físicos, [configure el servidor de configuración y el entorno del origen](physical-azure-disaster-recovery.md)
- [Más información](../virtual-network/virtual-networks-overview.md) sobre las redes de Azure.
- [Más información](../virtual-machines/managed-disks-overview.md) sobre discos administrados.
