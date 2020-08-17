---
title: Configuración de un laboratorio con GPU en Azure Lab Services | Microsoft Docs
description: Aprenda a configurar un laboratorio con máquinas virtuales de unidad de procesamiento de gráficos (GPU).
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: f41ad80e0e39d66020d039d6229a4b0fc62627f1
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286009"
---
# <a name="set-up-a-lab-with-gpu-virtual-machines"></a>Configuración de un laboratorio con máquinas virtuales de GPU

En este artículo se muestra cómo realizar las tareas siguientes:

- Elija entre unidades de procesamiento de gráficos de *visualización* y de *proceso*.
- Asegúrese de que estén instalados los controladores de GPU adecuados.

## <a name="choose-between-visualization-and-compute-gpu-sizes"></a>Elección entre tamaños de GPU de visualización y proceso
En la primera página del asistente para la creación de laboratorio, en la lista desplegable **Which virtual machine size do you need?** (¿Qué tamaño de máquina virtual necesita?), seleccione el tamaño de las máquinas virtuales que necesita para la clase.  

![Captura de pantalla del panel "New lab" (Nuevo laboratorio) para seleccionar un tamaño de máquina virtual](./media/how-to-setup-gpu/lab-gpu-selection.png)

En este proceso, tiene la opción de seleccionar GPU de **visualización** o de **proceso**.  Es importante elegir el tipo de GPU que se basa en el software que usarán los alumnos.  

Tal como se describe en la tabla siguiente, el tamaño de GPU de *proceso* está diseñado para aplicaciones de proceso intensivo.  Por ejemplo, el [tipo de clase Aprendizaje profundo en el procesamiento del lenguaje natural](./class-type-deep-learning-natural-language-processing.md) usa el tamaño **GPU pequeña (proceso)** .  La GPU de proceso es adecuada para este tipo de clase, ya que los alumnos usan plataformas de aprendizaje profundo y herramientas que se proporcionan en la [imagen de Data Science Virtual Machine](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804) para entrenar modelos de aprendizaje profundo con grandes conjuntos de datos.

