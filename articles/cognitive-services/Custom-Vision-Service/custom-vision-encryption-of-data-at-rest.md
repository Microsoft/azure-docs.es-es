---
title: Cifrado de datos en reposo de Custom Vision
titleSuffix: Azure Cognitive Services
description: Microsoft ofrece claves de cifrado administradas por Microsoft y también le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo, encontrará información sobre el cifrado de datos en reposo de Custom Vision y cómo habilitar y administrar CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: 39257419f179bdce8c94f2ddb3a7cd8f5ac2d34f
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077762"
---
# <a name="custom-vision-encryption-of-data-at-rest"></a>Cifrado de datos en reposo de Custom Vision

Custom Vision de Azure cifra automáticamente los datos al guardarlos en la nube. El cifrado de Custom Vision protege los datos y le ayuda a satisfacer los requisitos de cumplimiento normativo y seguridad de la organización.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Las claves que administre el cliente solo serán aquellos recursos disponibles que se hayan creado después del 11 de mayo de 2020. Para usar CMK con Custom Vision, tendrá que crear un nuevo recurso de Custom Vision. Una vez creado el recurso, puede usar Azure Key Vault para configurar la identidad administrada.

## <a name="regional-availability"></a>Disponibilidad regional

Actualmente, las claves administradas por el cliente están disponibles en estas regiones:

* Centro y Sur de EE. UU.
* Oeste de EE. UU. 2
* Este de EE. UU.
* US Gov - Virginia

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una lista completa de los servicios que admiten CMK, consulte [Claves administradas por el cliente para Cognitive Services](../encryption/cognitive-services-encryption-keys-portal.md).
* [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
* [Formulario de solicitud de claves administradas por el cliente de Cognitive Services](https://aka.ms/cogsvc-cmk)
