---
title: Uso de las API REST de Azure Blockchain Workbench
description: Escenarios de uso de la API REST de Azure Blockchain Workbench, versión preliminar
ms.date: 03/05/2020
ms.topic: how-to
ms.reviewer: brendal
ms.openlocfilehash: 696f1f2f96034f7a044f6a39182774c02804518f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87003073"
---
# <a name="using-the-azure-blockchain-workbench-preview-rest-api"></a>Uso de la API REST de Azure Blockchain Workbench, versión preliminar

La API REST de Azure Blockchain Workbench, versión preliminar, proporciona a los desarrolladores y trabajadores de la información una forma de crear integraciones enriquecidas para aplicaciones de cadenas de bloques. En este artículo se resaltan varios escenarios de uso de la API REST de Workbench. Por ejemplo, supongamos que quiere crear un cliente de cadena de bloques personalizado que permite a los usuarios que han iniciado sesión ver las aplicaciones de cadena de bloques asignadas e interactuar con ellas. El cliente puede usar la API de Blockchain Workbench para ver las instancias del contrato y tomar acciones en los contratos inteligentes.

## <a name="blockchain-workbench-api-endpoint"></a>Punto de conexión de API de Blockchain Workbench

Se accede a las API de Blockchain Workbench mediante un punto de conexión de la implementación. Para obtener la dirección URL del punto de conexión de API de la implementación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.
1. Elija el nombre del grupo de recursos en el que implementó Blockchain Workbench.
1. Seleccione el encabezado de columna **TIPO** para ordenar la lista alfabéticamente por tipo.
1. Hay dos recursos del tipo **App Service**. Seleccione un recurso del tipo **App Service** *con* el sufijo "-api".
1. En **Información general** de App Service, copie el valor **URL** que representa la dirección URL del punto de conexión de API a la instancia de Blockchain Workbench implementada.

    ![Dirección URL del punto de conexión de API de App Service](media/use-api/app-service-api.png)

## <a name="authentication"></a>Authentication

Las solicitudes a la API REST de Blockchain Workbench están protegidas con Azure Active Directory (Azure AD).

