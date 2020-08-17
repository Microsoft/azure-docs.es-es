---
title: 'Tutorial: Configuración de Samanage para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y cancelar el aprovisionamiento de forma automática de las cuentas de usuario de Azure AD para Samanage.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: 62d0392f-37d4-436e-9aff-22f4e5b83623
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/13/2020
ms.author: Zhchia
ms.openlocfilehash: 76b40ad0fd7403479ee7cec81e19bbb22213512d
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925182"
---
# <a name="tutorial-configure-samanage-for-automatic-user-provisioning"></a>Tutorial: Configuración de Samanage para el aprovisionamiento automático de usuarios
En este tutorial, se describen los pasos que debe realizar en Samanage y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [Samanage](https://www.samanage.com/pricing/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md).

## <a name="migrate-to-the-new-samange-application"></a>Migración a la nueva aplicación Samanage

Si tiene una integración existente con Samanage, consulte la sección siguiente sobre los próximos cambios. Si está configurando Samanage por primera vez, puede omitir esta sección y pasar a **Funcionalidades admitidas**.

#### <a name="whats-changing"></a>¿Qué está cambiando?
* Cambios en Azure AD: el método de autorización para aprovisionar usuarios en Samanage ha sido históricamente la **autorización básica**. Pronto verá que el método de autorización cambia a **token secreto de larga duración**.


#### <a name="what-do-i-need-to-do-to-migrate-my-existing-custom-integration-to-the-new-application"></a>¿Qué tengo que hacer para migrar mi integración personalizada existente a la nueva aplicación?
Si tiene una integración de Samanage existente con credenciales de administrador válidas, **no es necesario realizar ninguna acción**. La migración de los clientes se realizará automáticamente a la nueva aplicación. Este proceso se realiza por completo en segundo plano. Si las credenciales existentes expiran, o si necesita volver a autorizar el acceso a la aplicación, debe generar un token secreto de larga duración. Para generar un nuevo token, consulte el paso 2 de este artículo.


#### <a name="how-can-i-tell-if-my-application-has-been-migrated"></a>¿Cómo puedo saber si se ha migrado mi aplicación? 
Una vez migrada la aplicación, en la sección **Credenciales de administrador**, los campos **Nombre de usuario de administrador** y **Contraseña de administrador** se reemplazarán por un único campo **Token secreto**.

## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Samanage
> * Quitar usuarios de Samanage cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Samanage
> * Aprovisionar grupos y pertenencias a grupos en Samanage
> * [Inicio de sesión único](https://docs.microsoft.com/azure/active-directory/saas-apps/samanage-tutorial) en Samanage (recomendado)

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Un [inquilino de Samanage](https://www.samanage.com/pricing/) con el paquete profesional.
* Una cuenta de usuario de Samanage con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD y Samanage](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-samanage-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Samanage para admitir el aprovisionamiento con Azure AD

Para generar un token secreto para la autenticación, diríjase [aquí](https://help.samanage.com/s/article/Tutorial-Tokens-Authentication-for-API-Integration-1536721557657).

## <a name="step-3-add-samanage-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Samanage desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Samanage, agregue Samanage desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Samanage para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Samanage, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-samanage"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Samanage 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-samanage-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Samanage en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Samanage**.

    ![El vínculo a Samanage en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba `https://api.samanage.com` en la **URL de inquilino**.  Escriba el valor del token secreto recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Samanage. Si la conexión no se establece, asegúrese de que la cuenta de Samanage tiene permisos de administrador e inténtelo de nuevo.

    ![Aprovisionamiento](./media/samanage-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Samanage** (Sincronizar usuarios de Azure Active Directory con Samanage).

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Samanage en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Samanage con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de Samanage admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

      ![Asignaciones de usuario de Samanage](./media/samanage-provisioning-tutorial/user-attributes.png)

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Samanage** (Sincronizar grupos de Azure Active Directory con Samanage).

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Samanage en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Samanage con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      ![Asignaciones de grupo de Samanage](./media/samanage-provisioning-tutorial/group-attributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Samanage, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en Samanage.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

## <a name="connector-limitations"></a>Limitaciones del conector

Si selecciona la opción **Sincronizar todos los usuarios y grupos** y configura un valor para el atributo **roles** de Samanage, el valor del cuadro **Valor predeterminado si es nulo (opcional)** se debe expresar en el formato siguiente:

- {"displayName":"rol"}, donde el rol es el valor predeterminado que quiera.

## <a name="change-log"></a>Registro de cambios

* 22/04/2020: se actualizó el método de autorización de autenticación básica a token secreto de larga duración.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../manage-apps/configure-automatic-user-provisioning-portal.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)
