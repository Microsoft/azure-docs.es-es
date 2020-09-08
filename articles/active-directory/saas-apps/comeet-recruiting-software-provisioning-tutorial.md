---
title: 'Tutorial: Configuración de Comeet Recruiting Software para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar de forma automática el aprovisionamiento de cuentas de usuario de Comeet Recruiting Software.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 05/07/2019
ms.author: jeedes
ms.openlocfilehash: 5165c3f03d51403a978ad3574306bc27d931e1d7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88536903"
---
# <a name="tutorial-configure-comeet-recruiting-software-for-automatic-user-provisioning"></a>Tutorial: Configuración de Comeet Recruiting Software para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se deben realizar en Comeet Recruiting Software y Azure Active Directory (Azure AD) para configurar Azure AD con el fin de que aprovisione de forma automática y cancele el aprovisionamiento de usuarios o grupos en Comeet Recruiting Software.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Comeet Recruiting Software](https://www.comeet.co/)
* Una cuenta de usuario en Comeet Recruiting Software con permisos de administrador.

## <a name="add-comeet-recruiting-software-from-the-gallery"></a>Adición de Comeet Recruiting Software desde la galería

Antes de configurar Comeet Recruiting Software para el aprovisionamiento automático de usuarios con Azure AD, tendrá que agregarlo desde la Galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Comeet Recruiting Software desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Comeet Recruiting Software**, seleccione **Comeet Recruiting Software** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Comeet Recruiting Software en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-comeet-recruiting-software"></a>Asignación de usuarios a Comeet Recruiting Software

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Comeet Recruiting Software. Una vez que lo decida, puede asignar esos usuarios o grupos a Comeet Recruiting Software si sigue estas instrucciones:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-comeet-recruiting-software"></a>Sugerencias importantes para asignar usuarios a Comeet Recruiting Software

* Se recomienda asignar un único usuario de Azure AD a Comeet Recruiting Software para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Comeet Recruiting Software, debe seleccionar cualquier rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-comeet-recruiting-software"></a>Configuración del aprovisionamiento automático de usuarios para Comeet Recruiting Software 

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de AD Azure con el fin de crear, actualizar y deshabilitar usuarios o grupos en Comeet Recruiting Software en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Comeet Recruiting Software, mediante las instrucciones del [tutorial de inicio de sesión único de Comeet Recruiting Software](comeetrecruitingsoftware-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-comeet-recruiting-software-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Comeet Recruiting Software en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Comeet Recruiting Software**.

    ![Vínculo de Comeet Recruiting Software en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, especifique los valores de **URL de inquilino** y **Token secreto** de la cuenta de Comeet Recruiting Software, como se describe en el paso 6.

6. En la [consola de administración de Comeet Recruiting Software](https://app.comeet.co/), vaya a **Comeet > Settings > Authentication > Microsoft Azure** (Comeet > Configuración > Autenticación > Microsoft Azure) y copie el valor **Secret Token for your company** (Token secreto para la empresa) en el campo **Token secreto** de Azure AD.

    ![Aprovisionamiento de Comeet Recruiting Software](./media/comeet-recruiting-software-provisioning-tutorial/secret-token-1.png)

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD se puede conectar a Comeet Recruiting Software. Si se produce un error en la conexión, asegúrese de que la cuenta de Comeet Recruiting Software tiene permisos de administrador e inténtelo de nuevo.

    ![Token](common/provisioning-testconnection-token.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Comeet** (Sincronizar usuarios de Azure Active Directory con Comeet).

    ![Asignaciones de usuario de Comeet Recruiting Software](media/comeet-recruiting-software-provisioning-tutorial/user-mappings.png)

11. En la sección **Asignación de atributos**, revise los atributos de usuario que se han sincronizado entre Azure AD y Comeet Recruiting Software. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Comeet Recruiting Software para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupos de Comeet Recruiting Software](media/comeet-recruiting-software-provisioning-tutorial/user-mapping-attributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Comeet Recruiting Software, cambie el valor de **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Para definir los usuarios o grupos que quiere aprovisionar en Comeet Recruiting Software, seleccione los valores deseados en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir vínculos al informe de la actividad de aprovisionamiento, en el que se describen todas las acciones realizadas por el servicio de aprovisionamiento de Azure AD en Comeet Recruiting Software.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* En la actualidad, en Comeet Recruiting Software no se admiten grupos.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

