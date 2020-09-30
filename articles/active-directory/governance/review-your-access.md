---
title: 'Revisión del acceso a grupos y aplicaciones en las revisiones de acceso: Azure AD'
description: Obtenga información sobre cómo revisar su propio acceso a grupos o aplicaciones en las revisiones de acceso de Azure Active Directory.
services: active-directory
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/17/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0b52830e7cddb01c129df2344e26ac1684a88872
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89292731"
---
# <a name="review-access-for-yourself-to-groups-or-applications-in-azure-ad-access-reviews"></a>Revisión del acceso a los grupos o las aplicaciones en las revisiones de acceso de Azure AD

Azure Active Directory (Azure AD) simplifica el modo en que las empresas administran el acceso a los grupos o las aplicaciones en Azure AD y otras instancias de Microsoft Online Services, con una característica denominada revisiones de acceso de Azure AD.

En este artículo se describe cómo revisar su propio acceso a un grupo o una aplicación.

## <a name="review-your-access-using-my-apps"></a>Revisar el acceso desde Mis aplicaciones

El primer paso para realizar una revisión de acceso es buscar y abrir la revisión de acceso.

>[!IMPORTANT]
> Podría haber retrasos en la recepción del correo electrónico que, en algunos casos, podrían ser de hasta 24 horas. Agregue azure-noreply@microsoft.com a la lista de destinatarios seguros para asegurarse de recibir todos los correos electrónicos.

1. Busque un correo electrónico de Microsoft que le pide que revise el acceso. Este es un ejemplo de un correo electrónico para revisar su acceso a un grupo.

    ![Ejemplo de correo electrónico de Microsoft para revisar su acceso a un grupo](./media/review-your-access/access-review-email.png)

1. Haga clic en el vínculo **Review access** (Revisar acceso) para abrir la revisión de acceso.

Si no tiene el correo electrónico, puede encontrar las revisiones de acceso pendientes mediante estos pasos.

1. Inicie sesión en el portal Mis aplicaciones en [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portal Mis aplicaciones con las aplicaciones para las que tiene permiso](./media/review-your-access/myapps-access-panel.png)

1. En la esquina superior derecha de la página, haga clic en el símbolo de usuario que muestra la organización predeterminada y el nombre de usuario. Si aparece más de una organización, seleccione la que haya solicitado una revisión de acceso.

1. En el lado derecho de la página, haga clic en el icono **Access reviews** (Revisiones de acceso) para ver una lista de las revisiones de acceso pendientes.

    Si el icono no está visible, no hay ninguna revisión de acceso para esa organización y no se requiere ninguna acción en este momento.

    ![Lista de revisiones de acceso pendientes para aplicaciones y grupos](./media/review-your-access/access-reviews-list.png)

1. Haga clic en el vínculo **Begin review** (Comenzar revisión) correspondiente a la revisión de acceso que quiere realizar.

### <a name="perform-the-access-review"></a>Realización de la revisión de acceso

Una vez que ha abierto la revisión de acceso, puede ver los derechos de acceso.

1. Revise su acceso y decida si todavía lo necesita.

    Si la solicitud es revisar el acceso de otros usuarios, la página se verá diferente. Para obtener más información, consulte [Revisar el acceso a grupos o aplicaciones](perform-access-review.md).

    ![Abrir la revisión de acceso que le pregunta si aún necesita acceso a un grupo](./media/review-your-access/perform-access-review.png)

1. Seleccione **Yes** (Sí) para mantener el acceso o **No** para quitarlo.

1. Si selecciona **Yes** (Sí), es posible que necesite especificar una justificación en el cuadro **Reason** (Motivo).

    ![Revisión de acceso completada que pregunta si todavía necesita acceso a un grupo](./media/review-your-access/perform-access-review-submit.png)

1. Haga clic en **Enviar**.

    Su selección se envía y vuelve al portal Mis aplicaciones.

    Si quiere cambiar su respuesta, vuelva a abrir la página de revisiones de acceso y actualícela. Puede cambiar su respuesta en cualquier momento hasta que finalice la revisión de acceso.

    > [!NOTE]
    > Si indicó que ya no necesita acceso, este no se eliminará de inmediato. Se quitará cuando la revisión haya finalizado o cuando un administrador la detenga.

## <a name="review-your-own-access-using-my-access-new"></a>Revisar su propio acceso desde Mi acceso (novedad)

Puede probar la nueva experiencia con la interfaz de usuario actualizada de Mi acceso de un par de maneras diferentes:

### <a name="my-apps-portal"></a>Portal Mis aplicaciones

