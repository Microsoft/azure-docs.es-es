---
title: 'Tutorial: Configuración de Druva para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario en Druva.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: 4d29b1301f66f2834c974bdcd2675cf1a672c742
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555723"
---
# <a name="tutorial-configure-druva-for-automatic-user-provisioning"></a>Tutorial: Configuración de Druva para aprovisionar usuarios automáticamente

El objetivo de este tutorial es mostrar los pasos que se deben realizar en Druva y Azure Active Directory (Azure AD) a fin de configurar Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en Druva.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de Druva](https://www.druva.com/products/pricing-plans/).
* Una cuenta de usuario en Druva con permisos de administrador.

## <a name="assigning-users-to-druva"></a>Asignación de usuarios a Druva

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Druva. Una vez decidido, puede asignar esos usuarios o grupos a Druva mediante estas instrucciones:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-druva"></a>Sugerencias importantes para asignar usuarios a Druva

* Se recomienda asignar un único usuario de Azure AD a Druva para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Druva, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-druva-for-provisioning"></a>Configuración de Druva para el aprovisionamiento

Antes de configurar Druva para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en Druva.

1. Inicie sesión en la [consola de administración de Druva](https://console.druva.com). Vaya a **Druva > inSync**.

    ![Consola de administración de Druva](media/druva-provisioning-tutorial/menubar.png)

2. Vaya a **Manage** > **Deployments** > **Users** (Administrar > Implantaciones > Usuarios).

    ![Adición de SCIM en Druva](media/druva-provisioning-tutorial/manage.png)

3.  Navegue a **Settings** (Configuración). Haga clic en **Generate Token** (Generar token).

    ![Adición de SCIM en Druva](media/druva-provisioning-tutorial/settings.png)

4.  Copie el valor de **Auth token** (Token de autenticación). Este valor se escribirá en el campo **Token secreto** de la pestaña Aprovisionamiento en la aplicación Druva en Azure Portal.
    
    ![Adición de SCIM en Druva](media/druva-provisioning-tutorial/auth.png)

## <a name="add-druva-from-the-gallery"></a>Adición de Druva desde la galería

Para configurar Druva para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Druva desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Druva desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Druva**, seleccione **Druva** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Druva en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-druva"></a>Configuración del aprovisionamiento automático de usuarios en Druva 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Druva en función de las asignaciones de grupos o usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Druva siguiendo las instrucciones del [tutorial de inicio de sesión único de Druva](druva-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-druva-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Druva en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Druva**.

    ![Vínculo a Druva en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5.  En la sección Credenciales de administrador, escriba `https://apis.druva.com/insync/scim` en **URL de inquilino**. Escriba el valor de **Auth token** (Token de autenticación) en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Druva. Si la conexión no se establece, asegúrese de que la cuenta de Druva tiene permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y seleccione **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Druva** (Sincronizar usuarios de Azure Active Directory con Druva).

    ![Asignaciones de usuario de Druva](media/druva-provisioning-tutorial/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Druva en la sección **Asignaciones de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer las coincidencias con las cuentas de usuario de Druva, con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Druva](media/druva-provisioning-tutorial/userattribute.png)


10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el servicio de aprovisionamiento de Azure AD para Druva, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Seleccione los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere aprovisionar en Druva.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

    Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Druva.

    Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).
    
## <a name="connector-limitations"></a>Limitaciones del conector

* Druva necesita **mail** como atributo obligatorio. 

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md).
