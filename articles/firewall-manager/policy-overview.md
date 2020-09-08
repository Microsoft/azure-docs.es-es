---
title: Información general sobre la directiva de Azure Firewall Manager
description: Más información sobre las directivas de Azure Firewall Manager
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 34134f2c790851d34db7b5327aa76350d54d137d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075470"
---
# <a name="azure-firewall-manager-policy-overview"></a>Información general sobre la directiva de Azure Firewall Manager

Directiva de firewall es un recurso de Azure que contiene NAT, red y colecciones de reglas de aplicación así como valores de inteligencia sobre amenazas. Es un recurso global que se puede utilizar en varias instancias de Azure Firewall en centros virtuales protegidos y redes virtuales de centros. Las directivas funcionan entre regiones y suscripciones.

![Directiva de Azure Firewall Manager](media/policy-overview/policy-overview.png)

## <a name="policy-creation-and-association"></a>Creación y asociación de una directiva

Una directiva se puede crear y administrar de varias maneras, entre las que se incluyen Azure Portal, API REST, plantillas, Azure PowerShell y CLI.

También puede migrar las reglas existentes de Azure Firewall mediante el portal o mediante Azure PowerShell para crear las directivas. Para más información, consulte [Cómo migrar configuraciones de Azure Firewall a la directiva de Azure Firewall](migrate-to-policy.md). 

Las directivas se pueden asociar a uno o varios centros o redes virtuales. El firewall puede estar en cualquier suscripción asociada a su cuenta y en cualquier región.

## <a name="hierarchical-policies"></a>Directivas jerárquicas

Las nuevas directivas se pueden crear desde cero o heredar de directivas ya existentes. La herencia permite a DevOps crear directivas de firewall locales sobre la directiva base asignada de la organización.

Las directivas creadas con directivas primarias no vacías heredan todas las colecciones de reglas de la directiva primaria. Las colecciones de reglas de red heredadas de una directiva primaria siempre se priorizan por encima de las colecciones de reglas de red definidas como parte de una nueva directiva. La misma lógica también se aplica a las colecciones de reglas de aplicación. Las colecciones de reglas de red siempre se procesan antes que las colecciones de reglas de aplicación independientemente de la herencia.

El modo de inteligencia sobre amenazas también se hereda de la directiva primaria. Puede establecer el modo de inteligencia sobre amenazas en un valor diferente para invalidar este comportamiento, pero no puede desactivarlo. Solo es posible invalidarlo con un valor más estricto. Por ejemplo, si la directiva primaria está establecida en **solo Alerta**, puede configurar esta directiva local para **Alertar y denegar**.

Igual que en el modo de inteligencia sobre amenazas, la lista de permitidos de inteligencia sobre amenazas se hereda de la directiva primaria. La directiva secundaria puede agregar direcciones IP adicionales a la lista de permitidos.

Las colecciones de reglas NAT no se heredan porque son específicas de un firewall determinado.

Mediante la herencia, los cambios en la directiva primaria se aplican automáticamente a las directivas secundarias de firewall asociadas.

## <a name="traditional-rules-and-policies"></a>Reglas y directivas tradicionales

Azure Firewall admite las reglas y las directivas tradicionales. En la siguiente tabla se comparan las directivas y las reglas:


| Asunto | Directiva  | Reglas |
| ------- | ------- | ----- |
|Contains     |Reglas de NAT, red y aplicación, configuración de proxy DNS y DNS personalizado, grupos de direcciones IP y configuración de inteligencia sobre amenazas (incluida la lista de permitidos)|Reglas de NAT, red y aplicación, configuración de proxy DNS y DNS personalizado, grupos de direcciones IP y configuración de inteligencia sobre amenazas (incluida la lista de permitidos)|
|Protege     |Centros virtuales y redes virtuales|Solo redes virtuales|
|Experiencia del portal     |Administración centralizada mediante Firewall Manager|Experiencia de firewall independiente|
|Compatibilidad con varios firewalls     |La directiva de firewall es un recurso independiente que se puede usar entre firewalls.|Exporte e importe manualmente reglas o use soluciones de administración de terceros |
|Precios     |Facturado en función de la asociación del firewall. Consulte [Precios](#pricing).|Gratuito|
|Mecanismos de implementación admitidos     |Portal, API REST, plantillas, Azure PowerShell y CLI|Portal, API REST, plantillas, PowerShell y CLI. |

## <a name="pricing"></a>Precios

Las directivas se facturan en función de las asociaciones del firewall. Una directiva con una o cero asociaciones de firewall es gratuita. Una directiva con varias asociaciones de firewall se factura según una tarifa fija. Para más información, consulte [Precios de Azure Firewall Manager](https://azure.microsoft.com/pricing/details/firewall-manager/).

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo implementar una instancia de Azure Firewall, consulte [Tutorial: Proteja su red en la nube con la versión preliminar de Azure Firewall Manager en Azure Portal](secure-cloud-network.md).
