---
title: Administración de procesos en Azure Stack Edge Pro con GPU | Microsoft Docs
description: En este artículo se explica cómo administrar la configuración de procesos en Edge, como el desencadenador, los módulos, y ver y quitar la configuración por medio de Azure Portal en Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 58e199cc3c09e6b4ff333fb53c047598b1ec9b5f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890604"
---
# <a name="manage-compute-on-your-azure-stack-edge-pro-gpu"></a>Administración de procesos en Azure Stack Edge Pro con GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se explica cómo administrar procesos en Azure Stack Edge Pro. Puede administrar el proceso a través de Azure Portal o mediante la interfaz de usuario web local. Use Azure Portal para administrar los módulos, los desencadenadores y la configuración de proceso, y la interfaz de usuario web local para administrar los valores del proceso.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Administración de desencadenadores
> * Administración de la configuración del proceso


## <a name="manage-triggers"></a>Administración de desencadenadores

Los eventos son cosas que ocurren dentro del entorno de nube o en el dispositivo sobre las cuales quizás quiera tomar medidas. Por ejemplo, cuando se crea un archivo en un recurso compartido, hablamos de un evento. Los desencadenadores provocan los eventos. En Azure Stack Edge Pro, los desencadenadores pueden surgir en respuesta a eventos de archivos o según una programación.

- **Archivo**: estos desencadenadores surgen en respuesta a eventos de archivo como la creación de un archivo o su modificación.
- **Scheduled**: estos desencadenadores surgen en respuesta a una programación que puede definir con una fecha de inicio, una hora de inicio y el intervalo de repetición.


### <a name="add-a-trigger"></a>Incorporación de un desencadenador

Para crear un desencadenador, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Edge compute > Trigger** (Proceso perimetral > Desencadenador). Seleccione **+ Agregar desencadenador** en la barra de comandos.

    ![Selección de Agregar desencadenador](media/azure-stack-edge-j-series-manage-compute/add-trigger-1m.png)

2. En la hoja **Agregar desencadenador**, escriba un nombre único para el desencadenador.
    
    <!--Trigger names can only contain numbers, lowercase letters, and hyphens. The share name must be between 3 and 63 characters long and begin with a letter or a number. Each hyphen must be preceded and followed by a non-hyphen character.-->

3. Seleccione un **Tipo** para el desencadenador. Elija **Archivo** cuando el desencadenador sea la respuesta a un evento de archivo. Seleccione **Scheduled** (Programado) cuando quiera que el desencadenador se inicie a una hora determinada y se ejecuta según un intervalo de repetición especificado. Según la selección que haga, verá un conjunto distinto de opciones.

    - **File trigger** (Desencadenador de archivo): elija un recurso compartido montado en la lista desplegable. Cuando se active un evento de archivo en este recurso compartido, el desencadenador podría invocar una función de Azure.

        ![Incorporación de recurso compartido de SMB](media/azure-stack-edge-j-series-manage-compute/add-file-trigger.png)

    - **Scheduled trigger** (Desencadenador programado): especifique la fecha y hora de inicio y el intervalo de repetición en horas, minutos o segundos. Escriba también el nombre para un tema. Un tema le brindará la flexibilidad que necesita para enrutar el desencadenador a un módulo implementado en el dispositivo.

        Una cadena de ruta de ejemplo es: `"route3": "FROM /* WHERE topic = 'topicname' INTO BrokeredEndpoint("modules/modulename/inputs/input1")"`.

        ![Incorporación de un recurso compartido NFS](media/azure-stack-edge-j-series-manage-compute/add-scheduled-trigger.png)

4. Seleccione **Agregar** para crear el desencadenador. Una notificación muestra que la creación del desencadenador está en curso. Una vez creado el desencadenador, la hoja se actualiza para reflejar el desencadenador nuevo.
 
    ![Lista de desencadenadores actualizada](media/azure-stack-edge-j-series-manage-compute/add-trigger-2.png)

### <a name="delete-a-trigger"></a>Eliminación de un desencadenador

Para eliminar un desencadenador, siga estos pasos en Azure Portal.

1. En la lista de desencadenadores, seleccione el desencadenador que quiere eliminar.

    ![Seleccionar un desencadenador](media/azure-stack-edge-j-series-manage-compute/delete-trigger-1.png)

