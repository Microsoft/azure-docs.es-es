---
title: 'Tutorial: Configuración de SmartFile para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de SmartFile.
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
ms.openlocfilehash: 675d9c76841ba1a4b94b666c7ff4ac4287fc5264
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88525197"
---
# <a name="tutorial-configure-smartfile-for-automatic-user-provisioning"></a>Tutorial: Configuración de SmartFile para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en SmartFile y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en SmartFile.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de SmartFile](https://www.SmartFile.com/pricing/).
* Una cuenta de usuario de SmartFile con permisos de administrador.

## <a name="assigning-users-to-smartfile"></a>Asignación de usuarios a SmartFile

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a SmartFile. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a SmartFile:
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-smartfile"></a>Sugerencias importantes para asignar usuarios a SmartFile

* Se recomienda asignar un único usuario de Azure AD a SmartFile para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a SmartFile, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-smartfile-for-provisioning"></a>Configuración de SmartFile para el aprovisionamiento

Antes de configurar SmartFile para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en SmartFile y recopilar los detalles adicionales que sean necesarios.

1. Inicie sesión en la consola de administración de SmartFile. Vaya a la esquina superior derecha de la consola de administración de SmartFile. Seleccione **Clave del producto**.

    ![Consola de administración de SmartFile](media/smartfile-provisioning-tutorial/login.png)

2. Para generar un token de portador, copie la **clave del producto** y la **contraseña del producto**. Péguelas en un bloc de notas separadas por dos puntos (:).
    
     ![SmartFile Agregar SCIM](media/smartfile-provisioning-tutorial/auth.png)

    ![SmartFile Agregar SCIM](media/smartfile-provisioning-tutorial/key.png)

## <a name="add-smartfile-from-the-gallery"></a>Incorporación de SmartFile desde la galería

Para configurar SmartFile para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar SmartFile desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar SmartFile desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SmartFile**, seleccione **SmartFile** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![SmartFile en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-smartfile"></a>Configuración del aprovisionamiento automático de usuarios en SmartFile 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en SmartFile en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para SmartFile siguiendo las instrucciones del [tutorial de inicio de sesión único de SmartFile](SmartFile-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-smartfile-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para SmartFile en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **SmartFile**.

    ![Vínculo a SmartFile en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5.  En la sección **Credenciales de administrador**, escriba `https://<SmartFile sitename>.smartfile.com/ftp/scim` en la **URL de inquilino**. Un ejemplo podría tener un aspecto similar a `https://demo1test.smartfile.com/ftp/scim`. Escriba el valor **Token de portador** (ClaveDelProducto:ContraseñaDelProducto) que recuperó anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a SmartFile. Si la conexión no se establece, asegúrese de que la cuenta de SmartFile tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to SmartFile** (Sincronizar usuarios de Azure Active Directory con SmartFile).

    ![Asignaciones de usuario de SmartFile](media/smartfile-provisioning-tutorial/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre AzureSmartFileAD y SmartFile en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de SmartFile con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de SmartFile](media/smartfile-provisioning-tutorial/userattribute.png)

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to SmartFile** (Sincronizar grupos de Azure Active Directory con SmartFile).

    ![Asignaciones de grupos de SmartFile](media/smartfile-provisioning-tutorial/groupmapping.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y SmartFile en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de SmartFile con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de SmartFile](media/smartfile-provisioning-tutorial/groupattribute.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para SmartFile, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en SmartFile.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

    Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en SmartFile.

    Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).
    
## <a name="connector-limitations"></a>Limitaciones del conector

* SmartFile solo admite eliminaciones permanentes. 

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

 [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
