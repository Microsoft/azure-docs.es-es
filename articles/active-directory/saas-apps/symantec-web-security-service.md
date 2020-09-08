---
title: 'Tutorial: Configuración de Symantec Web Security Service (WSS) para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Symantec Web Security Service (WSS).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/23/2019
ms.author: Zhchia
ms.openlocfilehash: 083bb7549bb555cbbefaf680d401696ecd64b9c9
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544917"
---
# <a name="tutorial-configure-symantec-web-security-service-wss-for-automatic-user-provisioning"></a>Tutorial: Configuración de Symantec Web Security Service (WSS) para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Symantec Web Security Service (WSS) y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en Symantec Web Security Service (WSS).

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Symantec Web Security Service (WSS)](https://www.websecurity.symantec.com/buy-renew?inid=brmenu_nav_brhome)
* Una cuenta de usuario en Symantec Web Security Service (WSS) con permisos de administrador.

## <a name="assigning-users-to-symantec-web-security-service-wss"></a>Asignación de usuarios a Symantec Web Security Service (WSS)

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Symantec Web Security Service (WSS). Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Symantec Web Security Service (WSS):
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

##  <a name="important-tips-for-assigning-users-to-symantec-web-security-service-wss"></a>Sugerencias importantes para asignar usuarios a Symantec Web Security Service (WSS)

* Se recomienda asignar un único usuario de Azure AD a Symantec Web Security Service (WSS) para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Symantec Web Security Service (WSS), debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="setup-symantec-web-security-service-wss-for-provisioning"></a>Configuración de Symantec Web Security Service (WSS) para el aprovisionamiento

Antes de configurar Symantec Web Security Service (WSS) para el aprovisionamiento automático de usuarios con Azure AD, tendrá que habilitar el aprovisionamiento de SCIM en Symantec Web Security Service (WSS).

1. Inicie sesión en la [consola de administración de Symantec Web Security Service](https://portal.threatpulse.com/login.jsp). Vaya a **Solutions** > **Service** (Soluciones > Servicio).

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/service.png)

2. Vaya a **Account Maintenance** > **Integrations** > **New Integration** (Mantenimiento de cuenta > Integraciones > Nueva integración).

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/acount.png)

3.  Seleccione **Third-Party Users & Groups Sync** (Sincronización de usuarios y grupos de terceros). 

    ![Symantec Web Security Service](media/symantec-web-security-service/third-party-users.png)

4.  Copie los valores de **SCIM URL** (URL de SCIM) y **Token**. Estos valores son los que se especificarán en los campos **URL de inquilino** y **Token secreto** de la pestaña Aprovisionamiento de la aplicación Symantec Web Security Service (WSS) en Azure Portal.

    ![Symantec Web Security Service](media/symantec-web-security-service/scim.png)

## <a name="add-symantec-web-security-service-wss-from-the-gallery"></a>Adición de Symantec Web Security Service (WSS) desde la galería

Para configurar Symantec Web Security Service (WSS) para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Symantec Web Security Service (WSS) desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar Symantec Web Security Service (WSS) desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Symantec Web Security Service**, seleccione **Symantec Web Security Service** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Symantec Web Security Service (WSS) en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-symantec-web-security-service-wss"></a>Configuración del aprovisionamiento automático de usuarios en Symantec Web Security Service (WSS)

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Symantec Web Security Service (WSS) en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede elegir habilitar el inicio de sesión único basado en SAML para Symantec Web Security Service (WSS), para lo que debe seguir las instrucciones que se proporcionan en el [tutorial de inicio de sesión único de Symantec Web Security Service (WSS)](symantec-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-symantec-web-security-service-wss-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Symantec Web Security Service (WSS) en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Symantec Web Security Service**.

    ![Vínculo a Symantec Web Security Service (WSS) en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección Credenciales de administrador, escriba los valores de **URL de SCIM** y **Token** que recuperó anteriormente en **URL de inquilino** y **Token secreto** respectivamente. Haga clic en **Probar conexión**  para asegurarse de que Azure AD puede conectar con Symantec Web Security Service. Si la conexión no se establece, asegúrese de que la cuenta de Symantec Web Security Service (WSS) tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Symantec Web Security Service (WSS) [Sincronizar usuarios de Azure Active Directory con Symantec Web Security Service (WSS)]** .

    ![Asignaciones de usuario de Symantec Web Security Service (WSS)](media/symantec-web-security-service/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Symantec Web Security Service (WSS) en la sección **Attribute Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con las cuentas de usuario de Symantec Web Security Service (WSS) con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Asignaciones de usuario de Symantec Web Security Service (WSS)](media/symantec-web-security-service/userattribute.png)

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Symantec Web Security Service** (Sincronizar grupos de Azure Active Directory con Symantec Web Security Service).

    ![Asignaciones de usuario de Symantec Web Security Service (WSS)](media/symantec-web-security-service/groupmapping.png)

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Symantec Web Security Service (WSS) en la sección **Attribute Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con los grupos de Symantec Web Security Service (WSS) con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Asignaciones de usuario de Symantec Web Security Service (WSS)](media/symantec-web-security-service/groupattribute.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para Symantec Web Security Service, cambie el valor de **Estado de aprovisionamiento** a **Activado** en la **sección** Configuración.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Symantec Web Security Service (WSS).

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Para más información sobre el tiempo que se tarda en aprovisionar los usuarios o los grupos, consulte [¿Cuánto tiempo se tarda en aprovisionar usuarios?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users)

Puede usar la sección **Estado actual** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Symantec Web Security Service (WSS). Para obtener más información, vea [Comprobación del estado de aprovisionamiento](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Para leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
