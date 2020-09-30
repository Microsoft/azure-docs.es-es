---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: matthewrobertson
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: marobert
ms.openlocfilehash: 1235cb299fe887f20dd3f7e47c22eed2b9df6dc9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945738"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable&preserve-view=true), versión 8 o posterior.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Un recurso de Communication Services implementado y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Abra la ventana de terminal o de comandos y navegue hasta el directorio en el que quiere crear la aplicación de Java. Ejecute el siguiente comando para generar el proyecto de Java a partir de la plantilla maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Observará que la tarea "generar" creó un directorio con el mismo nombre que el objeto `artifactId`. En este directorio, el directorio src/main/java contiene el código fuente del proyecto, el directorio `src/test/java directory` contiene el origen de la prueba y el archivo `pom.xml` es el modelo de objetos del proyecto o POM.

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo **pom.xml** en el editor de texto. Agregue el siguiente elemento de dependencia al grupo de dependencias.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-administration</artifactId>
    <version>1.0.0-beta.1</version> 
</dependency>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Vaya al directorio */src/main/java/com/communication/quickstart*.
1. Abra el archivo *App.java* en el editor.
1. Reemplace la instrucción `System.out.println("Hello world!");`.
1. Agregue directivas `import`.

Use el código siguiente para empezar:

```java
import com.azure.communication.common.CommunicationUser;
import com.azure.communication.administration.models.CommunicationIdentityToken;
import com.azure.communication.administration.CommunicationIdentityClient;
import com.azure.communication.administration.CommunicationIdentityClientBuilder;
import java.io.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - User Access Tokens Quickstart");
        // Quickstart code goes here
    }
}
```

[!INCLUDE [User Access Tokens Object Model](user-access-tokens-object-model.md)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de `CommunicationIdentityClient` con la clave de acceso y el punto de conexión del recurso. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `main`:

```java
// Your can find your endpoint and access token from your resource in the Azure Portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
String accessToken = "SECRET";

// Create an HttpClient builder of your choice and customize it
// Use com.azure.core.http.netty.NettyAsyncHttpClientBuilder if that suits your needs
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
    .endpoint(endpoint)
    .credential(new CommunicationClientCredential(accessToken))
    .httpClient(httpClient)
    .buildClient();
```

Puede inicializar el cliente con cualquier cliente HTTP personalizado que implemente la interfaz de `com.azure.core.http.HttpClient`. En el código anterior se muestra el uso del [cliente HTTP Netty de Azure Core](https://docs.microsoft.com/java/api/overview/azure/core-http-netty-readme?view=azure-java-stable&preserve-view=true) proporcionado por `azure-core`.

## <a name="create-a-user"></a>Creación de un usuario

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Debe mantener una asignación entre los usuarios de la aplicación y las identidades generadas por Communication Services (por ejemplo, se pueden almacenar en la base de datos del servidor de aplicaciones).

```java
CommunicationUser user = communicationIdentityClient.createUser();
System.out.println("\nCreated a user with ID: " + user.getId());
```

## <a name="issue-user-access-tokens"></a>Emisión de tokens de acceso de usuario

Use el método `issueToken` para emitir un token de acceso para un usuario de Communication Services. Si no proporciona el parámetro `user` opcional, se creará un nuevo usuario y se devolverá con el token.

```java
// Issue an access token with the "voip" scope for a new user
List<String> scopes = new ArrayList<>(Arrays.asList("voip"));
CommunicationUserToken response = communicationIdentityClient.issueToken(user, scopes);
OffsetDateTime expiresOn = response.getExpiresOn();
String token = response.getToken();
String userId = response.getUser().getId();
System.out.println("\nIssued a access token with 'voip' scope for identity with ID: " + userId + ": " + token);
System.out.println(token);
```

Los tokens de acceso de usuario son credenciales de corta duración que deben volver a emitirse para evitar que los usuarios experimenten interrupciones del servicio. La propiedad de respuesta `expiresAt` indica la duración del token.

## <a name="revoke-user-access-tokens"></a>Revocación de tokens de acceso de usuario

En algunos casos, puede que necesite revocar explícitamente los tokens de acceso de usuario, por ejemplo, si un usuario cambia la contraseña que usa para autenticarse en el servicio. Se usa el método `revokeTokens` para invalidar todos los tokens de acceso de un usuario.

```java  
communicationIdentityClient.revokeTokens(user, OffsetDateTime.now());
System.out.println("\nRevoked tokens for the user with ID: " + user.getId());
```

## <a name="delete-a-user"></a>Eliminar un usuario

La eliminación de un usuario revoca todos los tokens activos e impide que se emitan tokens posteriores para las identidades. También quita todo el contenido conservado asociado al usuario.

```java
communicationIdentityClient.deleteUser(user);
System.out.println("\nSuccessfully deleted the identity with ID: " + user.getId());
```

## <a name="run-the-code"></a>Ejecución del código

Navegue hasta el directorio que contiene el archivo *pom.xml* y compile el proyecto mediante el siguiente comando `mvn`.

```console
mvn compile
```

A continuación, compile el paquete.

```console
mvn package
```

Ejecute el siguiente comando `mvn` para ejecutar la aplicación.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```
