---
title: 'Tutorial: Configuración de Harness para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Harness.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 10/29/2019
ms.author: Zhchia
ms.openlocfilehash: 56a865de8cb1be079f4935ef2a8f840f10589b26
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550017"
---
# <a name="tutorial-configure-harness-for-automatic-user-provisioning"></a>Tutorial: Configuración de Harness para el aprovisionamiento automático de usuarios

En este artículo, aprenderá a configurar Azure Active Directory (Azure AD) para aprovisionar y desaprovisionar automáticamente usuarios o grupos en Harness.

> [!NOTE]
> En este artículo se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información acerca de este servicio, así como ver las respuestas proporcionadas a las preguntas que se realizan con más frecuencia al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este artículo se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Harness](https://harness.io/pricing/)
* Una cuenta de usuario de Harness con permisos de *administrador*.

## <a name="assign-users-to-harness"></a>Asignación de usuarios a Harness

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Harness. Después, puede asignar estos usuarios o grupos a Harness siguiendo las instrucciones de [Asignación de un usuario o un grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-harness"></a>Sugerencias importantes para asignar usuarios a Harness

* Se recomienda asignar primero un usuario individual de Azure AD a Harness, para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Cuando asigne un usuario a Harness, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de **asignación**. Los usuarios con el rol de *Acceso predeterminado* quedan excluidos del aprovisionamiento.

## <a name="set-up-harness-for-provisioning"></a>Configuración de Harness para el aprovisionamiento

1. Inicie sesión en la [consola de administración de Harness](https://app.harness.io/#/login) y, después, vaya a **Continuous Security** > **Access Management** (Seguridad continua > Administración del acceso).

    ![Consola de administración de Harness](media/harness-provisioning-tutorial/admin.png)

1. Seleccione **API Keys** (Claves de API).

    ![Vínculo de claves de API de Harness](media/harness-provisioning-tutorial/apikeys.png)

1. Seleccione **Add API Key** (Agregar clave de API). 

    ![Vínculo para agregar clave de API de Harness](media/harness-provisioning-tutorial/addkey.png)

1. En el panel **Add API Key** (Agregar clave de API), haga lo siguiente:

    ![Panel para agregar clave de API](media/harness-provisioning-tutorial/title.png)
   
   a. En el cuadro **Name** (Nombre), escriba un nombre para el atributo.  
   b. En la lista desplegable **Permissions Inherited from** (Permisos heredados de), seleccione una opción. 
   
1. Seleccione **Submit** (Enviar).

1. Copie la **clave** para su uso posterior en este tutorial.

    ![Crear token en Harness](media/harness-provisioning-tutorial/token.png)

## <a name="add-harness-from-the-gallery"></a>Adición de Harness desde la galería

Antes de configurar Harness para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Harness desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón "Azure Active Directory"](common/select-azuread.png)

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Vínculo "Todas las aplicaciones"](common/enterprise-applications.png)

1. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón “Nueva aplicación”](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **Harness**, seleccione **Harness** en la lista de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Harness en la lista de resultados](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-harness"></a>Configuración del aprovisionamiento automático de usuarios en Harness 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Harness en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Harness siguiendo las instrucciones del [Tutorial de inicio de sesión único de Harness](https://docs.microsoft.com/azure/active-directory/saas-apps/harness-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque las dos características se complementan entre sí.

> [!NOTE]
> Para obtener más información sobre el punto de conexión de SCIM de Harness, consulte el artículo sobre las [claves de API](https://docs.harness.io/article/smloyragsm-api-keys) de Harness.

Para configurar el aprovisionamiento automático de usuarios para Harness en Azure AD, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com), seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Harness**.

    ![Vínculo a Harness en la lista de aplicaciones](common/all-applications.png)

1. Seleccione **Aprovisionamiento**.

    ![Botón de aprovisionamiento](common/provisioning.png)

1. En la lista desplegable **Modo de aprovisionamiento**, seleccione **Automático**.

    ![Lista desplegable "Modo de aprovisionamiento"](common/provisioning-automatic.png)

1. En **Credenciales de administrador**, haga lo siguiente:

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)
 
   a. En el cuadro **URL de inquilino**, escriba **`https://app.harness.io/gateway/api/scim/account/XCPzWkCIQ46ypIu2DeT7yw`** .  
   b. En el cuadro **Token secreto**, escriba el valor del token de autenticación SCIM que guardó en el paso 6 de la sección "Configuración de Harness para el aprovisionamiento".  
   c. Para asegurarse de que Azure AD puede conectarse a Harness, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de Harness tiene permisos de *administrador* y pruebe otra vez.

1. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Cuadro de texto "Correo electrónico de notificación"](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Harness** (Sincronizar usuarios de Azure Active Directory con Harness).

    ![Vínculo de sincronización de los usuarios de Azure Active Directory a Harness](media/harness-provisioning-tutorial/usermappings.png)

1. En **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan desde Azure AD a Harness. Los atributos seleccionados como *Coincidencia* se usan para buscar coincidencias con las cuentas de usuario de Harness con el objetivo de realizar operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**.

    ![Panel "Asignaciones de atributos" del usuario de Harness](media/harness-provisioning-tutorial/userattributes.png)

1. En **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Harness** (Sincronizar grupos de Azure Active Directory con Harness).

    ![Vínculo de sincronización de los grupos de Azure Active Directory a Harness](media/harness-provisioning-tutorial/groupmappings.png)

1. En **Asignaciones de atributos**, revise los atributos de grupo que se sincronizan desde Azure AD a Harness. Los atributos seleccionados como propiedades de *Coincidencia* se usan para establecer coincidencias con los grupos de Harness con el objetivo de realizar operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**.

    ![Panel "Asignaciones de atributos" de grupo de Harness](media/harness-provisioning-tutorial/groupattributes.png)

1. Para configurar los filtros de ámbito, consulte [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. En **Configuración**", para habilitar el servicio de aprovisionamiento de Azure AD para Harness, cambie el conmutador **Estado de aprovisionamiento** a **Activado**.

    ![Conmutador Estado de aprovisionamiento cambiado a "Activado".](common/provisioning-toggle-on.png)

1. En **Configuración**, en la lista desplegable **Ámbito**, seleccione el modo en que desea sincronizar los usuarios o grupos que está aprovisionando en Harness.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Botón "Guardar" del aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de los usuarios o grupos que se están aprovisionando. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Las sincronizaciones se producen aproximadamente cada 40 minutos, mientras se ejecuta el servicio de aprovisionamiento de Azure AD. El progreso se puede supervisar en la sección **Detalles de sincronización**. También puede seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha realizado el servicio de aprovisionamiento de Azure AD en Harness.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
