---
title: 'Reescritura de la dirección y la cadena de consulta de URL con Azure Application Gateway: Azure Portal'
description: Aprenda a usar Azure Portal para configurar una instancia de Azure Application Gateway para reescribir la dirección y la cadena de consulta URL
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: how-to
ms.date: 7/16/2020
ms.author: surmb
ms.openlocfilehash: 160d056447bd53ea01437acd372b5efeb15b4773
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083164"
---
# <a name="rewrite-url-with-azure-application-gateway---azure-portal-preview"></a>Reescritura de la dirección URL con Azure Application Gateway: Azure Portal (versión preliminar)

En este artículo se describe cómo usar Azure Portal para configurar una instancia de [Application Gateway SKU v2](application-gateway-autoscaling-zone-redundant.md) para reescribir la dirección URL.

>[!NOTE]
> La característica de reescritura de direcciones URL está en versión preliminar y solo está disponible para las SKU Standard_v2 y WAF_v2 de Application Gateway. No es aconsejable en entornos de producción. Para obtener más información sobre las versiones preliminares, consulte los [términos de uso aquí](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Debe tener una instancia de la SKU de Application Gateway v2 para completar los pasos descritos en este artículo. La reescritura de dirección URL no se admite en la SKU v1. Si no tiene la SKU de v2, cree una instancia de [SKU de Application Gateway v2](tutorial-autoscale-ps.md) antes de comenzar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/) con su cuenta de Azure.

## <a name="configure-url-rewrite"></a>Configuración de la reescritura de URL

En el ejemplo siguiente, siempre que la dirección URL de la solicitud contenga */article*, la ruta de dirección URL y la cadena de consulta de URL se reescriben.

`contoso.com/article/123/fabrikam` -> `contoso.com/article.aspx?id=123&title=fabrikam`

1. Seleccione **Todos los recursos** y seleccione su puerta de enlace de aplicación.

2. Seleccione **Reescrituras** en el panel izquierdo.

3. Seleccione **Conjunto de reescritura**:

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-1.png" alt-text="Agregar un conjunto de reescritura":::

4. Proporcione un nombre para el conjunto de reescritura y asócielo a una regla de enrutamiento:

    a. Escriba el nombre del conjunto de reescritura en el cuadro **Nombre**.
    
    b. Seleccione una o varias de las reglas enumeradas en la lista **Reglas de enrutamiento asociadas**. Esto se usa para asociar la configuración de reescritura al cliente de escucha de origen mediante la regla de enrutamiento. Puede seleccionar solo las reglas de enrutamiento que no se hayan asociado con otros conjuntos de reescritura. Las reglas que ya se han asociado con otros conjuntos de reescritura aparecen atenuadas.
    
    c. Seleccione **Next** (Siguiente).
    
    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-2.png" alt-text="Asociar a una regla":::

5. Crear una regla de reescritura:

    a. Seleccione **Agregar regla de reescritura**.
    
    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-3.png" alt-text="Agregar regla de reescritura":::
    
    b. Escriba un nombre para la regla de reescritura en el cuadro **Nombre de la regla de reescritura**. Escriba un número en el cuadro **Secuencia de reglas**.

6. En este ejemplo, se reescribirá la ruta de la dirección URL y la cadena de consulta de URL solo cuando la ruta de acceso contenga */article*. Para ello, agregue una condición para evaluar si la ruta de acceso de la dirección URL contiene */article*

    a. Seleccione **Agregar condición** y, luego, seleccione el cuadro que contiene las instrucciones **If** para expandirlo.
    
    b. Como en este ejemplo queremos comprobar el patrón */article* en la ruta de acceso de la dirección URL, en la lista **Tipo de variable para comprobar**, seleccione **Variable de servidor**.
    
    c. En la lista **Variable de servidor**, seleccione uri_path
    
    d. En **Distingue mayúsculas de minúsculas**, seleccione **No**.
    
    e. En la lista **Operador**, seleccione **igual (=)** .
    
    f. Escriba un patrón de expresión regular. En este ejemplo, usaremos el patrón `.*article/(.*)/(.*)`.
    
      () se usa para capturar la subcadena para su uso posterior en la creación de la expresión para reescribir la ruta de dirección URL. Para más información, consulte [esta página](rewrite-http-headers-url.md#capturing).

    g. Seleccione **Aceptar**.

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-4.png" alt-text="Condition":::

 

7. Agregar una acción para reescribir la dirección URL y la ruta de dirección URL

   a. En la lista **Tipo de reescritura**, seleccione **URL**.

   b. En la lista **Tipo de acción**, seleccione **Conjunto**.

   c. En **Componentes**, seleccione **Ruta de acceso de URL y cadena de consulta de URL**.

   d. En **Valor de ruta de acceso de URL**, escriba el nuevo valor de la ruta de acceso. En este ejemplo, usaremos **/article.aspx**. 

   e. En **Valor de cadena de consulta de URL**, escriba el nuevo valor de la cadena de consulta de dirección URL. En este ejemplo, usaremos **id={var_uri_path_1}&title={var_uri_path_2}**
    
    `{var_uri_path_1}` y `{var_uri_path_1}` se usan para recuperar las subcadenas capturadas durante la evaluación de la condición en esta expresión `.*article/(.*)/(.*)`.
    
   f. Seleccione **Aceptar**.

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-5.png" alt-text="Acción":::

8. Seleccione **Crear** para crear el conjunto de reescritura.

9. Compruebe que el nuevo conjunto de reescritura aparece en la lista de conjuntos de reescritura.

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-6.png" alt-text="Agregar regla de reescritura":::

## <a name="verify-url-rewrite-through-access-logs"></a>Comprobación de la reescritura de dirección URL mediante los registros de acceso

Observe los campos siguientes en los registros de acceso para comprobar si la reescritura de la dirección URL se produjo según lo esperado.

* **originalRequestUriWithArgs**: Este campo contiene la dirección URL de la solicitud original
* **requestUri**: Este campo contiene la dirección URL posterior a la operación de reescritura en Application Gateway

Para más información sobre todos los campos de los registros de acceso, consulte [este](application-gateway-diagnostics.md#for-application-gateway-and-waf-v2-sku) documento.

##  <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo configurar las reescrituras para algunos casos de uso comunes, consulte los [escenarios de reescritura comunes](rewrite-http-headers.md).
