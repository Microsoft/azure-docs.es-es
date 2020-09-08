---
title: Requisitos previos para la API de generación de informes de Azure Active Directory | Microsoft Docs
description: Conozca los requisitos previos para acceder a la API de generación de informes de Azure AD.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 03/04/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 22a8a0efe16b4ab2ea7b8a647284a3449741ac02
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226971"
---
# <a name="prerequisites-to-access-the-azure-active-directory-reporting-api"></a>Requisitos previos para acceder a la API de generación de informes de Azure Active Directory

Las [API de generación de informes de Azure Active Directory (Azure AD)](./concept-reporting-api.md) proporcionan acceso mediante programación a los datos a través de un conjunto de API de REST. A estas API se les puede llamar desde los lenguajes y herramientas de programación.

La API de generación de informes utiliza [OAuth](../../api-management/api-management-howto-protect-backend-with-aad.md) para autorizar el acceso a las API web.

Para preparar el acceso a la API de generación de informes, necesita lo siguiente:

1. [Asignar roles](#assign-roles)
2. [Requisitos de licencia](#license-requirements)
3. [Registro de una aplicación](#register-an-application)
4. [Concesión de permisos](#grant-permissions)
5. [Recopilación de configuraciones](#gather-configuration-settings)

## <a name="assign-roles"></a>Asignación de roles

Para acceder a los datos de informes a través de la API, debe tener asignado uno de los siguientes roles:

- Lector de seguridad

- Administrador de seguridad

- Administrador global

## <a name="license-requirements"></a>Requisitos de licencia

Un inquilino de Azure AD debe tener asociada una licencia de Azure AD Premium para acceder a los informes de inicio de sesión de inquilino. Se necesita una licencia Azure AD Premium P1 (o superior) para acceder a los informes de inicio de sesión de cualquier inquilino de Azure AD. Como alternativa, si el tipo de directorio es Azure AD B2C, se puede acceder a los informes de inicio de sesión a través de la API sin ningún requisito de licencia adicional. 


## <a name="register-an-application"></a>Registro de una aplicación

El registro es necesario aunque se acceda a la API de generación de informes mediante un script. El registro le proporciona un **identificador de aplicación**, que es necesario para las llamadas de autorización y permite que el código reciba tokens.

Si quiere configurar el directorio para que acceda a la API de generación de informes de Azure AD, debe iniciar sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador de Azure que también sea miembro del rol de directorio **Administrador global** en su inquilino de Azure AD.

> [!IMPORTANT]
> Las aplicaciones que se ejecutan con credenciales con privilegios de administrador pueden ser muy avanzadas, así que asegúrese de mantener seguras las credenciales del secreto o el identificador de la aplicación.
> 

**Registro de una aplicación de Azure AD**

1. En [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory** en el panel de navegación izquierdo.
   
    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. En la página **Azure Active Directory**, haga clic en **Registros de aplicaciones**.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/02.png) 

3. En la página **Registros de aplicaciones**, seleccione **Nuevo registro**.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/03.png)

4. La página **Registration an Application** (Registro de una aplicación):

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/04.png)

    a. En el cuadro de texto **Nombre**, escriba `Reporting API application`.

    b. Para la opción **Supported accounts type (Tipo de cuentas admitidas)** , seleccione **Accounts in this organizational only (Solo cuentas de esta organización)** .

    c. En **URL de redireccionamiento** seleccione el cuadro de texto **Web** y escriba `https://localhost`.

    d. Seleccione **Registrar**. 


## <a name="grant-permissions"></a>Concesión de permisos 

En función de la API a la que desea acceder, debe conceder los siguientes permisos a la aplicación:  

| API | Permiso |
| --- | --- |
| Microsoft Azure Active Directory | Leer datos de directorio |
| Microsoft Graph | Leer todos los datos del registro de auditoría |


![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/36.png)

En la siguiente sección se enumeran los pasos para ambas API. Si no desea acceder a una de las API, puede omitir los pasos relacionados.

**Para conceder a la aplicación permisos para usar las API:**


1. Seleccione **Permisos de API** y, después, **Agregar un permiso**. 

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/05.png)

2. En la **página Solicitud de permisos de API**, busque **APIs heredadas admitidas** **Azure Active Directory Graph**. 

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/06.png)

3. En la página  **Permisos necesarios**, seleccione **Permisos de la aplicación** y expanda la casilla **Directory.ReadAll** de **Directorio**.  Seleccione **Agregar permisos**.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/07.png)

4. En la página **Reporting API Application - API Permissions** (Informes de aplicación de API: Permisos de API ), seleccione **Conceder consentimiento del administrador** . 

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/08.png)

5. Nota: **Microsoft Graph** se agrega de forma predeterminada durante el registro de la API.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/15.png)

## <a name="gather-configuration-settings"></a>Recopilación de configuraciones 

