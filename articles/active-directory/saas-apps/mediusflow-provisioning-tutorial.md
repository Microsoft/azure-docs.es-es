---
title: 'Tutorial: Configuración de MediusFlow para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y desaprovisionar automáticamente las cuentas de usuario de Azure AD en MediusFlow.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 04/30/2020
ms.author: Zhchia
ms.openlocfilehash: e8a64b3f42e52363f70d536963594bb6083a0992
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548079"
---
# <a name="tutorial-configure-mediusflow-for-automatic-user-provisioning"></a>Tutorial: Configuración de MediusFlow para el aprovisionamiento automático de usuarios

En este tutorial se describen los pasos que debe realizar en MediusFlow y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y desaprovisiona automáticamente usuarios y grupos en [MediusFlow](https://www.mediusflow.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en MediusFlow
> * Eliminación de usuarios de MediusFlow cuando ya no necesiten el acceso
> * Sincronización de los atributos de usuario entre Azure AD y MediusFlow
> * Aprovisionamiento de grupos y pertenencias a grupos en MediusFlow
> * Inicio de sesión único en MediusFlow (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una suscripción de MediusFlow activa con un inquilino de control de calidad o de producción.
* Una cuenta de usuario de MediusFlow con derechos de acceso de administrador para poder definir la configuración en MediusFlow.
* Las empresas agregadas en el inquilino de MediusFlow donde se deben aprovisionar los usuarios.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos se [asignarán entre Azure AD y MediusFlow](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-mediusflow-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de MediusFlow para admitir el aprovisionamiento con Azure AD

### <a name="activate-the-microsoft-365-app-within-mediusflow"></a>Activación de la aplicación de Microsoft 365 en MediusFlow
Para empezar, habilite el acceso del inicio de sesión de Azure AD y la característica de configuración de Azure AD en MediusFlow mediante los pasos siguientes:

#### <a name="user-login"></a>Inicio de sesión de usuario
Para habilitar el flujo de inicio de sesión en Microsoft 365/Azure AD, consulte este artículo (https://success.mediusflow.com/documentation/administration_guide/user_login_and_transfer/office365userintegration/#user-login-setup).

#### <a name="user-transfer-configuration"></a>Configuración de transferencia de usuarios
Para habilitar el portal de configuración de los usuarios para el aprovisionamiento desde Azure AD, consulte [este](
https://success.mediusflow.com/documentation/administration_guide/user_login_and_transfer/office365userintegration/#user-sync-setup) artículo.

#### <a name="configure-user-provisioning"></a>Configurar aprovisionamiento de usuarios

1.  Inicie sesión en la [consola de administración de MediusFlow](https://office365.cloudapp.mediusflow.com/) con el identificador de inquilino.

    ![Authenticate](./media/mediusflow-provisioning-tutorial/1-auth.png)

2. Compruebe la conexión con MediusFlow.

    ![Verify](./media/mediusflow-provisioning-tutorial/2-verify-connection.png)

3. Proporcione el id. de inquilino de Azure AD.

    ![Proporcionar el identificador de inquilino](./media/mediusflow-provisioning-tutorial/3-provide-azuread-tenantid.png)

   Puede obtener más información en las [preguntas más frecuentes ](https://success.mediusflow.com/documentation/administration_guide/user_login_and_transfer/office365userintegration/#how-do-i-get-the-azure-tenant-id) sobre cómo encontrarlo.

4. Guarde la configuración.

    ![Authenticate](./media/mediusflow-provisioning-tutorial/4-save-config.png)

5. Seleccione el aprovisionamiento de usuarios y haga clic en **Aceptar**.

    ![Authenticate](./media/mediusflow-provisioning-tutorial/5-select-user-provisioning.png)

6. Haga clic en **Generate Secret Key** (Generar clave secreta). Copie y guarde este valor, que se escribirá en el campo **Token secreto** de la pestaña **Aprovisionamiento** en la aplicación MediusFlow en Azure Portal.

    ![Authenticate](./media/mediusflow-provisioning-tutorial/6-create-secret-1.png)

7. Haga clic en **Aceptar**.

    ![Authenticate](./media/mediusflow-provisioning-tutorial/7-confirm-secret.png)

8. Para que los usuarios se importen con un conjunto predefinido de roles, compañías y otras opciones de configuración generales en MediusFlow, antes deberá configurar esta opción. Para empezar, haga clic en **Add new configuration** (Agregar nueva configuración).

    ![Configurar usuarios](./media/mediusflow-provisioning-tutorial/8-configure-user-configuration-1.png)

9. Proporcione la configuración predeterminada para los usuarios. En esta vista, puede configurar el atributo predeterminado. Si la configuración estándar es correcta, basta con proporcionar un nombre de empresa válido. Dado que estos valores de configuración se capturan de MediusFlow, antes se deben configurar. Para obtener más información, consulte la sección **Requisitos previos** de este artículo:

    ![Configurar usuarios](./media/mediusflow-provisioning-tutorial/9-configure-user-config-detail-1.png)

10. Haga clic en el botón **Guardar** para guardar la configuración de los usuarios.

    ![Configurar usuarios](./media/mediusflow-provisioning-tutorial/10-done-1.png)

11. Para obtener el vínculo de aprovisionamiento de usuarios, haga clic en **Copiar vínculo de SCIM**. Copie y guarde este valor. Este valor se escribirá en el campo **Dirección URL del inquilino** de la pestaña **Aprovisionamiento** de la aplicación MediusFLow en Azure Portal.
 
    ![Configurar usuarios](./media/mediusflow-provisioning-tutorial/11-get-scim-link.png)

## <a name="step-3-add-mediusflow-from-the-azure-ad-application-gallery"></a>Paso 3. Agregar MediusFLow desde la galería de aplicaciones de Azure AD

Agregue MediusFLow desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en MediusFLow. Si ha configurado previamente MediusFLow para SSO, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a MediusFLow, debe seleccionar un rol que no sea **de acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-mediusflow"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en MediusFlow 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-mediusflow-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para MediusFlow en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **MediusFlow**.

    ![Vínculo a MediusFLow en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba el valor de **URL de inquilino** recuperado anteriormente. Escriba el valor del token secreto recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a MediusFLow. Si la conexión no se establece, asegúrese de que la cuenta de MediusFLow tenga permisos de administrador y vuelva a intentarlo.

      ![Aprovisionamiento](./media/mediusflow-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con MediusFLow**.

9. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y MediusFLow. Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con las cuentas de usuario de MediusFlow con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de MediusFlow admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |userName|String|
   |emails[type eq "work"].value|String|
   |name.displayName|String|
   |active|Boolean|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |externalId|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Referencia|


10. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con MediusFLow**.

11. En la sección **Asignaciones de atributos**, revise los atributos de grupo que se sincronizan entre Azure AD y MediusFlow. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con los grupos de MediusFlow para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |externalId|String|
      |members|Referencia|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para MediusFlow, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en MediusFlow.

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
