---
ms.openlocfilehash: 67d90836c382728f989ab2cb4fde4d81bac9eb25
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691057"
---
Este tutorial requiere los siguientes recursos de Azure:

* IoT Hub
* Cuenta de almacenamiento
* Cuenta de Azure Media Services
* Máquina virtual Linux en Azure, con el [entorno de ejecución de Azure IoT Edge](../../../../../iot-edge/how-to-install-iot-edge-linux.md) instalado

Para este inicio rápido, se recomienda usar el [script de configuración de recursos de Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) para implementar los recursos necesarios en su suscripción de Azure. Para hacerlo, siga estos pasos:

1. Abra [Azure Cloud Shell](https://shell.azure.com).
1. Si es la primera vez que usa Cloud Shell, se le pedirá que seleccione una suscripción para crear una cuenta de almacenamiento y un recurso compartido de Microsoft Azure Files. Seleccione **Create storage** (Crear almacenamiento) para crear una cuenta de almacenamiento para la información de la sesión de Cloud Shell. Esta cuenta de almacenamiento es independiente de la que creará el script para usarla con su cuenta de Azure Media Services.
1. En el menú desplegable del lado izquierdo de la ventana de Cloud Shell, seleccione el entorno **Bash**.

    ![Selector de entorno](../../../media/quickstarts/env-selector.png)
1. Ejecute el siguiente comando.

    ```
    bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
    ```
    
    Si el script finaliza correctamente, debería ver todos los recursos necesarios en su suscripción.
1. Después de que finalice el script, seleccione las llaves para exponer la estructura de carpetas. Verá algunos archivos en el directorio *~/clouddrive/lva-sample*. Archivos de interés en este inicio rápido:

     * ***~/clouddrive/lva-sample/edge-deployment/.env***: este archivo contiene las propiedades que utiliza Visual Studio Code para implementar módulos en un dispositivo perimetral.
     * ***~/clouddrive/lva-sample/appsetting.json***: Visual Studio Code utiliza este archivo para ejecutar el código de ejemplo.
     
    Necesitará estos archivos al configurar el entorno de desarrollo en Visual Studio Code en la sección siguiente. Es posible que desee copiarlos en un archivo local por el momento.
    
    ![Configuración de la aplicación](../../../media/quickstarts/clouddrive.png)
    