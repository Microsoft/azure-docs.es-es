---
title: ¿Qué es Azure DNS?
description: Información general del servicio de hospedaje de DNS en Microsoft Azure. Hospede el dominio en Microsoft Azure.
author: rohinkoul
ms.service: dns
ms.topic: overview
ms.date: 3/21/2019
ms.author: rohink
ms.openlocfilehash: d59f166897674e9dcff8086b8f5c906605155ee3
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924281"
---
# <a name="what-is-azure-dns"></a>¿Qué es Azure DNS?

Azure DNS es un servicio de hospedaje para dominios DNS que ofrece resolución de nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros de DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.

No se puede usar Azure DNS para comprar un nombre de dominio. Por una tarifa anual, puede comprar un nombre de dominio mediante [dominios de App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain#buy-the-domain) o un registrador de nombres de dominio de terceros. Los dominios se pueden hospedar a continuación en Azure DNS para la administración de registros. Para más información, vea [Delegación de un dominio en DNS de Azure](dns-domain-delegation.md).

Las siguientes características se incluyen con Azure DNS.

## <a name="reliability-and-performance"></a>Confiabilidad y rendimiento

Los dominios DNS de DNS de Azure se hospedan en la red global de servidores de nombres DNS de Azure. Azure DNS usa redes de difusión por proximidad. El servidor DNS más próximo disponible responde cada consulta de DNS para proporcionar un mejor rendimiento y alta disponibilidad para el dominio.

## <a name="security"></a>Seguridad

 Azure DNS se basa en Azure Resource Manager, que proporciona características tales como:

* [Control de acceso basado en rol de Azure (Azure RBAC)](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para controlar quién accede a acciones específicas en la organización.

* [Registros de actividad](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview): para supervisar cómo un usuario de su organización modificó un recurso o para encontrar errores al solucionar problemas.

* [Bloqueo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources) para bloquear una suscripción, un grupo de recursos o un recurso. Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen de forma accidental recursos críticos.

Para más información, consulte [Protección de registros y zonas DNS](dns-protect-zones-recordsets.md). 

## <a name="dnssec"></a>DNSSEC

DNS de Azure no es compatible actualmente con DNSSEC. En la mayoría de los casos, la necesidad de DNSSEC se puede reducir sistemáticamente mediante el uso de HTTPS/TLS en sus aplicaciones. Si DNSSEC es un requisito fundamental para sus zonas DNS, puede hospedar dichas zonas en proveedores de hospedaje de DNS de terceros.

## <a name="ease-of-use"></a>Facilidad de uso

 Azure DNS puede administrar registros DNS para los servicios de Azure y también proporciona el servicio de nombres de dominio para los recursos externos. Azure DNS está integrado en Azure Portal y usa las mismas credenciales, la misma facturación y el mismo contrato de soporte técnico que los demás servicios de Azure. 

La facturación de DNS se basa en el número de zonas DNS hospedadas en Azure y en el número de consultas de DNS recibidas. Para más información sobre precios, consulte [Precios de Azure DNS](https://azure.microsoft.com/pricing/details/dns/).

Los dominios y registros se pueden administrar mediante Azure Portal, cmdlets de Azure PowerShell y la CLI de Azure multiplataforma. Las aplicaciones que requieren la administración automática de DNS se pueden integrar con el servicio mediante las API REST y los SDK.

## <a name="customizable-virtual-networks-with-private-domains"></a>Redes virtuales personalizables con dominios privados

Azure DNS es compatible con dominios DNS privados. Esta característica permite usar sus propios nombres de dominio personalizados en sus redes virtuales privadas en lugar de los nombres proporcionados por Azure disponibles actualmente.

Para más información, consulte [Uso de Azure DNS para dominios privados](private-dns-overview.md).

## <a name="alias-records"></a>Registros de alias

Azure DNS admite conjuntos de registros de alias. Puede usar un conjunto de registros de alias que haga referencia a un recurso de Azure, como una dirección IP pública de Azure, un perfil de Azure Traffic Manager o un punto de conexión de Azure Content Delivery Network (CDN). Si cambia la dirección IP del recurso subyacente, el conjunto de registros de alias se actualiza sin problemas durante la resolución DNS. El conjunto de registros de alias apunta a la instancia de servicio, y la instancia de servicio está asociada con una dirección IP.

Además, ahora puede apuntar el vértice o dominio simple a un perfil de Traffic Manager o punto de conexión de CDN mediante un registro de alias. Un ejemplo es contoso.com.

Para obtener más información, consulte [Overview of Azure DNS alias records](dns-alias.md) (Introducción a los registros de alias de Azure DNS).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre zonas y registros DNS, consulte la [Introducción a las zonas y los registros DNS](dns-zones-records.md).

* Para más información sobre cómo crear una zona en Azure DNS, consulte [Creación de una zona DNS](./dns-getstarted-create-dnszone-portal.md).

* Para ver las preguntas más frecuentes sobre DNS de Azure, vea [Azure DNS FAQ (P+F de DNS de Azure)](dns-faq.md).
