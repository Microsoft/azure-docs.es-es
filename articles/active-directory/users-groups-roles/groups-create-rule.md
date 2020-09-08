---
title: 'Creación o edición de un grupo dinámico y obtención del estado: Azure AD | Microsoft Docs'
description: Cómo crear o actualizar reglas de pertenencia a grupos en Azure Portal y comprobar el estado de procesamiento.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f5c44f72021fa46352ae6ded8fe671ea4f1a4d60
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797180"
---
# <a name="create-or-update-a-dynamic-group-in-azure-active-directory"></a>Creación o actualización de un grupo dinámico en Azure Active Directory

En Azure Active Directory (Azure AD), puede usar reglas para determinar la pertenencia a grupos según las propiedades de usuario o dispositivo. En este artículo se explica cómo configurar una regla para un grupo dinámico en Azure Portal.
La pertenencia dinámica es compatible con grupos de seguridad o grupos de Microsoft 365. Cuando se aplica una regla de pertenencia a grupos, se evalúan los atributos de usuario y dispositivo para ver si coinciden con la regla de pertenencia. Cuando cambian los atributos de un usuario o dispositivo, se procesan todos los cambios de pertenencia de las reglas de grupo dinámico de la organización. Los usuarios y dispositivos se agregan o quitan si cumplen las condiciones de un grupo. Los grupos de seguridad se pueden aplicar a dispositivos o usuarios, pero los grupos de Office 365 solo pueden ser grupos de usuarios.

## <a name="rule-builder-in-the-azure-portal"></a>Generador de reglas en Azure Portal

Azure AD proporciona un generador de reglas para crear y actualizar las reglas importantes con mayor rapidez. El generador de reglas admite la construcción de hasta cinco expresiones. Facilita la creación de reglas con unas cuantas expresiones sencillas; no obstante, no se puede usar para reproducir todas las reglas. En caso de que no admita la regla que quiere crear, puede usar el cuadro de texto.

Estos son algunos ejemplos de reglas o sintaxis avanzadas para las que se recomienda construir mediante el cuadro de texto:

