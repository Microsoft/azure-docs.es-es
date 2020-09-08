---
title: 'Tutorial: Configuración del aprovisionamiento automático de usuarios de Figma con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Figma.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: f0f0d61788d7ac9b242dbc176a125b45753d2578
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555131"
---
# <a name="tutorial-configure-figma-for-automatic-user-provisioning"></a>Tutorial: Configuración de Figma para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se deben realizar en Figma y Azure Active Directory (Azure AD) a fin de configurar Azure AD para aprovisionar y desaprovisionar automáticamente usuarios o grupos en Figma.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de Figma](https://www.figma.com/pricing/).
* Una cuenta de usuario de Figma con permisos de administrador.

## <a name="assign-users-to-figma"></a>Asignar usuarios a Figma
Azure Active Directory usa el concepto de asignación para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Figma. Una vez decidido, puede asignar esos usuarios o grupos a Figma mediante estas instrucciones:
 
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)
## <a name="important-tips-for-assigning-users-to-figma"></a>Sugerencias importantes para asignar usuarios a Figma

 * Se recomienda asignar un único usuario de Azure AD a Figma para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Figma, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de acceso predeterminado quedan excluidos del aprovisionamiento.

## <a name="set-up-figma-for-provisioning"></a>Configurar Figma para el aprovisionamiento

Antes de configurar Figma para el aprovisionamiento automático de usuarios con Azure AD, debe recuperar alguna información de aprovisionamiento de Figma.

1. Inicie sesión en la [consola de administración de Figma](https://www.Figma.com/). Haga clic en el icono de engranaje que se encuentra junto al inquilino.

    ![FigmaFigma-employee-provision](media/Figma-provisioning-tutorial/image0.png)

2. Vaya a **General > Actualizar el registro en la configuración**.

    ![FigmaFigma-employee-provision](media/Figma-provisioning-tutorial/figma03.png)

3. Copie el **Id. de inquilino**. Este valor se usa para construir la dirección URL del punto de conexión SCIM que se va a especificar en el campo **URL de inquilino** de la pestaña Aprovisionamiento de la aplicación Figma en Azure Portal.

    ![Figma: Crear token](media/Figma-provisioning-tutorial/figma-tenantid.png)

4. Desplácese hacia abajo y haga clic en **Generar token de API**.

    ![Figma: Crear token](media/Figma-provisioning-tutorial/token.png)

5. Copie el valor **Token de API**. Este valor se escribe en el campo **Token secreto** de la pestaña Aprovisionamiento de la aplicación Figma en Azure Portal. 

    ![Figma: Crear token](media/Figma-provisioning-tutorial/figma04.png)

## <a name="add-figma-from-the-gallery"></a>Agregar Figma desde la galería

Para configurar Figma para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Figma desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Figma**, seleccione **Figma** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Figma en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-figma"></a>Configuración del aprovisionamiento automático de usuarios en Figma 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Figma en función de las asignaciones de grupos o usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Figma si sigue las instrucciones del [tutorial de inicio de sesión único de Figma](figma-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-figma--in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Figma en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Figma**.

    ![Vínculo a Figma en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://www.figma.com/scim/v2/<TenantID>` en **URL de inquilino**, donde **TenantID** es el valor que se ha recuperado anteriormente desde Figma. Escriba el valor **Token de API** en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Figma. Si la conexión no se establece, asegúrese de que la cuenta de Figma tiene permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Figma**.

    ![Figma: Asignaciones de usuario](media/Figma-provisioning-tutorial/figma05.png)

11. Examine los atributos de usuario que se sincronizan entre Azure AD y Figma en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Figma con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Figma: Atributos de usuario](media/Figma-provisioning-tutorial/figma06.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Figma, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Seleccione los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere aprovisionar en Figma.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Figma.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)