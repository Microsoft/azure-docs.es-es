---
title: 'Tutorial: Configuración de Tableau Online para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Tableau Online.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: da89bf6cb336087f97341ca57a9b2545c966a053
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88533447"
---
# <a name="tutorial-configure-tableau-online-for-automatic-user-provisioning"></a>Tutorial: Configuración de Tableau Online para el aprovisionamiento automático de usuarios

Este tutorial muestra los pasos que se deben realizar en Tableau Online y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios y grupos en Tableau Online.

> [!NOTE]
> En este tutorial se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prerequisites

En el escenario que se describe en este tutorial se supone que ya cuenta con los elementos siguientes:

*   Un inquilino de Azure AD.
*   Un [inquilino de Tableau Online](https://www.tableau.com/).
*   Una cuenta de usuario de Tableau Online con permisos de administrador.

> [!NOTE]
> La integración de aprovisionamiento de Azure AD se basa en la [API REST de Tableau Online](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm). Esta API está disponible para los desarrolladores de Tableau Online.

## <a name="add-tableau-online-from-the-azure-marketplace"></a>Adición de Tableau Online desde Azure Marketplace
Antes de configurar Tableau Online para el aprovisionamiento automático de usuarios con Azure AD, agregue Tableau Online desde Azure Marketplace en su lista de aplicaciones SaaS administradas.

Para eliminar Tableau Online de Marketplace, siga estos pasos.

1. En el panel de navegación de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Icono de Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Tableau Online** y seleccione **Tableau Online** desde el panel de resultados. Para agregar la aplicación, seleccione **Agregar**.

    ![Tableau Online en la lista de resultados](common/search-new-app.png)

## <a name="assign-users-to-tableau-online"></a>Asignación de usuarios a Tableau Online

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios o grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Tableau Online. Para asignar estos usuarios o grupos a Tableau Online, siga las instrucciones de [Asignación de un usuario o un grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal).

### <a name="important-tips-for-assigning-users-to-tableau-online"></a>Sugerencias importantes para asignar usuarios a Tableau Online

*   Se recomienda asignar un usuario individual de Azure AD a Tableau Online para probar la configuración del aprovisionamiento automático de usuarios. Asigne usuarios o grupos adicionales más tarde.

*   Cuando asigne un usuario a Tableau Online, seleccione un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configure-automatic-user-provisioning-to-tableau-online"></a>Configuración del aprovisionamiento automático de usuarios en Tableau Online

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de Azure AD. Úselo para crear, actualizar y deshabilitar usuarios o grupos en Tableau Online en función de las asignaciones de usuarios o grupos en Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Tableau Online. Siga las instrucciones del [Tutorial del inicio de sesión único de Tableau Online](tableauonline-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-provisioning-for-tableau-online-in-azure-ad"></a>Configuración del aprovisionamiento automático de usuarios para Tableau Online en Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Tableau Online**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Tableau Online**.

    ![Vínculo a Tableau Online en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento de Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningTab.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Modo de aprovisionamiento de Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningCredentials.png)

5. En la sección **Credenciales de administrador**, escriba los valores de dominio, nombre de usuario administrador, contraseña de administrador y dirección URL del contenido de la cuenta de Tableau Online:

   * En el cuadro **Dominio**, rellene el subdominio según el paso 6.

   * En el cuadro **Nombre de usuario de administrador**, rellene el nombre de usuario de la cuenta de administrador del inquilino de Clarizen. Un ejemplo es admin@contoso.com.

   * En el cuadro **Contraseña de administrador**, rellene la contraseña de la cuenta de administrador correspondiente al nombre de usuario de administrador.

   * En el cuadro **Dirección URL del contenido**, rellene el subdominio según el paso 6.

6. Después de iniciar sesión en la cuenta administrativa de Tableau Online, puede obtener los valores de **Dominio** y **Dirección URL del contenido** de la dirección URL de la página de administración.

    * El valor de **Dominio** de la cuenta de Tableau Online se puede copiar de esta parte de la dirección URL:

        ![Dominio de Tableau Online](./media/tableau-online-provisioning-tutorial/DomainUrlPart.png)

    * El valor de la **Dirección URL del contenido** de la cuenta de Tableau Online se puede copiar de esta sección. Es un valor que se define durante la configuración de la cuenta. En este ejemplo, el valor es "contoso":

        ![Dirección URL del contenido de Tableau Online](./media/tableau-online-provisioning-tutorial/ContentUrlPart.png)

        > [!NOTE]
        > El valor de **Dominio** puede ser diferente del que se muestra aquí.

7. Después de rellenar los cuadros que se muestran en el paso 5, seleccione **Probar conexión** para asegurarse de que Azure AD puede conectarse a Tableau Online. Si la conexión no se establece, asegúrese de que la cuenta de Tableau Online tiene permisos de administrador e inténtelo de nuevo.

    ![Prueba de conexión de Tableau Online](./media/tableau-online-provisioning-tutorial/TestConnection.png)

8. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de la persona o el grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Notificación por correo electrónico de Tableau Online](./media/tableau-online-provisioning-tutorial/EmailNotification.png)

9. Seleccione **Guardar**.

10. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Tableau**.

    ![Sincronización de usuarios de Tableau Online](./media/tableau-online-provisioning-tutorial/UserMappings.png)

11. Revise los atributos de usuario que se sincronizan entre Azure AD y Tableau Online en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Tableau Online con el objetivo de realizar operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de usuario coincidentes de Tableau Online](./media/tableau-online-provisioning-tutorial/UserAttributeMapping.png)

12. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con Tableau**.

    ![Sincronización de grupos de Tableau Online](./media/tableau-online-provisioning-tutorial/GroupMappings.png)

13. Revise los atributos de grupo que se sincronizan entre Azure AD y Tableau Online en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Tableau Online con el objetivo de realizar operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de grupo coincidentes de Tableau Online](./media/tableau-online-provisioning-tutorial/GroupAttributeMapping.png)

14. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para Tableau Online, vaya a **Configuración** y cambie el **Estado de aprovisionamiento** a **Activado**.

    ![Estado de aprovisionamiento de Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningStatus.png)

16. Defina los usuarios o grupos que quiere que se aprovisionen en Tableau Online. En la sección **Configuración**, seleccione los valores que quiere incluir en el **Ámbito**.

    ![Ámbito de Tableau Online](./media/tableau-online-provisioning-tutorial/ScopeSync.png)

17. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Guardado de Tableau Online](./media/tableau-online-provisioning-tutorial/SaveProvisioning.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Se produce aproximadamente cada 40 minutos mientras se ejecuta el servicio de aprovisionamiento de Azure AD. 

Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos al informe de actividad de aprovisionamiento. En el informe se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Tableau Online.

Para obtener información acerca de cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/tableau-online-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/tableau-online-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/tableau-online-provisioning-tutorial/tutorial_general_03.png
