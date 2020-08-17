---
title: Creación de una revisión de acceso de grupos y aplicaciones - Azure AD
description: Obtenga información sobre cómo crear una revisión de acceso de los miembros del grupo o de la aplicación en las revisiones de acceso de Azure Active Directory.
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
ms.date: 08/09/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9bb290106c5ceafe8c636bbeeab38b74ea475eb4
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056200"
---
# <a name="create-an-access-review-of-groups-and-applications-in-azure-ad-access-reviews"></a>Creación de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD

El acceso a los grupos y las aplicaciones para empleados e invitados cambia a lo largo del tiempo. Para reducir el riesgo derivado de las asignaciones de acceso obsoletas, los administradores pueden usar Azure Active Directory (Azure AD) para crear revisiones de acceso para los miembros de grupo o el acceso de aplicación. Si necesita revisar el acceso de manera rutinaria, también puede crear revisiones de acceso periódicas. Para más información sobre estos escenarios, vea [Administración del acceso de los usuarios con las revisiones de acceso de Azure AD](manage-user-access-with-access-reviews.md) y [Administración del acceso de los invitados con las revisiones de acceso de Azure AD](manage-guest-access-with-access-reviews.md).

En este artículo se describe cómo crear una o varias revisiones de acceso para el acceso de los miembros de un grupo o aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Azure AD Premium P2
- administrador global o administrador de usuarios.

