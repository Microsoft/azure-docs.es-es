---
title: 'Tutorial: Autenticación con Azure Functions: Azure Signalr'
description: En este tutorial, aprenderá cómo autenticar los clientes de Azure SignalR Service para el enlace con Azure Functions
author: sffamily
ms.service: signalr
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: zhshang
ms.custom: devx-track-javascript
ms.openlocfilehash: 45dc137141491938367fb57c6955e8e3145f8ff9
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050461"
---
# <a name="tutorial-azure-signalr-service-authentication-with-azure-functions"></a>Tutorial: autenticación de Azure SignalR Service con Azure Functions

Un tutorial detallado para crear un salón de chat con autenticación y mensajería privadas mediante Azure Functions, App Service y SignalR Service.

## <a name="introduction"></a>Introducción

### <a name="technologies-used"></a>Tecnologías usadas

* [Azure Functions](https://azure.microsoft.com/services/functions/?WT.mc_id=serverlesschatlab-tutorial-antchu): API de back-end para autenticar a los usuarios y enviar mensajes de chat
* [Azure SignalR Service](https://azure.microsoft.com/services/signalr-service/?WT.mc_id=serverlesschatlab-tutorial-antchu): Difusión de nuevos mensajes a los clientes de chat conectados
* [Azure Storage](https://azure.microsoft.com/services/storage/?WT.mc_id=serverlesschatlab-tutorial-antchu): Hospedaje del sitio web estático para la interfaz de usuario del cliente de chat

### <a name="prerequisites"></a>Prerrequisitos

El siguiente software es necesario para poder seguir este tutorial.

* [Git](https://git-scm.com/downloads)
* [Node.js](https://nodejs.org/en/download/) (versión 10.x)
* [SDK de .NET](https://www.microsoft.com/net/download) (versión 2.x, necesaria para las extensiones de Functions)
* [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools) (versión 2)
* [Visual Studio Code](https://code.visualstudio.com/) (VS Code) con las siguientes extensiones
  * [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions): trabajo con Azure Functions en VS Code
  * [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer): ofrece páginas web localmente para la realización de pruebas

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="sign-into-the-azure-portal"></a>Inicie sesión en Azure Portal.

Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con sus credenciales.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="create-an-azure-signalr-service-instance"></a>Creación de una instancia del servicio Azure SignalR

Va a compilar y probar la aplicación de Azure Functions localmente. La aplicación accederá a la instancia de Azure SignalR Service que se debe crear con antelación.

1. Haga clic en el botón **Crear un recurso** ( **+** ) para crear un nuevo recurso de Azure.

1. Busque **SignalR Service** y selecciónelo. Haga clic en **Crear**.

    ![Nueva instancia de SignalR Service](media/signalr-tutorial-authenticate-azure-functions/signalr-quickstart-new.png)

1. Escriba la siguiente información.

    | Nombre | Value |
    |---|---|
    | Nombre del recurso | Un nombre único para la instancia del servicio SignalR |
    | Resource group | Cree un grupo de recursos y asígnele con un nombre único |
    | Location | Selección de una ubicación cercana a usted |
    | Nivel de precios | Gratuito |

1. Haga clic en **Crear**.

1. Una vez implementada la instancia, ábrala en el portal y busque su página de configuración. Cambie la configuración del modo de servicio a *Sin servidor*.

    ![Modo SignalR Service](media/signalr-concept-azure-functions/signalr-service-mode.png)
    
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="initialize-the-function-app"></a>Inicialización de la aplicación de función

### <a name="create-a-new-azure-functions-project"></a>Creación de un proyecto de Azure Functions

1. En una nueva ventana de VS Code, utilice `File > Open Folder` en el menú para crear y abrir una carpeta vacía en una ubicación adecuada. Esta será la carpeta de proyecto principal para la aplicación que va a compilar.

1. Mediante la extensión de Azure Functions en VS Code, inicialice una aplicación de función en la carpeta de proyecto principal.
   1. Abra la paleta de comandos en VS Code seleccionando **Vista > Paleta de comandos** en el menú (método abreviado `Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).
   1. Busque el comando **Azure Functions: Create New Project** y selecciónelo.
   1. Debería aparecer la carpeta de proyecto principal. Selecciónela (o utilice "Examinar" para localizarla).
   1. En la solicitud para elegir un idioma, seleccione **JavaScript**.

      ![Creación de una aplicación de función](media/signalr-tutorial-authenticate-azure-functions/signalr-create-vscode-app.png)

### <a name="install-function-app-extensions"></a>Instalación de extensiones de la aplicación de función

Este tutorial utiliza enlaces de Azure Functions para interactuar con Azure SignalR Service. Al igual que la mayoría de los otros enlaces, los enlaces de SignalR Service están disponibles como una extensión que se debe instalar mediante la CLI de Azure Functions Core Tools antes de que se puedan usar.

1. Abra un terminal en VS Code; para ello, seleccione **Vista > Terminal** en el menú (Ctrl-\`).

1. Asegúrese de que la carpeta de proyecto principal es el directorio actual.

1. Instale la extensión de la aplicación de función de SignalR Service.

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.SignalRService -v 1.0.0
    ```

### <a name="configure-application-settings"></a>Configuración de la aplicación

Al ejecutar y depurar el entorno en tiempo de ejecución de Azure Functions localmente, la configuración de la aplicación se lee desde **local.settings.json**. Actualice este archivo con la cadena de conexión de la instancia de SignalR Service que creó anteriormente.

1. En VS Code, seleccione **local.settings.json** en el panel Explorador para abrirlo.

1. Reemplace el contenido del archivo por lo siguiente.

    ```json
    {
        "IsEncrypted": false,
        "Values": {
            "AzureSignalRConnectionString": "<signalr-connection-string>",
            "WEBSITE_NODE_DEFAULT_VERSION": "10.14.1",
            "FUNCTIONS_WORKER_RUNTIME": "node"
        },
        "Host": {
            "LocalHttpPort": 7071,
            "CORS": "http://127.0.0.1:5500",
            "CORSCredentials": true
        }
    }
    ```

   * Escriba la cadena de conexión de Azure SignalR Service en una configuración denominada `AzureSignalRConnectionString`. Obtenga el valor de la página **Claves** del recurso de Azure SignalR Service en Azure Portal; se puede utilizar la cadena de conexión principal o la secundaria.
   * El valor `WEBSITE_NODE_DEFAULT_VERSION` no se usa de forma local, pero es necesario cuando la implementación se realiza en Azure.
   * La sección `Host` configura el puerto y la configuración de CORS para el host de Functions local (esta configuración no tiene efecto cuando se ejecuta en Azure).

       > [!NOTE]
       > Normalmente, el servidor activo se configura para servir contenido desde `http://127.0.0.1:5500`. Si descubre que usa una dirección URL diferente o si usa un servidor HTTP diferente, cambie la configuración de `CORS` para reflejar el origen correcto.

     ![Obtención de una clave de SignalR Service](media/signalr-tutorial-authenticate-azure-functions/signalr-get-key.png)

1. Guarde el archivo.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="create-a-function-to-authenticate-users-to-signalr-service"></a>Creación de una función para autenticar usuarios en SignalR Service

Cuando se abre la aplicación de chat por primera vez en el explorador, este requiere unas credenciales de conexión válidas para conectarse a Azure SignalR Service. Va a crear una función desencadenada por HTTP denominada *negotiate* en la aplicación de función para devolver esta información de conexión.

> [!NOTE]
> Esta función debe denominarse *negotiate*, ya que el cliente de SignalR requiere un punto de conexión que termine en `/negotiate`.

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure Functions: Crear función**.

1. Cuando se le pida, proporcione la siguiente información.

    | Nombre | Value |
    |---|---|
    | Carpeta de la aplicación de función | Seleccione la carpeta de proyecto principal. |
    | Plantilla | Desencadenador HTTP |
    | Nombre | negotiate |
    | Nivel de autorización | Anónimas |

    Se crea una carpeta denominada **negotiate** que contiene la nueva función.

1. Abra **negotiate/function.json** para configurar enlaces para la función. Modifique el contenido del archivo para que sea el siguiente. Esto agrega un enlace de entrada que genera credenciales válidas para que un cliente se pueda conectar a un centro de Azure SignalR Service llamado `chat`.

    ```json
    {
        "disabled": false,
        "bindings": [
            {
                "authLevel": "anonymous",
                "type": "httpTrigger",
                "direction": "in",
                "name": "req"
            },
            {
                "type": "http",
                "direction": "out",
                "name": "res"
            },
            {
                "type": "signalRConnectionInfo",
                "name": "connectionInfo",
                "userId": "",
                "hubName": "chat",
                "direction": "in"
            }
        ]
    }
    ```

    La propiedad `userId` del enlace `signalRConnectionInfo` se usa para crear una conexión de SignalR Service autenticada. Deje la propiedad en blanco en caso de desarrollo local. La usará cuando la aplicación de función se implemente en Azure.

1. Abra **negotiate/index.js** para ver el cuerpo de la función. Modifique el contenido del archivo para que sea el siguiente.

    ```javascript
    module.exports = async function (context, req, connectionInfo) {
        context.res.body = connectionInfo;
    };
    ```

    Esta función toma la información de conexión de SignalR del enlace de entrada y la devuelve al cliente en el cuerpo de la respuesta HTTP. El cliente de SignalR usará esta información para conectarse a la instancia de SignalR Service.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="create-a-function-to-send-chat-messages"></a>Creación de una función para enviar mensajes de chat

La aplicación web también necesita una API de HTTP para enviar mensajes de chat. Va a crear una función desencadenada por HTTP denominada *SendMessage* que envía mensajes a todos los clientes conectados mediante SignalR Service.

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure Functions: Crear función**.

1. Cuando se le pida, proporcione la siguiente información.

    | Nombre | Value |
    |---|---|
    | Carpeta de la aplicación de función | Selección de la carpeta de proyecto principal |
    | Plantilla | Desencadenador HTTP |
    | Nombre | SendMessage |
    | Nivel de autorización | Anónimas |

    Se crea una carpeta denominada **SendMessage** que contiene la nueva función.

1. Abra **SendMessage/function.json** para configurar enlaces para la función. Modifique el contenido del archivo para que sea el siguiente.
    ```json
    {
        "disabled": false,
        "bindings": [
            {
                "authLevel": "anonymous",
                "type": "httpTrigger",
                "direction": "in",
                "name": "req",
                "route": "messages",
                "methods": [
                    "post"
                ]
            },
            {
                "type": "http",
                "direction": "out",
                "name": "res"
            },
            {
                "type": "signalR",
                "name": "$return",
                "hubName": "chat",
                "direction": "out"
            }
        ]
    }
    ```
    Esto hace que se realicen dos cambios en la función original:
    * Cambia la ruta a `messages` y restringe el desencadenador de HTTP para el método HTTP **POST**.
    * Agrega un enlace de salida de SignalR Service que envía un mensaje devuelto por la función a todos clientes conectados a un centro de SignalR Service llamado `chat`.

1. Guarde el archivo.

1. Abra **SendMessage/index.js** para ver el cuerpo de la función. Modifique el contenido del archivo para que sea el siguiente.

    ```javascript
    module.exports = async function (context, req) {
        const message = req.body;
        message.sender = req.headers && req.headers['x-ms-client-principal-name'] || '';

        let recipientUserId = '';
        if (message.recipient) {
            recipientUserId = message.recipient;
            message.isPrivate = true;
        }

        return {
            'userId': recipientUserId,
            'target': 'newMessage',
            'arguments': [ message ]
        };
    };
    ```

    Esta función toma el cuerpo de la solicitud HTTP y lo envía a los clientes conectados a SignalR Service, invocando una función denominada `newMessage` en cada cliente.

    La función puede leer la identidad del emisor y puede aceptar un valor de *destinatario* en el cuerpo del mensaje para permitir que se envíe un mensaje de forma privada a un solo usuario. Estas funcionalidades se utilizarán más adelante en el tutorial.

1. Guarde el archivo.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="create-and-run-the-chat-client-web-user-interface"></a>Creación y ejecución de la interfaz de usuario web del cliente de chat

La interfaz de usuario de la aplicación de chat es una aplicación de página única (SPA) creada con la plataforma Vue JavaScript. Se hospedará de forma independiente con respecto a la aplicación de función. Localmente, se ejecutará la interfaz web con la extensión de Live Server VS Code.

1. En VS Code, cree una carpeta denominada **content** en la raíz de la carpeta de proyecto principal.

1. En la carpeta **content**, cree un nuevo archivo denominado **index.html**.

1. Copie y peque el contenido de **[index.html](https://github.com/Azure-Samples/signalr-service-quickstart-serverless-chat/blob/2720a9a565e925db09ef972505e1c5a7a3765be4/docs/demo/chat-with-auth/index.html)**.

1. Guarde el archivo.

1. Presione **F5** para ejecutar la aplicación de función localmente y asociar un depurador.

1. Con **index.html** abierto, inicie Live Server mediante la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`) y seleccione **Live Server: Open with Live Server**. Live Server abrirá la aplicación en un explorador.

1. La aplicación se abre. Escriba un mensaje en el cuadro de chat y presione ENTRAR. Actualice la aplicación para ver los mensajes nuevos. Como no se ha configurado ninguna autenticación, todos los mensajes se enviarán como "anónimo".

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="deploy-to-azure-and-enable-authentication"></a>Implementación en Azure y habilitación de la autenticación

Hasta ahora, ha ejecutado la aplicación de función y la de chat de forma local. Ahora las implementará en Azure y habilitará la autenticación y la mensajería privadas en la aplicación.

### <a name="log-into-azure-with-vs-code"></a>Inicio de sesión en Azure con VS Code

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure: Sign in**.

1. Siga las instrucciones para completar el proceso de inicio de sesión en el explorador.

### <a name="create-a-storage-account"></a>Creación de una cuenta de Storage

Para una aplicación de función que se ejecute en Azure, se necesita una cuenta de Azure Storage. También va a hospedar la página web para el servicio de chat de la interfaz de usuario mediante la característica de los sitios web estáticos de Azure Storage.

1. En Azure Portal, haga clic en el botón **Crear un recurso** ( **+** ) para crear un recurso de Azure.

1. Seleccione la categoría **Almacenamiento** y **Cuenta de almacenamiento**.

1. Escriba la siguiente información.

    | Nombre | Value |
    |---|---|
    | Subscription | Seleccione la suscripción que contenga la instancia de SignalR Service |
    | Resource group | Seleccione el mismo grupo de recursos |
    | Nombre del recurso | Nombre único de la cuenta de almacenamiento |
    | Location | Seleccione la ubicación de los demás recursos |
    | Rendimiento | Estándar |
    | Tipo de cuenta | StorageV2 (uso general v2) |
    | Replicación | Almacenamiento con redundancia local (LRS) |
    | Nivel de acceso | Acceso frecuente |

1. Haga clic en **Revisar + crear** y en **Crear**.

### <a name="configure-static-websites"></a>Configuración de los sitios web estáticos

1. Al crear la cuenta de Storage, ábrala en Azure Portal.

1. Seleccione **Sitio web estático**.

1. Seleccione **Habilitado** para habilitar la característica de sitio web estático.

1. En **Nombre del documento de índice**, escriba *index.html*.

1. Haga clic en **Save**(Guardar).

1. Aparecerá un **punto de conexión principal**. Anote este valor. Lo necesitará para configurar la aplicación de función.

### <a name="configure-function-app-for-authentication"></a>Configuración de una aplicación de función para la autenticación

Hasta ahora, la aplicación de chat funciona de forma anónima. En Azure, deberá utilizar [Autenticación de App Service](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization) para autenticar al usuario. El identificador o el nombre de usuario del usuario autenticado se puede pasar al enlace *SignalRConnectionInfo* para generar información de conexión que se autentica como el usuario.

Cuando se envía un mensaje, la aplicación puede decidir enviarlo a todos los clientes conectados, o solo a aquellos clientes que se han a autenticado en un determinado usuario.

1. En VS Code, abra **negotiate/function.json**.

1. Inserte una [expresión de enlace](https://docs.microsoft.com/azure/azure-functions/functions-triggers-bindings) en la propiedad *userId* del enlace *SignalRConnectionInfo*: `{headers.x-ms-client-principal-name}`. Esto establece el valor en el nombre de usuario del usuario autenticado. El atributo tendrá ahora un aspecto similar al siguiente.

    ```json
    {
        "type": "signalRConnectionInfo",
        "name": "connectionInfo",
        "userId": "{headers.x-ms-client-principal-name}",
        "hubName": "chat",
        "direction": "in"
    }
    ```

1. Guarde el archivo.


### <a name="deploy-function-app-to-azure"></a>Implementación de una aplicación de función en Azure

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`) y seleccione el comando **Azure Functions: Deploy to Function App**.

1. Cuando se le pida, proporcione la siguiente información.

    | Nombre | Value |
    |---|---|
    | Carpeta de implementación | Seleccione la carpeta de proyecto principal. |
    | Subscription | Seleccione su suscripción. |
    | Aplicación de función | Seleccione **Create New Function App** (Crear aplicación de función). |
    | Nombre de la aplicación de funciones | Escriba un nombre único. |
    | Resource group | Seleccione el mismo grupo de recursos que la instancia de SignalR Service. |
    | Cuenta de almacenamiento | Seleccione la cuenta de almacenamiento que creó anteriormente |

    Se crea una aplicación de funciones en Azure y comienza la implementación. Espere a que la implementación se complete.

### <a name="upload-function-app-local-settings"></a>Carga de la configuración local de la aplicación de función

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure Functions: Upload local settings**.

1. Cuando se le pida, proporcione la siguiente información.

    | Nombre | Value |
    |---|---|
    | Archivo de configuración local | local.settings.json |
    | Subscription | Seleccione su suscripción. |
    | Aplicación de función | Seleccione la aplicación de función implementada anteriormente. |

La configuración local se carga en la aplicación de función en Azure. Si se le pide que sobrescriba la configuración existente, seleccione **Sí a todo**.


### <a name="enable-app-service-authentication"></a>Habilitación de la autenticación de App Service

La autenticación de App Service admite la autenticación con Azure Active Directory, Facebook, Twitter y cuentas de Microsoft y Google.

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure Functions: Open in portal**.

1. Seleccione la suscripción y el nombre de la aplicación de función para abrir la aplicación de función en Azure Portal.

1. Con la aplicación de función que abrió en el portal, busque la pestaña **Características de la plataforma** y seleccione **Autenticación o autorización**.

1. **Active** la autenticación de App Service.

1. En **Acción necesaria cuando la solicitud no está autenticada**, seleccione "Iniciar sesión con {el proveedor de autenticación que seleccionó anteriormente}".

1. En **URL de redirección externas permitidas**, escriba la dirección URL del punto de conexión web principal de la cuenta de almacenamiento que anotó anteriormente.

1. Siga la documentación del proveedor de inicio de sesión de su elección para completar la configuración.

    - [Azure Active Directory](https://docs.microsoft.com/azure/app-service/configure-authentication-provider-aad)
    - [Facebook](https://docs.microsoft.com/azure/app-service/configure-authentication-provider-facebook)
    - [Twitter](https://docs.microsoft.com/azure/app-service/configure-authentication-provider-twitter)
    - [Cuenta Microsoft](https://docs.microsoft.com/azure/app-service/configure-authentication-provider-microsoft)
    - [Google](https://docs.microsoft.com/azure/app-service/configure-authentication-provider-google)

### <a name="update-the-web-app"></a>Actualización de la aplicación web

1. En Azure Portal, navegue a la página de información general de la aplicación de función.

1. Copie la dirección URL de la aplicación de función.

    ![Obtener dirección URL](media/signalr-tutorial-authenticate-azure-functions/signalr-get-url.png)

1. En VS Code, abra **index.html** y reemplace el valor de `apiBaseUrl` por la dirección URL de la aplicación de función.

1. La aplicación se puede configurar con autenticación mediante Azure Active Directory, Facebook, Twitter o una cuenta de Microsoft o Google. Seleccione el proveedor de autenticación que va a utilizar mediante el establecimiento del valor de `authProvider`.

1. Guarde el archivo.

### <a name="deploy-the-web-application-to-blob-storage"></a>Implementación de la aplicación web en Blob Storage

La aplicación web se hospedará mediante la característica de sitios web estáticos de Azure Blob Storage.

1. Abra la paleta de comandos de VS Code (`Ctrl-Shift-P`, macOS: `Cmd-Shift-P`).

1. Busque y seleccione el comando **Azure Storage: Deploy to Static Website** (Implementar en sitio web estático).

1. Escriba los siguientes valores:

    | Nombre | Value |
    |---|---|
    | Subscription | Seleccione su suscripción. |
    | Cuenta de almacenamiento | Seleccione la cuenta de almacenamiento que creó anteriormente |
    | Carpeta de implementación | Seleccione **Examinar** y la carpeta *content* |

Los archivos de la carpeta *content* ahora deben implementarse en el sitio web estático.

### <a name="enable-function-app-cross-origin-resource-sharing-cors"></a>Habilitación del uso compartido de recursos entre orígenes (CORS) para la aplicación de función

Aunque hay una configuración de CORS en **local.settings.json**, esta no se propaga a la aplicación de función en Azure. Deberá configurarla de forma independiente.

1. Abra la aplicación de función en Azure Portal.

1. En la pestaña **Características de la plataforma**, seleccione **CORS**.

    ![Buscar CORS](media/signalr-tutorial-authenticate-azure-functions/signalr-find-cors.png)

1. En la sección *Orígenes permitidos*, agregue una entrada con el *punto de conexión principal* del sitio web estático como valor (quite el símbolo */* del final).

1. Para que el SDK de JavaScript de SignalR llame a la aplicación de función desde un explorador, debe estar habilitada la compatibilidad con las credenciales en CORS. Seleccione la casilla "Habilitar Access-Control-Allow-Credentials".

    ![Habilitar Access-Control-Allow-Credentials](media/signalr-tutorial-authenticate-azure-functions/signalr-cors-credentials.png)

1. Haga clic en **Guardar** para guardar la configuración de CORS.

### <a name="try-the-application"></a>Prueba de la aplicación

1. En un explorador, vaya al punto de conexión web principal de la cuenta de almacenamiento.

1. Seleccione **Iniciar sesión** para autenticarse con el proveedor de autenticación de su elección.

1. Para enviar mensajes públicos, escríbalos en el cuadro de chat principal.

1. Para enviar mensajes privados, haga clic en un nombre de usuario en el historial de chat. Solo el destinatario seleccionado recibirá estos mensajes.

Felicidades. Ha implementado una aplicación de chat en tiempo real, sin servidor.

![Demostración](media/signalr-tutorial-authenticate-azure-functions/signalr-serverless-chat.gif)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados en este tutorial, elimine el grupo de recursos mediante Azure Portal.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar Azure Functions con Azure SignalR Service. Conozca más información sobre la compilación de aplicaciones sin servidor en tiempo real con enlaces de SignalR Service para Azure Functions.

> [!div class="nextstepaction"]
> [Creación de aplicaciones en tiempo real con Azure Functions](signalr-concept-azure-functions.md)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsauth)

