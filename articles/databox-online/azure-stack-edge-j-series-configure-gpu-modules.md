---
title: Ejecución de un módulo de GPU en un dispositivo de Microsoft Azure Stack Edge Pro con GPU | Microsoft Docs
description: Describe cómo configurar y ejecutar un módulo en GPU en un dispositivo Azure Stack Edge Pro mediante Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/25/2020
ms.author: alkohli
ms.openlocfilehash: 8b9f1180639f638e72fdea2f87958628a2e9e86b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891470"
---
# <a name="configure-and-run-a-module-on-gpu-on-azure-stack-edge-pro-device"></a>Configuración y ejecución de un módulo en GPU en un dispositivo Azure Stack Edge Pro

El dispositivo Azure Stack Edge Pro contiene una o varias unidades de procesamiento gráfico (GPU). Las GPU son una opción popular para los cálculos de IA, ya que ofrecen funcionalidades de procesamiento en paralelo y son más rápidas en la representación de imágenes que las unidades de procesamiento central (CPU). Para más información sobre la GPU contenida en un dispositivo Azure Stack Edge Pro, vaya a [Especificaciones técnicas de dispositivos de Azure Stack Edge Pro](azure-stack-edge-gpu-technical-specifications-compliance.md).

En este artículo se describe cómo configurar y ejecutar un módulo en la GPU en un dispositivo Azure Stack Edge Pro. Aquí usará un módulo de contenedor **Digits** disponible públicamente, escrito para GPU de Nvidia T4. Este procedimiento se puede usar para configurar cualquier otro módulo publicado por Nvidia para estas GPU.


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Tiene acceso a un dispositivo Azure Stack Edge Pro con un nodo habilitado para GPU. Este dispositivo se activa con un recurso en Azure.  

## <a name="configure-module-to-use-gpu"></a>Configuración del módulo para usar la GPU

Para configurar un módulo para que use la GPU en un dispositivo Azure Stack Edge Pro para ejecutar un módulo, siga estos pasos.

1. En Azure Portal, vaya al recurso asociado con el dispositivo. 

2. Vaya a **Proceso perimetral > Comenzar**. En el icono **Configurar el proceso de Edge**, seleccione Configurar.

    ![Configuración del módulo para usar la GPU 1](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-1.png)

3. En la hoja **Configurar proceso de Edge**:

    1. En **IoT Hub**, elija **Crear nuevo**.
    2. Escriba un nombre para el recurso de IoT Hub que quiere crear para el dispositivo. Para usar un nivel gratis, seleccione un recurso existente. 
    3. Anote el dispositivo de IoT Edge y el dispositivo de puerta de enlace de IoT que se crean con el recurso de IoT Hub. Usará esta información en los pasos posteriores.

    ![Configuración del módulo para usar la GPU 2](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-2.png)

4. Se tarda unos minutos en crear el recurso de IoT Hub. Una vez creado el recurso, en el icono **Configurar proceso de Edge**, seleccione **Configuración de vista** para ver los detalles del recurso de IoT Hub.

    ![Configuración del módulo para usar la GPU 4](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-4.png)

5. Vaya a **Administración de dispositivos automática > IoT Edge**.

    ![Configuración del módulo para usar la GPU 6](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-2.png)

    En el panel derecho, se ve el dispositivo IoT Edge asociado al dispositivo Azure Stack Edge Pro. Esto corresponde al dispositivo de IoT Edge que creó en el paso anterior al crear el recurso de IoT Hub. 
    
6. Seleccione este dispositivo de IoT Edge.

   ![Configuración del módulo para usar la GPU 7](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-3.png)

7.  Seleccione **Set modules** (Establecer módulos).

    ![Configuración del módulo para usar la GPU 8](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-4.png)

8. En **+ Agregar**, seleccione **+ Módulo de IoT Edge**. 

    ![Configuración del módulo para usar la GPU 9](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-5.png)

9. En la pestaña **Agregar módulo IoT Edge**.

    1. Proporcione el **URI de la imagen**. Aquí usará el módulo **Digits** de Nvidia disponible públicamente. 
    
    2. Establezca **Directiva de reinicio** en **siempre**.
    
    3. Establezca **Estado deseado** en **en ejecución**.
    
    ![Configuración del módulo para usar la GPU 10](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-6.png)

10. En la pestaña **Variables de entorno**, indique el nombre de la variable y el valor correspondiente. 

    1. Para que el módulo actual use una GPU en este dispositivo, use NVIDIA_VISIBLE_DEVICES. 

    2. Define el valor como 0 o 1. Esto garantiza que el dispositivo use al menos una GPU para este módulo. Establecer el valor en 0 o 1 implica que este módulo está usando ambas GPU en el dispositivo.

        ![Configuración del módulo para usar la GPU 11](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-7.png)

        Para obtener más información sobre las variables de entorno que se pueden usar con la GPU de Nvidia, vaya a [nVidia container runtime](https://github.com/NVIDIA/nvidia-container-runtime#environment-variables-oci-spec).

    > [!NOTE]
    > Una GPU solo se puede asignar a un módulo. Sin embargo, un módulo puede usar una, ambas o ninguna GPU. 

11. Escriba un nombre para el módulo. En este momento, puede elegir proporcionar la opción de creación del contenedor y modificar la configuración del módulo gemelo, o bien, seleccionar **Agregar**. 

    ![Configuración del módulo para usar la GPU 12](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-8.png)

12. Asegúrese de que el módulo esté en ejecución y seleccione **Revisar y crear**.    

    ![Configuración del módulo para usar la GPU 13](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-9.png)

13. En la pestaña **Revisar y crear**, se muestran las opciones de implementación que ha seleccionado. Revise las opciones y seleccione **Crear**.
    
    ![Configuración del módulo para usar la GPU 14](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-10.png)

14. Anote el **estado de runtime** del módulo. 
    
    ![Configuración del módulo para usar la GPU 15](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-11.png)

    El módulo tarda un par de minutos en implementarse. Seleccione **Actualizar** y debería ver que el **estado de runtime** se actualiza a **en ejecución**.

    ![Configuración del módulo para usar la GPU 16](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-12.png)


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [Variables de entorno de que puede usar con la GPU de Nvidia](https://github.com/NVIDIA/nvidia-container-runtime#environment-variables-oci-spec).
