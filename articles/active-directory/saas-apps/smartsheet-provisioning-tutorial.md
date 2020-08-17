---
title: 'Tutorial: Configuración de Smartsheet para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Smartsheet.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: na
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/07/2019
ms.author: jeedes
ms.openlocfilehash: c23c57e77f69cd7043e09ff5e2809d8329d62da3
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925530"
---
# <a name="tutorial-configure-smartsheet-for-automatic-user-provisioning"></a>Tutorial: Configuración de Smartsheet para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Smartsheet y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en [Smartsheet](https://www.smartsheet.com/pricing). Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Smartsheet
> * Quitar usuarios de Smartsheet cuando ya no necesitan acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Smartsheet
> * Inicio de sesión único en Smartsheet (recomendado)

> [!NOTE]
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* [Un inquilino de Smartsheet](https://www.smartsheet.com/pricing).
* Una cuenta de usuario en un plan de Smartsheet Enterprise o Premier Enterprise con permisos de administrador del sistema.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y Smartsheet](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-smartsheet-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Smartsheet para admitir el aprovisionamiento con Azure AD

Antes de configurar Smartsheet para el aprovisionamiento automático de usuarios con Azure AD, deberá habilitar el aprovisionamiento SCIM en Smartsheet.

1. Inicie sesión como **SysAdmin** en el **[portal de Smartsheet](https://app.smartsheet.com/b/home)** y vaya a **Administrador de cuenta**.

    ![Administrador de cuenta de Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-accountadmin.png)

2. Vaya a **Security Controls > User Auto Provisioning > Edit** (Controles de seguridad > Aprovisionamiento automático de usuarios > Editar).

    ![Controles de seguridad de Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-securitycontrols.png)

3. Agregue y valide los dominios de correo electrónico para los usuarios que planea aprovisionar desde Azure AD a Smartsheet. Elija **No habilitado** para asegurarse de que todas las acciones de aprovisionamiento solo se originen en Azure AD, y también para asegurarse de que su lista de usuarios de Smartsheet esté sincronizada con las asignaciones de Azure AD.

    ![Aprovisionamiento de usuarios de Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-userprovisioning.png)

4. Una vez que se complete la validación, deberá activar el dominio. 

    ![Dominio de activación de Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-activatedomain.png)

5. Genere el **token secreto** necesario para configurar el aprovisionamiento automático de usuarios con Azure AD; para ello, vaya a **Apps and Integrations** (Aplicaciones e integraciones).

    ![Instalación de Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet05.png)

6. Elija el **acceso a la API**. Haga clic en **Generate new access token** (Generar nuevo token de acceso).

    ![Instalación de Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet06.png)

7. Defina el nombre del token de acceso de la API. Haga clic en **OK**.

    ![Instalación de Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet07.png)

8. Copie el token de acceso de la API y guárdelo, ya que esta será la única vez que podrá verlo. Este se pedirá en el campo del **token secreto**  en Azure AD.

    ![Token de Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet08.png)

## <a name="step-3-add-smartsheet-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Smartsheet desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento en Smartsheet, agregue Smartsheet desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Smartsheet para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Smartsheet, debe seleccionar un rol que no sea **Default Access** (Acceso predeterminado). Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Para garantizar la paridad en las asignaciones de roles de usuario entre Smartsheet y Azure AD, se recomienda usar las mismas asignaciones de roles que se encuentran en la lista completa de usuarios de Smartsheet. Para recuperar esta lista de usuarios de Smartsheet, vaya a **Account Admin > User Management > More Actions > Download User List (csv)** (Administrador de cuenta > Administración de usuarios > Más acciones > Descargar lista de usuarios).

* Para acceder a ciertas características de la aplicación, Smartsheet requiere que un usuario tenga varios roles. Para obtener más información sobre los tipos de usuarios y permisos en Smartsheet, vaya a [User Types and Permissions](https://help.smartsheet.com/learning-track/shared-users/user-types-and-permissions) (Tipos de usuarios y permisos).

*  Si un usuario tiene varios roles asignados en Smartsheet, **DEBE** asegurarse de que estas asignaciones de roles se replican en Azure AD para evitar un escenario en el que los usuarios puedan perder el acceso a los objetos de Smartsheet de forma permanente. Cada rol único en Smartsheet **DEBE** asignarse a un grupo diferente en Azure AD. El usuario **DEBE** agregarse a cada uno de los grupos correspondientes a los roles deseados. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

## <a name="step-5-configure-automatic-user-provisioning-to-smartsheet"></a>Paso 5. Configurar el aprovisionamiento automático de usuarios para Smartsheet 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Smartsheet en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-smartsheet-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Smartsheet en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Smartsheet**.

    ![Vínculo a Smartsheet en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba los valores de **URL base de SCIM 2.0 y token de acceso** que recuperó anteriormente desde Smartsheet en **URL de inquilino** y **Token secreto** respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Smartsheet. Si la conexión no se establece, asegúrese de que la cuenta de Smartsheet tiene permisos de administrador del sistema e inténtelo de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Smartsheet** (Sincronizar usuarios de Azure Active Directory con Smartsheet).

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Smartsheet en la sección **Attribute Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con las cuentas de usuario de Smartsheet con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |active|Boolean|
   |title|String|
   |userName|String|
   |name.givenName|String|
   |name.familyName|String|
   |phoneNumbers[type eq "work"].value|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "fax"].value|String|
   |externalId|String|
   |roles[primary eq "True"].display|String|
   |roles[primary eq "True"].type|String|
   |roles[primary eq "True"].value|String|
   |roles|String|
   urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|String|


10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el servicio de aprovisionamiento de Azure AD para Smartsheet, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Smartsheet.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="connector-limitations"></a>Limitaciones del conector

* Smartsheet no admite las eliminaciones temporales. Cuando el atributo **activo** de un usuario se establece en "False", Smartsheet elimina al usuario de forma permanente.

## <a name="change-log"></a>Registro de cambios

* 16/06/2020: se ha agregado compatibilidad con los atributos de extensión de empresa "Centro de costo", "División", "Administrador" y "Departamento" para los usuarios.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
