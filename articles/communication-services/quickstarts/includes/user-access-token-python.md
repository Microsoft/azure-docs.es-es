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
ms.openlocfilehash: 4be8821a949527fefcc9005b1de7f4f7c438c568
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945747"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7, 3.5 o versiones posteriores
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

   ```console
   mkdir user-tokens-quickstart && cd user-tokens-quickstart
   ```

1. Use un editor de texto para crear un archivo denominado **issue-tokens.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

   ```python
   import os
   from azure.communication.administration import CommunicationIdentityClient

   try:
      print('Azure Communication Services - User Access Tokens Quickstart')
      # Quickstart code goes here
   except Exception as ex:
      print('Exception:')
      print(ex)
   ```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de administración de Azure Communication Services para Python con el comando `pip install`.

```console
pip install azure-communication-administration
```

[!INCLUDE [User Access Tokens Object Model](user-access-tokens-object-model.md)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue este código dentro del bloque `try`:

```python
# This code demonstrates how to fetch your connection string
# from an environment variable.
connection_string = os.environ['COMMUNICATION_SERVICES_CONNECTION_STRING']

# Instantiate the identity client
client = CommunicationIdentityClient.from_connection_string(connection_string)
```

## <a name="create-a-user"></a>Creación de un usuario

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `create_user` para crear una nueva entrada en el directorio con un `Id` único. Debe mantener una asignación entre los usuarios de la aplicación y las identidades generadas por Communication Services (por ejemplo, se pueden almacenar en la base de datos del servidor de aplicaciones).

```python
user = client.create_user()
print("\nCreated a user with ID: " + user.identifier + ":")
```

## <a name="issue-user-access-tokens"></a>Emisión de tokens de acceso de usuario

Use el método `issue_token` para emitir un token de acceso para un usuario de Communication Services. Si no proporciona el parámetro `user` opcional, se creará un nuevo usuario y se devolverá con el token.

```python
# Issue an access token with the "voip" scope for a new user
token_result = client.issue_token(user, ["voip"])
expires_on = token_result.expires_on.strftime('%d/%m/%y %I:%M %S %p')
print("\nIssued a token with 'voip' scope that expires at " + expires_on + ":")
print(token_result.token)
```

Los tokens de acceso de usuario son credenciales de corta duración que deben volver a emitirse para evitar que los usuarios experimenten interrupciones del servicio. La propiedad de respuesta `expires_on` indica la duración del token.

## <a name="revoke-user-access-tokens"></a>Revocación de tokens de acceso de usuario

En algunos casos, puede que necesite revocar explícitamente los tokens de acceso de usuario, por ejemplo, si un usuario cambia la contraseña que usa para autenticarse en el servicio. Esta funcionalidad está disponible a través de la biblioteca cliente de administración de Azure Communication Services.

```python  
client.revoke_tokens(user)
print("\nSuccessfully revoked all tokens for user with ID: " + user.identifier)
```

## <a name="delete-a-user"></a>Eliminar un usuario

La eliminación de una identidad revoca todos los tokens activos e impide que se emitan tokens posteriores para las identidades. También quita todo el contenido conservado asociado al usuario.

```python
client.delete_user(user)
print("\nDeleted the user with ID: " + user.identifier)
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, navegue hasta el directorio que contiene el archivo *issue-token.py* y, a continuación, ejecute el comando `python` siguiente para ejecutar la aplicación.

```console
python ./issue-token.py
```
