---
title: Exención de un recurso de las recomendaciones de seguridad de Azure Security Center y de la puntuación segura
description: Sepa cómo eximir un recurso de las recomendaciones de seguridad y la puntuación segura.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: b4d043812800bec1070624382755aeb355ea595a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932433"
---
# <a name="exempt-a-resource-from-recommendations-and-secure-score"></a>Exención de un recurso de las recomendaciones y la puntuación segura

Una prioridad básica de cada equipo de seguridad es intentar asegurarse de que los analistas puedan centrarse en las tareas y los incidentes que importan en la organización. Security Center presenta muchas características para personalizar la información que más prioridad tiene y garantizar que la puntuación segura sea un reflejo válido de las decisiones de seguridad de la organización. La exención de recursos es una de estas características.

Cuando se investiga una recomendación de seguridad en Azure Security Center, uno de los primeros datos que se revisan es la lista de recursos afectados.

En ocasiones, aparecerá un recurso que no debe estar incluido. Es posible que lo haya corregido un proceso del que Security Center no ha realizado un seguimiento. O, quizás, la organización ha decidido aceptar el riesgo de ese recurso específico. 

En tales casos, puede crear una regla de exención para asegurarse de que el recurso no aparezca con los recursos en mal estado en el futuro y no afecte a la puntuación segura. 

El recurso se mostrará como no aplicable y el motivo aparecerá como "exento" con la justificación que seleccione.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Versión preliminar|
|Precios:|Se trata de una funcionalidad premium de Azure Policy que se ofrece a los clientes de Azure Defender sin costo adicional. En el caso de otros usuarios, pueden aplicarse cargos en el futuro.|
|Roles y permisos necesarios:|**propietario de la suscripción** o **colaborador de la directiva** para crear una exención.<br>Para crear una regla, necesita permisos para editar directivas en Azure Policy.<br>Obtenga más información en [Permisos de RBAC en Azure Policy](../governance/policy/overview.md#rbac-permissions-in-azure-policy).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||


## <a name="create-an-exemption-rule"></a>Creación de una regla de exención

1. En la lista de recursos en mal estado, seleccione el menú de puntos suspensivos ("...") del recurso que quiere excluir.

    :::image type="content" source="./media/exempt-resource/create-exemption.png" alt-text="Opción Crear una exención del menú contextual":::

    Se abre el panel Crear exención.

    :::image type="content" source="./media/exempt-resource/exemption-rule-options.png" alt-text="Panel Crear exención":::

1. Escriba los criterios y seleccione uno de ellos para la exención de este recurso:
    - **Mitigado**: este problema no es importante para el recurso porque se ha administrado mediante una herramienta o un proceso diferentes de los que se sugieren.
    - **Renuncia**: aceptación del riesgo de este recurso.
1. Seleccione **Guardar**.
1. Al cabo de un tiempo (puede tardar hasta 24 horas):
    - El recurso no afecta a la puntuación segura.
    - El recurso se muestra en la pestaña **No aplicable** de la página de detalles de la recomendación.
    - La franja de información en la parte superior de la página de detalles de la recomendación muestra el número de recursos exentos:
        
        :::image type="content" source="./media/exempt-resource/info-banner.png" alt-text="Número de recursos exentos":::

1. Para revisar los recursos exentos, abra la pestaña **No aplicable**.

    :::image type="content" source="./media/exempt-resource/modifying-exemption.png" alt-text="Modificación de una exención":::

    La razón de cada exención se incluye en la tabla. 

    Para modificar o eliminar una exención, seleccione el menú de puntos suspensivos ("...") como se muestra (2).


## <a name="review-all-of-the-exemption-rules-on-your-subscription"></a>Revisión de todas las reglas de exención de la suscripción

Las reglas de exención usan Azure Policy para crear una exención para el recurso en la asignación de directiva.

Puede usar Azure Policy para realizar un seguimiento de todas las exenciones de la página **Exención**:

:::image type="content" source="./media/exempt-resource/policy-page-exemption.png" alt-text="Página de exenciones de Azure Policy":::



## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a eximir un recurso de una recomendación para que no afecte a la puntuación segura. Para más información sobre la puntuación segura, consulte:

- [Puntuación de seguridad de Azure Security Center](secure-score-security-controls.md)