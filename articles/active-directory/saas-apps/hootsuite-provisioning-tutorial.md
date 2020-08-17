---
title: 'Tutorial: Configuración de Hootsuite para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y cancelar el aprovisionamiento de forma automática de las cuentas de usuario desde Azure AD en Hootsuite.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: fbbee24e-272f-4fa6-819c-10c548bf0215
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/15/2020
ms.author: Zhchia
ms.openlocfilehash: 2c31aabb57a9e1f4b29f535271691e17ea4f8559
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924111"
---
# <a name="tutorial-configure-hootsuite-for-automatic-user-provisioning"></a>Tutorial: Configuración de Hootsuite para el aprovisionamiento automático de usuarios

En este tutorial se describen los pasos que debe realizar en Hootsuite y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [Hootsuite](https://hootsuite.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md).

## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Hootsuite
> * Eliminación de usuarios de Hootsuite cuando ya no necesiten acceso
> * Mantenimiento de los atributos de usuario sincronizados entre Azure AD y Hootsuite
> * Aprovisionamiento de grupos y pertenencias a grupos en Hootsuite
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/hootsuite-tutorial) en Hootsuite (recomendado)

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una cuenta de usuario con [Hootsuite](http://www.hootsuite.com/) que tenga permisos de **administración de miembros** en la organización.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y Hootsuite](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-hootsuite-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Hootsuite para admitir el aprovisionamiento con Azure AD

Póngase en contacto con dev.support@hootsuite.com para obtener un token secreto de larga duración que será necesario en los pasos posteriores. 

## <a name="step-3-add-hootsuite-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Hootsuite desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Hootsuite, agregue Hootsuite desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Hootsuite para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Hootsuite, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-hootsuite"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Hootsuite 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-hootsuite-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Hootsuite en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](./media/hootsuite-provisioning-tutorial/enterprise-applications.png)

    ![Hoja Todas las aplicaciones](./media/hootsuite-provisioning-tutorial/all-applications.png)

2. En la lista de aplicaciones, seleccione **Hootsuite**.

    ![Vínculo a Hootsuite en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**. Haga clic en **Introducción**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

    ![Hoja Introducción](./media/hootsuite-provisioning-tutorial/get-started.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://platform.hootsuite.com/scim/v2` en la URL de inquilino. Escriba el valor del token secreto de larga duración recuperado antes en el **Paso 2**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Hootsuite. Si se produce un error en la conexión, asegúrese de que la cuenta de Hootsuite tiene permisos de administrador e inténtelo de nuevo.

    ![Aprovisionamiento](./media/hootsuite-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Aprovisionar usuarios de Azure Active Directory**.

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Hootsuite en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer coincidencias con las cuentas de usuario de Hootsuite a fin de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de Hootsuite admite el filtrado de usuarios en función de ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |userName|String|
   |emails[type eq "work"].value|String|
   |active|Boolean|
   |DisplayName|String|
   |preferredLanguage|String|
   |timezone|String|
   |urn:ietf:params:scim:schemas:extension:Hootsuite:2.0:User:organizationIds|String|
   |urn:ietf:params:scim:schemas:extension:Hootsuite:2.0:User:teamIds|String|

10. Para habilitar el servicio de aprovisionamiento de Azure AD para Hootsuite, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

11. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiera aprovisionar en Hootsuite.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

12. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

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
