---
title: Cifrado de datos en reposo del servicio Form Recognizer
titleSuffix: Azure Cognitive Services
description: Microsoft ofrece claves de cifrado administradas por Microsoft y también le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo se trata el cifrado de datos en reposo de Form Recognizer y cómo habilitar y administrar las CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 07/10/2020
ms.author: egeaney
ms.openlocfilehash: cf2a6a96517d1c3d63989bc511957019f078cd42
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090882"
---
# <a name="form-recognizer-encryption-of-data-at-rest"></a>Cifrado de datos en reposo de Form Recognizer

Form Recognizer de Azure cifra automáticamente los datos al guardarlos en la nube. El cifrado de Form Recognizer protege los datos y le ayuda a satisfacer los requisitos de cumplimiento normativo y de seguridad de la organización.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Las claves que administre el cliente solo serán aquellos recursos disponibles que se hayan creado después del 11 de mayo de 2020. Para usar CMK con Form Recognizer, deberá crear un nuevo recurso de Form Recognizer. Una vez creado el recurso, puede usar Azure Key Vault para configurar la identidad administrada.

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Formulario de solicitud de claves que administra el cliente de Form Recognizer](https://aka.ms/cogsvc-cmk)
* [Más información sobre Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)