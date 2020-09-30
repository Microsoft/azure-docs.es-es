---
title: Configuración de la recuperación ante desastres de Hyper-V (con VMM) mediante Azure Site Recovery
description: Obtenga información sobre cómo configurar la recuperación ante desastres de máquinas virtuales de Hyper-V locales en nubes de VMM de System Center en Azure mediante Site Recovery.
ms.topic: tutorial
ms.date: 03/19/2020
ms.custom: MVC
ms.openlocfilehash: c806f968bc6530879f64ddbf6fd4c7d45aa7a8d3
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442827"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-in-vmm-clouds-to-azure"></a>Configuración de la recuperación ante desastres de las máquinas virtuales de Hyper-V locales en nubes de VMM en Azure

En este tutorial se describe cómo habilitar la replicación de máquinas virtuales de Hyper-V locales administradas por System Center Virtual Machine Manager (VMM) en Azure, mediante [Azure Site Recovery](site-recovery-overview.md). Si no usa VMM, [siga este tutorial](hyper-v-azure-tutorial.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Seleccionar el origen y destino de la replicación.
> * Configurar el entorno de replicación de origen, incluidos los componentes de Site Recovery locales y el entorno de replicación de destino.
> * Configure la asignación de red entre las redes de máquina virtual de VMM y las redes virtuales de Azure.
> * Crear una directiva de replicación.
> * Habilitar la replicación para una máquina virtual.

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Usan opciones predeterminadas siempre que es posible y no muestran todos los valores y las rutas de acceso posibles. Para instrucciones detalladas, revise los artículos de la sección de **procedimientos** de la [documentación de Site Recovery](./index.yml).

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial se supone que ya ha completado los siguientes tutoriales:

1. [Preparación de Azure](tutorial-prepare-azure.md)
1. [Preparación de los servidores Hyper-V locales](hyper-v-prepare-on-premises-tutorial.md)

## <a name="select-a-replication-goal"></a>Selección de un objetivo de replicación

1. En Azure Portal, vaya a **Almacenes de Recovery Services** y seleccione el almacén **ContosoVMVault** que se creó en el tutorial [Preparación de Azure](tutorial-prepare-azure.md#create-a-recovery-services-vault).
1. En **Introducción**, seleccione **Site Recovery** > **Preparar infraestructura** y configure los siguientes valores:
    1. **Objetivo de protección** >  **¿Dónde están ubicadas las máquinas?** , seleccione **Local**.
    1. **¿A dónde quiere replicar las máquinas?** , seleccione **A Azure**.
    1. **¿Las máquinas están virtualizadas?** (), seleccione **Sí, con Hyper-V**.
    1. **¿Usa System Center VMM para administrar sus hosts de Hyper-V?** , seleccione **Sí**.
1. Seleccione **Aceptar**.

   ![Objetivo de replicación](./media/hyper-v-vmm-azure-tutorial/replication-goal.png)

## <a name="confirm-deployment-planning"></a>Confirmación del planeamiento de la implementación

1. En **Planeamiento de implementación**, si planea una implementación de gran tamaño, descargue Deployment Planner para Hyper-V desde el vínculo de la página. [Obtenga más información](hyper-v-deployment-planner-overview.md) acerca del planeamiento de implementaciones de Hyper-V.
1. En este tutorial, no se necesita Deployment Planner. En **¿Completó el planeamiento de implementación?** , seleccione **Lo haré más tarde** y, luego, seleccione **Aceptar**.

## <a name="set-up-the-source-environment"></a>Configuración del entorno de origen

Al configurar el entorno de origen, debe instalar el proveedor de Azure Site Recovery en el servidor VMM y registrar el servidor en el almacén. Debe instalar el agente de Azure Recovery Services en cada host de Hyper-V.

1. **Preparar infraestructura**. Seleccione **Origen**.
1. **Preparar origen**. Seleccione **+VMM** para agregar un servidor VMM. En **Agregar servidor**, compruebe que aparezca **System Center VMM server** (Servidor VMM de System Center) en **Tipo de servidor**.
1. Descargue el programa de instalación del proveedor de Microsoft Azure Site Recovery.
1. Descargue la clave de registro del almacén. Necesita esta clave al ejecutar el programa de instalación del proveedor. La clave será válida durante cinco días a partir del momento en que se genera.
1. Descargue el programa de instalación del agente de Microsoft Azure Recovery Services.

   ![Descarga del proveedor, de la clave de registro y del agente](./media/hyper-v-vmm-azure-tutorial/download-vmm.png)

### <a name="install-the-provider-on-the-vmm-server"></a>Instalación del proveedor en el servidor VMM

1. En la opción **Microsoft Update** del Asistente para la instalación del proveedor de Azure Site Recovery. Opte por usar Microsoft Update para buscar actualizaciones del proveedor.
1. **Instalación**. Acepte la ubicación de instalación predeterminada del proveedor y seleccione **Instalar**.
1. Después de la instalación, en el Asistente para registro de Microsoft Azure Site Recovery, seleccione **Configuración del almacén**, **Examinar** y, en **Archivo de clave**, seleccione el archivo de clave del almacén que descargó.
1. Especifique la suscripción de Azure Site Recovery y el nombre del almacén (**ContosoVMVault**). Especifique un nombre descriptivo para identificar el servidor VMM en el almacén.
1. **Configuración de proxy**. Seleccione **Conectar directamente con Azure Site Recovery sin un servidor proxy**.
1. Acepte la ubicación predeterminada del certificado que se usa para cifrar los datos. Los datos cifrados se descifrarán cuando se realice la conmutación por error.
1. **Sincronice los metadatos de la nube**. Seleccione **Sincronizar los metadatos de la nube con el portal Site Recovery**. Esta acción solo se debe ejecutar una vez en cada servidor. Después, seleccione **Registrar**.
1. Después de registrar el servidor en el almacén, haga clic en **Finalizar**.

Al finalizar el registro, los metadatos del servidor se recuperan mediante Azure Site Recovery y el servidor VMM se muestra en **Site Recovery Infrastructure** (Infraestructura de Site Recovery).

### <a name="install-the-recovery-services-agent-on-hyper-v-hosts"></a>Instalación del agente de Recovery Services en hosts de Hyper-V

Instale el agente en cada host de Hyper-V que contiene las máquinas virtuales que desea replicar.

En el Asistente para la instalación del agente de Microsoft Azure Recovery Services, configure estos valores:

1. **Comprobación de requisitos previos**. Seleccione **Next** (Siguiente). Todos los requisitos previos que falten se instalarán automáticamente.
1. **Configuración de la instalación**. Acepte la ubicación de instalación. Luego, seleccione **Instalar**.

    >[!NOTE]
    >En el caso de Azure Site Recovery, no se requiere la **ubicación en caché**.

1. **Instalación**. Cuando la instalación finalice, seleccione **Cerrar** para finalizar el asistente.

   ![Instalar agente](./media/hyper-v-vmm-azure-tutorial/mars-install.png)

## <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

1. Seleccione **Preparar infraestructura** > **Destino**.
1. Seleccione la suscripción y el grupo de recursos (**ContosoRG**)donde se crearán las máquinas virtuales de Azure después de la conmutación por error.
1. Seleccione el modelo de implementación de **Resource Manager**.

Site Recovery comprueba que tiene hay una o varias redes y cuentas de almacenamiento de Azure.

## <a name="configure-network-mapping"></a>Configurar asignación de red

1. En **Infraestructura de Site Recovery** > **Asignaciones de red** > **Asignación de red**. Seleccione el icono **+Asignación de red**.
1. **Agregar asignación de red**. Seleccione el servidor de **System Center VMM de origen**. En **Destino** seleccione Azure.
1. Compruebe la suscripción y el modelo de implementación después de la conmutación por error.
1. **Red de origen**. Seleccione la red de máquinas virtuales locales de origen.
1. **Red de destino**. Seleccione la red de Azure en la que se ubicarán las máquinas virtuales de Azure de réplica cuando creen después de la conmutación por error. Después, seleccione **Aceptar**.

   ![Asignación de red](./media/hyper-v-vmm-azure-tutorial/network-mapping-vmm.png)

## <a name="set-up-a-replication-policy"></a>Configurar una directiva de replicación

1. Seleccione **Preparar infraestructura** > **Configuración de la replicación** >  **+Crear y asociar**.
1. En **Crear y asociar directiva**, especifique un nombre de directiva. Usamos **ContosoReplicationPolicy**.
1. Acepte la configuración predeterminada y seleccione **Aceptar**:
   - **Frecuencia de copia** indica que, después de la replicación inicial, los datos diferenciales se replicarán cada cinco minutos.
   - **Retención de punto de recuperación** indica que cada punto de recuperación se conservará durante dos horas.
   - **Frecuencia de instantánea coherente con la aplicación** indica que cada hora se crearán puntos de recuperación que contengan instantáneas coherentes con la aplicación.
   - **Hora de inicio de la replicación inicial** indica que la replicación inicial comenzará de inmediato.
   - **Cifrar datos almacenados en Azure** está establecido en la configuración predeterminada **Desactivado** e indica que los datos en reposo de Azure no están cifrados.
1. Después de que se cree la directiva, seleccione **Aceptar**. Cuando se crea una nueva directiva, se asocia automáticamente a la nube de VMM.

## <a name="enable-replication"></a>Habilitar replicación

1. **Replicar la aplicación**. Seleccione **Origen**.
1. **Origen**. Seleccione la nube de VMM. Después, seleccione **Aceptar**.
1. **Destino**. Compruebe el destino (Azure) y la suscripción del almacén y seleccione el modelo de **Resource Manager**.
1. Seleccione la cuenta de almacenamiento **contosovmsacct1910171607** y la red de Azure **ContosoASRnet**.
1. **Máquinas virtuales** > **Seleccionar**. Seleccione la máquina virtual que desea replicar. Después, seleccione **Aceptar**.

   Puede hacer un seguimiento del progreso de la acción **Habilitar protección** en **Trabajos** > **Trabajos de Site Recovery**. Una vez concluido el trabajo **Finalizar protección**, la replicación inicial finaliza y la máquina virtual está preparada para la conmutación por error.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exploración de la recuperación ante desastres](tutorial-dr-drill-azure.md)