En esta sección se muestra cómo obtener la siguiente configuración del directorio:

- Nombre de dominio
- Id. de cliente
- Secreto del cliente

Necesitará estos valores al configurar llamadas a la API de generación de informes. 

### <a name="get-your-domain-name"></a>Obtención del nombre de dominio

**Para obtener el nombre del dominio:**

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.
   
    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. En la página **Azure Active Directory**, seleccione **Nombres de dominio personalizados**.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/09.png) 

3. Copie el nombre del dominio en la lista de dominios.


### <a name="get-your-applications-client-id"></a>Obtención del id. de cliente de la aplicación

**Para obtener el id. de cliente de la aplicación:**

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory**.
   
    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. Seleccione la aplicación en la página **Registros de aplicaciones**.

3. En la página de la aplicación, vaya a **Id. de aplicación** y seleccione **Hacer clic para copiar**.

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/11.png) 


### <a name="get-your-applications-client-secret"></a>Obtención del secreto de cliente de la aplicación
 Evite errores al intentar obtener acceso a los registros de auditoría o al inicio de sesión mediante la API.

**Para obtener el secreto de cliente de la aplicación:**

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory**.
   
    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2.  Seleccione la aplicación en la página **Registros de aplicaciones**.

3.  Seleccione **Certificados y Secretos** en la página **API Application**, en la sección **Secretos de cliente**, haga clic en **+ Nuevo secreto de cliente**. 

    ![Registre la aplicación](./media/howto-configure-prerequisites-for-reporting-api/12.png)

5. En la página **Add a client secret** (Agregar un secreto de cliente), agregue:

    a. En el cuadro de texto **Descripción**, escriba `Reporting API`.

    b. En **Expiración**, seleccione **En 2 años**.

    c. Haga clic en **Save**(Guardar).

    d. Copie el valor de la clave.

## <a name="troubleshoot-errors-in-the-reporting-api"></a>Solución de errores de la API de generación de informes

En esta sección se enumeran los mensajes de error más comunes con los que puede encontrarse al acceder a los informes de actividad mediante Microsoft Graph API y los pasos para su resolución.

### <a name="error-failed-to-get-user-roles-from-microsoft-graph"></a>Error: No se pudieron obtener los roles de usuario de Microsoft Graph

 Inicie sesión en su cuenta con los dos botones de inicio de sesión de la interfaz de usuario del Probador de Graph para no recibir un error al intentar iniciar sesión con el Probador de Graph. 

![Explorador de Graph](./media/troubleshoot-graph-api/graph-explorer.png)

### <a name="error-failed-to-do-premium-license-check-from-microsoft-graph"></a>Error: No se pudo realizar la comprobación de licencia Premium de Microsoft Graph 

Si se encuentra con este mensaje de error al intentar acceder a los inicios de sesión mediante el Probador de Graph, seleccione **Modificar permisos** debajo de su cuenta en la barra de navegación de la izquierda y seleccione **Tasks.ReadWrite** y **Directory.Read.All**. 

![Modificación de la interfaz de usuario de permisos](./media/troubleshoot-graph-api/modify-permissions.png)

### <a name="error-tenant-is-not-b2c-or-tenant-doesnt-have-premium-license"></a>Error: El inquilino no es B2C o el inquilino no tiene licencia Premium

El acceso a los informes de inicio de sesión requiere una licencia Premium 1 (P1) de Azure Active Directory. Si ve este mensaje de error al acceder a los inicios de sesión, asegúrese de que su inquilino tiene una licencia P1 de Azure AD.

### <a name="error-the-allowed-roles-does-not-include-user"></a>Error: los roles permitidos no incluyen Usuario. 

 Evite errores al intentar obtener acceso a los registros de auditoría o al inicio de sesión mediante la API. Asegúrese de que la cuenta forma parte de los roles **Lector de seguridad** o **Lector de informes** en su inquilino de Azure Active Directory.

### <a name="error-application-missing-aad-read-directory-data-permission"></a>Error: Falta el permiso de AAD de la aplicación "Leer datos de directorio" 

### <a name="error-application-missing-microsoft-graph-api-read-all-audit-log-data-permission"></a>Error: Falta el permiso de Microsoft Graph API de la aplicación "Leer todos los datos de registro de auditoría"

Siga los pasos del artículo [Requisitos previos para acceder a la API de generación de informes de Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md) para asegurarse de que la aplicación se ejecuta con el conjunto correcto de permisos. 

## <a name="next-steps"></a>Pasos siguientes

* [Obtención de datos mediante la API de generación de informes de Azure Active Directory con certificados](tutorial-access-api-with-certificates.md)
* [Referencia de la API de auditoría](/graph/api/resources/directoryaudit?view=graph-rest-beta) 
* [Referencia de la API de generación de informes de actividad de inicio de sesión](/graph/api/resources/signin?view=graph-rest-beta)