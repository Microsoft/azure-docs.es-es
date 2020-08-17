---
title: 'Tutorial: Aprovisionamiento de usuarios para Slack: Azure AD'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Slack.
services: active-directory
documentationcenter: ''
author: ArvindHarinder1
manager: CelesteDG
ms.assetid: d4ca2365-6729-48f7-bb7f-c0f5ffe740a3
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/06/2020
ms.author: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9aa97595f9a6ab2a866a8c8ebccde7e53854dbd1
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924553"
---
# <a name="tutorial-configure-slack-for-automatic-user-provisioning"></a>Tutorial: Configuración de Slack para aprovisionar usuarios automáticamente

El objetivo de este tutorial es explicar los pasos que debe realizar en Slack y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD para Slack. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Slack
> * Quitar usuarios de Slack cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Slack
> * Aprovisionar grupos y pertenencias a grupos en Slack
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/slack-tutorial) en Slack (recomendado)


## <a name="prerequisites"></a>Requisitos previos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* Un inquilino de Slack con el [plan Plus](https://aadsyncfabric.slack.com/pricing) o uno superior habilitado.
* Una cuenta de usuario de Slack con permisos de administrador de equipo.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y Slack](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-add-slack-from-the-azure-ad-application-gallery"></a>Paso 2. Adición de Slack desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Slack, agregue Slack desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Slack para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-3-define-who-will-be-in-scope-for-provisioning"></a>Paso 3. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Slack, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).

## <a name="step-4-configure-automatic-user-provisioning-to-slack"></a>Paso 4. Configuración del aprovisionamiento automático de usuarios a Slack 

Esta sección lo guía a través de los pasos necesarios para conectar la API de aprovisionamiento de su cuenta de usuario de Slack, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de Slack en función de la asignación de grupos y usuarios Azure AD.

### <a name="to-configure-automatic-user-account-provisioning-to-slack-in-azure-ad"></a>Para configurar el aprovisionamiento de cuentas de usuario automático para Slack en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Slack**.

    ![Vínculo a Slack en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En **Credenciales de administrador**, haga clic en **Autorizar**. Se abrirá un cuadro de diálogo de autorización de Slack en una nueva ventana del explorador.

    ![Authorization](media/slack-provisioning-tutorial/authorization.png)


6. En esa nueva ventana, inicie sesión en Slack con su cuenta de administrador de equipo. En el cuadro de diálogo de autorización que aparece, seleccione el equipo de Slack para el que desea habilitar el aprovisionamiento y, luego, seleccione **Autorizar**. Cuando termina, vuelva a Azure Portal para completar la configuración de aprovisionamiento.

    ![Cuadro de diálogo de autorización](./media/slack-provisioning-tutorial/slackauthorize.png)

7. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación de Slack. Si se produce un error de conexión, asegúrese de que su cuenta de Slack tiene permisos de administrador de equipo y vuelva a intentar el paso de "Autorizar".

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Seleccione **Guardar**.

10. En la sección Asignaciones, seleccione **Synchronize Azure Active Directory Users to** (Sincronizar usuarios de Azure Active Directory con Slack).

11. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizarán de Azure AD a Slack. Tenga en cuenta que los atributos seleccionados como propiedades de **Coincidencia** se usarán para establecer coincidencias con las cuentas de usuario de Slack con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |active|Boolean|
   |externalId|String|
   |DisplayName|String|
   |name.familyName|String|
   |name.givenName|String|
   |title|String|
   |emails[type eq "work"].value|String|
   |userName|String|
   |nickName|String|
   |addresses[type eq "untyped"].streetAddress|String|
   |addresses[type eq "untyped"].locality|String|
   |addresses[type eq "untyped"].region|String|
   |addresses[type eq "untyped"].postalCode|String|
   |addresses[type eq "untyped"].country|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "work"].value|String|
   |roles[primary eq "True"].value|String|
   |locale|String|
   |name.honorificPrefix|String|
   |photos[type eq "photo"].value|String|
   |profileUrl|String|
   |timezone|String|
   |userType|String|
   |urn:scim:schemas:extension:enterprise:1.0.department|String|
   |urn:scim:schemas:extension:enterprise:1.0.manager|Referencia|
   |urn:scim:schemas:extension:enterprise:1.0.employeeNumber|String|
   |urn:scim:schemas:extension:enterprise:1.0.costCenter|String|
   |urn:scim:schemas:extension:enterprise:1.0.organization|String|
   |urn:scim:schemas:extension:enterprise:1.0.division|String|

12. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Slack** (Sincronizar grupos de Azure Active Directory con Slack).

13. En la sección **Asignaciones de atributos**, revise los atributos de grupo que se sincronizarán de Azure AD a Slack. Tenga en cuenta que los atributos seleccionados como propiedades de **Coincidencia** se usarán para establecer coincidencias con los grupos de Slack con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |members|Referencia|

14. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Slack, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

16. Seleccione los valores deseados en **Ámbito**, en la sección **Configuración** para definir los usuarios o grupos que quiere aprovisionar en Slack.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

17. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-5-monitor-your-deployment"></a>Paso 5. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

## <a name="troubleshooting-tips"></a>Sugerencias para la solución de problemas

* Cuando configure el atributo  **displayName** de Slack, tenga en cuenta los siguientes comportamientos:

  * Los valores no son completamente únicos (por ejemplo, dos usuarios pueden tener el mismo nombre para mostrar).

  * Admite caracteres no latinos, espacios y mayúsculas y minúsculas. 
  
  * La puntuación permitida incluye puntos, caracteres de subrayado, guiones, apóstrofes, corchetes (por ejemplo, **( [ { } ] )** ) y separadores (por ejemplo, **, / ;** ).
  
  * la propiedad displayName no puede tener un carácter "@". Si se incluye un carácter "@", es posible que encuentre un evento omitido en los registros de aprovisionamiento con la descripción "AttributeValidationFailed".

  * Solo se actualiza si estos dos parámetros están configurados en el área de trabajo y organización de Slack - **La sincronización de perfiles está habilitada** y **Los usuarios no pueden cambiar su nombre para mostrar**.

* El atributo **userName** de Slack debe tener menos de 21 caracteres y tener un valor único.

* Slack solo permite la coincidencia con los atributos **userName** y **email**.  
  
* Los códigos de error comunes están documentados en la documentación oficial de Slack: https://api.slack.com/scim#errors

## <a name="change-log"></a>Registro de cambios

* 16/06/2020: se modifica el atributo DisplayName para que solo se actualice durante la creación de nuevos usuarios.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)