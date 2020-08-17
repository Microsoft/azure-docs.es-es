---
title: 'Colaboración en la base de conocimiento: QnA Maker'
description: QnA Maker permite que varias personas colaboren en una base de conocimiento. Esta característica se proporciona con el Control de acceso basado en rol de Azure.
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 4c550a36dd045873df4bc4a382e1c884466f95ad
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87054077"
---
# <a name="collaboration-with-authors-and-editors"></a>Colaboración con creadores y editores

La colaboración se proporciona en el nivel de recursos de QnA Maker para que pueda restringir el acceso de los colaboradores según el rol del colaborador. Obtenga más información sobre los [conceptos](../Concepts/role-based-access-control.md) de autenticación de colaboradores en QnA Maker.

## <a name="add-role-based-access-rbac-to-your-qna-maker-resource"></a>Incorporación del acceso basado en roles (RBAC) al recurso de QnA Maker

QnA Maker permite que varias personas colaboren en todas las bases de conocimiento del mismo recurso de QnA Maker. Esta característica se proporciona con el [Control de acceso basado en rol](../../../active-directory/role-based-access-control-configure.md) de Azure.

## <a name="access-at-the-qna-maker-resource-level"></a>Acceso en el nivel de recursos de QnA Maker

No puede compartir una base de conocimiento particular en un servicio QnA Maker. Si desea un control de acceso más pormenorizado, considere la distribución de las bases de conocimiento entre diferentes recursos de QnA Maker y, a continuación, agregue los roles a cada recurso.

## <a name="add-role-to-resource"></a>Agregar un rol a un recurso

### <a name="add-a-user-account-to-the-qna-maker-resource"></a>Agregar una cuenta de usuario al recurso de QnA Maker

En los pasos siguientes se usa el rol de colaborador, pero cualquiera de los [roles](../reference-role-based-access-control.md) se puede agregar mediante estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya al recurso de QnA Maker.

    ![Lista de recursos de QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-resource-list.png)

1. Vaya a la pestaña **Access Control (IAM)** .

    ![QnA Maker (IAM)](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam.png)

1. Seleccione **Agregar**.

    ![Agregar QnA Maker (IAM)](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add.png)

1. Seleccione un rol en la lista siguiente:

    |Role|
    |--|
    |Propietario|
    |Colaborador|
    |Lector de QnA Maker de Cognitive Services|
    |Editor de QnA Maker de Cognitive Services|
    |Usuario de Cognitive Services|

    :::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-add-role-iam.png" alt-text="Adición de rol de QnA Maker (IAM).":::

1. Escriba la dirección de correo electrónico del usuario y presione **Guardar**.

    ![Agregar correo electrónico de QnA Maker (IAM)](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add-email.png)

### <a name="view-qna-maker-knowledge-bases"></a>Visualización de knowledge bases de QnA Maker

Cuando la persona con quien comparte el servicio QnA Maker inicia sesión en el [portal de QnA Maker](https://qnamaker.ai), puede ver todas las bases de conocimiento de ese servicio según el rol.

Cuando esa persona selecciona una knowledge base, su rol actual en ese recurso de QnA Maker es visible junto al nombre de la knowledge base.

:::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-knowledge-base-role-name.png" alt-text="Captura de pantalla de knowledge base en modo de edición con el nombre de rol entre paréntesis junto al nombre de la knowledge base en la esquina superior izquierda de la página web.":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Probar una base de conocimiento](./test-knowledge-base.md)

Obtenga más información sobre la colaboración:
* Control de acceso basado en roles de [Azure](../../../active-directory/role-based-access-control-configure.md)
* [Conceptos](../Concepts/role-based-access-control.md) sobre el control de acceso basado en rol de QnA Maker
