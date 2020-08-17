---
title: 'Tutorial: Configuración de 15Five para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en 15Five.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: a276c004-9f71-4efc-8cca-1f615760249f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: dbed26bc6175b464b59fc48c733b15e6fecc7fc1
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924945"
---
# <a name="tutorial-configure-15five-for-automatic-user-provisioning"></a>Tutorial: Configuración de 15Five para aprovisionar usuarios automáticamente

El objetivo de este tutorial es mostrar los pasos que se realizan en 15Five y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en [15Five](https://www.15five.com/pricing/). Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory.

> [!NOTE]
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en 15Five
> * Eliminar usuarios de 15Five cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y 15Five
> * Aprovisionar grupos y pertenencias a grupos en 15Five
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/15five-tutorial) en 15Five (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* [Un inquilino de 15Five](https://www.15five.com/pricing/).
* Una cuenta de usuario de 15Five con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD e 15Five](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-15five-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de 15Five para admitir el aprovisionamiento con Azure AD

Antes de configurar 15Five para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en 15Five.

1. Inicie sesión en la [consola de administración de 15Five](https://my.15five.com/). Vaya a **Features > Integrations** (Características > Integraciones).

    ![Consola de administración de 15Five](media/15five-provisioning-tutorial/integration.png)

2.  Haga clic en **SCIM 2.0**.

    ![Consola de administración de 15Five](media/15five-provisioning-tutorial/image00.png)

3.  Vaya a **SCIM integration > Generate OAuth token** (Integración de SCIM > Generar token de OAuth).

    ![SCIM de incorporación de 15Five](media/15five-provisioning-tutorial/image02.png)

4.  Copie los valores de **URL base de SCIM 2.0** y **token de acceso**. Estos valores se escriben en el campo **URL de inquilino** y **Token secreto** de la pestaña Aprovisionamiento de la aplicación 15Five en Azure Portal.
    
    ![SCIM de incorporación de 15Five](media/15five-provisioning-tutorial/image03.png)

## <a name="step-3-add-15five-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de 15Five desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de 15Five, agregue 15Five desde la galería de aplicaciones de Azure AD. Si ha configurado previamente 15Five para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a 15Five, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).

## <a name="step-5-configure-automatic-user-provisioning-to-15five"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en 15Five 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en 15Five en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-15five-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para 15Five en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **15Five**.

    ![Vínculo a 15Five en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5.  En la sección Credenciales de administrador, escriba los valores de **URL base de SCIM 2.0 y token de acceso** que recuperó anteriormente en los campos **URL de inquilino** y **Token secreto** respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a 15Five. Si la conexión no se establece, asegúrese de que la cuenta de 15Five tiene permisos de administrador y pruebe otra vez.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to 15Five** (Sincronizar usuarios de Azure Active Directory con 15Five).

9. Examine los atributos de usuario que se sincronizan entre Azure AD y 15Five en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de 15Five con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.


   |Atributo|Tipo|
   |---|---|
   |active|Boolean|
   |title|String|
   |emails[type eq "work"].value|String|
   |userName|String|
   |name.givenName|String|
   |name.familyName|String|
   |externalId|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Referencia|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|String|
   |urn:ietf:params:scim:schemas:extension:15Five:2.0:User:location|String|
   |urn:ietf:params:scim:schemas:extension:15Five:2.0:User:startDate|String|

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to 15Five** (Sincronizar grupos de Azure Active Directory con 15Five).

11. Examine los atributos de grupo que se sincronizan entre Azure AD y 15Five en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades **Coincidentes** se usan para establecer correspondencia con las cuentas del usuario en 15Five a fin de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |externalId|String|
      |DisplayName|String|
      |members|Referencia|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para 15Five, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en 15Five.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

    Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose.

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  
    
## <a name="connector-limitations"></a>Limitaciones del conector

* 15Five no admite las eliminaciones temporales de los usuarios.

## <a name="change-log"></a>Registro de cambios

* 16/06/2020: se ha agregado compatibilidad con el atributo de extensión "Manager" y los atributos personalizados "Location" y "Start Date" para los usuarios.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md).
