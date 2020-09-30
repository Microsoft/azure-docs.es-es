---
title: archivo de inclusión
description: archivo de inclusión
services: active-directory
author: msaburnley
ms.service: active-directory
ms.topic: include
ms.date: 03/30/2020
ms.author: ajburnle
ms.custom: include file
ms.openlocfilehash: 12c9b1226e3ba928a4062049c7839d4e46ef727d
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "91024999"
---
## <a name="for-users-in-your-directory"></a>para los usuarios del directorio

Siga estos pasos si quiere que los usuarios que están en el directorio puedan solicitar este paquete de acceso. Al definir la directiva de solicitud, puede especificar usuarios individuales o, más comúnmente, grupos de usuarios. Por ejemplo, es posible que su organización ya tenga un grupo, como **Todos los empleados**.  Si ese grupo se agrega a la directiva para los usuarios que pueden solicitar acceso, cualquier miembro de ese grupo podrá solicitar acceso.

1. En la sección **Users who can request access** (Usuarios que pueden solicitar acceso), haga clic en **For users in your directory** (Para usuarios del directorio).

    Cuando se selecciona esta opción, aparecen nuevas opciones para restringir aún más quién en el directorio puede solicitar este paquete de acceso.

    ![Paquete de acceso - Solicitudes - Para los usuarios que están en el directorio](./media/active-directory-entitlement-management-request-policy/for-users-in-your-directory.png)

1. Seleccione una de las siguientes opciones:

    |  |  |
    | --- | --- |
    | **Usuarios y grupos específicos** | Elija esta opción si desea que solo los usuarios y grupos del directorio que especifique puedan solicitar este paquete de acceso. |
    | **Todos los miembros (excepto invitados)** | Elija esta opción si desea que todos los usuarios miembros del directorio puedan solicitar este paquete de acceso. Esta opción no incluye ningún usuario invitado al que pueda haber invitado en su directorio. |
    | **Todos los usuarios (incluidos invitados)** | Elija esta opción si desea que todos los usuarios miembros y los usuarios invitados del directorio puedan solicitar este paquete de acceso. |

    Los usuarios invitados hacen referencia a usuarios externos que han sido invitados al directorio con [Azure AD B2B](../articles/active-directory/b2b/what-is-b2b.md). Para más información sobre las diferencias entre los usuarios miembros y los usuarios invitados, consulte [¿Cuáles son los permisos de usuario predeterminados en Azure Active Directory?](../articles/active-directory/fundamentals/users-default-permissions.md)

1. Si seleccionó **Usuarios y grupos específicos**, haga clic en **Agregar usuarios y grupos**.

1. En el panel Seleccionar usuarios y grupos, seleccione los usuarios y grupos que quiera agregar.

    ![Paquete de acceso - Solicitudes - Seleccionar usuarios y grupos](./media/active-directory-entitlement-management-request-policy/select-users-groups.png)

1. Haga clic en **Seleccionar** para agregar los usuarios y grupos.

1. Vaya a la sección [Aprobación](#approval).

## <a name="for-users-not-in-your-directory"></a>para los usuarios que no están en el directorio

 Los **Usuarios que no están en el directorio** se refieren a los usuarios que están en otro dominio o directorio de Azure AD. Es posible que estos usuarios no hayan sido invitados todavía al directorio. Los directorios de Azure AD deben configurarse para permitir invitaciones en **Restricciones de colaboración**. Para más información, consulte [Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios](../articles/active-directory/b2b/delegate-invitations.md).

> [!NOTE]
> Se creará una cuenta de usuario invitado para un usuario que aún no esté en el directorio cuya solicitud se haya aprobado o aprobado automáticamente. Se invitará al invitado, pero no recibirá un correo electrónico de invitación. En su lugar, recibirá un correo electrónico cuando se entregue su asignación de paquete de acceso. De forma predeterminada, más adelante cuando ese usuario invitado ya no tenga ninguna asignación de paquete de acceso porque su última asignación ha expirado o se ha cancelado, se bloqueará el inicio de sesión de esa cuenta de usuario invitado y se eliminará posteriormente. Si quiere que los usuarios invitados permanezcan en el directorio de manera indefinida, incluso si no tienen ninguna asignación de paquete de acceso, puede cambiar la configuración de la administración de derechos. Para más información acerca del objeto de usuario de invitado, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](../articles/active-directory/b2b/user-properties.md).

