---
title: Configuración de la recuperación ante desastres de Hyper-V mediante Azure Site Recovery
description: Aprenda a configurar la recuperación ante desastres de máquinas virtuales de Hyper-V locales (sin VMM) en Azure con Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 11/12/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 5ce98c785700301bba92926d7d5a243b614eca7f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504235"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-to-azure"></a>Configurar la recuperación ante desastres de máquinas virtuales de Hyper-V locales en Azure

El servicio [Azure Site Recovery](site-recovery-overview.md) contribuye a la estrategia de recuperación ante desastres mediante la administración y la coordinación de la replicación, la conmutación por error y la conmutación por recuperación de máquinas locales y máquinas virtuales (VM) de Azure.

Este es el tercer tutorial de una serie. En él se muestra cómo configurar la recuperación ante desastres de máquinas virtuales de Hyper-V locales en Azure. Este tutorial se aplica a las máquinas virtuales de Hyper-V no administradas por Microsoft System Center Virtual Machine Manager (VMM).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Seleccionar el origen y destino de la replicación.
> * Configurar el entorno de replicación de origen, incluidos los componentes de Site Recovery locales y el entorno de replicación de destino.
> * Crear una directiva de replicación.
> * Habilitar la replicación para una máquina virtual.

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Usan opciones predeterminadas siempre que es posible y no muestran todos los valores y las rutas de acceso posibles. Para instrucciones detalladas, revise los artículos de la sección de **procedimientos** de la [documentación de Site Recovery](./index.yml).



## <a name="before-you-begin"></a>Antes de empezar

Este es el tercer tutorial de una serie. En él se da por hecho que ya ha realizado las tareas de los tutoriales anteriores:

1. [Preparación de Azure](tutorial-prepare-azure.md)
2. [Preparación de un entorno de Hyper-V local](./hyper-v-prepare-on-premises-tutorial.md)

## <a name="select-a-replication-goal"></a>Selección de un objetivo de replicación

1. En Azure Portal, vaya a **Almacenes de Recovery Services** y seleccione el almacén. Hemos preparado el almacén **ContosoVMVault** en el tutorial anterior.
2. En **Introducción**, seleccione **Site Recovery**y, luego, **Preparar la infraestructura**.
3. En **Objetivo de protección** >  **¿Dónde están ubicadas las máquinas?** , seleccione **Local**.
4. En **¿En dónde quiere replicar las máquinas?** , seleccione **En Azure**.
5. En **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**.
6. En **Utiliza System Center VMM para administrar los hosts de Hyper-V**, seleccione **No**.
7. Seleccione **Aceptar**.

    ![Captura de pantalla de las opciones de Objetivo de protección en Preparar infraestructura.](./media/hyper-v-azure-tutorial/replication-goal.png)

## <a name="confirm-deployment-planning"></a>Confirmación del planeamiento de la implementación

1. En **Planeamiento de implementación**, si planea una implementación de gran tamaño, descargue Deployment Planner para Hyper-V desde el vínculo de la página. [Obtenga más información](hyper-v-deployment-planner-overview.md) acerca del planeamiento de implementaciones de Hyper-V.
2. En este tutorial, no se necesita Deployment Planner. En **¿Completó el planeamiento de implementación?** , seleccione **Lo haré más tarde** y, luego, seleccione **Aceptar**.

    ![Captura de pantalla de las opciones de Planeamiento de implementación en Preparar infraestructura.](./media/hyper-v-azure-tutorial/deployment-planning.png)

## <a name="set-up-the-source-environment"></a>Configuración del entorno de origen

Para configurar el entorno de origen, cree un sitio de Hyper-V y agregue a ese sitio los hosts de Hyper-V que contengan las máquinas virtuales que quiere replicar. A continuación, descargue e instale el proveedor de Azure Site Recovery y el agente de Azure Recovery Services en cada host y registre el sitio Hyper-V en el almacén.

1. En **Preparar infraestructura**, seleccione **Origen**.
2. En **Preparar origen**, seleccione **+ Sitio Hyper-V**.
3. En **Crear sitio Hyper-V** , especifique el nombre del sitio. Vamos a usar **ContosoHyperVSite**.

    ![Captura de pantalla de la selección de Sitio Hyper-V en Preparar infraestructura.](./media/hyper-v-azure-tutorial/hyperv-site.png)

4. Una vez creado el sitio, en **Preparar origen** > **Paso 1: Seleccionar sitio de Hyper-V**, seleccione el sitio que creó.
5. Seleccione **+ Servidor de Hyper-V**.

    ![Captura de pantalla de la selección de Hyper-V Server en Preparar infraestructura.](./media/hyper-v-azure-tutorial/hyperv-server.png)

6. Descargue el programa de instalación del proveedor de Microsoft Azure Site Recovery.
7. Descargue la clave de registro del almacén. Necesitará esta clave para instalar el proveedor. La clave será válida durante cinco días a partir del momento en que se genera.

    ![Captura de pantalla de las opciones para descargar el proveedor y la clave de registro.](./media/hyper-v-azure-tutorial/download.png)
    

### <a name="install-the-provider"></a>Instalación del proveedor

