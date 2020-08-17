---
title: 'Solicitud de un paquete de acceso: administración de derechos de Azure AD'
description: Aprenda a usar el portal Mi acceso para solicitar acceso a un paquete de acceso en la administración de derechos de Azure Active Directory.
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
ms.openlocfilehash: 527255653bf2779b36e0e463cd32a0b334c00f96
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798501"
---
# <a name="request-access-to-an-access-package-in-azure-ad-entitlement-management"></a>Solicitud de acceso a un paquete de acceso en la administración de derechos de Azure AD

Con la administración de derechos de Azure AD, los paquetes de acceso permiten realizar una instalación única de los recursos y de las directivas que administran automáticamente el acceso durante todo el período de vida del paquete. 

Los administradores de paquetes de acceso pueden configurar directivas para requerir que los usuarios tengan que aprobar el acceso a los paquetes de acceso. Los usuarios que necesite acceder a un paquete de acceso pueden enviar una solicitud para obtenerlo. En este artículo se describe cómo enviar una solicitud de acceso.

## <a name="sign-in-to-the-my-access-portal"></a>Inicio de sesión en el portal Mi acceso

El primer paso es iniciar sesión en el portal Mi acceso, donde puede solicitar acceso a un paquete de acceso.

**Rol necesario:** Solicitante

1. Busque un correo electrónico o un mensaje del jefe de proyecto o el director comercial con el que trabaja. El correo electrónico debe incluir un vínculo al paquete acceso al que necesitará acceso. El vínculo comienza con `myaccess`, incluye una sugerencia de directorio y termina con un identificador de paquete de acceso.  (Para el gobierno de EE. UU., el dominio puede ser `https://myaccess.microsoft.us` en su lugar).
 
    `https://myaccess.microsoft.com/@<directory_hint>#/access-packages/<access_package_id>`

1. Abra el vínculo.

1. Inicie sesión en el portal Mi acceso.

    Asegúrese de usar la cuenta de su organización (profesional o educativa). Si no está seguro, póngase en contacto con el jefe de proyecto o el director comercial.

## <a name="request-an-access-package"></a>Solicitud de un paquete de acceso

Una vez que haya encontrado el paquete de acceso en el portal Mi acceso, puede enviar una solicitud.

**Rol necesario:** Solicitante

1. Busque el paquete de acceso en la lista.  Si es necesario, puede escribir una cadena de búsqueda y seleccionar el filtro **Nombre**, **Catálogo** o **Recursos** para buscar.

    ![Portal Mi acceso: búsqueda de recursos](./media/entitlement-management-request-access/my-access-resource-search.png)

1. Haga clic en la marca de verificación para seleccionar el paquete de acceso.

1. Haga clic en **Solicitar acceso** para abrir el panel Solicitar acceso.

    ![Portal Mi acceso: paquetes de acceso](./media/entitlement-management-request-access/my-access-request-access-button.png)

1. Si se muestra el cuadro **Justificación comercial**, escriba una justificación para necesitar acceso.

1. Si la opción **¿Se trata de una solicitud para un período específico?** está habilitada, seleccione **Sí** o **No**.

1. Si es necesario, especifique las fechas de inicio y finalización.

    ![Portal Mi acceso: solicitud de acceso](./media/entitlement-management-shared/my-access-request-access.png)

1. Cuando termine, haga clic en **Enviar** para enviar la solicitud.

1. Haga clic en **Historial de solicitudes** para ver una lista de sus solicitudes con el estado.

    Si el paquete de acceso requiere aprobación, la solicitud se encuentra en estado pendiente de aprobación.

### <a name="select-a-policy"></a>Selección de una directiva

Si va a solicitar acceso a un paquete de acceso que tiene varias directivas aplicables, es posible que se le pida que seleccione una directiva. Por ejemplo, un administrador de paquetes de acceso puede configurar un paquete de acceso con dos directivas para dos grupos de empleados internos. La primera directiva podría permitir el acceso durante 60 días y requerir aprobación. La segunda directiva podría permitir el acceso durante 2 días y no requerir aprobación. Si se encuentra con este escenario, debe seleccionar la directiva que quiere utilizar.

![Portal Mi acceso: solicitud de acceso (varias directivas)](./media/entitlement-management-request-access/my-access-multiple-policies.png)

## <a name="resubmit-a-request"></a>Nuevo envío de una solicitud

Al solicitar acceso a un paquete de acceso, es posible que se deniegue la solicitud o que esta expire si los aprobadores no responden a tiempo. Si necesita acceso, puede intentarlo de nuevo y volver a enviar la solicitud. En el procedimiento siguiente se explica cómo volver a enviar una solicitud de acceso:

**Rol necesario:** Solicitante

1. Inicie sesión en el portal **Mi acceso**.

1. Haga clic en **Historial de solicitudes** en el menú de navegación de la izquierda.

1. Busque el paquete de acceso para el que va a volver a enviar una solicitud.

1. Haga clic en la marca de verificación para seleccionar el paquete de acceso.

1. Haga clic en el vínculo azul **Ver** a la derecha del paquete de acceso seleccionado.
    
    ![Selección del paquete de acceso y visualización del vínculo](./media/entitlement-management-request-access/resubmit-request-select-request-and-view.png)

    Se abrirá un panel a la derecha con el historial de solicitudes para el paquete de acceso.
    
    ![Selección del botón Volver a enviar](./media/entitlement-management-request-access/resubmit-request-select-resubmit.png)

1. Haga clic en el botón **Volver a enviar**, situado en la parte inferior del panel.

## <a name="cancel-a-request"></a>Cancelación de una solicitud

Si envía una solicitud de acceso y esta aún se encuentra en estado **pendiente de aprobación**, puede cancelarla.

**Rol necesario:** Solicitante

1. En el portal Mi acceso, a la izquierda, haga clic en **Historial de solicitudes** para ver una lista de sus solicitudes con el estado.

1. Haga clic en el vínculo **Ver** de la solicitud que quiere cancelar.

1. Si la solicitud aún se encuentra en estado **pendiente de aprobación**, puede hacer clic en **Cancelar solicitud** para cancelarla.

    ![Portal Mi acceso: Cancelar solicitud](./media/entitlement-management-request-access/my-access-cancel-request.png)

1. Haga clic en **Historial de solicitudes** para confirmar la cancelación de la solicitud.

## <a name="next-steps"></a>Pasos siguientes

- [Aprobación o denegación de solicitudes de acceso](entitlement-management-request-approve.md)
- [Solicitud de notificaciones de proceso y correo electrónico](entitlement-management-process.md)