Siga estos pasos si quiere que los usuarios que no están en el directorio soliciten este paquete de acceso:

1. En la sección **Usuarios que pueden solicitar acceso**, haga clic en **Para los usuarios que no están en el directorio**.

    Cuando se selecciona esta opción, aparecen nuevas opciones.

    ![Paquete de acceso - Solicitudes - Para los usuarios que no están en el directorio](./media/active-directory-entitlement-management-request-policy/for-users-not-in-your-directory.png)

1. Seleccione una de las siguientes opciones:

    |  |  |
    | --- | --- |
    | **Organizaciones conectadas específicas** | Elija esta opción si desea seleccionar de una lista de organizaciones que el administrador haya agregado anteriormente. Todos los usuarios de las organizaciones seleccionadas pueden solicitar este paquete de acceso. |
    | **Todas las organizaciones conectadas** | Elija esta opción si todos los usuarios de las organizaciones conectadas pueden solicitar este paquete de acceso. |
    | **Todos los usuarios (todas las organizaciones conectadas y todos los nuevos usuarios externos)** | Elija esta opción si todos los usuarios de todas las organizaciones conectadas pueden solicitar este paquete de acceso y que la configuración de la lista de permitidos o denegados de B2B tenga prioridad para todos los usuarios externos nuevos. |

    Una organización conectada es un dominio o directorio externo de Azure AD con el que tiene relación.

1. Si seleccionó **Organizaciones conectadas específicas**, haga clic en **Agregar directorios** para seleccionar en una lista de las organizaciones conectadas que el administrador haya agregado anteriormente.

1. Escriba el nombre o el nombre de dominio para buscar una organización anteriormente conectada.

    ![Paquete de acceso - Solicitudes - Seleccionar directorios](./media/active-directory-entitlement-management-request-policy/select-directories.png)

    Si la organización con la que desea colaborar no está en la lista, puede pedir al administrador que la agregue como una organización conectada. Para más información, consulte [Incorporación de una organización conectada](../articles/active-directory/governance/entitlement-management-organization.md).

1. Una vez que haya seleccionado todas las organizaciones conectadas, haga clic en **Seleccionar**.

    > [!NOTE]
    > Todos los usuarios de las organizaciones conectadas seleccionadas podrán solicitar este paquete de acceso. Esto incluye a los usuarios de Azure AD de todos los subdominios asociados a la organización, a menos que los dominios estén bloqueados por la lista de permitidos o denegados de Azure B2B. Para obtener más información, consulte [Allow or block invitations to B2B users from specific organizations](../articles/active-directory/b2b/allow-deny-list.md) (Permitir o bloquear invitaciones a usuarios de B2B procedentes de determinadas organizaciones).