Para realizar una solicitud autenticada a las API REST, el código de cliente requiere autenticación con credenciales válidas para poder llamar a la API. La autenticación la coordina Azure AD entre los distintos actores y proporciona al cliente un [token de acceso](../../active-directory/develop/developer-glossary.md#access-token) como prueba de la autenticación. Después, el token se envía en el encabezado de autorización HTTP de las solicitudes de la API REST. Para más información sobre la autenticación de Azure AD, consulte [Azure Active Directory para desarrolladores](../../active-directory/develop/index.yml).

Consulte [Ejemplos de la API REST](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples) para obtener ejemplos sobre cómo realizar la autenticación.

## <a name="using-postman"></a>Uso de Postman

Si desea probar o experimentar con las API de Workbench, puede usar [Postman](https://www.postman.com) para realizar llamadas de API a su implementación. [Descargue una colección de Postman de ejemplo de solicitudes de API de Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples/postman) desde GitHub. Consulte el archivo Léame para más información sobre la autenticación y el uso de las solicitudes de API de ejemplo.

## <a name="create-an-application"></a>Crear una aplicación

Se usan dos llamadas de API para crear una aplicación de Blockchain Workbench. Este método solo lo pueden realizar los usuarios que son administradores de Workbench.

Use la [API POST de aplicaciones](/rest/api/azure-blockchain-workbench/applications/applicationspost) para cargar el archivo JSON de la aplicación y obtener un identificador de aplicación.

### <a name="applications-post-request"></a>Solicitud POST de aplicaciones

Use el parámetro **appFile** para enviar el archivo de configuración como parte del cuerpo de la solicitud.

``` http
POST /api/v1/applications
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="appFile"; filename="/C:/smart-contract-samples/HelloWorld.json"
Content-Type: application/json
```

### <a name="applications-post-response"></a>Respuesta POST de aplicaciones

El identificador de aplicación creado se devuelve en la respuesta. Necesitará el identificador de aplicación para asociar el archivo de configuración con el archivo de código cuando llame a la siguiente API.

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
1
```

### <a name="contract-code-post-request"></a>Solicitud POST de código de contrato

Use la [API POST de código de contrato de aplicaciones](/rest/api/azure-blockchain-workbench/applications/contractcodepost) pasando el identificador de aplicación para cargar el archivo de código de Solidity de la aplicación. La carga puede ser un archivo de Solidity único o un archivo comprimido que contenga archivos de Solidity.

Reemplace los siguientes valores:

| Parámetro | Value |
|-----------|-------|
| {applicationId} | Valor devuelto por la API POST de aplicaciones. |
| {ledgerId} | Índice del libro de contabilidad. Este valor suele ser 1. También puede consultar la [Tabla de libros de contabilidad](data-sql-management-studio.md) para obtener el valor. |

``` http
POST /api/v1/applications/{applicationId}/contractCode?ledgerId={ledgerId}
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="contractFile"; filename="/C:/smart-contract-samples/HelloWorld.sol"
```

### <a name="contract-code-post-response"></a>Respuesta POST de código de contrato

Si se realiza correctamente, la respuesta incluye el identificador de código de contrato creado de la [tabla ContractCode](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
2
```

## <a name="assign-roles-to-users"></a>Asignación de roles a usuarios

Use la [API POST de asignación de roles de aplicaciones](/rest/api/azure-blockchain-workbench/applications/roleassignmentspost) pasando el identificador de aplicación, el identificador de usuario y el identificador del rol de aplicación para crear una asignación de usuario a rol en la aplicación de cadena de bloques especificada. Este método solo lo pueden realizar los usuarios que son administradores de Workbench.

### <a name="role-assignments-post-request"></a>Solicitud POST de asignación de roles

Reemplace los siguientes valores:

| Parámetro | Value |
|-----------|-------|
| {applicationId} | Valor devuelto por la API POST de aplicaciones. |
| {userId} | Valor del identificador de usuario de la [tabla User](data-sql-management-studio.md). |
| {applicationRoleId} | Valor del identificador de rol de aplicación asociado al identificador de aplicación de la [tabla ApplicationRole](data-sql-management-studio.md). |

``` http
POST /api/v1/applications/{applicationId}/roleAssignments
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "userId": {userId},
  "applicationRoleId": {applicationRoleId}
}
```

### <a name="role-assignments-post-response"></a>Respuesta POST de asignación de roles

Si se realiza correctamente, la respuesta incluye el identificador de asignación de roles creado de la [tabla RoleAssignment](data-sql-management-studio.md).

``` http
HTTP/1.1 200
1
```

## <a name="list-applications"></a>Enumerar aplicaciones

Use la [API GET de aplicaciones](/rest/api/azure-blockchain-workbench/applications/applicationsget) para recuperar todas las aplicaciones de Blockchain Workbench del usuario. En este ejemplo, el usuario con sesión iniciada tiene acceso a dos aplicaciones:

- [Asset Transfer](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer/readme.md)
- [Refrigerated Transportation](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/refrigerated-transportation/readme.md)

### <a name="applications-get-request"></a>Solicitud GET de aplicaciones

``` http
GET /api/v1/applications
Authorization : Bearer {access token}
```

### <a name="applications-get-response"></a>Respuesta GET de aplicaciones

La respuesta enumera todas las aplicaciones de cadena de bloques a las que un usuario tiene acceso en Blockchain Workbench. Los administradores de Blockchain Workbench obtienen todas las aplicaciones de cadena de bloques. Sin embargo, los que no son administradores de Workbench obtienen todas las aplicaciones de cadena de bloques para las que tienen asociado al menos un rol de aplicación o un rol de instancia de contrato inteligente.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications?skip=2",
    "applications": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
            "displayName": "Asset Transfer",
            "createdByUserId": 1,
            "createdDtTm": "2018-04-28T05:59:14.4733333",
            "enabled": true,
            "applicationRoles": null
        },
        {
            "id": 2,
            "name": "RefrigeratedTransportation",
            "description": "Application to track end-to-end transportation of perishable goods.",
            "displayName": "Refrigerated Transportation",
            "createdByUserId": 7,
            "createdDtTm": "2018-04-28T18:25:38.71",
            "enabled": true,
            "applicationRoles": null
        }
    ]
}
```

## <a name="list-workflows-for-an-application"></a>Enumerar flujos de trabajo para una aplicación

Use la [API GET de flujos de trabajo de aplicaciones](/rest/api/azure-blockchain-workbench/applications/workflowsget) para enumerar todos los flujos de trabajo de una aplicación de cadena de bloques especificada a la que un usuario tiene acceso en Blockchain Workbench. Cada aplicación de cadena de bloques tiene uno o varios flujos de trabajo y cada flujo de trabajo tiene cero o instancias de contrato inteligente. Para una aplicación de cliente de cadena de bloques que tiene un solo flujo de trabajo, se recomienda omitir el flujo de la experiencia de usuario que permite a los usuarios seleccionar el flujo de trabajo adecuado.

### <a name="application-workflows-request"></a>Solicitud de flujos de trabajo de aplicaciones

``` http
GET /api/v1/applications/{applicationId}/workflows
Authorization: Bearer {access token}
```

### <a name="application-workflows-response"></a>Respuesta de flujos de trabajo de aplicaciones

Los administradores de Blockchain Workbench obtienen todos los flujos de trabajo de cadena de bloques. Sin embargo, los administradores que no son de Workbench obtienen todas las cadenas de bloque para las que tienen asociado al menos un rol de aplicación o un rol de instancia de contrato inteligente.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications/1/workflows?skip=1",
    "workflows": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Handles the business logic for the asset transfer scenario",
            "displayName": "Asset Transfer",
            "applicationId": 1,
            "constructorId": 1,
            "startStateId": 1
        }
    ]
}
```

