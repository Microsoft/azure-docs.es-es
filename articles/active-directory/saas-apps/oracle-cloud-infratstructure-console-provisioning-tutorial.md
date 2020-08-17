---
title: 'Tutorial: Configuración de Oracle Cloud Infrastructure Console para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y cancelar el aprovisionamiento de forma automática de las cuentas de usuario de Azure AD en Oracle Cloud Infrastructure Console.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: e22c8746-7638-4a0f-ae08-7db0c477cf52
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/16/2020
ms.author: Zhchia
ms.openlocfilehash: 94f5be93d95cc1c524e6db464cac68879ed0b0e7
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926797"
---
# <a name="tutorial-configure-oracle-cloud-infrastructure-console-for-automatic-user-provisioning"></a>Tutorial: Configuración de Oracle Cloud Infrastructure Console para el aprovisionamiento automático de usuarios

En este tutorial, se describen los pasos que debe realizar en Oracle Cloud Infrastructure Console y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y anula el aprovisionamiento automáticamente de usuarios y grupos en [Oracle Cloud Infrastructure Console](https://www.oracle.com/cloud/free/?source=:ow:o:p:nav:0916BCButton&intcmp=:ow:o:p:nav:0916BCButton) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Oracle Cloud Infrastructure Console
> * Eliminación de usuarios en Oracle Cloud Infrastructure Console cuando ya no requieren acceso
> * Mantenimiento de la sincronización de los atributos de usuario entre Azure AD y Oracle Cloud Infrastructure Console
> * Aprovisionamiento de grupos y pertenencia a grupos en Oracle Cloud Infrastructure Console
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/oracle-cloud-tutorial) en Oracle Cloud Infrastructure Console (recomendado)

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Un [inquilino](https://www.oracle.com/cloud/sign-in.html?intcmp=OcomFreeTier&source=:ow:o:p:nav:0916BCButton) de Oracle Cloud Infrastructure Control.
* Una cuenta de usuario en Oracle Cloud Infrastructure Control con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos desea [asignar entre Azure AD y Oracle Cloud Infrastructure Console](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-oracle-cloud-infrastructure-console-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Oracle Cloud Infrastructure Console para admitir el aprovisionamiento con Azure AD

1. Inicie sesión en el portal de administración de Oracle Cloud Infrastructure Console. En la esquina superior izquierda de la pantalla, vaya a **Identidad > Federación**.

    ![Oracle Admin](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/identity.png)

2. Haga clic en la dirección URL que se muestra en la página junto a Oracle Identity Cloud Service Console.

    ![Dirección URL de Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/url.png)

3. Haga clic en **Agregar proveedor de identidades** para crear un nuevo proveedor de identidades. Guarde el identificador de IdP que se usará como parte de la dirección URL del inquilino. Haga clic en el icono más situado junto a la pestaña **Aplicaciones** para crear un AppRole de administrador de dominio de identidad IDCS de concesión y cliente OAuth.

    ![Icono de Oracle Cloud](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/add.png)

4. Siga las capturas de pantalla que aparecen a continuación para configurar la aplicación. Una vez finalizada la configuración, haga clic en **Guardar**.

    ![Configuración de Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/configuration.png)

    ![Directiva de emisión de tokens de Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/token-issuance.png)

5. En la pestaña Configuraciones de la aplicación, expanda la opción **Información general** para recuperar el identificador de cliente y el secreto de cliente.

    ![Generación de token de Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/general-information.png)

6. Para generar un token secreto Base64, codifique el identificador de cliente y el secreto de cliente en el formato **client ID:Client Secret**. Guarde el token secreto. Este valor se escribirá en el campo **Token secreto** de la pestaña Aprovisionamiento de la aplicación Oracle Cloud Infrastructure Console en Azure Portal.

## <a name="step-3-add-oracle-cloud-infrastructure-console-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Oracle Cloud Infrastructure Console desde la galería de aplicaciones de Azure AD

Agregue Oracle Cloud Infrastructure Console desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en Oracle Cloud Infrastructure Console. Si ha configurado previamente Oracle Cloud Infrastructure Console para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Oracle Cloud Infrastructure Console, debe seleccionar un rol que no sea de **acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-oracle-cloud-infrastructure-console"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Oracle Cloud Infrastructure Console 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-oracle-cloud-infrastructure-console-in-azure-ad"></a>A fin de configurar el aprovisionamiento automático de usuarios en Oracle Cloud Infrastructure Console en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Oracle Cloud Infrastructure Console**.

    ![El vínculo de Oracle Cloud Infrastructure Console de la lista Aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba la **URL de inquilino** con el formato `https://<IdP ID>.identity.oraclecloud.com/admin/v1`. Por ejemplo, `https://idcs-0bfd023ff2xx4a98a760fa2c31k92b1d.identity.oraclecloud.com/admin/v1`. Escriba el valor del token secreto recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para confirmar que Azure AD puede conectarse a Oracle Cloud Infrastructure Console. Si la conexión no se establece, asegúrese de que la cuenta de Oracle Cloud Infrastructure Console tiene permisos de administrador e inténtelo de nuevo.

    ![Aprovisionamiento](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Oracle Cloud Infrastructure Console**.

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Oracle Cloud Infrastructure Console en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con las cuentas de usuario de Oracle Cloud Infrastructure Console con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de Oracle Cloud Infrastructure Console admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |userName|String|
      |active|Boolean|
      |title|String|
      |emails[type eq "work"].value|String|
      |preferredLanguage|String|
      |name.givenName|String|
      |name.familyName|String|
      |addresses[type eq "work"].formatted|String|
      |addresses[type eq "work"].locality|String|
      |addresses[type eq "work"].region|String|
      |addresses[type eq "work"].postalCode|String|
      |addresses[type eq "work"].country|String|
      |addresses[type eq "work"].streetAddress|String|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|String|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Referencia|
      |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|String|
      |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:bypassNotification|Boolean|
      |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:isFederatedUser|Boolean|

10. En la sección **Asignaciones**, seleccione **Sincronización de grupos de Azure Active Directory con Oracle Cloud Infrastructure Console**.

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Oracle Cloud Infrastructure Console en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con los grupos de Oracle Cloud Infrastructure Console con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |externalId|String|
      |members|Referencia|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Oracle Cloud Infrastructure Console, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiera que se aprovisionen en Oracle Cloud Infrastructure Console.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

* Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
* Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
* Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)
