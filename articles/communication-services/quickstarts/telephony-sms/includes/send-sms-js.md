---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: dademath
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 07/28/2020
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: cdd4988f9a23904c0771852c4539aa9bce2ee683
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946003"
---
Introducción a Azure Communication Services mediante la biblioteca cliente de SMS de JavaScript de Communication Services para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-js-pr/tree/feature/communication/sdk/communication/communication-sms) | [Package (NPM)](https://www.npmjs.com/package/@azure/communication-sms) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Recurso activo de Communication Services y una cadena de conexión. [Creación de un recurso de Communication Services](../../create-communication-resource.md).
- Número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `node --version` para comprobar que Node.js está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una nueva aplicación Node.js

En primer lugar, abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

Use un editor de texto para crear un archivo denominado **send-sms.js** en el directorio raíz del proyecto. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar la biblioteca cliente de SMS de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-sms --save
```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente de SMS de Azure Communication Services para Node.js.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta clase con la información de suscripción y úsela para enviar mensajes de texto. |
| SendSmsOptions | Esta interfaz proporciona opciones para configurar los informes de entrega. Si `enable_delivery_report` se establece en `true`, se emitirá un evento cuando la entrega se realice correctamente. |
| SendMessageRequest | Esta interfaz es el modelo para la creación de la solicitud de SMS (por ejemplo, configurar los números de teléfono de origen y destino y el contenido del SMS). |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Importe la clase **SmsClient** desde la biblioteca cliente y cree una instancia con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

Agregue el siguiente código a **send-sms.js**:

```javascript
const { SmsClient } = require('@azure/communication-sms');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the SMS client
const smsClient = new SmsClient(connectionString);
```

## <a name="send-an-sms-message"></a>Enviar un mensaje SMS

Envíe un mensaje de texto con una llamada al método `send`. Agregue este código al final de **send-sms.js**:

```javascript
await smsClient.send({
  from: "<leased-phone-number>",
  to: ["<to-phone-number>"],
  message: "Hello World 👋🏻 via Sms"
}, {
  enableDeliveryReport: true //Optional parameter
});
```

Debe reemplazar `<leased-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number>` por el número de teléfono al que desea enviar un mensaje. Todos los parámetros de número de teléfono deben cumplir el [estándar E.164](../../../concepts/telephony-sms/plan-solution.md#optional-reading-international-public-telecommunication-numbering-plan-e164).

El parámetro `enableDeliveryReport` es un parámetro opcional que puede usar para configurar los informes de entrega. Esto resulta útil para aquellos escenarios en los que desee emitir eventos cuando se entreguen mensajes SMS. Consulte la guía de inicio rápido [Controlar eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

## <a name="run-the-code"></a>Ejecución del código

Use el comando `node` para ejecutar el código que agregó al archivo **send-sms.js**.

```console

node ./send-sms.js

```