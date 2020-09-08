---
title: Importación de la API de SOAP con Azure Portal | Microsoft Docs
description: Obtenga información sobre cómo importar una representación XML estándar de una API SOAP y, a continuación, probar la API en los portales de Azure y para desarrolladores.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/22/2020
ms.author: apimpm
ms.openlocfilehash: 583179c4678b43bef25174835ec2701d571b37b6
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905657"
---
# <a name="import-soap-api"></a>Importación de una API de SOAP

Este artículo explica cómo importar una representación XML estándar de una API de SOAP. En el artículo también se muestra cómo probar la API de API Management.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Importación de una API de SOAP
> * Prueba de la API en Azure Portal
> * Pruebe la API en el Portal para desarrolladores

## <a name="prerequisites"></a>Prerrequisitos

Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.
2. Seleccione **WSDL** en la lista **Add a new API** (Agregar una nueva API).

    ![API de SOAP](./media/import-soap-api/wsdl-api.png)
3. En la **especificación WSDL**, escriba la dirección URL donde reside la API de SOAP.
4. El botón de radio **Paso a través de SOAP** está seleccionado de forma predeterminada. Con esta selección, la API se va a exponer como SOAP. El consumidor tiene que utilizar reglas SOAP. Si desea usar Restify en la API, siga los pasos descritos en [Import a SOAP API and convert it to REST](restify-soap-api.md) (Importación de una API de SOAP y conversión a REST).

    ![Paso a través](./media/import-soap-api/pass-through.png)
5. Presione Tab.

    Los siguientes campos se rellenan con la información de la API de SOAP: Nombre para mostrar, Nombre y Descripción.
6. Agregue un sufijo URL de API. El sufijo es un nombre que identifica esta API específica en esta instancia de API Management. Tiene que ser único en esta instancia de API Management.
7. Publique la API asociándola a un producto. En este caso, se usa el producto "*Unlimited*".  Si desea que la API se publique y esté disponible para los desarrolladores, agréguela a un producto. Puede hacerlo durante la creación de la API o configurarla más adelante.

    Los productos son asociaciones de una o varias API. Puede incluir varias API y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Si creó la instancia de API Management, ya es un administrador, así que de forma predeterminada está suscrito a todos los productos.

    De forma predeterminada, cada instancia de API Management incluye dos productos de ejemplo:

    * **Starter**
    * **Sin límite**   
8. Escriba otros valores de la API. Puede establecer los valores durante la creación o luego accediendo a la pestaña **Ajustes**. Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#-import-and-publish-a-backend-api).
9. Seleccione **Crear**.

### <a name="test-the-new-api-in-the-administrative-portal"></a>Prueba de la nueva API en el portal de administración

Se puede llamar a las operaciones directamente desde el portal administrativo para desarrolladores, lo que proporciona una forma cómoda de ver y probar las operaciones de una API.  

1. Seleccione la API que creó en los pasos anteriores.
2. Presione la pestaña **Prueba**.
3. Seleccione alguna operación.

    La página muestra los campos de parámetros de consulta y los campos para los encabezados. Uno de los encabezados es "Ocp-Apim-Suscripción-Key", para la clave de suscripción del producto que está asociado a esta API. Si ha creado la instancia de API Management, significa que ya es administrador, por lo que la clave se rellena automáticamente. 
1. Presione **Enviar**.

    Back-end responde con **200 Aceptar** y algunos datos.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)