- Regla con más de cinco expresiones
- La regla de subordinados directos
- Configuración de la [precedencia de operadores](groups-dynamic-membership.md#operator-precedence)
- [Reglas con expresiones complejas](groups-dynamic-membership.md#rules-with-complex-expressions); por ejemplo, `(user.proxyAddresses -any (_ -contains "contoso"))`

> [!NOTE]
> Es posible que el generador de reglas no pueda mostrar algunas reglas construidas en el cuadro de texto. En este caso, podría aparecer un mensaje. El generador de reglas no cambia la sintaxis admitida, la validación ni el procesamiento de reglas de grupos dinámicos de ninguna manera.

![Adición de una regla de pertenencia a un grupo dinámico](./media/groups-create-rule/update-dynamic-group-rule.png)

Para ver ejemplos de sintaxis, propiedades admitidas, operadores y valores de una regla de pertenencia, consulte [Reglas de pertenencia dinámica a grupos de Azure Active Directory](groups-dynamic-membership.md).

## <a name="to-create-a-group-membership-rule"></a>Para crear una regla de pertenencia a grupo, siga estos pasos:

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta que tenga el rol de administrador global, administrador de Intune o administrador de usuarios en la organización de Azure AD.
1. Busque y seleccione **Grupos**.
1. Seleccione **Todos los grupos** y, luego, **Nuevo grupo**.

   ![Selección del comando para agregar nuevo grupo](./media/groups-create-rule/create-new-group-azure-active-directory.png)

1. En la página **Grupo**, escriba un nombre y una descripción para el nuevo grupo. Seleccione un **tipo de pertenencia** para los usuarios o dispositivos y, luego, seleccione **Agregar una consulta dinámica**. El generador de reglas admite hasta cinco expresiones. Para agregar más de cinco expresiones, debe usar el cuadro de texto.

   ![Adición de una regla de pertenencia a un grupo dinámico](./media/groups-create-rule/add-dynamic-group-rule.png)

1. Para ver las propiedades de extensión personalizadas disponibles para su consulta de pertenencia, siga estos pasos:
   1. Seleccione **Obtener las propiedades de extensión personalizadas**.
   1. Escriba el identificador de aplicación y, luego, seleccione **Actualizar propiedades**.
1. Después de crear la regla, seleccione **Guardar**.
1. Seleccione **Crear** en la página **Nuevo grupo** para crear el grupo.

Si la regla que escribió no es válida, aparece una explicación de por qué no se pudo procesar la regla en una notificación de Azure en el portal. Léala con cuidado para saber cómo corregir la regla.

## <a name="to-update-an-existing-rule"></a>Para actualizar una regla existente

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta con el rol de administrador global, administrador de grupos, administrador de Intune o administrador de usuarios en la organización de Azure AD.
1. Seleccione **Grupos** > **Todos los grupos**.
1. Seleccione un grupo para abrir su perfil.
1. En la página de perfil del grupo, seleccione **Reglas de pertenencia dinámica**. El generador de reglas admite hasta cinco expresiones. Para agregar más de cinco expresiones, debe usar el cuadro de texto.

   ![Adición de una regla de pertenencia a un grupo dinámico](./media/groups-create-rule/update-dynamic-group-rule.png)

1. Para ver las propiedades de extensión personalizadas disponibles para su regla de pertenencia, siga estos pasos:
   1. Seleccione **Obtener las propiedades de extensión personalizadas**.
   1. Escriba el identificador de aplicación y, luego, seleccione **Actualizar propiedades**.
1. Después de actualizar la regla, seleccione **Guardar**.

## <a name="turn-on-or-off-welcome-email"></a>Activación o desactivación del correo electrónico de bienvenida

Cuando se crea un nuevo grupo de Microsoft 365, se envía una notificación de bienvenida por correo a los usuarios que se agregaron al grupo. Más tarde, si cambian los atributos de un usuario o dispositivo, se procesan los cambios de pertenencia de todas las reglas de grupo dinámico de la organización. Los usuarios que se agregan también reciben la notificación de bienvenida. Este comportamiento se puede desactivar en [Exchange PowerShell](/powershell/module/exchange/users-and-groups/Set-UnifiedGroup?view=exchange-ps).

## <a name="check-processing-status-for-a-rule"></a>Comprobación del estado de procesamiento de una regla

Puede ver el estado de procesamiento de la pertenencia y la última fecha actualizada en la página **Información general** del grupo dinámico.
  
  ![visualización del estado del grupo dinámico](./media/groups-create-rule/group-status.png)

Los mensajes de estado siguientes se pueden mostrar para el estado de **procesamiento de la pertenencia**:

- **Evaluando**:  se ha recibido el cambio de grupo y se están evaluando las actualizaciones.
- **Procesamiento**: las actualizaciones se están procesando.
- **Actualización completada**: se ha completado el procesamiento y se han realizado todas las actualizaciones aplicables.
- **Error de procesamiento**:  no se pudo completar el procesamiento debido a un error al evaluar la regla de pertenencia.
- **Actualización en pausa**: el administrador han pausado las actualizaciones de la regla de pertenencia dinámica. MembershipRuleProcessingState se establece en "Paused".

Los mensajes de estado siguientes se pueden mostrar para el estado **Última actualización de la pertenencia**:

- &lt;**Fecha y hora**&gt;: la última vez que se actualizó la pertenencia.
- **En curso**: las actualizaciones están actualmente en curso.
- **Desconocido**: no se puede recuperar la hora de la última actualización. El grupo podría ser nuevo.

Si se produce un error al procesar la regla de pertenencia para un grupo específico, se muestra una alerta en la parte superior de la **página de información general** del grupo. Si no se puede procesar ninguna actualización de pertenencia dinámica pendiente para todos los grupos de la organización durante más de 24 horas, se muestra una alerta en la parte superior de **Todos los grupos**.

![alertas de mensajes de error de procesamiento](./media/groups-create-rule/processing-error.png)

En estos artículos se proporciona información adicional sobre los grupos en Azure Active Directory.

- [Consulta de los grupos existentes](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Crear un nuevo grupo y agregar miembros](../fundamentals/active-directory-groups-create-azure-portal.md)
- [Administración de la configuración de un grupo](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Administrar la pertenencia a grupos](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Administrar reglas dinámicas de los usuarios de un grupo](groups-dynamic-membership.md)