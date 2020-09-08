---
title: 'Tutorial: Configuración de Jive para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Jive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: fec959ee34f9491b6510390d4b390e652690897e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547093"
---
# <a name="tutorial-configure-jive-for-automatic-user-provisioning"></a>Tutorial: Configuración de Jive para aprovisionar usuarios automáticamente

El objetivo de este tutorial es explicar los pasos que debe realizar en Jive y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD para Jive.

## <a name="prerequisites"></a>Requisitos previos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

*   Un inquilino de Azure Active Directory.
*   Una suscripción habilitada para inicio de sesión único en Jive.
*   Una cuenta de usuario de Jive con permisos de Team Admin (administrador de equipo).

## <a name="assigning-users-to-jive"></a>Asignación de usuarios a Jive

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD.

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceso a la aplicación Jive. Una vez decidido, puede asignar estos usuarios a la aplicación de Jive siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-jive"></a>Sugerencias importantes para asignar usuarios a Jive

*   Se recomienda asignar un único usuario de Azure AD a Jive para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

*   Al asignar a un usuario a Jive, debe seleccionar un rol de usuario válido. El rol "Acceso predeterminado" no funciona para realizar el aprovisionamiento.

## <a name="enable-user-provisioning"></a>Habilitación del aprovisionamiento de usuarios

Esta sección le guía en la conexión de Azure AD a la API de aprovisionamiento de cuentas de usuario de Jive, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de Jive en función de la asignación de grupos y usuarios Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Jive siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-user-account-provisioning"></a>Para configurar el aprovisionamiento automático de cuentas de usuario:

El objetivo de esta sección es describir cómo habilitar el aprovisionamiento de cuentas de usuario de Active Directory para Jive.
Como parte de este procedimiento, es necesario proporcionar un token de seguridad de usuario que deberá solicitar a Jive.com.

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

1. Si ya ha configurado Jive para el inicio de sesión único, busque la instancia de Jive mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Jive** en la Galería de aplicaciones. Seleccione Jive en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

1. Seleccione la instancia de Jive y, después, seleccione la pestaña **Aprovisionamiento**.

1. Establezca el **modo de aprovisionamiento** en **Automático**. 

    ![Aprovisionamiento](./media/jive-provisioning-tutorial/provisioning.png)

1. En la sección **Credenciales de administrador**, proporcione los siguientes valores de configuración:
   
    a. En el cuadro de texto **Nombre de usuario del administrador de Jive**, escriba un nombre de cuenta de Jive que tenga asignado el perfil **Administrador del sistema** en Jive.com.
   
    b. En el cuadro de texto **Contraseña de administrador de Jive** , escriba la contraseña para esta cuenta.
   
    c. En el cuadro de texto **Dirección URL de inquilino de Jive** , escriba la dirección URL del inquilino de Jive.
      
      > [!NOTE]
      > La dirección URL del inquilino de Jive es la que se usa en su organización para iniciar sesión en Jive.  
      > Normalmente, la dirección URL tiene el formato siguiente: **www.\<organization\>.jive.com**.          

1. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación de Jive.

1. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla que aparece a continuación.

1. Haga clic en **Guardar**.

1. En la sección Asignaciones, seleccione **Sincronizar usuarios de Azure Active Directory con Jive.**

1. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y Jive. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Jive con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Jive, cambie el **Estado de aprovisionamiento** a **Activado** en la sección Configuración

1. Haga clic en **Guardar**.

Esta acción inicia la sincronización inicial de todos los usuarios y grupos asignados a Jive en la sección Usuarios y grupos. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos a los registros de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento en la aplicación de Jive.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración del inicio de sesión único](jive-tutorial.md)
