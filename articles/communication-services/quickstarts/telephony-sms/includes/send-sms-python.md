---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: danieldoolabh
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 09/03/2020
ms.topic: include
ms.custom: include file
ms.author: dadoolab
ms.openlocfilehash: 3fe27bf31385e0310211b5e1b0b7bbfa636eb19b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946019"
---
Introducción a Azure Communication Services mediante la biblioteca cliente de SMS de Python de Communication Services para enviar mensajes de texto.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](#todo-sdk-repo) | [Package (PiPy)](#todo-nuget) | [Samples](#todo-samples)--> 

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- [Python](https://www.python.org/downloads/) 2.7, 3.5 o versiones posteriores
- Recurso activo de Communication Services y una cadena de conexión. [Creación de un recurso de Communication Services](../../create-communication-resource.md).
- Número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `python --version` para comprobar que Python está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o el terminal, cree un nuevo directorio para la aplicación y navegue hasta él.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Use un editor de texto para crear un archivo denominado **send-sms.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

```python
import os
from azure.communication.sms import PhoneNumber
from azure.communication.sms import SendSmsOptions
from azure.communication.sms import SmsClient

try:
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-the-package"></a>Instalar el paquete

En el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Communication Services para Python con el comando `pip install`.

```console
pip install azure-communication-sms
```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente de SMS de Azure Communication Services para Python.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta clase con la información de suscripción y úsela para enviar mensajes de texto. |
| SendSmsOptions | Esta clase proporciona opciones para configurar los informes de entrega. Si enable_delivery_report se establece en True, se emitirá un evento cuando la entrega se realice correctamente |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de la clase **SmsClient** con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

```python
# This code demonstrates how to fetch your connection string
# from an environment variable.
connection_string = os.getenv('COMMUNICATION_SERVICES_CONNECTION_STRING')

# Create the SmsClient object which will be used to send SMS messages
sms_client = SmsClient.from_connection_string(connection_string)
```

## <a name="send-an-sms-message"></a>Enviar un mensaje SMS

Envíe un mensaje de texto con una llamada al método Enviar. Agregue este código al final del bloque `try` en **send-sms.py**:

```python

# calling send() with sms values
sms_response = sms_client.send(
        from_phone_number=PhoneNumber("<leased-phone-number>"),
        to_phone_numbers=[PhoneNumber("<to-phone-number>")],
        message="Hello World via SMS",
        send_sms_options=SendSmsOptions(enable_delivery_report=True)) # optional property

```

Debe reemplazar `<leased-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de servicio de comunicación y `<to-phone-number>` por el número de teléfono al que desea enviar un mensaje. Todos los parámetros de número de teléfono deben cumplir el [estándar E.164](../../../concepts/telephony-sms/plan-solution.md#optional-reading-international-public-telecommunication-numbering-plan-e164).

El parámetro `send_sms_options` es un parámetro opcional que puede usar para configurar los informes de entrega. Esto resulta útil para aquellos escenarios en los que desee emitir eventos cuando se entreguen mensajes SMS. Consulte la guía de inicio rápido [Controlar eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `python`.

```console
python send-sms.py
```
