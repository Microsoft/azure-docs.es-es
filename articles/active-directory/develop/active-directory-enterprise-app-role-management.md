---
title: Configuración de la notificación de rol emitida en el token SAML para aplicaciones empresariales en Azure AD | Microsoft Docs
description: Aprenda a configurar la notificación de rol emitida en el token SAML para aplicaciones empresariales en Azure Active Directory
services: active-directory
documentationcenter: ''
author: jeevansd
manager: CelesteDG
editor: ''
ms.assetid: eb2b3741-3cde-45c8-b639-a636f3df3b74
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/22/2019
ms.author: jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 15165bce70a9bc2fbf3eb840ca8bce4fd5073280
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544628"
---
# <a name="how-to-configure-the-role-claim-issued-in-the-saml-token-for-enterprise-applications"></a>Procedimientos para: Configuración de la notificación de rol emitida en el token SAML para aplicaciones empresariales

Mediante Azure Active Directory (Azure AD) se puede personalizar el tipo de notificación de la notificación de rol en el token de respuesta que se recibe después de autorizar una aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure AD con configuración de directorios.
- Una suscripción que tenga el inicio de sesión único (SSO) habilitado. El inicio de sesión único se debe configurar con la aplicación.

## <a name="when-to-use-this-feature"></a>Cuándo usar esta característica

Si la aplicación espera que se pasen roles personalizados en una respuesta de SAML, es preciso usar esta característica. Puede crear tantos roles como sea necesario devolver desde Azure AD a la aplicación.

## <a name="create-roles-for-an-application"></a>Creación de roles para una aplicación

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione el icono **Azure Active Directory**.

    ![Icono de Azure Active Directory][1]

2. Seleccione **Aplicaciones empresariales**. Después, seleccione **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales][2]

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del cuadro de diálogo.

    ![Botón "Nueva aplicación"][3]

4. En el cuadro de búsqueda, escriba el nombre de la aplicación y seleccione la aplicación en el panel de resultados. Seleccione el botón **Agregar** para agregar la aplicación.

    ![Aplicación en la lista de resultados](./media/active-directory-enterprise-app-role-management/tutorial_app_addfromgallery.png)

5. Después de agregar la aplicación, vaya a la página **Propiedades** y copie el identificador del objeto.

    ![Página de propiedades](./media/active-directory-enterprise-app-role-management/tutorial_app_properties.png)