Para obtener más información, consulte [Requisitos de licencia](access-reviews-overview.md#license-requirements).

## <a name="create-one-or-more-access-reviews"></a>Creación de una o varias revisiones de acceso

1. Inicie sesión en Azure Portal y abra la [página de Identity Governance](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/).

1. En el menú izquierdo, haga clic en **Revisiones de acceso**.

1. Haga clic en **Nueva revisión de acceso** para crear una nueva revisión de acceso.

    ![Panel de revisión de acceso de Identity Governance](./media/create-access-review/access-reviews.png)

1. Ponga un nombre a la revisión de acceso. Opcionalmente, asigne a la revisión una descripción. El nombre y la descripción se muestran a los revisores.

    ![Creación de una revisión de acceso: nombre y descripción de la revisión](./media/create-access-review/name-description.png)

1. Establezca un valor para **Fecha de inicio**. De forma predeterminada, una revisión de acceso ocurre una vez, se inicia a la misma hora en que se crea y finaliza en un mes. Puede cambiar las fechas de inicio y de finalización para hacer que una revisión de acceso se inicie en el futuro, transcurridos tantos días como desee.

    ![Creación de una revisión de acceso: fechas de inicio y de finalización](./media/create-access-review/start-end-dates.png)

1. Para que la revisión de acceso sea periódica, cambie la opción **Frecuencia** de **Una vez** a **Semanal**, **Mensual**,  **Trimestral**, **Semestral** o **Anual**. Use el control deslizante o el cuadro de texto **Duración** para definir cuántos días se abrirá cada revisión de la serie periódica para que los revisores escriban datos. Por ejemplo, la duración máxima que puede establecer para una revisión mensual es 27 días, con el fin de evitar la superposición de revisiones.

1. Use el valor **Fin** para especificar cómo finalizar la serie de revisión de acceso periódica. La serie puede terminar de tres formas: se ejecuta continuamente para iniciar revisiones indefinidamente, hasta una fecha concreta, o hasta que se haya completado un número definido de veces. Un administrador de usuarios o un administrador de empresa puede detener la serie después de su creación cambiando la fecha en **Configuración**, de manera que termine en esa fecha.

1. En la sección **Usuarios**, especifique los usuarios a los que se aplica esa revisión de acceso. Las revisiones de acceso pueden ser de los miembros de un grupo o de los usuarios que se hayan asignado a una aplicación. Puede limitar aún más el ámbito de la revisión de acceso para revisar solo los usuarios invitados que sean miembros (o se hayan asignado a la aplicación), en lugar de revisar todos los usuarios miembros o que tengan acceso a la aplicación.

    ![Creación de una revisión de acceso: usuarios](./media/create-access-review/users.png)

1. En la sección **grupo**, seleccione uno o varios grupos cuya pertenencia le gustaría revisar.

    > [!NOTE]
    > Si selecciona más de uno, se crearán varias revisiones de acceso. Por ejemplo, al seleccionar cinco grupos, se crearán cinco revisiones de acceso independientes.
    
    ![Creación de una revisión de acceso: selección del grupo](./media/create-access-review/select-group.png)

1. En la sección **Aplicaciones** (si seleccionó **Asignada a una aplicación** en el paso 8), seleccione las aplicaciones cuyo acceso le gustaría revisar.

    > [!NOTE]
    > Si selecciona más de una aplicación, se crearán varias revisiones de acceso. Por ejemplo, al seleccionar cinco aplicaciones, se crearán cinco revisiones de acceso independientes.
    
    ![Creación de una revisión de acceso: selección de la aplicación](./media/create-access-review/select-application.png)

1. En la sección **Revisores**, seleccione una o más personas para revisar todos los usuarios del ámbito. También puede seleccionar que los miembros revisen su propio acceso. Si el recurso es un grupo, puede pedir a sus propietarios que hagan la revisión. También puede requerir que los revisores proporcionen un motivo cuando aprueben el acceso.

    ![Creación de una revisión de acceso: revisores](./media/create-access-review/reviewers.png)

1. En la sección **Programas**, seleccione el programa que desea usar. El **programa predeterminado** siempre está presente.

    ![Creación de una revisión de acceso: programas](./media/create-access-review/programs.png)

    Puede simplificar el seguimiento y la recopilación de revisiones de acceso para distintos fines organizándolos en programas. Cada revisión de acceso se puede vincular a un programa. Por lo tanto, cuando prepare los informes para un auditor, puede centrarse en las revisiones de acceso en el ámbito de una iniciativa determinada. Los programas y los resultados de la revisión de acceso son visibles para los usuarios con los roles Administrador global, Administrador de usuario, Administrador de seguridad o Lector de seguridad.

    Para ver una lista de los programas, vaya a la página de revisiones de acceso y seleccione **Programas**. Si tiene el rol Administrador global o Administrador de usuario, puede crear programas adicionales. Por ejemplo, puede elegir tener un programa cada para iniciativa de cumplimiento de normas u objetivo empresarial. Si ya no necesita un programa y no tiene ningún control vinculado a él, puede eliminarlo.

### <a name="upon-completion-settings"></a>Configuración de finalización

1. Para especificar lo que sucede una vez finalizada una revisión, expanda la sección **Configuración de finalización**.

    ![Creación de una revisión de acceso: configuración de finalización](./media/create-access-review/upon-completion-settings.png)

1. Si desea quitar automáticamente el acceso para los usuarios que se han denegado, establezca **Aplicar automáticamente los resultados al recurso** en **Habilitar**. Si desea aplicar manualmente los resultados cuando se complete la revisión, establezca el conmutador en **Deshabilitar**.

1. Use la lista **Si el revisor no responde** para especificar lo que ocurre con los usuarios a los que el revisor no ha revisado dentro del período de revisión. Este valor no afecta a los usuarios que los revisores revisaron manualmente. Si la decisión final del revisor es Denegar, se quitará el acceso del usuario.

    - **Sin cambios**: dejar el acceso del usuario sin cambios
    - **Quitar acceso**: quitar el acceso del usuario
    - **Aprobar acceso**: aprobar el acceso del usuario
    - **Aceptar recomendaciones**: aceptar la recomendación del sistema sobre la denegación o aprobación del acceso continuo del usuario

### <a name="advanced-settings"></a>Configuración avanzada

1. Para especificar configuraciones adicionales, expanda la sección **Configuración avanzada**.

    ![Creación de una revisión de acceso: configuración avanzada](./media/create-access-review/advanced-settings-preview.png)

1. Establezca **Mostrar recomendaciones** en **Habilitar** para mostrar las recomendaciones del sistema de los revisores según la información del acceso del usuario.

1. Establezca **Requerir motivo de la aprobación** en **Habilitar** para requerir que el revisor proporcione un motivo para la aprobación.

1. Establezca **Notificaciones de correo** en **Habilitar**para que Azure AD envíe notificaciones de correo electrónico a los revisores cuando se inicia una revisión de acceso y a los administradores cuando se complete.

1. Establezca **Avisos** en **Habilitar** para que Azure AD envíe recordatorios de revisiones de acceso en curso a los revisores que no hayan completado su revisión. 

    >[!NOTE]
    > De forma predeterminada, Azure AD envía automáticamente un recordatorio, hacia la mitad del plazo de finalización, a los revisores que no hayan respondido.

1. (Versión preliminar) El contenido del correo electrónico enviado a los revisores se genera automáticamente en función de los detalles de la revisión, como el nombre de la revisión, el nombre del recurso, la fecha de vencimiento, etc. Si necesita una forma de comunicar más información, como instrucciones adicionales o información de contacto, puede especificar estos detalles en el correo electrónico de contenido adicional para el revisor que se incluirá en la invitación y en los correos electrónicos de recordatorio enviados a los revisores asignados. La sección resaltada a continuación es donde se mostrará esta información.

    ![Revisión del acceso de los usuarios a un grupo](./media/create-access-review/review-users-access-group.png)

## <a name="start-the-access-review"></a>Inicio de la revisión de acceso

Una vez que haya especificado la configuración para una revisión de acceso, haga clic en **Iniciar**. La revisión de acceso aparecerá en la lista con un indicador de su estado.

![Lista de las revisiones de acceso y su estado](./media/create-access-review/access-reviews-list.png)

De forma predeterminada, Azure AD envía un correo electrónico a los revisores poco después de iniciar la revisión. Si decide no hacer que Azure AD envíe el correo electrónico, asegúrese de informar a los revisores de que hay una revisión de acceso esperando para que la lleven a cabo. Puede mostrarles las instrucciones sobre cómo [revisar el acceso a grupos o aplicaciones](perform-access-review.md). Si la revisión es para que los invitados revisen su propio acceso, muéstreles las instrucciones sobre cómo [revisar su propio acceso a los grupos o aplicaciones](review-your-access.md).

Si ha asignado como revisores a invitados y no han aceptado la invitación, no recibirán un correo electrónico de las revisiones de acceso, ya que, antes de proceder a la revisión, deben aceptar la invitación.

## <a name="access-review-status-table"></a>Tabla de estado de revisión de acceso

| Estado | Definición |
|--------|------------|
|NotStarted | Se ha creado la revisión y la detección de usuarios está en espera para empezar. |
|Inicializando   | La detección de usuarios está en curso para identificar a todos los usuarios que forman parte de la revisión. |
|Iniciando | La revisión se está iniciando. Si se habilitan las notificaciones por correo electrónico, se envían mensajes de correo electrónico a los revisores. |
|InProgress | La revisión se ha iniciado. Si se habilitan las notificaciones por correo electrónico, se habrán enviado mensajes de correo electrónico a los revisores. Los revisores pueden enviar decisiones hasta la fecha de vencimiento. |
|Completando | La revisión se ha completado y se están enviando mensajes de correo electrónico al propietario de la revisión. |
|AutoReviewing (En autorrevisión) | La revisión se encuentra en una fase de revisión del sistema. El sistema está registrando decisiones para los usuarios que no se han revisado en función de las recomendaciones o decisiones previamente configuradas. |
|AutoReviewed (Autorrevisado) | El sistema ha registrado decisiones para todos los usuarios que no se han revisado. La revisión está lista para continuar con **Aplicando** si está habilitada la opción de aplicación automática. |
|Aplicando | No habrá ningún cambio en el acceso para los usuarios que se han aprobado. |
|Aplicado | Los usuarios denegados, si los hay, se han quitado del recurso o directorio. |

## <a name="create-reviews-via-apis"></a>Creación de revisiones mediante API

También puede crear las revisiones de acceso mediante API. Qué hacer para administrar las revisiones de acceso de grupos y usuarios de aplicación en Azure Portal también pueden realizarse mediante Microsoft Graph API. Para más información, consulte [Azure AD Access Reviews API reference](https://docs.microsoft.com/graph/api/resources/accessreviews-root?view=graph-rest-beta) (Referencia de API de revisiones de acceso de Azure AD). Para obtener un ejemplo de código, consulte [Example of retrieving Azure AD Access Reviews via Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/m-p/236096) (Ejemplo de recuperación de las revisiones de acceso de Azure AD a través de Microsoft Graph).

## <a name="next-steps"></a>Pasos siguientes

- [Revisión del acceso a grupos o aplicaciones](perform-access-review.md)
- [Revisión del acceso de uno mismo a grupos o aplicaciones](review-your-access.md)
- [Completar una revisión de acceso de grupos o aplicaciones](complete-access-review.md)
