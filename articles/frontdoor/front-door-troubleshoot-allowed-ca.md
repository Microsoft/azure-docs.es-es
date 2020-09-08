---
title: CA permitidas para habilitar HTTPS personalizado en Azure Front Door
description: Si utiliza su propio certificado para habilitar HTTPS en un dominio personalizado, debe utilizar una entidad de certificación permitida (CA) para crearlo.
services: frontdoor
documentationcenter: ''
author: duongau
ms.assetid: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/23/2018
ms.author: duau
ms.openlocfilehash: 7bdef37561687b49b030d8237472c0d35f945c13
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399130"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https-on-azure-front-door"></a>Entidades de certificación permitidas para habilitar HTTPS personalizado en Azure Front Door

Para un dominio personalizado de Azure Front Door, al [habilitar la característica HTTPS mediante su propio certificado](front-door-custom-domain-https.md?tabs=option-2-enable-https-with-your-own-certificate), debe usar una entidad de certificación (CA) permitida para crear el certificado TLS/SSL. De lo contrario, si usa una entidad de certificación no permitida o un certificado autofirmado, la solicitud se rechazará.

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]
