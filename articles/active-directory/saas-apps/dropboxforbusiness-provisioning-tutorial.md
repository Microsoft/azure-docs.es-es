---
title: 'Tutorial: Configuración de Dropbox Business para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Dropbox Business.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: e3dd0a31e23418c5871ac3db80b2ba1543576fac
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555772"
---
# <a name="tutorial-configure-dropbox-for-business-for-automatic-user-provisioning"></a>Tutorial: Configuración de Dropbox for Business para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Dropbox Business y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Dropbox Business.

> [!IMPORTANT]
> Microsoft y Dropbox dejarán de usar la antigua integración de Dropbox, lo cual entrará en vigor el 1 de abril de 2021. Para evitar la interrupción del servicio, se recomienda migrar a la nueva integración de Dropbox que admite grupos. Para migrar a la nueva integración de Dropbox, agregue y configure una nueva instancia de Dropbox para el aprovisionamiento en el inquilino de Azure AD, siguiendo los pasos que se indican a continuación. Una vez configurada la nueva integración de Dropbox, deshabilite el aprovisionamiento en la integración de Dropbox antigua para evitar conflictos de aprovisionamiento.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Dropbox Business](https://www.dropbox.com/business/pricing)
* Una cuenta de usuario de Dropbox Business con permisos de administrador

## <a name="add-dropbox-for-business-from-the-gallery"></a>Incorporación de Dropbox Business desde la galería

Antes de configurar Dropbox Business para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Dropbox Business desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Dropbox Business desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Dropbox Business**, seleccione **Dropbox Business** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Dropbox for Business en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-dropbox-for-business"></a>Asignación de usuarios a Dropbox for Business

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Dropbox Business. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Dropbox Business:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-dropbox-for-business"></a>Sugerencias importantes para asignar usuarios a Dropbox for Business

* Se recomienda asignar un único usuario de Azure AD a Dropbox Business para probar la configuración automática de aprovisionamiento de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Dropbox Business, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-dropbox-for-business"></a>Configuración del aprovisionamiento automático de usuarios en Dropbox Business 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Dropbox Business en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Dropbox Business siguiendo las instrucciones del [tutorial de inicio de sesión único de Dropbox Business](dropboxforbusiness-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-dropbox-for-business-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Dropbox Business en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Dropbox for Business**.

    ![El vínculo de Dropbox for Business en la lista Aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En **Credenciales de administrador**, haga clic en **Autorizar**. Se abre un cuadro de diálogo de inicio de sesión de Dropbox for Business en una nueva ventana del explorador.

    ![Aprovisionamiento ](common/provisioning-oauth.png)

6. En el cuadro de diálogo **Sign in to Dropbox Business to link with Azure AD** (Iniciar sesión en Dropbox Business para vincularlo a Azure AD), inicie sesión en su inquilino de Dropbox Business.

    ![Inicio de sesión en Dropbox Business](media/dropboxforbusiness-provisioning-tutorial/dropbox01.png)

7. Tras completar los pasos 5 y 6, haga clic en **Prueba de conexión** para asegurarse de que Azure AD se puede conectar a Dropbox Business. Si la conexión falla, asegúrese de que la cuenta de Dropbox Business tiene permisos de administrador e inténtelo de nuevo.

    ![Token](common/provisioning-testconnection-oauth.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Dropbox** (Sincronizar usuarios de Azure Active Directory con Dropbox).

    ![Asignaciones de usuario en Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-mapping.png)

11. Examine los atributos de usuario que se sincronizan entre Azure AD y Dropbox en la sección **Attribute Mappings** (Asignación de atributos). Los atributos seleccionados como propiedades **Matching** (Coincidencia) se usan para buscar coincidencias con las cuentas de usuario de Dropbox con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-attributes.png)

12. En la sección **Mappings** (Asignaciones), seleccione **Synchronize Azure Active Directory Groups to Dropbox** (Sincronizar grupos de Azure Active Directory con Dropbox).

    ![Asignaciones de grupo de Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-mapping.png)

13. Examine los atributos de grupo que se sincronizan entre Azure AD y Dropbox en la sección **Attribute Mappings** (Asignaciones de atributos). Los atributos seleccionados como propiedades **Matching** (Coincidencia) se usan para buscar coincidencias con los grupos de Dropbox para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-attributes.png)

14. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Dropbox, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

16. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Dropbox.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

17. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Dropbox.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector
 
* Dropbox no es compatible con la suspensión de los usuarios invitados. Si se suspende un usuario invitado, se elimina.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

