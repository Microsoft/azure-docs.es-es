---
title: Uso de reglas de eliminación de alertas para eliminar falsos positivos u otras alertas de seguridad no deseadas en Azure Security Center.
description: En este artículo se explica cómo usar las reglas de eliminación de Azure Security Center para ocultar las alertas de Azure Defender no deseadas.
author: memildin
manager: rkarlin
services: security-center
ms.author: memildin
ms.date: 09/10/2020
ms.service: security-center
ms.topic: conceptual
ms.openlocfilehash: 0d4c2ddc6b18d2f6767fb3a2761bc6a247e101a1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904890"
---
# <a name="suppress-alerts-from-azure-defender"></a>Eliminación de alertas de Azure Defender

En esta página se explica cómo usar reglas de eliminación de alertas para eliminar falsos positivos u otras alertas de seguridad no deseadas en Azure Defender.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Versión preliminar|
|Precios:|Gratuito<br>(La mayoría de las alertas de seguridad solo están disponibles con Azure Defender).|
|Roles y permisos necesarios:|Los roles de **Administrador de seguridad** y **Propietario** pueden crear o eliminar reglas.<br>Los roles de **Lector de seguridad** y **Lector** pueden ver las reglas.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||


## <a name="what-are-suppression-rules"></a>¿Qué son las reglas de supresión?

Los distintos planes de Azure Defender detectan amenazas en cualquier área del entorno y generan alertas de seguridad.

Si una alerta no es interesante ni pertinente, puede descartarla manualmente. También puede usar la característica de reglas de eliminación para descartar de forma automática alertas similares en el futuro. Por lo general, se usaría una regla de eliminación para:

- Suprimir las alertas identificadas como falsos positivos

- Suprimir las alertas que se desencadenan con demasiada frecuencia para ser útiles

Las reglas de eliminación definen los criterios que se deben seguir para saber qué alertas se deben descartar automáticamente.

> [!CAUTION]
> La eliminación de alertas de seguridad reduce la eficacia de la protección contra amenazas de Azure Defender. Debe comprobar de forma minuciosa el posible efecto de cualquier regla de eliminación y supervisarla a lo largo del tiempo.

:::image type="content" source="./media/alerts-suppression-rules/create-suppression-rule.gif" alt-text="Creación de una regla de eliminación de alertas":::

## <a name="create-a-suppression-rule"></a>Creación de una regla de eliminación

Hay varias maneras de crear reglas para eliminar las alertas de seguridad no deseadas:

- Para eliminar alertas en el nivel del grupo de administración, use Azure Policy.
- Para eliminar alertas en el nivel de suscripción, puede usar Azure Portal o la API REST, como se explica a continuación.

Las reglas de eliminación solo pueden descartar las alertas que ya se hayan desencadenado en las suscripciones seleccionadas.

Para crear una regla directamente en Azure Portal:

