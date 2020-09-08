---
title: 'Tutorial: Configuración de Netskope Administrator Console para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar cuentas de usuario en Netskope Administrator Console, así como cancelar el aprovisionamiento automáticamente.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 11/07/2019
ms.author: Zhchia
ms.openlocfilehash: cef356619d437d53d94c8f5d3be67cf6ac95501c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554563"
---
# <a name="tutorial-configure-netskope-administrator-console-for-automatic-user-provisioning"></a>Tutorial: Configuración de Netskope Administrator Console para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que hay que realizar en Netskope Administrator Console y en Azure Active Directory (Azure AD) para configurar Azure AD para aprovisionar usuarios o grupos en Netskope Administrator Console y cancelar el aprovisionamiento automáticamente.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Netskope Administrator Console](https://www.netskope.com/)
* Una cuenta de usuario en Netskope Administrator Console con permisos de administrador

## <a name="assigning-users-to-netskope-administrator-console"></a>Configuración del inicio de sesión único en Netskope Administrator Console

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Netskope Administrator Console. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Netskope Administrator Console:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-netskope-administrator-console"></a>Consejos importantes a la hora de asignar usuarios a Netskope Administrator Console

* Se recomienda asignar un solo usuario de Azure AD a Netskope Administrator Console para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Netskope Administrator Console, hay que seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-netskope-administrator-console-for-provisioning"></a>Configuración de Netskope Administrator Console para el aprovisionamiento

1. Inicie sesión en la [consola de administración de Netskope Administrator Console](https://netskope.goskope.com/). Vaya a **Home > Settings** (Inicio > Configuración).

    ![Consola de administración de Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/admin.png)

2.  Vaya a **Tools** (Herramientas). En el menú **Tools** (Herramientas), vaya a **Directory Tools > SCIM INTEGRATION** (Herramientas de directorio > INTEGRACIÓN DE SCIM).

    ![Herramientas de Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/tools.png)

    ![Adición de SCIM en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/directory.png)

3. Desplácese hacia abajo y haga clic en el botón **Add Token** (Agregar token). En el cuadro de diálogo **Add OAuth Client Name** (Agregar nombre de cliente de OAuth), indique un nombre de cliente en **CLIENT NAME** y haga clic en el botón **Save** (Guardar).

    ![Adición de token en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/add.png)

    ![Nombre de cliente en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/clientname.png)

3.  Copie los valores de **SCIM Server URL** (URL de servidor de SCIM) y **TOKEN**. Estos valores se escribirán en los campos URL de inquilino y Token secreto respectivamente de la pestaña Aprovisionamiento de la aplicación Netskope Administrator Console en Azure Portal.

    ![Creación de token en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/token.png)

## <a name="add-netskope-administrator-console-from-the-gallery"></a>Incorporación de Netskope Administrator Console desde la galería

Antes de configurar Netskope Administrator Console para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Netskope Administrator Console desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Haga lo siguiente para agregar Netskope Administrator Console desde la galería de aplicaciones de Azure AD:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Netskope Administrator Console**, seleccione **Netskope Administrator Console** en el panel de resultados y, después, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Netskope Administrator Console en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-netskope-administrator-console"></a>Configuración del aprovisionamiento automático de usuarios en Netskope Administrator Console 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Netskope Administrator Console en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Netskope Administrator Console mediante las instrucciones del [Tutorial de inicio de sesión único de Netskope Administrator Console](https://docs.microsoft.com/azure/active-directory/saas-apps/netskope-cloud-security-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

> [!NOTE]
> Para más información sobre el punto de conexión de SCIM de Netskope Administrator Console, consulte [esto](https://docs.google.com/document/d/1n9P_TL98_kd1sx5PAvZL2HS6MQAqkQqd-OSkWAAU6ck/edit#heading=h.prxq74iwdpon).

### <a name="to-configure-automatic-user-provisioning-for-netskope-administrator-console-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios en Netskope Administrator Console en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Netskope Administrator Console**.

    ![Vínculo de Netskope Administrator Console en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, en **URL de inquilino**, escriba el valor de **URL de servidor de SCIM** recuperado anteriormente. Escriba el valor de **TOKEN** recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para confirmar que Azure AD puede conectarse a Netskope Administrator Console. Si la conexión no se establece, asegúrese de que la cuenta de Netskope Administrator Console tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Netskope Administrator Console**.

    ![Asignaciones de usuarios en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/usermappings.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Netskope Administrator Console en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con las cuentas de usuario de Netskope Administrator Console con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/userattributes.png)

10. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con Netskope Administrator Console**.

    ![Asignaciones de grupos en Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/groupmappings.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Netskope Administrator Console en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con los grupos de Netskope Administrator Console con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de Netskope Administrator Console](media/netskope-administrator-console-provisioning-tutorial/groupattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Netskope Administrator Console, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiera que se aprovisionen en Netskope Administrator Console.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Netskope Administrator Console.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

