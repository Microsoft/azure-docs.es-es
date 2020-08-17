---
title: API de Azure FarmBeats
description: Aprenda sobre las API de Azure FarmBeats, que proporcionan a las empresas agrícolas una interfaz RESTful estandarizada con respuestas basadas en JSON.
author: sunasing
ms.topic: article
ms.date: 11/04/2019
ms.author: sunasing
ms.openlocfilehash: f15bee7e802b04d04a3c87d7f84fc975b88bf260
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536579"
---
# <a name="azure-farmbeats-apis"></a>API de Azure FarmBeats

En este artículo se describen las API de Azure FarmBeats. Las API de Azure FarmBeats proporcionan a los negocios agrícolas una interfaz RESTful estandarizada con respuestas basadas en JSON para ayudarle a aprovechar las funcionalidades de Azure Farmbeats, como las siguientes:

- API para obtener datos de sensor, de cámara, de dron, del tiempo, de satélite y terrestres mantenidos.
- Normalización y contextualización de datos en proveedores de datos comunes.
- Acceso esquematizado y capacidades de consulta en todos los datos ingeridos.
- Generación automática de metadatos que se pueden consultar, basados en las características agronómicas.
- Agregados de secuencia de tiempo generados automáticamente para la compilación rápida de modelos.
- Motor de Azure Data Factory integrado para compilar fácilmente canalizaciones personalizadas de procesamiento de datos.

## <a name="application-development"></a>Desarrollo de aplicaciones

