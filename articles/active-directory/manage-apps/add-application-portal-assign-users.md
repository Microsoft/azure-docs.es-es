---
title: 'Inicio rápido: Asignación de usuarios a una aplicación que usa Azure Active Directory como proveedor de identidades'
description: Esta guía de inicio rápido le guiará en el proceso de permitir que los usuarios usen una aplicación que ha configurado para usar Azure AD como proveedor de identidades.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 09/01/2020
ms.author: kenwith
ms.openlocfilehash: 9d396c0787798c783b44748dc2461c073334cc7e
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322507"
---
# <a name="quickstart-assign-users-to-an-app-that-is-using-azure-ad-as-an-identity-provider"></a>Inicio rápido: Asignación de usuarios a una aplicación que usa Azure AD como proveedor de identidades

En la guía de inicio rápido anterior, configuró las propiedades de una aplicación. Al establecer las propiedades, configuró la experiencia para los usuarios asignados y sin asignar. En esta guía de inicio rápido se explica el proceso de asignación de usuarios a la aplicación.

## <a name="prerequisites"></a>Requisitos previos

Para asignar usuarios a una aplicación que ha agregado al inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- Opcional: Haber finalizado el inicio rápido de [Visualización de las aplicaciones](view-applications-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Incorporación de una aplicación](add-application-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Configuración de una aplicación](add-application-portal-configure.md).

>[!IMPORTANT]
>Use un entorno que no sea de producción para probar los pasos de esta guía de inicio rápido.

## <a name="assign-users-to-an-app"></a>Asignación de usuarios a una aplicación
1. En el portal de Azure AD, seleccione **Aplicaciones empresariales**. Después, busque y seleccione la aplicación que desea configurar.
2. En el menú de navegación de la izquierda, seleccione **Usuarios y grupos**.
   > [!NOTE]
   > Algunas de las aplicaciones de Microsoft 365 y Office 365 requieren el uso de PowerShell. 
3. Seleccione el botón **Agregar usuario**.
4. En el panel **Agregar asignación**, seleccione **Usuarios y grupos**.
5. Seleccione el usuario o el grupo que desea asignar a la aplicación. También puede empezar a escribir el nombre del usuario o grupo en el cuadro de búsqueda. Puede elegir varios usuarios y grupos; sus selecciones aparecerán en **Elementos seleccionados**.
    > [!IMPORTANT]
    > Cuando se asigna un grupo a una aplicación, solo tendrán acceso los usuarios del grupo. La asignación no se aplica en cascada a los grupos anidados.

    > [!NOTE]
    > La asignación basada en grupos requiere Azure Active Directory Premium P1 o P2. La asignación basada en grupo se admite en grupos de seguridad únicamente. Actualmente no se admiten las pertenencias a grupos anidados y los grupos de Office 365. Para conocer más requisitos de licencia de las características abordadas en este artículo, vea la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory). 
6. Cuando termine, elija **Seleccionar**.
   ![Asignación de un usuario o un grupo a la aplicación](./media/assign-user-or-group-access-portal/assign-users.png)
7. En el panel **Usuarios y grupos**, seleccione uno o varios usuarios o grupos de la lista y, luego, haga clic en el botón **Seleccionar** en la parte inferior del panel.
8. Si la aplicación lo permite, puede asignar un rol al usuario o al grupo. En el panel **Agregar asignación**, elija **Seleccionar rol**. Después, en el panel **Seleccionar rol**, seleccione el rol que quiera aplicar a los usuarios o grupos escogidos y, luego, haga clic en **Aceptar** en la parte inferior del panel. 
    > [!NOTE]
    > Si la aplicación no permite seleccionar roles, se asignará el rol de acceso predeterminado. En este caso, la aplicación administrará el nivel de acceso que tienen los usuarios.
9. En el panel **Agregar asignación**, haga clic en el botón **Asignar** en la parte inferior del panel.

> [!NOTE]
> Puede anular la asignación de usuarios o grupos mediante el mismo procedimiento. Seleccione el usuario o el grupo cuya asignación desea anular y, a continuación, seleccione **Eliminar**. Algunas de las aplicaciones de Microsoft 365 y Office 365 requieren el uso de PowerShell. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con la guía de inicio rápido, considere la posibilidad de eliminar la aplicación. De este modo, podrá mantener limpio el inquilino de prueba. La eliminación de la aplicación se trata en el último inicio rápido de esta serie, consulte [Eliminación de una aplicación](delete-application-portal.md).

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo, donde aprenderá a configurar el inicio de sesión único en una aplicación.
> [!div class="nextstepaction"]
> [Configuración del inicio de sesión único](add-application-portal-setup-sso.md)