1. Vaya a la sección [Aprobación](#approval).

## <a name="none-administrator-direct-assignments-only"></a>ninguno (solo para las asignaciones directas del administrador)

Siga estos pasos si quiere omitir las solicitudes de acceso y permitir a los administradores asignar directamente a usuarios específicos al paquete de acceso. Los usuarios no tendrán que solicitar el paquete de acceso. Aún podrá establecer la configuración del ciclo de vida, pero no hay ninguna configuración de solicitud.

1. En la sección **Usuarios que pueden solicitar acceso**, haga clic en **Ninguno (solo para las asignaciones directas del administrador)** .

    ![Paquete de acceso - Solicitudes - Ninguno (solo para las asignaciones directas del administrador)](./media/active-directory-entitlement-management-request-policy/none-admin-direct-assignments-only.png)

    Después de crear el paquete de acceso, puede asignar directamente usuarios internos y externos específicos al paquete de acceso. Si especifica un usuario externo, se creará una cuenta de usuario invitado en su directorio. Para información sobre la asignación directa de un usuario, consulte [Visualización, incorporación y eliminación de asignaciones en un paquete de acceso](../articles/active-directory/governance/entitlement-management-access-package-assignments.md).

1. Vaya a la sección [Habilitación de solicitudes](#enable-requests).

## <a name="approval"></a>Aprobación

En la sección Aprobación, especifique si se requiere una aprobación cuando los usuarios soliciten este paquete de acceso. La configuración de aprobación funciona de la siguiente manera:

- Una solicitud para una aprobación de una sola fase solo debe aprobarla uno de los aprobadores seleccionados o un aprobador de reserva. 
- Una solicitud para una aprobación de dos fases solo debe aprobarla uno de los aprobadores seleccionados de cada fase.
- El aprobador puede ser un administrador, un patrocinador interno o un patrocinador externo, dependiendo de quién rige el acceso a la directiva.
- Para una aprobación de una sola fase o de dos fases no se necesita la aprobación de cada uno de los aprobadores seleccionados.
- La decisión de aprobación se basa en el aprobador que revise primero la solicitud.

Para ver una demostración de cómo agregar aprobadores a una directiva de solicitud, vea el vídeo siguiente:

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4cZfg]

Para ver una demostración de cómo agregar una aprobación de varias fases a una directiva de solicitud, vea el vídeo siguiente:

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4d1Jw]

Siga estos pasos para especificar la configuración de aprobación de las solicitudes para el paquete de acceso:

1. Para requerir la aprobación de las solicitudes de los usuarios seleccionados, establezca el botón de alternancia **Requerir aprobación** en **Sí**. O bien, para que las solicitudes se aprueban automáticamente, establezca el botón de alternancia en **No**.

1. Para requerir que los usuarios proporcionen una justificación para solicitar el paquete de acceso, establezca la opción **R** en **Sí**.
    
1. Ahora determine si las solicitudes requerirán la aprobación de una sola fase o de dos fases. Establezca el botón de alternancia **Número de fases** en **1** para una aprobación de una sola fase o en **2**, para la aprobación de dos fases.

    ![Acceder a paquete - Solicitudes - Configuración de aprobación](./media/active-directory-entitlement-management-request-policy/approval.png)

Siga los pasos que se indican a continuación para agregar aprobadores después de seleccionar el número de fases que requiere: 

### <a name="single-stage-approval"></a>Aprobación de una sola fase

1. Agregue el **Primer aprobador**:
    
    Si la directiva está configurada para regir el acceso de los usuarios en el directorio, puede seleccionar **Administrador como aprobador**. O bien agregue un usuario específico con un clic en **Agregar aprobadores** después de seleccionar Elegir aprobadores específicos en el menú desplegable.
    
    ![Paquete de acceso - Solicitudes - Para usuarios en el directorio - Primer aprobador](./media/active-directory-entitlement-management-request-policy/approval-single-stage-first-approver-manager.png)

    Si esta directiva está configurada para regir el acceso de usuarios que no están en el directorio, puede seleccionar **Patrocinador externo** o **Patrocinador interno**. O bien agregue un usuario específico con un clic en **Agregar aprobadores** o grupos en Elegir aprobadores específicos.
    
    ![Paquete de acceso - Solicitudes - Para usuarios fuera del directorio - Primer aprobador](./media/active-directory-entitlement-management-request-policy/out-directory-first-approver.png)
    
1. Si seleccionó **Administrador** como el primer aprobador, haga clic en **Agregar reserva** para seleccionar uno o más usuarios o grupos del directorio como aprobador de reserva. Los aprobadores de reserva reciben la solicitud si la administración de derechos no encuentra al administrador del usuario que solicita el acceso.

    La administración de derechos encuentra al administrador mediante el atributo **Manager**. El atributo está en el perfil del usuario en Azure AD. Para más información, consulte [Incorporación o actualización de la información de perfil de un usuario mediante Azure Active Directory](../articles/active-directory/fundamentals/active-directory-users-profile-azure-portal.md).

1. Si seleccionó **Elegir aprobadores específicos**, haga clic en **Agregar aprobadores** para seleccionar uno o varios usuarios o grupos del directorio para que sean aprobadores.

