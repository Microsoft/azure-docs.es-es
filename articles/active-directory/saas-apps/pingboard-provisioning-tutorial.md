---
title: 'Tutorial: Aprovisionamiento de usuarios para Pingboard: Azure AD'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente las cuentas de usuario para Pingboard.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: arvinh
ms.openlocfilehash: 29e28e20bd9b471604a450ddb36ef867f0608e06
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553769"
---
# <a name="tutorial-configure-pingboard-for-automatic-user-provisioning"></a>Tutorial: Configuración de Pingboard para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que hay que realizar para habilitar el aprovisionamiento y el desaprovisionamiento automáticos de las cuentas de usuario de Azure Active Directory (Azure AD) a Pingboard.

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure AD
* Una [cuenta profesional](https://pingboard.com/pricing) de un inquilino de Pingboard
* Una cuenta de usuario de Pingboard con permisos de administrador

> [!NOTE]
> La integración del aprovisionamiento de Azure AD se basa ([API de Pingboard](https://pingboard.docs.apiary.io/#)) que está disponible para su cuenta.

## <a name="assign-users-to-pingboard"></a>Asignación de usuarios a Pingboard

Azure AD usa un concepto denominado "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios asignados a una aplicación en Azure AD. 

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios de Azure AD necesitan acceso a la aplicación de Pingboard. Una vez decidido, puede asignar estos usuarios a la aplicación de Pingboard siguiendo estas instrucciones:

[Asignación de un usuario a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-pingboard"></a>Sugerencias importantes para asignar usuarios a Pingboard

Se recomienda asignar un único usuario de Azure AD a Pingboard para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios.

## <a name="configure-user-provisioning-to-pingboard"></a>Configuración del aprovisionamiento de usuarios para Pingboard 

Esta sección le guía en el proceso de conectar su instancia de Azure AD a la API de aprovisionamiento de cuentas de Pingboard. También configurará el servicio de aprovisionamiento para crear, actualizar y deshabilitar las cuentas de usuario asignadas en Pingboard en función de las asignaciones de usuario en Azure AD.

> [!TIP]
> Para habilitar el inicio de sesión único basado en SAML para Pingboard, siga las instrucciones que se proporcionan en [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-account-provisioning-to-pingboard-in-azure-ad"></a>Configuración del aprovisionamiento automático de cuentas de usuario para Pingboard en Azure AD

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory**  > **Aplicaciones empresariales** > **Todas las aplicaciones**.

1. Si ya configuró Pingboard para el inicio de sesión único, busque la instancia de Pingboard mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Pingboard** en la galería de aplicaciones. Seleccione **Pingboard** en los resultados de búsqueda y agrégalo a la lista de aplicaciones.

1. Seleccione la instancia de Pingboard y, después, la pestaña **Aprovisionamiento**.

1. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de Pingboard](./media/pingboard-provisioning-tutorial/pingboardazureprovisioning.png)

1. En la sección **Credenciales de administración**, realice los pasos siguientes:

    a. En **URL de inquilino**, escriba `https://your_domain.pingboard.com/scim/v2` y sustituya "your_domain" por su dominio real.

    b. Inicie sesión en [Pingboard](https://pingboard.com/) mediante la cuenta de administrador.

    c. Seleccione **Add-Ons** >  (Complementos) **Integrations** >  (Integraciones) **Azure Active Directory**.

    d. Haga clic en la pestaña **Configure** (Configurar) y seleccione **Enable user provisioning from Azure** (Habilitar el aprovisionamiento de usuarios desde Azure).

    e. Copie el token de **OAuth Bearer Token** (Token de portador de OAuth) e insértelo en **Token secreto**.

1. En Azure Portal, seleccione **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación Pingboard. Si se produce un error de conexión, asegúrese de que su cuenta de Pingboard tiene permisos de administrador y vuelva a intentar el paso **Probar conexión**.

1. Escriba la dirección de correo electrónico de una persona o grupo que quiera que reciba las notificaciones de error de aprovisionamiento en **Correo electrónico de notificación**. Active la casilla situada debajo.

1. Seleccione **Guardar**.

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Pingboard** (Sincronizar usuarios de Azure Active Directory con Pingboard).

1. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se van a sincronizar entre Azure AD y Pingboard. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Pingboard con el objetivo de realizar operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**. Para más información, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios](../app-provisioning/customize-application-attributes.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Pingboard, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

1. Haga clic en **Guardar** para iniciar la sincronización inicial de los usuarios asignados a Pingboard.

La sincronización inicial tarda más tiempo en ejecutarse que las posteriores, que se producen aproximadamente cada 40 minutos, siempre y cuando se esté ejecutando el servicio. Use la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos a los registros de actividad de aprovisionamiento. En los registros se describen todas las acciones que realiza el servicio de aprovisionamiento en la aplicación Pingboard.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración del inicio de sesión único](pingboard-tutorial.md)
