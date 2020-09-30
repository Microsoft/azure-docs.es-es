---
title: 'Creación de una dirección IP pública: Azure Portal'
description: Aprenda a crear una IP pública en Azure Portal
services: virtual-network
documentationcenter: na
author: blehr
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2020
ms.author: blehr
ms.openlocfilehash: 49a89ee90d28c9c7a3f59424b773ee0f221381e4
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301037"
---
# <a name="quickstart-create-a-public-ip-address-using-the-azure-portal"></a>Inicio rápido: Creación de una dirección IP pública mediante Azure Portal

En este artículo se muestra cómo crear un recurso de dirección IP pública mediante Azure Portal. Para más información sobre los recursos que se pueden asociar, la diferencia entre la SKU básica y estándar y otra información relacionada, consulte [Direcciones IP públicas](https://docs.microsoft.com/azure/virtual-network/public-ip-addresses).  En este ejemplo, nos centraremos solo en las direcciones IPv4. Para más información sobre las direcciones IPv6, consulte [IPv6 para la red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/ipv6-overview).

# <a name="standard-sku---using-zones"></a>[**SKU estándar: uso de zonas**](#tab/option-create-public-ip-standard-zones)

Use los pasos siguientes para crear una dirección IP pública estándar con redundancia de zona denominada **myStandardZRPublicIP**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Crear un recurso**. 
3. En el cuadro de búsqueda, escriba *Dirección IP pública*.
4. En los resultados de la búsqueda, escriba **Dirección IP pública**. A continuación, en la página **Dirección IP pública** seleccione **Crear**.
5. En la página **Crear dirección IP pública**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                       |
    | ---                     | ---                         |
    | Versión de la dirección IP              | Seleccione IPv4                 |    
    | SKU                     | Seleccione **Estándar**.         |
    | Nombre                    | Escriba *myStandardZRPublicIP*          |
    | Asignación de dirección IP   | Tenga en cuenta que se bloqueará como "estática".                                        |
    | Tiempo de espera de inactividad (minutos)  | Deje el valor en 4.        |
    | Etiqueta de nombre DNS          | Deje el valor en blanco.    |
    | Subscription            | Seleccione su suscripción.   |
    | Resource group          | Seleccione **Crear nuevo**, escriba myResourceGroup y seleccione **Aceptar**. |
    | Location                | Seleccione **Este de EE. UU. 2**.      |
    | Zona de disponibilidad       | Seleccione **Con redundancia de zona** o elija una zona específica (consulte la nota siguiente). |

Tenga en cuenta que estas opciones solo son selecciones válidas en regiones con [zonas de disponibilidad](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).  (También puede seleccionar una zona específica en estas regiones, aunque no sea resistente a errores de zona).

# <a name="standard-sku---no-zones"></a>[**SKU estándar: sin zonas**](#tab/option-create-public-ip-standard)

Use los pasos siguientes para crear una dirección IP pública estándar como un recurso que no es de zona denominado **myStandardPublicIP**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Crear un recurso**. 
3. En el cuadro de búsqueda, escriba *Dirección IP pública*.
4. En los resultados de la búsqueda, escriba **Dirección IP pública**. A continuación, en la página **Dirección IP pública** seleccione **Crear**.
5. En la página **Crear dirección IP pública**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                       |
    | ---                     | ---                         |
    | Versión de la dirección IP              | Seleccione IPv4                 |    
    | SKU                     | Seleccione **Estándar**.         |
    | Nombre                    | Escriba *myStandardPublicIP*          |
    | Asignación de dirección IP   | Tenga en cuenta que se bloqueará como "estática".                                        |
    | Tiempo de espera de inactividad (minutos)  | Deje el valor en 4.        |
    | Etiqueta de nombre DNS          | Deje el valor en blanco.    |
    | Subscription            | Seleccione su suscripción.   |
    | Resource group          | Seleccione **Crear nuevo**, escriba myResourceGroup y seleccione **Aceptar**. |
    | Location                | Seleccione **Este de EE. UU. 2**.      |
    | Zona de disponibilidad       | Seleccione **Ninguna zona** (y consulte la nota siguiente) |

Esta selección es válida en todas las regiones y es la selección predeterminada para las direcciones IP públicas estándar en regiones sin [Availability Zones](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-create-public-ip-basic)

Use los pasos siguientes para crear una dirección IP pública estática básica denominada **myBasicPublicIP**.  Las direcciones IP públicas básicas no tienen el concepto de zonas de disponibilidad.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Crear un recurso**. 
3. En el cuadro de búsqueda, escriba *Dirección IP pública*.
4. En los resultados de la búsqueda, escriba **Dirección IP pública**. A continuación, en la página **Dirección IP pública** seleccione **Crear**.
5. En la página **Crear dirección IP pública**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                       |
    | ---                     | ---                         |
    | Versión de la dirección IP              | Seleccione IPv4                 |    
    | SKU                     | Seleccione **Estándar**.         |
    | Nombre                    | Escriba *myBasicPublicIP*          |
    | Asignación de dirección IP   | Elija **Estática** (vea la nota a continuación).                                     |
    | Tiempo de espera de inactividad (minutos)  | Deje el valor en 4.        |
    | Etiqueta de nombre DNS          | Deje el valor en blanco.    |
    | Subscription            | Seleccione su suscripción.   |
    | Resource group          | Seleccione **Crear nuevo**, escriba myResourceGroup y seleccione **Aceptar**. |
    | Location                | Seleccione **Este de EE. UU. 2**.      |

Si es admisible que la dirección IP cambie con el tiempo, se puede seleccionar la asignación de IP **dinámica**.

---

## <a name="additional-information"></a>Información adicional 

Para más información sobre los campos individuales enumerados anteriormente, consulte [Administración de direcciones IP públicas](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address).

## <a name="next-steps"></a>Pasos siguientes
- Asocie una [dirección IP pública a una máquina virtual](https://docs.microsoft.com/azure/virtual-network/associate-public-ip-address-vm#azure-portal).
- Obtenga más información acerca de las [direcciones IP públicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) en Azure.
- Obtenga más información acerca de toda la [configuración de direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).