1. Inicie sesión en el portal Mis aplicaciones en [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portal Mis aplicaciones con las aplicaciones para las que tiene permiso](./media/review-your-access/myapps-access-panel.png)

2. Haga clic en el icono **Revisiones de acceso** para ver una lista de las revisiones de acceso pendientes.

    > [!NOTE]
    > Si el icono **Revisiones de acceso** no está visible, no hay ninguna revisión de acceso pendiente para esa organización y no se requiere ninguna acción en este momento.

3. Haga clic en **Pruébelo** en el banner de la parte superior de la página para ir a la nueva experiencia de Mi acceso.

    ![Lista de revisiones de acceso pendientes para aplicaciones y grupos con el mensaje emergente de nueva experiencia disponible que se muestra durante la versión preliminar](./media/review-your-access/banner-your-access.png)

4. Continúe en la sección **Realización de la revisión de acceso**

### <a name="email"></a>Email

>[!IMPORTANT]
> Podría haber retrasos en la recepción del correo electrónico que, en algunos casos, podrían ser de hasta 24 horas. Agregue azure-noreply@microsoft.com a la lista de destinatarios seguros para asegurarse de recibir todos los correos electrónicos.

1. Busque un correo electrónico de Microsoft en el que se le pide que revise el acceso. Puede ver un mensaje de correo electrónico de ejemplo a continuación:

 ![Ejemplo de correo electrónico de Microsoft para revisar el acceso a un grupo](./media/review-your-access/access-review-email-preview.png)

2. Haga clic en el vínculo **Review access** (Revisar acceso) para abrir la revisión de acceso.

3. Continúe en la sección **Realización de la revisión de acceso**

>[!NOTE]
>Si al hacer clic en Iniciar una revisión va directamente a **Mis aplicaciones**, siga los pasos indicados en la sección anterior titulada **Portal Mis aplicaciones**.

### <a name="directly-at-my-access"></a>Directamente en Mi acceso

También puede ver las revisiones de acceso pendientes mediante el explorador para abrir Mi acceso.

1. Inicie sesión en Mi acceso en https://myaccess.microsoft.com/.

2. Seleccione **Revisiones de acceso** en el menú de la barra del lateral izquierda para ver una lista de las revisiones de acceso pendientes que tiene asignadas.

   ![Revisiones de acceso en el menú](./media/review-your-access/access-review-menu.png)

### <a name="perform-the-access-review"></a>Realización de la revisión de acceso

1. En Grupos y aplicaciones, puede ver:
    
    - **Nombre** Nombre de la revisión de acceso.
    - **Vencimiento** Fecha de vencimiento de la revisión. Después de esta fecha, los usuarios cuyo acceso ha sido denegado pueden quitarse del grupo o la aplicación que se está revisando.
    - **Recurso** Nombre del recurso que se está revisando.
    - **Progreso** Número de usuarios revisados con respecto al número total de usuarios que forman parte de esta revisión de acceso.
    
2. Haga clic en el nombre de una revisión de acceso para comenzar.

   ![Lista de revisiones de acceso pendientes para aplicaciones y grupos](./media/review-your-access/access-reviews-list-preview.png)

3. Revise su acceso y decida si todavía lo necesita.

    Si la solicitud es revisar el acceso de otros usuarios, la página se verá diferente. Para obtener más información, consulte [Revisar el acceso a grupos o aplicaciones](perform-access-review.md).

    ![Abrir la revisión de acceso que le pregunta si aún necesita acceso a un grupo](./media/review-your-access/review-access-preview.png)

1. Seleccione **Sí** para mantener el acceso o **No** para quitarlo.

1. Si selecciona **Yes** (Sí), es posible que necesite especificar una justificación en el cuadro **Reason** (Motivo).

    ![Revisión de acceso completada que pregunta si todavía necesita acceso a un grupo](./media/review-your-access/review-access-yes-preview.png)

1. Haga clic en **Enviar**.

    Su selección se envía y vuelve a la página Mi acceso.

    Si quiere cambiar su respuesta, vuelva a abrir la página de revisiones de acceso y actualícela. Puede cambiar su respuesta en cualquier momento hasta que finalice la revisión de acceso.

    > [!NOTE]
    > Si indicó que ya no necesita acceso, este no se eliminará de inmediato. Se quitará cuando la revisión haya finalizado o cuando un administrador la detenga.

## <a name="next-steps"></a>Pasos siguientes

- [Completar una revisión de acceso de grupos o aplicaciones](complete-access-review.md)
