---
title: 'Tutorial: Configuración de ServiceNow para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD para ServiceNow.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: joflore
ms.assetid: 4d6f06dd-a798-4c22-b84f-8a11f1b8592a
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/10/2019
ms.author: jeedes
ms.openlocfilehash: 099216d5ffde48c404ae733c09b568b3859c25ad
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922207"
---
# <a name="tutorial-configure-servicenow-for-automatic-user-provisioning"></a>Tutorial: Configuración de ServiceNow para el aprovisionamiento automático de usuarios

En este tutorial, se describen los pasos que debe realizar en ServiceNow y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y desaprovisiona automáticamente usuarios y grupos en [ServiceNow](https://www.servicenow.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en ServiceNow
> * Quitar usuarios de ServiceNow cuando ya no necesiten el acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y ServiceNow
> * Aprovisionamiento de grupos y pertenencias a grupos en ServiceNow
> * [Inicio de sesión único](servicenow-tutorial.md) en ServiceNow (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una [instancia de ServiceNow](https://www.servicenow.com/) de Calgary o superior
* Una [instancia de ServiceNow Express](https://www.servicenow.com/) de Helsinki o superior
* Una cuenta de usuario de ServiceNow con el rol de administrador

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos [se asignarán entre Azure AD y ServiceNow](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-servicenow-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de ServiceNow para admitir el aprovisionamiento con Azure AD

1. Identifique el nombre de la instancia de ServiceNow. Puede encontrar el nombre de instancia en la dirección URL que usa para acceder a ServiceNow. En el ejemplo siguiente, el nombre de instancia es dev35214.

   ![Instancia de ServiceNow](media/servicenow-provisioning-tutorial/servicenow_instance.png)

2. Obtenga las credenciales de un administrador de ServiceNow. Navegue hasta el perfil de usuario en ServiceNow y compruebe que el usuario tiene el rol de administrador. 

   ![Rol de administrador de ServiceNow](media/servicenow-provisioning-tutorial/servicenow-admin-role.png)

3. Asegúrese de que la siguiente configuración está **deshabilitada** en ServiceNow:

   1. Seleccione **System Security (Seguridad del sistema)**  > **High security settings (Configuración de alta seguridad)**  > **Require basic authentication for incoming SCHEMA requests (Requerir autenticación básica para solicitudes SCHEMA entrantes)** .
   2. Seleccione **System Properties (Propiedades del sistema)**  > **Web Services (Servicios web)**  > **Require basic authorization for incoming SOAP requests (Requerir autenticación básica para solicitudes SOAP entrantes)** .
     
   > [!IMPORTANT]
   > Si esta configuración está *habilitada*, el motor de aprovisionamiento no podrá comunicarse con ServiceNow.

## <a name="step-3-add-servicenow-from-the-azure-ad-application-gallery"></a>Paso 3. Agregar ServiceNow desde la galería de aplicaciones de Azure AD

Agregue ServiceNow desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en ServiceNow. Si ha configurado previamente ServiceNow para SSO, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a ServiceNow, debe seleccionar un rol que no sea **de acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-servicenow"></a>Paso 5. Configurar el aprovisionamiento automático de usuarios en ServiceNow 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-servicenow-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para ServiceNow en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ServiceNow**.

    ![Vínculo a ServiceNow en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba sus credenciales de administrador y su nombre de usuario de ServiceNow. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a ServiceNow. Si la conexión no se establece, asegúrese de que la cuenta de ServiceNow tiene permisos de administrador e inténtelo de nuevo.

    ![Aprovisionamiento](./media/servicenow-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to** (Sincronizar usuarios de Azure Active Directory con ServiceNow).

9. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y ServiceNow. Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con las cuentas de usuario de ServiceNow con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de ServiceNow admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to SmartFile** (Sincronizar grupos de Azure Active Directory con ServiceNow).

11. En la sección **Asignaciones de atributos**, revise los atributos de grupo que se sincronizan entre Azure AD y ServiceNow. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de ServiceNow con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para ServiceNow, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en ServiceNow.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="troubleshooting-tips"></a>Sugerencias para la solución de problemas
* **InvalidLookupReference:** al aprovisionar ciertos atributos, como departamento y la ubicación en ServiceNow, los valores ya deben existir en una tabla de referencia de ServiceNow. Por ejemplo, puede tener dos ubicaciones (Seattle, Los Angeles) y tres departamentos (ventas, finanzas, marketing) en la tabla **insertar nombre de tabla** en ServiceNow. Si intenta aprovisionar un usuario del departamento "ventas" y la ubicación "Seattle", se aprovisionará correctamente. Si intenta aprovisionar un usuario del departamento "ventas" y la ubicación "LA", el usuario no se aprovisionará. La ubicación LA debe agregarse a la tabla de referencia en ServiceNow o el atributo de usuario de Azure AD debe actualizarse para que coincida con el formato en ServiceNow. 
* **EntryJoiningPropertyValueIsMissing:** Revise las [asignaciones de atributos ](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes) para identificar el atributo coincidente. Este valor debe estar presente en el usuario o grupo que está intentando aprovisionar. 
* Revise la [API de SOAP de ServiceNow](https://docs.servicenow.com/bundle/newyork-application-development/page/integrate/web-services-apis/reference/r_DirectWebServiceAPIFunctions.html) para comprender los requisitos o las limitaciones (por ejemplo, el formato para especificar el código de país de un usuario).
* Las solicitudes de aprovisionamiento se envían de forma predeterminada a https://{nombre-de-instancia}.service-now.com/{nombre-de-tabla}. Si necesita una dirección URL de inquilino personalizada, puede proporcionar la dirección URL completa en el campo de nombre de instancia.
* **ServiceNowInstanceInvalid** 
  
  `Details: Your ServiceNow instance name appears to be invalid.  Please provide a current ServiceNow administrative user name and          password along with the name of a valid ServiceNow instance.`                                                              

   Este error indica un problema de comunicación con la instancia de ServiceNow. Asegúrese de que la siguiente configuración está *deshabilitada* en ServiceNow:
   
   1. Seleccione **System Security (Seguridad del sistema)**  > **High security settings (Configuración de alta seguridad)**  > **Require basic authentication for incoming SCHEMA requests (Requerir autenticación básica para solicitudes SCHEMA entrantes)** .
   2. Seleccione **System Properties (Propiedades del sistema)**  > **Web Services (Servicios web)**  > **Require basic authorization for incoming SOAP requests (Requerir autenticación básica para solicitudes SOAP entrantes)** .

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
