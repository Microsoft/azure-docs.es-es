---
title: Ejemplo de elementos principales de una llamada grupal
titleSuffix: An Azure Communication Services sample overview
description: Información general sobre el ejemplo de elementos principales de llamada al usar Azure Communication Services para que los desarrolladores puedan obtener más información sobre el funcionamiento interno del ejemplo.
author: ddematheu
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: caee5686695594604f49dcbade54342a9134abc0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945459"
---
# <a name="get-started-with-the-group-calling-hero-sample"></a>Introducción al ejemplo de elementos principales de una llamada

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!----
> [!WARNING]
> Add links to our Hero Sample repo when the sample is publicly available.
---->

En el **ejemplo de elementos principales de llamada grupal** de Azure Communication Services se muestra cómo se puede usar la biblioteca cliente web de llamadas de Communication Services para crear una experiencia de llamada grupal.

En esta guía de inicio rápido de ejemplo, veremos cómo funciona el ejemplo antes de ejecutarlo en la máquina local. A continuación, implementaremos el ejemplo en Azure con sus propios recursos de Azure Communication Services.

> [!IMPORTANT]
> [Descargue el ejemplo de GitHub](https://github.com/Azure/Communication/tree/master/samples).

## <a name="overview"></a>Información general

El ejemplo tiene una aplicación del lado cliente y una aplicación del lado servidor. La **aplicación del lado cliente** es una aplicación web React/Redux que usa el marco de interfaz de usuario Fluent de Microsoft. Esta aplicación envía solicitudes a una **aplicación del lado servidor** de ASP.NET Core que ayuda a la aplicación del lado cliente a conectarse a Azure. 

El ejemplo tendrá una apariencia similar a la siguiente:

:::image type="content" source="./media/calling/landing-page.png" alt-text="Captura de pantalla que muestra la página de aterrizaje de la aplicación de ejemplo.":::

Cuando se presiona el botón "Start a call", la aplicación web captura un token de acceso de usuario de la aplicación del lado servidor. A continuación, este token se usa para conectar la aplicación cliente con Azure Communication Services. Una vez recuperado el token, se le pedirá que especifique la cámara y el micrófono que quiere usar. Podrá deshabilitar o habilitar los dispositivos con los controles de alternancia:

:::image type="content" source="./media/calling/pre-call.png" alt-text="Captura de pantalla que muestra la pantalla anterior a la llamada en la aplicación de ejemplo.":::

Una vez que configure el nombre para mostrar y los dispositivos, puede unirse a la sesión de llamada. Ahora verá el lienzo de llamada principal en el que se encuentra la experiencia de llamada principal.

:::image type="content" source="./media/calling/main-app.png" alt-text="Captura de pantalla que muestra la pantalla principal de la aplicación de ejemplo.":::

Componentes de la pantalla principal de llamada:

- **Galería multimedia**: la fase principal en la que se muestran los participantes. Si un participante tiene habilitada la cámara, aquí se muestra su fuente de vídeo. Cada participante tiene un icono individual que muestra su nombre para mostrar y la transmisión de vídeo (si hay alguna).
- **Encabezado**: aquí es donde se encuentran los controles de llamada principales para ajustar la configuración y la barra lateral de participantes, activar o desactivar el vídeo y mezclas, compartir la pantalla y abandonar la llamada.
- **Barra lateral**: aquí es donde se muestran los participantes y la información de configuración cuando se ajustan con los controles del encabezado. El componente se puede descartar con la "X" de la esquina superior derecha. En la barra lateral de participantes se mostrará una lista de participantes y un vínculo para invitar a más usuarios a conversar. La barra lateral de configuración permite configurar las opciones del micrófono y la cámara.

A continuación encontrará más información sobre los requisitos previos, los pasos para configurar el ejemplo y los tutoriales paso a paso que le ayudarán a familiarizarse con los distintos componentes.

## <a name="prerequisites"></a>Requisitos previos

- Cree una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Node.js (12.18.4 y posterior)](https://nodejs.org/en/download/)
- [Visual Studio (2019 y posterior)](https://visualstudio.microsoft.com/vs/)
- [.NET Core 2.2](https://dotnet.microsoft.com/download/dotnet-core/2.2) (asegúrese de instalar la versión que se corresponda con su instancia de Visual Studio, 32 o 64 bits)
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../quickstarts/create-communication-resource.md). Deberá registrar la **cadena de conexión** del recurso para esta guía de inicio rápido.

## <a name="locally-deploy-the-service--client-applications"></a>Implementación local del servicio y las aplicaciones cliente

El ejemplo de llamada grupal consta esencialmente dos aplicaciones: ClientApp y Service.NET.

Cuando queremos realizar una implementación localmente, es necesario iniciar ambas aplicaciones. Cuando se visita la aplicación de servidor desde el explorador, se usa la instancia de ClientApp implementada localmente para la experiencia del usuario.

Para probar el ejemplo localmente, puede abrir varias sesiones del explorador con la dirección URL de su llamada para simular una llamada multiusuario.

### <a name="before-running-the-sample-for-the-first-time"></a>Antes de ejecutar el ejemplo por primera vez

1. Abra una instancia de PowerShell, Terminal Windows, símbolo del sistema o equivalente y navegue hasta el directorio donde le gustaría clonar el ejemplo.
2. `git clone`
3. Vaya a la carpeta **Calling/ClientApp** y ejecute `npm run setup`.
   1. Si ve un error 1, busque más arriba en la salida una dirección URL a la que tendrá que ir para autorizar al cliente. (La dirección URL tendrá el siguiente aspecto: `app.vssps.visualstudio.com/oauth2/authorize?clientid=...`). Una vez que visite la dirección URL en un explorador, copie el comando desde la ventana del explorador y ejecútelo.
   2. Vuelva a ejecutar el comando `npm run setup-vsts-auth` una vez que complete el paso anterior.
4. Obtenga el valor de `Connection String` desde Azure Portal. Para obtener más información sobre las cadenas de conexión, consulte [Creación de un recurso de Azure Communication Services](../quickstarts/create-communication-resource.md).
5. Una vez que obtenga la cadena de conexión, agregue la cadena de conexión al archivo **Calling/appsetting.json** que se encuentra en la carpeta de Service .NET. Escriba la cadena de conexión en la variable: `ResourceConnectionString`.

### <a name="local-run"></a>Ejecución local

1. Vaya a la carpeta Calling.
2. Abra la solución `Calling.csproj` en Visual Studio.
2. Ejecute el proyecto `Calling`.*

*El explorador se abrirá en `localhost:5000` (donde el nodo implementa la aplicación cliente). La aplicación no es compatible con Internet Explorer.

#### <a name="troubleshooting"></a>Solución de problemas

- La solución no se compila; se producen errores durante la instalación o compilación de NPM.

   Intente limpiar o recompilar los proyectos.

## <a name="publish-the-sample-to-azure"></a>Publicación del ejemplo en Azure

1. Haga clic con el botón derecho en el proyecto `Calling` y seleccione Publicar.
2. Cree un nuevo perfil de publicación y seleccione la suscripción a Azure.
3. Antes de publicar, agregue la cadena de conexión con `Edit App Service Settings` y rellene el valor `ResourceConnectionString` como clave y proporcione la cadena de conexión (copiada de appsettings.json) como valor.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- Familiarícese con [el uso de la biblioteca cliente de llamada](../quickstarts/voice-video-calling/calling-client-samples.md)
- Más información sobre las [funcionalidades de la biblioteca cliente de llamada](../quickstarts/voice-video-calling/calling-client-samples.md)
- Más información sobre [cómo funciona la llamada](../concepts/voice-video-calling/about-call-types.md)

## <a name="additional-reading"></a>Lecturas adicionales

- [Versión preliminar de Azure Communication](https://github.com/Azure/communication-preview): para obtener más información sobre el SDK web de llamada
- [Redux](https://redux.js.org/): administración de estado del lado cliente
- [FluentUI](https://developer.microsoft.com/fluentui#/): biblioteca de interfaz de usuario con tecnología de Microsoft
- [React](https://reactjs.org/): biblioteca para compilar interfaces de usuario
- [ASP.NET Core](https://docs.microsoft.com/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-3.1&preserve-view=true): marco para compilar aplicaciones web