1. En la página de alertas de seguridad de Security Center:

    - Busque la alerta específica que ya no desee ver y, en el menú de puntos suspensivos (...) de la alerta, seleccione **Crear regla de eliminación**:

        [![Opción **Crear regla de eliminación**](media/alerts-suppression-rules/auto-dismiss-future-option.png)](media/alerts-suppression-rules/auto-dismiss-future-option.png#lightbox)

    - O bien, seleccione el vínculo de **reglas de eliminación** en la parte superior de la página y, en la página de reglas de eliminación, seleccione **Create new suppression rule** (Crear nueva regla de eliminación):

        ![Botón Crear nueva regla de eliminación**](media/alerts-suppression-rules/create-new-suppression-rule.png)

1. En el panel de la nueva regla de eliminación, escriba los detalles de la nueva regla.
    - La regla puede descartar la alerta en **todos los recursos** para que no reciba ninguna alerta como esta en el futuro.     
    - La regla puede descartar la alerta **según criterios específicos**: cuando se refiere a una dirección IP, un nombre de proceso, una cuenta de usuario, un recurso de Azure o una ubicación específicos.

    > [!TIP]
    > Si abrió la página de la nueva regla desde una alerta concreta, la alerta y la suscripción se configurarán de forma automática en la nueva regla. Si ha usado el vínculo **Crear nueva regla de eliminación**, las suscripciones seleccionadas coincidirán con el filtro actual del portal.

    [![Panel de creación de regla de eliminación](media/alerts-suppression-rules/new-suppression-rule-pane.png)](media/alerts-suppression-rules/new-suppression-rule-pane.png#lightbox)
1. Escriba los detalles de la regla:
    - **Nombre**: un nombre para la regla. Los nombres de las reglas deben comenzar por una letra o un número, tener entre 2 y 50 caracteres, y no contener símbolos, excepto guiones (-) o guiones bajos (_). 
    - **Estado**: puede ser Habilitado o Deshabilitado.
    - **Motivo**: seleccione uno de los motivos incluidos u "otro" si ninguno de ellos se adapta a sus necesidades.
    - **Fecha de expiración**: una fecha y hora de finalización para la regla. Las reglas se pueden ejecutar hasta seis meses.
1. Si lo desea, pruebe la regla con el botón **Simular** para ver el número de alertas que se habrían descartado si esta regla hubiera estado activa.
1. Guarde la regla. 


## <a name="edit-a-suppression-rules"></a>Edición de reglas de eliminación

Para editar las reglas que ha creado, use la página de reglas de eliminación.

1. En la página de alertas de seguridad de Security Center, seleccione el vínculo de **reglas de eliminación** en la parte superior de la página.
1. Se abre la página de reglas de supresión con todas las reglas de las suscripciones seleccionadas.

    [![Lista de reglas de eliminación](media/alerts-suppression-rules/suppression-rules-page.png)](media/alerts-suppression-rules/suppression-rules-page.png#lightbox)

1. Para editar una sola regla, abra el menú de puntos suspensivos (...) de la regla y seleccione **Editar**.
1. Realice los cambios necesarios y seleccione **Aplicar**. 

## <a name="delete-a-suppression-rule"></a>Supresión de una regla de eliminación

Para eliminar una o varias reglas que haya creado, use la página de reglas de eliminación.

1. En la página de alertas de seguridad de Security Center, seleccione el vínculo de **reglas de eliminación** en la parte superior de la página.
1. Se abre la página de reglas de supresión con todas las reglas de las suscripciones seleccionadas.
1. Para eliminar una sola regla, abra el menú de puntos suspensivos (...) de la regla y seleccione **Eliminar**.
1. Para eliminar varias reglas, active las casillas de las reglas que se vayan a eliminar y seleccione **Eliminar**.
    ![Eliminación de una o varias reglas de eliminación](media/alerts-suppression-rules/delete-multiple-alerts.png)

## <a name="view-suppressed-alerts"></a>Visualización de alertas suprimidas

Todavía se generarán alertas que coincidan con las reglas de eliminación habilitadas, pero su estado se establecerá en **descartada**. Puede ver el estado en Azure Portal o en cualquier lugar donde acceda a las alertas de seguridad Security Center. 

> [!TIP]
> [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) no creará incidentes para las alertas eliminadas. En el caso de otros SIEM, puede filtrar las alertas eliminadas mediante el estado de las alertas ("descartadas").

Use el filtro de Security Center para ver las alertas descartadas por las reglas.

* En la página de alertas de seguridad de Security Center, abra las opciones de filtro y seleccione **Descartada**.  

   [![Visualización de las alertas descartadas](media/alerts-suppression-rules/view-dismissed-alerts.png)](media/alerts-suppression-rules/view-dismissed-alerts.png#lightbox)


## <a name="create-and-manage-suppression-rules-with-the-api"></a>Creación y administración de reglas de eliminación con la API

Puede crear, ver o eliminar reglas de eliminación de alertas a través de la API REST de Security Center. 

Los métodos HTTP pertinentes para la eliminación de reglas en la API REST son:

- **PUT**: para crear o actualizar una regla de eliminación en una suscripción especificada.

- **GET**.

    - para enumerar todas las reglas configuradas para una suscripción especificada. Este método devuelve una matriz de las reglas aplicables.

    - Para obtener los detalles de una regla específica en una suscripción especificada. Este método devuelve una regla de eliminación.

    - Para simular el efecto de una regla de eliminación todavía en la fase de diseño. Esta llamada identifica qué alertas existentes se habrían descartado si la regla hubiera estado activa.

- **DELETE**: elimina una regla existente (pero no cambia el estado de las alertas que ya ha descartado).

Para obtener información completa y ejemplos de uso, consulte la [documentación de la API](https://docs.microsoft.com/rest/api/securitycenter/). 


## <a name="next-steps"></a>Pasos siguientes

En este artículo se describen las reglas de eliminación en Azure Security Center que descartan automáticamente las alertas no deseadas.

Para más información sobre las alertas de seguridad de Azure Defender, consulte las siguientes páginas:

- [Alertas de seguridad y la cadena de eliminación de intenciones](alerts-reference.md): una guía de referencia de las alertas de seguridad que puede recibir de Azure Defender.