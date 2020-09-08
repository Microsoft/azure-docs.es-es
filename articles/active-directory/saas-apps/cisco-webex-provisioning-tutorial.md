---
title: 'Tutorial: Configuración de Cisco Webex para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente las cuentas de usuario para Cisco Webex.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: bdf2bf4102795d35df7eae658e3a24028da79b33
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551309"
---
# <a name="tutorial-configure-cisco-webex-for-automatic-user-provisioning"></a>Tutorial: Configuración de Cisco Webex para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se deben realizar en Cisco Webex y Azure Active Directory (Azure AD) para configurar Azur  AD de modo que aprovisione y desaprovisione automáticamente los usuarios en Cisco Webex.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de Cisco Webex](https://www.webex.com/pricing/index.html).
* Una cuenta de usuario de Cisco Webex con permisos de administrador.

## <a name="adding-cisco-webex-from-the-gallery"></a>Adición de Cisco Webex desde la galería

Antes de configurar Cisco Webex para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Cisco Webex desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Cisco Webex desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Cisco Webex**, seleccione **Cisco Webex** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Cisco Webex en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-cisco-webex"></a>Asignación de usuarios a Cisco Webex

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han "asignado" a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios de Azure AD necesitan acceder a Cisco Webex. Una vez decidido, puede asignar estos usuarios a Cisco Webex siguiendo estas instrucciones:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cisco-webex"></a>Sugerencias importantes para asignar usuarios a Cisco Webex

* Se recomienda asignar un único usuario de Azure AD a Cisco Webex para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios.

* Al asignar un usuario a Cisco Webex, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-cisco-webex"></a>Configuración del aprovisionamiento automático de usuarios en Cisco Webex

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios en Cisco Webex en función de las asignaciones de usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-cisco-webex-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Cisco Webex en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Cisco Webex**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Cisco Webex**.

    ![Vínculo a Cisco Webex en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento en Cisco Webex](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento en Cisco Webex](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba la **URL de inquilino** y el **token secreto** de la cuenta de Cisco Webex.

    ![Aprovisionamiento en Cisco Webex](./media/cisco-webex-provisioning-tutorial/secrettoken1.png)

6.  En el campo **URL de inquilino**, escriba un valor en el formato de `https://api.ciscoweb.com/v1/scim/[OrgId]`. Para obtener `[OrgId]`, inicie sesión en el [centro de control de Cisco Webex](https://admin.webex.com/login). Haga clic en el nombre de la organización en la parte inferior izquierda y copie el valor de **id. de la organización**. 

    * Para obtener el valor de **token secreto**, navegue hasta esta [URL](https://idbroker.webex.com/idb/saml2/jsp/doSSO.jsp?type=login&goto=https%3A%2F%2Fidbroker.webex.com%2Fidb%2Foauth2%2Fv1%2Fauthorize%3Fresponse_type%3Dtoken%26client_id%3DC4ca14fe00b0e51efb414ebd45aa88c1858c3bfb949b2405dba10b0ca4bc37402%26redirect_uri%3Dhttp%253A%252F%252Flocalhost%253A3000%252Fauth%252Fcode%26scope%3Dspark%253Apeople_read%2520spark%253Apeople_write%2520Identity%253ASCIM%26state%3Dthis-should-be-a-random-string-for-security-purpose). En la página de inicio de sesión de Webex que se muestra, inicie sesión con la cuenta de administrador de Cisco Webex completa de su organización. Se mostrará una página de error en que se le indicará que no se puede acceder al sitio, pero es normal.

        ![Aprovisionamiento en Cisco Webex](./media/cisco-webex-provisioning-tutorial/test.png)
 
    * Copie el valor del token de portador generado mediante la dirección URL, como se indica a continuación. Este token es válido durante 365 días.
        
        ![Aprovisionamiento en Cisco Webex](./media/cisco-webex-provisioning-tutorial/test1.png)

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Cisco Webex. Si la conexión no se establece, asegúrese de que la cuenta de Cisco Webex tenga permisos de administrador y pruebe otra vez.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)
   
8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Cisco Webex** (Sincronizar usuarios de Azure Active Directory con Cisco Webex).

    ![Aprovisionamiento en Cisco Webex](./media/cisco-webex-provisioning-tutorial/usermapping.png)

11. Revise los atributos de usuario que se sincronizan entre Azure AD y Cisco Webex en la sección **Attribute Mapping** (Asignaciones de atributos). Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Cisco Webex con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Aprovisionamiento en Cisco Webex](./media/cisco-webex-provisioning-tutorial/usermappingattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Cisco Webex, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Para definir los usuarios o grupos que desea que se aprovisionen en Cisco Webex, elija los valores deseados en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos al informe de la actividad de aprovisionamiento, que describe todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Cisco Webex.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Actualmente, Cisco Webex se encuentra en la fase de pruebas de campo tempranas (EFT) de Cisco. Para más información, póngase en contacto con el [equipo de soporte técnico de Cisco](https://www.webex.co.in/support/support-overview.html). 
* Para obtener más información sobre la configuración de Cisco Webex, consulte la documentación de Cisco [aquí](https://help.webex.com/en-us/aumpbz/Synchronize-Azure-Active-Directory-Users-into-cisco-webex-Control-Hub).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)