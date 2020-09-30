---
title: 'Revisión del acceso a los grupos y las aplicaciones en las revisiones de acceso: Azure AD'
description: Aprenda a revisar el acceso de los miembros del grupo o de las aplicaciones en las revisiones de acceso de Azure Active Directory.
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
ms.openlocfilehash: efab031ac2ad96524ce12e6d1c4a942f01ce2344
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299973"
---
# <a name="review-access-to-groups-and-applications-in-azure-ad-access-reviews"></a>Revisión del acceso a los grupos y las aplicaciones en las revisiones de acceso de Azure AD

Azure Active Directory (Azure AD) simplifica el modo en que las empresas administran el acceso a los grupos y las aplicaciones en Azure AD y otras instancias de Microsoft Online Services, gracias a una característica denominada revisiones de acceso de Azure AD. En este artículo se explica cómo un revisor designado lleva a cabo una revisión de acceso de los miembros de un grupo o de los usuarios con acceso a una aplicación. Si quiere revisar el acceso a un paquete, lea [Revisión del acceso de un paquete de acceso de administración de derechos de Azure AD](entitlement-management-access-reviews-review-access.md).

## <a name="perform-access-review-using-my-apps"></a>Revisión de acceso con Mis aplicaciones

Puede iniciar el proceso de revisión de acceso desde el correo electrónico de notificación o si va directamente al sitio.

- **Correo electrónico**:

>[!IMPORTANT]
> Puede que haya retrasos en la recepción del correo electrónico que, en algunos casos, podrían ser de hasta 24 horas. Agregue azure-noreply@microsoft.com a la lista de destinatarios seguros para asegurarse de recibir todos los correos electrónicos.

1. Busque un correo electrónico de Microsoft en el que se le pide que revise el acceso. Este es un ejemplo de un correo electrónico para revisar el acceso de un grupo.

    ![Ejemplo de correo electrónico de Microsoft para revisar el acceso a un grupo](./media/perform-access-review/access-review-email.png)

1. Haga clic en el vínculo **Start review** (Iniciar revisión) para abrir la revisión de acceso.