## <a name="create-a-contract-instance"></a>Creación de una instancia de contrato

Use la [API POST de contratos V2](/rest/api/azure-blockchain-workbench/contractsv2/contractpost) para crear una nueva instancia de contrato inteligente para un flujo de trabajo. Los usuarios solo pueden crear una nueva instancia de contrato inteligente si el usuario está asociado a un rol de aplicación que puede iniciar una instancia de contrato inteligente para el flujo de trabajo.

> [!NOTE]
> En este ejemplo, se usa la versión 2 de la API. Las API de contratos versión 2 proporcionan más granularidad para los campos ProvisioningStatus asociados.

### <a name="contracts-post-request"></a>Solicitud POST de contratos

Reemplace los siguientes valores:

| Parámetro | Value |
|-----------|-------|
| {workflowId} | El valor del identificador de flujo de trabajo es el valor de ConstructorID del contrato de la [tabla Workflow](data-sql-management-studio.md). |
| {contractCodeId} | Valor del identificador del código de contrato de la [tabla ContractCode](data-sql-management-studio.md). Debe correlacionar el identificador de aplicación y el identificador de libro de contabilidad para la instancia de contrato que desea crear. |
| {connectionId} | Valor del identificador de conexión de la [tabla Connection](data-sql-management-studio.md). |

En el cuerpo de la solicitud, establezca los valores con la siguiente información:

| Parámetro | Value |
|-----------|-------|
| workflowFunctionID | Identificador de la [tabla WorkflowFunction](data-sql-management-studio.md). |
| workflowActionParameters | Pares de nombre y valor de los parámetros pasados al constructor. Para cada parámetro, use el valor de workflowFunctionParameterID de la tabla [WorkflowFunctionParameter](data-sql-management-studio.md). |

``` http
POST /api/v2/contracts?workflowId={workflowId}&contractCodeId={contractCodeId}&connectionId={connectionId}
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "workflowFunctionID": 2,
  "workflowActionParameters": [
    {
      "name": "message",
      "value": "Hello, world!",
      "workflowFunctionParameterId": 3
    }
  ]
}
```

### <a name="contracts-post-response"></a>Respuesta POST de contratos