| Size | Núcleos | RAM | Descripción | 
| ---- | ----- | --- | ----------- | 
| GPU pequeña (proceso) | -&nbsp;6&nbsp;núcleos<br>-&nbsp;56&nbsp;GB&nbsp;de RAM  | [Standard_NC6](https://docs.microsoft.com/azure/virtual-machines/nc-series) |Este tamaño es más adecuado para aplicaciones de proceso intensivo, como la inteligencia artificial (IA) y el aprendizaje profundo. |

Los tamaños de GPU de *visualización* están diseñados para aplicaciones con uso intensivo de gráficos.  Por ejemplo, el [tipo de clase Ingeniería de SOLIDWORKS](./class-type-solidworks.md) muestra cómo usar el tamaño **GPU pequeña (visualización)** .  La GPU de visualización es adecuada para este tipo de clase, ya que los alumnos interactúan con el entorno de diseño asistido por PC (CAD) de SOLIDWORKS 3D para el modelado y la visualización de objetos sólidos.

| Size | Núcleos | RAM | Descripción | 
| ---- | ----- | --- | ----------- | 
| GPU pequeña (visualización) | -&nbsp;6&nbsp;núcleos<br>-&nbsp;56&nbsp;GB&nbsp;de RAM  | [Standard_NV6](https://docs.microsoft.com/azure/virtual-machines/nv-series) | Este tamaño es el más adecuado para la visualización, la realización de streaming, los juegos y la codificación de forma remota que usan marcos como OpenGL y DirectX. |
| GPU mediana (visualización) | -&nbsp;12&nbsp;núcleos<br>-&nbsp;112&nbsp;GB&nbsp;de RAM  | [Standard_NV12](https://docs.microsoft.com/azure/virtual-machines/nv-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) | Este tamaño es el más adecuado para la visualización, la realización de streaming, los juegos y la codificación de forma remota que usan marcos como OpenGL y DirectX. |

## <a name="ensure-that-the-appropriate-gpu-drivers-are-installed"></a>Garantía de que los controladores de GPU adecuados estén instalados
Para aprovechar las funcionalidades de GPU de las máquinas virtuales de laboratorio, asegúrese de que se instalen los controladores de GPU adecuados.  En el asistente para la creación de laboratorio, cuando seleccione un tamaño de VM de GPU, puede seleccionar la opción **Install GPU drivers** (Instalar controladores de GPU).  

![Captura de pantalla del panel "New lab" (Nuevo laboratorio) que muestra la opción "Install GPU drivers" (Instalar controladores de GPU)](./media/how-to-setup-gpu/lab-gpu-drivers.png)

Como se muestra en la imagen anterior, está opción está habilitada de manera predeterminada, lo que garantiza que los controladores *más recientes* estén instalados para el tipo de GPU e imagen que seleccionó.
- Al seleccionar un tipo de GPU de *proceso*, las máquinas virtuales de laboratorio cuentan con tecnología de la GPU [NVIDIA Tesla K80](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-product-literature/Tesla-K80-BoardSpec-07317-001-v05.pdf).  En este caso, se instalan los controladores más recientes de [Compute Unified Device Architecture (CUDA)](http://developer.download.nvidia.com/compute/cuda/2_0/docs/CudaReferenceManual_2.0.pdf), lo que permite una informática de alto rendimiento.
- Al seleccionar un tamaño de GPU de *visualización*, las máquinas virtuales de laboratorio cuentan con tecnología de la GPU [NVIDIA Tesla M60](https://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) y la [tecnología GRID](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/solutions/resources/documents1/NVIDIA_GRID_vPC_Solution_Overview.pdf).  En este caso, se instalan los controladores de GRID más recientes, lo que permite el uso de aplicaciones con uso intensivo de gráficos.

### <a name="install-the-drivers-manually"></a>Instalación manual de los controladores
Es posible que tenga que instalar una versión de controlador distinta de la versión más reciente.  En esta sección se muestra cómo instalar manualmente los controladores adecuados, en función de si se usa una GPU de *proceso* o una GPU de *visualización*.

#### <a name="install-the-compute-gpu-drivers"></a>Instalación de los controladores de GPU de proceso

Para instalar manualmente los controladores para el tamaño de la GPU de proceso, haga lo siguiente:

1. En el asistente para la creación de laboratorio, al [crear el laboratorio](./how-to-manage-classroom-labs.md), deshabilite la opción **Install GPU drivers** (Instalar controladores de GPU).

1. Una vez creado el laboratorio, conéctese a la máquina virtual de plantilla para instalar los controladores adecuados.

   ![Captura de pantalla de la página de descarga de controladores de NVIDIA](./media/how-to-setup-gpu/nvidia-driver-download.png) 

   a. En un explorador, vaya a la [página de descarga de controladores de NVIDIA](https://www.nvidia.com/Download/index.aspx).  
   b. Establezca **Product Type** (Tipo de producto) en **Tesla**.  
   c. Establezca **Product Series** (Serie del producto) en **K-Series**.  
   d. Establezca **Operating System** (Sistema operativo) según el tipo de imagen base que seleccionó al crear el laboratorio.  
   e. Establezca **CUDA Toolkit** (Kit de herramientas de CUDA) en la versión del controlador de CUDA que necesita.  
   f. Seleccione **Search** (Buscar) para buscar los controladores.  
   g. Seleccione **Download** (Descargar) para descargar el instalador.  
   h. Ejecute el instalador para que los controladores se instalen en la máquina virtual de plantilla.  
1. Para comprobar que los controladores se instalaron correctamente, siga las instrucciones que aparecen en la sección [Validación de los controladores instalados](how-to-setup-lab-gpu.md#validate-the-installed-drivers). 
1. Una vez instalados los controladores y otro software necesario para la clase, seleccione **Publicar** para crear las máquinas virtuales de los alumnos.

> [!NOTE]
> Si usa una imagen de Linux, después de descargar el instalador, instale los controladores según las instrucciones que aparecen en [Instalación de controladores CUDA en Linux](https://docs.microsoft.com/azure/virtual-machines/linux/n-series-driver-setup?toc=/azure/virtual-machines/linux/toc.json#install-cuda-drivers-on-n-series-vms).

#### <a name="install-the-visualization-gpu-drivers"></a>Instalación de los controladores de GPU de visualización

Para instalar manualmente los controladores para el tamaño de la GPU de visualización, haga lo siguiente:

1. En el asistente para la creación de laboratorio, al [crear el laboratorio](./how-to-manage-classroom-labs.md), deshabilite la opción **Install GPU drivers** (Instalar controladores de GPU).
1. Una vez creado el laboratorio, conéctese a la máquina virtual de plantilla para instalar los controladores adecuados.
1. Instale los controladores de GRID que proporciona Microsoft en la máquina virtual de plantilla según las instrucciones del sistema operativo:
   -  [Controladores de NVIDIA GRID para Windows](https://docs.microsoft.com/azure/virtual-machines/windows/n-series-driver-setup#nvidia-grid-drivers)
   -  [Controladores de NVIDIA GRID para Linux](https://docs.microsoft.com/azure/virtual-machines/linux/n-series-driver-setup?toc=/azure/virtual-machines/linux/toc.json#nvidia-grid-drivers)
  
1. Reinicie la máquina virtual de plantilla.
1. Para comprobar que los controladores se instalaron correctamente, siga las instrucciones que aparecen en la sección [Validación de los controladores instalados](how-to-setup-lab-gpu.md#validate-the-installed-drivers).
1. Una vez instalados los controladores y otro software necesario para la clase, seleccione **Publicar** para crear las máquinas virtuales de los alumnos.

### <a name="validate-the-installed-drivers"></a>Validación de los controladores instalados
En esta sección se describe cómo validar que los controladores de GPU están instalados correctamente.

#### <a name="windows-images"></a>Imágenes de Windows
1.  Siga las instrucciones que aparecen en la sección de comprobación de la instalación de los controladores de [Instalación de controladores de GPU de NVIDIA en VM de la serie N con Windows](https://docs.microsoft.com/azure/virtual-machines/windows/n-series-driver-setup#verify-driver-installation).
1.  Si usa una GPU de *visualización*, también puede:
    - Ver y ajustar la configuración de GPU en el panel de control de NVIDIA. Para hacerlo, en el **Panel de control de Windows**, seleccione **Hardware** y, luego, seleccione **Panel de control de NVIDIA**.

      ![Captura de pantalla del panel de control de Windows que muestra el vínculo del panel de control de NVIDIA](./media/how-to-setup-gpu/control-panel-nvidia-settings.png) 

     - Use el **Administrador de tareas** para ver el rendimiento de la GPU.  Para hacerlo, seleccione la pestaña **Rendimiento** y, luego, la opción **GPU**.

       ![Captura de pantalla que muestra la pestaña Rendimiento de la GPU del Administrador de tareas](./media/how-to-setup-gpu/task-manager-gpu.png) 

      > [!IMPORTANT]
      > Solo se puede tener acceso a la configuración del panel de control de NVIDIA para las GPU de *visualización*.  Si intenta abrir el panel de control de NVIDIA para una GPU de proceso, obtendrá el error siguiente: "NVIDIA Display settings are not available.  You are not currently using a display attached to an NVIDIA GPU". ("La configuración de la pantalla de NVIDIA no está disponible. Actualmente no usa ninguna pantalla conectada a una GPU de NVIDIA").  Del mismo modo, la información de rendimiento de la GPU en el Administrador de tareas solo se proporciona para la GPU de visualización.

#### <a name="linux-images"></a>Imágenes de Linux
Siga las instrucciones que aparecen en la sección de comprobación de la instalación de los controladores de [Instalación de controladores de GPU de NVIDIA en VM de la serie N con Linux](https://docs.microsoft.com/azure/virtual-machines/linux/n-series-driver-setup#verify-driver-installation).

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

- [Creación y administración de laboratorios educativos](how-to-manage-classroom-labs.md)
- [Tipo de clase Diseño asistido por PC (CAD) de SOLIDWORKS](class-type-solidworks.md)
- [Tipo de clase MATLAB (laboratorio de matrices)](class-type-matlab.md)



