---
title: 'Tutorial: Configuración de Proxyclick para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente las cuentas de usuario para Proxyclick.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 06/3/2019
ms.author: jeedes
ms.openlocfilehash: 54a7723e07bd348e31b564707da9e78e96593d3c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553371"
---
# <a name="tutorial-configure-proxyclick-for-automatic-user-provisioning"></a>Tutorial: configuración de Proxyclick para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Proxyclick y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Proxyclick.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Proxyclick](https://www.proxyclick.com/pricing)
* Una cuenta de usuario de Proxyclick con permisos de administrador.

## <a name="add-proxyclick-from-the-gallery"></a>Incorporación de Proxyclick desde la galería

Antes de configurar Proxyclick para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Proxyclick desde la galería de aplicaciones de Azure AD hasta la lista de aplicaciones SaaS administradas.

**Para agregar Proxyclick desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Proxyclick**, seleccione **Proxyclick** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Proxyclick en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-proxyclick"></a>Asignación de usuarios a Proxyclick

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Proxyclick. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Proxyclick:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-proxyclick"></a>Sugerencias importantes para asignar usuarios a Proxyclick

* Se recomienda asignar un único usuario de Azure AD a Proxyclick para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Proxyclick, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-proxyclick"></a>Configuración del aprovisionamiento automático de usuarios en Proxyclick 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Proxyclick en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Proxyclick siguiendo las instrucciones del [tutorial de inicio de sesión único de Proxyclick](proxyclick-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-proxyclick-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Proxyclick en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Proxyclick**.

    ![Vínculo de Proxyclick en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. Para recuperar los valores de **URL de inquilino** y **Token secreto** de su cuenta Proxyclick, siga el tutorial, tal como se describe en el paso 6.

6. Inicie sesión en [Proxyclick Admin Console](https://app.proxyclick.com/login//?destination=%2Fdefault). Vaya a **Settings (Configuración)**  > **Integrations (Integraciones)**  > **Browse Marketplace (Explorar Marketplace)** .

    ![Settings (Configuración) de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick09.png)

    ![Integrations (Integraciones) de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick01.png)

    ![Marketplace de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick02.png)

    Seleccione **Azure AD**. Haga clic en **Install now** (Instalar ahora).

    ![Proxyclick Azure AD](media/proxyclick-provisioning-tutorial/proxyclick03.png)

    ![Instalación de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick04.png)

    Seleccione **User Provisioning** (Aprovisionamiento de usuarios) y haga clic en **Start integration** (Iniciar integración). 

    ![Aprovisionamiento de usuarios de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick05.png)

    La interfaz de usuario adecuada para la configuración debería aparecer ahora bajo **Settings (Configuración)**  > **Integrations (Integraciones)** . Seleccione **Settings** (Configuración) bajo **Azure AD (User Provisioning)** (Azure AD (aprovisionamiento de usuarios).

    ![Creación de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick06.png)

    Puede encontrar los valores de **Tenant URL** (URL de inquilino) y **Secret Token** (Token secreto) aquí.

    ![Creación de token de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick07.png)

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD pueda conectarse a Proxyclick. Si la conexión no se establece, asegúrese de que la cuenta de Proxyclick tiene permisos de administrador y pruebe de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Proxyclick** (Sincronizar usuarios de Azure Active Directory con Proxyclick).

    ![Asignaciones de usuario de Proxyclick](media/proxyclick-provisioning-tutorial/Proxyclick-user-mappings.png)

11. Revise los atributos de usuario que se sincronizan entre Azure AD y Proxyclick en la sección **Asignaciones de atributos**. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con las cuentas de usuario de Proxyclick para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Proxyclick](media/proxyclick-provisioning-tutorial/Proxyclick-user-attribute.png)

13. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

14. Para habilitar el servicio de aprovisionamiento de Azure AD para Proxyclick, cambie la opción **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

15. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Proxyclick.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

16. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Proxyclick.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Proxyclick requiere que los atributos **emails** y **userName** tengan el mismo valor de origen. Si se actualiza cualquiera de los atributos, modificará el otro valor.
* Proxyclick no admite el aprovisionamiento de grupos.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

