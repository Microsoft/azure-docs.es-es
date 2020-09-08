---
title: 'Tutorial: Configuración de iPass SmartConnect para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario de iPass SmartConnect.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: 533f31882ad06ecc4b411fe0f686ab51f189c0a3
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552917"
---
# <a name="tutorial-configure-ipass-smartconnect-for-automatic-user-provisioning"></a>Tutorial: Configuración de iPass SmartConnect para aprovisionar usuarios automáticamente

El objetivo de este tutorial es mostrar los pasos que se realizan en iPass SmartConnect y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en iPass SmartConnect.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de iPass SmartConnect](https://www.ipass.com/buy-ipass/).
* Una cuenta de usuario de iPass SmartConnect con permisos de administrador.

## <a name="assigning-users-to-ipass-smartconnect"></a>Asignación de usuarios a iPass SmartConnect

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a iPass SmartConnect. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a iPass SmartConnect:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-ipass-smartconnect"></a>Sugerencias importantes para asignar usuarios a iPass SmartConnect

* Se recomienda asignar un único usuario de Azure AD a iPass SmartConnect para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a iPass SmartConnect, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-ipass-smartconnect-for-provisioning"></a>Configuración de iPass SmartConnect para el aprovisionamiento

Antes de configurar iPass SmartConnect para el aprovisionamiento automático de usuarios con Azure AD, tendrá que recuperar la información de configuración de la consola de administración de iPass SmartConnect:

1. Para recuperar el token de portador necesario para autenticarse en el punto de conexión de SCIM de iPass SmartConnect, debe hacer referencia a la primera vez que configuró iPass SmartConnect, ya que es el único momento en que se proporciona este valor. 
2. Si no tiene el token de portador, póngase en contacto con el [equipo de soporte técnico de iPass SmartConnect](mailto:help@ipass.com) para recuperar uno nuevo.

## <a name="add-ipass-smartconnect-from-the-gallery"></a>Adición de iPass SmartConnect desde la galería

Para configurar Smartsheet para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar iPass SmartConnect desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar iPass SmartConnect desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **iPass SmartConnect**, seleccione **iPass SmartConnect** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![iPass SmartConnect en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-ipass-smartconnect"></a>Configuración del aprovisionamiento automático de usuarios en iPass SmartConnect 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en iPass SmartConnect en función de las asignaciones de grupos o usuarios de Azure AD.

> [!TIP]
>  También puede optar por habilitar el inicio de sesión único basado en SAML para BitaBIZ siguiendo las instrucciones del [tutorial de inicio de sesión único de iPass SmartConnect](ipasssmartconnect-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-ipass-smartconnect-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para iPass SmartConnect en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **iPass SmartConnect**.

    ![El vínculo de iPass SmartConnect en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://openmobile.ipass.com/moservices/scim/v1` en la **URL de inquilino**. Escriba el token de portador recuperado anteriormente en el **token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a iPass SmartConnect. Si se produce un error en la conexión, asegúrese de que la cuenta de iPass SmartConnect tiene permisos de administrador y vuelva a intentarlo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to iPass SmartConnect** (Sincronizar los usuarios de Azure Active Directory para iPass SmartConnect).

    ![Asignaciones de usuario de iPass SmartConnect](media/ipass-smartconnect-provisioning-tutorial/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD e iPass SmartConnect en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de iPass SmartConnect con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Asignaciones de usuario de iPass SmartConnect](media/ipass-smartconnect-provisioning-tutorial/userattribute.png)


10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el servicio de aprovisionamiento de Azure AD para iPass SmartConnect, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en iPass SmartConnect.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en iPass SmartConnect.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* iPass SmartConnect solo acepta los nombres de usuario que tienen sus dominios registrados en la consola de administración de iPass SmartConnect.  

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
