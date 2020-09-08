---
title: 'Tutorial: Configuración de Visitly para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Visitly.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 08/30/2019
ms.author: Zhchia
ms.openlocfilehash: 15031a3e139265410179baa13db9ed3f1b41cf17
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88531810"
---
# <a name="tutorial-configure-visitly-for-automatic-user-provisioning"></a>Tutorial: Configuración de Visitly para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Visitly y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en Visitly.

> [!NOTE]
> En este tutorial se describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para obtener más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Visitly](https://www.visitly.io/pricing/)
* Una cuenta de usuario de Visitly con permisos de administrador

## <a name="assign-users-to-visitly"></a>Asignación de usuarios a Visitly 

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios o grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Visitly. Después, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Visitly:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-visitly"></a>Sugerencias importantes para asignar usuarios a Visitly 

* Se recomienda asignar primero un usuario individual de Azure AD a Visitly para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Cuando asigne un usuario a Visitly, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de acceso predeterminado quedan excluidos del aprovisionamiento.

## <a name="set-up-visitly-for-provisioning"></a>Configuración de Visitly para el aprovisionamiento

Antes de configurar Visitly para el aprovisionamiento automático de usuarios con Azure AD, debe habilitar el aprovisionamiento del sistema de administración de identidades entre dominios (SCIM) en Visitly.

1. Inicie sesión en [Visitly](https://app.visitly.io/login). Seleccione **Integraciones** > **Host Synchronization** (Hospedar sincronización).

    ![Hospedar sincronización](media/Visitly-provisioning-tutorial/login.png)

2. Selección la sección **Azure AD**.

    ![Sección Azure AD](media/Visitly-provisioning-tutorial/integration.png)

3. Copie la **clave de API**. Estos valores se escriben en el campo **Token secreto** de la pestaña **Aprovisionamiento** de la aplicación Visitly en Azure Portal.

    ![Clave de API](media/Visitly-provisioning-tutorial/token.png)


## <a name="add-visitly-from-the-gallery"></a>Incorporación de Visitly desde la galería

Para configurar Visitly para el aprovisionamiento automático de usuarios con Azure AD, agregue Visitly desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

Para agregar Visitly desde la galería de aplicaciones de Azure AD, siga estos pasos.

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Visitly**, seleccione **Visitly** en el panel de resultados y luego seleccione **Agregar** para agregar la aplicación.

    ![Visitly en la lista de resultados](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-visitly"></a>Configuración del aprovisionamiento automático de usuarios en Visitly 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Visitly en función de las asignaciones de grupos o usuarios de Azure AD.

> [!TIP]
> Para habilitar el inicio de sesión único basado en SAML para Visitly siga las instrucciones del [tutorial de inicio de sesión único de Visitly](Visitly-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-provisioning-for-visitly-in-azure-ad"></a>Configuración del aprovisionamiento automático de usuarios para Visitly en Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Todas las aplicaciones](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Visitly**.

    ![Vínculo a Visitly en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Modo de aprovisionamiento establecido en Automático](common/provisioning-automatic.png)

5. En la sección Credenciales de administrador, escriba los valores de `https://api.visitly.io/v1/usersync/SCIM` y **Clave de API** que recuperó anteriormente en **URL de inquilino** y **Token secreto** respectivamente. Para asegurarse de que Azure AD puede conectarse a Visitly, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de Visitly tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino y token](common/provisioning-testconnection-tenanturltoken.png)

6. En el cuadro **Dirección de correo electrónico para notificaciones**, escriba la dirección de correo electrónico de la persona o grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Dirección de correo electrónico para notificaciones](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Box** (Sincronizar usuarios de Azure Active Directory con Visitly).

    ![Asignaciones de usuario de Visitly](media/visitly-provisioning-tutorial/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Visitly en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con las cuentas de usuario de Visitly para las operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**.

    ![Atributos de usuario de Visitly](media/visitly-provisioning-tutorial/userattribute.png)

10. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Visitly, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Visitly.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Para más información sobre el tiempo que se tarda en aprovisionar los usuarios o los grupos, consulte [¿Cuánto tiempo se tarda en aprovisionar usuarios?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Puede usar la sección **Estado actual** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Visitly. Para obtener más información, vea [Comprobación del estado de aprovisionamiento](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Para leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

Visitly no admite eliminaciones permanentes. Solo se admiten eliminaciones temporales.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
