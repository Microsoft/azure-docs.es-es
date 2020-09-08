---
title: 'Tutorial: Aprovisionamiento de usuarios para GitHub: Azure AD'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de GitHub.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: arvinh
ms.openlocfilehash: 7408d7609cbceb4ac39298680b6d3854a2d71306
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589221"
---
# <a name="tutorial-configure-github-for-automatic-user-provisioning"></a>Tutorial: Configuración de GitHub para aprovisionar usuarios automáticamente

El objetivo de este tutorial es explicar los pasos que hay que realizar en GitHub y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD en GitHub.

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure Active Directory
* Una organización GitHub creada en [GitHub Enterprise Cloud](https://help.github.com/articles/github-s-products/#github-enterprise), que requiere el [plan de facturación de GitHub Enterprise](https://help.github.com/articles/github-s-billing-plans/#billing-plans-for-organizations).
* Una cuenta de usuario de GitHub con permisos de administrador para la organización
* Asegúrese de que se ha proporcionado acceso de OAuth para su organización, tal como se describe [aquí](https://help.github.com/en/github/setting-up-and-managing-organizations-and-teams/approving-oauth-apps-for-your-organization)
* El aprovisionamiento de SCIM para una sola organización solo se admite cuando SSO está habilitado en el nivel de organización

> [!NOTE]
> La integración del aprovisionamiento de Azure AD se basa en la [API de GitHub SCIM](https://developer.github.com/v3/scim/), que está disponible para los clientes de la [nube de GitHub Enterprise](https://help.github.com/articles/github-s-products/#github-enterprise) en el [plan de facturación de GitHub Enterprise](https://help.github.com/articles/github-s-billing-plans/#billing-plans-for-organizations).

## <a name="assigning-users-to-github"></a>Asignación de usuarios a GitHub

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD. 

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceso a la aplicación GitHub. Una vez decidido, puede asignar estos usuarios a la aplicación de GitHub siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-github"></a>Sugerencias importantes para asignar usuarios a GitHub

* Se recomienda asignar un único usuario de Azure AD a GitHub para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a GitHub, debe seleccionar el rol **Usuario** u otro válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. El rol **Acceso predeterminado** no funciona para realizar el aprovisionamiento y estos usuarios se omiten.

## <a name="configuring-user-provisioning-to-github"></a>Configuración del aprovisionamiento de usuarios en GitHub

Esta sección lo guía a través de los pasos necesarios para conectar la API de aprovisionamiento de cuentas de usuario de GitHub, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas en GitHub en función de la asignación de grupos y usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para GitHub siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-account-provisioning-to-github-in-azure-ad"></a>Configuración del aprovisionamiento de cuentas de usuario automático para GitHub en Azure AD

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

2. Si ya ha configurado GitHub para el inicio de sesión único, busque la instancia de GitHub mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **GitHub** en la galería de aplicaciones. Seleccione GitHub en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

3. Seleccione la instancia de GitHub y, luego, la pestaña **Aprovisionamiento**.

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de GitHub](./media/github-provisioning-tutorial/GitHub1.png)

5. En **Credenciales de administrador**, haga clic en **Autorizar**. Con esta operación se abre un cuadro de diálogo de autorización de GitHub en una nueva ventana del explorador. Tenga en cuenta que debe asegurarse de que está aprobado para autorizar el acceso. Siga las instrucciones descritas [aquí](https://help.github.com/github/setting-up-and-managing-organizations-and-teams/approving-oauth-apps-for-your-organization).

6. En esa nueva ventana, inicie sesión en GitHub con su cuenta de administrador. En el cuadro de diálogo de autorización que aparece, seleccione el equipo de GitHub para el que desea habilitar el aprovisionamiento y, luego, seleccione **Autorizar**. Cuando termina, vuelva a Azure Portal para completar la configuración de aprovisionamiento.

    ![Cuadro de diálogo de autorización](./media/github-provisioning-tutorial/GitHub2.png)

7. En Azure Portal, escriba la **dirección URL del inquilino** y haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación de GitHub. Si la conexión no se establece, asegúrese de que la cuenta de GitHub tenga permisos de administrador y de que la **dirección URL del inquilino** se haya escrito correctamente; luego repita el paso "Autorizar" (puede constituir la **dirección URL de inquilino** con la regla: `https://api.github.com/scim/v2/organizations/<Organization_name>`, puede encontrar las organizaciones en la cuenta de GitHub: **Configuración** > **Organizaciones**).

    ![Cuadro de diálogo de autorización](./media/github-provisioning-tutorial/GitHub3.png)

8. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla "Enviar una notificación por correo electrónico cuando se produzca un error".

9. Haga clic en **Save**(Guardar).

10. En la sección Asignaciones, seleccione **Synchronize Azure Active Directory Users to GitHub** (Sincronizar usuarios de Azure Active Directory con GitHub).

11. En la sección **Attribute Mappings** (Asignaciones de atributos), revise los atributos de usuario que se sincronizan entre Azure AD y GitHub. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de GitHub con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

12. Para habilitar el servicio de aprovisionamiento de Azure AD para GitHub, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

13. Haga clic en **Save**(Guardar).

Esta operación inicia la sincronización inicial de todos los usuarios y grupos asignados a GitHub en la sección Usuarios y grupos. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos a los registros de actividad de aprovisionamiento, que describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
