---
title: Actualización de una asignación existente desde el portal
description: Obtenga información acerca del mecanismo para actualizar una asignación de plano técnico existente desde el portal en Azure Blueprints.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: ddaac8110885d817bc96745965c1af7e522b1919
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651367"
---
# <a name="how-to-update-an-existing-blueprint-assignment"></a>Procedimiento para actualizar una asignación de plano técnico

Cuando se asigna un plano técnico, se puede actualizar la asignación. Hay varias razones para actualizar una asignación, entre las que se incluyen:

- Agregar o quitar un [bloqueo de recursos](../concepts/resource-locking.md)
- Cambiar el valor de [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters)
- Actualizar la asignación por una versión **publicada** más reciente del plano técnico

## <a name="updating-assignments"></a>Actualización de asignaciones

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. Seleccione **Planos técnicos asignados** en la página de la izquierda.

1. En la lista de planos técnicos, seleccione el botón izquierdo en la asignación del plano técnico. Después, use el botón **Actualizar asignación** o haga clic con el botón derecho en la asignación del plano técnico y seleccione **Actualizar asignación**.

   :::image type="content" source="../media/update-existing-assignments/update-assignment.png" alt-text="Captura de pantalla de la página Asignación de plano técnico con el botón &quot;Actualizar asignación&quot; resaltado." border="false":::

1. La página **Asignar plano técnico** se carga con todos los valores rellenados de la asignación original. Puede cambiar la **versión de la definición de plano técnico**, el estado de **Asignación de bloqueo** y cualquiera de los parámetros dinámicos que existan en la definición del plano técnico. Seleccione **Asignar** cuando haya terminado de realizar los cambios.

1. En la página de detalles de la asignación actualizada, consulte el nuevo estado. En este ejemplo, hemos agregado el **bloqueo** a la asignación.

   :::image type="content" source="../media/update-existing-assignments/updated-assignment.png" alt-text="Captura de pantalla de la página Asignación de plano técnico con el botón &quot;Actualizar asignación&quot; resaltado." border="false":::

1. Explore los detalles acerca de otras **operaciones de asignación** con la lista desplegable. La tabla de **recursos administrados** se actualiza mediante la operación de asignación seleccionada.

   :::image type="content" source="../media/update-existing-assignments/assignment-operations.png" alt-text="Captura de pantalla de la página Asignación de plano técnico con el botón &quot;Actualizar asignación&quot; resaltado." border="false":::

## <a name="rules-for-updating-assignments"></a>Reglas de la actualización de asignaciones

La implementación de las asignaciones actualizadas sigue unas reglas importantes. Estas reglas determinan lo que sucede en los recursos ya implementados. El cambio solicitado y el tipo de recurso del artefacto que se va a implementar o actualizar determinan las acciones que se realizan.

- Asignaciones de roles
  - Si cambia el rol o el encargado de rol (usuario, grupo o aplicación), se crea una nueva asignación de roles. Las asignaciones de roles implementadas previamente se dejan en su lugar.
- Asignaciones de directiva
  - Si cambian los parámetros de la asignación de directiva, se actualiza la asignación existente.
  - Si la definición de la asignación de directiva se cambia, se crea una asignación de directiva nueva.
    Las asignaciones de directivas implementadas previamente se dejan en su lugar.
  - Si se quita el artefacto de asignación de directiva del plano técnico, las asignaciones de directivas implementadas anteriormente se dejan en su lugar.
- Plantillas de Azure Resource Manager (plantillas de ARM)
  - La plantilla se procesa mediante Resource Manager como **PUT**. Como cada tipo de recurso trata esta acción de forma distinta, revise la documentación de cada recurso incluido para determinar el efecto de esta acción cuando la ejecuta Blueprints.

## <a name="possible-errors-on-updating-assignments"></a>Posibles errores en la actualización de asignaciones

Al actualizar las asignaciones, es posible realizar cambios que se interrumpan cuando se ejecutan. Un ejemplo se da al cambiar la ubicación de un grupo de recursos una vez que se haya implementado. Se puede realizar cualquier cambio que admita [Resource Manager](../../../azure-resource-manager/management/overview.md), pero si este produjera un error en Resource Manager, también lo generaría en la asignación.

No hay ningún límite en el número de veces que una asignación se puede actualizar. Si se produce un error, determínelo y realice otra actualización en la asignación.  Escenarios de error de ejemplo:

- Un parámetro incorrecto
- Un objeto ya existente
- Cambio no admitido por Resource Manager

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.