---
title: 'Tutorial: Habilitación de la autenticación en una aplicación web de Python'
titleSuffix: Azure AD B2C
description: En este tutorial, aprenderá a usar Azure Active Directory B2C para proporcionar inicio de sesión de usuario en una aplicación web de Python Flask.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 06/12/2020
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.custom: devx-track-python
ms.openlocfilehash: 6a113b3a3df475853f3690a0e932378bc0e2ab02
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844633"
---
# <a name="tutorial-enable-authentication-in-a-python-web-application-with-azure-ad-b2c"></a>Tutorial: Habilitación de la autenticación en una aplicación web de Python mediante Azure AD B2C

En este tutorial se muestra cómo usar Azure Active Directory B2C (Azure AD B2C) para que los usuarios se registren e inicien sesión en una aplicación web de Python Flask.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregue una dirección URL de respuesta a una aplicación registrada en el inquilino de Azure AD B2C.
> * Descargue un código de ejemplo de GitHub.
> * Modifique el código de ejemplo de la aplicación para que funcione con el inquilino.
> * Regístrese con el flujo de usuario de registro o inicio de sesión

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de continuar con los pasos en este tutorial, necesita tener los siguientes recursos de Azure AD B2C:

* [Inquilino de Azure AD B2C](tutorial-create-tenant.md)
* [Aplicación registrada](tutorial-register-applications.md) en el inquilino y su *Id. de aplicación (cliente)* y *secreto de cliente*
* [Flujos de usuario creados](tutorial-create-user-flows.md) en su inquilino

También necesita los siguientes elementos en su entorno de desarrollo:

