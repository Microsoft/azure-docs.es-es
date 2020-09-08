---
title: Administración de usuarios excluidos de las directivas de acceso condicional - Azure AD
description: Aprenda a usar revisiones de acceso de Azure Active Directory (Azure AD) para administrar los usuarios a los que se les ha excluido de las directivas de acceso condicional.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 06/17/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: def74cccb3ad2c45a18210cc9d5a0d7ff0ba76f6
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783695"
---
# <a name="use-azure-ad-access-reviews-to-manage-users-excluded-from-conditional-access-policies"></a>Uso de revisiones de acceso de Azure AD para administrar los usuarios a los que se les ha excluido de las directivas de acceso condicional

En un mundo ideal, todos los usuarios seguirían las directivas de acceso para proteger el acceso a los recursos de la organización. A veces, sin embargo, hay casos empresariales en los que hay que hacer excepciones. En este artículo se incluyen algunos ejemplos de situaciones en las que es posible que se necesiten exclusiones. Usted, como administrador de TI, puede administrar esta tarea, evitar la omisión de las excepciones de directivas y proporcionar a los auditores pruebas de que tales excepciones se revisan con regularidad mediante las revisiones de acceso de Azure Active Directory (Azure AD).

>[!NOTE]
> Se requiere una licencia válida de Azure AD Premium P2, Enterprise Mobility + Security E5 de pago, o una licencia de prueba para usar las revisiones de acceso de Azure AD. Para obtener más información, consulte [Ediciones de Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="why-would-you-exclude-users-from-policies"></a>¿En qué casos hay que excluir a usuarios de las directivas?

Supongamos que, como administrador, decide usar el [acceso condicional de Azure AD](../conditional-access/concept-conditional-access-policy-common.md) para requerir Multi-Factor Authentication (MFA) y limitar las solicitudes de autenticación a redes o dispositivos específicos. Durante la planeación de la implementación, se da cuenta de que no todos los usuarios pueden cumplir estos requisitos. Por ejemplo, puede que tenga usuarios que trabajan desde oficinas remotas y que no forman parte de la red interna. También puede que tenga que dar cabida a los usuarios que se conectan mediante dispositivos no compatibles mientras esperan a que se reemplacen esos dispositivos. En resumen, la empresa necesita que estos usuarios inicien sesión y hagan su trabajo, de modo que los excluye de las directivas de acceso condicional.

En otro ejemplo, podría usar [ubicaciones con nombre](../conditional-access/location-condition.md) en el acceso condicional para especificar un conjunto de países y regiones desde los que no quiere permitir que los usuarios accedan a su inquilino.

![Ubicaciones con nombre en el acceso condicional](./media/conditional-access-exclusion/named-locations.png)

Pero algunos usuarios todavía podrían tener un motivo válido para iniciar sesión desde estos países o regiones bloqueados. Por ejemplo, los usuarios podrían viajar por trabajo y necesitar acceso a los recursos corporativos. En este caso, la directiva de acceso condicional para bloquear estos países o regiones podría usar un grupo de seguridad en la nube dedicado a los usuarios excluidos de la directiva. Los usuarios que necesiten acceder mientras viajan, pueden agregarse a sí mismos al grupo mediante la [administración de grupos de autoservicio de Azure AD](../users-groups-roles/groups-self-service-management.md).

Otro caso podría ser el de una directiva de acceso condicional que [bloquea la autenticación heredada para la mayoría de los usuarios](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/07/azure-ad-conditional-access-support-for-blocking-legacy-auth-is-in-public-preview/). No obstante, si algunos usuarios tienen que usar métodos de autenticación heredada para acceder a los recursos a través de Office 2010 o de clientes basados en IMAP/SMTP/POP, puede excluir a estos usuarios de la directiva que bloquea estos métodos de autenticación.

>[!NOTE]
>Microsoft recomienda encarecidamente que bloquee el uso de protocolos heredados en el inquilino para mejorar su seguridad.

## <a name="why-are-exclusions-challenging"></a>¿Por qué constituyen un reto las exclusiones?

En Azure AD, puede definir el ámbito de una directiva de acceso condicional para un conjunto de usuarios. También puede configurar exclusiones mediante la selección de roles, usuarios individuales o invitados de Azure AD. Tenga en cuenta que, cuando se configuran exclusiones, la intención de la directiva no se puede aplicar a los usuarios excluidos. Si las exclusiones se configuran mediante una lista de usuarios o grupos de seguridad locales heredados, tendrá una visibilidad limitada de las exclusiones. Como resultado:

- Es posible que los usuarios no sepan que están excluidos.

- Los usuarios pueden unirse al grupo de seguridad para omitir la directiva.

- Es posible que los usuarios excluidos cumplieran los requisitos para la exclusión, pero que ya no lo hagan.

Con frecuencia, cuando se configura por primera vez una exclusión, hay una pequeña lista de usuarios que omiten la directiva. Con el tiempo, se agregan más y más usuarios a la exclusión, y la lista crece. En algún momento, necesitará revisar la lista y confirmar que cada uno de estos usuarios debe seguir excluido. La administración de la lista de exclusiones desde un punto de vista técnico puede ser relativamente sencilla, pero ¿quién toma las decisiones empresariales y cómo puede asegurarse de que todo es auditable? Sin embargo, si configura la exclusión mediante un grupo de Azure AD, puede usar las revisiones de acceso como un control de compensación para aumentar la visibilidad y reducir el número de usuarios excluidos.

## <a name="how-to-create-an-exclusion-group-in-a-conditional-access-policy"></a>Creación de un grupo de exclusión en una directiva de acceso condicional

Siga estos pasos para crear un grupo de Azure AD y una directiva de acceso condicional que no se aplique a ese grupo.

### <a name="create-an-exclusion-group"></a>Creación de un grupo de exclusión

1. Inicie sesión en Azure Portal.

2. En el panel de navegación izquierdo, haga clic en **Azure Active Directory** y, después, en **Grupos**.

3. En el menú superior, haga clic en **Nuevo grupo** para abrir el panel de grupo.

4. En la lista **Tipo de grupo**, seleccione **Seguridad**. Especifique un nombre y una descripción.

5. Asegúrese de establecer el tipo de **Pertenencia** en **Asignado**.

6. Seleccione los usuarios que deben formar parte de este grupo de exclusión y, a continuación, haga clic en **Crear**.

![Panel de nuevo grupo en Azure Active Directory](./media/conditional-access-exclusion/new-group.png)

### <a name="create-a-conditional-access-policy-that-excludes-the-group"></a>Creación de una directiva de acceso condicional que excluye al grupo

Ahora puede crear una directiva de acceso condicional que use este grupo de exclusión.

1. En el panel de navegación izquierdo, haga clic en **Azure Active Directory** y, luego, en **Acceso condicional** para abrir la hoja **Directivas**.

2. Haga clic en **Nueva directiva** para que se abra el panel **Nuevo**.

3. Especifique un nombre.

4. En Asignaciones, haga clic en **Usuarios y grupos**.

5. En la pestaña **Incluir**, seleccione **Todos los usuarios**.

6. En la pestaña **Excluir**, agregue una marca de verificación en **Usuarios y grupos** y, a continuación, haga clic en **Seleccionar usuarios excluidos**.

7. Seleccione el grupo de exclusión que creó.

   > [!NOTE] 
   > Se recomienda excluir al menos una cuenta de administrador de la directiva cuando realice pruebas para asegurarse de que no está bloqueado en el inquilino.

8. Continúe con la configuración de la directiva de acceso condicional basada en los requisitos de su organización.

![Selección del panel de usuarios excluidos del acceso condicional](./media/conditional-access-exclusion/select-excluded-users.png)
  
Vamos a ver dos ejemplos en los que puede usar las revisiones de acceso para administrar exclusiones en las directivas de acceso condicional.

## <a name="example-1-access-review-for-users-accessing-from-blocked-countriesregions"></a>Ejemplo 1: Revisión de acceso para usuarios que acceden desde países o regiones bloqueados

Supongamos que tiene una directiva de acceso condicional que bloquea el acceso desde determinados países o regiones. Incluye un grupo que está excluido de la directiva. Esta es una revisión de acceso recomendada en la que se revisan los miembros del grupo.

> [!NOTE] 
> Se necesita un rol de administrador de empresa o administrador de usuarios para crear revisiones de acceso.

1. La revisión se realizará cada semana.

2. Esta revisión no terminará nunca para asegurarse de que siempre mantiene actualizado este grupo de exclusión.

3. Todos los miembros de este grupo estarán en el ámbito de la revisión.

4. Cada usuario necesitará dar fe de que aún necesita acceso desde estos países o regiones bloqueados, por lo que necesitará seguir siendo miembro del grupo.

5. Si el usuario no responde a la solicitud de revisión, se le quitará automáticamente del grupo y, por tanto, ya no podrá acceder al inquilino mientras esté de viaje por esos países o regiones.

6. Habilite las notificaciones de correo para comunicar a los usuarios el inicio y la finalización de la revisión de acceso.

    ![Creación de un panel de revisión de acceso para el ejemplo 1](./media/conditional-access-exclusion/create-access-review-1.png)

## <a name="example-2-access-review-for-users-accessing-with-legacy-authentication"></a>Ejemplo 2: Revisión de acceso para aquellos usuarios que acceden con autenticación heredada

Supongamos que tiene una directiva de acceso condicional que bloquea el acceso a los usuarios que usan la autenticación heredada y versiones anteriores del cliente y que incluye un grupo excluido de la directiva. Esta es una revisión de acceso recomendada en la que se revisan los miembros del grupo.

1. Esta revisión debería ser una revisión periódica.

2. Se debería revisar a todos los miembros del grupo.

3. Se puede configurar para que aparezcan los propietarios de las unidades de negocio como los revisores seleccionados.

4. Aplique automáticamente los resultados y quite a los usuarios a los que no se les ha autorizado a que sigan usando los métodos de autenticación heredada.

5. Podría ser conveniente habilitar las recomendaciones para que los revisores de grupos grandes pudieran tomar fácilmente sus decisiones.

6. Habilite las notificaciones de correo electrónico para que los usuarios reciban notificaciones sobre el inicio y finalización de la revisión de acceso.

    ![Creación de un panel de revisión de acceso para el ejemplo 2](./media/conditional-access-exclusion/create-access-review-2.png)

>[!IMPORTANT] 
>Si tiene muchos grupos de exclusión y, por consiguiente, necesita crear varias revisiones de acceso, existe ahora una API en el punto de conexión beta de Microsoft Graph que le permite crear y administrar los grupos mediante programación. Para empezar, puede consultar la [referencia sobre la API de revisiones de acceso de Azure AD](/graph/api/resources/accessreviews-root?view=graph-rest-beta) y el [ejemplo de recuperación de revisiones de acceso de Azure AD mediante Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/td-p/236096).

## <a name="access-review-results-and-audit-logs"></a>Resultados de la revisión de acceso y registros de auditoría

Ahora que ya tiene todo en marcha, el grupo, la directiva de acceso condicional y las revisiones de acceso, es hora de supervisar y hacer un seguimiento de los resultados de estas revisiones.

1. En Azure Portal, abra la hoja **Revisiones de acceso**.

2. Abra el control y el programa que ha creado para administrar el grupo de exclusión.

3. Haga clic en **Resultados** para ver a quién se le autorizó a permanecer en la lista y quién se ha quitado.

    ![Los resultados de las revisiones de acceso muestran quién ha recibido aprobación](./media/conditional-access-exclusion/access-reviews-results.png)

4. A continuación, haga clic en **Registros de auditoría** para ver las acciones que se realizaron durante esta revisión.

    ![Registros de auditoría de revisiones de acceso que muestran acciones](./media/conditional-access-exclusion/access-reviews-audit-logs.png)

Como administrador de TI, sabrá que la administración de grupos de exclusión de las directivas a veces es inevitable. Sin embargo, el mantenimiento de los grupos, la revisión regular de estos por parte del propietario de la empresa o de los propios usuarios, y la auditoría de estos cambios se puede realizar más fácilmente con las revisiones de acceso de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una revisión de acceso de grupos o aplicaciones](create-access-review.md)
- [¿Qué es el acceso condicional en Azure Active Directory?](../conditional-access/overview.md)