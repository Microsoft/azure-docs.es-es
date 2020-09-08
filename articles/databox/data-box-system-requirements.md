---
title: Requisitos del sistema de Microsoft Azure Data Box | Microsoft Docs
description: Obtenga información sobre requisitos del sistema importantes tanto de Azure Data Box como de los clientes que se conectan a Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/20/2020
ms.author: alkohli
ms.openlocfilehash: 71f499e1cdd7ccf22d90b21ce04299798978a828
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926610"
---
# <a name="azure-data-box-system-requirements"></a>Requisitos del sistema de Azure Data Box

En este artículo se describen los requisitos del sistema importantes tanto de Microsoft Azure Data Box como de los clientes de que se conectan a Data Box. Se recomienda leer detenidamente la siguiente información antes de implementar Data Box y volver a consultarla según sea necesario durante la implementación y el posterior funcionamiento.

Los requisitos del sistema incluyen:

* **Requisitos de software de los hosts que se conectan a Data Box**: describe las plataformas compatibles, los exploradores de la interfaz de usuario web local, los clientes SMB y los requisitos adicionales de los hosts que se pueden conectar a Data Box.
* **Requisitos de red de Data Box**: proporciona información acerca de los requisitos de red para el funcionamiento del óptimo de Data Box.


## <a name="software-requirements"></a>Requisitos de software

Los requisitos de software incluyen información acerca de los sistemas operativos y exploradores compatibles con la interfaz de usuario web local y los clientes SMB.

### <a name="supported-operating-systems-for-clients"></a>Sistemas operativos compatibles para clientes

[!INCLUDE [data-box-supported-os-clients](../../includes/data-box-supported-os-clients.md)]


### <a name="supported-filesystems-for-linux-clients"></a>Sistemas de archivos compatibles para clientes Linux

[!INCLUDE [data-box-supported-file-systems-clients](../../includes/data-box-supported-file-systems-clients.md)]


> [!IMPORTANT] 
> No se admite la conexión a recursos compartidos de Data Box a través de REST para los pedidos de exportación. 

### <a name="supported-storage-accounts"></a>Cuentas de almacenamiento admitidas

[!INCLUDE [data-box-supported-storage-accounts](../../includes/data-box-supported-storage-accounts.md)]


### <a name="supported-storage-types"></a>Tipos de almacenamiento compatibles

[!INCLUDE [data-box-supported-storage-types](../../includes/data-box-supported-storage-types.md)]

### <a name="supported-web-browsers"></a>Exploradores web compatibles

[!INCLUDE [data-box-supported-web-browsers](../../includes/data-box-supported-web-browsers.md)]

## <a name="networking-requirements"></a>Requisitos de red

El centro de datos debe tener una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. Si no hay disponible una conexión 10 GbE, se puede usar un vínculo de datos de 1 GbE para copiar datos, pero la velocidad de copia resultará afectada.

### <a name="port-requirements"></a>Requisitos de puerto

En la siguiente tabla se enumeran los puertos que deben abrirse en el firewall para permitir el tráfico de SMB o NFS. En esta tabla, *dentro* o *entrante* hace referencia a la dirección desde la que el cliente entrante solicita acceso al dispositivo. *Fuera* o *saliente* hace referencia a la dirección en la que el dispositivo de Data Box envía datos externamente después de la implementación: por ejemplo, saliente a Internet.

[!INCLUDE [data-box-port-requirements](../../includes/data-box-port-requirements.md)]


## <a name="next-steps"></a>Pasos siguientes

* [Implementación de Azure Data Box](data-box-deploy-ordered.md)
