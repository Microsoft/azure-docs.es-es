---
title: Qué es la característica de registro automático de zonas privadas de Azure DNS
description: Información general de la característica de registro automático de zonas privadas de Azure DNS
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 9/24/2019
ms.author: rohink
ms.openlocfilehash: 9da94f80f9a9c1b3cba7b8e3ac4fef7e717918c9
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042758"
---
# <a name="what-is-the-autoregistration-feature-of-azure-dns-private-zones"></a>Qué es la característica de registro automático de zonas privadas de Azure DNS

La característica de registro automático de zonas privadas de Azure DNS aborda el problema de la administración de registros DNS para las máquinas virtuales implementadas en una red virtual. Cuando [vincula una red virtual](./private-dns-virtual-network-links.md) con una zona DNS privada y habilita el registro automático para todas las máquinas virtuales, los registros DNS de las máquinas virtuales implementadas en la red virtual se crean automáticamente en la zona DNS privada. Además de reenviar registros búsqueda directa (registros A), también se crean automáticamente registros de búsqueda inversa (registros PTR) para las máquinas virtuales.
Si agrega más máquinas virtuales a la red virtual, los registros DNS de estas máquinas virtuales también se crean automáticamente en la zona DNS privada vinculada.

Al eliminar una máquina virtual, sus registros DNS se eliminan automáticamente de la zona DNS privada.

Puede habilitar el registro automático si selecciona la opción "Habilitar el registro automático" durante la creación de un vínculo de red virtual.

![Habilitación del registro automático](./media/privatedns-concepts/enable-autoregistration.png)

## <a name="restrictions"></a>Restricciones

* El registro automático solo funciona para las máquinas virtuales. En el caso de todos los demás recursos, como los equilibradores de carga internos, etc., puede crear registros DNS manualmente en la zona DNS privada vinculada a la red virtual.
* Los registros DNS se crean automáticamente solo para la NIC de la máquina virtual principal. Si las máquinas virtuales tienen más de una NIC, puede crear manualmente los registros DNS para las demás interfaces de red.
* Los registros DNS se crean automáticamente solo si la NIC de la máquina virtual principal usa DHCP. Si se configuran direcciones IP estáticas (por ejemplo, para usar [varias direcciones IP en Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#os-config)), el registro automático no crea registros para esa máquina virtual.
* No se admite el registro automático para IPv6 (registros AAAA).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, como el comportamiento específico que se puede esperar con determinadas clases de operaciones, consulte [Preguntas frecuentes de DNS privado](./dns-faq-private.md).