Instale el archivo de configuración descargado (AzureSiteRecoveryProvider.exe) en cada host de Hyper-V que quiera agregar al sitio Hyper-V. El programa de instalación instala el proveedor de Azure Site Recovery y el agente de Recovery Services en cada host de Hyper-V.

1. Ejecute el archivo de configuración.
2. En el Asistente para la instalación del proveedor de Azure Site Recovery > **Microsoft Update**, decida usar Microsoft Update para comprobar las actualizaciones del proveedor.
3. En **Instalación**, acepte la ubicación predeterminada de instalación para el proveedor y el agente y haga clic en **Instalar**.
4. Después de la instalación, en el Asistente para registro de Microsoft Azure Site Recovery > **Configuración del almacén**, seleccione **Examinar** y, en **Archivo de clave**, seleccione el archivo de clave del almacén que descargó.
5. Especifique la suscripción de Azure Site Recovery, el nombre del almacén (**ContosoVMVault**) y el sitio de Hyper-V (**ContosoHyperVSite**) al que pertenece el servidor Hyper-V.
6. En **Configuración de proxy**, seleccione **Conectar directamente con Azure Site Recovery sin un servidor proxy**.
7. En **Registro**, cuando el servidor se haya registrado en el almacén, seleccione **Finalizar**.

Los metadatos del servidor de Hyper-V se recuperan mediante Azure Site Recovery y el servidor se muestra en **Site Recovery Infrastructure** > **Hyper-V Hosts** (Infraestructura de Site Recovery > Hosts de Hyper-V). Este proceso puede tardar hasta 30 minutos.

#### <a name="install-the-provider-on-a-hyper-v-core-server"></a>Instalación del proveedor en un servidor central de Hyper-V

Si ejecuta un servidor central de Hyper-V, descargue el archivo de instalación y haga lo siguiente:

1. Extraiga los archivos de AzureSiteRecoveryProvider.exe a un directorio local mediante este comando:

    `AzureSiteRecoveryProvider.exe /x:. /q`
 
2. Ejecute `.\setupdr.exe /i`. Los resultados se registran en %Programdata%\ASRLogs\DRASetupWizard.log.

3. Registre el servidor mediante este comando:

    ```
    cd  "C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe" /r /Friendlyname "FriendlyName of the Server" /Credentials "path to where the credential file is saved"
    ```

## <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

Seleccione y compruebe los recursos de destino:

1. Seleccione **Preparar infraestructura** > **Destino**.
2. Seleccione la suscripción y el grupo de recursos **ContosoRG** donde se crearán las máquinas virtuales de Azure después de la conmutación por error.
3. Seleccione el modelo de implementación de **Resource Manager**.

Site Recovery comprueba que tiene una o más redes y cuentas de Azure Storage compatibles.

## <a name="set-up-a-replication-policy"></a>Configurar una directiva de replicación

1. Seleccione **Preparar infraestructura** > **Configuración de la replicación** >  **+Crear y asociar**.
2. En **Crear y asociar directiva**, especifique un nombre de directiva. Usamos **ContosoReplicationPolicy**.
3. En este tutorial, dejaremos los valores predeterminados:
    - **Frecuencia de copia** indica la frecuencia con la que se replicarán los datos diferenciales (después de la replicación inicial). La frecuencia predeterminada es cada cinco minutos.
    - **Retención de punto de recuperación** indica que los puntos de recuperación se conservarán durante dos horas. El valor máximo permitido de retención al proteger las máquinas virtuales hospedadas en hosts de Hyper-V es de 24 horas.
    - **Frecuencia de instantánea coherente con la aplicación** indica que cada hora se crearán puntos de recuperación que contengan instantáneas coherentes con la aplicación.
    - **Hora de inicio de la replicación inicial** indica que la replicación inicial comenzará de inmediato.
4. Después de que se cree la directiva, seleccione **Aceptar**. Cuando se crea una directiva, esta se asocia automáticamente con el sitio Hyper-V. En este tutorial, este sitio es **ContosoHyperVSite**.

    ![Directiva de replicación](./media/hyper-v-azure-tutorial/replication-policy.png)

## <a name="enable-replication"></a>Habilitar replicación

1. En **Replicar la aplicación**, seleccione **Origen**.
2. En **Origen**, seleccione el sitio **ContosoHyperVSite**. Después, seleccione **Aceptar**.
3. En **Destino**, compruebe Azure como destino, la suscripción del almacén y el modelo de implementación de **Resource Manager**.
4. Si usa la configuración del tutorial, seleccione la cuenta de almacenamiento **contosovmsacct1910171607** creada en el tutorial anterior para los datos replicados. Seleccione también la red **ContosoASRnet**, donde se ubicarán las máquinas virtuales de Azure tras la conmutación por error.
5. En **Máquinas virtuales** > **Seleccionar**, seleccione la máquina virtual que quiere replicar. Después, seleccione **Aceptar**.

   Puede hacer un seguimiento del progreso de la acción **Habilitar protección** en **Trabajos** > **Trabajos de Site Recovery**. Una vez concluido el trabajo **Finalizar protección**, la replicación inicial finaliza y la máquina virtual está preparada para la conmutación por error.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Exploración de la recuperación ante desastres](tutorial-dr-drill-azure.md)
