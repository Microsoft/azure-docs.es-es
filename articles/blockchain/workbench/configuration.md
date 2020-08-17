---
title: Referencia de metadatos sobre la configuración de Azure Blockchain Workbench
description: Introducción a los metadatos de configuración de aplicaciones de la versión preliminar de Azure Blockchain Workbench.
ms.date: 12/09/2019
ms.topic: article
ms.reviewer: brendal
ms.openlocfilehash: f0ba19bf1d7fdf05014ac199fae9392b5c3249d1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073082"
---
# <a name="azure-blockchain-workbench-configuration-reference"></a>Referencia sobre la configuración de Azure Blockchain Workbench

Las aplicaciones de Azure Blockchain Workbench son flujos de trabajo de varias partes definidas por metadatos de configuración y código de contrato inteligente. Los metadatos de configuración definen los flujos de trabajo de alto nivel y el modelo de interacción de la aplicación de cadena de bloques. Los contratos inteligentes definen la lógica de negocios de la aplicación de cadena de bloques. Workbench usa la configuración y el código de contrato inteligente para generar experiencias de usuario de aplicaciones de cadena de bloques.

Los metadatos de configuración especifican la siguiente información para cada aplicación de cadena de bloques:

* Nombre y descripción de la aplicación de cadena de bloques
* Funciones exclusivas de los usuarios que pueden actuar o participar en la aplicación de cadena de bloques
* Uno o varios flujos de trabajo. Cada flujo de trabajo actúa como una máquina de estados que controla el flujo de la lógica de negocios. Los flujos de trabajo pueden ser independiente o interactuar entre sí.

Cada flujo de trabajo definido especifica lo siguiente:

* Nombre y descripción del flujo de trabajo
* Estados del flujo de trabajo.  Cada estado es una etapa del flujo de control de la lógica de negocios. 
* Las acciones para efectuar la transición al estado siguiente
* Los roles de usuario permitidos para iniciar cada acción
* Los contratos inteligentes que representan la lógica de negocios en archivos de código

## <a name="application"></a>Application

Una aplicación de cadena de bloques contiene metadatos de configuración, flujos de trabajo y roles de los usuarios que pueden actuar o participar en la aplicación.

