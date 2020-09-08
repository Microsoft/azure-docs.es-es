---
title: 'Tutorial: Configuración de G Suite para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD para G Suite.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 01/06/2020
ms.author: Zhchia
ms.openlocfilehash: 27e34a66bb6dfa642b84bd8997b2b02c4981788e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551565"
---
# <a name="tutorial-configure-g-suite-for-automatic-user-provisioning"></a>Tutorial: Configuración de G Suite para el aprovisionamiento automático de usuarios

En este tutorial se describen los pasos que debe realizar en G Suite y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [G Suite](https://gsuite.google.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

> [!NOTE]
> El conector de G Suite se actualizó recientemente, en octubre de 2019. Entre los cambios realizados en el conector de G Suite se incluyen los siguientes:
>
> * Se ha agregado compatibilidad con atributos adicionales de usuario y grupo de G Suite.
> * Se han actualizado los nombres de atributo de destino de G Suite para que coincidan con lo que se define [aquí](https://developers.google.com/admin-sdk/directory).
> * Se han actualizado las asignaciones de atributos predeterminadas.

## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en G Suite
> * Eliminar usuarios de G Suite cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y G Suite
> * Aprovisionar grupos y pertenencias a grupos en G Suite
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/google-apps-tutorial) en G Suite (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* [Un inquilino de G Suite](https://gsuite.google.com/pricing.html)
* Una cuenta de usuario en G Suite con permisos de administrador

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y G Suite](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-g-suite-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de G Suite para admitir el aprovisionamiento con Azure AD

Antes de configurar G Suite para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en G Suite.

1. Inicie sesión en la [consola de administración de G Suite](https://admin.google.com/) con su cuenta de administrador y haga clic en **Security** (Seguridad). Si no ve el vínculo, puede estar oculto debajo del menú **More Controls** (Más controles) en la parte inferior de la pantalla.

    ![Seguridad de G Suite](./media/google-apps-provisioning-tutorial/gapps-security.png)

2. En la página **Security** (Seguridad), haga clic en **API Reference** (Referencia de API).

    ![API de G Suite](./media/google-apps-provisioning-tutorial/gapps-api.png)

3. Seleccione **Enable API access**.

    ![API de G Suite habilitada](./media/google-apps-provisioning-tutorial/gapps-api-enabled.png)

    > [!IMPORTANT]
   > El nombre en Azure AD de cada uno de los usuarios que quiera aprovisionar en G Suite **tiene que** estar vinculado a un dominio personalizado. Por ejemplo, G Suite no acepta los nombres de usuario parecidos a bob@contoso.onmicrosoft.com. Por otro lado, bob@contoso.com se acepta. Puede cambiar el dominio de un usuario existente siguiendo las instrucciones indicadas [aquí](https://docs.microsoft.com/azure/active-directory/fundamentals/add-custom-domain).

4. Una vez que haya agregado y comprobado los dominios personalizados que desee con Azure AD, deberá comprobarlos de nuevo con G Suite. Para comprobar los dominios en G Suite, consulte los pasos siguientes:

    a. En la [consola de administración de G Suite](https://admin.google.com/), seleccione **Domains** (Dominios).

    ![Dominios de G Suite](./media/google-apps-provisioning-tutorial/gapps-domains.png)

    b. Haga clic en **Add a domain or a domain alias** (Agregar un dominio o un alias de dominio).

    ![Adición de un dominio de G Suite](./media/google-apps-provisioning-tutorial/gapps-add-domain.png)

    c. Seleccione **Add another domain** (Añadir otro dominio) y escriba el nombre del dominio que desee agregar.

    ![Adición de otro dominio de G Suite](./media/google-apps-provisioning-tutorial/gapps-add-another.png)

    d. Haga clic en **Continue and verify domain ownership** (Continuar y comprobar la propiedad del dominio). A continuación, siga los pasos para comprobar que posee el nombre de dominio. Para obtener instrucciones completas sobre cómo comprobar un dominio con Google, consulte [Verificar que eres el propietario de un sitio web](https://support.google.com/webmasters/answer/35179).

    e. Repita los pasos anteriores para todos los dominios adicionales que vaya a agregar a G Suite.

5. A continuación, determine qué cuenta de administrador quiere usar para administrar el aprovisionamiento de usuarios en G Suite. Vaya a **Admin Roles** (Roles de administrador).

    ![Administrador de G Suite](./media/google-apps-provisioning-tutorial/gapps-admin.png)

6. En el **rol administrativo** de esa cuenta, edite los valores de **Privileges** (Privilegios) para ese rol. Asegúrese de que habilita todos los **privilegios de la API de administración** para que esta cuenta pueda usarse para el aprovisionamiento.

    ![Privilegios de administración de G Suite](./media/google-apps-provisioning-tutorial/gapps-admin-privileges.png)

## <a name="step-3-add-g-suite-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de G Suite desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de G Suite, agregue G Suite desde la galería de aplicaciones de Azure AD. Si ha configurado previamente G Suite para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a G Suite, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-g-suite"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en G Suite 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

> [!NOTE]
> Para más información sobre el punto de conexión de la API de directorio de G Suite, consulte [Directory API](https://developers.google.com/admin-sdk/directory).

### <a name="to-configure-automatic-user-provisioning-for-g-suite-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para G Suite en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**. Los usuarios deberán iniciar sesión en portal.azure.com y no podrán usar aad.portal.azure.com

    ![Hoja Aplicaciones empresariales](./media/google-apps-provisioning-tutorial/enterprise-applications.png)

    ![Hoja Todas las aplicaciones](./media/google-apps-provisioning-tutorial/all-applications.png)

2. En la lista de aplicaciones, seleccione **G Suite**.

    ![Vínculo a G Suite en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**. Haga clic en **Introducción**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

      ![Hoja Introducción](./media/google-apps-provisioning-tutorial/get-started.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En **Credenciales de administrador**, haga clic en **Autorizar**. Se redirigirá al usuario a un cuadro de diálogo de autorización de Google en una nueva ventana del explorador.

      ![Autorización de G Suite](./media/google-apps-provisioning-tutorial/authorize-1.png)

6. Confirme que quiere conceder permisos de Azure AD para realizar cambios en el inquilino de G Suite. Seleccione **Aceptar**.

     ![Autorización de un inquilino de G Suite](./media/google-apps-provisioning-tutorial/gapps-auth.png)

7. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a G Suite. Si la conexión no se establece, asegúrese de que la cuenta de G Suite tiene permisos de administrador y pruebe de nuevo. Luego vuelva a intentar el paso **Autorizar**.

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Aprovisionar usuarios de Azure Active Directory**.

9. Revise los atributos de usuario que se sincronizan entre Azure AD y G Suite en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer coincidencias con las cuentas de usuario de G Suite con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de G Suite admite el filtrado de usuarios en función de ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |primaryEmail|String|
   |relations.[type eq "manager"].value|String|
   |name.familyName|String|
   |name.givenName|String|
   |suspended|String|
   |externalIds.[type eq "custom"].value|String|
   |externalIds.[type eq "organization"].value|String|
   |addresses.[type eq "work"].country|String|
   |addresses.[type eq "work"].streetAddress|String|
   |addresses.[type eq "work"].region|String|
   |addresses.[type eq "work"].locality|String|
   |addresses.[type eq "work"].postalCode|String|
   |emails.[type eq "work"].address|String|
   |organizations.[type eq "work"].department|String|
   |organizations.[type eq "work"].title|String|
   |phoneNumbers.[type eq "work"].value|String|
   |phoneNumbers.[type eq "mobile"].value|String|
   |phoneNumbers.[type eq "work_fax"].value|String|
   |emails.[type eq "work"].address|String|
   |organizations.[type eq "work"].department|String|
   |organizations.[type eq "work"].title|String|
   |phoneNumbers.[type eq "work"].value|String|
   |phoneNumbers.[type eq "mobile"].value|String|
   |phoneNumbers.[type eq "work_fax"].value|String|
   |addresses.[type eq "home"].country|String|
   |addresses.[type eq "home"].formatted|String|
   |addresses.[type eq "home"].locality|String|
   |addresses.[type eq "home"].postalCode|String|
   |addresses.[type eq "home"].region|String|
   |addresses.[type eq "home"].streetAddress|String|
   |addresses.[type eq "other"].country|String|
   |addresses.[type eq "other"].formatted|String|
   |addresses.[type eq "other"].locality|String|
   |addresses.[type eq "other"].postalCode|String|
   |addresses.[type eq "other"].region|String|
   |addresses.[type eq "other"].streetAddress|String|
   |addresses.[type eq "work"].formatted|String|
   |changePasswordAtNextLogin|String|
   |emails.[type eq "home"].address|String|
   |emails.[type eq "other"].address|String|
   |externalIds.[type eq "account"].value|String|
   |externalIds.[type eq "custom"].customType|String|
   |externalIds.[type eq "customer"].value|String|
   |externalIds.[type eq "login_id"].value|String|
   |externalIds.[type eq "network"].value|String|
   |gender.type|String|
   |GeneratedImmutableId|String|
   |Identificador|String|
   |ims.[type eq "home"].protocol|String|
   |ims.[type eq "other"].protocol|String|
   |ims.[type eq "work"].protocol|String|
   |includeInGlobalAddressList|String|
   |ipWhitelisted|String|
   |organizations.[type eq "school"].costCenter|String|
   |organizations.[type eq "school"].department|String|
   |organizations.[type eq "school"].domain|String|
   |organizations.[type eq "school"].fullTimeEquivalent|String|
   |organizations.[type eq "school"].location|String|
   |organizations.[type eq "school"].name|String|
   |organizations.[type eq "school"].symbol|String|
   |organizations.[type eq "school"].title|String|
   |organizations.[type eq "work"].costCenter|String|
   |organizations.[type eq "work"].domain|String|
   |organizations.[type eq "work"].fullTimeEquivalent|String|
   |organizations.[type eq "work"].location|String|
   |organizations.[type eq "work"].name|String|
   |organizations.[type eq "work"].symbol|String|
   |OrgUnitPath|String|
   |phoneNumbers.[type eq "home"].value|String|
   |phoneNumbers.[type eq "other"].value|String|
   |websites.[type eq "home"].value|String|
   |websites.[type eq "other"].value|String|
   |websites.[type eq "work"].value|String|
   

10. En la sección **Asignaciones**, seleccione **Aprovisionar grupos de Azure Active Directory**.

11. Revise los atributos de grupo que se sincronizan entre Azure AD y G Suite en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer coincidencias con los grupos de G Suite con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |email|String|
      |Members|String|
      |name|String|
      |description|String|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para G Suite, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en G Suite.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose.

> [!NOTE]
> Si los usuarios ya tienen una cuenta personal o de consumidor existente con la dirección de correo electrónico del usuario de Azure AD, es posible que se produzca algún problema que pueda resolverse mediante la herramienta Google Transfer antes de realizar la sincronización de directorios.

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
