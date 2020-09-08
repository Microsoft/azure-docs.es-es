---
title: Establecimiento o modificación de directivas de Azure API Management | Microsoft Docs
description: Aprenda a establecer o modificar directivas de Azure API Management. Estas directivas son documentos XML que describen una secuencia de instrucciones de entrada y salida.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/01/2018
ms.author: apimpm
ms.openlocfilehash: 3a8d3309cdc5ddb0ae75ed6fcf0376118275b835
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848016"
---
# <a name="how-to-set-or-edit-azure-api-management-policies"></a>Establecimiento o modificación de directivas de Azure API Management

La definición de la directiva es un documento XML que describe una secuencia de declaraciones de entrada y de salida. El XML se puede editar directamente en la ventana de definición. También puede seleccionar una directiva predefinida de la lista que se proporciona a la derecha de la ventana de directiva. Las instrucciones aplicables al ámbito actual están habilitadas y resaltadas. Al hacer clic en una declaración habilitada, se agregará el XML correspondiente en la ubicación del cursor en la vista de definición. 

Para obtener más información sobre directivas, consulte [Directivas de Azure API Management](api-management-howto-policies.md).

## <a name="set-or-edit-a-policy"></a>Establecimiento o modificación de directivas

Para establecer o modificar una directiva, siga estos pasos:

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Vaya a la instancia de APIM.
3. Haga clic en la pestaña **API**.

    ![Editar directiva](./media/set-edit-policies/code-editor.png)

4. Seleccione una de las API que haya importado previamente.
5. Seleccione la pestaña **Diseño**.
6. Seleccione una operación en la que desee aplicar la directiva. Si desea aplicar la directiva a todas las operaciones, seleccione **Todas las operaciones**.
7. Seleccione el icono **</>** (editor de código) en la sección **Procesamiento de entrada** o **Procesamiento de salida**.
8. Pegue el código de la directiva que desee en uno de los bloques adecuados.

    ```xml
    <policies>
        <inbound>
            <base />
        </inbound>
        <backend>
            <base />
        </backend>
        <outbound>
            <base />
        </outbound>
        <on-error>
            <base />
        </on-error>
    </policies>
    ```
 
## <a name="configure-scope"></a>Configuración del ámbito

Las directivas se pueden configurar globalmente o en el ámbito de un producto, una API o una operación. Para comenzar a configurar una directiva, se debe seleccionar en primer lugar el ámbito en el que se debe aplicar.

Los ámbitos de la directiva se evalúan en el orden siguiente:

1. Ámbito global
2. Ámbito del producto
3. Ámbito de la API
4. Ámbito de la operación

Las instrucciones dentro de las directivas se evalúan según la ubicación del elemento `base`, si existe. Una directiva global no tiene una directiva principal y el uso del elemento `<base>` en ella no surte efecto.

Para ver las directivas en el ámbito actual en el editor de directivas, haga clic en **Recalcular la directiva en vigor del ámbito seleccionado**.

### <a name="global-scope"></a>Ámbito global

El ámbito global se configura para **todas las API** de su instancia de APIM.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a la instancia de APIM.
2. Haga clic en **Todas las API**.

    ![Ámbito global](./media/api-management-howto-policies/global-scope.png)

3. Haga clic en el icono de triángulo.
4. Seleccione **Editor de código**.
5. Agregue o modifique directivas.
6. Presione **Save**(Guardar). 

    Los cambios se propagan inmediatamente a la puerta de enlace de API Management.

### <a name="product-scope"></a>Ámbito del producto

El ámbito del producto está configurado para el producto seleccionado.

1. Haga clic en **Productos**.

    ![Ámbito del producto](./media/api-management-howto-policies/product-scope.png)

2. Seleccione el producto en el que desea aplicar las directivas.
3. Haga clic en **Directivas**.
4. Agregue o modifique directivas.
5. Presione **Save**(Guardar). 

### <a name="api-scope"></a>Ámbito de la API

El ámbito de la API está configurado para **todas las operaciones** de la API seleccionada.

1. Seleccione la **API** en la que desea aplicar directivas.

    ![Ámbito de la API](./media/api-management-howto-policies/api-scope.png)

2. Seleccione **Todas las operaciones**.
3. Haga clic en el icono de triángulo.
4. Seleccione **Editor de código**.
5. Agregue o modifique directivas.
6. Presione **Save**(Guardar). 

### <a name="operation-scope"></a>Ámbito de la operación 

El ámbito de la operación está configurado para la operación seleccionada.

1. Seleccione una **API**.
2. Seleccione la operación en la que desea aplicar directivas.

    ![Ámbito de la operación](./media/api-management-howto-policies/operation-scope.png)

3. Haga clic en el icono de triángulo.
4. Seleccione **Editor de código**.
5. Agregue o modifique directivas.
6. Presione **Save**(Guardar). 

## <a name="next-steps"></a>Pasos siguientes

Vea también los siguientes temas relacionados:

+ [API de transformación](transform-api.md)
+ En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
+ [Ejemplos de directivas](policy-samples.md)
