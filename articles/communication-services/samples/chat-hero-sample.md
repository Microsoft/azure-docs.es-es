---
title: Ejemplo de elementos principales de un chat grupal
titleSuffix: An Azure Communication Services sample overview
description: Información general sobre el ejemplo de elementos principales de chat al usar Azure Communication Services para que los desarrolladores puedan obtener más información sobre el funcionamiento interno del ejemplo y aprender a modificarlo.
author: ddematheu
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 295c4bde64ad21a19d21fd48f2556114b26b202d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945282"
---
# <a name="get-started-with-the-group-chat-hero-sample"></a>Introducción al ejemplo de elementos principales de un chat grupal

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!----
> [!WARNING]
> links to our Hero Sample repo need to be updated when the sample is publicly available.
---->

En el ejemplo de salón de chat de grupo **de Azure Communication Services** se muestra cómo se puede usar la biblioteca de cliente web de chat de servicios de comunicación para crear una experiencia de llamada de grupo.

En esta guía de inicio rápido de ejemplo, veremos cómo funciona el ejemplo antes de ejecutarlo en la máquina local. A continuación, implementaremos el ejemplo en Azure con sus propios recursos de Azure Communication Services.

> [!IMPORTANT]
> [Descargue el ejemplo de GitHub](https://github.com/Azure/Communication/tree/master/samples).

## <a name="overview"></a>Información general

El ejemplo tiene una aplicación del lado cliente y una aplicación del lado servidor. La **aplicación del lado cliente** es una aplicación web React/Redux que usa el marco de interfaz de usuario Fluent de Microsoft. Esta aplicación envía solicitudes a una **aplicación del lado servidor** de ASP.NET Core que ayuda a la aplicación del lado cliente a conectarse a Azure. 

El ejemplo tendrá una apariencia similar a la siguiente:

:::image type="content" source="./media/chat/landing-page.png" alt-text="Captura de pantalla que muestra la página de aterrizaje de la aplicación de ejemplo.":::

Cuando se presiona el botón "Start chat", la aplicación web captura un token de acceso de usuario de la aplicación del lado servidor. A continuación, este token se usa para conectar la aplicación cliente con Azure Communication Services. Una vez que se recupera el token, se le pedirá que especifique su nombre y el emoji que le representará en el chat. 

:::image type="content" source="./media/chat/pre-chat.png" alt-text="Captura de pantalla que muestra la pantalla anterior al chat de la aplicación.":::

Una vez que configure el nombre para mostrar y el emoji, puede unirse a la sesión de chat. Ahora verá el lienzo de chat principal donde se encuentra la experiencia de chat principal.

:::image type="content" source="./media/chat/main-app.png" alt-text="Captura de pantalla que muestra la pantalla principal de la aplicación de ejemplo.":::

Componentes de la pantalla principal de chat:

- **Área de chat principal**: Aquí se encuentra la experiencia de chat principal, donde los usuarios pueden enviar y recibir mensajes. Para enviar mensajes, puede usar el área de entrada y presionar Entrar (o usar el botón de envío). Los mensajes de chat recibidos se clasifican por remitente con el nombre y el emoji correctos. Verá dos tipos de notificaciones en el área de chat: 1) notificaciones de escritura cuando un usuario esté escribiendo y 2) notificaciones de envío y lectura para los mensajes.
- **Encabezado**: Aquí es donde el usuario verá el título del hilo de chat y los controles para alternar las barras laterales de participantes y de configuración, y un botón Leave para salir de la sesión de chat.
- **Barra lateral**: Aquí es donde se muestran los participantes y la información de configuración cuando se alternan con los controles del encabezado. La barra lateral de participantes contiene una lista de participantes en el chat y un vínculo para invitar a los participantes a la sesión de chat. La barra lateral de configuración permite configurar el título del hilo de chat. 

A continuación encontrará más información sobre los requisitos previos y los pasos para configurar el ejemplo.

