---
title: 'Tutorial: Configuración de Meta Networks Connector para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario a Meta Networks Connector.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 10/01/2019
ms.author: Zhchia
ms.openlocfilehash: 17ec8c04e3313155bdd00f119dc2f75882f51f33
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88535021"
---
# <a name="tutorial-configure-meta-networks-connector-for-automatic-user-provisioning"></a>Tutorial: Configuración de Meta Networks Connector para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Meta Networks Connector y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Meta Networks Connector.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Meta Networks Connector](https://www.metanetworks.com/)
* Una cuenta de usuario en Meta Networks Connector con permisos de administrador.

## <a name="assigning-users-to-meta-networks-connector"></a>Asignación de usuarios a Meta Networks Connector

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Meta Networks Connector. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Meta Networks Connector:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-meta-networks-connector"></a>Sugerencias importantes para asignar usuarios a Meta Networks Connector

* Se recomienda asignar un único usuario de Azure AD a Meta Networks Connector para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Meta Networks Connector, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-meta-networks-connector-for-provisioning"></a>Configuración de Meta Networks Connector para el aprovisionamiento

1. Inicie sesión en la [consola de administración de Meta Networks Connector](https://login.metanetworks.com/login/) con el nombre de su organización. Vaya a **Administración > Claves de API** .

    ![Consola de administración de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/apikey.png)

2.  Haga clic en el signo más en la esquina superior derecha de la pantalla para crear una **clave de API** nueva.

    ![Signo más de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/plusicon.png)

3.  Establezca el **Nombre de la clave de API** y la **Descripción de la clave de API**.

    ![Crear token de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/keyname.png)

4.  Active los privilegios de **Escritura** para **Grupos** y **Usuarios**.

    ![Privilegios de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/privileges.png)

5.  Haga clic en **Agregar**. Copie el valor **SECRET** y guárdelo, porque esta será la única vez que podrá verlo. Este valor se escribirá en el campo Token secreto de la pestaña Aprovisionamiento en la aplicación Meta Networks Connector en Azure Portal.

    ![Crear token de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/token.png)

6.  Para agregar un IdP, vaya a **Administración > Configuración > IdP > Crear nuevo**.

    ![Agregar IdP de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/newidp.png)

7.  En la página **Configuración de IdP**, puede asignar un **Nombre** a la configuración del IdP y elegir un **Icono**.

    ![Nombre de IdP de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/idpname.png)

    ![Icono de IdP de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/icon.png)

8.  En **Configure SCIM**  (Configurar SCIM), seleccione el nombre de la clave de API que se creó en los pasos anteriores. Haga clic en **Guardar**.

    ![Configurar SCIM de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/configure.png)

9.  Vaya a **Administración > Configuración > pestaña IdP**. Haga clic en el nombre de la configuración de IdP que se creó en los pasos anteriores para ver el **id. del IdP**. Este **identificador** se agregan al final de la **URL de inquilino** al escribir el valor en el campo **URL de inquilino** en la pestaña Aprovisionamiento de la aplicación Meta Networks Connector en Azure Portal.

    ![Identificar de IdP de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/idpid.png)

## <a name="add-meta-networks-connector-from-the-gallery"></a>Incorporación de Meta Networks Connector desde la galería

Antes de configurar Meta Networks Connector para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Meta Networks Connector desde la Galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Meta Networks Connector desde la Galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Meta Networks Connector**, seleccione **Meta Networks Connector** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Meta Networks Connector en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-meta-networks-connector"></a>Configuración del aprovisionamiento automático de usuarios en Meta Networks Connector 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Meta Networks Connector en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Meta Networks Connector. Para ello, siga las instrucciones que aparecen en el [tutorial de inicio de sesión único de Meta Networks Connector Single](https://docs.microsoft.com/azure/active-directory/saas-apps/metanetworksconnector-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-meta-networks-connector-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Meta Networks Connector en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Meta Networks Connector**.

    ![Vínculo de Meta Networks Connector en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://api.metanetworks.com/v1/scim/<IdP ID>` en la **URL de inquilino**. Escriba el valor **SCIM Authentication Token** (Token de autenticación de SCIM) recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Meta Networks Connector. Si la conexión no se establece, asegúrese de que la cuenta de Meta Networks Connector tenga permisos de administrador y pruebe otra vez.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Meta Networks Connector** (Sincronizar usuarios de Azure Active Directory con Meta Networks Connector).

    ![Asignaciones de usuario de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/usermappings.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Meta Networks Connector en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Meta Networks Connector con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/userattributes.png)

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Meta Networks Connector** (Sincronizar grupos de Azure Active Directory con Meta Networks Connector).

    ![Asignaciones de grupo de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupmappings.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Meta Networks Connector en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Meta Networks Connector con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Meta Networks Connector, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Defina los usuarios o grupos que quiere que se aprovisionen en Meta Networks Connector; para ello, elija los valores deseados en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Meta Networks Connector.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

