---
title: 'Tutorial: Configuración de Zendesk para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y desaprovisionar automáticamente cuentas de usuario en Zendesk.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: 5c038cc309d8f1f13a04de01ed83e04afa70ba6d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546176"
---
# <a name="tutorial-configure-zendesk-for-automatic-user-provisioning"></a>Tutorial: Configuración de Zendesk para el aprovisionamiento automático de usuarios

Este tutorial muestra los pasos que se deben realizar en Zendesk y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios y grupos en Zendesk.

> [!NOTE]
> En este tutorial se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario que se describe en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure AD.
* Un inquilino de Zendesk con el plan Professional o uno superior habilitado.
* Una cuenta de usuario de Zendesk con permisos de administrador.

## <a name="add-zendesk-from-the-azure-marketplace"></a>Adición de Zendesk desde Azure Marketplace

Antes de configurar Zendesk para el aprovisionamiento automático de usuarios con Azure AD, agregue Zendesk desde Azure Marketplace en su lista de aplicaciones SaaS administradas.

Para eliminar Zendesk de Marketplace, siga estos pasos:

1. En el panel de navegación de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Icono de Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zendesk** y seleccione **Zendesk** del panel de resultados. Para agregar la aplicación, seleccione **Agregar**.

    ![Zendesk en la lista de resultados](common/search-new-app.png)

## <a name="assign-users-to-zendesk"></a>Asignación de usuarios a Zendesk

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios o grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Zendesk. Para asignar estos usuarios o grupos a Zendesk, siga las instrucciones de [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-zendesk"></a>Sugerencias importantes para asignar usuarios a Zendesk

* Actualmente, los roles de Zendesk se rellenan automática y dinámicamente en la interfaz de usuario de Azure Portal. Antes de asignar roles de Zendesk a los usuarios, asegúrese de que se ha completado una sincronización inicial con Zendesk para recuperar los roles más recientes en el inquilino de Zendesk.

* Se recomienda asignar un único usuario de Azure AD a Zendesk para probar la configuración inicial de aprovisionamiento automático de usuarios. Puede asignar usuarios o grupos adicionales más adelante tras la correcta realización de las pruebas.

* Al asignar un usuario a Zendesk, seleccione un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configure-automatic-user-provisioning-to-zendesk"></a>Configuración del aprovisionamiento automático de usuarios en Zendesk 

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de Azure AD. Úsela para crear, actualizar y deshabilitar usuarios o grupos en Zendesk en función de las asignaciones de usuarios o grupos en Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Zendesk. Siga las instrucciones del [Tutorial del inicio de sesión único de Zendesk](zendesk-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-provisioning-for-zendesk-in-azure-ad"></a>Configuración del aprovisionamiento automático de usuarios para Zendesk en Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Zendesk**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zendesk**.

    ![Vínculo de Zendesk en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk16.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Modo de aprovisionamiento de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk1.png)

5. En la sección **Credenciales del administrador**, escriba el nombre del usuario administrador, el token secreto y el dominio de la cuenta de Zendesk. Algunos ejemplos de estos valores son:

   * En el cuadro **Nombre de usuario administrador**, rellene el nombre de usuario de la cuenta de administrador del inquilino de Zendesk. Un ejemplo es admin@contoso.com.

   * En el cuadro **Token secreto**, rellene el token secreto tal y como se describe en el paso 6.

   * En el cuadro **Dominio**, rellene el subdominio del inquilino de Zendesk. Por ejemplo, para una cuenta con una dirección URL de inquilino con el valor `https://my-tenant.zendesk.com`, el subdominio es **my-tenant**.

6. El token secreto para su cuenta de Zendesk está ubicado en **Administración** > **API** > **Configuración**. Asegúrese de que **Token de acceso** está establecido en **Habilitado**.

    ![Configuración de administrador de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk4.png)

    ![Token secreto de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk2.png)

7. Después de rellenar los cuadros que se muestran en el paso 5, seleccione **Probar conexión** para asegurarse de que Azure AD puede conectarse a Zendesk. Si la conexión no se establece, asegúrese de que la cuenta de Zendesk tiene permisos de administrador y pruébelo de nuevo.

    ![Conexión de prueba de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk19.png)

8. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de la persona o el grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk9.png)

9. Seleccione **Guardar**.

10. En la sección **Mappings** (Asignaciones), seleccione **Synchronize Azure Active Directory Users to Zendesk** (Sincronizar usuarios de Azure Active Directory con Zendesk).

    ![Sincronización del usuario de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk10.png)

11. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y ZenDesk. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con las cuentas de usuario de Zendesk para las operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de usuario coincidentes de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk11.png)

12. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Zendesk** (Sincronizar grupos de Azure Active Directory con Zendesk).

    ![Sincronización del grupo de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk12.png)

13. Revise los atributos de grupo que se sincronizan entre Azure AD y Zendesk en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Matching** (Coincidencia) se usan para buscar coincidencias con los grupos de Zendesk para las operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de grupo coincidentes de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk13.png)

14. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para Zendesk, en la sección **Configuración**, cambie **Estado de aprovisionamiento** a **Activado**.

    ![Estado de aprovisionamiento de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk14.png)

16. Defina los usuarios o grupos que quiere que se aprovisionen en Zendesk. En la sección **Configuración**, seleccione los valores que quiere incluir en el **Ámbito**.

    ![Ámbito de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk15.png)

17. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Operación Guardar en Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk18.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Se produce aproximadamente cada 40 minutos mientras se ejecuta el servicio de aprovisionamiento de Azure AD. 

Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos al informe de actividad de aprovisionamiento. En el informe se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Zendesk.

Para obtener información acerca de cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Zendesk admite el uso de grupos para usuarios con roles **Agente** únicamente. Para más información, consulte la [documentación de Zendesk](https://support.zendesk.com/hc/en-us/articles/203661966-Creating-managing-and-using-groups).

* Cuando un rol personalizado se asigna a un usuario o grupo, el servicio automático de aprovisionamiento de usuarios de Azure AD también asigna el rol predeterminado **Agente**. Solo se puede asignar un sol personalizado a los agentes. Para más información, consulte la [documentación de Zendesk API](https://developer.zendesk.com/rest_api/docs/support/users#json-format-for-agent-or-admin-requests). 

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zendesk-tutorial/tutorial_general_01.png
[2]: ./media/zendesk-tutorial/tutorial_general_02.png
[3]: ./media/zendesk-tutorial/tutorial_general_03.png
