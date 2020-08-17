---
title: Autorrevisión de un paquete de acceso en la administración de derechos de Azure AD
description: Obtenga información sobre cómo revisar el acceso de los usuarios de los paquetes de acceso de administración de derechos en las revisiones de acceso de Azure Active Directory (versión preliminar).
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 31c44f2423cdc5c43638fe2515757bcb11a9814c
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798450"
---
# <a name="self-review-of-an-access-package-in-azure-ad-entitlement-management"></a>Autorrevisión de un paquete de acceso en la administración de derechos de Azure AD

La administración de derechos de Azure AD simplifica el modo en que las empresas administran el acceso a grupos, aplicaciones y sitios de SharePoint. En este artículo se describe cómo un usuario realiza una revisión automática de los paquetes de acceso asignados.

## <a name="open-the-access-review"></a>Abrir la revisión de acceso

Para realizar una revisión de acceso, primero debe abrir la revisión de acceso. Use el procedimiento siguiente para buscar y abrir la revisión de acceso:

1. Es posible que reciba un correo electrónico de Microsoft en el que se le pide que revise el acceso. Busque el correo electrónico para abrir la revisión de acceso. Este es un ejemplo de un correo electrónico en el que se solicita una revisión de acceso: 
    
    ![Correo electrónico del revisor de la revisión de acceso](./media/entitlement-management-access-reviews-review-access/self-review-reviewer-email.png)

1. Haga clic en el vínculo **Revisar acceso**.

1. También puede ir directamente a https://myaccess.microsoft.com para buscar las revisiones de acceso pendientes si no recibe un correo electrónico.  (Para el Gobierno de EE. UU., utilice en su lugar `https://myaccess.microsoft.us`.)

1. Haga clic en **Revisiones de acceso** en la barra de navegación de la izquierda para ver una lista de las revisiones de acceso pendientes que tiene asignadas.


1.  Haga clic en la revisión que quiera empezar.

## <a name="perform-the-access-review"></a>Realización de la revisión de acceso

Una vez que abra la revisión de acceso, puede ver el acceso. Use el procedimiento siguiente para realizar la revisión de acceso:

1.  Decida si todavía necesita acceder al paquete de acceso. Por ejemplo, el proyecto en el que trabaja no se ha completado, por lo que todavía necesita acceso para seguir trabajando en el proyecto.

1.  Seleccione **Yes** (Sí) para mantener el acceso o **No** para quitarlo.
    >[!NOTE]
    >Si ha indicado que ya no necesita acceso, no se le eliminará del paquete de acceso de inmediato. Se le quitará del paquete de acceso cuando finalice la revisión o si un administrador la detiene.

1.  Si ha hecho clic en **Sí**, es posible que tenga que incluir una declaración de justificación en el cuadro **Motivo**.

1.  Haga clic en **Enviar**.

Puede volver a la revisión si cambia de opinión y decide cambiar la respuesta antes de que finalice la revisión.

## <a name="next-steps"></a>Pasos siguientes

- [Revisión del acceso a los paquetes de acceso](entitlement-management-access-reviews-review-access.md) 
