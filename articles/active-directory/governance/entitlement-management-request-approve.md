---
title: 'Aprobación o denegación de solicitudes de acceso: administración de derechos de Azure AD'
description: Obtenga información sobre cómo usar el portal Mi acceso para aprobar o denegar solicitudes para un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4703a7d1928415d78eae63c42051542b035d3b1a
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798365"
---
# <a name="approve-or-deny-access-requests-in-azure-ad-entitlement-management"></a>Aprobación o denegación de solicitudes de acceso en la administración de derechos de Azure AD

Con la administración de derechos de Azure AD, puede configurar directivas para requerir aprobación para los paquetes de acceso y elegir uno o varios aprobadores. En este artículo se describe cómo los aprobadores designados pueden aprobar o denegar solicitudes para paquetes de acceso.

## <a name="open-request"></a>Abrir una solicitud

El primer paso para aprobar o denegar solicitudes de acceso es buscar y abrir la solicitud de acceso pendiente de aprobación. Hay dos maneras de abrir la solicitud de acceso.

**Rol necesario:** Aprobador

1. Buscar un correo electrónico de Microsoft Azure que le pida que apruebe o deniegue una solicitud. Este es un ejemplo de correo electrónico:

    ![Correo electrónico de solicitud de aprobación para un paquete de acceso](./media/entitlement-management-shared/approver-request-email.png)

1. Haga clic en el vínculo **Aprobar o denegar una solicitud** para abrir la solicitud de acceso.

1. Inicie sesión en el portal Mi acceso.

Si no tiene el correo electrónico, puede encontrar las solicitudes de acceso pendientes de aprobación mediante estos pasos.

1. Inicie sesión en el portal Mi acceso en [https://myaccess.microsoft.com](https://myaccess.microsoft.com).  (Para el Gobierno de EE. UU., el dominio del vínculo del portal Mi acceso será `myaccess.microsoft.us`).

1. En el menú izquierdo, haga clic en **Aprobaciones** para ver una lista de las solicitudes de acceso pendientes de aprobación.

1. En la pestaña **Pendientes**, busque la solicitud.

## <a name="approve-or-deny-request"></a>Aprobar o denegar una solicitud

Después de abrir una solicitud de acceso pendiente de aprobación, puede ver los detalles que le ayudarán a tomar una decisión de aprobación o denegación.

**Rol necesario:** Aprobador

1. Haga clic en el vínculo **Ver** para abrir el panel Solicitud de acceso.

1. Haga clic en **Detalles** para ver más información sobre la solicitud de acceso.

    Los detalles incluyen el nombre de usuario, la organización, las fechas de inicio y fin del acceso (si se proporcionan), la justificación comercial, cuándo se envió la solicitud y cuándo expirará.

1. Haga clic en **Aprobar** o **Denegar**.

1. Si es necesario, escriba un motivo.

    ![Portal Mi acceso: solicitud de acceso](./media/entitlement-management-request-approve/my-access-approve-request.png)

1. Haga clic en **Enviar** para enviar su decisión.

    Si se configura una directiva con varios aprobadores, solo un aprobador tiene que tomar una decisión sobre la aprobación pendiente. Una vez que un aprobador haya enviado su decisión sobre la solicitud de acceso, la solicitud se completa y ya no está disponible para que los otros aprobadores la aprueben o denieguen. Los otros aprobadores pueden consultar la decisión sobre la solicitud y el aprobador que tomó la decisión en su portal Mi acceso. En este momento, solo se admite la aprobación de fase única.

    Si ninguno de los aprobadores configurados puede aprobar o denegar la solicitud de acceso, la solicitud expira tras la duración de la solicitud configurada. El usuario recibe una notificación que le indica que su solicitud de acceso ha expirado y que debe volver a enviar la solicitud de acceso.

## <a name="next-steps"></a>Pasos siguientes

- [Solicitud de acceso a un paquete de acceso](entitlement-management-request-access.md)
- [Solicitud de notificaciones de proceso y correo electrónico](entitlement-management-process.md)