6. Abra el [Probador de Azure AD Graph](https://developer.microsoft.com/graph/graph-explorer) en otra ventana y siga estos pasos:

     a. Inicie sesión en el sitio del Probador de Graph con las credenciales de administrador o coadministrador global del inquilino.

    b. Para crear los roles es preciso tener permisos suficientes. Seleccione **Modificar permisos** para obtener los permisos.

      ![Botón "Modificar permisos"](./media/active-directory-enterprise-app-role-management/graph-explorer-new9.png)

    c. Seleccione los siguientes permisos en la lista (si no los tiene ya) y seleccione **Modificar permisos**.

      ![Lista de permisos y botón "Modificar permisos"](./media/active-directory-enterprise-app-role-management/graph-explorer-new10.png)

    > [!Note]
    > El rol de administrador de aplicaciones en la nube y de administrador de aplicaciones no funcionará en este escenario dado que se necesitan los permisos de administrador global para la lectura y escritura de directorios.

    d. Acepte el consentimiento. Ha vuelto a iniciar sesión en el sistema.

    e. Cambie la versión a **beta** y recupere la lista de entidades de servicio del inquilino con la siguiente consulta:

     `https://graph.microsoft.com/beta/servicePrincipals`

      Si usa varios directorios, siga este patrón: `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

      ![Cuadro de diálogo del Probador de Graph, con la consulta para capturar entidades de servicio](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

      > [!Note]
      > Ya estamos en proceso de actualizar las API para que los clientes puedan ver alguna interrupción en el servicio.

    f. En la lista de entidades de servicio capturadas, seleccione la que necesita modificar. También puede usar Ctrl + F para buscar la aplicación en todas las entidades de servicio enumeradas. Busque el identificador de objeto que copió de la página **Propiedades** y use la consulta siguiente para acceder a la entidad de servicio:

      `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

      ![Consulta para obtener la entidad de servicio que necesita modificar](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

    g. Extraiga la propiedad **appRoles** del objeto de entidad de servicio.

      ![Detalles de la propiedad appRoles](./media/active-directory-enterprise-app-role-management/graph-explorer-new3.png)

      > [!Note]
      > Si usa la aplicación personalizada (no la aplicación de Azure Marketplace), verá dos roles predeterminados: user y msiam_access. En el caso de la aplicación Marketplace, msiam_access es el único rol predeterminado. No es necesario realizar ningún cambio en los roles predeterminados.

    h. Genere roles nuevos para la aplicación.

      El siguiente JSON siguiente es un ejemplo del objeto **appRoles**. Cree un objeto similar para agregar los roles que desee para la aplicación.

      ```
      {
         "appRoles": [
          {
              "allowedMemberTypes": [
                  "User"
              ],
              "description": "msiam_access",
              "displayName": "msiam_access",
              "id": "b9632174-c057-4f7e-951b-be3adc52bfe6",
              "isEnabled": true,
              "origin": "Application",
              "value": null
          },
          {
              "allowedMemberTypes": [
                  "User"
              ],
              "description": "Administrators Only",
              "displayName": "Admin",
              "id": "4f8f8640-f081-492d-97a0-caf24e9bc134",
              "isEnabled": true,
              "origin": "ServicePrincipal",
              "value": "Administrator"
          }
      ]
      }
      ```

      > [!Note]
      > Solo puede agregar nuevos roles, además de msiam_access, para la operación de revisión. Además, puede agregar tantos roles como su organización necesite. Azure AD enviará el valor de estos roles como valor de notificación en la respuesta de SAML. Para generar los valores del GUID del identificador de los nuevos roles use herramientas web como [esta](https://www.guidgenerator.com/)

    i. Vuelva al Probador de Graph y cambie el método de **GET** a **PATCH**. Aplique la revisión al objeto de entidad de servicio para obtener los roles deseados mediante la actualización de la propiedad **appRoles** para que sea similar a la del ejemplo anterior. Seleccione **Ejecutar consulta** para ejecutar la operación de aplicación de revisión. Un mensaje confirma la creación del rol.

      ![Operación de aplicación de revisión con mensaje de resultado correcto](./media/active-directory-enterprise-app-role-management/graph-explorer-new11.png)

7. Una vez que haya revisado la entidad de servicio con más roles, podrá asignar usuarios a los roles correspondientes. Para asignar los usuarios, vaya al portal y, después, a la aplicación. Seleccione la pestaña **Usuarios y grupos**. Esta pestaña enumera todos los usuarios y grupos que ya están asignados a la aplicación. Puede agregar nuevos usuarios a los nuevos roles. También puede seleccionar un usuario existente y seleccionar **Editar** para cambiar el rol.

    ![Pestaña "Usuarios y grupos"](./media/active-directory-enterprise-app-role-management/graph-explorer-new5.png)

    Para asignar el rol a cualquier usuario, seleccione el rol nuevo y seleccione el botón **Asignar** de la parte inferior de la página.

    ![Paneles "Editar asignación" y "Seleccionar rol"](./media/active-directory-enterprise-app-role-management/graph-explorer-new6.png)

    > [!Note]
    > Para ver los nuevos roles es preciso actualizar la sesión en Azure Portal.

8. Actualice la tabla **Atributos** para definir una asignación personalizada de la notificación de rol.

9. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:

    | Nombre de atributo | Valor de atributo |
    | -------------- | ----------------|
    | Nombre del rol  | user.assignedroles |

    >[!NOTE]
    >Si el valor de notificación de rol es null, a continuación, Azure AD no envía este valor en el token y ésta es la predeterminada según el diseño.

     a. Haga clic en **editar** icono para abrir **atributos de usuario y notificaciones** cuadro de diálogo.

      ![Botón "Agregar atributo"](./media/active-directory-enterprise-app-role-management/editattribute.png)

    b. En el **Administrar notificaciones de usuario** cuadro de diálogo, agregar el atributo token SAML, haga clic en **agregar nueva notificación**.

      ![Botón "Agregar atributo"](./media/active-directory-enterprise-app-role-management/tutorial_attribute_04.png)

      ![Panel "Agregar atributo"](./media/active-directory-enterprise-app-role-management/tutorial_attribute_05.png)

    c. En el cuadro **Nombre**, escriba el nombre de atributo según sea necesario. En este ejemplo se utiliza **Role Name** como nombre de la notificación.

    d. Deje el cuadro **Espacio de nombres** en blanco.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Seleccione **Guardar**.

10. Para probar la aplicación en un inicio de sesión único iniciado por un proveedor de identidades, inicie sesión en el [Panel de acceso](https://myapps.microsoft.com) y seleccione el icono de la aplicación. En el token SAML, debería ver todos los roles asignados al usuario con el nombre de notificación que les ha dado.

## <a name="update-an-existing-role"></a>Actualización de un rol existente

Para actualizar un rol existente, siga estos pasos:

1. Abra el [Explorador de Azure AD Graph](https://developer.microsoft.com/graph/graph-explorer).

2. Inicie sesión en el sitio del Probador de Graph con las credenciales de administrador o coadministrador global del inquilino.

3. Cambie la versión a **beta** y recupere la lista de entidades de servicio del inquilino con la siguiente consulta:

    `https://graph.microsoft.com/beta/servicePrincipals`

    Si usa varios directorios, siga este patrón: `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

    ![Cuadro de diálogo del Probador de Graph, con la consulta para capturar entidades de servicio](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

4. En la lista de entidades de servicio capturadas, seleccione la que necesita modificar. También puede usar Ctrl + F para buscar la aplicación en todas las entidades de servicio enumeradas. Busque el identificador de objeto que copió de la página **Propiedades** y use la consulta siguiente para acceder a la entidad de servicio:

    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

    ![Consulta para obtener la entidad de servicio que necesita modificar](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

5. Extraiga la propiedad **appRoles** del objeto de entidad de servicio.

    ![Detalles de la propiedad appRoles](./media/active-directory-enterprise-app-role-management/graph-explorer-new3.png)

6. Para actualizar el rol existente, siga estos pasos.

    ![Cuerpo de la solicitud para "PATCH," con "description" y "displayname" resaltados](./media/active-directory-enterprise-app-role-management/graph-explorer-patchupdate.png)

     a. Cambie el método de **GET** a **PATCH**.

    b. Copie los roles existentes y péguelos en **Cuerpo de la solicitud**.

    c. Actualice el valor de un rol mediante la actualización de la descripción del rol, el valor del rol o el nombre para mostrar del rol, lo que sea necesario.

    d. Después de actualizar todos los roles necesarios, seleccione **Ejecutar consulta**.

## <a name="delete-an-existing-role"></a>Eliminación de un rol existente

Para eliminar un rol existente, siga estos pasos:

1. Abra el sitio del [Probador de Azure AD Graph](https://developer.microsoft.com/graph/graph-explorer) en otra ventana.

2. Inicie sesión en el sitio del Probador de Graph con las credenciales de administrador o coadministrador global del inquilino.

3. Cambie la versión a **beta** y recupere la lista de entidades de servicio del inquilino con la siguiente consulta:

    `https://graph.microsoft.com/beta/servicePrincipals`

    Si usa varios directorios, siga este patrón: `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

    ![Cuadro de diálogo del Probador de Graph, con la consulta para capturar la lista de entidades de servicio](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

4. En la lista de entidades de servicio capturadas, seleccione la que necesita modificar. También puede usar Ctrl + F para buscar la aplicación en todas las entidades de servicio enumeradas. Busque el identificador de objeto que copió de la página **Propiedades** y use la consulta siguiente para acceder a la entidad de servicio:

    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

    ![Consulta para obtener la entidad de servicio que necesita modificar](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

5. Extraiga la propiedad **appRoles** del objeto de entidad de servicio.

    ![Detalles de la propiedad appRoles del objeto de entidad de servicio](./media/active-directory-enterprise-app-role-management/graph-explorer-new7.png)

6. Para eliminar el rol existente, siga estos pasos.

    ![Cuerpo de la solicitud de "PATCH" con IsEnabled establecido en false](./media/active-directory-enterprise-app-role-management/graph-explorer-new8.png)

     a. Cambie el método de **GET** a **PATCH**.

    b. Copie los roles existentes de la aplicación y péguelos en **Cuerpo de la solicitud**.

    c. Establezca el valor de **IsEnabled** en **false** en el rol que desee eliminar.

    d. Seleccione **Ejecutar consulta**.

    > [!NOTE]
    > Asegúrese de que dispone del rol msiam_access y de que el identificador coincide en el rol generado.

7. Una vez que el rol esté deshabilitado, elimine el bloque del rol de la sección **appRoles**. Mantenga el método **PATCH** y seleccione **Ejecutar consulta**.

8. Después de ejecutar la consulta el rol se elimina.

    > [!NOTE]
    > Para poder quitar el rol es preciso deshabilitar el rol.

## <a name="next-steps"></a>Pasos siguientes

Para conocer los pasos adicionales, consulte la [documentación de la aplicación](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list).

<!--Image references-->
<!--Image references-->

[1]: ./media/active-directory-enterprise-app-role-management/tutorial_general_01.png
[2]: ./media/active-directory-enterprise-app-role-management/tutorial_general_02.png
[3]: ./media/active-directory-enterprise-app-role-management/tutorial_general_03.png
[4]: ./media/active-directory-enterprise-app-role-management/tutorial_general_04.png
