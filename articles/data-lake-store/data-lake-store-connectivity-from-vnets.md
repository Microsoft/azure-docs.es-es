---
title: Conexión a Azure Data Lake Storage Gen1 desde redes virtuales | Microsoft Docs
description: Obtenga información acerca de cómo habilitar el acceso a Azure Data Lake Storage Gen1 de máquinas virtuales de Azure que tienen acceso restringido a los recursos.
services: data-lake-store,data-catalog
documentationcenter: ''
author: esung22
manager: mtillman
editor: cgronlun
ms.assetid: 683fcfdc-cf93-46c3-b2d2-5cb79f5e9ea5
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 01/31/2018
ms.author: elsung
ms.openlocfilehash: 0fa836ea31793d9177ad6e838ddea1516bf51733
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191402"
---
# <a name="access-azure-data-lake-storage-gen1-from-vms-within-an-azure-vnet"></a>Acceso a Azure Data Lake Storage Gen1 desde máquinas virtuales de una red virtual de Azure
Azure Data Lake Storage Gen1 es un servicio de PaaS que se ejecuta en direcciones IP de Internet pública. Normalmente, cualquier servidor que puede conectarse a la Internet pública también puede conectarse a los puntos de conexión de Azure Data Lake Storage Gen1. De forma predeterminada, todas las máquinas virtuales que se encuentran en redes virtuales de Azure pueden obtener acceso a Internet y, por tanto, a Azure Data Lake Storage Gen1. Sin embargo, se puede configurar que las máquinas virtuales de una red virtual no tengan acceso a Internet. Estas máquinas virtuales tampoco pueden tener acceso a Azure Data Lake Storage Gen1. Puede bloquear el acceso a la Internet pública a las máquinas virtuales de las redes virtuales de Azure con cualquiera de los siguientes enfoques:

* Configurando grupos de seguridad de red (NSG)
* Configurando rutas definidas por el usuario (UDR)
* Intercambiando rutas a través de BGP (protocolo de enrutamiento dinámico de normalización industrial) cuando se usa ExpressRoute para bloquear el acceso a Internet

En este artículo, aprenderá a habilitar el acceso a Azure Data Lake Storage Gen1 en las máquinas virtuales de Azure cuyo acceso a los recursos se ha restringido mediante uno de los tres métodos mencionados anteriormente.

## <a name="enabling-connectivity-to-azure-data-lake-storage-gen1-from-vms-with-restricted-connectivity"></a>Habilitación de la conectividad a Azure Data Lake Storage Gen1 desde máquinas virtuales con conectividad restringida
Para obtener acceso a Azure Data Lake Storage Gen1 desde este tipo de máquinas virtuales, debe configurarlas para que tengan acceso a la dirección IP de la región en la que está disponible la cuenta de Azure Data Lake Storage Gen1. Puede identificar las direcciones IP de las cuentas de Data Lake Storage Gen1 resolviendo los nombres DNS de las cuentas (`<account>.azuredatalakestore.net`). Para resolver los nombres DNS de las cuentas, puede usar herramientas como **nslookup**. Abra un símbolo del sistema en el equipo y ejecute el siguiente comando:

```console
nslookup mydatastore.azuredatalakestore.net
```

La salida debe ser similar a la siguiente. El valor de la propiedad **Address** es la dirección IP asociada con la cuenta de Data Lake Storage Gen1.

```output
Non-authoritative answer:
Name:    1434ceb1-3a4b-4bc0-9c69-a0823fd69bba-mydatastore.projectcabostore.net
Address:  104.44.88.112
Aliases:  mydatastore.azuredatalakestore.net
```


### <a name="enabling-connectivity-from-vms-restricted-by-using-nsg"></a>Habilitación de la conectividad en las máquinas virtuales restringidas mediante NSG
Cuando una regla de NSG se usa para bloquear el acceso a Internet, puede crear otro NSG que permita el acceso a la dirección IP de Data Lake Storage Gen1. Para más información sobre las reglas de NSG, consulte la [introducción a los grupos de seguridad de red](../virtual-network/security-overview.md). Para instrucciones sobre cómo crear NSG, consulte el artículo sobre la [creación de un grupo de seguridad de red](../virtual-network/tutorial-filter-network-traffic.md).

### <a name="enabling-connectivity-from-vms-restricted-by-using-udr-or-expressroute"></a>Habilitación de la conectividad en las máquinas virtuales restringidas mediante UDR o ExpressRoute
Cuando se utilizan rutas, UDR o rutas intercambiables a través de BGP, para bloquear el acceso a Internet, debe configurarse una ruta especial para que las máquinas virtuales de estas subredes puedan tener acceso a los puntos de conexión de Data Lake Storage Gen1. Para más información, consulte la [introducción a las rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md). Para obtener instrucciones sobre cómo crear UDR, consulte el artículo sobre cómo [crear UDR en Resource Manager](../virtual-network/tutorial-create-route-table-powershell.md).

### <a name="enabling-connectivity-from-vms-restricted-by-using-expressroute"></a>Habilitación de la conectividad en las máquinas virtuales restringidas mediante ExpressRoute
Cuando se configura un circuito ExpressRoute, los servidores locales pueden tener acceso a Data Lake Storage Gen1 a través de pares públicos. Para obtener más información sobre cómo configurar ExpressRoute para pares públicos, consulte las [preguntas más frecuentes de ExpressRoute](../expressroute/expressroute-faqs.md).

## <a name="see-also"></a>Consulte también
* [Introducción a Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Protección de los datos almacenados en Azure Data Lake Storage Gen1](data-lake-store-security-overview.md)

