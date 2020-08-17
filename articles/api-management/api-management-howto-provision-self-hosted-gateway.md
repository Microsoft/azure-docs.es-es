---
title: Aprovisionamiento de una puerta de enlace autohospedada en Azure API Management | Microsoft Docs
description: Aprenda a aprovisionar una puerta de enlace autohospedada en Azure API Management.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/31/2020
ms.author: apimpm
ms.openlocfilehash: e79248e16ca21ae84022f8ac7f280d93f489c6a4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050338"
---
# <a name="provision-a-self-hosted-gateway-in-azure-api-management"></a>Aprovisionamiento de una puerta de enlace autohospedada en Azure API Management

El aprovisionamiento de un recurso de puerta de enlace en su instancia de Azure API Management es un requisito previo para implementar una puerta de enlace autohospedada. En este artículo se describen los pasos necesarios para aprovisionar un recurso de puerta de enlace en API Management.

## <a name="prerequisites"></a>Prerequisites

Completar la guía de inicio rápido siguiente: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="provision-a-self-hosted-gateway"></a>Aprovisionamiento de una puerta de enlace autohospedada

1. Seleccione las **Puertas de enlace** en **Implementación e infraestructura**.
2. Haga clic en **+ Agregar**.
3. Escribe el **Nombre** y la **Región** de la puerta de enlace.
> [!TIP]
> La **Región** especifica la ubicación prevista de los nodos de puerta de enlace que se asociarán a este recurso de puerta de enlace. Es semánticamente equivalente a una propiedad similar asociada a cualquier recurso de Azure, pero se le puede asignar un valor de cadena arbitrario.

4. Opcionalmente, puede escribir una **Descripción** del recurso de puerta de enlace.
5. Opcionalmente, puede seleccionar **+** en **API** para asociar una o más API a este recurso de puerta de enlace.
> [!IMPORTANT]
> De forma predeterminada, ninguna de las API existentes se asociará con el nuevo recurso de puerta de enlace. Por lo tanto, los intentos de invocarlas a través de la nueva puerta de enlace producirán la respuesta `404 Resource Not Found`.

6. Haga clic en **Agregar**.

Ahora el recurso de puerta de enlace se ha aprovisionado en su instancia de API Management. Puede continuar con la implementación de la puerta de enlace.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md)
* Obtenga más información sobre el procedimiento de [Implementación de una puerta de enlace autohospedada en Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md)
* Obtenga más información sobre el procedimiento de [Implementación de una puerta de enlace autohospedada en Docker](how-to-deploy-self-hosted-gateway-docker.md)
