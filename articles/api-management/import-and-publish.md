---
title: Importación y publicación de su primera API en Azure API Management
description: Obtenga información sobre cómo importar una API de especificación de OpenAPI en Azure API Management y probar luego la API en Azure Portal.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 71a239393c61dccf39ed505aa2b08d7612b7e370
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905557"
---
# <a name="import-and-publish-your-first-api"></a>Importación y publicación de la primera API

En este tutorial se muestra cómo importar una API de back-end de especificación de OpenAPI en formato JSON en Azure API Management. Microsoft proporciona la API de back-end y la hospeda en Azure en [https://conferenceapi.azurewebsites.net?format=json](https://conferenceapi.azurewebsites.net?format=json).

Una vez que la API de back-end se importa en API Management, la API de API Management se convierte en una fachada de la API de back-end. Puede personalizar la fachada en API Management según sus necesidades sin modificar la API de back-end. Para obtener más información, consulte [Transformación y protección de una API](transform-api.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Importación de una API en API Management
> * Prueba de la API en Azure Portal

![Nueva API](./media/api-management-import-and-publish/created-api.png)

## <a name="prerequisites"></a>Requisitos previos

- Conocer la [terminología de Azure API Management](api-management-terminology.md).
- [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-backend-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

Esta sección explica cómo importar y publicar una API de back-end de especificación de OpenAPI.

1. En el panel de navegación izquierdo de la instancia de API Management, seleccione **API** en la sección **API Management**.
1. Seleccione el icono **OpenAPI** y, a continuación, seleccione **Completo** en la pantalla emergente.
1. En la pantalla **Crear desde la especificación de OpenAPI**, use los valores de la tabla siguiente para crear la API.

   El asterisco rojo junto a un campo del formulario indica que el campo es obligatorio. Puede establecer los valores de API durante la creación o más adelante si accede a la pestaña **Configuración**.

   ![Creación de una API](./media/api-management-import-and-publish/create-api.png)

   |Configuración|Value|Descripción|
   |-------|-----|-----------|
   |**Especificación OpenAPI**|*https:\//conferenceapi.azurewebsites.net?format=json*|Servicio que implementa la API. API Management envía las solicitudes a esta dirección.|
   |**Nombre para mostrar**|Después de escribir la dirección URL del servicio anterior, API Management rellena este campo según el código JSON.|El nombre se muestra en el portal para desarrolladores.|
   |**Nombre**|Después de escribir la dirección URL del servicio anterior, API Management rellena este campo según el código JSON.|Nombre único para la API.|
   |**Descripción**|Después de escribir la dirección URL del servicio anterior, API Management rellena este campo según el código JSON.|Descripción opcional de la API.|
   |**Esquema URL**|**HTTPS**|Determina los protocolos que se pueden usar para acceder a la API.|
   |**Sufijo de dirección URL de API**|*conference*|Sufijo que se anexa a la dirección URL base del servicio API Management. API Management distingue las API por su sufijo, por lo que el sufijo debe ser único para cada API de un publicador determinado.|
   |**Etiquetas**| |Etiquetas para organizar las API para buscar, agrupar o filtrar.|
   |**Productos**|**Sin límite**|Asociación de una o más API. Cada instancia de API Management incluye dos productos de ejemplo: **Starter** y **Unlimited**. Para publicar la API, hay que asociarla a un producto (en este ejemplo, **Unlimited**).<br/>Puede incluir varias API en un producto y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. Para agregar esta API a otro producto, escriba o seleccione el nombre del producto. Repita este paso para agregar la API a varios productos. También puede agregar las API a los productos más adelante desde la página **Configuración**.<br/>Para acceder a la API, los desarrolladores primero deben suscribirse a un producto. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. <br/>Si creó la instancia de API Management, ya es un administrador, por lo que está suscrito a todos los productos de la instancia.|
   |**Puertas de enlace**|**Administrado**|Puertas de enlace de API que exponen la API. Este campo solo está disponible en los niveles de servicio **Developer** y **Premium**.<br/>Puerta de enlace **administrada** indica la puerta de enlace integrada en el servicio API Management y hospedada por Microsoft en Azure. Otras puertas de enlace son las [puertas de enlace autohospedadas](self-hosted-gateway-overview.md) que solo están disponibles en los niveles de servicio Premium y Developer. Puede implementarlas en el entorno local o en otras nubes.<br/>Si no se selecciona ninguna puerta de enlace, la API no estará disponible y las solicitudes de API no se realizarán correctamente.|
   |**¿Definir versión de esta API?**|Seleccionar o anular la selección|Para más información sobre las versiones, consulte [Publicación de varias versiones de la API](api-management-get-started-publish-versions.md).|

   > [!NOTE]
   > Para publicar la API a los consumidores de API, tiene que asociarla a un producto.

2. Seleccione **Crear**.

Si surgen problemas con la importación de una definición de API, consulte la [lista de problemas y restricciones conocidos](api-management-api-import-restrictions.md).

## <a name="test-the-new-api-in-the-azure-portal"></a>Prueba de la nueva API en Azure Portal

Puede llamar a las operaciones de la API directamente desde Azure Portal, que proporciona una forma cómoda de ver y probar las operaciones.

1. En el panel de navegación izquierdo de la instancia de API Management, seleccione **API** en la sección **API Management** y, a continuación, seleccione **Demo Conference API**.
1. Seleccione la pestaña **Prueba** y, a continuación, seleccione **GetSpeakers**. En la página se muestran los **parámetros de consulta** y los **encabezados**, si los hay. **Clave de suscripción de APIM de OCP** se rellena automáticamente con la clave de suscripción asociada a esta API.
1. Seleccione **Enviar**.

   ![Prueba del mapa de API](./media/api-management-import-and-publish/01-import-first-api-01.png)

   El back-end responde con **200 Aceptar** y algunos datos.

## <a name="next-steps"></a><a name="next-steps"> </a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Importación de la primera API
> * Prueba de la API en Azure Portal

Avance al siguiente tutorial para aprender a crear y publicar un producto:

> [!div class="nextstepaction"]
> [Creación y publicación de un producto](api-management-howto-add-products.md)