Si se realiza correctamente, la API de asignaciones de roles devuelve el valor de ContractActionID de la [tabla ContractActionParameter](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
4
```

## <a name="list-smart-contract-instances-for-a-workflow"></a>Enumerar las instancias de contrato inteligente para un flujo de trabajo

Use la [API GET de contratos](/rest/api/azure-blockchain-workbench/contractsv2/contractsget) para mostrar todas las instancias de contrato inteligente de un flujo de trabajo. O bien, puede permitir a los usuarios profundizar en cualquiera de las instancias de contrato inteligente que se muestran.

### <a name="contracts-request"></a>Solicitud de contratos

En este ejemplo, considere la posibilidad de que un usuario desee interactuar con una de las instancias de contrato inteligente para tomar medidas.

``` http
GET api/v1/contracts?workflowId={workflowId}
Authorization: Bearer {access token}
```

### <a name="contracts-response"></a>Respuesta de contratos

La respuesta enumera todas las instancias de contrato inteligente del flujo de trabajo especificado. Los administradores de Workbench obtienen todas las instancias de contrato inteligente. Sin embargo, los administradores que no son de Workbench obtienen todas las instancias de contrato inteligente para las que tienen asociado al menos un rol de aplicación o un rol de instancia de contrato inteligente.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts?skip=3&workflowId=1",
    "contracts": [
        {
            "id": 1,
            "provisioningStatus": 2,
            "connectionID": 1,
            "ledgerIdentifier": "0xbcb6127be062acd37818af290c0e43479a153a1c",
            "deployedByUserId": 1,
            "workflowId": 1,
            "contractCodeId": 1,
            "contractProperties": [
                {
                    "workflowPropertyId": 1,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 2,
                    "value": "My first car"
                },
                {
                    "workflowPropertyId": 3,
                    "value": "54321"
                },
                {
                    "workflowPropertyId": 4,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 5,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 6,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 7,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 8,
                    "value": "0xd882530eb3d6395e697508287900c7679dbe02d7"
                }
            ],
            "transactions": [
                {
                    "id": 1,
                    "connectionId": 1,
                    "transactionHash": "0xf3abb829884dc396e03ae9e115a770b230fcf41bb03d39457201449e077080f4",
                    "blockID": 241,
                    "from": "0xd882530eb3d6395e697508287900c7679dbe02d7",
                    "to": null,
                    "value": 0,
                    "isAppBuilderTx": true
                }
            ],
            "contractActions": [
                {
                    "id": 1,
                    "userId": 1,
                    "provisioningStatus": 2,
                    "timestamp": "2018-04-29T23:41:14.9333333",
                    "parameters": [
                        {
                            "name": "Description",
                            "value": "My first car"
                        },
                        {
                            "name": "Price",
                            "value": "54321"
                        }
                    ],
                    "workflowFunctionId": 1,
                    "transactionId": 1,
                    "workflowStateId": 1
                }
            ]
        }
    ]
}
```

## <a name="list-available-actions-for-a-contract"></a>Enumerar las acciones disponibles para un contrato

Use la [API GET de acciones de contrato](/rest/api/azure-blockchain-workbench/contractsv2/contractactionget) para mostrar las acciones de usuario disponibles en función del estado del contrato. 

### <a name="contract-action-request"></a>Solicitud de acciones de contrato

En este ejemplo, el usuario examina todas las acciones disponibles para un nuevo contrato inteligente que ha creado.

``` http
GET /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
```

### <a name="contract-action-response"></a>Respuesta de acciones de contrato

La respuesta enumera todas las acciones que un usuario puede realizar dado el estado actual de la instancia de contrato inteligente especificada.

* Modify: permite al usuario modificar la descripción y el precio de un recurso.
* Terminate: permite al usuario finalizar el contrato del recurso.

Los usuarios obtienen todas las acciones aplicables si el usuario tiene un rol de aplicación asociado o está asociado a un rol de instancia de contrato inteligente para el estado actual de la instancia de contrato inteligente especificada.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts/1/actions?skip=2",
    "workflowFunctions": [
        {
            "id": 2,
            "name": "Modify",
            "description": "Modify the description/price attributes of this asset transfer instance",
            "displayName": "Modify",
            "parameters": [
                {
                    "id": 1,
                    "name": "description",
                    "description": "The new description of the asset",
                    "displayName": "Description",
                    "type": {
                        "id": 2,
                        "name": "string",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                },
                {
                    "id": 2,
                    "name": "price",
                    "description": "The new price of the asset",
                    "displayName": "Price",
                    "type": {
                        "id": 3,
                        "name": "money",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                }
            ],
            "workflowId": 1
        },
        {
            "id": 3,
            "name": "Terminate",
            "description": "Used to cancel this particular instance of asset transfer",
            "displayName": "Terminate",
            "parameters": [],
            "workflowId": 1
        }
    ]
}
```

## <a name="execute-an-action-for-a-contract"></a>Ejecutar una acción para un contrato

Use la [API POST de acciones de contrato](/rest/api/azure-blockchain-workbench/contractsv2/contractactionpost) para realizar acciones en la instancia de contrato inteligente especificada.

### <a name="contract-action-post-request"></a>Solicitud POST de acciones de contrato

En este caso, considere el escenario en el que un usuario quiere modificar la descripción y el precio de un recurso.

``` http
POST /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
actionInformation: {
    "workflowFunctionId": 2,
    "workflowActionParameters": [
        {
            "name": "description",
            "value": "My updated car"
        },
        {
            "name": "price",
            "value": "54321"
        }
    ]
}
```

Los usuarios solo pueden ejecutar la acción dada el estado actual de la instancia de contrato inteligente especificada y el rol de aplicación asociada del usuario o el rol de instancia de contrato inteligente.

### <a name="contract-action-post-response"></a>Respuesta POST de acciones de contrato

Si la publicación se realiza correctamente, se devuelve una respuesta HTTP 200 OK sin ningún cuerpo de respuesta.

``` http
HTTP/1.1 200 OK
Content-type: application/json
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información de referencia sobre las API de Blockchain Workbench, consulte [Referencia de la API REST de Azure Blockchain Workbench](/rest/api/azure-blockchain-workbench).
