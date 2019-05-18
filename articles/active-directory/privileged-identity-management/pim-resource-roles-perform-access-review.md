---
title: Revisar el acceso a los roles de recursos de Azure de PIM - Azure Active Directory | Microsoft Docs
description: Obtenga información sobre cómo revisar el acceso de roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 03/30/2018
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: ede412292caf682c3078002d23e1956bd68d1378
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602206"
---
# <a name="review-access-to-azure-resource-roles-in-pim"></a>Revisar el acceso a los roles de recursos de Azure en PIM
Azure Active Directory (Azure AD) Privileged Identity Management (PIM) simplifica cómo las empresas administran el acceso con privilegios a los recursos de Azure. 

Si se le ha asignado un rol administrativo, el administrador de roles con privilegios de su organización le puede pedir que revise y confirme periódicamente que aún necesita ese rol para su trabajo. Puede que reciba un correo electrónico con un vínculo o que vaya directamente al [Portal de Azure](https://portal.azure.com). Siga los pasos de este artículo para realizar una autorrevisión de los roles asignados.

Si es un administrador de roles con privilegios interesado en las revisiones de acceso, puede obtener más información en [Inicio de una revisión de acceso](pim-resource-roles-start-access-review.md).

## <a name="add-the-privileged-identity-management-application"></a>Incorporación de la aplicación Privileged Identity Management
Puede usar la aplicación PIM de Azure Active Directory (Azure AD) en [Azure Portal](https://portal.azure.com/) para realizar la revisión. Si no tiene la aplicación en el portal, siga estos pasos para comenzar.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).
2. Seleccione su nombre de usuario en la esquina superior derecha de Azure Portal y seleccione el directorio donde va a trabajar.
3. Seleccione **Todos los servicios** y use el cuadro de texto **Filtro** para buscar *Azure AD Privileged Identity Management*.
4. Seleccione **Anclar al panel** y, después, seleccione **Crear**. Se abre la aplicación PIM.

## <a name="approve-or-deny-access"></a>Aprobación o denegación de acceso
Al aprobar o denegar el acceso, lo que hace es indicar al revisor si aún usa o no este rol. Elija **Aprobar** si desea permanecer en el rol o **Denegar** si no va a volver a necesitar el acceso. El estado cambia solo cuando el revisor aplica a los resultados.

Para buscar y completar la revisión de acceso, siga estos pasos:
1. Vaya a la aplicación PIM de Azure AD.
2. Seleccione la hoja **Revisar acceso**.

   ![Captura de pantalla de la aplicación PIM, con la hoja Revisar acceso seleccionada](media/pim-resource-roles-perform-access-review/rbac-access-review-complete.png)

3. Seleccione la revisión que quiere completar. 
4. Elija **Aprobar** o **Denegar**. En el cuadro de texto **Proporcionar un motivo**, puede que necesite incluir un motivo de su decisión.

   ![Captura de pantalla de la página de detalles de la revisión](media/pim-resource-roles-perform-access-review/rbac-access-review-choice.png)

## <a name="next-steps"></a>Pasos siguientes

- [Realizar una revisión de acceso de Mis roles de Azure AD en PIM](pim-how-to-perform-security-review.md)
