---
title: 'Tutorial: Configuración de NetSuite OneWorld para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Más información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y NetSuite OneWorld.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 158b66d63292a1ffb503f3f87eddfd33a2b09b71
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554488"
---
# <a name="tutorial-configuring-netsuite-for-automatic-user-provisioning"></a>Tutorial: Configuración de Netsuite para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que hay que realizar en NetSuite OneWorld y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD en NetSuite.

## <a name="prerequisites"></a>Prerequisites

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

*   Un inquilino de Azure Active Directory.
*   Una suscripción a NetSuite OneWorld. Tenga en cuenta que el aprovisionamiento automático de usuarios solo es compatible con NetSuite OneWorld.
*   Una cuenta de usuario de NetSuite con permisos de administrador.

## <a name="assigning-users-to-netsuite-oneworld"></a>Asignación de usuarios a NetSuite OneWorld

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación en Azure AD.

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceso a la aplicación Netsuite. Una vez decidido, puede asignar estos usuarios a la aplicación Netsuite siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-netsuite-oneworld"></a>Sugerencias importantes para asignar usuarios a NetSuite OneWorld

*   Se recomienda asignar un único usuario de Azure AD a Netsuite para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

*   Al asignar a un usuario a Netsuite, debe seleccionar un rol de usuario válido. El rol "Acceso predeterminado" no funciona para realizar el aprovisionamiento.

## <a name="enable-user-provisioning"></a>Habilitación del aprovisionamiento de usuarios

Esta sección le guía en la conexión de Azure AD a la API de aprovisionamiento de cuentas de usuario de Netsuite, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de Netsuite en función de la asignación de grupos y usuarios Azure AD.

> [!TIP] 
> También puede habilitar el inicio de sesión único basado en SAML para Netsuite siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-user-account-provisioning"></a>Para configurar el aprovisionamiento automático de cuentas de usuario:

El objetivo de esta sección es describir cómo habilitar el aprovisionamiento de cuentas de usuario de Active Directory para Netsuite.

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

1. Si ya ha configurado Netsuite para el inicio de sesión único, busque la instancia de Netsuite mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Netsuite** en la galería de aplicaciones. Seleccione Netsuite en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

1. Seleccione la instancia de Netsuite y luego, la pestaña **Aprovisionamiento**.

1. Establezca el **modo de aprovisionamiento** en **Automático**. 

    ![Aprovisionamiento](./media/netsuite-provisioning-tutorial/provisioning.png)

1. En la sección **Credenciales de administrador**, proporcione los siguientes valores de configuración:
   
    a. En el cuadro de texto **Nombre de usuario administrador**, escriba un nombre de cuenta de Netsuite que tenga asignado el perfil **Administrador del sistema** en Netsuite.com.
   
    b. En el cuadro de texto **Contraseña de administrador**, escriba la contraseña de esta cuenta.
      
1. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación Netsuite.

1. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla.

1. Haga clic en **Guardar**.

1. En la sección Asignaciones, seleccione **Sincronizar usuarios de Azure Active Directory con Netsuite.**

1. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y Netsuite. Tenga en cuenta que los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con las cuentas de usuario de Netsuite con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Netsuite, cambie el **Estado de aprovisionamiento** a **Activado** en la sección Configuración

1. Haga clic en **Guardar**.

Esta acción inicia la sincronización inicial de todos los usuarios y grupos asignados a Netsuite en la sección Usuarios y grupos. Tenga en cuenta que la sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos a los registros de actividad de aprovisionamiento, que describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento en la aplicación de Netsuite.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración del inicio de sesión único](netsuite-tutorial.md)
