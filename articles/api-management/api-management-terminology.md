---
title: Terminología de API Management de Azure | Microsoft Docs
description: Este artículo proporciona definiciones para los términos que son específicos de API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 10/11/2017
ms.author: apimpm
ms.openlocfilehash: 5bc76d2526c5585071a240af36b8a31e3de5708f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024935"
---
# <a name="azure-api-management-terminology"></a>Terminología de Azure API Management

Este artículo proporciona definiciones para los términos que son específicos de API Management (APIM).

## <a name="term-definitions"></a>Definiciones de términos

* **API de back-end**: servicio HTTP que implementa la API y sus operaciones. 
* **API de front-end**/**API APIM**: una API de APIM no hospeda las API, crea fachadas para las API con el fin de personalizarlas según sus necesidades sin tocar la API de back-end. Para obtener más información, consulte [Importación y publicación de la primera API](import-and-publish.md).
* **Producto de APIM** producto que contiene una o varias API, así como una cuota de uso y los términos de uso. Puede incluir varias API y ofrecerlas a los desarrolladores mediante el Portal para desarrolladores. Para obtener más información, consulte [Creación y publicación de un producto](api-management-howto-add-products.md).
* **Operación de API de APIM**: cada API representa un conjunto de operaciones disponibles para los desarrolladores. Cada API de APIM contiene una referencia a un servicio back-end que implementa la API y sus operaciones se asignan a las operaciones implementadas por dicho servicio. Para obtener más información, consulte [Simulación de respuestas de API](mock-api-responses.md).
* **Versión**: habrá quienes quieran publicar características de la API nuevas o diferentes para algunos usuarios, mientras que otros preferirán quedarse con la API que actualmente les vale. Para obtener más información, consulte [Publicación de varias versiones de la API](api-management-get-started-publish-versions.md).
* **Revisión**: cuando la API esté lista y los desarrolladores empiecen a usarla, normalmente hay que tener cuidado al realizar cambios en dicha API y, al mismo, tiempo, no interrumpir a quienes la llaman. También resulta útil informar a los desarrolladores de los cambios realizados. Para obtener más información, consulte el artículo sobre el [uso de revisiones](api-management-get-started-revise-api.md).
* **Portal para desarrolladores de**: los clientes (programadores) deben usar el Portal para desarrolladores para acceder a las API. Se puede personalizar el Portal para desarrolladores. Para obtener más información, consulte cómo [personalizar el Portal para desarrolladores](api-management-customize-styles.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una instancia](get-started-create-service-instance.md)

