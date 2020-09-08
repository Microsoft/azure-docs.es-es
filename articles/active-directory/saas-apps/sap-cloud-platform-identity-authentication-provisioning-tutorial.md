---
title: 'Tutorial: Configuración de SAP Cloud Platform Identity Authentication para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de SAP Cloud Platform Identity Authentication.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 09/19/2019
ms.author: Zhchia
ms.openlocfilehash: c330d8000825b2e0ccbdab8c5c1cdefa4d3d45c8
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543382"
---
# <a name="tutorial-configure-sap-cloud-platform-identity-authentication-for-automatic-user-provisioning"></a>Tutorial: Configuración de SAP Cloud Platform Identity Authentication para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en SAP Cloud Platform Identity Authentication y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en SAP Cloud Platform Identity Authentication.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de SAP Cloud Platform Identity Authentication](https://cloudplatform.sap.com/pricing.html)
* Una cuenta de usuario en SAP Cloud Platform Identity Authentication con permisos de administrador.

## <a name="assigning-users-to-sap-cloud-platform-identity-authentication"></a>Asignación de usuarios a SAP Cloud Platform Identity Authentication

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a SAP Cloud Platform Identity Authentication. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a SAP Cloud Platform Identity Authentication:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-sap-cloud-platform-identity-authentication"></a>Sugerencias importantes para asignar usuarios a SAP Cloud Platform Identity Authentication

* Se recomienda asignar un único usuario de Azure AD a SAP Cloud Platform Identity Authentication para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a SAP Cloud Platform Identity Authentication, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-sap-cloud-platform-identity-authentication-for-provisioning"></a>Configuración de SAP Cloud Platform Identity Authentication para el aprovisionamiento

1. Inicie sesión en [SAP Cloud Platform Identity Authentication Admin Console](https://sapmsftintegration.accounts.ondemand.com/admin). Vaya a **Users & Authorizations > Administrators** (Usuarios y Autorizaciones > Administradores).

    ![Consola de administración de SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/adminconsole.png)

2.  Presione el botón **+Add** (+Agregar) en el panel de la izquierda para agregar un nuevo administrador a la lista. Elija **Add System** (Agregar sistema) y escriba el nombre del sistema.   

> [!NOTE]
> El usuario administrador de SAP Cloud Platform Identity Authentication debe ser del tipo **System** (Sistema). La creación de un usuario administrador normal puede generar errores de tipo *no autorizado* durante el aprovisionamiento.   

3.  En Configure Authorizations (Configurar autorizaciones), cambie el botón de alternancia a **Manage Users** (Administrar usuarios) y **Manage Groups** (Administrar grupos).

    ![Incorporación de SCIM en SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/configurationauth.png)

4. Recibirá un correo electrónico para activar su cuenta y establecer una contraseña para **SAP Cloud Platform Identity Authentication Service**.

4.  Copie el **User ID** (Identificador de usuario) y la **Password** (Contraseña). Estos valores se escribirán en los campos Nombre de usuario de administrador y Contraseña de administrador respectivamente en la pestaña Aprovisionamiento de la aplicación SAP Cloud Platform Identity Authentication en Azure Portal.

## <a name="add-sap-cloud-platform-identity-authentication-from-the-gallery"></a>Adición de SAP Cloud Platform Identity Authentication desde la galería

Antes de configurar SAP Cloud Platform Identity Authentication para el aprovisionamiento automático de usuarios con Azure AD, debe agregar SAP Cloud Platform Identity Authentication desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar SAP Cloud Platform Identity Authentication desde la galería de aplicaciones de Azure AD, realice los siguientes pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SAP Cloud Platform Identity Authentication**, seleccione **SAP Cloud Platform Identity Authentication** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![SAP Cloud Platform Identity Authentication en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-sap-cloud-platform-identity-authentication"></a>Configuración del aprovisionamiento automático de usuarios en SAP Cloud Platform Identity Authentication 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en SAP Cloud Platform Identity Authentication en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para SAP Cloud Platform Identity Authentication, siguiendo las instrucciones proporcionadas en el [tutorial de inicio de sesión único de SAP Cloud Platform Identity Authentication](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-sap-cloud-platform-identity-authentication-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para SAP Cloud Platform Identity Authentication en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **SAP Cloud Platform Identity Authentication**.

    ![Vínculo SAP Cloud Platform Identity Authentication en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://<tenantID>.accounts.ondemand.com/service/scim ` en la **URL de inquilino**. Escriba el **identificador de usuario** y la **contraseña** recuperados antes en **Nombre de usuario administrador** y **Contraseña de administrador**, respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD pueda conectarse a SAP Cloud Platform Identity Authentication. Si la conexión no se establece, asegúrese de que la cuenta de SAP Cloud Platform Identity Authentication tiene permisos de administrador y pruébelo de nuevo.

    ![URL de inquilino + Token](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/testconnection.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to SAP Cloud Platform Identity Authentication** (Sincronizar usuarios de Azure Active Directory con SAP Cloud Platform Identity Authentication).

    ![Asignaciones de usuario de SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/mapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y SAP Cloud Platform Identity Authentication en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de SAP Cloud Platform Identity Authentication con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/userattributes.png)

10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el servicio de aprovisionamiento de Azure AD para SAP Cloud Platform Identity Authentication, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Defina los usuarios y grupos que desearía aprovisionar en SAP Cloud Platform Identity Authentication; para ello, elija los valores que desee en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en SAP Cloud Platform Identity Authentication.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* El punto de conexión SCIM de SAP Cloud Platform Identity Authentication requiere que determinados atributos tengan un formato específico. Puede obtener más información sobre estos atributos y su formato específico [aquí](https://help.sap.com/viewer/6d6d63354d1242d185ab4830fc04feb1/Cloud/en-US/b10fc6a9a37c488a82ce7489b1fab64c.html#).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

