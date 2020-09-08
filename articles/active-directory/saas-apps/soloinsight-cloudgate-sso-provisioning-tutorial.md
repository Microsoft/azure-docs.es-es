---
title: 'Tutorial: Configuración de Soloinsight-CloudGate SSO para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario de Soloinsight-CloudGate SSO.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 10/14/2019
ms.author: Zhchia
ms.openlocfilehash: 32fd40ec4827c2995d156b0e238b48ff8cc24e65
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545121"
---
# <a name="tutorial-configure-soloinsight-cloudgate-sso-for-automatic-user-provisioning"></a>Tutorial: Configuración de Soloinsight-CloudGate SSO para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Soloinsight-CloudGate SSO y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Soloinsight-CloudGate SSO.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Soloinsight-CloudGate SSO](https://www.soloinsight.com/)
* Una cuenta de usuario en Soloinsight-CloudGate SSO con permisos de administrador.

## <a name="assigning-users-to-soloinsight-cloudgate-sso"></a>Asignación de usuarios a Soloinsight-CloudGate SSO

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Soloinsight-CloudGate SSO. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Soloinsight-CloudGate SSO:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-soloinsight-cloudgate-sso"></a>Sugerencias importantes para asignar usuarios a Soloinsight-CloudGate SSO

* Se recomienda asignar un único usuario de Azure AD a Soloinsight-CloudGate SSO para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Soloinsight-CloudGate SSO, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-soloinsight-cloudgate-sso-for-provisioning"></a>Configuración de Soloinsight-CloudGate SSO para el aprovisionamiento

1. Inicie sesión en la [consola de administración de Soloinsight-CloudGate SSO](https://soloinsight.sigateway.com/login). Vaya a **Administration > System Settings**.

    ![Consola de administración de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/admin.png)

2.  Navegue a **General**.

    ![Adición de SCIM en Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/config.png)

3.  Desplácese hasta el final de la página para obtener **Tenant URL** y **Secret Token**. Copie el valor de **Secret Token**. Este valor se escribirá en el campo Token secreto de la pestaña Aprovisionamiento en la aplicación Soloinsight-CloudGate SSO en Azure Portal.

    ![Creación de token de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/token.png)

## <a name="add-soloinsight-cloudgate-sso-from-the-gallery"></a>Incorporación de Soloinsight-CloudGate SSO desde la galería

Antes de configurar Soloinsight-CloudGate SSO para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Soloinsight-CloudGate SSO desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Soloinsight-CloudGate SSO desde la galería de aplicaciones de Azure AD, realice los pasos siguientes:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Soloinsight-CloudGate SSO**, seleccione **Soloinsight-CloudGate SSO** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Soloinsight-CloudGate SSO en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-soloinsight-cloudgate-sso"></a>Configuración del aprovisionamiento automático de usuarios en Soloinsight-CloudGate SSO 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Soloinsight-CloudGate SSO en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por seguir las instrucciones del [tutorial de Soloinsight-CloudGate SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/soloinsight-cloudgate-sso-tutorial) para habilitar el inicio de sesión único basado en SAML en Soloinsight-CloudGate SSO. El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-soloinsight-cloudgate-sso-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Soloinsight-CloudGate SSO en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Soloinsight CloudGate SSO**.

    ![Vínculo de Soloinsight-CloudGate SSO en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://sigateway.com/scim/v2/sync/serviceproviderconfig` en la **URL de inquilino**. Escriba el valor **SCIM Authentication Token** (Token de autenticación de SCIM) recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Soloinsight-CloudGate SSO. Si la conexión no se establece, asegúrese de que la cuenta de Soloinsight-CloudGate SSO tenga permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Soloinsight-CloudGate SSO** (Sincronizar usuarios de Azure Active Directory con Soloinsight-CloudGate SSO).

    ![Asignaciones de usuario de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/usermappings.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Soloinsight-CloudGate SSO en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Soloinsight-CloudGate SSO con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/userattributes.png)

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Soloinsight-CloudGate SSO** (Sincronizar grupos de Azure Active Directory con Soloinsight-CloudGate SSO).

    ![Asignaciones de grupos de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupmappings.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Soloinsight-CloudGate SSO en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Soloinsight-CloudGate SSO con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupos de Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Soloinsight-CloudGate SSO, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Soloinsight-CloudGate SSO.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Soloinsight-CloudGate SSO.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

