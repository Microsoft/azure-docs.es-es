---
title: Creación y publicación de un producto en Azure API Management
description: Obtenga información acerca de cómo crear y publicar productos en Azure API Management. Una vez que se publica un producto, los desarrolladores pueden empezar a usar sus API.
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
ms.date: 08/10/2018
ms.author: apimpm
ms.openlocfilehash: 69b5e381ed8446b45f68b4b1ce9bb13df47039c0
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904911"
---
# <a name="create-and-publish-a-product"></a>Crear y publicar un producto  

En Azure API Management, un producto contiene una o varias API, así como una cuota de uso y los términos de uso. Una vez publicado un producto, los desarrolladores pueden suscribirse al producto y comenzar a usar las API del producto.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear y publicar un producto
> * Agregar una API al producto

![Incorporación de un tutorial del producto](media/api-management-howto-add-products/added-product.png)

## <a name="prerequisites"></a>Requisitos previos

+ Conocer la [terminología de API Management de Azure](api-management-terminology.md).
+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Además, realice el siguiente tutorial: [Importación y publicación de la primera API](import-and-publish.md).

## <a name="create-and-publish-a-product"></a>Crear y publicar un producto

![Add product](media/api-management-howto-add-products/02-create-publish-product-01.png)

1. Haga clic en **Productos** en el menú de la izquierda para mostrar la página **Productos**.
2. Haga clic en **+ Agregar**.

    Al agregar un producto, se debe proporcionar la siguiente información: 

    | Nombre                     | Descripción                                                                                                                                                                                                                                                                                                             |
    |--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Nombre para mostrar             | El nombre que desea que aparezca en el **portal para desarrolladores**.                                                                                                                                                                                                                                                        |
    | Nombre                     | Un nombre descriptivo del producto.                                                                                                                                                                                                                                                                                      |
    | Descripción              | El campo **Configuración** le permite incluir información detallada sobre el producto, por ejemplo, su finalidad, las API a las que proporciona acceso y otra información útil.                                                                                                                                               |
    | State                    | Presione **Publicado** si desea publicar el producto. Para poder llamar a las API de un producto, este debe publicarse. De forma predeterminada, los nuevos productos no se publican y solo son visibles para el grupo **Administradores**.                                                                                      |
    | Requiere suscripción    | Seleccione **Require subscription** (Requerir suscripción) si se requiere que se suscriba un usuario para usar el producto.                                                                                                                                                                                                                                   |
    | Requiere aprobación        | Seleccione **Require approval** (Requerir aprobación) si desea que un administrador revise y acepte o rechace los intentos de suscripción a este producto. Si la casilla está desactivada, los intentos de suscripción se aprueban automáticamente.                                                                                                                         |
    | Límite de recuento de suscripciones | Para limitar el número de varias suscripciones simultáneas, escriba el límite de suscripciones.                                                                                                                                                                                                                                |
    | Términos legales              | Puede incluir los términos de uso del producto que deben aceptar los suscriptores para usarlo.                                                                                                                                                                                                             |
    | API existentes                     | Los productos son asociaciones de una o varias API. Puede incluir varias API y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. <br/> Puede agregar una API existente durante la creación del producto. Puede agregar una API al producto más adelante, bien desde la página **Configuración** del producto o al crear una API. |

3. Haga clic en **Crear** para crear el nuevo producto.

### <a name="add-more-configurations"></a>Adición de más configuraciones

Puede continuar con la configuración del producto después de guardarla si elige la pestaña **Configuración**. 

Vea o agregue suscriptores al producto desde la pestaña **Suscripciones**.

La visibilidad de un producto para los desarrolladores o invitados se puede establecer en la pestaña **Control de acceso**.

## <a name="add-apis-to-a-product"></a><a name="add-apis"> </a>Incorporación de API a un producto

Los productos son asociaciones de una o varias API. Puede incluir varias API y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. Puede agregar una API existente durante la creación del producto. Puede agregar una API al producto más adelante, bien desde la página **Configuración** de los productos o durante la creación de dicha API.

En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Si creó la instancia de APIM, ya es un administrador, así que, de forma predeterminada, está suscrito a todos los productos.

### <a name="add-an-api-to-an-existing-product"></a>Adición de una API a un producto existente

![agregar API de producto](media/api-management-howto-add-products/02-create-publish-product-02.png)

1. En la pestaña **Productos**, seleccione un producto.
2. Vaya a la pestaña **API**.
3. Haga clic en **+ Agregar**.
4. Elija una API y haga clic en **Seleccionar**.

> [!TIP]
> Puede crear o actualizar la suscripción del usuario para un *producto* con claves de suscripción personalizadas [mediante la API REST](/rest/api/apimanagement/2019-12-01/subscription/createorupdate) o un comando de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear y publicar un producto
> * Agregar una API al producto

Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Create blank API and mock API responses](mock-api-responses.md) (Creación de una API en blanco y simulación de respuestas de API)