- **Si no tiene el correo electrónico**, puede encontrar las revisiones de acceso pendientes con estos pasos.

    1. Inicie sesión en el portal Mis aplicaciones en [https://myapps.microsoft.com](https://myapps.microsoft.com).

        ![Portal Mis aplicaciones con las aplicaciones para las que se tiene permiso](./media/perform-access-review/myapps-access-panel.png)

    1. En la esquina superior derecha de la página, haga clic en el usuario que aparece junto a su nombre y la organización predeterminada. Si aparece más de una organización, seleccione la que haya solicitado una revisión de acceso.

    1. Haga clic en el icono **Revisiones de acceso** para ver una lista de las revisiones de acceso pendientes.

        > [!NOTE]
        > Si el icono **Revisiones de acceso** no está visible, no hay ninguna revisión de acceso pendiente para esa organización y no se requiere ninguna acción en este momento.

        ![Lista de revisiones de acceso pendientes para aplicaciones y grupos](./media/perform-access-review/access-reviews-list.png)

    1. Haga clic en el vínculo **Begin review** (Comenzar revisión) correspondiente a la revisión de acceso que quiere realizar.

Al abrir la revisión de acceso, se ven los nombres de los usuarios cuyo acceso debe revisarse.

Si la solicitud es para revisar su propio acceso, la página tendrá un aspecto diferente. Para más información, consulte [Revisar el acceso por sí mismo para grupos o aplicaciones](review-your-access.md).

![Revisión de acceso abierta con los usuarios que se deben revisar](./media/perform-access-review/perform-access-review.png)

Hay dos maneras de aprobar o denegar el acceso:

- Puede aprobar o denegar el acceso para uno o más usuarios de forma "manual". Para ello, elija la acción adecuada para cada solicitud de usuario.
- Puede aceptar las recomendaciones del sistema.

### <a name="approve-or-deny-access-for-one-or-more-users"></a>Aprobación o denegación del acceso de uno o varios usuarios

1. Revise la lista de usuarios y decida si aprueba o deniega su acceso continuado.

    - Para aprobar o denegar el acceso de un solo usuario, haga clic en la fila para abrir una ventana y especificar la acción que se realizará. 
    - Para aprobar o denegar el acceso de varios usuarios, agregue marcas de verificación junto a los usuarios y, luego, haga clic en el botón **Review X user(s)** (Revisar usuarios: X) para abrir una ventana y especificar la acción que se realizará.

1. Haga clic en **Aprobar** o **Denegar**. 

    ![Ventana de acción que incluye las opciones Aprobar, Denegar y Desconocido](./media/perform-access-review/approve-deny.png)
    >[!NOTE]
    > Si no está seguro, puede hacer clic en **Desconocido**. Así el usuario mantiene el acceso y su elección se registra en los registros de auditoría.

1. El administrador de la revisión de acceso puede exigirle que indique el motivo de la decisión en el cuadro **Motivo**, aunque no se requiera uno. Puede indicar una razón para la decisión; la información que incluya estará disponible para otros revisores.

1. Una vez que ha especificado la acción para realizar, haga clic en **Guardar**.

    >[!NOTE]
    > Puede cambiar la respuesta en cualquier momento hasta el fin de la revisión de acceso. Si quiere cambiar la respuesta, seleccione la fila y actualice la respuesta. Por ejemplo, puede aprobar el acceso de un usuario denegado anteriormente o denegar el de un usuario aprobado antes.

    >[!IMPORTANT]
    > - Si un usuario tiene el acceso denegado, no se quita inmediatamente. Se quita cuando el período de revisión ha finalizado o cuando un administrador detiene la revisión si [Aplicar automáticamente](complete-access-review.md#apply-the-changes) está habilitado.
    > - Si hay varios revisores, se registra la última respuesta enviada. Considere un ejemplo donde un administrador designa dos revisores: Alice y Bob. Alice primero abre la revisión de acceso y aprueba la solicitud de acceso de un usuario. Antes de que finalice el período de revisión, Bob abre la revisión de acceso y deniega el acceso a la misma solicitud aprobada anteriormente por Alice. La última decisión de denegación de acceso es la respuesta que se registra.

### <a name="approve-or-deny-access-based-on-recommendations"></a>Aprobación o denegación del acceso según las recomendaciones

Para facilitar las revisiones de acceso y que sean más rápidas, también se proporcionan recomendaciones que se pueden aceptar con un solo clic. Las recomendaciones se generan en función de actividad de inicio de sesión del usuario.

1. En la barra azul en la parte inferior de la página, haga clic en **Aceptar recomendaciones**.

    ![Revisión de acceso abierta que muestra el botón Aceptar recomendaciones](./media/perform-access-review/accept-recommendations.png)

    Verá un resumen de las acciones recomendadas.

    ![Ventana que muestra un resumen de las acciones recomendadas](./media/perform-access-review/accept-recommendations-summary.png)

1. Haga clic en **Aceptar** para aceptar las recomendaciones.

## <a name="perform-access-review-using-my-access-new"></a>Revisión de acceso con Mi acceso (nuevo)

Puede acceder a la nueva experiencia de revisor con la interfaz de usuario actualizada de Mi acceso de un par de maneras diferentes:

### <a name="my-apps-portal"></a>Portal Mis aplicaciones

1. Inicie sesión en Mis aplicaciones en [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portal Mis aplicaciones con las aplicaciones para las que se tiene permiso](./media/perform-access-review/myapps-access-panel.png)

2. Haga clic en el icono **Revisiones de acceso** para ver una lista de las revisiones de acceso pendientes.

    > [!NOTE]
    > Si el icono **Revisiones de acceso** no está visible, no hay ninguna revisión de acceso pendiente para esa organización y no se requiere ninguna acción en este momento.

![Lista de revisiones de acceso pendientes para aplicaciones y grupos con el mensaje emergente de nueva experiencia disponible que se muestra durante la versión preliminar](./media/perform-access-review/banner.png)

3. Haga clic en **Pruébelo** en el mensaje emergente de la parte superior de la página. Esta acción le lleva a la nueva experiencia Mi acceso.
  
### <a name="email"></a>Email

  >[!IMPORTANT]
> Puede que haya retrasos en la recepción del correo electrónico que, en algunos casos, podrían ser de hasta 24 horas. Agregue azure-noreply@microsoft.com a la lista de destinatarios seguros para asegurarse de recibir todos los correos electrónicos.

   1. Busque un correo electrónico de Microsoft en el que se le pide que revise el acceso. Puede ver un mensaje de correo electrónico de ejemplo a continuación:

   ![Ejemplo de correo electrónico de Microsoft para revisar el acceso a un grupo](./media/perform-access-review/access-review-email-preview.png)

   2. Haga clic en el vínculo **Start review** (Iniciar revisión) para abrir la revisión de acceso.

>[!NOTE]
>Si al hacer clic en Iniciar una revisión va directamente a **Mis aplicaciones**, siga los pasos indicados en la sección anterior titulada **Portal Mis aplicaciones**.

### <a name="navigate-to-my-access-directly"></a>Ir a Mi acceso directamente

También puede ver las revisiones de acceso pendientes mediante el explorador para abrir Mi acceso.

1. Inicie sesión en Mi acceso en https://myaccess.microsoft.com/.

2. Seleccione **Revisiones de acceso** en el menú de la barra del lateral izquierdo para ver una lista de las revisiones de acceso pendientes que tiene asignadas.

   ![Revisiones de acceso en el menú](./media/perform-access-review/access-review-menu.png)

### <a name="approve-or-deny-access-for-one-or-more-users"></a>Aprobación o denegación del acceso de uno o varios usuarios

Después de abrir Mi acceso en Grupos y aplicaciones, puede ver:

- **Nombre** Nombre de la revisión de acceso.
- **Vencimiento** Fecha de vencimiento de la revisión. Después de esta fecha, los usuarios cuyo acceso ha sido denegado pueden quitarse del grupo o la aplicación que se está revisando.
- **Recurso** Nombre del recurso que se está revisando.
- **Progreso** Número de usuarios revisados con respecto al número total de usuarios que forman parte de esta revisión de acceso.

Haga clic en el nombre de una revisión de acceso para comenzar.

![Lista de revisiones de acceso pendientes para aplicaciones y grupos](./media/perform-access-review/access-reviews-list-preview.png)

Una vez abierta, se ve la lista de usuarios en el ámbito de la revisión de acceso. Si la solicitud es para revisar su propio acceso, la página tendrá un aspecto diferente. Para más información, consulte [Revisar el acceso por sí mismo para grupos o aplicaciones](review-your-access.md).

Hay dos maneras de aprobar o denegar el acceso:

- Puede aprobar o denegar el acceso para uno o más usuarios de forma manual.
- Puede aceptar las recomendaciones del sistema.

#### <a name="manually-approve-or-deny-access-for-one-or-more-users"></a>Aprobación o denegación manual del acceso de uno o varios usuarios

1. Revise la lista de usuarios y decida si aprueba o deniega su acceso continuado.
2. Seleccione uno o más usuarios haciendo clic en el círculo situado junto a sus nombres.
3. Seleccione **Aprobar** o **Denegar** en la barra superior.
    - Si no está seguro, puede hacer clic en **Desconocido**. El usuario mantiene el acceso y su elección se registra en los registros de auditoría. Es importante que tenga en cuenta que toda la información que proporciona está disponible para otros revisores. Estos pueden leer los comentarios y tenerlos en cuenta al revisar la solicitud.

    ![Revisión de acceso abierta con los usuarios que es necesario revisar](./media/perform-access-review/user-list-preview.png)

4. El administrador de la revisión de acceso puede exigirle que indique el motivo de la decisión en el cuadro **Motivo**, aunque no se requiera uno. Puede indicar una razón para la decisión; la información que incluya estará disponible para que la consulten otros revisores.

5. Haga clic en **Enviar**.
    - Puede cambiar su respuesta en cualquier momento hasta que finalice la revisión de acceso. Si quiere cambiar la respuesta, seleccione la fila y actualice la respuesta. Por ejemplo, puede aprobar el acceso de un usuario denegado anteriormente o denegar el de un usuario aprobado antes.

 >[!IMPORTANT]
 > - Si un usuario tiene el acceso denegado, no se quita inmediatamente. Se quita cuando el período de revisión ha finalizado o cuando un administrador detiene la revisión. 
 > - Si hay varios revisores, se registra la última respuesta enviada. Imagine un ejemplo en el que un administrador designa dos revisores: Alice y Bob. Alice primero abre la revisión de acceso y aprueba la solicitud de acceso de un usuario. Antes de que finalice el período de revisión, Bob abre la revisión de acceso y deniega el acceso a la misma solicitud aprobada anteriormente por Alice. La última decisión de denegación de acceso es la respuesta que se registra.

#### <a name="approve-or-deny-access-based-on-recommendations"></a>Aprobación o denegación del acceso según las recomendaciones

Para facilitar las revisiones de acceso y que sean más rápidas, también se proporcionan recomendaciones que se pueden aceptar con un solo clic. Las recomendaciones se generan en función de actividad de inicio de sesión del usuario.

1. Seleccione uno o más usuarios y haga clic en **Aceptar recomendaciones**.

    ![Revisión de acceso abierta que muestra el botón Aceptar recomendaciones](./media/perform-access-review/accept-recommendations-preview.png)

1. Haga clic en **Enviar** para aceptar las recomendaciones.

Para aceptar recomendaciones para todos los usuarios, asegúrese de que no haya ninguno seleccionado y haga clic en el botón **Aceptar recomendaciones** de la barra superior.

>[!NOTE]
>Cuando se aceptan recomendaciones, las decisiones anteriores no se modifican.

## <a name="next-steps"></a>Pasos siguientes

- [Completar una revisión de acceso de grupos o aplicaciones](complete-access-review.md)