2. Haga clic con el botón derecho y seleccione **Eliminar**.

    ![Seleccionar Eliminar](media/azure-stack-edge-j-series-manage-compute/delete-trigger-2.png)

3. Cuando se le pida confirmación, haga clic en **Sí**.

    ![Confirmar eliminación](media/azure-stack-edge-j-series-manage-compute/delete-trigger-3.png)

La lista de desencadenadores se actualiza para reflejar la eliminación.

## <a name="manage-compute-configuration"></a>Administración de la configuración del proceso

Use Azure Portal para ver la configuración de proceso, quitar una configuración de proceso ya existente o actualizarla con el fin de sincronizar las claves de acceso del dispositivo IoT y el dispositivo IoT Edge de Azure Stack Edge Pro.

### <a name="view-compute-configuration"></a>Vista de la configuración de proceso

Para ver la configuración de proceso del dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Proceso perimetral > Módulos**. Seleccione **View compute** (Ver proceso) en la barra de comandos.

    ![Seleccionar Ver proceso](media/azure-stack-edge-j-series-manage-compute/view-compute-1.png)

2. Tome nota de la configuración de proceso del dispositivo. Cuando configuró el proceso, creó un recurso de IoT Hub. En ese recurso de IoT Hub, se configuró un dispositivo IoT y un dispositivo IoT Edge. Solo los módulos de Linux se admiten para ejecución en el dispositivo IoT Edge.

    ![Ver configuración](media/azure-stack-edge-j-series-manage-compute/view-compute-2.png)


### <a name="remove-compute-configuration"></a>Eliminación de la configuración de proceso

Para quitar la configuración de proceso perimetral existente del dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Proceso perimetral > Empezar**. Seleccione **Remove compute** (Quitar proceso) en la barra de comandos.

    ![Seleccionar Quitar proceso](media/azure-stack-edge-j-series-manage-compute/remove-compute-1.png)

2. Si quita la configuración de proceso, deberá volver a configurar el dispositivo en caso de que nuevamente necesite usar el proceso. Cuando se le pida confirmación, seleccione **Sí**.

    ![Seleccionar Quitar proceso](media/azure-stack-edge-j-series-manage-compute/remove-compute-2.png)

### <a name="sync-up-iot-device-and-iot-edge-device-access-keys"></a>Sincronización de las claves de acceso de dispositivo IoT y dispositivo IoT Edge

Cuando se configura el proceso en Azure Stack Edge Pro, se crea un dispositivo IoT y un dispositivo IoT Edge. A estos dispositivos se les asigna automáticamente claves de acceso simétricas. Como procedimiento recomendado de seguridad, estas claves se rotan de manera periódica a través del servicio IoT Hub.

Para rotar estas claves, puede ir al servicio IoT Hub que creó y seleccionar el dispositivo IoT o el dispositivo IoT Edge. Cada dispositivo tiene una clave de acceso primaria y una clave de acceso secundaria. Asigne la clave de acceso primaria a la clave de acceso secundario y luego vuelva a generar la clave de acceso primaria.

Si las claves del dispositivo IoT y del dispositivo IoT Edge se han rotado, deberá actualizar la configuración en Azure Stack Edge Pro para tener las claves de acceso más recientes. La sincronización permite que el dispositivo obtenga las claves más recientes del dispositivo IoT y del dispositivo IoT Edge. Azure Stack Edge Pro solo usa las claves de acceso primarias.

Para sincronizar las claves de acceso del dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Proceso perimetral > Comenzar**. Seleccione **Refresh configuration** (Actualizar configuración) en la barra de comandos.

    ![Seleccionar Actualizar configuración](media/azure-stack-edge-j-series-manage-compute/refresh-configuration-1.png)

2. Seleccione **Sí** cuando se pida confirmación.

    ![Seleccionar Sí cuando se pida confirmación](media/azure-stack-edge-j-series-manage-compute/refresh-configuration-2.png)

3. Salga del cuadro de diálogo cuando haya finalizado la sincronización.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [solucionar problemas en Azure Stack Edge Pro](azure-stack-edge-gpu-troubleshoot.md).
