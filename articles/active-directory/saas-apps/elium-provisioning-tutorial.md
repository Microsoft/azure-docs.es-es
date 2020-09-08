---
title: 'Tutorial: Configuración de Elium para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario en Elium.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 08/19/2019
ms.author: Zhchia
ms.openlocfilehash: d9bb21ab4b5610c8318fe4ee869f1140faf55367
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544237"
---
# <a name="tutorial-configure-elium-for-automatic-user-provisioning"></a>Tutorial: Configuración de Elium para el aprovisionamiento automático de usuarios

En este tutorial se muestra cómo configurar Elium y Azure Active Directory (Azure AD) para aprovisionar y desaprovisionar automáticamente usuarios o grupos en Elium.

> [!NOTE]
> En este tutorial se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio y de su funcionamiento, así como ver las preguntas más frecuentes, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información acerca de los términos de uso generales para las características de Azure en la versión preliminar, consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

En este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Elium](https://www.elium.com/pricing/)
* Una cuenta de usuario de Elium con permisos de administrador

## <a name="assigning-users-to-elium"></a>Asignación de usuarios a Elium

Azure AD usa un concepto que se denomina *asignaciones* para determinar qué usuarios reciben acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios y grupos de Azure AD necesitan acceder a Elium. A continuación, asigne dichos usuarios y grupos a Ellium mediante los pasos que se indican en [Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-elium"></a>Sugerencias importantes para asignar usuarios a Elium 

Se recomienda asignar primero un usuario individual de Azure AD a Elium para probar la configuración del aprovisionamiento automático de usuarios. Más tarde, se pueden asignar más usuarios y grupos.

Al asignar un usuario a Elium, debe seleccionar un rol válido específico de la aplicación (si hay alguno disponible) en el cuadro de diálogo de asignación. Los usuarios que tienen el rol **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-elium-for-provisioning"></a>Configuración de Elium para el aprovisionamiento

Antes de configurar Elium para el aprovisionamiento automático de usuarios con Azure AD, debe habilitar el aprovisionamiento de sistema de administración de identidades entre dominios (SCIM) en Elium. Siga estos pasos:

1. Inicie sesión en Elium y vaya a **Mi perfil** > **Configuración**.

    ![Elemento de menú Configuración en Elium](media/Elium-provisioning-tutorial/setting.png)

1. En la esquina inferior izquierda en **AVANZADO**, seleccione **Seguridad**.

    ![Vínculo a Seguridad en Elium](media/Elium-provisioning-tutorial/security.png)

1. Copie los valores de **URL de inquilino** y **Token secreto**. Usará estos valores más tarde en los campos correspondientes de la pestaña **Aprovisionamiento** de la aplicación Elium en Azure Portal.

    ![Campos URL de inquilino y Token secreto en Elium](media/Elium-provisioning-tutorial/token.png)

## <a name="add-elium-from-the-gallery"></a>Adición de Elium desde la galería

Para configurar Elium para el aprovisionamiento automático de usuarios con Azure AD, debe agregar Elium desde la galería de aplicaciones de Azure AD a la lista de aplicaciones de software como servicio (SaaS) administradas. Siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Elemento de menú Azure Active Directory](common/select-azuread.png)

1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

     ![Hoja Aplicaciones empresariales de Azure AD](common/enterprise-applications.png)

1. Para agregar una aplicación nueva, en la parte superior del panel, seleccione **Nueva aplicación**.

    ![Vínculo a la aplicación nueva](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **Elium**, seleccione **Elium** en la lista de resultados y luego seleccione **Agregar** para agregar la aplicación.

    ![Cuadro de búsqueda de la galería](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-elium"></a>Configuración del aprovisionamiento automático de usuarios en Elium

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios y grupos en Elium en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único para Elium basado en el lenguaje de marcado de aserción de seguridad (SAML) siguiendo las instrucciones del [tutorial de inicio de sesión único de Elium](Elium-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque las dos características se complementan entre sí.

Para configurar el aprovisionamiento automático de usuarios para Elium en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione **Aplicaciones empresariales** y, a continuación, **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales de Azure AD](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Elium**.

    ![Lista de aplicaciones en la hoja Aplicaciones empresariales](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento en la hoja Aplicaciones empresariales](common/provisioning.png)

1. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Configuración automática para el modo de aprovisionamiento](common/provisioning-automatic.png)

1. En la sección **Credenciales de administrador**, escriba **\<tenantURL\>/scim/v2** en el campo **URL de inquilino**. (**tenantURL** es el valor recuperado anteriormente de la consola de administración de Elium). Escriba también el valor de **Token secreto** en el campo **Token secreto**. Por último, seleccione **Prueba de conexión** para comprobar si Azure AD se puede conectar a Elium. Si la conexión no se establece, asegúrese de que la cuenta de Elium tiene permisos de administrador e inténtelo de nuevo.

    ![Campos URL de inquilino y Token secreto en Credenciales de administrador](common/provisioning-testconnection-tenanturltoken.png)

1. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que recibirá las notificaciones de error de aprovisionamiento. A continuación, seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

1. Haga clic en **Save**(Guardar).

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Elium** (Sincronizar usuarios de Azure Active Directory con Elium).

    ![Vínculo a Sincronizar para asignar usuarios de Azure AD en Elium](media/Elium-provisioning-tutorial/usermapping.png)

1. Examine los atributos de usuario que se sincronizan entre Azure AD y Elium en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Elium con el objetivo de realizar operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**.

    ![Asignaciones de atributos entre Azure AD y Elium](media/Elium-provisioning-tutorial/userattribute.png)

1. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Elium, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento establecido en Activado](common/provisioning-toggle-on.png)

1. Para definir los usuarios y grupos que quiere que se aprovisionen en Elium, seleccione los valores deseados en el cuadro de lista desplegable **Ámbito** en la sección **Configuración**.

    ![Cuadro de lista Ámbito del aprovisionamiento](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Botón Guardar para la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El proceso de sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Para obtener más información acerca del tiempo necesario para el aprovisionamiento, consulte [¿Cuánto tiempo se tarda en aprovisionar usuarios?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Use la sección **Estado actual** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento. En el informe de actividad de aprovisionamiento se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Elium. Para obtener más información, vea [Comprobación del estado de aprovisionamiento](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Para leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
