---
title: Administración central de Azure Firewall
description: Más información sobre la administración central de Azure Firewall Manager
author: vhorne
ms.service: firewall
services: firewall
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: victorh
ms.openlocfilehash: 23a7682d8a64de57db4ff9ae785ada90d4a06944
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084677"
---
# <a name="azure-firewall-central-management"></a>Administración central de Azure Firewall

Si administra varios firewalls, sabe que las reglas de firewall que cambian continuamente dificultan su sincronización. Los equipos centrales de TI necesitan una forma de definir directivas de firewall base y aplicarlas entre varias unidades de negocio. Al mismo tiempo, los equipos de DevOps quieren crear sus propias directivas de firewall derivadas locales para mejorar la agilidad.

Azure Firewall Manager puede ayudarle a resolver estos problemas.


## <a name="azure-firewall-manager"></a>Azure Firewall Manager

Azure Firewall Manager es un servicio de administración de seguridad de la red que proporciona una directiva de seguridad central y la administración de rutas para perímetros de seguridad basados en la nube. Permite a los equipos de TI empresariales definir de forma fácil reglas de nivel de aplicación y red de forma central para el filtrado del tráfico entre varias instancias de Azure Firewall. Puede abarcar diferentes regiones y suscripciones de Azure en arquitecturas de concentrador y radio para la regulación y la protección del tráfico. También proporciona una mejor agilidad de DevOps con las directivas de seguridad de firewall locales derivadas que se implementan en las organizaciones.

### <a name="firewall-policy"></a>Directiva de firewall

Una directiva de firewall es un recurso de Azure que contiene colecciones de reglas de NAT, red y aplicación, así como valores de configuración de inteligencia sobre amenazas. Se trata de un recurso global que se puede usar en varias instancias de Azure Firewall en *centros virtuales protegidos* y *redes virtuales de centro de conectividad*. Se pueden crear directivas desde cero o heredarse de directivas ya existentes. La herencia permite a DevOps crear directivas de firewall locales sobre la directiva base ordenada por la organización. Las directivas funcionan entre regiones y suscripciones.
 
Puede crear una directiva de firewall y asociaciones con Azure Firewall Manager. Sin embargo, también puede crear y administrar una directiva mediante la API REST, plantillas, Azure PowerShell y la CLI. Cuando se crea una directiva, se puede asociar a un firewall de un centro de conectividad WAN virtual, lo que lo convierte en un *centro virtual protegido*, o a un firewall de una red virtual, lo que lo convierte en una *red virtual de centro de conectividad*.

### <a name="pricing"></a>Precios

Las directivas se facturan en función de las asociaciones del firewall. Una directiva con una o cero asociaciones de firewall es gratuita. Una directiva con varias asociaciones de firewall se factura según una tarifa fija. Para más información, consulte [Precios de Azure Firewall Manager](https://azure.microsoft.com/pricing/details/firewall-manager/).

## <a name="azure-firewall-management-partners"></a>Asociados de Azure Firewall Management

Las siguientes soluciones de terceros destacadas admiten la administración central de Azure Firewall con las API REST de Azure estándar. Cada una de estas soluciones tiene sus propias características y funciones únicas:

- [AlgoSec CloudFlow](https://www.algosec.com/azure/) 
- [Barracuda Cloud Security Guardian](https://www.barracuda.com/products/cloudsecurityguardian/for_azure)
- [Tufin Orca](https://www.tufin.com/products/tufin-orca)


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Firewall Manager, consulte [¿Qué es Azure Firewall Manager?](../firewall-manager/overview.md)