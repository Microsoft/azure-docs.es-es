---
title: ¿Qué es DNS privado de Azure?
description: Este artículo comienza con información general acerca del servicio de hospedaje de DNS privado en Microsoft Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: overview
ms.date: 6/12/2019
ms.author: rohink
ms.openlocfilehash: 540c4d6f2dde64cef9b5795d635ac6259bfc69c3
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985638"
---
# <a name="what-is-azure-private-dns"></a>¿Qué es DNS privado de Azure?

El Sistema de nombres de dominio, o DNS, es responsable de traducir o resolver el nombre del servicio en su dirección IP.  Azure DNS es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Además de admitir los dominios DNS con conexión a Internet, Azure DNS ahora también admite zonas DNS privadas.

DNS privado de Azure proporciona un servicio DNS confiable y seguro para administrar y resolver los nombres de dominio en una red virtual sin necesidad de agregar una solución DNS personalizada. Al utilizar zonas DNS privadas, puede usar sus propios nombres de dominio personalizados en lugar de los nombres proporcionados por Azure que están disponibles actualmente. El uso de los nombres de dominio personalizados facilitan la adaptación de la arquitectura de red virtual para satisfacer mejor las necesidades de su organización. Proporciona la resolución de nombres para las máquinas virtuales (VM) dentro de una red virtual y entre redes virtuales. Además, puede configurar nombres de zonas con una vista de horizonte dividido que permite que una zona DNS privada y otra pública compartan el nombre.

Para resolver los registros de una zona DNS privada de la red virtual, debe vincular esta con la zona. Las redes virtuales vinculadas tienen acceso completo y pueden resolver todos los registros DNS que se publican en la zona privada. Además, también puede habilitar el registro automático en un vínculo de red virtual. Si habilita el registro automático en un vínculo de red virtual, los registros DNS para las máquinas virtuales en esa red virtual se registran en la zona privada. Cuando se habilita el registro automático, Azure DNS también actualiza los registros de zona cuando se crea una máquina virtual, se cambia su dirección IP o se elimina.

![Información general de DNS](./media/private-dns-overview/scenario.png)

> [!NOTE]
> Como procedimiento recomendado, no use un dominio *.local* para la zona DNS privada. No todos los sistemas operativos admiten esto.

## <a name="benefits"></a>Ventajas

DNS privado de Azure proporciona las ventajas siguientes:

* **Eliminación de la necesidad de contar con soluciones DNS personalizadas**. Anteriormente, muchos clientes crearon soluciones DNS personalizadas para administrar zonas DNS en su red virtual. Ahora puede administrar las zonas DNS con la infraestructura nativa de Azure, lo que evita tener que crear y administrar soluciones DNS personalizadas.

* **Uso de todos los tipos de registros de DNS comunes**. Azure DNS admite los registros A, AAAA, CNAME, MX, PTR, SOA, SRV y TXT.

* **Administración automática de registros de nombres de host**. Además de hospedar los registros de DNS personalizados, Azure mantiene automáticamente los registros de nombres de host de las máquinas virtuales en las redes virtuales especificadas. En este escenario, puede optimizar los nombres de dominio usados sin necesidad de crear soluciones DNS personalizadas ni de modificar aplicaciones.

* **Resolución de nombres de host entre redes virtuales**. A diferencia de los nombres de host proporcionados por Azure, las zonas DNS privadas pueden compartirse entre redes virtuales. Esta funcionalidad simplifica los escenarios de detección de servicios y de distintas redes, como el emparejamiento de red virtual.

* **Experiencia del usuario y herramientas familiares**. Para reducir la curva de aprendizaje, este servicio usa herramientas de Azure DNS establecidas (Azure Portal, Azure PowerShell, la CLI de Azure, plantillas de Azure Resource Manager y la API REST).

* **Compatibilidad con DNS de horizonte dividido**. Con Azure DNS, puede crear zonas con el mismo nombre que se resuelve en diferentes respuestas desde dentro de una red virtual y desde la red pública de Internet. Un escenario típico de DNS de horizonte dividido es proporcionar una versión dedicada de un servicio para usarla dentro de la red virtual.

* **Disponibilidad en todas las regiones de Azure**. La característica de zonas privadas de Azure DNS está disponible en todas las regiones de Azure de la nube pública de Azure.

## <a name="capabilities"></a>Capacidades

Azure DNS ofrece las funcionalidades siguientes:

* **Registro automático de máquinas virtuales desde una única red virtual vinculada a una zona privada con registro automático habilitado**. Las máquinas virtuales se registran (agregan) a la zona privada como registros D que apuntan a sus direcciones IP privadas. Cuando se elimina una máquina virtual de un vínculo de red virtual con el registro automático habilitado, Azure DNS también quita automáticamente el registro DNS correspondiente de la zona privada vinculada.

* **Se admite la resolución DNS de reenvío entre redes virtuales vinculadas a la zona privada**. Para la resolución DNS entre redes virtuales, no hay ninguna dependencia explícita de que las redes virtuales estén emparejadas unas con otras. Sin embargo, es posible que desee emparejar redes virtuales en otras situaciones (por ejemplo, tráfico HTTP).

* **Se admite la búsqueda inversa de DNS dentro del ámbito de la red virtual**. La búsqueda inversa de DNS de una dirección IP privada dentro de la red virtual asignada a una zona privada devuelve el FQDN que incluye el nombre de host/registro, así como el nombre de zona como sufijo.

## <a name="other-considerations"></a>Otras consideraciones

Azure DNS tiene las siguientes limitaciones:

* Una red virtual concreta se puede vincular a una sola zona privada como si se habilitara el registro automático de los registros DNS de una red virtual. Sin embargo, puede vincular varias redes virtuales a una sola zona DNS.
* El DNS inverso solo funciona en el espacio de direcciones IP privadas de la red virtual vinculada.
* El DNS inverso de una dirección IP privada de una red virtual vinculada devuelve *internal.cloudapp.net* como sufijo predeterminado para la máquina virtual. En el caso de las redes virtuales vinculadas a una zona privada con el registro automático habilitado, el DNS inverso de una dirección IP privada devuelve dos FQDN: uno con el valor predeterminado de sufijo *internal.cloudapp.net* y otro con el sufijo de la zona privada.
* El reenvío condicional no se admite actualmente de forma nativa. Para habilitar la resolución entre Azure y las redes locales, consulte [Resolución de nombres para las máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).
 
## <a name="pricing"></a>Precios

Para obtener información de precios, consulte [Precios de Azure DNS](https://azure.microsoft.com/pricing/details/dns/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, como el comportamiento específico que se puede esperar con determinadas clases de operaciones, consulte [Preguntas frecuentes de DNS privado](./dns-faq-private.md).

* Visite [Información general sobre zonas y registros de DNS](dns-zones-records.md) para obtener información sobre zonas y registros DNS.

* Obtenga información sobre las demás [funcionalidades de red](../networking/networking-overview.md) clave de Azure.
