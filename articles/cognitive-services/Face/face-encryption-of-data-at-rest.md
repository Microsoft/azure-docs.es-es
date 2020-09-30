---
title: Cifrado de datos en reposo del servicio Face
titleSuffix: Azure Cognitive Services
description: Microsoft ofrece claves de cifrado administradas por Microsoft y también le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo se trata el cifrado de datos en reposo de Face y cómo habilitar y administrar las CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: eab90fc2cb30ae8e9f1c19bdbefc6fbc88c32f76
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079275"
---
# <a name="face-service-encryption-of-data-at-rest"></a>Cifrado de datos en reposo del servicio Face

El servicio Face cifra automáticamente los datos al guardarlos en la nube. El cifrado del servicio Face protege los datos y le ayuda a satisfacer los requisitos de cumplimiento y de seguridad de la organización.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Las claves administradas por el cliente solo están disponibles en el plan de tarifa E0. Para solicitar la capacidad de usar claves administradas por el cliente, rellene y envíe el [formulario de solicitud de claves administradas por el cliente del servicio Face](https://aka.ms/cogsvc-cmk). Tardará de 3 a 5 días hábiles aproximadamente en recibir una respuesta sobre el estado de la solicitud. En función de la demanda, es posible que se coloque en una cola y se apruebe a medida que haya espacio disponible. Una vez recibida la aprobación para usar CMK con el servicio Face, deberá crear un recurso de Face y seleccionar el plan de tarifa E0. Después de crear el recurso de Face con el plan de tarifa E0, puede usar Azure Key Vault para configurar la identidad administrada.

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una lista completa de los servicios que admiten CMK, consulte [Claves administradas por el cliente para Cognitive Services](../encryption/cognitive-services-encryption-keys-portal.md).
* [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
* [Formulario de solicitud de claves administradas por el cliente de Cognitive Services](https://aka.ms/cogsvc-cmk)
