---
title: Inicio rápido de Microsoft Azure Data Box Disk | Microsoft Docs
description: Use esta guía de inicio rápido para implementar rápidamente Azure Data Box Disk en Azure Portal
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: quickstart
ms.date: 01/09/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to quickly deploy Data Box Disk so as to import data into Azure.
ms.openlocfilehash: 3b158e0743a811f0d8f478c15b64c2b8b99a748a
ms.sourcegitcommit: 33091f0ecf6d79d434fa90e76d11af48fd7ed16d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/09/2019
ms.locfileid: "54156007"
---
# <a name="quickstart-deploy-azure-data-box-disk-using-the-azure-portal"></a>Inicio rápido: Implementación de Azure Data Box Disk mediante Azure Portal

En esta guía de inicio rápido se describe cómo implementar Azure Data Box Disk mediante Azure Portal. Los pasos incluyen cómo crear un pedido, recibir los discos, desempaquetarlos, conectarlos y copiar datos en los discos para que se carguen en Azure rápidamente. 

Para obtener instrucciones detalladas para realizar una implementación y el seguimiento del proceso, vaya a [Tutorial: Pedido de Azure Data Box](data-box-disk-deploy-ordered.md). 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar:

- Asegúrese de que su suscripción está habilitada para el servicio Azure Data Box. Para habilitar la suscripción para este servicio, [regístrese en el servicio](https://aka.ms/azuredataboxfromdiskdocs).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [http://aka.ms/azuredataboxfromdiskdocs](https://aka.ms/azuredataboxfromdiskdocs).

## <a name="order"></a>Orden

Este paso tarda aproximadamente 5 minutos.

1. Cree un nuevo recurso de Azure Data Box en Azure Portal. 
2. Seleccione una suscripción habilitada para este servicio y elija el tipo de transferencia como de **importación**. Proporcione el **país de origen** donde residen los datos y la **región de destino de Azure** para la transferencia de datos.
3. Seleccione **Data Box Disk**. La capacidad máxima de la solución es de 35 TB y puede crear varios pedidos de disco para los tamaños de datos mayores.  
4. Escriba los detalles del pedido y la información de envío. Si el servicio está disponible en su región, proporcione las direcciones de correo electrónico de notificación, revise el resumen, y a continuación, cree el pedido. 

Una vez que se creó el pedido, los discos están preparados para su envío. 

## <a name="unpack"></a>Desempaquetado

Este paso tarda aproximadamente 5 minutos.

Data Box Disk se envía por correo en una caja de UPS Express. Abra la caja y compruebe que contiene:

- De 1 a 5 discos USB en envoltorio de burbujas.
- Un cable de conexión por disco. 
- Una etiqueta para el envío de devolución.

## <a name="connect-and-unlock"></a>Conexión y desbloqueo

Este paso tarda aproximadamente 5 minutos.

1. Utilice el cable incluido para conectar el disco a una máquina Windows/Linux que ejecute un sistema operativo compatible. Para más información sobre las versiones de sistemas operativos compatibles, vaya a [Requisitos del sistema de Azure Data Box Disk](data-box-disk-system-requirements.md). 
2. Para desbloquear el disco:

    1. En Azure Portal, vaya a **General > Detalles del dispositivo** y obtenga la clave de paso.
    2. Descargue y extraiga la herramienta de desbloqueo de Data Box Disk específica del sistema operativo en el equipo que se usa para copiar los datos en discos. 
    3. Ejecute la herramienta de desbloqueo de Data Box Disk y proporcione la clave de paso. Si desea reinsertar algún disco, vuelva a ejecutar la herramienta de desbloqueo y escriba la clave de paso. **No utilice el cuadro de diálogo de BitLocker ni la clave de BitLocker para desbloquear el disco.** Para más información sobre cómo desbloquear los discos, vaya a [Desbloqueo de discos en el cliente de Windows](data-box-disk-deploy-set-up.md#unlock-disks-on-windows-client) o [Desbloqueo de discos en el cliente de Linux](data-box-disk-deploy-set-up.md#unlock-disks-on-linux-client).
    4. La herramienta muestra la letra de unidad asignada al disco. Tome nota de la letra de unidad del disco. Se usa en los pasos siguientes.

## <a name="copy-data-and-validate"></a>Copia de datos y validación

El tiempo en completar esta operación depende del tamaño de los datos. 

1. La unidad contiene las carpetas *PageBlob*, *BlockBlob* y *DataBoxDiskImport*. Arrastre y coloque los datos que deben importarse como blobs en bloques en la carpeta *BlockBlob* para copiarlos. De forma similar, puede arrastrar y colocar los datos como VHD/VHDX en la carpeta *PageBlob*.

    Se crea un contenedor en la cuenta de almacenamiento de Azure para cada subcarpeta bajo las carpetas *BlockBlob* y *PageBlob*. Todos los archivos bajo las carpetas *BlockBlob* y *PageBlob* se copian en un contenedor predeterminado `$root` bajo la cuenta de Azure Storage.

    > [!NOTE] 
    > - Todos los contenedores y blobs deben adecuarse a las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-and-page-blob-naming-conventions). Si no se siguen estas reglas, se producirá un error en la carga de datos en Azure.
    > - Asegúrese de que los archivos no superen ~4,75 TiB para blobs en bloques y ~8 TiB para blobs en páginas.

2. (Opcional) Una vez completada la copia, le recomendamos que ejecute el script `DataBoxDiskValidation.cmd` proporcionado en la carpeta *DataBoxDiskImport* para generar sumas de comprobación para la validación. Según el tamaño de los datos, este paso puede tardar un rato. 
3. Desconecte la unidad. 

## <a name="ship-to-azure"></a>Envío a Azure

Este paso tarda entre 5 y 7 minutos en completarse.

1. Coloque todos los discos juntos en el paquete original. Utilice la etiqueta de envío incluida. Si la etiqueta se daña o se pierde, descárguela desde el portal. Vaya a **Información general** y haga clic en **Download shipping label** (Descargar etiqueta de envío) desde la barra de comandos.
2. Entregue el paquete precintado en la ubicación para realizar el envío.  

El servicio Data Box Disk envía una notificación por correo electrónico y actualiza el estado del pedido en Azure Portal.

## <a name="verify-your-data"></a>Verificación de los datos

El tiempo en completar esta operación depende del tamaño de los datos.

1. Cuando Data Box Disk se conecta a la red del centro de datos de Azure, se inicia automáticamente la carga de datos en Azure. 
2. El servicio Azure Data Box le notifica a través de Azure Portal que la copia de datos se ha completado. 
    
    1. Compruebe en los registros de errores si hay errores y tome las medidas adecuadas.
    2. Compruebe que los datos estén en las cuentas de almacenamiento antes de eliminarlos del origen.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este paso tarda de 2 a 3 minutos en completarse.

Para limpiarlos, puede cancelar el pedido de Data Box y, a continuación, eliminarlo.

- Puede cancelar el pedido de Data Box en Azure Portal antes de procesar el pedido. Una vez que el pedido se procesa, no se puede cancelar. El pedido sigue su curso hasta que alcanza la fase de finalización. 

    Para cancelar el pedido, vaya a **Información general** y haga clic en **Cancelar** desde la barra de comandos.  

- Podrá eliminar el pedido una vez que el estado se muestre como **Completed** (Completado) o **Canceled** (Cancelado) en Azure Portal. 

    Para eliminar el pedido, vaya a **Información general** y haga clic en **Eliminar** desde la barra de comandos.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado Azure Data Box Disk para ayudar a importar los datos en Azure. Para más información sobre la administración de Azure Data Box Disk, pase al tutorial siguiente: 

> [!div class="nextstepaction"]
> [Uso de Azure Portal para administrar Azure Data Box Disk](data-box-portal-ui-admin.md)
