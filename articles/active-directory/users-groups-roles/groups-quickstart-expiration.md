---
title: Inicio rápido de directiva de expiración de grupo - Azure AD | Microsoft Docs
description: Expiración de grupos de Microsoft 365 en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 178f6a2c1a71d088eeb1a3ea6f7e39198d401ff4
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213723"
---
# <a name="quickstart-set-microsoft-365-groups-to-expire-in-azure-active-directory"></a>Inicio rápido: Establecimiento de grupos de Microsoft 365 para que expiren en Azure Active Directory

En este inicio rápido, puede establecer la directiva de expiración de los grupos de Microsoft 365. Cuando los usuarios pueden configurar sus propios grupos, el número de grupos sin uso se puede multiplicar. Una manera de administrar los grupos sin uso es establecer la expiración de dichos grupos, para reducir el mantenimiento de la eliminación manual de grupos.

La directiva de expiración es sencilla:

- Los grupos con actividades de usuario se renuevan automáticamente a medida que se aproxima la expiración.
- Los propietarios del grupo reciben una notificación para renovar un grupo que va a expirar.
- Se eliminará cualquier grupo que no se renueve.
- El propietario de un grupo o un administrador de Azure AD pueden restaurar un grupo de Microsoft 365 eliminado al cabo de 30 días.

> [!NOTE]
> Ahora, los grupos usan la inteligencia de Azure AD para que se renueven automáticamente en función de si se han usado recientemente. Esta decisión de renovación se basa en la actividad de los usuarios en grupos en los servicios de Office 365 como Outlook, SharePoint, Teams, Yammer y otros.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisite"></a>Requisito previo

 Para configurar la expiración de grupos, es necesario contar como mínimo con el rol de Usuario administrador en la organización.

## <a name="turn-on-user-creation-for-groups"></a>Activación de la creación de grupos por el usuario

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de Usuario administrador.

2. Seleccione **Grupos** y, a continuación, seleccione **General**.
  
   ![Página de configuración de autoservicio de grupos](./media/groups-quickstart-expiration/self-service-settings.png)

3. Establezca **Users can create Microsoft 365 groups** (Los usuarios pueden crear grupos de Microsoft 365) en **Sí**.

4. Seleccione **Guardar** para guardar la configuración de grupos cuando haya terminado.

## <a name="set-group-expiration"></a>Establecimiento de la expiración de grupo

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory** > **Grupos** > **Expiración** para abrir la configuración de expiración.
  
   ![Página de configuración de expiración del grupo](./media/groups-quickstart-expiration/expiration-settings.png)

2. Establezca el intervalo de expiración. Seleccione un valor preestablecido o escriba un valor personalizado superior a 31 días. 

3. Especifique una dirección de correo electrónico a la que deben enviarse las notificaciones de expiración cuando un grupo no tiene propietario.

4. En este inicio rápido, establezca **Habilite la expiración de estos grupos de Microsoft 365** en **Todos**.

5. Seleccione **Guardar** para guardar la configuración de expiración cuando haya terminado.

Eso es todo. En este inicio rápido, ha establecido correctamente la directiva de expiración de los grupos de Microsoft 365 seleccionados.

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="to-remove-the-expiration-policy"></a>Para eliminar la directiva de expiración, siga estos pasos:

1. Asegúrese de que ha iniciado sesión en [Azure Portal](https://portal.azure.com) con una cuenta que sea administrador global de la organización de Azure AD.
2. Seleccione **Azure Active Directory** > **Grupos** > **Expiración**.
3. Establezca **Habilite la expiración de estos grupos de Microsoft 365** en **Ninguno**.

### <a name="to-turn-off-user-creation-for-groups"></a>Para desactivar la creación de grupos por el usuario, siga estos pasos:

1. Seleccione **Azure Active Directory** > **Grupos** > **General**. 
2. Establezca **Users can create Microsoft 365 groups in Azure portals** (Los usuarios pueden crear grupos de Microsoft 365 en Azure Portal) en **No**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la expiración, incluidas las instrucciones de PowerShell y las limitaciones técnicas, consulte el artículo siguiente:

> [!div class="nextstepaction"]
> [Expiration policy PowerShell](groups-lifecycle.md) (Directiva de expiración de PowerShell)
