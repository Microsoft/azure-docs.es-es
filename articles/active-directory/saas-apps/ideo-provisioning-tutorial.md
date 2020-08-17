---
title: 'Tutorial: Configuración de IDEO para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en IDEO.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: de4f06a3-83e9-46ce-80ee-03d706b91c81
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/24/2019
ms.author: Zhchia
ms.openlocfilehash: a778675bade3b381907a823e74f07ab4e7f96581
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926882"
---
# <a name="tutorial-configure-ideo-for-automatic-user-provisioning"></a>Tutorial: Configuración de IDEO para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en IDEO y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y desaprovisionar automáticamente usuarios o grupos en IDEO. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

> [!NOTE]
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en IDEO
> * Quitar usuarios de IDEO cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD e IDEO
> * Aprovisionar grupos y pertenencias a grupos en IDEO
> * Inicio de sesión único en IDEO (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global).
* [Un inquilino de IDEO](https://www.shape.space/product/pricing)
* Una cuenta de usuario en IDEO | Shape con permisos de administrador.


## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos quiere [asignar entre Azure AD e IDEO](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-ideo-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de IDEO para admitir el aprovisionamiento con Azure AD

Antes de configurar IDEO para el aprovisionamiento automático de usuarios con Azure AD, debe recuperar alguna información de aprovisionamiento de IDEO.

* Para el **Token secreto**, ponte en contacto con el equipo de soporte técnico de IDEO en productsupport@ideo.com. Este valor se escribirá en el campo **Token secreto** de la pestaña Aprovisionamiento de la aplicación IDEO en Azure Portal. 

## <a name="step-3-add-ideo-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de IDEO desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de IDEO, agregue IDEO desde la galería de aplicaciones de Azure AD. Si ha configurado previamente IDEO para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a para empezar a administrar el aprovisionamiento en IDEO, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-ideo"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en IDEO 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en IDEO en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-ideo-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para IDEO en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **IDEO**.

    ![Vínculo a IDEO en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba los valores de **URL base de SCIM 2.0 y token de acceso**, que recuperó anteriormente del equipo de soporte técnico de IDEO, en los campos **URL de inquilino** y **Token secreto**, respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a IDEO. Si la conexión no se establece, asegúrese de que la cuenta de IDEO tenga permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to IDEO** (Sincronizar usuarios de Azure Active Directory con IDEO).

9. Examine los atributos de grupo que se sincronizan entre Azure AD e IDEO en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de IDEO con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |userName|String|
   |emails[type eq "work"].value|String|
   |active|Boolean|
   |name.givenName|String|
   |name.familyName|String|

10. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con IDEO**.
   
11. Examine los atributos de grupo que se sincronizan entre Azure AD e IDEO en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer correspondencia con los grupos de IDEO a fin de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|
      |---|---|
      |DisplayName|String|
      |members|Referencia|

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para IDEO, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en IDEO.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="change-log"></a>Registro de cambios

* 06/15/2020: Se agregó compatibilidad para usar las operaciones PATCH para los grupos en lugar de PUT.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)


