---
title: Incorporación del inicio de sesión con Microsoft a una aplicación web en Python de la Plataforma de identidad de Microsoft | Azure
description: Aprenda a implementar el inicio de sesión de Microsoft en una aplicación web de Python mediante OAuth2
services: active-directory
author: abhidnya13
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/25/2019
ms.author: abpati
ms.custom: aaddev, devx-track-python, scenarios:getting-started, languages:Python
ms.openlocfilehash: 6b58e927952b2a51289c3017455cc7d66545fe86
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120327"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-a-python-web-app"></a>Inicio rápido: Adición del inicio de sesión con Microsoft a una aplicación web de Python

En este inicio rápido, aprenderá a integrar una aplicación web de Python con la plataforma de identidad de Microsoft. Su aplicación iniciará la sesión de un usuario, obtendrá un token de acceso para llamar a Microsoft Graph API y realizará una solicitud a Microsoft Graph API.

Cuando haya completado la guía, la aplicación podrá aceptar inicios de sesión de cuentas Microsoft personales (como outlook.com, live.com y otras), y cuentas profesionales y educativas de cualquier empresa u organización que utilice Azure Active Directory. (Para ilustrar este tema, consulte el apartado en el que se explica el [funcionamiento del ejemplo](#how-the-sample-works).)

## <a name="prerequisites"></a>Prerrequisitos

Para ejecutar este ejemplo, necesitará lo siguiente:

- [Python 2.7+](https://www.python.org/downloads/release/python-2713) o [Python 3+](https://www.python.org/downloads/release/python-364/)
- [Flask](http://flask.pocoo.org/), [Flask-Session](https://pypi.org/project/Flask-Session/), [solicitudes](https://requests.kennethreitz.org/en/master/)
- [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python)

> [!div renderon="docs"]
>
> ## <a name="register-and-download-your-quickstart-app"></a>Registro y descarga de la aplicación de inicio rápido
>
> Tiene dos opciones para iniciar la aplicación de inicio rápido: rápido (opción 1) y manual (opción 2)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Opción 1: registrar y configurar de modo automático la aplicación y, a continuación, descargar el código de ejemplo
>
> 1. Vaya a [Azure Portal: registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/PythonQuickstartPage/sourceType/docs).
> 1. Escriba un nombre para la aplicación y seleccione **Registrar**.
> 1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Opción 2: registrar y configurar manualmente la aplicación y el código de ejemplo
>
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
>
> Para registrar la aplicación y agregar la información de registro de la aplicación a la solución de forma manual, siga estos pasos:
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
> 1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
> 1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
> 1. Seleccione **Nuevo registro**.
> 1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
>      - En la sección **Nombre**, escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación, por ejemplo, `python-webapp`.
>      - En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
>      - Seleccione **Registrar**.
>      - En la página de **información general** de la aplicación, anote el valor del **Identificador de aplicación (cliente)** para su uso posterior.
> 1. Seleccione la **Autenticación** en el menú y, después, agregue la siguiente información:
>    - Agregue la configuración de la plataforma **web**. Agregue `http://localhost:5000/getAToken` como **URI de redirección**.
>    - Seleccione **Guardar**.
> 1. En el menú de la izquierda, elija **Certificates & secrets** (Certificados y secretos) y haga clic en **Nuevo secreto de cliente** en la sección **Client Secrets** (Secretos de cliente):
>
>      - Escriba una descripción de la clave (del secreto de la aplicación de instancia).
>      - Seleccione una duración de la clave de **En 1 año**.
>      - Al hacer clic en **Agregar**, se mostrará el valor de la clave.
>      - Copie el valor de la clave. Lo necesitará más adelante.
> 1. Seleccione la sección **Permisos de API**.
>
>      - Haga clic en el botón **Agregar un permiso** y, a continuación,
>      - Asegúrese de que la pestaña **API de Microsoft** esté seleccionada.
>      - En la sección *API de Microsoft más usadas*, haga clic en **Microsoft Graph**.
>      - En la sección **Permisos delegados**, asegúrese de que estén marcados los permisos correctos: **User.ReadBasic.All**. Si es necesario, utilice el cuadro de búsqueda.
>      - Seleccione el botón **Agregar permisos**.
>
> [!div class="sxs-lookup" renderon="portal"]
>
> #### <a name="step-1-configure-your-application-in-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
>
> Para que el código de ejemplo de este inicio rápido funcione, es preciso:
>
> 1. Agregue una dirección URL de respuesta como `http://localhost:5000/getAToken`.
> 1. Cree un secreto de cliente.
> 1. Agregue el permiso delegado User.ReadBasic.All de Microsoft Graph API.
>
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-aspnet-webapp/green-check.png) La aplicación está configurada con este atributo

#### <a name="step-2-download-your-project"></a>Paso 2: Descarga del proyecto
> [!div renderon="docs"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-python-webapp/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> Descargue el proyecto y extraiga el archivo ZIP en la carpeta local más próxima a la carpeta raíz (por ejemplo, **C:\Azure-Samples**)
> [!div renderon="portal" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-python-webapp/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-application"></a>Paso 3: Configuración de la aplicación
>
> 1. Extraiga el archivo ZIP en la carpeta local más próxima a la carpeta raíz (por ejemplo, **C:\Azure-Samples**).
> 1. Si usa un entorno de desarrollo integrado, abra el ejemplo en su IDE favorito (opcional).
> 1. Abra el archivo **app_config.py**, que se puede encontrar en la carpeta raíz, y reemplace el siguiente fragmento de código:
>
> ```python
> CLIENT_ID = "Enter_the_Application_Id_here"
> CLIENT_SECRET = "Enter_the_Client_Secret_Here"
> AUTHORITY = "https://login.microsoftonline.com/Enter_the_Tenant_Name_Here"
> ```
> Donde:
>
> - `Enter_the_Application_Id_here`: es el identificador de aplicación de la aplicación que registró.
> - `Enter_the_Client_Secret_Here`: es el valor de **Secreto de cliente** que creó en **Certificates & Secrets** (Certificados y secretos) para la aplicación registrada.
> - `Enter_the_Tenant_Name_Here`: el valor de **Id. de directorio (inquilino)** de la aplicación que registró.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-run-the-code-sample"></a>Paso 3: Ejecución del ejemplo de código

> [!div renderon="docs"]
> #### <a name="step-4-run-the-code-sample"></a>Paso 4: Ejecución del ejemplo de código

1. Deberá instalar la biblioteca Python de MSAL, el marco de Flask, las sesiones de Flask para la administración de sesiones del lado servidor y solicitudes mediante pip de la manera siguiente:

    ```Shell
    pip install -r requirements.txt
    ```

2. Ejecute app.py desde el shell o la línea de comandos:

    ```Shell
    python app.py
    ```
   > [!IMPORTANT]
   > Esta aplicación de inicio rápido usa un secreto de cliente para identificarse como cliente confidencial. Como el secreto de cliente se agrega como texto sin formato a los archivos del proyecto, por motivos de seguridad, se recomienda que use un certificado en lugar de un secreto de cliente antes de considerar el uso de la aplicación en producción. Para más información sobre cómo usar un certificado, consulte [estas instrucciones](./active-directory-certificate-credentials.md).

## <a name="more-information"></a>Más información

### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo
![Muestra cómo funciona la aplicación de ejemplo generada por este inicio rápido.](media/quickstart-v2-python-webapp/python-quickstart.svg)

### <a name="getting-msal"></a>Obtención de MSAL
MSAL es la biblioteca que se usa para iniciar la sesión de los usuarios y solicitar los tokens que se usan para acceder a una API protegida por la Plataforma de identidad de Microsoft.
Puede agregar MSAL Python a la aplicación mediante Pip.

```Shell
pip install msal
```

### <a name="msal-initialization"></a>Inicialización de MSAL
Para agregar la referencia a MSAL Python, agregue el código siguiente en la parte superior del archivo en el que va a usar MSAL:

```Python
import msal
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las aplicaciones web que inician la sesión de los usuarios y que luego llaman a las API web:

> [!div class="nextstepaction"]
> [Escenario: Aplicaciones web que inician la sesión de los usuarios](scenario-web-app-sign-user-overview.md)

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]