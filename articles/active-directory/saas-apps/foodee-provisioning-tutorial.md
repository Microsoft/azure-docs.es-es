---
title: 'Tutorial: Configuración de Foodee para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Foodee.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 08/30/2019
ms.author: Zhchia
ms.openlocfilehash: 40549fa8880fb4dc4d29046b0037621610f0b130
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544190"
---
# <a name="tutorial-configure-foodee-for-automatic-user-provisioning"></a>Tutorial: Tutorial: Configuración de Foodee para aprovisionar usuarios automáticamente

En este artículo se muestra cómo configurar Azure Active Directory (Azure AD) en Foodee y Azure AD para aprovisionar o desaprovisionar automáticamente usuarios o grupos en Foodee.

> [!NOTE]
> En este artículo se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para saber qué hace este servicio y de su funcionamiento, así como obtener las respuestas proporcionadas a las preguntas que se realizan con más frecuencia al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información sobre la característica Términos de uso de Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial se supone que cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Foodee](https://www.food.ee/about/)
* Una cuenta de usuario de Foodee con permisos de administrador

## <a name="assign-users-to-foodee"></a>Asignación de usuarios a Foodee 

Azure AD usa un concepto que se denomina *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Foodee. Una vez haya tomado esta determinación, puede asignar estos usuarios o grupos a Foodee si sigue las instrucciones de [Asignación de un usuario o un grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-foodee"></a>Sugerencias importantes para asignar usuarios a Foodee 

Cuando esté asignando usuarios, tenga en cuenta las siguientes sugerencias:

* Se recomienda asignar solo un usuario individual de Azure AD a Foodee para probar la configuración del aprovisionamiento automático de usuarios. Asigne usuarios o grupos adicionales más tarde.

* Al asignar un usuario a Foodee, debe seleccionar un rol válido específico de la aplicación, si está disponible, en el panel **Asignación**. Los usuarios que tienen el rol *Acceso predeterminado* quedan excluidos del aprovisionamiento.

## <a name="set-up-foodee-for-provisioning"></a>Configuración de Foodee para el aprovisionamiento

Antes de configurar Foodee para el aprovisionamiento automático de usuarios con Azure AD, debe habilitar el aprovisionamiento del sistema de administración de identidades entre dominios (SCIM) en Foodee.

1. Inicie sesión en [Foodee](https://www.food.ee/login/) y, a continuación, seleccione el identificador de inquilino.

    ![Foodee](media/Foodee-provisioning-tutorial/tenant.png)

1. En **Enterprise Portal**, seleccione **Inicio de sesión único**.

    ![Menú del panel izquierdo de Enterprise Portal de Foodee](media/Foodee-provisioning-tutorial/scim.png)

1. Copie el valor del cuadro **Token de API** para su uso posterior. Lo escribirá en el campo **Token secreto** de la pestaña **Aprovisionamiento** de la aplicación Foodee en Azure Portal.

    ![Foodee](media/Foodee-provisioning-tutorial/token.png)

## <a name="add-foodee-from-the-gallery"></a>Incorporación de Foodee desde la galería

Para configurar Foodee para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Foodee desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

Para agregar Foodee desde la galería de aplicaciones de Azure AD, haga lo siguiente:

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Comando de Azure Active Directory](common/select-azuread.png)

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una aplicación nueva, en la parte superior del panel, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **Foodee**, seleccione **Foodee** en el panel de resultados y luego seleccione **Agregar** para agregar la aplicación.

    ![Foodee en la lista de resultados](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-foodee"></a>Configuración del aprovisionamiento automático de usuarios en Foodee 

En esta sección, configurará el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Foodee en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Foodee si sigue las instrucciones del [tutorial de inicio de sesión único de Foodee](Foodee-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque las dos características se complementan entre sí.

Para configurar el aprovisionamiento automático de usuarios para Foodee en Azure AD, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com), seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista **Aplicaciones**, seleccione **Foodee**.

    ![Vínculo a Foodee en la lista de aplicaciones](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

1. En la lista desplegable **Modo de aprovisionamiento**, seleccione **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

1. En **Credenciales de administrador**, haga lo siguiente:

   a. En el cuadro **URL de inquilino**, escriba el valor **https:\//concierge.food.ee/scim/v2** que recuperó antes.

   b. En el cuadro **Token secreto**, escriba el valor de **Token de API** que recuperó antes.
   
   c. Para asegurarse de que Azure AD puede conectarse a Foodee, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de Foodee tiene permisos de administrador y pruebe de nuevo.

    ![Vínculo Probar conexión](common/provisioning-testconnection-tenanturltoken.png)

1. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Cuadro de texto Correo electrónico de notificación](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Foodee** (Sincronizar usuarios de Azure Active Directory con Foodee).

    ![Asignaciones de usuario de Foodee](media/Foodee-provisioning-tutorial/usermapping.png)

1. En **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan desde Azure AD a Foodee. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las *cuentas de usuario* de Foodee con el objetivo de realizar operaciones de actualización. 

    ![Asignaciones de usuario de Foodee](media/Foodee-provisioning-tutorial/userattribute.png)

1. Para confirmar los cambios, seleccione **Guardar**.
1. En **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Foodee** (Sincronizar grupos de Azure Active Directory con Foodee).

    ![Asignaciones de usuario de Foodee](media/Foodee-provisioning-tutorial/groupmapping.png)

1. En **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan desde Azure AD a Foodee. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las *cuentas de grupo* de Foodee con el objetivo de realizar operaciones de actualización.

    ![Asignaciones de usuario de Foodee](media/Foodee-provisioning-tutorial/groupattribute.png)

1. Para confirmar los cambios, seleccione **Guardar**.
1. Configure los filtros de ámbito. Para aprender a hacerlo, consulte las instrucciones del artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Foodee, en la sección **Configuración**, cambie el **Estado de aprovisionamiento** a **Activado**.

    ![El conmutador Estado de aprovisionamiento](common/provisioning-toggle-on.png)

1. En **Configuración**, en la lista desplegable **Ámbito**, defina los usuarios o grupos que desea aprovisionar para Foodee.

    ![Lista desplegable Ámbito de aprovisionamiento](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Botón Guardar para la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

La operación anterior inicia la sincronización inicial de los usuarios o grupos que ha definido en la lista desplegable **Ámbito**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Para más información, consulte [¿Cuánto tiempo se tarda en aprovisionar usuarios?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Puede usar la sección **Estado actual** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento. En el informe se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Foodee. Para obtener más información, vea [Comprobación del estado de aprovisionamiento](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Para leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
