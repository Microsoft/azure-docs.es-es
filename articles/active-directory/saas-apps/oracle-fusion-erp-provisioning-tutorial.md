---
title: 'Tutorial: Configuración de Oracle Fusion ERP para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario en Oracle Fusion ERP.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/26/2019
ms.author: Zhchia
ms.openlocfilehash: ad6b24c4bbfc2e117c010b99247cb8eb9925ff3e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543829"
---
# <a name="tutorial-configure-oracle-fusion-erp-for-automatic-user-provisioning"></a>Tutorial: Configuración de Oracle Fusion ERP para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Oracle Fusion ERP y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en Oracle Fusion ERP.

> [!NOTE]
>  Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para obtener más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* Un [inquilino de Oracle Fusion ERP](https://www.oracle.com/applications/erp/).
* Una cuenta de usuario en Oracle Fusion ERP con permisos de administrador

## <a name="assign-users-to-oracle-fusion-erp"></a>Asignación de usuarios a Oracle Fusion ERP 
Azure Active Directory usa el concepto de asignación para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Oracle Fusion ERP. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Oracle Fusion ERP:
 
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md) 

 ## <a name="important-tips-for-assigning-users-to-oracle-fusion-erp"></a>Sugerencias importantes para asignar usuarios a Oracle Fusion ERP 

 * Se recomienda asignar un único usuario de Azure AD a Oracle Fusion ERP para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Oracle Fusion ERP, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de acceso predeterminado quedan excluidos del aprovisionamiento.

## <a name="set-up-oracle-fusion-erp-for-provisioning"></a>Configuración de Oracle Fusion ERP para el aprovisionamiento

Antes de configurar Oracle Fusion ERP para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en Oracle Fusion ERP.

1. Inicie sesión en la [Consola de administración de Oracle Fusion ERP](https://cloud.oracle.com/sign-in).

2. Haga clic en Navigator (Navegador) en la esquina superior izquierda. En **Tools** (Herramientas), seleccione **Security Console** (Consola de seguridad).

    ![Adición de SCIM en Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/login.png)

3. Desplácese a **Users** (Usuarios).
    
    ![Adición de SCIM en Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/user.png)

4. Guarde el nombre de usuario y la contraseña de la cuenta de usuario de administrador que utilizará para iniciar sesión en la consola de administración de Oracle Fusion ERP. Estos valores deben especificarse en los campos **Nombre de usuario de administrador** y **Contraseña de administrador** en la pestaña Aprovisionamiento de la aplicación Oracle Fusion ERP en Azure Portal.

## <a name="add-oracle-fusion-erp-from-the-gallery"></a>Adición de Oracle Fusion ERP desde la galería

Para configurar Oracle Fusion ERP para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Oracle Fusion ERP desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Oracle Fusion ERP desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Oracle Fusion ERP** y seleccione **Oracle Fusion ERP** en el panel de resultados.

    ![Oracle Fusion ERP en la lista de resultados](common/search-new-app.png)

 ## <a name="configure-automatic-user-provisioning-to-oracle-fusion-erp"></a>Configuración del aprovisionamiento automático de usuarios en Oracle Fusion ERP 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Oracle Fusion ERP en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Oracle Fusion ERP si sigue las instrucciones del [tutorial de inicio de sesión único de Oracle Fusion ERP](oracle-fusion-erp-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

> [!NOTE]
> Para más información sobre el punto de conexión SCIM de Oracle Fusion ERP, consulte [API REST para las características comunes en la nube de aplicaciones Oracle](https://docs.oracle.com/en/cloud/saas/applications-common/18b/farca/index.html).

### <a name="to-configure-automatic-user-provisioning-for-fuze-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Fuze en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Oracle Fusion ERP**.

    ![El vínculo de Oracle Fusion ERP en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://ejlv.fa.em2.oraclecloud.com/hcmRestApi/scim/` en la **URL de inquilino**. Escriba el nombre de usuario y la contraseña de administrador recuperados anteriormente en los campos **Nombre de usuario de administrador** y **Contraseña de administrador**. Haga clic en **Probar conexión** entre Azure AD y Oracle Fusion ERP. 

    ![Adición de SCIM en Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/admin.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Fuze** (Sincronizar usuarios de Azure Active Directory con Oracle Fusion ERP).

    ![Adición de SCIM en Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/user-mapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Oracle Fusion ERP en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer las coincidencias con las cuentas de usuario de Oracle Fusion ERP, con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Adición de SCIM en Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/user-attribute.png)

10. En la sección **Asignaciones**, seleccione **Sincronización de grupos de Azure Active Directory con Oracle Fusion ERP**.

    ![Asignaciones de grupos de Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/groupmappings.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Oracle Fusion ERP en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer las coincidencias con los grupos de Oracle Fusion ERP, con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/groupattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Oracle Fusion ERP, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Oracle Fusion ERP.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

    Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Oracle Fusion ERP.

    Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Oracle Fusion ERP solo admite la autenticación básica para su punto de conexión SCIM.
* Oracle Fusion ERP no admite el aprovisionamiento de grupos.
* Los roles en Oracle Fusion ERP se asignan a grupos en Azure AD. Para asignar roles a los usuarios de Oracle Fusion ERP desde Azure AD, deberá asignar los usuarios a los grupos de Azure AD que desee y que tengan el nombre de los roles de Oracle Fusion ERP.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
