---
title: Filtrado de FQDN de Azure Firewall en reglas de red (versión preliminar)
description: Cómo usar el filtrado de FQDN de Azure Firewall en reglas de red
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 07/30/2020
ms.author: victorh
ms.openlocfilehash: 6e90a8bc0998b43a84658958215e4b7977f8fdd0
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461313"
---
# <a name="use-fqdn-filtering-in-network-rules-preview"></a>Uso del filtrado de FQDN en reglas de red (versión preliminar)

> [!IMPORTANT]
> El filtrado de FQDN en las reglas de red se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Un nombre de dominio completo (FQDN) representa un nombre de dominio de un host o una o varias direcciones IP. Puede usar FQDN en reglas de red basadas en la resolución DNS en la directiva de firewall y Azure Firewall. Esta funcionalidad permite filtrar el tráfico saliente con cualquier protocolo TCP/UDP (incluidos NTP, SSH, RDP y mucho más). Debe habilitar el proxy DNS para usar FQDN en sus reglas de red. Para obtener más información, consulte [Configuración DNS de Azure Firewall (versión preliminar)](dns-settings.md).

## <a name="how-it-works"></a>Funcionamiento

Una vez que defina qué servidor DNS necesita su organización (Azure DNS o su propio DNS personalizado), Azure Firewall traduce el FQDN a una o varias direcciones IP basadas en el servidor DNS seleccionado. Esta traducción se produce para el procesamiento de reglas de red y de aplicación.

¿Cuál es la diferencia entre usar nombres de dominio en reglas de aplicación y usarlos en reglas de red? 

- El filtrado de FQDN en reglas de aplicación para HTTP/S y MSSQL se basa en un proxy transparente de nivel de aplicación y el encabezado SNI. Como tal, puede distinguir entre dos FQDN que se resuelven en la misma dirección IP. Este no es el caso del filtrado de FQDN en reglas de red. Use siempre reglas de aplicación cuando sea posible.
- En las reglas de aplicación, puede usar HTTP/S y MSSQL como los protocolos seleccionados. En las reglas de red, puede usar cualquier protocolo TCP/UDP con sus FQDN de destino.

## <a name="next-steps"></a>Pasos siguientes

[Configuración DNS de Azure Firewall](dns-settings.md)