| Campo | Descripción | Obligatorio |
|-------|-------------|:--------:|
| ApplicationName | Nombre único de aplicación. El contrato inteligente correspondiente debe usar el mismo **ApplicationName** para la clase de contrato aplicable.  | Sí |
| DisplayName | Nombre descriptivo para mostrar de la aplicación. | Sí |
| Descripción | Descripción de la aplicación. | No |
| ApplicationRoles | Colección de [ApplicationRoles](#application-roles). Roles de los usuarios que pueden actuar o participar en la aplicación.  | Sí |
| Workflows | Colección de [flujos de trabajo](#workflows). Cada flujo de trabajo actúa como una máquina de estados que controla el flujo de la lógica de negocios. | Sí |

Para obtener un ejemplo, consulte [ejemplo de archivo de configuración](#configuration-file-example).

## <a name="workflows"></a>Workflows

Se puede modelar la lógica de negocios de una aplicación como una máquina de estados en la que la realización de una acción provoca que el flujo de la lógica de negocios pase de un estado a otro. Un flujo de trabajo es una colección de estos estados y acciones. Cada flujo de trabajo consta de uno o varios contratos inteligentes que representan la lógica de negocios en archivos de código. Un contrato ejecutable es una instancia de un flujo de trabajo.

| Campo | Descripción | Obligatorio | Longitud máxima |
|-------|-------------|:--------:|-----------:|
| Nombre | Nombre único de flujo de trabajo. El contrato inteligente correspondiente debe usar el mismo **nombre** para la clase de contrato aplicable. | Sí | 50 |
| DisplayName | Nombre descriptivo para mostrar del flujo de trabajo. | Sí | 255 |
| Descripción | Descripción del flujo de trabajo. | No | 255 |
| Iniciadores | Colección de [ApplicationRoles](#application-roles). Roles que se asignan a los usuarios que están autorizados para crear contratos en el flujo de trabajo. | Sí | |
| StartState | Nombre del estado inicial del flujo de trabajo. | Sí | |
| Propiedades | Colección de [identificadores](#identifiers). Representa los datos que se pueden leer fuera de la cadena o visualizar en una herramienta de experiencia del usuario. | Sí | |
| Constructor | Define los parámetros de entrada para la creación de una instancia del flujo de trabajo. | Sí | |
| Functions | Una colección de [funciones](#functions) que se pueden ejecutar en el flujo de trabajo. | Sí | |
| States | Una colección de [estados](#states) del flujo de trabajo. | Sí | |

Para obtener un ejemplo, consulte [ejemplo de archivo de configuración](#configuration-file-example).

## <a name="type"></a>Tipo

Tipos de datos admitidos.

| Tipo | Descripción |
|-------|-------------|
| address  | Tipo de dirección de Blockchain, como *contratos* o *usuarios*. |
| array    | Matriz de nivel único de tipo integer, booleano, money o time. Las matrices pueden ser estáticas o dinámicas. Use **ElementType** para especificar el tipo de datos de los elementos dentro de la matriz. Consulte la [configuración de ejemplo](#example-configuration-of-type-array). |
| bool     | Tipo de datos booleano. |
| contrato | Dirección de tipo contrato. |
| enum     | Conjunto enumerado de valores con nombre. Cuando se usa el tipo de enumeración, también especifica una lista de EnumValues. El valor está limitado a 255 caracteres. Los caracteres de valor válidos incluyen letras mayúsculas y minúsculas (A-z, a-z) y números (0-9). Consulte [configuración de ejemplo y uso en Solidity](#example-configuration-of-type-enum). |
| int      | Tipo de datos Integer. |
| money    | Tipo de datos Money. |
| state    | Estado de flujo de trabajo. |
| string  | Tipo de datos String. 4000 caracteres como máximo. Consulte la [configuración de ejemplo](#example-configuration-of-type-string). |
| usuario     | Dirección de tipo usuario. |
| time     | Tipo de datos Time. |
|`[ Application Role Name ]`| Cualquier nombre especificado en un rol de aplicación. Se limita solo a los usuarios de ese tipo de rol. |

### <a name="example-configuration-of-type-array"></a>Configuración de ejemplo de tipo matriz

```json
{
  "Name": "Quotes",
  "Description": "Market quotes",
  "DisplayName": "Quotes",
  "Type": {
    "Name": "array",
    "ElementType": {
        "Name": "int"
    }
  }
}
```

#### <a name="using-a-property-of-type-array"></a>Uso de una propiedad de tipo matriz

Si define una propiedad como de tipo matriz en la configuración, tiene que incluir una función get explícita para devolver la propiedad pública del tipo matriz en Solidity. Por ejemplo:

```
function GetQuotes() public constant returns (int[]) {
     return Quotes;
}
```

### <a name="example-configuration-of-type-string"></a>Configuración de ejemplo de tipo string

``` json
{
  "Name": "description",
  "Description": "Descriptive text",
  "DisplayName": "Description",
  "Type": {
    "Name": "string"
  }
}
```

### <a name="example-configuration-of-type-enum"></a>Configuración de ejemplo de tipo enum

``` json
{
  "Name": "PropertyType",
  "DisplayName": "Property Type",
  "Description": "The type of the property",
  "Type": {
    "Name": "enum",
    "EnumValues": ["House", "Townhouse", "Condo", "Land"]
  }
}
```

#### <a name="using-enumeration-type-in-solidity"></a>Uso del tipo de enumeración en Solidity

Una vez que se define una enumeración en la configuración, puede usar los tipos de enumeración en Solidity. Por ejemplo, puede definir una enumeración denominada PropertyTypeEnum.

```
enum PropertyTypeEnum {House, Townhouse, Condo, Land} PropertyTypeEnum public PropertyType; 
```

La lista de cadenas debe coincidir con la configuración y el contrato inteligente para que sean válidas y declaraciones coherentes en Blockchain Workbench.

Ejemplo de asignación:

```
PropertyType = PropertyTypeEnum.Townhouse;
```

Ejemplo de parámetro de la función: 

``` 
function AssetTransfer(string description, uint256 price, PropertyTypeEnum propertyType) public
{
    InstanceOwner = msg.sender;
    AskingPrice = price;
    Description = description;
    PropertyType = propertyType;
    State = StateType.Active;
    ContractCreated();
}

```

## <a name="constructor"></a>Constructor

Define los parámetros de entrada de una instancia de un flujo de trabajo.

| Campo | Descripción | Obligatorio |
|-------|-------------|:--------:|
| Parámetros | Colección de [identificadores](#identifiers) necesarios para iniciar un contrato inteligente. | Sí |

### <a name="constructor-example"></a>Ejemplo de constructor

``` json
{
  "Parameters": [
    {
      "Name": "description",
      "Description": "The description of this asset",
      "DisplayName": "Description",
      "Type": {
        "Name": "string"
      }
    },
    {
      "Name": "price",
      "Description": "The price of this asset",
      "DisplayName": "Price",
      "Type": {
        "Name": "money"
      }
    }
  ]
}
```

## <a name="functions"></a>Functions

Define las funciones que se pueden ejecutar en el flujo de trabajo.

| Campo | Descripción | Obligatorio | Longitud máxima |
|-------|-------------|:--------:|-----------:|
| Nombre | El nombre único de la función. El contrato inteligente correspondiente debe usar el mismo **nombre** para la función aplicable. | Sí | 50 |
| DisplayName | Nombre descriptivo para mostrar de la función. | Sí | 255 |
| Descripción | Descripción de la función | No | 255 |
| Parámetros | Colección de [identificadores](#identifiers) que se corresponden con los parámetros de la función. | Sí | |

### <a name="functions-example"></a>Ejemplo de funciones

``` json
"Functions": [
  {
    "Name": "Modify",
    "DisplayName": "Modify",
    "Description": "Modify the description/price attributes of this asset transfer instance",
    "Parameters": [
      {
        "Name": "description",
        "Description": "The new description of the asset",
        "DisplayName": "Description",
        "Type": {
          "Name": "string"
        }
      },
      {
        "Name": "price",
        "Description": "The new price of the asset",
        "DisplayName": "Price",
        "Type": {
          "Name": "money"
        }
      }
    ]
  },
  {
    "Name": "Terminate",
    "DisplayName": "Terminate",
    "Description": "Used to cancel this particular instance of asset transfer",
    "Parameters": []
  }
]

```

## <a name="states"></a>States

Una colección de estados únicos dentro de un flujo de trabajo. Cada estado captura un paso del flujo de control de la lógica de negocios. 

| Campo | Descripción | Obligatorio | Longitud máxima |
|-------|-------------|:--------:|-----------:|
| Nombre | Nombre único del estado. El contrato inteligente correspondiente debe usar el mismo **nombre** para el estado aplicable. | Sí | 50 |
| DisplayName | Nombre descriptivo para mostrar del estado. | Sí | 255 |
| Descripción | Descripción del estado. | No | 255 |
| PercentComplete | Un valor entero que se muestra en la interfaz de usuario de Blockchain Workbench para mostrar el progreso en el flujo de control de la lógica de negocios. | Sí | |
| Estilo | Sugerencia visual que indica si el estado representa un estado correcto o erróneo. Hay dos valores válidos: `Success` o `Failure`. | Sí | |
| Transiciones | Colección de las [transiciones](#transitions) disponibles desde el estado actual hasta el siguiente conjunto de estados. | No | |

### <a name="states-example"></a>Ejemplo de estados

``` json
"States": [
    {
      "Name": "Active",
      "DisplayName": "Active",
      "Description": "The initial state of the asset transfer workflow",
      "PercentComplete": 20,
      "Style": "Success",
      "Transitions": [
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Cancels this instance of asset transfer",
          "Function": "Terminate",
          "NextStates": [ "Terminated" ],
          "DisplayName": "Terminate Offer"
        },
        {
          "AllowedRoles": [ "Buyer" ],
          "AllowedInstanceRoles": [],
          "Description": "Make an offer for this asset",
          "Function": "MakeOffer",
          "NextStates": [ "OfferPlaced" ],
          "DisplayName": "Make Offer"
        },
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Modify attributes of this asset transfer instance",
          "Function": "Modify",
          "NextStates": [ "Active" ],
          "DisplayName": "Modify"
        }
      ]
    },
    {
      "Name": "Accepted",
      "DisplayName": "Accepted",
      "Description": "Asset transfer process is complete",
      "PercentComplete": 100,
      "Style": "Success",
      "Transitions": []
    },
    {
      "Name": "Terminated",
      "DisplayName": "Terminated",
      "Description": "Asset transfer has been canceled",
      "PercentComplete": 100,
      "Style": "Failure",
      "Transitions": []
    }
  ]
```

## <a name="transitions"></a>Transiciones

Las acciones disponibles hasta el siguiente estado. Uno o varios roles de usuario pueden realizar una acción en cada estado donde, a su vez, una acción puede realizar la transición de un estado a otro en el flujo de trabajo. 

| Campo | Descripción | Obligatorio |
|-------|-------------|:--------:|
| AllowedRoles | Lista de roles de aplicación a los que se permite iniciar la transición. Todos los usuarios del rol especificado pueden llevar a cabo la acción. | No |
| AllowedInstanceRoles | Lista de roles de usuario que participan o de los cuales se especifica en el contrato inteligente que tienen permiso para iniciar la transición. Los roles de instancia se definen en **Propiedades** en los flujos de trabajo. AllowedInstanceRoles representa a un usuario que participa en una instancia de un contrato inteligente. AllowedInstanceRoles le ofrece la posibilidad de restringir la ejecución de una acción en un rol de usuario en una instancia del contrato.  Por ejemplo, puede que desee permitir solo al usuario que creó el contrato (InstanceOwner) poder terminarlo, en lugar de a todos los usuarios del tipo de rol (Propietario) si especificó ese rol en AllowedRoles. | No |
| DisplayName | Nombre descriptivo para mostrar de la transición. | Sí |
| Descripción | Descripción de la transición. | No |
| Función | El nombre de la función que va a iniciar la transición. | Sí |
| NextStates | Una colección de los posibles estados siguientes después de una transición correcta. | Sí |

### <a name="transitions-example"></a>Ejemplo de transiciones

``` json
"Transitions": [
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Cancels this instance of asset transfer",
    "Function": "Terminate",
    "NextStates": [ "Terminated" ],
    "DisplayName": "Terminate Offer"
  },
  {
    "AllowedRoles": [ "Buyer" ],
    "AllowedInstanceRoles": [],
    "Description": "Make an offer for this asset",
    "Function": "MakeOffer",
    "NextStates": [ "OfferPlaced" ],
    "DisplayName": "Make Offer"
  },
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Modify attributes of this asset transfer instance",
    "Function": "Modify",
    "NextStates": [ "Active" ],
    "DisplayName": "Modify"
  }
]

```

## <a name="application-roles"></a>Roles de aplicación

Los roles de aplicación definen un conjunto de roles que se pueden asignar a los usuarios que deseen actuar o participar en la aplicación. Estos roles se pueden usar para restringir las acciones y la participación en la aplicación de cadena de bloques y en los flujos de trabajo correspondientes. 

| Campo | Descripción | Obligatorio | Longitud máxima |
|-------|-------------|:--------:|-----------:|
| Nombre | El nombre único del rol de aplicación. El contrato inteligente correspondiente debe usar el mismo **nombre** para el rol aplicable. Se reservan los nombres de tipo base. No puede asignar un nombre a un rol de aplicación con el mismo nombre como [Tipo](#type)| Sí | 50 |
| Descripción | Descripción del rol de aplicación. | No | 255 |

### <a name="application-roles-example"></a>Ejemplo de roles de aplicación

``` json
"ApplicationRoles": [
  {
    "Name": "Appraiser",
    "Description": "User that signs off on the asset price"
  },
  {
    "Name": "Buyer",
    "Description": "User that places an offer on an asset"
  }
]
```
## <a name="identifiers"></a>Identificadores

Los identificadores representan una colección de información que se usa para describir las propiedades, el constructor y los parámetros de función del flujo de trabajo. 

| Campo | Descripción | Obligatorio | Longitud máxima |
|-------|-------------|:--------:|-----------:|
| Nombre | El nombre único de la propiedad o parámetro. El contrato inteligente correspondiente debe usar el mismo **nombre** para la propiedad o parámetro aplicables. | Sí | 50 |
| DisplayName | Nombre descriptivo para mostrar de la propiedad o parámetro. | Sí | 255 |
| Descripción | Descripción de la propiedad o parámetro. | No | 255 |
| Tipo | [Tipo de datos](#type) de la propiedad. | Sí |

### <a name="identifiers-example"></a>Ejemplo de identificadores

``` json
"Properties": [
  {
    "Name": "State",
    "DisplayName": "State",
    "Description": "Holds the state of the contract",
    "Type": {
      "Name": "state"
    }
  },
  {
    "Name": "Description",
    "DisplayName": "Description",
    "Description": "Describes the asset being sold",
    "Type": {
      "Name": "string"
    }
  }
]
```

## <a name="configuration-file-example"></a>Ejemplo de archivo de configuración

La transferencia de recursos es un escenario de contrato inteligente pensado para la compra y venta de recursos de alto valor, que requieren un inspector y un evaluador. Los vendedores pueden enumerar los recursos mediante la creación de una instancia de un contrato inteligente de transferencia de recursos. Los compradores pueden realizar ofertas mediante la ejecución de una acción en el contrato inteligente y las demás partes pueden realizar acciones para inspeccionar o evaluar el recurso. Una vez que se marca el recurso como inspeccionado y evaluado, el comprador y el vendedor confirmarán la venta de nuevo antes de establecer el contrato como completo. En cada punto del proceso, todos los participantes pueden ver el estado del contrato a medida que se actualiza. 

Para más información, incluidos los archivos de código, consulte el [ejemplo de transferencia de recursos de Azure Blockchain Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer)

El archivo de configuración siguiente es para el ejemplo de transferencia de recursos:

``` json
{
  "ApplicationName": "AssetTransfer",
  "DisplayName": "Asset Transfer",
  "Description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
  "ApplicationRoles": [
    {
      "Name": "Appraiser",
      "Description": "User that signs off on the asset price"
    },
    {
      "Name": "Buyer",
      "Description": "User that places an offer on an asset"
    },
    {
      "Name": "Inspector",
      "Description": "User that inspects the asset and signs off on inspection"
    },
    {
      "Name": "Owner",
      "Description": "User that signs off on the asset price"
    }
  ],
  "Workflows": [
    {
      "Name": "AssetTransfer",
      "DisplayName": "Asset Transfer",
      "Description": "Handles the business logic for the asset transfer scenario",
      "Initiators": [ "Owner" ],
      "StartState":  "Active",
      "Properties": [
        {
          "Name": "State",
          "DisplayName": "State",
          "Description": "Holds the state of the contract",
          "Type": {
            "Name": "state"
          }
        },
        {
          "Name": "Description",
          "DisplayName": "Description",
          "Description": "Describes the asset being sold",
          "Type": {
            "Name": "string"
          }
        },
        {
          "Name": "AskingPrice",
          "DisplayName": "Asking Price",
          "Description": "The asking price for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "OfferPrice",
          "DisplayName": "Offer Price",
          "Description": "The price being offered for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "InstanceAppraiser",
          "DisplayName": "Instance Appraiser",
          "Description": "The user that appraises the asset",
          "Type": {
            "Name": "Appraiser"
          }
        },
        {
          "Name": "InstanceBuyer",
          "DisplayName": "Instance Buyer",
          "Description": "The user that places an offer for this asset",
          "Type": {
            "Name": "Buyer"
          }
        },
        {
          "Name": "InstanceInspector",
          "DisplayName": "Instance Inspector",
          "Description": "The user that inspects this asset",
          "Type": {
            "Name": "Inspector"
          }
        },
        {
          "Name": "InstanceOwner",
          "DisplayName": "Instance Owner",
          "Description": "The seller of this particular asset",
          "Type": {
            "Name": "Owner"
          }
        }
      ],
      "Constructor": {
        "Parameters": [
          {
            "Name": "description",
            "Description": "The description of this asset",
            "DisplayName": "Description",
            "Type": {
              "Name": "string"
            }
          },
          {
            "Name": "price",
            "Description": "The price of this asset",
            "DisplayName": "Price",
            "Type": {
              "Name": "money"
            }
          }
        ]
      },
      "Functions": [
        {
          "Name": "Modify",
          "DisplayName": "Modify",
          "Description": "Modify the description/price attributes of this asset transfer instance",
          "Parameters": [
            {
              "Name": "description",
              "Description": "The new description of the asset",
              "DisplayName": "Description",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "price",
              "Description": "The new price of the asset",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Terminate",
          "DisplayName": "Terminate",
          "Description": "Used to cancel this particular instance of asset transfer",
          "Parameters": []
        },
        {
          "Name": "MakeOffer",
          "DisplayName": "Make Offer",
          "Description": "Place an offer for this asset",
          "Parameters": [
            {
              "Name": "inspector",
              "Description": "Specify a user to inspect this asset",
              "DisplayName": "Inspector",
              "Type": {
                "Name": "Inspector"
              }
            },
            {
              "Name": "appraiser",
              "Description": "Specify a user to appraise this asset",
              "DisplayName": "Appraiser",
              "Type": {
                "Name": "Appraiser"
              }
            },
            {
              "Name": "offerPrice",
              "Description": "Specify your offer price for this asset",
              "DisplayName": "Offer Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Reject",
          "DisplayName": "Reject",
          "Description": "Reject the user's offer",
          "Parameters": []
        },
        {
          "Name": "AcceptOffer",
          "DisplayName": "Accept Offer",
          "Description": "Accept the user's offer",
          "Parameters": []
        },
        {
          "Name": "RescindOffer",
          "DisplayName": "Rescind Offer",
          "Description": "Rescind your placed offer",
          "Parameters": []
        },
        {
          "Name": "ModifyOffer",
          "DisplayName": "Modify Offer",
          "Description": "Modify the price of your placed offer",
          "Parameters": [
            {
              "Name": "offerPrice",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Accept",
          "DisplayName": "Accept",
          "Description": "Accept the inspection/appraisal results",
          "Parameters": []
        },
        {
          "Name": "MarkInspected",
          "DisplayName": "Mark Inspected",
          "Description": "Mark the asset as inspected",
          "Parameters": []
        },
        {
          "Name": "MarkAppraised",
          "DisplayName": "Mark Appraised",
          "Description": "Mark the asset as appraised",
          "Parameters": []
        }
      ],
      "States": [
        {
          "Name": "Active",
          "DisplayName": "Active",
          "Description": "The initial state of the asset transfer workflow",
          "PercentComplete": 20,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancels this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate Offer"
            },
            {
              "AllowedRoles": [ "Buyer" ],
              "AllowedInstanceRoles": [],
              "Description": "Make an offer for this asset",
              "Function": "MakeOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Make Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Modify attributes of this asset transfer instance",
              "Function": "Modify",
              "NextStates": [ "Active" ],
              "DisplayName": "Modify"
            }
          ]
        },
        {
          "Name": "OfferPlaced",
          "DisplayName": "Offer Placed",
          "Description": "Offer has been placed for the asset",
          "PercentComplete": 30,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Accept the proposed offer for the asset",
              "Function": "AcceptOffer",
              "NextStates": [ "PendingInspection" ],
              "DisplayName": "Accept Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you previously placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Modify the price that you specified for your offer",
              "Function": "ModifyOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Modify Offer"
            }
          ]
        },
        {
          "Name": "PendingInspection",
          "DisplayName": "Pending Inspection",
          "Description": "Asset is pending inspection",
          "PercentComplete": 40,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark this asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "Inspected" ],
              "DisplayName": "Mark Inspected"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "Appraised" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Inspected",
          "DisplayName": "Inspected",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Appraised",
          "DisplayName": "Appraised",
          "Description": "Asset has been appraised, now awaiting inspection",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark the asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Inspected"
            }
          ]
        },
        {
          "Name": "NotionalAcceptance",
          "DisplayName": "Notional Acceptance",
          "Description": "Asset has been inspected and appraised, awaiting final sign-off from buyer and seller",
          "PercentComplete": 50,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "SellerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "BuyerAccepted" ],
              "DisplayName": "BuyerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "BuyerAccepted",
          "DisplayName": "Buyer Accepted",
          "Description": "Buyer has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            }
          ]
        },
        {
          "Name": "SellerAccepted",
          "DisplayName": "Seller Accepted",
          "Description": "Seller has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "Accepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "Accepted",
          "DisplayName": "Accepted",
          "Description": "Asset transfer process is complete",
          "PercentComplete": 100,
          "Style": "Success",
          "Transitions": []
        },
        {
          "Name": "Terminated",
          "DisplayName": "Terminated",
          "Description": "Asset transfer has been canceled",
          "PercentComplete": 100,
          "Style": "Failure",
          "Transitions": []
        }
      ]
    }
  ]
}
```
## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de la API de REST de Azure Blockchain Workbench](/rest/api/azure-blockchain-workbench)
