---
title: 'Tutorial: Configuración de Webroot Security Awareness Training para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD a Webroot Security Awareness Training.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 455f4396-930e-4db5-a167-d3ea6a860a17
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2020
ms.author: Zhchia
ms.openlocfilehash: 0b9792ce25541917bfde3291b3864f58d70d6d03
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924043"
---
# <a name="tutorial-configure-webroot-security-awareness-training-for-automatic-user-provisioning"></a>Tutorial: Configuración de Webroot Security Awareness Training para el aprovisionamiento automático de usuarios

En este tutorial, se describen los pasos que debe realizar en Webroot Security Awareness Training y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y desaprovisiona automáticamente usuarios y grupos en [Webroot Security Awareness Training](https://www.webroot.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Webroot Security Awareness Training
> * Eliminar usuarios en WebRoot Security Awareness Training cuando ya no requieran acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Webroot Security Awareness Training
> * Aprovisionar grupos y pertenencias a grupos en WebRoot Security Awareness Training

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* Una consola de proveedor de servicios administrados con Webroot Security Awareness Training habilitado para al menos uno de los sitios.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos [asignar entre Azure AD y Webroot Security Awareness Training](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-webroot-security-awareness-training-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Webroot Security Awareness Training para admitir el aprovisionamiento con Azure AD

### <a name="obtain-a-secret-token"></a>Obtención de un token secreto

Para conectar el sitio a Azure AD, deberá obtener un **token secreto** para ese sitio en la consola de administración de Webroot.

1. Inicie sesión en la [consola de administración de Webroot](https://identity.webrootanywhere.com/v1/Account/login#tab_customers).

2. En la pestaña **Sitios**, haga clic en el icono de engranaje de la columna Security Awareness Training del sitio que quiera conectar con Azure AD.

    ![Icono de engranaje](./media/webroot-security-awareness-training-provisioning-tutorial/gear-icon.png)

3. Haga clic en el botón **Configurar la integración de Azure AD**.

    ![Configuración de la integración de Azure AD](./media/webroot-security-awareness-training-provisioning-tutorial/configure-azure-ad-integration.png)

4. Copie y guarde el **token secreto**. Este valor se escribirá en el campo Token secreto de la pestaña Aprovisionamiento de la aplicación Webroot Security Awareness Training en Azure Portal.

5. Haga clic en **Done**(Listo).

    ![Copia del token secreto](./media/webroot-security-awareness-training-provisioning-tutorial/copy-secret-token.png)

## <a name="step-3-add-webroot-security-awareness-training-from-the-azure-ad-application-gallery"></a>Paso 3. Incorporación de Webroot Security Awareness Training desde la galería de aplicaciones de Azure AD

Agregue Webroot Security Awareness Training desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento a WebRoot Security Awareness Training. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Webroot Security Awareness Training, debe seleccionar un rol que no sea **de acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-webroot-security-awareness-training"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios a Webroot Security Awareness Training 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-webroot-security-awareness-training-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Webroot Security Awareness Training en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Webroot Security Awareness Training**.

    ![Webroot Security Awareness Training en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://awarenessapi.webrootanywhere.com/api/v2/scim` en la **URL de inquilino**. Escriba el valor del token secreto recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Webroot Security Awareness Training. Si la conexión no se establece, asegúrese de que la cuenta de Webroot Security Awareness Training tiene permisos de administrador e inténtelo de nuevo.

    ![Aprovisionamiento](./media/webroot-security-awareness-training-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Aprovisionar usuarios de Azure Active Directory**.

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Webroot Security Awareness Training en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Webroot Security Awareness Training con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de Webroot Security Awareness Training admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |externalId|String|&check;|
   |name.givenName|String|
   |name.familyName|String|
   |emails[type eq "work"].value|String|

10. En la sección **Asignaciones**, seleccione **Aprovisionar grupos de Azure Active Directory**.

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Webroot Security Awareness Training en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Webroot Security Awareness Training con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|Compatible con el filtrado|
      |---|---|---|
      |DisplayName|String|&check;|
      |members|Referencia|
      |externalId|String|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Webroot Security Awareness Training, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Webroot Security Awareness Training.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)