1. En el cuadro bajo **¿Dentro de cuántos días es necesario tomar una decisión?** , especifique el número de días que un aprobador tiene que revisar una solicitud para este paquete de acceso.

    Si no se aprueba una solicitud durante este período de tiempo, se rechazará automáticamente. El usuario tendrá que enviar otra solicitud para el paquete de acceso.

1. Para requerir que los aprobadores justifiquen su decisión, establezca la opción Requerir justificación del aprobador en **Sí**.

    La justificación es visible para otros aprobadores y el solicitante.

### <a name="2-stage-approval"></a>Aprobación en 2 fases

Si seleccionó una aprobación de dos fases, deberá agregar otro aprobador.

1. Agregue el **Segundo aprobador**: 
    
    Si los usuarios están en el directorio, agregue un usuario específico como el segundo aprobador mediante un clic en **Agregar aprobadores** en Elegir aprobadores específicos.

    ![Paquete de acceso - Solicitudes - Para usuarios en el directorio - Segundo aprobador](./media/active-directory-entitlement-management-request-policy/in-directory-second-approver.png)

    Si los usuarios no están en el directorio, seleccione **Patrocinador interno** o **Patrocinador externo** como el segundo aprobador. Después de seleccionar el aprobador, agregue los aprobadores de reserva.

    ![Paquete de acceso - Solicitudes - Para usuarios fuera del directorio - Segundo aprobador](./media/active-directory-entitlement-management-request-policy/out-directory-second-approver.png) 

1. Especifique el número de días que tiene el segundo aprobador para aprobar la solicitud en el cuadro bajo **¿Dentro de cuántos días es necesario tomar una decisión?** 

1. Establezca el botón de alternancia Requerir justificación del aprobador en **Sí** o en **No**.

### <a name="alternate-approvers"></a>Aprobadores alternativos

Puede especificar aprobadores alternativos de manera similar a cómo se especifican el primer y el segundo aprobador que pueden aprobar solicitudes. Tener aprobadores alternativos lo ayudará a asegurarse de que las solicitudes se aprueban o deniegan antes de que expiren (tiempo de expiración). Puede enumerar como aprobadores alternativos el primer aprobador y el segundo aprobador para la aprobación de dos fases. 

Al especificar aprobadores alternativos, en caso de que el primer o el segundo aprobador no pueda aprobar ni rechazar la solicitud, la solicitud pendiente se reenvía a los aprobadores alternativos según la programación de reenvío que haya especificado durante la configuración de la directiva. Reciben un correo electrónico para aprobar o denegar la solicitud pendiente.

Después de reenviar la solicitud a los aprobadores alternativos, el primer o el segundo aprobador puede seguir aprobando o denegando la solicitud. Los aprobadores alternativos usan el mismo sitio Mi acceso para aprobar o denegar la solicitud pendiente.

Podemos enumerar personas o grupos de personas como aprobadores y aprobadores alternativos. Asegúrese de que se muestran distintos conjuntos de personas como el primer aprobador, el segundo aprobador y los aprobadores alternativos.
Por ejemplo, si especificó que Alice y Bob sean los primeros aprobadores, indique que Carol y Dave serán los aprobadores alternativos. Siga estos pasos para agregar aprobadores alternativos a un paquete de acceso:

1. En el Primer aprobador, el Segundo aprobador o en ambos, haga clic en **Mostrar configuración de solicitud avanzada**.

    ![Paquete de acceso - Directiva - Mostrar configuración de solicitud avanzada](./media/active-directory-entitlement-management-request-policy/alternate-approvers-click-advanced-request.png)

1. Establezca la opción **Si no se toman medidas, ¿quiere realizar el reenvío a los aprobadores alternativos?** en **Sí**.

1. Haga clic en **Agregar aprobadores alternativos** y seleccione los aprobadores alternativos de la lista.

    ![Paquete de acceso - Directiva - Agregar aprobadores alternativos](./media/active-directory-entitlement-management-request-policy/alternate-approvers-add.png)

