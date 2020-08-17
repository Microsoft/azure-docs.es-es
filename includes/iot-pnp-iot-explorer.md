---
author: ericmitt
ms.author: ericmitt
ms.service: iot-pnp
ms.topic: include
ms.date: 07/13/2020
ms.openlocfilehash: 3ab9161c006a88a254b247a921512698ed46fa95
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87351927"
---
1. Abra el explorador Azure IoT.

1. En la página **Centros de IoT**, si aún no ha agregado una conexión al centro de IoT, seleccione **+ Agregar conexión**. Escriba la cadena de conexión del centro de IoT que creó previamente y seleccione **Guardar**.

1. En la página **IoT Plug and Play Settings** (Configuración de IoT Plug and Play), seleccione **+ Agregar > Carpeta local** y seleccione la carpeta local *models* donde guardó los archivos de modelo.

1. En la página **Centros de IoT**, haga clic en el nombre del centro con el que desea trabajar. Verá una lista de dispositivos registrados en el centro de IoT.

1. Haga clic en el **Id. de dispositivo** del dispositivo que creó anteriormente.

1. En el menú de la izquierda se muestran los diferentes tipos de información disponibles para el dispositivo.

1. Seleccione **IoT Plug and Play components** (Componentes de IoT Plug and Play) para ver la información del modelo del dispositivo.

1. Puede ver los distintos componentes del dispositivo. El componente predeterminado y los adicionales. Seleccione un componente con el que trabajar.

1. Seleccione la página **Telemetría** y haga clic en **Iniciar** para ver los datos de telemetría que envía el dispositivo para este componente.

1. Seleccione la página **Propiedades (solo lectura)** para ver las propiedades de solo lectura que se muestran para este componente.

1. Seleccione la página **Properties (writable)** (Propiedades [editables]) para ver las propiedades de escritura que puede actualizar para este componente.

1. Seleccione una propiedad por su **nombre**, escriba un nuevo valor para ella y seleccione **Update desired value** (Actualizar el valor deseado).

1. Para ver el nuevo valor, seleccione el botón **Actualizar**.

1. Seleccione la página **Comandos** para ver todos los comandos para este componente.

1. Seleccione el comando que desea probar para establecer el parámetro, si existe. Seleccione **Enviar comando** para llamar al comando en el dispositivo. Puede ver que el dispositivo responde al comando en la ventana del símbolo del sistema donde se está ejecutando el código de ejemplo.
