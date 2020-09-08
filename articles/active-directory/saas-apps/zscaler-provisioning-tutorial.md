---
title: 'Tutorial: Configuración de Zscaler para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Zscaler.
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
ms.openlocfilehash: a3424a42e86b47d4103dd41ca143b5bc7be3796a
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545937"
---
# <a name="tutorial-configure-zscaler-for-automatic-user-provisioning"></a>Tutorial: Configuración de Zscaler para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Zscaler y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Zscaler.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../active-directory-saas-app-provisioning.md).
>

## <a name="prerequisites"></a>Prerrequisitos

En el escenario que se describe en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure AD.
* Un inquilino de Zscaler.
* Una cuenta de usuario de Zscaler con permisos de administrador.

> [!NOTE]
> La integración de aprovisionamiento de Azure AD se basa en la SCIM API de Zscaler, que está disponible para los desarrolladores de Zscaler para las cuentas con el paquete Enterprise.

## <a name="adding-zscaler-from-the-gallery"></a>Incorporación de Zscaler desde la galería

Antes de configurar Zscaler para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Zscaler desde la galería de aplicaciones de Azure AD hasta la lista de aplicaciones SaaS administradas.

**Para agregar Zscaler desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler**, seleccione **Zscaler** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Zscaler en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-zscaler"></a>Asignación de usuarios a Zscaler

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han "asignado" a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Zscaler. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Zscaler:

* [Asignar un usuario o grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-zscaler"></a>Sugerencias importantes para asignar usuarios a Zscaler

* Se recomienda asignar un único usuario de Azure AD a Zscaler para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Zscaler, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-zscaler"></a>Configuración del aprovisionamiento automático de usuarios en Zscaler

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Zscaler en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Zscaler siguiendo las instrucciones del [tutorial de inicio de sesión único de Zscaler](zscaler-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-zscaler-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Zscaler en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, a continuación, **Zscaler**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zscaler**.

    ![El vínculo de Zscaler en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/provisioning-tab.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/provisioning-credentials.png)

5. En la sección **Admin Credentials** (Credenciales de administrador), especifique los valores de **Tenant URL** (URL de inquilino) y **Secret Token** (Token secreto) de la cuenta de Zscaler, como se describe en el paso 6.

6. Para obtener la **dirección URL del inquilino** y el **token secreto**, vaya a **Administration > Authentication Settings** (Administración > Configuración de autenticación) en la interfaz de usuario del portal de Zscaler y haga clic en  **SAML** en **Authentication Type** (Tipo de autenticación).

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/secret-token-1.png)

    Haga clic en **Configure SAML** (Configurar SAML) para abrir las opciones de **configuración de SAML**.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/secret-token-2.png)

    Seleccione **Enable SCIM-Based Provisioning** (Habilitar aprovisionamiento basado en SCIM) para recuperar **Base URL** (URL Base) y **Bearer Token** (Token de portador) y, después, guarde la configuración. Copie la **dirección URL Base** en la **dirección URL del inquilino** y el **token de portador** en el **token secreto** en Azure Portal.

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Zscaler. Si la conexión no se establece, asegúrese de que la cuenta de Zscaler Zscalertiene permisos de administrador y pruebe de nuevo.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/test-connection.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/notification.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Mappings** (Asignaciones), seleccione **Synchronize Azure Active Directory Users to Zscaler** (Sincronizar usuarios de Azure Active Directory con Zscaler).

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/user-mappings.png)

11. Examine los atributos de usuario que se sincronizan entre Azure AD y Zscaler en la sección **Attribute Mappings** (Asignaciones de atributos). Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Zscaler con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/user-attribute-mappings.png)

12. En la sección **Mappings** (Asignaciones), seleccione **Synchronize Azure Active Directory Groups to Zscaler** (Sincronizar grupos de Azure Active Directory con Zscaler).

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/group-mappings.png)

13. Examine los atributos de grupo que se sincronizan entre Azure AD y Zscaler en la sección **Attribute Mappings** (Asignaciones de atributos). Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con los grupos de Zscaler para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/group-attribute-mappings.png)

14. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](./../active-directory-saas-scoping-filters.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para Zscaler, cambie el valor de **Provisioning Status** (Estado de aprovisionamiento) a **On** (Activado) en la sección **Settings** (Configuración).

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/provisioning-status.png)

16. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Zscaler.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/scoping.png)

17. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Aprovisionamiento de Zscaler](./media/zscaler-provisioning-tutorial/save-provisioning.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Zscaler.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../active-directory-saas-provisioning-reporting.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../active-directory-saas-provisioning-reporting.md)

<!--Image references-->
[1]: ./media/zscaler-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-provisioning-tutorial/tutorial-general-03.png
