---
title: 'Tutorial: Configuración de OfficeSpace Software para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de OfficeSpace Software.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 10/02/2019
ms.author: Zhchia
ms.openlocfilehash: ab2fd7f78373f9fee349e59c374088f67f165339
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554211"
---
# <a name="tutorial-configure-officespace-software-for-automatic-user-provisioning"></a>Tutorial: Configuración de OfficeSpace Software para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se deben realizar en OfficeSpace Software y Azure Active Directory (Azure AD) para configurar Azure AD con el fin de que aprovisione de forma automática y cancele el aprovisionamiento de usuarios o grupos en OfficeSpace Software.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* Un [inquilino de OfficeSpace Software](https://www.officespacesoftware.com/)
* Una cuenta de usuario en OfficeSpace Software con permisos de administrador.

## <a name="assigning-users-to-officespace-software"></a>Asignación de usuarios a OfficeSpace Software

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a OfficeSpace Software. Una vez que lo decida, puede asignar esos usuarios o grupos a OfficeSpace Software si sigue estas instrucciones:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-officespace-software"></a>Sugerencias importantes para asignar usuarios a OfficeSpace Software

* Se recomienda asignar un único usuario de Azure AD a OfficeSpace Software para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a OfficeSpace Software, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-officespace-software-for-provisioning"></a>Configuración de OfficeSpace Software para el aprovisionamiento

1. Inicie sesión en la [consola de administración de OfficeSpace Software](https://support.officespacesoftware.com/hc). Vaya a **Configuración > conectores**.

    ![Consola de administración de OfficeSpace Software](media/officespace-software-provisioning-tutorial/settings.png)

2.  Vaya a **Sincronización de directorios > SCIM**.

    ![OfficeSpace Software Add SCIM](media/officespace-software-provisioning-tutorial/scim.png)

3.  Copie el valor de **SCIM Authentication Token** (Token de autenticación de SCIM). Este valor se escribe en el campo Token secreto de la pestaña Aprovisionamiento de la aplicación OfficeSpace Software en Azure Portal.

    ![OfficeSpace Software Create Token](media/officespace-software-provisioning-tutorial/token.png)

## <a name="add-officespace-software-from-the-gallery"></a>Adición de OfficeSpace Software desde la galería

Antes de configurar OfficeSpace Software para el aprovisionamiento automático de usuarios con Azure AD, tendrá que agregarlo desde la Galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar OfficeSpace Software desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **OfficeSpace Software**, seleccione **OfficeSpace Software** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![OfficeSpace Software en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-officespace-software"></a>Configuración para el aprovisionamiento automático de usuarios a OfficeSpace Software 

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de AD Azure con el fin de crear, actualizar y deshabilitar usuarios o grupos en OfficeSpace Software en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para OfficeSpace Software, mediante las instrucciones del [Tutorial de inicio de sesión único de OfficeSpace Software](https://docs.microsoft.com/azure/active-directory/saas-apps/officespace-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-officespace-software-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para OfficeSpace Software en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **OfficeSpace Software**.

    ![Vínculo a OfficeSpace Software en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba la dirección URL `https://<subdomain>.officespacesoftware.com/api/scim/v2/` en la **URL de inquilino**. Por ejemplo, `https://contoso.officespacesoftware.com/api/scim/v2/`. Escriba el valor **SCIM Authentication Token** (Token de autenticación de SCIM) recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a OfficeSpace Software. Si se produce un error en la conexión, asegúrese de que la cuenta de OfficeSpace Software tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to OfficeSpace Software** (Sincronizar usuarios de Azure Active Directory con OfficeSpace Software).

    ![Asignaciones de usuario de OfficeSpace Software](media/officespace-software-provisioning-tutorial/usermappings.png)

9. En la sección **Asignación de atributos**, revise los atributos de usuario que se han sincronizado entre Azure AD y OfficeSpace Software. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con las cuentas de usuario de OfficeSpace Software para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de OfficeSpace Software](media/officespace-software-provisioning-tutorial/userattributes.png)

11. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Para habilitar el servicio de aprovisionamiento de Azure AD para OfficeSpace Software, cambie el valor de **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

13. Para definir los usuarios o grupos que quiere aprovisionar en OfficeSpace Software, seleccione los valores deseados en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

14. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en OfficeSpace Software.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

