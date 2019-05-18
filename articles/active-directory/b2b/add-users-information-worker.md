---
title: Incorporación de usuarios de colaboración B2B como trabajadores de la información a Azure Active Directory | Microsoft Docs
description: La colaboración B2B permite a los trabajadores de la información y a los propietarios de aplicaciones agregar usuarios invitados a Azure AD para el acceso | Microsoft Docs
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 12/19/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: e8606a0d4e203e1a910a5cd15ca83a622f5286bd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812537"
---
# <a name="how-users-in-your-organization-can-invite-guest-users-to-an-app"></a>¿Cómo pueden los usuarios de la organización invitar a usuarios invitados a una aplicación?

Una vez que se haya agregado un usuario invitado al directorio en Azure AD, un propietario de aplicación puede enviar a este un vínculo directo a la aplicación que desea compartir. Los administradores de Azure AD también pueden configurar la administración autoservicio para la galería o las aplicaciones basadas en SAML en su inquilino de Azure AD. De esta forma, los propietarios de aplicaciones puedan administrar sus propios usuarios invitados, incluso aunque los usuarios invitados no se hayan agregado al directorio aún. Si una aplicación está configurada para el autoservicio, el propietario de la aplicación usa su panel de acceso para invitar a un usuario invitado a una aplicación o para agregar un usuario invitado a un grupo que tiene acceso a la aplicación. La administración autoservicio de una galería o aplicación basada en SAML requiere algo de configuración inicial por parte de un administrador. El siguiente es un resumen de los pasos de configuración (para obtener instrucciones detalladas, consulte [Requisitos previos](#prerequisites) más adelante en esta página):

 - Habilite la administración de grupos de autoservicio para el inquilino
 - Cree un grupo para asignar a la aplicación y hacer que el usuario sea un propietario
 - Configure la aplicación para el autoservicio y asigne el grupo a la aplicación

> [!NOTE]
> En este artículo se describe cómo configurar la administración autoservicio para la galería y las aplicaciones basadas en SAML que ha agregado al inquilino de Azure AD. También puede [configurar grupos de autoservicio de Office 365](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-self-service-management) para que los usuarios puedan administrar el acceso a sus propios grupos de Office 365. Para conocer más formas en las que los usuarios pueden compartir sus archivos y aplicaciones, consulte [Acceso de invitado en Grupos de Office 365](https://support.office.com/article/guest-access-in-office-365-groups-bfc7a840-868f-4fd6-a390-f347bf51aff6) y [Compartir archivos de SharePoint o carpetas](https://support.office.com/article/share-sharepoint-files-or-folders-1fe37332-0f9a-4719-970e-d2578da4941c).

## <a name="invite-a-guest-user-to-an-app-from-the-access-panel"></a>Invitación a un usuario invitado a una aplicación desde el panel de acceso

Después de configurar una aplicación para el autoservicio, los propietarios de la aplicación pueden usar su propio panel de acceso para invitar a un usuario invitado a la aplicación que desean compartir. No es necesario que se agregue el usuario invitado a Azure AD con antelación. 

1. Para abrir el panel de acceso, vaya a `https://myapps.microsoft.com`.
2. Apunte a la aplicación, seleccione el icono de puntos suspensivos (**...**) y, a continuación, seleccione **Administrar aplicación**.
 
   ![Captura de pantalla muestra el menú secundario de administrar aplicaciones de la aplicación de Salesforce](media/add-users-iw/access-panel-manage-app.png)
 
3. En la parte superior de la lista de usuarios, seleccione **+**.
   
   ![Captura de pantalla muestra el signo más para agregar miembros a la aplicación](media/add-users-iw/access-panel-manage-app-add-user.png)
   
4. En el cuadro de búsqueda **Agregar miembros**, escriba la dirección de correo electrónico para el usuario invitado. Opcionalmente, puede incluir un mensaje de bienvenida.
   
   ![Captura de pantalla muestra el Agregar ventana miembros para agregar un invitado](media/add-users-iw/access-panel-invitation.png)
   
5. Seleccione **Agregar** para enviar una invitación al usuario invitado. Después de enviar la invitación, la cuenta de usuario se agrega automáticamente al directorio como invitado.

## <a name="invite-someone-to-join-a-group-that-has-access-to-the-app"></a>Invitar a alguien a unirse a un grupo que tiene acceso a la aplicación
Después de configurar una aplicación para el autoservicio, los propietarios de la aplicación pueden invitar a los usuarios a los grupos que ellos administran y que tienen acceso a las aplicaciones que desean compartir. No es necesario que los usuarios invitados ya existan en el directorio. El propietario de la aplicación debe seguir estos pasos para invitar a un usuario invitado al grupo para que pueda acceder a la aplicación.

1. Asegúrese de que es propietario del grupo de autoservicio que tiene acceso a la aplicación que desea compartir.
2. Para abrir el panel de acceso, vaya a `https://myapps.microsoft.com`.
3. Seleccione la aplicación **Grupos**.
   
   ![Captura de pantalla muestra la aplicación de grupos en el Panel de acceso](media/add-users-iw/access-panel-groups.png)
   
4. En **Grupos de mi propiedad**, seleccione el grupo que tiene acceso a la aplicación que desea compartir.
   
   ![Captura de pantalla que muestra dónde debe seleccionar un grupo en los grupos de mi propiedad](media/add-users-iw/access-panel-groups-i-own.png)
   
5. En la parte superior de la lista de miembros de grupos, seleccione **+**.
   
   ![Captura de pantalla muestra el signo más para agregar miembros al grupo](media/add-users-iw/access-panel-groups-add-member.png)
   
6. En el cuadro de búsqueda **Agregar miembros**, escriba la dirección de correo electrónico para el usuario invitado. Opcionalmente, puede incluir un mensaje de bienvenida.
   
   ![Captura de pantalla muestra el Agregar ventana miembros para agregar un invitado](media/add-users-iw/access-panel-invitation.png)
   
7. Seleccione **Agregar** para enviar automáticamente la invitación al usuario invitado. Después de enviar la invitación, la cuenta de usuario se agrega automáticamente al directorio como invitado.


## <a name="prerequisites"></a>Requisitos previos

La administración de autoservicio de aplicaciones requiere algo de configuración inicial por parte de un administrador global y un administrador de Azure AD. Como parte de esta configuración, deberá configurar la aplicación para el autoservicio y asignar un grupo a la aplicación que pueda administrar el propietario de la misma. También puede configurar el grupo para permitir que cualquier persona solicite la pertenencia, pero requiere la aprobación del propietario del grupo. (Más información acerca de la [administración de grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-self-service-management)). 

> [!NOTE]
> No puede agregar usuarios invitados a un grupo dinámico o a uno que se ha sincronizado con la instancia local de Active Directory.

### <a name="enable-self-service-group-management-for-your-tenant"></a>Habilite la administración de grupos de autoservicio para el inquilino
1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. Seleccione **Grupos**.
4. En **Configuración**, seleccione **General**.
5. En **Administración de grupos de autoservicio**, junto a **Los propietarios pueden administrar solicitudes de pertenencia a grupos en el Panel de acceso**, seleccione **Sí**.
6. Seleccione **Guardar**.

### <a name="create-a-group-to-assign-to-the-app-and-make-the-user-an-owner"></a>Cree un grupo para asignar a la aplicación y hacer que el usuario sea un propietario
1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global o administrador de Azure AD.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. Seleccione **Grupos**.
4. Seleccione **Nuevo grupo**.
5. En **Tipo de grupo**, seleccione **Seguridad**.
6. Escriba un **Nombre de grupo** y una **Descripción del grupo**.
7. En **Tipo de pertenencia**, seleccione **Asignado**.
8. Seleccione **Crear** y cierre la página **Grupo**.
9. En la página **Grupos - Todos los grupos**, abra el grupo. 
10. En **Administrar**, seleccione **Propietarios** > **Agregar propietarios**. Busque el usuario que debe administrar el acceso a la aplicación. Seleccione el usuario y luego haga clic en **Seleccionar**.

### <a name="configure-the-app-for-self-service-and-assign-the-group-to-the-app"></a>Configure la aplicación para el autoservicio y asigne el grupo a la aplicación
1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global o administrador de Azure AD.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. En **Administrar**, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
4. En la lista de aplicaciones, busque y abra la aplicación.
5. En **Administrar**, seleccione **Inicio de sesión único** y configure la aplicación para el inicio de sesión único. (Para más información, consulte [Administración del inicio de sesión único para aplicaciones empresariales](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-portal)).
6. En **Administrar**, seleccione **Autoservicio** y configure el acceso a la aplicación de autoservicio. (Para más información, consulte [Uso del acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/application-access-panel-self-service-applications-how-to)). 

    > [!NOTE]
    > En la opción **¿A qué grupo se deberían agregar los usuarios asignados?** seleccione el grupo que creó en la sección anterior.
7. En **Administrar**, seleccione **Usuarios y grupos** y compruebe que el grupo de autoservicio que creó aparece en la lista.
8. Para agregar la aplicación al panel de acceso del propietario del grupo, seleccione **Agregar usuario** > **Usuarios y grupos**. Busque al propietario del grupo y seleccione el usuario, haga clic en **Seleccionar** y luego haga clic en **Asignar** para agregar el usuario a la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [¿Cómo agregan los administradores de Azure Active Directory usuarios de colaboración B2B?](add-users-administrator.md)
- [Canje de invitación de colaboración B2B](redemption-experience.md)
- [Concesión de licencias de colaboración B2B de Azure AD](licensing-guidance.md)