## <a name="prerequisites"></a>Requisitos previos

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Node.js (8.11.2 y posterior)](https://nodejs.org/en/download/)
- [Visual Studio (2017 y posterior)](https://visualstudio.microsoft.com/vs/)
- [.NET Core 2.2](https://dotnet.microsoft.com/download/dotnet-core/2.2) (asegúrese de instalar la versión que se corresponda con su instancia de Visual Studio, 32 o 64 bits)
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../quickstarts/create-communication-resource.md). Deberá registrar la **cadena de conexión** del recurso para esta guía de inicio rápido.

## <a name="locally-deploying-the-service--client-app"></a>Implementación local del servicio y la aplicación cliente

El ejemplo de chat de un solo hilo son esencialmente dos "aplicaciones": una aplicación cliente y otra servidor.

Abra Visual Studio en chat.csproj y ejecútelo en modo de depuración. Esto iniciará el servicio de front-end del chat. Cuando se visite la aplicación de servidor desde el explorador, el tráfico se redirigirá hacia el servicio de front-end de chat implementado localmente.

Para probar el ejemplo localmente, puede abrir varias sesiones del explorador con la dirección URL de su chat para simular un chat multiusuario.

### <a name="before-running-the-sample-for-the-first-time"></a>Antes de ejecutar el ejemplo por primera vez

1. Abra una instancia de PowerShell, Terminal Windows, símbolo del sistema o equivalente y navegue hasta el directorio donde le gustaría clonar el ejemplo.
2. `git clone`
3. Vaya a la carpeta **Chat/ClientApp** y ejecute `npm run setup`.
   1. Si ve un error 1, busque más arriba en la salida una dirección URL a la que tendrá que ir para autorizar al cliente. (La dirección URL tendrá el siguiente aspecto: `app.vssps.visualstudio.com/oauth2/authorize?clientid=...`). Una vez que visite la dirección URL en un explorador, copie el comando desde la ventana del explorador y ejecútelo.
   2. Vuelva a ejecutar el comando `npm run setup` una vez que complete el paso anterior.
4. Obtenga `Connection String` de Azure Portal. Para obtener más información sobre las cadenas de conexión, consulte [Creación de un recurso de Azure Communication Services](../quickstarts/create-communication-resource.md).
5. Una vez que obtenga `Connection String`, agregue la cadena de conexión al archivo **Chat/appsettings.json** que se encuentra en la carpeta Chat. Escriba la cadena de conexión en la variable: `ResourceConnectionString`.

### <a name="local-run"></a>Ejecución local

1. Vaya a la carpeta Chat.
2. Abra la solución `Chat.csproj` en Visual Studio.
3. Ejecute el proyecto `Chat`*.

\* El explorador se abrirá en localhost:5000 (donde el nodo implementa la aplicación cliente). La aplicación no es compatible con Internet Explorer.

#### <a name="troubleshooting"></a>Solución de problemas

- La solución no se compila, se producen errores durante la instalación o compilación de NPM.

Limpieza o recompilación de la solución de C#

## <a name="publish-the-sample-to-azure"></a>Publicación del ejemplo en Azure

1. Haga clic con el botón derecho en el proyecto `Chat` y seleccione Publicar.
2. Cree un nuevo perfil de publicación y seleccione la suscripción a Azure.
3. Antes de publicar, agregue la cadena de conexión con `Edit App Service Settings` y rellene `ResourceConnectionString` como clave y proporcione la cadena de conexión (copiada de appsettings.json) como valor.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- Más información sobre los [conceptos de chat](../concepts/chat/concepts.md)
- Familiarización con nuestra [biblioteca cliente de chat](../concepts/chat/sdk-features.md)

## <a name="additional-reading"></a>Lecturas adicionales

- [Versión preliminar de Azure Communication](https://github.com/Azure/communication-preview): para obtener más información sobre el SDK web de chat
- [Redux](https://redux.js.org/): Administración de estado del lado cliente
- [FluentUI](https://developer.microsoft.com/fluentui#/): Biblioteca de interfaz de usuario con tecnología de Microsoft
- [React](https://reactjs.org/): Biblioteca para compilar interfaces de usuario
- [ASP.NET Core](https://docs.microsoft.com/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-3.1&preserve-view=true): Marco para compilar aplicaciones web
