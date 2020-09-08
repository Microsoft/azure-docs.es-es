---
title: 'Tutorial: Configuración de Salesforce para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Salesforce.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 08/01/2019
ms.author: jeedes
ms.openlocfilehash: 66bba0d4825300a6cead2453a67518af631234b9
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543511"
---
# <a name="tutorial-configure-salesforce-for-automatic-user-provisioning"></a>Tutorial: Configuración de Salesforce para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que debe realizar en Salesforce y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD para Salesforce.

## <a name="prerequisites"></a>Requisitos previos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure Active Directory
* Un inquilino de Salesforce.com

> [!IMPORTANT]
> Si utiliza una cuenta de prueba de Salesforce.com, no podrá configurar el aprovisionamiento automático de usuarios. Las cuentas de prueba no disponen del acceso necesario a la API habilitado hasta que se adquieren. Puede evitar esta limitación utilizando una [cuenta gratuita de desarrollador](https://developer.salesforce.com/signup) para completar este tutorial.

Si está utilizando un entorno de espacio aislado de Salesforce, consulte el [Tutorial de integración de espacio aislado de Salesforce](https://go.microsoft.com/fwLink/?LinkID=521879).

## <a name="assigning-users-to-salesforce"></a>Asignación de usuarios a Salesforce

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD.

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a la aplicación de Salesforce. Una vez decidido, puede asignar estos usuarios a la aplicación Salesforce siguiendo las instrucciones de: [Asignación de un usuario o un grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal).

### <a name="important-tips-for-assigning-users-to-salesforce"></a>Sugerencias importantes para asignar usuarios a Salesforce

* Se recomienda asignar un único usuario de Azure AD a Salesforce para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar a un usuario a Salesforce, debe seleccionar un rol de usuario válido. El rol "Acceso predeterminado" no funciona con el aprovisionamiento.

    > [!NOTE]
    > Esta aplicación importa los perfiles desde Salesforce como parte del proceso de aprovisionamiento, que el cliente puede seleccionar al asignar usuarios en Azure AD. Tenga en cuenta que los perfiles que se importarán desde Salesforce aparecen como roles en Azure AD.

## <a name="enable-automated-user-provisioning"></a>Habilitación del aprovisionamiento automático de usuarios

Esta sección le guía por los pasos necesarios para conectar la [API - v40 de aprovisionamiento de cuentas de usuario de Salesforce](https://developer.salesforce.com/docs/atlas.en-us.208.0.api.meta/api/implementation_considerations.htm), así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de Salesforce en función de la asignación de grupos y usuarios en Azure AD.

> [!Tip]
> También puede decidir habilitar el inicio de sesión único basado en SAML para Salesforce siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-account-provisioning"></a>Configurar el aprovisionamiento automático de cuentas de usuario

El objetivo de esta sección es describir cómo habilitar el aprovisionamiento de cuentas de usuario de Active Directory para Salesforce.

1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

2. Si ya ha configurado Salesforce para el inicio de sesión único, busque la instancia de Salesforce mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Salesforce** en la galería de aplicaciones. Seleccione Salesforce en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

3. Seleccione la instancia de Salesforce y, después, seleccione la pestaña **Aprovisionamiento**.

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento](./media/salesforce-provisioning-tutorial/provisioning.png)

5. En la sección **Credenciales de administrador**, proporcione los siguientes valores de configuración:

    a. Como **nombre de usuario del administrador**, escriba un nombre de cuenta de espacio de Salesforce que tenga asignado el perfil **Administrador del sistema** en Salesforce.com.

    b. En el cuadro de texto **Contraseña de administrador**, escriba la contraseña de esta cuenta.

6. Para obtener el token de seguridad de Salesforce, abra una nueva pestaña e inicie sesión en la misma cuenta de administrador de Salesforce. En la esquina superior derecha de la página, haga clic en su nombre y, a continuación, haga clic en **Configuración**.

    ![Habilitar el aprovisionamiento automático de usuarios](./media/salesforce-provisioning-tutorial/sf-my-settings.png "Habilitar el aprovisionamiento automático de usuarios")

7. En el panel de navegación izquierdo, haga clic en **Mi información personal** para expandir la sección relacionada y haga clic en **Reset My Security Token** (Restablecer mi token de seguridad).
  
    ![Habilitar el aprovisionamiento automático de usuarios](./media/salesforce-provisioning-tutorial/sf-personal-reset.png "Habilitar el aprovisionamiento automático de usuarios")

8. En la página **Reset Security Token** (Restablecer token de seguridad), haga clic en el botón **Reset Security Token** (Restablecer token de seguridad).

    ![Habilitar el aprovisionamiento automático de usuarios](./media/salesforce-provisioning-tutorial/sf-reset-token.png "Habilitar el aprovisionamiento automático de usuarios")

9. Compruebe la bandeja de entrada de correo electrónico asociada a esta cuenta de administrador. Busque un correo electrónico de Salesforce.com que contenga el nuevo token de seguridad.

10. Copie el token, vaya a la ventana de Azure AD y péguelo en el campo **Token secreto**.

11. Si la instancia de Salesforce está en la nube de administración pública de Salesforce, es obligatorio especificar un valor en **URL de inquilino**. De lo contrario, este valor es opcional. Escriba la dirección URL del inquilino con el formato "https://\<your-instance\>.my.salesforce.com" y sustituya \<your-instance\> por el nombre de la instancia de Salesforce.

12. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación Salesforce.

13. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla que aparece a continuación.

14. Haga clic en **Guardar**.  

15. En la sección Asignaciones, seleccione **Synchronize Azure Active Directory Users to** (Sincronizar usuarios de Azure Active Directory con Salesforce).

16. En la sección **Attribute Mappings** (Asignaciones de atributos), revise los atributos de usuario que se sincronizarán entre Azure AD y Salesforce. Tenga en cuenta que los atributos seleccionados como propiedades de **Coincidencia** se usarán para buscar coincidencias con las cuentas de usuario de Salesforce con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

17. Para habilitar el servicio de aprovisionamiento de Azure AD para Salesforce, cambie el **estado de aprovisionamiento** a **Activado** en la sección Configuración.

18. Haga clic en **Guardar**.

> [!NOTE]
> Una vez que los usuarios están aprovisionados en la aplicación Salesforce, el administrador debe configurar los valores específicos del idioma para ellos. Consulte [este](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5) artículo para obtener más detalles sobre la configuración del idioma.

Esta acción inicia la sincronización inicial de todos los usuarios y grupos asignados a Salesforce en la sección Usuarios y grupos. Tenga en cuenta que la sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos a los registros de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento en la aplicación de Salesforce.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="common-issues"></a>Problemas comunes
* Si tiene problemas para autorizar el acceso a Salesforce, asegúrese de lo siguiente:
    * Las credenciales usadas tienen acceso de administrador a Salesforce.
    * La versión de Salesforce que usa es compatible con el acceso web (por ejemplo, las ediciones Developer, Enterprise, Sandbox y Unlimited de Salesforce.)
    * El acceso a la API web está habilitado para el usuario.
* El servicio de aprovisionamiento de Azure AD admite el idioma de aprovisionamiento, la configuración regional y la zona horaria para un usuario. Estos atributos se encuentran en las asignaciones de atributos predeterminadas pero no tienen un atributo de origen predeterminado. Asegúrese de seleccionar el atributo de origen predeterminado y de que el atributo de origen esté en el formato esperado por SalesForce. Por ejemplo, localeSidKey para inglés (Estados Unidos) es en_US. Revise la guía proporcionada [ aquí ](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5) para determinar el formato apropiado de localeSidKey. Los formatos de languageLocaleKey se pueden encontrar [aquí](https://help.salesforce.com/articleView?id=faq_getstart_what_languages_does.htm&type=5). Además de garantizar que el formato sea correcto, es posible que deba asegurarse de que el idioma esté habilitado para sus usuarios como se describe [ aquí ](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5). 
* **SalesforceLicenseLimitExceeded:** No se pudo crear el usuario en la aplicación de destino porque no hay licencias disponibles para este usuario. Puede adquirir licencias adicionales para la aplicación de destino o revisar las asignaciones de usuario y la configuración de asignación de atributos para asegurarse de que se asignan los atributos correctos a los usuarios correctos.
* **SalesforceDuplicateUserName:** No se puede aprovisionar al usuario porque tiene un 'Nombre de usuario' de Salesforce.com que está duplicado en otro inquilino de Salesforce.com.  En Salesforce.com, los valores del atributo 'Nombre de usuario' deben ser únicos en todos los inquilinos de Salesforce.com.  De forma predeterminada, el valor de userPrincipalName de un usuario en Azure Active Directory se convierte en su 'Nombre de usuario' en Salesforce.com.   Tiene dos opciones.  Una opción es buscar y cambiar el nombre del usuario con el duplicado 'Nombre de usuario' en el otro inquilino de Salesforce.com, si también administra ese otro inquilino.  La otra opción es eliminar el acceso del usuario de Azure Active Directory al inquilino de Salesforce.com con el que está integrado su directorio. Se volverá a intentar esta operación en el siguiente intento de sincronización. 
* **SalesforceRequiredFieldMissing:** Salesforce requiere que ciertos atributos estén presentes en el usuario para crear o actualizar el usuario correctamente. A este usuario le falta uno de los atributos requeridos. Asegúrese de que atributos como correo electrónico y alias se llenen en todos los usuarios que quiera que se aprovisionen en Salesforce. Puede buscar usuarios que no tengan estos atributos mediante [filtros de alcance basados ​​en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 
* La asignación de atributos predeterminada para el aprovisionamiento en Salesforce incluye la expresión SingleAppRoleAssignments para asignar appRoleAssignments en Azure AD a ProfileName en Salesforce. Asegúrese de que los usuarios no tengan varias asignaciones de roles de aplicación en Azure AD, ya que la asignación de atributos solo admite el aprovisionamiento de un rol. 
* Salesforce requiere que las actualizaciones de correo electrónico se aprueben manualmente antes del cambio. Por tanto, es posible que vea varias entradas en los registros de aprovisionamiento para actualizar el correo electrónico del usuario (hasta que se haya aprobado el cambio de correo electrónico).


## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración del inicio de sesión único](https://docs.microsoft.com/azure/active-directory/active-directory-saas-salesforce-tutorial)
