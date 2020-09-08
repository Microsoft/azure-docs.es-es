---
title: 'Tutorial: Configuración de Keeper Password Manager & Digital Vault para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Keeper Password Manager & Digital Vault.
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
ms.openlocfilehash: a9ca77a059625ace13e6798e3fde84f11f3fe1db
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546802"
---
# <a name="tutorial-configure-keeper-password-manager--digital-vault-for-automatic-user-provisioning"></a>Tutorial: Configurar Keeper Password Manager & Digital Vault para el aprovisionamiento de usuario automático

El objetivo de este tutorial es mostrar los pasos que se realizan en Keeper Password Manager & Digital Vault y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Keeper Password Manager & Digital Vault.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Keeper Password Manager & Digital Vault](https://keepersecurity.com/pricing.html?t=e)
* Una cuenta de usuario en Keeper Password Manager & Digital Vault con permisos de administrador.

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>Incorporar Keeper Password Manager & Digital Vault desde la galería

Antes de configurar Keeper Password Manager & Digital Vault para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Keeper Password Manager & Digital Vault desde la galería de aplicaciones de Azure AD hasta la lista de aplicaciones SaaS administradas.

**Para agregar Keeper Password Manager & Digital Vault desde la galería de Azure AD, realice los pasos siguientes:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Keeper Password Manager & Digital Vault**, seleccione **Keeper Password Manager & Digital Vault** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Keeper Password Manager & Digital Vault en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-keeper-password-manager--digital-vault"></a>Asignación de usuarios a Keeper Password Manager & Digital Vault

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Keeper Password Manager & Digital Vault. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Keeper Password Manager & Digital Vault:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-keeper-password-manager--digital-vault"></a>Sugerencias importantes para la asignación de usuarios a Keeper Password Manager & Digital Vault

* Se recomienda asignar un único usuario de Azure AD a Keeper Password Manager & Digital Vault para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Keeper Password Manager & Digital Vault, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-keeper-password-manager--digital-vault"></a>Configurar el aprovisionamiento de usuario automático en Keeper Password Manager & Digital Vault 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en aprovisionamiento de usuario automático en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML de Keeper Password Manager & Digital Vault, siguiendo las instrucciones proporcionadas en el [tutorial de inicio de sesión único de Keeper Password Manager & Digital Vault](keeperpasswordmanager-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-keeper-password-manager--digital-vault-in-azure-ad"></a>Para configurar el aprovisionamiento de usuario automático para Keeper Password Manager & Digital Vault en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Keeper Password Manager & Digital Vault**.

    ![Vínculo a Keeper Password Manager & Digital Vault en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, especifique los valores de **URL de inquilino** y **Token secreto** de la cuenta de Keeper Password Manager & Digital Vault, como se describe en el paso 6.

6. Inicie sesión en la [consola de administración de Keeper](https://keepersecurity.com/console/#login). Haga clic en **Administrador** y seleccione un nodo existente o cree uno nuevo. Navegue hasta la pestaña **Aprovisionamiento** y haga clic en **Agregar método**.

    ![Consola de administración de Keeper](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-admin-console.png)

    Seleccione **SCIM (System for Cross-Domain Identity Management)** .

    ![Keeper Agregar SCIM](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-add-scim.png)

    Haga clic en **Crear token de aprovisionamiento**.

    ![Keeper Crear punto de conexión](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-endpoint.png)

    Copie los valores de **URL** y **Token** y péguelos en **URL de inquilino** y **Token secreto** en Azure AD. Haga clic en **Guardar** para completar la configuración de aprovisionamiento en Keeper.

    ![Keeper Crear token](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-token.png)

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Keeper Password Manager & Digital Vault. Si la conexión no se establece, asegúrese de que la cuenta de Keeper Password Manager & Digital Vault tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Keeper Password Manager & Digital Vault**.

    ![Keeper Asignaciones de usuario](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-mappings.png)

11. Revise los atributos de usuario que se sincronizan entre Azure AD y Keeper Password Manager & Digital Vault en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Keeper Password Manager & Digital Vault con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Keeper Atributos de usuario](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-attributes.png)

12. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con Keeper Password Manager & Digital Vault**.

    ![Keeper Asignaciones de grupos](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-mappings.png)

13. Revise los atributos de grupo que se sincronizan entre Azure AD y Keeper Password Manager & Digital Vault en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Keeper Password Manager & Digital Vault con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Keeper Atributos de grupo](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-attributes.png)

14. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para Keeper Password Manager & Digital Vault, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

16. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Keeper Password Manager & Digital Vault.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

17. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Keeper Password Manager & Digital Vault.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Keeper Password Manager & Digital Vault requiere que **emails** y **userName** tengan el mismo valor de origen, pues las actualizaciones de cualquiera de los dos atributos modificará el otro valor.
* Keeper Password Manager & Digital Vault no permite eliminar usuarios, solo deshabilitarlos. Los usuarios deshabilitados se mostrarán como bloqueados en la interfaz de usuario de la consola de administración de Keeper.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

