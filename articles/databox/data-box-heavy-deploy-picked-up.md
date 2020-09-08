---
title: Tutorial para devolver Azure Data Box Heavy | Microsoft Docs
description: En este tutorial, aprenderá a devolver un dispositivo Azure Data Box Heavy, lo que incluye la preparación del envío, el envío de Data Box Heavy, la comprobación de la carga de datos y la eliminación de datos.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: fa4df7119763e835cf62e67512105b748c7e2265
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921017"
---
::: zone target = "docs"

# <a name="tutorial-return-azure-data-box-heavy-and-verify-data-upload-to-azure"></a>Tutorial: Devolución de Azure Data Box Heavy y comprobación de la carga de datos en Azure

::: zone-end

::: zone target = "chromeless"

## <a name="return-azure-data-box-heavy-and-verify-data-upload-to-azure"></a>Devolución de Azure Data Box Heavy y comprobación de la carga de datos en Azure

::: zone-end

::: zone target = "docs"

En este tutorial se describe cómo devolver Azure Data Box Heavy y comprobar los datos cargados en Azure.

En este tutorial, aprenderá sobre temas como:

> [!div class="checklist"]
> * Prerrequisitos
> * Preparación para el envío
> * Envío de Data Box Heavy a Microsoft
> * Comprobación de la carga de datos en Azure
> * Eliminación de datos de Data Box Heavy

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que:

- Ha completado el [Tutorial: Copia de datos a Azure Data Box y comprobación de](data-box-heavy-deploy-copy-data.md).
- Los trabajos de copia están completos. Preparación para el envío no se puede ejecutar mientras que los trabajos de copia están en curso.


## <a name="prepare-to-ship"></a>Preparación para el envío

[!INCLUDE [data-box-heavy-prepare-to-ship](../../includes/data-box-heavy-prepare-to-ship.md)]

::: zone-end

::: zone target = "chromeless"

## <a name="prepare-to-ship"></a>Preparación para el envío

Antes de prepararse para enviar, asegúrese de que los trabajos de copia se han completado.

1. Vaya a la página Preparación para el envío de la interfaz de usuario web local y comience la preparación del envío.
2. Desactive el dispositivo desde la interfaz de usuario web local. Quite los cables del dispositivo.

Ya está listo para enviar el dispositivo.

::: zone-end

## <a name="ship-data-box-heavy-back"></a>Devolución de Data Box Heavy

1. Asegúrese de que el dispositivo está apagado y de que se quitado todos los cables. Enrolle y coloque de forma segura los cuatro cables de alimentación en la bandeja a la que puede acceder desde la parte posterior del dispositivo.
2. El dispositivo se envía con la tarifa LTL mediante FedEx en los Estados Unidos y DHL en Europa.

    1. Póngase en contacto con [Operaciones de Data Box](mailto:DataBoxOps@microsoft.com) para informar sobre la recogida y obtener la etiqueta de envío de devolución.
    2. Llame al número local del transportista para programar la recogida.
    3. Asegúrese de que la etiqueta de envío se muestra de forma destacada en el exterior del envío.
    4. Asegúrese de que se quitan las etiquetas del envío anterior del dispositivo.
3. Una vez que el transportista recoge y examina el dispositivo Data Box Heavy, el estado del pedido en el portal se actualiza a **Picked up** (Recogido). También se muestra un identificador de seguimiento.

::: zone target = "docs"

## <a name="verify-data-upload-to-azure"></a>Comprobación de la carga de datos en Azure

Cuando Microsoft recibe y examina el dispositivo, el estado del pedido se actualiza a **Received** (Recibido). El dispositivo, a continuación, se somete a una verificación física de daños o signos de manipulación.

Una vez completada la comprobación, el dispositivo Data Box Heavy se conecta a la red del centro de datos de Azure. La copia de datos se inicia automáticamente. Dependiendo del tamaño de los datos, la operación de copia puede tardar horas o días en completarse. Puede supervisar el progreso del trabajo de copia en el portal.

Una vez finalizada la copia, el estado del pedido se actualiza a **Completed** (Completado).

Compruebe que los datos se han cargado en Azure antes de eliminarlos del origen. Los datos pueden estar en:

- Sus cuentas de Azure Storage. Al copiar los datos en Data Box, dependiendo del tipo, estos se cargan en una de las siguientes rutas de acceso de la cuenta de Azure Storage.

  - Para los blobs en bloques y los blobs en páginas: `https://<storage_account_name>.blob.core.windows.net/<containername>/files/a.txt`
  - Para Azure Files: `https://<storage_account_name>.file.core.windows.net/<sharename>/files/a.txt`

    Como alternativa, puede ir a su cuenta de almacenamiento de Azure en Azure Portal e ir desde allí.

- Sus grupos de recursos de disco administrados. Al crear discos administrados, los discos duros virtuales se cargan como blobs en páginas y se convierten en discos administrados. Los discos administrados se conectan a los grupos de recursos especificados en el momento de creación del pedido. 

    - Si la copia en los discos administrados de Azure se realizó correctamente, puede ir a **Detalles del pedido** en Azure Portal y tomar nota de los grupos de recursos especificados para los discos administrados.

        ![Identificación de grupos de recursos de disco administrados](media/data-box-deploy-copy-data-from-vhds/order-details-managed-disk-resource-groups.png)

        Vaya al grupo de recursos anotado y busque los discos administrados.

        ![Disco administrado conectado a grupos de recursos](media/data-box-deploy-copy-data-from-vhds/managed-disks-resource-group.png)

    - Si copió un VHDX o un disco duro virtual dinámico o de diferenciación, el VHD o VHDX se carga en la cuenta de almacenamiento provisional como si fuera un blob en páginas, pero se producen un error en la conversión del disco duro virtual a disco administrado. Vaya a su almacenamiento provisional **Cuenta de almacenamiento > Blobs** y seleccione el contenedor adecuado (SSD estándar, HDD estándar o SSD Premium). Los discos duros virtuales se cargan como blobs en páginas en la cuenta de almacenamiento provisional.
    
::: zone-end

::: zone target = "chromeless"

## <a name="verify-data-upload-to-azure"></a>Comprobación de la carga de datos en Azure

Cuando el dispositivo Data Box Heavy se conecta a la red del centro de datos de Azure, se inicia automáticamente la carga de datos en Azure. El servicio Data Box le notifica a través de Azure Portal que la copia de datos se ha completado.

- Compruebe en los registros de errores si hay errores y tome las medidas adecuadas.
- Compruebe que los datos estén en las cuentas de almacenamiento antes de eliminarlos del origen.

::: zone-end

## <a name="erasure-of-data-from-data-box-heavy"></a>Eliminación de datos de Data Box Heavy
 
Una vez que se completa la carga en Azure, Data Box elimina los datos de los discos según las [directrices de la revisión 1 de NIST SP 800-88](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi). Una vez completada la eliminación, puede [descargar el historial de pedidos](data-box-portal-admin.md#download-order-history).

::: zone target = "docs"

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box, como:

> [!div class="checklist"]
> * Prerrequisitos
> * Preparación para el envío
> * Envío de Data Box Heavy a Microsoft
> * Comprobación de la carga de datos en Azure
> * Eliminación de datos de Data Box Heavy

Pase al siguiente artículo para obtener información sobre cómo administrar Data Box Heavy a través de la interfaz de usuario web local.

> [!div class="nextstepaction"]
> [Uso de la interfaz de usuario web local para administrar Azure Data Box](./data-box-local-web-ui-admin.md)

::: zone-end


