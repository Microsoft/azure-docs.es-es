---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 07/13/2020
ms.author: jingwang
ms.openlocfilehash: 6c348b3dd0005eeab154aa2d74abc617cbd1d0cb
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529402"
---
<!--
    Separate the generic requirement on Self-hosted Integration Runtime set-up from connector articles.
-->
Si el almacén de datos se encuentra en una red local, una red virtual de Azure o una nube privada virtual de Amazon, debe configurar un [entorno de ejecución de integración autohospedado](../articles/data-factory/create-self-hosted-integration-runtime.md) para conectarse a él.

Si el almacén de datos es un servicio de datos en la nube administrado, puede usar Azure Integration Runtime. Si el acceso está restringido a las direcciones IP que están en la lista de permitidos en las reglas de firewall, puede optar por agregar [direcciones IP de Azure Integration Runtime](../articles/data-factory/azure-integration-runtime-ip-addresses.md) a la lista de permitidos. 

Consulte [Estrategias de acceso a datos](../articles/data-factory/data-access-strategies.md) para más información sobre los mecanismos de seguridad de red y las opciones que admite Data Factory.