* [Visual Studio Code](https://code.visualstudio.com/) u otro editor de código
* [Python](https://nodejs.org/en/download/) 2.7+ o 3+

## <a name="add-a-redirect-uri"></a>Incorporación de un URI de redirección

En el segundo tutorial que completó como parte de los requisitos previos, registró una aplicación web en Azure AD B2C. Para habilitar la comunicación con el código de ejemplo en este tutorial, agregue una dirección URL de respuesta (también denominado URI de redirección) al registro de aplicación.

Para actualizar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](https://aka.ms/b2cappregtraining).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones**, la pestaña **Aplicaciones propias** y, por último, la aplicación *webapp1*.
1. En **Administrar**, seleccione **Autenticación**.
1. En **Web**, seleccione el vínculo **Agregar URI** y, a continuación, escriba `http://localhost:5000/getAToken` en el cuadro de texto.
1. Seleccione **Guardar**.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Seleccione **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Aplicaciones (heredado)** y, a continuación, la aplicación *webapp1*.
1. En **URL de respuesta**, agregue `http://localhost:5000/getAToken`.
1. Seleccione **Guardar**.
* * *

## <a name="get-the-sample-code"></a>Obtención del código de ejemplo

En este tutorial, va a configurar un código de ejemplo que puede descargar desde GitHub para que funcione con el inquilino de B2C. En el ejemplo se muestra cómo una aplicación web de Python Flask puede usar Azure AD B2C para el registro y el inicio de sesión de usuario.

[Descargue un archivo .ZIP](https://github.com/Azure-Samples/ms-identity-python-webapp/archive/master.zip) o clone el [repositorio del ejemplo de código](https://github.com/Azure-Samples/ms-identity-python-webapp) de GitHub.

```console
git clone https://github.com/Azure-Samples/ms-identity-python-webapp.git
```

## <a name="update-the-sample"></a>Actualización del ejemplo

Una vez que haya conseguido el ejemplo, configure la aplicación para que use el inquilino de Azure AD B2C, el registro de aplicación y los flujos de usuario.

En el directorio raíz del proyecto:

1. Cambie el nombre del archivo *app_config.py* a *app_config.py.OLD*.
1. Cambie el nombre del archivo *app_config_b2c.py* a *app_config.py*.

Actualice el nuevo archivo *app_config.py* con los valores del inquilino de Azure AD B2C y el registro de aplicación que creó como parte de los requisitos previos.

1. Abra el archivo *app_config.py* en el editor.
1. Actualice el valor de `b2c_tenant` con el nombre de su inquilino de Azure AD B2C, por ejemplo *contosob2c*.
1. Actualice cada uno de los valores de `*_user_flow` para que coincidan con los nombres de los flujos de usuario que creó como parte de los requisitos previos.
1. Actualice el valor de `CLIENT_ID` con el valor de **Id. de aplicación (cliente)** de la aplicación web registrada como parte de los requisitos previos.
1. Actualice el valor de `CLIENT_SECRET` con el valor del **secreto de cliente** que creó en los requisitos previos. Para mayor seguridad, considere la posibilidad de almacenarlo en una **variable de entorno**, como se recomienda en los comentarios.

La sección superior de *app_config.py* debe tener ahora un aspecto similar al siguiente fragmento de código:

```python
import os

b2c_tenant = "contosob2c"
signupsignin_user_flow = "B2C_1_signupsignin1"
editprofile_user_flow = "B2C_1_profileediting1"
resetpassword_user_flow = "B2C_1_passwordreset1"
authority_template = "https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{user_flow}"

CLIENT_ID = "11111111-1111-1111-1111-111111111111" # Application (client) ID of app registration

CLIENT_SECRET = "22222222-2222-2222-2222-222222222222" # Placeholder - for use ONLY during testing.
# In a production app, we recommend you use a more secure method of storing your secret,
# like Azure Key Vault. Or, use an environment variable as described in Flask's documentation:
# https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables
# CLIENT_SECRET = os.getenv("CLIENT_SECRET")
# if not CLIENT_SECRET:
#     raise ValueError("Need to define CLIENT_SECRET environment variable")
```

> [!WARNING]
> Como se indica en los comentarios del fragmento de código, se recomienda **no almacenar secretos en texto no cifrado** del código de la aplicación. En el ejemplo de código se usa la variable codificada de forma rígida *únicamente por comodidad*. Considere la posibilidad de usar una variable de entorno o un almacén de secretos, como Azure Key Vault.

## <a name="run-the-sample"></a>Ejecución del ejemplo

1. En la consola o el terminal, cambie al directorio que contiene el ejemplo. Por ejemplo:

    ```console
    cd ms-identity-python-webapp
    ```
1. Ejecute los siguientes comandos para instalar los paquetes necesarios desde PyPi y ejecutar la aplicación web en la máquina local:

    ```console
    pip install -r requirements.txt
    flask run --host localhost --port 5000
    ```

    La ventana de consola muestra el número de puerto de la aplicación que se ejecuta localmente:

    ```console
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://localhost:5000/ (Press CTRL+C to quit)
    ```

1. Vaya a `http://localhost:5000` para ver la aplicación web que se ejecuta en la máquina local.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-01.png" alt-text="Explorador Web que muestra la aplicación web de Python Flask que se ejecuta localmente":::

### <a name="sign-up-using-an-email-address"></a>Registro con una dirección de correo electrónico

Esta aplicación de ejemplo admite el registro, el inicio de sesión y el restablecimiento de contraseña. En este tutorial se registra mediante una dirección de correo electrónico.

1. Seleccione **Iniciar sesión** para iniciar el flujo de usuario *B2C_1_signupsignin1* que ha especificado en un paso anterior.
1. Azure AD B2C presenta una página de inicio de sesión que incluye un vínculo de registro. Como no tiene aún una cuenta, seleccione el vínculo **Registrarse ahora**.
1. El flujo de trabajo del registro presenta una página para recopilar y verificar la identidad del usuario con una dirección de correo electrónico. El flujo de trabajo del registro también recopila la contraseña de usuario y los atributos requeridos definidos en el flujo de usuario.

    Utilice una dirección de correo electrónico válida y valídela mediante el código de verificación. Establezca una contraseña. Especifique valores para los atributos solicitados.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-02.png" alt-text="Página de registro que muestra el flujo de usuario de Azure AD B2C":::

1. Seleccione **Crear** para crear una cuenta local en el directorio de Azure AD B2C.

Al seleccionar **Crear**, la aplicación muestra el nombre del usuario que ha iniciado sesión.

:::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-03.png" alt-text="Explorador Web que muestra la aplicación web de Python Flask con el usuario que ha iniciado sesión":::

Si desea probar el inicio de sesión, seleccione el vínculo **Cerrar sesión**, luego seleccione **Iniciar sesión** e inicie sesión con la dirección de correo electrónico y la contraseña que especificó al registrarse.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado una aplicación web de Python Flask a fin de que funcione con un flujo de usuario en el inquilino de Azure AD B2C para proporcionar la funcionalidad de registro e inicio de sesión. Ha completado estos pasos:

> [!div class="checklist"]
> * Ha agregado una dirección URL de respuesta a una aplicación registrada en el inquilino de Azure AD B2C.
> * Ha descargado un código de ejemplo desde GitHub.
> * Ha modificado el código de ejemplo de la aplicación para que funcione con el inquilino.
> * Se ha registrado con el flujo de usuario de registro o inicio de sesión.

A continuación, aprenda a personalizar la interfaz de usuario de las páginas de flujo de usuario que se muestran a los usuarios mediante Azure AD B2C:

> [!div class="nextstepaction"]
> [Tutorial: Personalización de la interfaz de experiencias de usuario en Azure Active Directory B2C](tutorial-customize-ui.md)