Las API de FarmBeats contienen documentación técnica de Swagger. Para obtener información sobre las API y sus solicitudes o respuestas correspondientes, vea [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

En la tabla siguiente se resumen todos los objetos y recursos de FarmBeats Datahub:

| Objetos y recursos | Descripción
--- | ---|
Granja | La granja corresponde a una ubicación física de interés dentro del sistema FarmBeats. Cada granja tiene un nombre de granja y un identificador de granja único. |
Dispositivo  | Device se corresponde con un dispositivo físico presente en la granja. Cada dispositivo tiene un id. de dispositivo único. El dispositivo normalmente se aprovisiona a una granja con un identificador de granja.
DeviceModel  | DeviceModel se corresponde con los metadatos del dispositivo, como el fabricante o el tipo de dispositivo, que es puerta de enlace o nodo.
Sensor  | El sensor corresponde a un sensor físico que registra valores. Un sensor normalmente se conecta a un dispositivo con un identificador de dispositivo.
SensorModel  | SensorModel se corresponde con los metadatos del sensor, por ejemplo, el fabricante, el tipo de sensor (analógico o digital) o la medida de sensor (como temperatura ambiente y presión).
Telemetría  | La telemetría proporciona la capacidad de leer los mensajes de telemetría para un determinado intervalo de tiempo y sensor.
Trabajo  | El trabajo corresponde a cualquier flujo de trabajo de actividades, que se ejecutan en el sistema FarmBeats para obtener el resultado pretendido. Cada trabajo está asociado a un id. de trabajo y a un tipo de trabajo.
JobType  | JobType corresponde a distintos tipos de trabajo que el sistema admite. Incluye los tipos de trabajo definidos por el sistema y los definidos por el usuario.
ExtendedType  | ExtendedType corresponde a la lista de tipos definidos por el sistema y por el usuario en el sistema. ExtendedType ayuda a configurar un nuevo tipo de sensor, escena o archivo de escena en el sistema FarmBeats.
Asociado  | El asociado corresponde al asociado de integración de sensor e imágenes para FarmBeats.
Escena  | La escena corresponde a cualquier salida generada en el contexto de una granja. Cada escena tiene un identificador de escena, un origen de la escena, un tipo de escena y un identificador de granja asociados. Cada id. de escena puede tener varios archivos de escenas asociados.
SceneFile |SceneFile corresponde a todos los archivos que se generan para una sola escena. Un único id. de escena puede tener varios id. de SceneFile asociados.
Regla  |La regla corresponde a una condición de datos relacionados con la granja para desencadenar una alerta. Cada regla está en el contexto de los datos de la granja.
Alerta  | La alerta corresponde a una notificación que se genera cuando se cumple una condición de regla. Cada alerta está en el contexto de una regla.
RoleDefinition  | RoleDefinition define las acciones permitidas y no permitidas para un rol.
RoleAssignment  |RoleAssignment se corresponde a la asignación de un rol a un usuario o una entidad de servicio.

### <a name="data-format"></a>Formato de datos

JSON es un formato de datos común independiente del lenguaje que proporciona una representación de texto simple de estructuras de datos arbitrarias. Para más información, consulte el [sitio web de JSON](https://www.json.org/).

## <a name="authentication-and-authorization"></a>Autenticación y autorización

Las solicitudes HTTP a la API REST están protegidas con Azure Active Directory (Azure AD).
Para realizar una solicitud autenticada a las API REST, el código de cliente requiere autenticación con credenciales válidas para poder llamar a la API. La autenticación se coordina entre los distintos actores mediante Azure AD. Proporciona al cliente un token de acceso como prueba de la autenticación. Después, el token se envía en el encabezado de autorización HTTP de las solicitudes de la API REST. Para obtener más información sobre la autenticación de Azure AD, vea [Azure Active Directory](https://portal.azure.com) para desarrolladores.

El token de acceso se debe enviar en las solicitudes de API posteriores en la sección de encabezado como:

```http
headers = {"Authorization": "Bearer " + **access_token**}
```

### <a name="http-request-headers"></a>Encabezados de solicitud HTTP

Estos son los encabezados de solicitud más comunes que deben especificarse al realizar una llamada API al centro de datos de Azure FarmBeats.


**Encabezado** | **Descripción y ejemplo**
--- | ---
Content-Type  | El formato de la solicitud (Content-Type: application/<format>). En el caso de las API de centro de datos de Azure FarmBeats, el formato es JSON. Content-Type: application/json
Authorization  | Especifica el token de acceso necesario para realizar una llamada API. Autorización: Bearer <Access-Token>
Accept | El formato de respuesta. En el caso de las API de centro de datos de Azure FarmBeats, el formato es JSON. Accept: application/json

### <a name="api-requests"></a>Solicitudes de API

Para realizar una solicitud de API REST, se combina el método HTTP (GET, POST, PUT o DELETE), la dirección URL del servicio de API, el URI a un recurso para consultar, enviar datos, actualizar o eliminar, y, a continuación, se agregan uno o más encabezados de solicitud HTTP.

La dirección URL al servicio de API es la dirección URL del centro de datos, por ejemplo, https://\<yourdatahub-website-name>.azurewebsites.net.

También se pueden incluir parámetros de consulta en las llamadas GET para filtrar y ordenar los datos en las respuestas y limitar su tamaño.

La siguiente solicitud de ejemplo se usa para obtener la lista de dispositivos:

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>”
```

La mayoría de llamadas GET, POST y PUT requieren un cuerpo de la solicitud JSON.

La siguiente solicitud de ejemplo crea un dispositivo. Esta solicitud tiene un código JSON de entrada con el cuerpo de la solicitud.

```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H  "accept: application/json" -H  "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,  \"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

### <a name="query-parameters"></a>Parámetros de consulta

En el caso de las llamadas GET de REST, se pueden filtrar, limitar el tamaño y ordenar los datos en una respuesta de API al incluir uno o varios parámetros de consulta en el URI de solicitud. En el caso de los parámetros de consulta, vea la documentación de la API y las llamadas GET individuales.
Por ejemplo, al consultar la lista de dispositivos (llamada GET en /Device), se pueden especificar los siguientes parámetros de consulta:

![Lista de dispositivos](./media/references-for-azure-farmbeats/query-parameters-device-1.png)

### <a name="error-handling"></a>Control de errores

Las API del centro de datos de Azure FarmBeats devuelven los errores HTTP estándar. Los códigos de error más comunes son los siguientes:

 |Código de error             | Descripción |
 |---                    | --- |
 |200                    | Correcto |
 |201                    | Creación (publicación) correcta. |
 |400                    | Solicitud incorrecta. Hay un error en la solicitud. |
 |401                    | No autorizado. El autor de la llamada de la API no está autorizado para acceder al recurso. |
 |404                    | Recurso no encontrado. |
 |5XX                    | Error interno del servidor. Los códigos de error que empiezan por 5XX significan que hay algún error en el servidor. Consulte los registros de servidor y la sección siguiente para obtener más detalles. |


Además de los errores HTTP estándar, las API del centro de datos de Azure FarmBeats también devuelven errores internos en el siguiente formato:

```json
    {
      "message": "<More information on the error>",
      "status": "<error code>”,
      "code": "<InternalErrorCode>",
      "moreInfo": "<Details of the error>"
    }
```

En este ejemplo, cuando se creó una granja, el campo obligatorio "Name" no se especificó en la carga de entrada. El mensaje de error resultante sería el siguiente:

 ```json    
    {
      "message": "Model validation failed",
      "status": 400,
      "code": "ModelValidationFailed",
      "moreInfo": "[\"The Name field is required.\"]"
    }
  ```

## <a name="add-users-or-app-registrations-to-azure-active-directory"></a>Incorporación de usuarios o registros de aplicaciones a Azure Active Directory

Se puede acceder a las API de Azure FarmBeats mediante un usuario o un registro de aplicación en Azure Active Directory. Para crear un registro de aplicación en Azure Active Directory, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com) y seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**. También puede usar una cuenta existente.
2. Para una cuenta nueva, haga lo siguiente:

    - Escriba un nombre.
    - Seleccione **Cuentas de este directorio organizativo solo (inquilino único)** .
    - Use los valores predeterminados en el resto de campos.
    - Seleccione **Registrar**.

3. En el registro de aplicación nuevo o existente, panel **Información general**, haga lo siguiente:

    - Capture el **Id. de cliente** y el **Id. de inquilino**.
    - Vaya a **Certificados y secretos** para generar un secreto de cliente nuevo y capturar el **Secreto de cliente**.
    - Vuelva a **Información general** y seleccione el vínculo situado junto a **Administrar la aplicación en el directorio local**.
    - Vaya a **Propiedades** para capturar el **Id. de objeto**.

4. Vaya a la instancia de Datahub Swagger (https://<yourdatahub>.azurewebsites.net/swagger/index.html) y haga lo siguiente:
    - Vaya a **RoleAssignment API**.
    - Realice una operación POST con el fin de crear un objeto **RoleAssignment** para el **Id. de objeto** que se acaba de crear.
 
```json
{
  "roleDefinitionId": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
  "objectId": "objectId from step 3 above",
  "objectIdType": "ServicePrincipalId",
  "tenantId": "tenant id of your Azure subscription"
}
```

  > [!NOTE]
  > Para más información sobre cómo agregar usuarios y registro de Active Directory, vea [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).

Después de completar los pasos anteriores, el registro de aplicación (cliente) puede llamar a las API de Azure FarmBeats mediante un token de acceso a través de la autenticación de portador.

Use el token de acceso para enviarlo en las solicitudes de API posteriores en la sección de encabezado de la siguiente manera:

```http
headers = {"Authorization": "Bearer " + **access_token**, "Content-Type" : "application/json" }
```
