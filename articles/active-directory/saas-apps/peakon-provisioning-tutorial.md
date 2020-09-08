---
title: 'Tutorial: Configurar el aprovisionamiento automático de usuarios de Peakon con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Peakon.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 06/28/2019
ms.author: zhchia
ms.openlocfilehash: b16ca0c396e2c406677af351cf6b1de2a3499353
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543682"
---
# <a name="tutorial-configure-peakon-for-automatic-user-provisioning"></a>Tutorial: Configurar Peakon para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Peakon y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Peakon.

> [!NOTE]
>  Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes

* Un inquilino de Azure AD.
* [Un inquilino de Peakon](https://peakon.com/us/pricing/).
* Una cuenta de usuario de Peakon con permisos de administrador.

## <a name="assigning-users-to-peakon"></a>Asignación de usuarios a Peakon

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Peakon. Una vez decidido, puede asignar estos usuarios o grupos a Peakon con la instrucciones siguientes:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-peakon"></a>Sugerencias importantes para asignar usuarios a Peakon 

* Se recomienda asignar un único usuario de Azure AD a Peakon para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Peakon, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-peakon-for-provisioning"></a>Configurar Peakon para el aprovisionamiento

1.  Inicie sesión en la [consola de administración de Peakon](https://app.Peakon.com/login). Haga clic en **Configuración**. 

    ![Consola de administración de Peakon](media/Peakon-provisioning-tutorial/Peakon-admin-configuration.png)

2.  Seleccione **Integraciones**.
    
    ![Peakon-employee-provision](media/Peakon-provisioning-tutorial/Peakon-select-integration.png)

3.  Habilite **Employee Provisioning** (Aprovisionamiento de empleados).

    ![Peakon-employee-provision](media/Peakon-provisioning-tutorial/peakon05.png)

4.  Copie los valores de **SCIM 2.0 URL** y **Token de portador de OAuth**. Estos valores se escriben en el campo **URL de inquilino** y **Token secreto** de la pestaña Aprovisionamiento de la aplicación Peakon en Azure Portal.

    ![Peakon: Crear token](media/Peakon-provisioning-tutorial/peakon04.png)

## <a name="add-peakon-from-the-gallery"></a>Agregar Peakon desde la galería

Para configurar Peakon para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Peakon desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Peakon**, seleccione **Peakon** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Peakon en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-peakon"></a>Configuración del aprovisionamiento automático de usuarios en Peakon 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en Peakon en función de las asignaciones de grupos o usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Peakon si sigue las instrucciones del [tutorial de inicio de sesión único de Peakon](peakon-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-peakon--in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Peakon en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Peakon**.

    ![Vínculo a Peakon en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba los valores de **SCIM 2.0 URL** y **Token de portador de OAuth** recuperados anteriormente en **URL de inquilino** y **Token secreto** respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Peakon. Si la conexión no se establece, asegúrese de que la cuenta de Peakon tiene permisos de administrador y pruebe otra vez.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

7. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

8. Haga clic en **Save**(Guardar).

9. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Peakon** (Sincronizar usuarios de Azure Active Directory con Peakon).

    ![Peakon: Asignaciones de usuario](media/Peakon-provisioning-tutorial/Peakon-user-mappings.png)

10. Revise los atributos de usuario que se sincronizan entre Azure AD y Peakon en la sección **Asignaciones de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Peakon con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Peakon: Atributos de usuario](media/Peakon-provisioning-tutorial/Peakon-user-attributes.png)

12. Para configurar filtros de ámbito, vea las siguientes instrucciones, que se proporcionan en el [tutorial de filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
    
    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Peakon.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Todos los atributos de usuario personalizados de Peakon deben ampliarse a partir de la extensión de usuario SCIM personalizada de Peakon de `urn:ietf:params:scim:schemas:extension:peakon:2.0:User`.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)