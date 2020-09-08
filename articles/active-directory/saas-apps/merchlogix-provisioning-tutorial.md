---
title: 'Tutorial: Configuración de MerchLogix para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de MerchLogix.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: zhchia
ms.openlocfilehash: 6fe256660dc73aa411cf06e2b56ce9ef26934e2f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548011"
---
# <a name="tutorial-configure-merchlogix-for-automatic-user-provisioning"></a>Tutorial: Configuración de MerchLogix para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en MerchLogix y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en MerchLogix.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* Un inquilino de MerchLogix
* Un contacto técnico en MerchLogix que puede proporcionar la dirección URL del punto de conexión de SCIM y el token secreto necesario para el aprovisionamiento de usuarios.

## <a name="adding-merchlogix-from-the-gallery"></a>Adición de MerchLogix desde la galería

Antes de configurar MerchLogix para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar MerchLogix desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar MerchLogix desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, haga clic en el icono de **Azure Active Directory**. 

    ![Botón Azure Active Directory][1]

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Sección Aplicaciones empresariales][2]

3. Para agregar MerchLogix, haga clic en el botón **Nueva aplicación** en la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación][3]

4. En el cuadro de búsqueda, escriba **MerchLogix**.

5. En el panel de resultados, seleccione **MerchLogix** y, luego, haga clic en el botón **Agregar** para agregar MerchLogix a la lista de aplicaciones SaaS.

    ![Aprovisionamiento de MerchLogix][4]

## <a name="assigning-users-to-merchlogix"></a>Asignación de usuarios a MerchLogix

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han "asignado" a una aplicación en Azure AD. 

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a MerchLogix. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a MerchLogix:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-merchlogix"></a>Sugerencias importantes para asignar usuarios a MerchLogix

* Se recomienda asignar un único usuario de Azure AD a MerchLogix para probar la configuración inicial de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos una vez realizadas las pruebas correctamente.

* Al asignar un usuario a MerchLogix, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-merchlogix"></a>Configuración del aprovisionamiento automático de usuarios en MerchLogix 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en MerchLogix en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para MerchLogix siguiendo las instrucciones del [tutorial de inicio de sesión único de MerchLogix](merchlogix-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-merchlogix-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para MerchLogix en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

2. Seleccione MerchLogix en la lista de aplicaciones SaaS.

3. Seleccione la pestaña **Aprovisionamiento**.

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de MerchLogix](./media/merchlogix-provisioning-tutorial/Merchlogix1.png)

5. En la sección **Credenciales de administrador**:

    * En el campo **URL de inquilino**, escriba la dirección URL de punto de conexión de SCIM proporcionada por el contacto técnico de MerchLogix.

    * En el campo **Token secreto**, escriba el token secreto proporcionada por el contacto técnico de MerchLogix.

6. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a MerchLogix. Si la conexión no se establece, asegúrese de que la cuenta de MerchLogix tenga permisos de administrador y pruebe otra vez.

7. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

8. Haga clic en **Save**(Guardar).

9. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con MerchLogix**.

10. Revise los atributos de usuario que se sincronizan entre Azure AD y MerchLogix en la sección **Attribute Mapping** (Asignaciones de atributos). Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de MerchLogix con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

11. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con MerchLogix**.

12. Revise los atributos de grupo que se sincronizan entre Azure AD y MerchLogix en la sección **Attribute Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con los grupos de MerchLogix para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

13. Para habilitar el servicio de aprovisionamiento de Azure AD para MerchLogix, cambie el valor de **Provisioning Status** (Estado de aprovisionamiento) a **On** (Activado) en la sección **Settings** (Configuración).

14. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en MerchLogix.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: common/select-azuread.png
[2]: common/enterprise-applications.png
[3]: common/add-new-app.png
[4]: common/search-new-app.png
