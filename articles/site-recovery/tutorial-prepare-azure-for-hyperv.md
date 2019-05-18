---
title: Preparación de los recursos de Azure para la recuperación ante desastres de máquinas locales
description: Aprenda a preparar Azure para la recuperación ante desastres de máquinas virtuales locales de Hyper-V mediante Azure Site Recovery.
author: rayne-wiselman
ms.service: site-recovery
services: site-recovery
ms.topic: tutorial
ms.date: 04/08/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 29189a5919a01fcb897758fb64ca9e84b9381fb6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410902"
---
# <a name="prepare-azure-resources-for-disaster-recovery-of-on-premises-machines"></a>Preparación de los recursos de Azure para la recuperación ante desastres de máquinas locales

 [Azure Site Recovery](site-recovery-overview.md) ayuda con la continuidad empresarial y la recuperación ante desastres (BCDR) al mantener sus aplicaciones empresariales en ejecución durante las interrupciones planeadas y no planeadas. Azure Site Recovery administra y coordina la recuperación ante desastres de máquinas locales y máquinas virtuales de Azure, lo que incluye la replicación, la conmutación por error y la recuperación.

Este tutorial es el primero de una serie que describe cómo configurar la recuperación ante desastres para máquinas virtuales locales de Hyper-V.

> [!NOTE]
> Los tutoriales se diseñan para mostrarle la ruta de implementación más sencilla para un escenario. Estos tutoriales usan opciones predeterminadas siempre que es posible y no muestran todos los valores y las rutas de acceso posibles. Para más información, consulte la sección de "procedimientos" de cada escenario correspondiente.

En este tutorial se muestra cómo preparar los componentes de Azure cuando se desean replicar máquinas virtuales locales (Hyper-V) en Azure. Aprenderá a:

> [!div class="checklist"]
> * Compruebe que la cuenta de Azure tiene permisos de replicación.
> * Cree una cuenta de almacenamiento de Azure que almacene imágenes de las máquinas replicadas.
> * Cree un almacén de Recovery Services que almacene metadatos e información de configuración de las máquinas virtuales y otros componentes de replicación.
> * Configure una red de Azure. Cuando se crean máquinas virtuales de Azure después de la conmutación por error, se unen a esta red.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="sign-in"></a>Iniciar sesión

Inicie sesión en el [Azure Portal](https://portal.azure.com).

## <a name="verify-account-permissions"></a>Comprobar los permisos de la cuenta

Si acaba de crear su cuenta de Azure gratis, ya es el administrador de esa suscripción. Si no lo es, solicite al administrador que le asigne los permisos que necesita. Para habilitar la replicación de una nueva máquina virtual, debe tener permiso para:

- Crear una máquina virtual en el grupo de recursos seleccionado.
- Crear una máquina virtual en la red virtual seleccionada.
- Escribir en la cuenta de almacenamiento seleccionada.

Para completar estas tareas su cuenta debe tener asignado el rol integrado de colaborador de la máquina virtual. Para administrar las operaciones de Site Recovery en un almacén, su cuenta debe tener asignado el rol integrado de colaborador de Site Recovery.

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Las imágenes de máquinas replicadas se conservan en Azure Storage. Las máquinas virtuales de Azure se crean desde el almacenamiento cuando se realiza la conmutación por error desde el entorno local en Azure. La cuenta de almacenamiento debe estar en la misma región que el almacén de Recovery Services.

1. En el menú [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Almacenamiento** > **Cuenta de almacenamiento: blob, archivo, tabla, cola**.
2. En **Crear cuenta de almacenamiento**, escriba un nombre para la cuenta.  El nombre que elija debe ser único dentro de Azure, tener de 3 a 24 caracteres y usar solo letras minúsculas y números. Para este tutorial, use **contosovmsacct1910171607**.
3. En **Modelo de implementación**, seleccione **Resource Manager**.
4. En **Tipo de cuenta** seleccione **Storage (uso general v1)**. No seleccione Blob Storage.
5. En **Replicación**, seleccione el valor predeterminado **Almacenamiento con redundancia geográfica con acceso de lectura** como redundancia de almacenamiento. Deje la opción "Se requiere transferencia segura" como deshabilitada.
6. En **Rendimiento**, seleccione **Estándar**. A continuación, en **Nivel de acceso**, seleccione la opción predeterminada **Frecuente**.
7. En **Suscripción**, elija la suscripción en la que desea crear la nueva cuenta de almacenamiento.
8. En **Grupo de recursos**, especifique un nuevo grupo de recursos. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Use **ContosoRG** en este tutorial.
9. En **Ubicación**, elija la ubicación geográfica de la cuenta de almacenamiento. Para este tutorial, use **Oeste de Europa**.
10. Seleccione **Crear** para crear la cuenta de almacenamiento.

   ![Crear una cuenta de almacenamiento](media/tutorial-prepare-azure/create-storageacct.png)

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Servicios de recuperación

1. En Azure Portal, seleccione **+Crear un recurso** y, en Azure Marketplace, busque Recovery Services.
2. Seleccione **Backup and Site Recovery (OMS)**. A continuación, en la página **Backup and Site Recovery**, seleccione **Crear**.
1. En **Almacén de Recovery Services > Nombre**, escriba un nombre descriptivo para identificar el almacén. En este tutorial, se usa **ContosoVMVault**.
2. En **Grupo de recursos**, seleccione un grupo de recursos existente o cree uno nuevo. Use **contosoRG** en este tutorial.
3. En **Ubicación**, seleccione la región en la que debería estar el almacén. Para este tutorial, use **Oeste de Europa**.
4. Para acceder rápidamente al almacén desde el panel, seleccione **Anclar al panel** > **Crear**.

![Crear un nuevo almacén](./media/tutorial-prepare-azure/new-vault-settings.png)

El nuevo almacén aparecerá en **Panel** > **Todos los recursos** y en la página principal de **Almacenes de Recovery Services**.

## <a name="set-up-an-azure-network"></a>Configurar una red de Azure

Cuando se crean máquinas virtuales de Azure desde el almacenamiento después de la conmutación por error, se unen a esta red.

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Redes** > **Red virtual**. Deje Resource Manager seleccionado como modelo de implementación.
2. En **Nombre**, escriba un nombre de red. El nombre debe ser único dentro del grupo de recursos de Azure. Use **ContosoASRnet**en este tutorial.
3. Especifique el grupo de recursos en el que desea crear la red. En este tutorial, use el grupo de recursos existente **contosoRG**.
4. En **Intervalo de direcciones** , escriba **10.0.0.0/24** como intervalo de la red. No hay ninguna subred para esta red.
5. En **Suscripción**, seleccione la suscripción en la que se creará la red.
6. En **Ubicación**, elija **Europa Occidental**. La red virtual de Azure debe estar en la misma región que el almacén de Recovery Services.
7. Deje las opciones predeterminadas de protección básica contra DDoS, con ningún punto de conexión de servicio en la red.
8. Seleccione **Crear**.

![Creación de una red virtual](media/tutorial-prepare-azure/create-network.png)

La red virtual tarda unos segundos en crearse. Una vez creada, la verá en el panel de Azure Portal.

## <a name="useful-links"></a>Vínculos útiles

Más información sobre lo siguiente:
- [Redes de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)
- [Discos administrados](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview)



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Preparación de la infraestructura de Hyper-V local para la recuperación ante desastres en Azure](hyper-v-prepare-on-premises-tutorial.md)
