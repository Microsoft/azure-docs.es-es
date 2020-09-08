---
title: Importación de una API de SOAP y conversión a REST con Azure Portal | Microsoft Docs
description: Obtenga información sobre cómo importar una API SOAP, convertirla a REST con API Management y, a continuación, probar la API en los portales de Azure y para desarrolladores.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 11/22/2017
ms.author: apimpm
ms.openlocfilehash: a87f4d0748d14ab995f75279b6a192f350165d6f
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843834"
---
# <a name="import-a-soap-api-and-convert-to-rest"></a>Importación de API de SOAP y conversión en REST

Este artículo muestra cómo importar una API de SOAP y convertirla en REST. El artículo también muestra cómo probar la API de APIM.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Importación de API de SOAP y conversión en REST
> * Prueba de la API en Azure Portal
> * Pruebe la API en el Portal para desarrolladores

## <a name="prerequisites"></a>Prerequisites

Completar la guía de inicio rápido siguiente: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

1. Seleccione **API** en **API MANAGEMENT**.
2. Seleccione **WSDL** en la lista **Add a new API** (Agregar una nueva API).

    ![API DE SOAP](./media/restify-soap-api/wsdl-api.png)
3. En la **especificación WSDL**, escriba la dirección URL donde reside la API de SOAP.
4. Haga clic en el botón de radio **De SOAP a REST**. Cuando se hace clic en esta opción, APIM trata de realizar una transformación automática entre XML y JSON. En este caso, los consumidores deben llamar a la API como API RESTful, que devuelve JSON. APIM convierte cada solicitud en una llamada SOAP.

    ![De SOAP a REST](./media/restify-soap-api/soap-to-rest.png)

5. Presione Tab.

    Los siguientes campos se rellenan con la información de la API de SOAP: Nombre para mostrar, Nombre y Descripción.
6. Agregue un sufijo URL de API. El sufijo es un nombre que identifica esta API concreta en esta instancia de APIM. Debe ser exclusivo en esta instancia de APIM.
9. Publique la API asociándola a un producto. En este caso, se usa el producto "*Unlimited*".  Si desea que la API se publique y esté disponible para los desarrolladores, agréguela a un producto. Puede hacerlo durante la creación de la API o configurarla más adelante.

    Los productos son asociaciones de una o varias API. Puede incluir varias API y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Si creó la instancia de APIM, ya es un administrador, así que, de forma predeterminada, está suscrito a todos los productos.

    De forma predeterminada, cada instancia de API Management incluye dos productos de ejemplo:

    * **Starter**
    * **Sin límite**   
10. Seleccione **Crear**.

## <a name="test-the-new-api-in-the-azure-portal"></a>Prueba de la nueva API en Azure Portal

Se puede llamar a las operaciones directamente desde Azure Portal, lo que proporciona una forma cómoda de ver y probar las operaciones de una API.  

1. Seleccione la API que creó en los pasos anteriores.
2. Presione la pestaña **Prueba**.
3. Seleccione alguna operación.

    La página muestra los campos de parámetros de consulta y los campos para los encabezados. Uno de los encabezados es "Ocp-Apim-Suscripción-Key", para la clave de suscripción del producto que está asociado a esta API. Si ha creado la instancia APIM, ya es administrador, por lo que la clave se rellena automáticamente. 
1. Presione **Enviar**.

    Back-end responde con **200 Aceptar** y algunos datos.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)