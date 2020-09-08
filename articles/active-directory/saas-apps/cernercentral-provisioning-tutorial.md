---
title: 'Tutorial: Aprovisionamiento de usuarios para Cerner Central: Azure AD'
description: Aprenda a configurar Azure Active Directory para aprovisionar usuarios en una lista de Cerner Central automáticamente.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: arvinh
ms.openlocfilehash: 58b991f5b229d924bc933ff34987db24bc895e10
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88529872"
---
# <a name="tutorial-configure-cerner-central-for-automatic-user-provisioning"></a>Tutorial: Configuración de Cerner Central para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que debe realizar en Cerner Central y Azure AD para aprovisionar y cancelar el aprovisionamiento de cuentas de usuario de Azure AD en una lista de Cerner Central automáticamente.

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure Active Directory
* Un inquilino de Cerner Central

> [!NOTE]
> Azure Active Directory se integra con Cerner Central mediante el protocolo [SCIM](http://www.simplecloud.info/).

## <a name="assigning-users-to-cerner-central"></a>Asignación de usuarios a Cerner Central

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación en Azure AD. 

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios y/o grupos de Azure AD representan a los usuarios que necesitan acceso a Cerner Central. Una vez decidido, puede asignar estos usuarios a la aplicación Cerner Central siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cerner-central"></a>Sugerencias importantes para asignar usuarios a Cerner Central

* Se recomienda asignar un solo usuario de Azure AD a Cerner Central para probar la configuración del aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

* Una vez completada la prueba inicial para un solo usuario, Cerner Central recomienda que se aprovisione en la lista de usuarios de Cerner la asignación de toda la lista de usuarios para que tengan acceso a cualquier solución de Cerner (no solo Cerner Central).  Otras soluciones de Cerner aprovechan esta lista de usuarios en la lista de usuarios de Cerner.

* Al asignar un usuario a Cerner Central, debe seleccionar el rol **Usuario** en el cuadro de diálogo de asignación. Los usuarios con el rol de "Acceso predeterminado" quedan excluidos del aprovisionamiento.

## <a name="configuring-user-provisioning-to-cerner-central"></a>Configuración del aprovisionamiento de usuarios en Cerner Central

Esta sección le guía a través de los pasos necesarios para conectar Azure AD con la lista de usuarios de Cerner Central mediante la API de aprovisionamiento de la cuenta de usuario SCIM de Cerner Central, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas en Cerner Central en función de la asignación de grupos y usuarios en Azure AD.

> [!TIP]
> Además, para habilitar el inicio de sesión único basado en SAML para Cerner Central, puede seguir las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí. Para más información, vea el [Tutorial: Integración de Azure Active Directory con Cerner Central](cernercentral-tutorial.md).

### <a name="to-configure-automatic-user-account-provisioning-to-cerner-central-in-azure-ad"></a>Para configurar el aprovisionamiento de cuentas de usuario automático para Cerner Central en Azure AD, siga estos pasos:

Para aprovisionar cuentas de usuario en Cerner Central, debe crear una cuenta del sistema de Cerner Central en Cerner y generar un token de portador de OAuth que Azure AD puede usar para conectarse al punto de conexión SCIM de Cerner. También se recomienda realizar la integración en un entorno de espacio aislado de Cerner antes de su implementación en producción.

1. El primer paso consiste en asegurarse de que las personas que administran la integración de Cerner y Azure AD tienen una cuenta de CernerCare, obligatoria para acceder a la documentación necesaria para llevar a cabo las instrucciones. Si es necesario, utilice las direcciones URL siguientes para crear cuentas de CernerCare en los entornos correspondientes.

   * Espacio aislado: https://sandboxcernercare.com/accounts/create

   * Producción: https://cernercare.com/accounts/create  

2. A continuación, se debe crear una cuenta de sistema para Azure AD. Siga estas instrucciones para solicitar una cuenta del sistema para los entornos de producción y de espacio aislado.

   * Instrucciones: https://wiki.ucern.com/display/CernerCentral/Requesting+A+System+Account

   * Espacio aislado: https://sandboxcernercentral.com/system-accounts/

   * Producción: https://cernercentral.com/system-accounts/

3. A continuación, genere un token de portador de OAuth para cada una de las cuentas del sistema. Para ello, siga las instrucciones que se describen a continuación.

   * Instrucciones: https://wiki.ucern.com/display/public/reference/Accessing+Cerner%27s+Web+Services+Using+A+System+Account+Bearer+Token

   * Espacio aislado: https://sandboxcernercentral.com/system-accounts/

   * Producción: https://cernercentral.com/system-accounts/

4. Por último, para completar la configuración, tiene que adquirir identificadores de dominio kerberos de lista de usuarios tanto para el espacio aislado como para los entornos de producción en Cerner. Para obtener información sobre cómo adquirirlo, consulte https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+SCIM. 

5. Ahora puede configurar Azure AD para aprovisionar cuentas de usuario para Cerner. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

6. Si ya ha configurado Cerner Central para el inicio de sesión único, busque la instancia de Cerner Central con el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Cerner Central** en la galería de aplicaciones. Seleccione Cerner Central en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

7. Seleccione la instancia de Cerner Central y la pestaña **Aprovisionamiento**.

8. Establezca el **modo de aprovisionamiento** en **Automático**.

   ![Aprovisionamiento de Cerner Central](./media/cernercentral-provisioning-tutorial/Cerner.PNG)

9. Rellene los campos siguientes en **Credenciales de administrador**:

   * En el campo **URL de inquilino**, escriba una dirección URL en el formato siguiente y sustituya "User-Roster-Realm-ID" por el identificador de dominio kerberos que obtuvo en el paso 4.

    > Espacio aislado: https://user-roster-api.sandboxcernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 
    > 
    > Producción: https://user-roster-api.cernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 

   * En el campo **Token secreto**, escriba el token de portador de OAuth que generó en el paso 3 y haga clic en **Prueba de conexión**.

   * Debería ver una notificación que le indica que el proceso se ha realizado correctamente en el lado superior derecho del portal.

1. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla que aparece a continuación.

1. Haga clic en **Save**(Guardar).

1. En la sección **Asignaciones de atributos**, revise los atributos de usuario y de grupo que se van a sincronizar desde Azure AD con Cerner Central. Los atributos seleccionados como propiedades **Matching** se usarán para establecer coincidencias con las cuentas de usuario y los grupos de Cerner Central para las operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Cerner Central, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

1. Haga clic en **Save**(Guardar).

Esta acción inicia la sincronización inicial de todos los usuarios y grupos asignados a Cerner Central en la sección Usuarios y grupos. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos a los registros de actividad de aprovisionamiento, que describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento en la aplicación de Cerner Central.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Cerner Central: Publishing identity data using Azure AD](https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+Azure+AD) (Cerner Central: Publicación de datos de identidad mediante Azure AD)
* [Tutorial: Configuración de Cerner Central para el inicio de sesión único con Azure Active Directory](cernercentral-tutorial.md)
* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/active-directory-saas-provisioning-reporting).
