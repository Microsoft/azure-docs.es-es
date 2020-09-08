---
title: 'Tutorial: Configuración de Velpic para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente las cuentas de usuario para Velpic.
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
ms.openlocfilehash: fbed4b888c48a518d9f10a91ff0494aa7bdc1843
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88532371"
---
# <a name="tutorial-configuring-velpic-for-automatic-user-provisioning"></a>Tutorial: Configuración de Velpic para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que debe realizar en Velpic y Azure AD para aprovisionar y desaprovisionar automáticamente las cuentas de usuario de Azure AD para Velpic.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure Active Directory
* Un inquilino de Velpic con el plan [Enterprise](https://www.velpic.com/pricing.html) o uno superior habilitado
* Una cuenta de usuario de Velpic con permisos de administrador

## <a name="assigning-users-to-velpic"></a>Asignación de usuarios a Velpic

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento de cuentas de usuario automático, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD. 

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceder a la aplicación de Velpic. Una vez decidido, puede asignar estos usuarios a la aplicación de Velpic siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-velpic"></a>Sugerencias importantes para asignar usuarios a Velpic

* Se recomienda asignar un único usuario de Azure AD a Velpic para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Velpic, debe seleccionar el rol **Usuario** u otro válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Tenga en cuenta que el rol **Acceso predeterminado** no funciona para realizar el aprovisionamiento y estos usuarios se omitirán.

## <a name="configuring-user-provisioning-to-velpic"></a>Configuración del aprovisionamiento de usuarios para Velpic

Esta sección lo guía a través de los pasos necesarios para conectar la API de aprovisionamiento de su cuenta de usuario de Velpic, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de Velpic en función de la asignación de grupos y usuarios Azure AD.

> [!TIP]
> También puede elegir habilitar el inicio de sesión único basado en SAML para Velpic siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-account-provisioning-to-velpic-in-azure-ad"></a>Para configurar el aprovisionamiento automático de cuentas de usuario para Velpic en Azure AD, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

2. Si ya ha configurado Velpic para el inicio de sesión único, busque la instancia de Velpic mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Velpic** en la galería de aplicaciones. Seleccione Velpic en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

3. Seleccione la instancia de Velpic y, después, la pestaña **Aprovisionamiento**.

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de Velpic](./media/velpic-provisioning-tutorial/Velpic1.png)

5. En la sección **Credenciales de administrador**, introduzca la **dirección URL de inquilino y el token secreto** de Velpic (puede encontrar estos valores en la cuenta de Velpic): **Manage** > **Integration** > **Plugin** > **SCIM** [Administrar > Integración > Complemento > SCIM]).

    ![Valores de autorización](./media/velpic-provisioning-tutorial/Velpic2.png)

6. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación Velpic. Si la conexión no se establece, asegúrese de que la cuenta de Velpic tiene permisos de administrador de equipo e intente el paso 5 de nuevo.

7. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla que aparece a continuación.

8. Haga clic en **Save**(Guardar).

9. En la sección Asignaciones, seleccione **Synchronize Azure Active Directory Users to Velpic** (Sincronizar usuarios de Azure Active Directory con Velpic).

10. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizarán de Azure AD con Velpic. Tenga en cuenta que los atributos seleccionados como propiedades de **Coincidencia** se usarán para establecer coincidencias con las cuentas de usuario de Velpic con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

11. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Velpic, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

12. Haga clic en **Save**(Guardar).

Esta acción iniciará la sincronización inicial de todos los usuarios y grupos asignados a Velpic en la sección Usuarios y grupos. Tenga en cuenta que la sincronización inicial tardará más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos a los informes de actividad de aprovisionamiento, que describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)