1. En el cuadro **¿Dentro de cuántos días quiere realizar el reenvío a los aprobadores alternativos?** , escriba el número de días durante los cuales los aprobadores pueden aprobar o denegar una solicitud. Si ningún aprobador ha aprobado o denegado la solicitud en ese período, la solicitud expirará (tiempo de expiración) y el usuario deberá enviar otra solicitud para el paquete de acceso. 

    Las solicitudes solo se pueden reenviar a aprobadores alternativos un día después de que la duración de la solicitud llega a la mitad y la decisión de los aprobadores principales debe ser agotar el tiempo de espera después de 4 días como mínimo. Si el tiempo de espera de la solicitud es menor o igual que 3, no hay tiempo suficiente para reenviar la solicitud a los aprobadores alternativos. En este ejemplo, la duración de la solicitud es de 14 días. De este modo, la mitad de la duración de la solicitud es el día 7. Por lo tanto, la solicitud no se puede reenviar antes del día 8. Además, las solicitudes no se pueden reenviar el último día de la duración de la solicitud. Por lo tanto, en el ejemplo, la solicitud se puede reenviar a más tardar el día 13.

## <a name="enable-requests"></a>Habilitación de solicitudes

1. Si quiere que el paquete de acceso esté disponible de inmediato para los usuarios de la directiva de solicitud para que puedan solicitarlo, cambie el conmutador de alternancia Habilitar a **Sí**.

    Podrá habilitarla en todo momento cuando haya terminado de crear el paquete de acceso.

    Si seleccionó **Ninguno (solo para las asignaciones directas del administrador)** y establece la habilitación en **No**, los administradores no podrán asignar directamente este paquete de acceso.

    ![Paquete de acceso: directiva; configuración de habilitación de la directiva](./media/active-directory-entitlement-management-request-policy/enable-requests.png)

1. Haga clic en **Next**.

## <a name="add-requestor-information-preview-to-an-access-package"></a>Incorporación de la información del solicitante (versión preliminar) para un paquete de acceso

1. Vaya a la pestaña **Requestor information** (Información del solicitante) y haga clic en la subpestaña **Preguntas**.
 
1. Escriba lo que desea pedir al solicitante, lo cual también se conoce como cadena que se va a mostrar, para la pregunta del cuadro **Pregunta**.

    ![Paquete de acceso: Directiva Habilitar la configuración de la información del solicitante](./media/active-directory-entitlement-management-request-policy/add-requestor-info-question.png)

1. Si desea agregar sus propias opciones de localización, haga clic en **Add localization** (Agregar traducción).
    1. Una vez en el panel **Add localizations for question** (Agregar traducciones para la pregunta), seleccione el **código de idioma** para el idioma en el que va a traducir la pregunta.
    1. En el idioma configurado, escriba la pregunta en el cuadro **Localized Text** (Texto traducido).
    1. Una vez que haya agregado todas las traducciones necesarias, haga clic en **Guardar**.

    ![Paquete de acceso: Directiva Configurar texto traducido](./media/active-directory-entitlement-management-request-policy/add-localization-question.png)

1. Seleccione el **formato de respuesta** en el que desea que los solicitantes respondan. Entre los formatos de respuesta se incluyen: *texto breve*, *opciones múltiples* y *texto largo*.
 
    ![Paquete de acceso: Directiva Seleccionar ver y editar el formato de respuesta de opciones múltiples](./media/active-directory-entitlement-management-request-policy/answer-format-view-edit.png)
 
1. Si selecciona varias opciones, haga clic en el botón **Ver y editar** para configurar las opciones de respuesta.
    1. Después de seleccionar Ver y editar, se abrirá el panel **View/edit question** (Ver y editar pregunta).
    1. Escriba las opciones de respuesta que desea dar al solicitante al responder a la pregunta de los cuadros **Answer values** (Valores de respuesta).
    1. Escriba tantas respuestas como sea necesario y haga clic en **Save** (Guardar).
    
    ![Paquete de acceso: Directiva Escribir opciones múltiples](./media/active-directory-entitlement-management-request-policy/answer-multiple-choice.png)
  
1. Para requerir que los solicitantes respondan a esta pregunta al solicitar acceso a un paquete de acceso, haga clic en la casilla situada debajo de **Required** (Obligatorio).

1. Haga clic en Next (Siguiente).
