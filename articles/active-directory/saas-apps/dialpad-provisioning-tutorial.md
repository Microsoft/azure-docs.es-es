---
title: 'Tutorial: Configuración de Dialpad para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Dialpad.
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
ms.openlocfilehash: 86fedfc75d95ef8d864a50d2425e329cec3ec84f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88536585"
---
# <a name="tutorial-configure-dialpad-for-automatic-user-provisioning"></a>Tutorial: Configuración de Dialpad para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Dialpad y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Dialpad.

> [!NOTE]
>  Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

> Este conector está actualmente en versión preliminar. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de Dialpad](https://www.dialpad.com/pricing/).
* Una cuenta de usuario de Dialpad con permisos de administrador.

## <a name="assign-users-to-dialpad"></a>Asignación de usuarios a Dialpad
Azure Active Directory usa el concepto de asignación para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Dialpad. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Dialpad:
 
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md) 

 ## <a name="important-tips-for-assigning-users-to-dialpad"></a>Sugerencias importantes para asignar usuarios a Dialpad

 * Se recomienda asignar un único usuario de Azure AD a Dialpad para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Dialpad, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de acceso predeterminado quedan excluidos del aprovisionamiento.

## <a name="setup-dialpad-for-provisioning"></a>Configuración de Dialpad para el aprovisionamiento

Antes de configurar Dialpad para el aprovisionamiento automático de usuarios con Azure AD, debe recuperar información de aprovisionamiento de Dialpad.

1. Inicie sesión en la [consola de administración de Dialpad](https://dialpadbeta.com/login) y seleccione **Configuración de administrador**. Asegúrese de que **My Company** (Mi empresa) está seleccionado en la lista desplegable. Vaya a **Autenticación > Claves de API**.

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/dialpad01.png)

2. Haga clic en **Add a key** (Agregar una clave) para generar una clave nueva y configure las propiedades de su token secreto.

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/dialpad02.png)

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/dialpad03.png)

3. Haga clic en el botón **Click to show value** (Hacer clic para ver el valor) para que aparezca la clave de API creada recientemente y copie el valor mostrado. Este valor se escribe en el campo **Token secreto** de la pestaña Aprovisionamiento de la aplicación Dialpad en Azure Portal. 

    ![Creación de token en Dialpad](media/dialpad-provisioning-tutorial/dialpad04.png)

## <a name="add-dialpad-from-the-gallery"></a>Incorporación de Dialpad desde la galería

Para configurar Dialpad para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar la aplicación desde la galería de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Dialpad desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Dialpad** y seleccione **Dialpad** en el panel de resultados.
    ![Dialpad en la lista de resultados](common/search-new-app.png)

5. Vaya a la **dirección URL** resaltada a continuación desde un explorador diferente. 

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/dialpad05.png)

6. En la esquina superior derecha, seleccione **Iniciar sesión > Use Dialpad online** (Usar Dialpad en línea).

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/dialpad06.png)

7. Como Dialpad es una aplicación de OpenIDConnect, inicie sesión en Dialpad con su cuenta Microsoft profesional.

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/loginpage.png)

8. Después de autenticarse, acepte la petición de consentimiento de la página de consentimiento. La aplicación se agregará automáticamente a su inquilino y será redirigido a su cuenta de Dialpad.

    ![Incorporación de SCIM en Dialpad](media/dialpad-provisioning-tutorial/redirect.png)

 ## <a name="configure-automatic-user-provisioning-to-dialpad"></a>Configuración del aprovisionamiento automático de usuarios en Dialpad

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Dialpad en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-dialpad-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Dialpad en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Dialpad**.

    ![Vínculo a Dialpad en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://dialpad.com/scim` en la **URL de inquilino**. Escriba el valor que recuperó y guardó anteriormente de Dialpad en **Token secreto**. Haga clic en **Prueba de conexión** para asegurarse de que Azure AD puede conectarse a Dialpad. Si la conexión no se establece, asegúrese de que la cuenta de Dialpad tiene permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Dialpad** (Sincronizar usuarios de Azure Active Directory con Dialpad).

    ![Asignaciones de usuarios en Dialpad](media/dialpad-provisioning-tutorial/dialpad-user-mappings-new.png)

9. Examine los atributos de usuario que se sincronizan entre Azure AD y Dialpad en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Matching** (Coincidente) se usan para buscar coincidencias con las cuentas de usuario de Dialpad para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario en Dialpad](media/dialpad-provisioning-tutorial/dialpad07.png)

10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Dialpad, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Dialpad.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Dialpad.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).
##  <a name="connector-limitations"></a>Limitaciones del conector
* Dialpad no admite cambios de nombre de grupo actualmente. Esto significa que los cambios realizados en la propiedad **displayName** de un grupo en Azure AD no se actualizará ni reflejará en Dialpad.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
