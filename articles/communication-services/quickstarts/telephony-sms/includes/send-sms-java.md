---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: chrwhit
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: chrwhit
ms.openlocfilehash: 72bb38b7d9d28366fb75c5656832a1f37c770fd2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945994"
---
Para empezar a usar Azure Communication Services, utilice la biblioteca cliente de SMS de Java de Communication Services para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-net-pr/tree/feature/communication/sdk/communication/Azure.Communication.Sms#todo-update-to-public) | [Artifact (Maven)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable&preserve-view=true), versión 8 o posterior.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `mvn -v` para comprobar que Maven está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Abra la ventana de terminal o de comandos y navegue hasta el directorio en el que quiere crear la aplicación de Java. Ejecute el siguiente comando para generar el proyecto de Java a partir de la plantilla maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

El objetivo "generar" creó un directorio con el mismo nombre que el elemento artifactId. En este directorio, el directorio **src/main/java** contiene el código fuente del proyecto, el **directorio src/test/java** contiene el origen de la prueba, y el archivo **pom.xml** es el modelo de objetos del proyecto o POM.

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo **pom.xml** en el editor de texto. Agregue el siguiente elemento de dependencia al grupo de dependencias.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0-beta.1</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Agregue la dependencia `azure-core-http-netty` en su archivo **pom.xml**:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core-http-netty</artifactId>
    <version>1.3.0</version>
</dependency>
```

Abra **/src/main/java/com/communication/quickstart/App.java** en un editor de texto, agregue directivas de importación y quite la instrucción `System.out.println("Hello world!");`:

```java
package com.communication.quickstart;

import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;

import com.azure.communication.common.CommunicationClientCredential;
import com.azure.communication.common.PhoneNumber;
import com.azure.communication.sms.SmsClient;
import com.azure.communication.sms.SmsClientBuilder;
import com.azure.communication.sms.models.SendSmsOptions;
import com.azure.communication.sms.models.SendSmsResponse;
import com.azure.core.http.HttpClient;
import com.azure.core.http.netty.NettyAsyncHttpClientBuilder;

public class App
{
    public static void main( String[] args ) throws IOException, NoSuchAlgorithmException, InvalidKeyException
    {
        // Quickstart code goes here
    }
}

```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente de SMS de Azure Communication Services para Java.

| Nombre                                                             | Descripción                                                                                     |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| SmsClientBuilder              | Esta clase crea SmsClient. Se proporciona con el punto de conexión, la credencial y un cliente HTTP. |
| SmsClient                    | Esta clase es necesaria para la funcionalidad de los SMS, que se utiliza para enviar mensajes SMS.                |
| SendSmsResponse               | Esta clase contiene la respuesta del servicio SMS.                                          |
| CommunicationClientCredential | Esta clase controla las solicitudes de firmas.                                                            |
| PhoneNumber                   | Esta clase contiene información sobre el número de teléfono.

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de `SmsClient` con su cadena de conexión. El código siguiente recupera el punto de conexión y las cadenas de credenciales del recurso de las variables de entorno denominadas `COMMUNICATION_SERVICES_ENDPOINT_STRING` y `COMMUNICATION_SERVICES_CREDENTIAL_STRING` (credencial hace referencia a `Key` de Azure Portal. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `main`:

```java
// Create an HttpClient builder of your choice and customize it
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

CommunicationClientCredential credential = new CommunicationClientCredential(accessKey);

// Configure and build a new SmsClient
SmsClient client = new SmsClientBuilder()
    .endpoint(endpoint)
    .credential(credential)
    .httpClient(httpClient)
    .buildClient();
```

Puede inicializar el cliente con cualquier cliente HTTP personalizado que implemente la interfaz `com.azure.core.http.HttpClient`. En el código anterior se muestra el uso del [cliente HTTP Netty de Azure Core](https://docs.microsoft.com/java/api/overview/azure/core-http-netty-readme?view=azure-java-stable&preserve-view=true) proporcionado por `azure-core`.

## <a name="send-an-sms-message"></a>Enviar un mensaje SMS

Envíe un mensaje SMS con una llamada al método sendMessage. Agregue este código al final del método `main`:

```java
List<PhoneNumber> to = new ArrayList<PhoneNumber>();
to.add(new PhoneNumber("<to-phone-number>"));

// SendSmsOptions is an optional field. It can be used
// to enable a delivery report to the Azure Event Grid
SendSmsOptions options = new SendSmsOptions();
options.setEnableDeliveryReport(true);

// Send the message and check the response for a message id
SendSmsResponse response = client.sendMessage(
    new PhoneNumber("<leased-phone-number>"),
    to,
    "<message-text>",
    options);

System.out.println("MessageId: " + response.getMessageId());
```

Debe reemplazar `<leased-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number>` por el número de teléfono al que quiere enviar un mensaje. Todos los parámetros de número de teléfono deben cumplir el [estándar E.164](../../../concepts/telephony-sms/plan-solution.md#optional-reading-international-public-telecommunication-numbering-plan-e164).

El parámetro `enableDeliveryReport` es un parámetro opcional que puede usar para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

<!--todo: the signature of the `sendMessage` method changes when configuring delivery reporting. Need to confirm that this is how our client library is to be used.-->

## <a name="run-the-code"></a>Ejecución del código

Navegue hasta el directorio que contiene el archivo **pom.xml** y compile el proyecto mediante el siguiente comando `mvn`.

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
