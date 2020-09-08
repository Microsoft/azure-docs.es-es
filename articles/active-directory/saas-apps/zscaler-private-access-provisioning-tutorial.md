---
title: 'Tutorial: configuración de Zscaler Private Access (ZPA) para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Zscaler Private Access (ZPA).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 10/07/2019
ms.author: Zhchia
ms.openlocfilehash: c605efe60e279ca3f5f14055e39bf95dc677542b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545848"
---
# <a name="tutorial-configure-zscaler-private-access-zpa-for-automatic-user-provisioning"></a>Tutorial: configuración de Zscaler Private Access (ZPA) para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en Zscaler Private Access (ZPA) y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios o grupos en Zscaler Private Access (ZPA).

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de Zscaler Private Access (ZPA)](https://www.zscaler.com/pricing-and-plans#contact-us).
* Una cuenta de usuario en Zscaler Private Access (ZPA) con permisos de administrador.

## <a name="assigning-users-to-zscaler-private-access-zpa"></a>Asignación de usuarios a Zscaler Private Access (ZPA)

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a Zscaler Private Access (ZPA). Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a Zscaler Private Access (ZPA):
* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-zscaler-private-access-zpa"></a>Sugerencias importantes para asignar usuarios a Zscaler Private Access (ZPA)

* Se recomienda asignar un único usuario de Azure AD a Zscaler Private Access (ZPA) para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a Zscaler Private Access (ZPA), debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="set-up-zscaler-private-access-zpa-for-provisioning"></a>Configuración de Zscaler Private Access (ZPA) para el aprovisionamiento

1. Inicie sesión en la [consola de administración de Zscaler Private Access (ZPA)](https://admin.private.zscaler.com/). Vaya a **Administración > Configuración de IdP**.

    ![Consola de administración de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/idpconfig.png)

2.  Asegúrese de que se ha configurado un IdP para el **inicio de sesión único**. Si no se configuró ningún IdP, agregue uno haciendo clic en el icono de signo más situado en la esquina superior derecha de la pantalla.

    ![Incorporación de SCIM en Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/plusicon.png)

3. Siga los pasos del **Asistente para agregar la configuración de IdP** para agregar un IdP. Deje el campo **Inicio de sesión único** establecido en **Usuario**. Proporcione un **Nombre** y seleccione el **Dominio** de la lista desplegable. Haga clic en **Siguiente** para ir a la siguiente ventana.

    ![Incorporación de IdP en Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/addidp.png)

4. Descargue el **certificado del proveedor de servicios**. Haga clic en **Siguiente** para ir a la siguiente ventana.

    ![Certificado de proveedor de servicio de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/spcertificate.png)

5. En la ventana siguiente, cargue el **certificado del proveedor de servicios** descargado previamente.

    ![Carga del certificado de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/uploadfile.png)

6.  Desplácese hacia abajo para proporcionar la **Dirección URL de inicio de sesión único** y el **Id. de entidad de IdP**.

    ![Id. de IdP de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/idpid.png)

7.  Desplácese hacia abajo para **Habilitar la sincronización de SCIM**. Haga clic en el botón **Generar nuevo token**. Copie el **Token de portador**. Este valor se escribirá en el campo Token secreto de la pestaña Aprovisionamiento en la aplicación Zscaler Private Access (ZPA) en Azure Portal.

    ![Creación de un token en Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/token.png)

8.  Para buscar la **URL de inquilino**, vaya a **Administración > Configuración de IdP**. Haga clic en el nombre de la configuración de IdP recién agregada que aparece en la página.

    ![Nombre de IdP de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/idpname.png)

9.  Desplácese hacia abajo para ver el **Punto de conexión del proveedor de servicios de SCIM** al final de la página. Copie el **Punto de conexión del proveedor de servicios de SCIM**. Este valor se escribirá en el campo Dirección URL del inquilino de la pestaña Aprovisionamiento en la aplicación Zscaler Private Access (ZPA) en Azure Portal.

    ![URL de SCIM en Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/tenanturl.png)


## <a name="add-zscaler-private-access-zpa-from-the-gallery"></a>Incorporación de Zscaler Private Access (ZPA) desde la galería

Antes de configurar Zscaler Private Access (ZPA) para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar Zscaler Private Access (ZPA) desde la galería de aplicaciones de Azure AD hasta la lista de aplicaciones SaaS administradas.

**Para agregar Zscaler Private Access (ZPA) desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler Private Access (ZPA)** , seleccione **Zscaler Private Access (ZPA)** en el panel de resultados y después haga clic en el botón **Agregar** para agregar la aplicación.

    ![Zscaler Private Access (ZPA) en la lista de resultados](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-zscaler-private-access-zpa"></a>Configuración del aprovisionamiento automático de usuarios para Zscaler Private Access (ZPA) 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de AD Azure para crear, actualizar y deshabilitar usuarios o grupos en Zscaler Private Access (ZPA) en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para Zscaler Private Access (ZPA) siguiendo las instrucciones del [tutorial de inicio de sesión único de Zscaler Private Access (ZPA)](https://docs.microsoft.com/azure/active-directory/saas-apps/zscalerprivateaccess-tutorial). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

> [!NOTE]
> Para obtener más información sobre el punto de conexión de SCIM de Zscaler Private Access, consulte [este documento](https://www.zscaler.com/partners/microsoft).

### <a name="to-configure-automatic-user-provisioning-for-zscaler-private-access-zpa-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Zscaler Private Access (ZPA) en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zscaler Private Access (ZPA)** .

    ![Vínculo a Zscaler Private Access (ZPA) en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba el valor del **Punto de conexión de proveedor de servicios de SCIM** recuperado anteriormente en **Dirección URL del inquilino**. Escriba el valor del **token de portador** recuperado anteriormente en **Token secreto**. Haga clic en **Prueba de conexión** para asegurarse de que Azure AD puede conectarse a Zscaler Private Access (ZPA). Si la conexión no se establece, asegúrese de que la cuenta de Zscaler Private Access (ZPA) tiene permisos de administrador y pruebe de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con Zscaler Private Access (ZPA)** .

    ![Asignaciones de usuario de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/usermappings.png)

9. Examine los atributos de usuario que se sincronizan entre Azure AD y Zscaler Private Access (ZPA) en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Zscaler Private Access (ZPA) con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de usuario de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/userattributes.png)

10. En la sección **Asignaciones**, seleccione **Sincronizar grupos de Azure Active Directory con Zscaler Private Access (ZPA)** .

    ![Asignaciones de grupo de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/groupmappings.png)

11. Examine los atributos de grupo que se sincronizan entre Azure AD y Zscaler Private Access (ZPA) en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Zscaler Private Access (ZPA) con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Atributos de grupo de Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/groupattributes.png)

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Zscaler Private Access (ZPA), cambie el valor de **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Zscaler Private Access (ZPA).

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en Zscaler Private Access (ZPA).

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

