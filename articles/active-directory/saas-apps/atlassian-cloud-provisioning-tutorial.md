---
title: 'Tutorial: Configuración de Atlassian Cloud para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente las cuentas de usuario para Atlassian Cloud.
services: active-directory
documentationcenter: ''
author: zhchia
writer: zhchia
manager: beatrizd-msft
ms.assetid: na
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/27/2019
ms.author: jeedes
ms.openlocfilehash: 12e94cfd576ced8c0e2768dd508e7f6f1a0056eb
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927477"
---
# <a name="tutorial-configure-atlassian-cloud-for-automatic-user-provisioning"></a>Tutorial: Configuración de Atlassian Cloud para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Atlassian Cloud y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y cancelar el aprovisionamiento de usuarios o grupos en [Atlassian Cloud](https://www.atlassian.com/licensing/cloud) automáticamente. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Atlassian Cloud
> * Eliminar usuarios de Atlassian Cloud cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Atlassian Cloud
> * Aprovisionar grupos y pertenencias a grupos en Atlassian Cloud
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/atlassian-cloud-tutorial) en Atlassian Cloud (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* [Un inquilino de Atlassian Cloud](https://www.atlassian.com/licensing/cloud)
* Una cuenta de usuario de Atlassian Cloud con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y Atlassian Cloud](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes).

## <a name="step-2-configure-atlassian-cloud-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Atlassian Cloud para admitir el aprovisionamiento con Azure AD

1. Navegue a [Atlassian Organization Manager](https://admin.atlassian.com) **> seleccione la organización > directorio**.

    ![Aprovisionamiento de Atlassian Cloud](./media/atlassian-cloud-provisioning-tutorial/select-directory.png)

2. Haga clic en **Aprovisionamiento de usuarios** y en **Crear un directorio**. Copie la **Directory base URL** (URL base de directorio) y **Token de portador** que se escribirán en los campos **URL de inquilino** y **Token secreto** en la pestaña Aprovisionamiento de la aplicación Atlassian Cloud en el portal de Azure AD, respectivamente.

    ![Aprovisionamiento de Atlassian Cloud](./media/atlassian-cloud-provisioning-tutorial/secret-token-1.png) ![Aprovisionamiento de Atlassian Cloud](./media/atlassian-cloud-provisioning-tutorial/secret-token-2.png) ![Aprovisionamiento de Atlassian Cloud](./media/atlassian-cloud-provisioning-tutorial/secret-token-3.png)


## <a name="step-3-add-atlassian-cloud-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Atlassian Cloud desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Atlassian Cloud, agregue Atlassian Cloud desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Atlassian Cloud para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Atlassian Cloud, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configuring-automatic-user-provisioning-to-atlassian-cloud"></a>Paso 5. Configurar el aprovisionamiento automático de usuarios en Atlassian Cloud 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Atlassian Cloud en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-atlassian-cloud-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Atlassian Cloud en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, a continuación, **Atlassian Cloud**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Atlassian Cloud**.

    ![Vínculo de Atlassian Cloud en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba la **URL de inquilino** y el **Token secreto** que obtuvo anteriormente de la cuenta de Atlassian Cloud. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Atlassian Cloud. Si la conexión no se establece, asegúrese de que la cuenta de Atlassian Cloud tiene permisos de administrador y pruébelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Atlassian Cloud** (Sincronizar usuarios de Azure Active Directory con Atlassian Cloud).

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Atlassian Cloud en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Atlassian Cloud con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |userName|String|
   |active|Boolean|
   |name.familyName|String|
   |name.givenName|String|
   |emails[type eq "work"].value|String|   

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Atlassian Cloud** (Sincronizar grupos de Azure Active Directory con Atlassian Cloud).

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Atlassian Cloud en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Atlassian Cloud para las operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |externalId|String|
      |members|Referencia|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Atlassian Cloud, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Atlassian Cloud.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

16. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose.

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="connector-limitations"></a>Limitaciones del conector

* Atlassian Cloud permite el aprovisionamiento de usuarios solo de [dominios comprobados](https://confluence.atlassian.com/cloud/organization-administration-938859734.html).
* Atlassian Cloud no admite cambios de nombre de grupo actualmente. Esto significa que los cambios realizados en la propiedad displayName de un grupo en Azure AD no se actualizará ni reflejará en Atlassian Cloud.
* El valor del atributo de usuario **mail** de Azure AD solo se rellena si el usuario tiene un buzón de Microsoft Exchange. Si el usuario no tiene ninguno, se recomienda asignar un atributo diferente al atributo **emails** en Atlassian Cloud.

## <a name="change-log"></a>Registro de cambios

* 15/06/2020: se ha agregado compatibilidad con la revisión por lotes para grupos.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-03.png