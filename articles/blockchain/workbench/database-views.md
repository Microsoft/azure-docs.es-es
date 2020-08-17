---
title: Vistas de base de datos de Azure Blockchain Workbench
description: Introducción a las vistas de base de datos SQL de Azure Blockchain Workbench (versión preliminar) disponibles.
ms.date: 09/05/2019
ms.topic: article
ms.reviewer: mmercuri
ms.openlocfilehash: e715bd2297c1e051ae92b15d73c945c9e80c3d94
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87003328"
---
# <a name="azure-blockchain-workbench-database-views"></a>Vistas de base de datos de Azure Blockchain Workbench

Azure Blockchain Workbench (versión preliminar) entrega datos de libros de contabilidad distribuida a una base de datos SQL *fuera de la cadena*. La base de datos fuera de la cadena hace posible usar SQL y las herramientas existentes, como [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017), para interactuar con los datos de la cadena de bloques.

Azure Blockchain Workbench proporciona un conjunto de vistas de base de datos que dan acceso a los datos y que le resultarán útiles al realizar las consultas. Estas vistas están muy desnormalizadas lo cual facilita los primeros pasos para generar rápidamente informes, análisis y, en caso contrario, consumir los datos de la cadena de bloques con las herramientas existentes y sin tener que volver a entrenar al personal de la base de datos.

Esta sección incluye una introducción sobre las vistas de base de datos y los datos que contienen.

> [!NOTE]
> No se admite ningún uso directo de las tablas de base de datos que se encuentran en la base de datos fuera de estas vistas, aunque sea posible.
>

## <a name="vwapplication"></a>vwApplication

Esta vista proporciona información detallada sobre **aplicaciones** que se han cargado en Azure Blockchain Workbench.

| Nombre                             | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                   |
|----------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                    | int           | No          | Un identificador único para la aplicación |
| ApplicationName                  | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDescription           | nvarchar(255) | Sí         | Una descripción de la aplicación |
| ApplicationDisplayName           | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled               | bit           | No          | Identifica si la aplicación está habilitada actualmente<br /> **Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos. |
| UploadedDtTm                     | datetime2(7)  | No          | La fecha y hora en que se cargó un contrato |
| UploadedByUserId                 | int           | No          | El identificador del usuario que ha cargado la aplicación |
| UploadedByUserExternalId         | nvarchar(255) | No          | El identificador externo del usuario que ha cargado la aplicación. De forma predeterminada, este Id. es el usuario de Azure Active Directory para el consorcio.                                                                                                |
| UploadedByUserProvisioningStatus | int           | No          | Identifica el estado actual del proceso de aprovisionamiento para el usuario. Los valores posibles son: <br />0: la API ha creado un usuario<br />1: se ha asociado una clave con el usuario en la base de datos<br />2: el usuario está completamente aprovisionado                         |
| UploadedByUserFirstName          | nvarchar(50)  | Sí         | El nombre del usuario que ha cargado el contrato |
| UploadedByUserLastName           | nvarchar(50)  | Sí         | Los apellidos del usuario que ha cargado el contrato |
| UploadedByUserEmailAddress       | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario que ha cargado el contrato |

## <a name="vwapplicationrole"></a>vwApplicationRole

Esta vista proporciona información detallada sobre los roles que se han definido en las aplicaciones de Azure Blockchain Workbench.

Por ejemplo, en una aplicación de *transferencia de recursos*, se pueden definir roles como *Comprador* y *Vendedor*.

| Nombre                   | Tipo             | Puede ser NULL | Descripción                                       |
|------------------------|------------------|-------------|---------------------------------------------------|
| ApplicationId          | int              | No          | Un identificador único para la aplicación           |
| ApplicationName        | nvarchar(50)     | No          | Nombre de la aplicación                       |
| ApplicationDescription | nvarchar(255)    | Sí         | Una descripción de la aplicación                  |
| ApplicationDisplayName | nvarchar(255)    | No          | El nombre que aparecerá en una interfaz de usuario      |
| RoleId                 | int              | No          | Un identificador único para un rol de la aplicación |
| RoleName               | nvarchar50)      | No          | El nombre del rol                              |
| RoleDescription        | description(255) | Sí         | Una descripción del rol                         |

## <a name="vwapplicationroleuser"></a>vwApplicationRoleUser

Esta vista proporciona información detallada sobre los roles que se han definido en las aplicaciones de Azure Blockchain Workbench y sobre los usuarios asociados a ellas.

Por ejemplo, en una aplicación de *transferencia de recursos*, *John Smith* se puede asociar al rol *Comprador*.

| Nombre                       | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                           |
|----------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId              | int           | No          | Un identificador único para la aplicación                                                                                                                                                                                               |
| ApplicationName            | nvarchar(50)  | No          | Nombre de la aplicación                                                                                                                                                                                                           |
| ApplicationDescription     | nvarchar(255) | Sí         | Una descripción de la aplicación                                                                                                                                                                                                      |
| ApplicationDisplayName     | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario                                                                                                                                                                                          |
| ApplicationRoleId          | int           | No          | Un identificador único para un rol de la aplicación                                                                                                                                                                                     |
| ApplicationRoleName        | nvarchar50)   | No          | El nombre del rol                                                                                                                                                                                                                  |
| ApplicationRoleDescription | nvarchar(255) | Sí         | Una descripción del rol                                                                                                                                                                                                             |
| UserId                     | int           | No          | El identificador del usuario asociado al rol |
| UserExternalId             | nvarchar(255) | No          | El identificador externo del usuario asociado con el rol. De forma predeterminada, este Id. es el usuario de Azure Active Directory para el consorcio.                                                                     |
| UserProvisioningStatus     | int           | No          | Identifica el estado actual del proceso de aprovisionamiento para el usuario. Los valores posibles son: <br />0: la API ha creado un usuario<br />1: se ha asociado una clave con el usuario en la base de datos<br />2: el usuario está completamente aprovisionado |
| UserFirstName              | nvarchar(50)  | Sí         | El nombre del usuario asociado al rol |
| UserLastName               | nvarchar(255) | Sí         | Los apellidos del usuario asociado al rol |
| UserEmailAddress           | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario asociado al rol |

## <a name="vwconnectionuser"></a>vwConnectionUser

Esta vista proporciona información detallada sobre las conexiones que se han definido en Azure Blockchain Workbench y sobre los usuarios asociados a ellas. Para cada conexión, esta vista contiene los datos siguientes:

-   Detalles del libro de contabilidad asociado
-   Información del usuario asociado

| Nombre                     | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                           |
|--------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId             | int           | No          | El identificador único de una conexión de Azure Blockchain Workbench |
| ConnectionEndpointUrl    | nvarchar(50)  | No          | La dirección URL del punto de conexión de una conexión |
| ConnectionFundingAccount | nvarchar(255) | Sí         | La cuenta de financiación asociada a una conexión, si procede |
| LedgerId                 | int           | No          | Identificador único de un libro de contabilidad |
| LedgerName               | nvarchar(50)  | No          | El nombre del libro de contabilidad |
| LedgerDisplayName        | nvarchar(255) | No          | El nombre del libro de contabilidad que se va a mostrar en la interfaz de usuario |
| UserId                   | int           | No          | El identificador del usuario asociado a la conexión |
| UserExternalId           | nvarchar(255) | No          | El identificador externo del usuario asociado a la conexión. De forma predeterminada, este Id. es el usuario de Azure Active Directory para el consorcio. |
| UserProvisioningStatus   | int           | No          |Identifica el estado actual del proceso de aprovisionamiento para el usuario. Los valores posibles son: <br />0: la API ha creado un usuario<br />1: se ha asociado una clave con el usuario en la base de datos<br />2: el usuario está completamente aprovisionado |
| UserFirstName            | nvarchar(50)  | Sí         | El nombre del usuario asociado a la conexión |
| UserLastName             | nvarchar(255) | Sí         | Los apellidos del usuario asociado a la conexión |
| UserEmailAddress         | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario asociado a la conexión |

## <a name="vwcontract"></a>vwContract

Esta vista proporciona detalles sobre los contratos implementados. Para cada contrato, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Implementación asociada del libro de contabilidad para la función
-   Detalles del usuario que inició la acción
-   Detalles relacionados con el bloque y las transacciones de la cadena de bloques

| Nombre                                     | Tipo           | Puede ser NULL | Descripción                                                                                                                                                                                                                                                   |
|------------------------------------------|----------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId                             | int            | No          | El identificador único de una conexión de Azure Blockchain Workbench.                                                                                                                                                                                         |
| ConnectionEndpointUrl                    | nvarchar(50)   | No          | La dirección URL del punto de conexión de una conexión |
| ConnectionFundingAccount                 | nvarchar(255)  | Sí         | La cuenta de financiación asociada a una conexión, si procede |
| LedgerId                                 | int            | No          | Identificador único de un libro de contabilidad |
| LedgerName                               | nvarchar(50)   | No          | El nombre del libro de contabilidad |
| LedgerDisplayName                        | nvarchar(255)  | No          | El nombre del libro de contabilidad que se va a mostrar en la interfaz de usuario |
| ApplicationId                            | int            | No          | Un identificador único para la aplicación |
| ApplicationName                          | nvarchar (50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName                   | nvarchar (255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                       | bit            | No          | Identifica si la aplicación está habilitada actualmente.<br /> **Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos.  |
| WorkflowId                               | int            | No          | Un identificador único del flujo de trabajo asociado a un contrato |
| WorkflowName                             | nvarchar(50)   | No          | El nombre del flujo de trabajo asociado a un contrato |
| WorkflowDisplayName                      | nvarchar(255)  | No          | El nombre del flujo de trabajo asociado al contrato que se muestra en la interfaz de usuario |
| WorkflowDescription                      | nvarchar(255)  | Sí         | La descripción del flujo de trabajo asociado a un contrato |
| ContractCodeId                           | int            | No          | Un identificador único del código de contrato asociado al contrato |
| ContractFileName                         | int            | No          | El nombre del archivo que contiene el código de contrato inteligente de este flujo de trabajo. |
| ContractUploadedDtTm                     | int            | No          | La fecha y hora en que se cargó el código de contrato |
| ContractId                               | int            | No          | El identificador único del contrato |
| ContractProvisioningStatus               | int            | No          | Identifica el estado actual del proceso de aprovisionamiento del contrato. Los valores posibles son: <br />0: la API ha creado el contrato en la base de datos<br />1: el contrato se ha enviado al libro de contabilidad<br />2: el contrato se ha implementado correctamente en el libro de contabilidad<br />3 o 4: no se pudo realizar la implementación del contrato en el libro de contabilidad<br />5: el contrato se implementó correctamente en el libro de contabilidad <br /><br />A partir de la versión 1.5, se admiten los valores de 0 a 5. Para compatibilidad con versiones anteriores en la versión actual, la vista **vwContractV0** está disponible y solo admite valores de 0 a 2. |
| ContractLedgerIdentifier                 | nvarchar (255) |             | La dirección de correo electrónico del usuario que ha implementado el contrato |
| ContractDeployedByUserId                 | int            | No          | Un identificador externo para el usuario que ha implementado el contrato. De forma predeterminada, este Id. es el identificador único global que representa el Id. de Azure Active Directory del usuario.                                                                                                          |
| ContractDeployedByUserExternalId         | nvarchar(255)  | No          | Un identificador externo para el usuario que ha implementado el contrato. De forma predeterminada, este Id. es el identificador único global que representa el Id. de Azure Active Directory del usuario.                                                                                                         |
| ContractDeployedByUserProvisioningStatus | int            | No          | Identifica el estado actual del proceso de aprovisionamiento para el usuario. Los valores posibles son: <br />0: la API ha creado un usuario<br />1: se ha asociado una clave con el usuario en la base de datos <br />2: el usuario está completamente aprovisionado                     |
| ContractDeployedByUserFirstName          | nvarchar(50)   | Sí         | El nombre del usuario que ha implementado el contrato |
| ContractDeployedByUserLastName           | nvarchar(255)  | Sí         | Los apellidos del usuario que ha implementado el contrato |
| ContractDeployedByUserEmailAddress       | nvarchar(255)  | Sí         | La dirección de correo electrónico del usuario que ha implementado el contrato |

## <a name="vwcontractaction"></a>vwContractAction

Esta vista representa la mayoría de la información relacionada con las acciones realizadas en los contratos y está diseñada para facilitar los escenarios comunes de creación de informes. Para cada acción realizada, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Definición de función y parámetros de contrato inteligente
-   Implementación asociada del libro de contabilidad para la función
-   Valores de instancia específicos proporcionados para los parámetros
-   Detalles del usuario que inició la acción
-   Detalles relacionados con el bloque y las transacciones de la cadena de bloques

| Nombre                                     | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                                                                    |
|------------------------------------------|---------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                            | int           | No          | Un identificador único para la aplicación |
| ApplicationName                          | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName                   | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                       | bit           | No          | Este campo identifica si la aplicación está habilitada actualmente. Nota: incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos.                                                  |
| WorkflowId                               | int           | No          | Identificador único de un flujo de trabajo |
| WorkflowName                             | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName                      | nvarchar(255) | No          | El nombre del flujo de trabajo que se va a mostrar en una interfaz de usuario |
| WorkflowDescription                      | nvarchar(255) | Sí         | La descripción del flujo de trabajo |
| ContractId                               | int           | No          | Un identificador único del contrato |
| ContractProvisioningStatus               | int           | No          | Identifica el estado actual del proceso de aprovisionamiento del contrato. Los valores posibles son: <br />0: la API ha creado el contrato en la base de datos<br />1: el contrato se ha enviado al libro de contabilidad<br />2: el contrato se ha implementado correctamente en el libro de contabilidad<br />3 o 4: no se pudo realizar la implementación del contrato en el libro de contabilidad<br />5: el contrato se implementó correctamente en el libro de contabilidad <br /><br />A partir de la versión 1.5, se admiten los valores de 0 a 5. Para compatibilidad con versiones anteriores en la versión actual, la vista **vwContractActionV0** está disponible y solo admite valores de 0 a 2. |
| ContractCodeId                           | int           | No          | Un identificador único para la implementación del código del contrato |
| ContractLedgerIdentifier                 | nvarchar(255) | Sí         | Un identificador único asociado a la versión implementada de un contrato inteligente de un libro de contabilidad distribuida específico. Por ejemplo, Ethereum. |
| ContractDeployedByUserId                 | int           | No          | El identificador único del usuario que ha implementado el contrato |
| ContractDeployedByUserFirstName          | nvarchar(50)  | Sí         | El nombre del usuario que ha implementado el contrato |
| ContractDeployedByUserLastName           | nvarchar(255) | Sí         | Los apellidos del usuario que ha implementado el contrato |
| ContractDeployedByUserExternalId         | nvarchar(255) | No          | El identificador externo del usuario que ha implementado el contrato. De forma predeterminada, este Id. es el identificador único global que representa su identidad en la instancia de Azure Active Directory del consorcio.                                                                                                                                                |
| ContractDeployedByUserEmailAddress       | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario que ha implementado el contrato |
| WorkflowFunctionId                       | int           | No          | El identificador único de una función de flujo de trabajo |
| WorkflowFunctionName                     | nvarchar(50)  | No          | El nombre de la función |
| WorkflowFunctionDisplayName              | nvarchar(255) | No          | El nombre de una función que se mostrará en la interfaz de usuario |
| WorkflowFunctionDescription              | nvarchar(255) | No          | La descripción de la función |
| ContractActionId                         | int           | No          | El identificador único de una acción del contrato |
| ContractActionProvisioningStatus         | int           | No          | Identifica el estado actual del proceso de aprovisionamiento de la acción del contrato. Los valores posibles son: <br />0: la API ha creado la acción del contrato en la base de datos<br />1: la acción del contrato se ha enviado al libro de contabilidad<br />2: la acción del contrato se ha implementado correctamente en el libro de contabilidad<br />3 o 4: no se pudo realizar la implementación del contrato en el libro de contabilidad<br />5: el contrato se implementó correctamente en el libro de contabilidad <br /><br />A partir de la versión 1.5, se admiten los valores de 0 a 5. Para compatibilidad con versiones anteriores en la versión actual, la vista **vwContractActionV0** está disponible y solo admite valores de 0 a 2. |
| ContractActionTimestamp                  | datetime(2,7) | No          | La marca de tiempo de la acción del contrato |
| ContractActionExecutedByUserId           | int           | No          | El identificador único del usuario que ha ejecutado la acción del contrato |
| ContractActionExecutedByUserFirstName    | int           | Sí         | El nombre del usuario que ha ejecutado la acción del contrato |
| ContractActionExecutedByUserLastName     | nvarchar(50)  | Sí         | Los apellidos del usuario que ha ejecutado la acción del contrato |
| ContractActionExecutedByUserExternalId   | nvarchar(255) | Sí         | El identificador externo del usuario que ha ejecutado la acción del contrato. De forma predeterminada, este Id. es el identificador único global que representa su identidad en la instancia de Azure Active Directory del consorcio. |
| ContractActionExecutedByUserEmailAddress | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario que ha ejecutado la acción del contrato |
| WorkflowFunctionParameterId              | int           | No          | Un identificador único de un parámetro de la función |
| WorkflowFunctionParameterName            | nvarchar(50)  | No          | El nombre de un parámetro de la función |
| WorkflowFunctionParameterDisplayName     | nvarchar(255) | No          | El nombre de un parámetro de la función que se mostrará en la interfaz de usuario |
| WorkflowFunctionParameterDataTypeId      | int           | No          | El identificador único para el tipo de datos asociado a un parámetro de función del flujo de trabajo |
| WorkflowParameterDataTypeName            | nvarchar(50)  | No          | El nombre de un tipo de datos asociado a un parámetro de función del flujo de trabajo |
| ContractActionParameterValue             | nvarchar(255) | No          | El valor del parámetro almacenado en el contrato inteligente |
| BlockHash                                | nvarchar(255) | Sí         | Hash del bloque |
| BlockNumber                              | int           | Sí         | El número del bloque en el libro de contabilidad |
| BlockTimestamp                           | datetime(2,7) | Sí         | La marca de tiempo del bloque |
| TransactionId                            | int           | No          | Un identificador único para la transacción |
| TransactionFrom                          | nvarchar(255) | Sí         | La entidad que originó la transacción |
| TransactionTo                            | nvarchar(255) | Sí         | La entidad con la que se ha realizado la transacción |
| TransactionHash                          | nvarchar(255) | Sí         | El hash de una transacción |
| TransactionIsWorkbenchTransaction        | bit           | Sí         | Un bit que indica si la transacción es una transacción de Azure Blockchain Workbench. |
| TransactionProvisioningStatus            | int           | Sí         | Identifica el estado actual del proceso de aprovisionamiento de la transacción. Los valores posibles son: <br />0: la API ha creado la transacción en la base de datos<br />1: la transacción se ha enviado al libro de contabilidad<br />2: la transacción se ha implementado correctamente en el libro de contabilidad                 |
| TransactionValue                         | decimal(32,2) | Sí         | El valor de la transacción |

## <a name="vwcontractproperty"></a>vwContractProperty

Esta vista representa la mayoría de la información relacionada con las propiedades asociadas a un contrato y está diseñada para facilitar los escenarios comunes de creación de informes. Para cada propiedad que se toma, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Detalles del usuario que ha implementado el flujo de trabajo
-   Definición de la propiedad de contrato inteligente asociada
-   Valores de instancia específicos para las propiedades
-   Detalles de la propiedad de estado del contrato

| Nombre                               | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | No          | Un identificador único para la aplicación |
| ApplicationName                    | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName             | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                 | bit           | No          | Identifica si la aplicación está habilitada actualmente.<br />**Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos.                      |
| WorkflowId                         | int           | No          | El identificador único del flujo de trabajo |
| WorkflowName                       | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName                | nvarchar(255) | No          | El nombre del flujo de trabajo que se va a mostrar en la interfaz de usuario |
| WorkflowDescription                | nvarchar(255) | Sí         | La descripción del flujo de trabajo |
| ContractId                         | int           | No          | El identificador único del contrato |
| ContractProvisioningStatus         | int           | No          | Identifica el estado actual del proceso de aprovisionamiento del contrato. Los valores posibles son: <br />0: la API ha creado el contrato en la base de datos<br />1: el contrato se ha enviado al libro de contabilidad<br />2: el contrato se ha implementado correctamente en el libro de contabilidad<br />3 o 4: no se pudo realizar la implementación del contrato en el libro de contabilidad<br />5: el contrato se implementó correctamente en el libro de contabilidad <br /><br />A partir de la versión 1.5, se admiten los valores de 0 a 5. Para compatibilidad con versiones anteriores en la versión actual, la vista **vwContractPropertyV0** está disponible y solo admite valores de 0 a 2. |
| ContractCodeId                     | int           | No          | Un identificador único para la implementación del código del contrato |
| ContractLedgerIdentifier           | nvarchar(255) | Sí         | Un identificador único asociado a la versión implementada de un contrato inteligente de un libro de contabilidad distribuida específico. Por ejemplo, Ethereum. |
| ContractDeployedByUserId           | int           | No          | El identificador único del usuario que ha implementado el contrato |
| ContractDeployedByUserFirstName    | nvarchar(50)  | Sí         | El nombre del usuario que ha implementado el contrato |
| ContractDeployedByUserLastName     | nvarchar(255) | Sí         | Los apellidos del usuario que ha implementado el contrato |
| ContractDeployedByUserExternalId   | nvarchar(255) | No          | El identificador externo del usuario que ha implementado el contrato. De forma predeterminada, este Id. es el identificador único global que representa su identidad en la instancia de Azure Active Directory del consorcio. |
| ContractDeployedByUserEmailAddress | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario que ha implementado el contrato |
| WorkflowPropertyId                 | int           |             | El identificador único de una propiedad de un flujo de trabajo |
| WorkflowPropertyDataTypeId         | int           | No          | El identificador del tipo de datos de la propiedad |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | No          | El nombre del tipo de datos de la propiedad |
| WorkflowPropertyName               | nvarchar(50)  | No          | El nombre de la propiedad del flujo de trabajo |
| WorkflowPropertyDisplayName        | nvarchar(255) | No          | El nombre para mostrar de la propiedad del flujo de trabajo |
| WorkflowPropertyDescription        | nvarchar(255) | Sí         | Descripción de la propiedad |
| ContractPropertyValue              | nvarchar(255) | No          | El valor de una propiedad del contrato |
| StateName                          | nvarchar(50)  | Sí         | Si esta propiedad contiene el estado del contrato, es el nombre para mostrar del estado. Si no está asociado con el estado, el valor será NULL. |
| StateDisplayName                   | nvarchar(255) | No          | Si esta propiedad contiene el estado, es el nombre para mostrar del estado. Si no está asociado con el estado, el valor será NULL. |
| StateValue                         | nvarchar(255) | Sí         | Si esta propiedad contiene el estado, es el valor del mismo. Si no está asociado con el estado, el valor será NULL. |

## <a name="vwcontractstate"></a>vwContractState

Esta vista representa la mayoría de la información relacionada con el estado de un contrato específico y está diseñada para facilitar los escenarios comunes de creación de informes. Cada registro de esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Detalles del usuario que ha implementado el flujo de trabajo
-   Definición de la propiedad de contrato inteligente asociada
-   Detalles de la propiedad de estado del contrato

| Nombre                               | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | No          | Un identificador único para la aplicación |
| ApplicationName                    | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName             | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                 | bit           | No          | Identifica si la aplicación está habilitada actualmente.<br />**Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos. |
| WorkflowId                         | int           | No          | El identificador único del flujo de trabajo |
| WorkflowName                       | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName                | nvarchar(255) | No          | El nombre que se muestra en la interfaz de usuario |
| WorkflowDescription                | nvarchar(255) | Sí         | La descripción del flujo de trabajo |
| ContractLedgerImplementationId     | nvarchar(255) | Sí         | Un identificador único asociado a la versión implementada de un contrato inteligente de un libro de contabilidad distribuida específico. Por ejemplo, Ethereum. |
| ContractId                         | int           | No          | Un identificador único del contrato |
| ContractProvisioningStatus         | int           | No          |Identifica el estado actual del proceso de aprovisionamiento del contrato. Los valores posibles son: <br />0: la API ha creado el contrato en la base de datos<br />1: el contrato se ha enviado al libro de contabilidad<br />2: el contrato se ha implementado correctamente en el libro de contabilidad<br />3 o 4: no se pudo realizar la implementación del contrato en el libro de contabilidad<br />5: el contrato se implementó correctamente en el libro de contabilidad <br /><br />A partir de la versión 1.5, se admiten los valores de 0 a 5. Para compatibilidad con versiones anteriores en la versión actual, la vista **vwContractStateV0** está disponible y solo admite valores de 0 a 2. |
| ConnectionId                       | int           | No          | El identificador único de la instancia de la cadena de bloques en la que se implementa el flujo de trabajo |
| ContractCodeId                     | int           | No          | Un identificador único para la implementación del código del contrato |
| ContractDeployedByUserId           | int           | No          | El identificador único del usuario que ha implementado el contrato |
| ContractDeployedByUserExternalId   | nvarchar(255) | No          | El identificador externo del usuario que ha implementado el contrato. De forma predeterminada, este Id. es el identificador único global que representa su identidad en la instancia de Azure Active Directory del consorcio. |
| ContractDeployedByUserFirstName    | nvarchar(50)  | Sí         | El nombre del usuario que ha implementado el contrato |
| ContractDeployedByUserLastName     | nvarchar(255) | Sí         | Los apellidos del usuario que ha implementado el contrato |
| ContractDeployedByUserEmailAddress | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario que ha implementado el contrato |
| WorkflowPropertyId                 | int           | No          | El identificador único de una propiedad del flujo de trabajo |
| WorkflowPropertyDataTypeId         | int           | No          | El identificador del tipo de datos de la propiedad del flujo de trabajo |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | No          | El nombre del tipo de datos de la propiedad del flujo de trabajo |
| WorkflowPropertyName               | nvarchar(50)  | No          | El nombre de la propiedad del flujo de trabajo |
| WorkflowPropertyDisplayName        | nvarchar(255) | No          | El nombre para mostrar de la propiedad que va a aparecer en una interfaz de usuario |
| WorkflowPropertyDescription        | nvarchar(255) | Sí         | La descripción de la propiedad |
| ContractPropertyValue              | nvarchar(255) | No          | El valor de una propiedad almacenado en el contrato |
| StateName                          | nvarchar(50)  | Sí         | Si esta propiedad contiene el estado, es el nombre para mostrar del estado. Si no está asociado con el estado, el valor será NULL. |
| StateDisplayName                   | nvarchar(255) | No          | Si esta propiedad contiene el estado, es el nombre para mostrar del estado. Si no está asociado con el estado, el valor será NULL. |
| StateValue                         | nvarchar(255) | Sí         | Si esta propiedad contiene el estado, es el valor del mismo. Si no está asociado con el estado, el valor será NULL. |

## <a name="vwuser"></a>vwUser

Esta vista proporciona detalles sobre los miembros del consorcio que se han aprovisionado para el uso de Azure Blockchain Workbench. De forma predeterminada, los datos se rellenan mediante el aprovisionamiento inicial del usuario.

| Nombre               | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                               |
|--------------------|---------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                 | int           | No          | El identificador único de un usuario |
| ExternalID         | nvarchar(255) | No          | El identificador externo de un usuario. De forma predeterminada, este Id. es el identificador único global que representa el Id. de Azure Active Directory del usuario. |
| ProvisioningStatus | int           | No          |Identifica el estado actual del proceso de aprovisionamiento para el usuario. Los valores posibles son: <br />0: la API ha creado un usuario<br />1: se ha asociado una clave con el usuario en la base de datos<br />2: el usuario está completamente aprovisionado |
| Nombre          | nvarchar(50)  | Sí         | El nombre del usuario |
| Apellidos           | nvarchar(50)  | Sí         | Los apellidos del usuario |
| EmailAddress       | nvarchar(255) | Sí         | La dirección de correo electrónico del usuario. |

## <a name="vwworkflow"></a>vwWorkflow

Esta vista representa los principales metadatos del flujo de trabajo así como las funciones y parámetros de este. Diseñada para los informes, esta vista también contiene metadatos sobre la aplicación asociada al flujo de trabajo. Esta vista contiene datos de varias tablas subyacentes que facilitan la creación de informes sobre los flujos de trabajo. Para cada flujo de trabajo, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Información de estado de inicio del flujo de trabajo asociado

| Nombre                              | Tipo          | Puede ser NULL | Descripción                                                                                                                                |
|-----------------------------------|---------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                     | int           | No          | Un identificador único para la aplicación |
| ApplicationName                   | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName            | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                | bit           | No          | Identifica si la aplicación está habilitada |
| WorkflowId                        | int           | Sí         | Identificador único de un flujo de trabajo |
| WorkflowName                      | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName               | nvarchar(255) | No          | El nombre que se muestra en la interfaz de usuario |
| WorkflowDescription               | nvarchar(255) | Sí         | La descripción del flujo de trabajo. |
| WorkflowConstructorFunctionId     | int           | No          | El identificador de la función de flujo de trabajo que actúa como constructor del flujo de trabajo |
| WorkflowStartStateId              | int           | No          | Un identificador único del estado |
| WorkflowStartStateName            | nvarchar(50)  | No          | Nombre del estado |
| WorkflowStartStateDisplayName     | nvarchar(255) | No          | El nombre del estado que aparecerá en la interfaz de usuario |
| WorkflowStartStateDescription     | nvarchar(255) | Sí         | La descripción del estado de flujo de trabajo |
| WorkflowStartStateStyle           | nvarchar(50)  | Sí         | Este valor permite identificar el porcentaje completado del flujo de trabajo cuando se encuentra en este estado |
| WorkflowStartStateValue           | int           | No          | Valor del estado |
| WorkflowStartStatePercentComplete | int           | No          | Una descripción de texto que proporciona una sugerencia a los clientes sobre cómo representar este estado en la interfaz de usuario. Entre los estados admitidos se incluyen *Correcto* y *Error* |

## <a name="vwworkflowfunction"></a>vwWorkflowFunction

Esta vista representa los principales metadatos del flujo de trabajo así como las funciones y parámetros de este. Diseñada para los informes, esta vista también contiene metadatos sobre la aplicación asociada al flujo de trabajo. Esta vista contiene datos de varias tablas subyacentes que facilitan la creación de informes sobre los flujos de trabajo. Para cada función del flujo de trabajo, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Información detallada de la función del flujo de trabajo

| Nombre                                 | Tipo          | Puede ser NULL | Descripción                                                                          |
|--------------------------------------|---------------|-------------|--------------------------------------------------------------------------------------|
| ApplicationId                        | int           | No          | Un identificador único para la aplicación |
| ApplicationName                      | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName               | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled                   | bit           | No          | Identifica si la aplicación está habilitada |
| WorkflowId                           | int           | No          | Identificador único de un flujo de trabajo |
| WorkflowName                         | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName                  | nvarchar(255) | No          | El nombre del flujo de trabajo que se va a mostrar en la interfaz de usuario |
| WorkflowDescription                  | nvarchar(255) | Sí         | La descripción del flujo de trabajo |
| WorkflowFunctionId                   | int           | No          | Identificador único de una función |
| WorkflowFunctionName                 | nvarchar(50)  | Sí         | El nombre de la función |
| WorkflowFunctionDisplayName          | nvarchar(255) | No          | El nombre de una función que se mostrará en la interfaz de usuario |
| WorkflowFunctionDescription          | nvarchar(255) | Sí         | La descripción de la función del flujo de trabajo |
| WorkflowFunctionIsConstructor        | bit           | No          | Identifica si la función del flujo de trabajo actúa como constructor de este |
| WorkflowFunctionParameterId          | int           | No          | El identificador único de un parámetro de una función |
| WorkflowFunctionParameterName        | nvarchar(50)  | No          | El nombre de un parámetro de la función |
| WorkflowFunctionParameterDisplayName | nvarchar(255) | No          | El nombre de un parámetro de la función que se mostrará en la interfaz de usuario |
| WorkflowFunctionParameterDataTypeId  | int           | No          | El identificador único para el tipo de datos asociado a un parámetro de función del flujo de trabajo |
| WorkflowParameterDataTypeName        | nvarchar(50)  | No          | El nombre de un tipo de datos asociado a un parámetro de función del flujo de trabajo |

## <a name="vwworkflowproperty"></a>vwWorkflowProperty

Esta vista representa las propiedades definidas para un flujo de trabajo. Para cada propiedad, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Detalles de la propiedad de flujo de trabajo

| Nombre                         | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | No          | Un identificador único para la aplicación |
| ApplicationName              | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName       | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| ApplicationEnabled           | bit           | No          | Identifica si la aplicación está habilitada actualmente.<br />**Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos. |
| WorkflowId                   | int           | No          | El identificador único del flujo de trabajo |
| WorkflowName                 | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName          | nvarchar(255) | No          | El nombre del flujo de trabajo que aparecerá en una interfaz de usuario |
| WorkflowDescription          | nvarchar(255) | Sí         | Una descripción del flujo de trabajo |
| WorkflowPropertyID           | int           | No          | El identificador único de una propiedad de un flujo de trabajo |
| WorkflowPropertyName         | nvarchar(50)  | No          | El nombre de la propiedad |
| WorkflowPropertyDescription  | nvarchar(255) | Sí         | Descripción de la propiedad |
| WorkflowPropertyDisplayName  | nvarchar(255) | No          | El nombre que aparecerá en una interfaz de usuario |
| WorkflowPropertyWorkflowId   | int           | No          | El identificador del flujo de trabajo al que está asociada esta propiedad |
| WorkflowPropertyDataTypeId   | int           | No          | El identificador del tipo de datos definido para la propiedad |
| WorkflowPropertyDataTypeName | nvarchar(50)  | No          | El nombre del tipo de datos definido para la propiedad |
| WorkflowPropertyIsState      | bit           | No          | Este campo identifica si esta propiedad de flujo de trabajo contiene el estado del flujo de trabajo |

## <a name="vwworkflowstate"></a>vwWorkflowState

Esta vista representa las propiedades asociadas a un flujo de trabajo. Para cada contrato, esta vista contiene los datos siguientes:

-   Definición de aplicación asociada
-   Definición de flujo de trabajo asociada
-   Información de estado de flujo de trabajo

| Nombre                         | Tipo          | Puede ser NULL | Descripción                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | No          | Un identificador único para la aplicación |
| ApplicationName              | nvarchar(50)  | No          | Nombre de la aplicación |
| ApplicationDisplayName       | nvarchar(255) | No          | Una descripción de la aplicación |
| ApplicationEnabled           | bit           | No          | Identifica si la aplicación está habilitada actualmente.<br />**Nota:** Incluso aunque una aplicación pueda aparecer como deshabilitada en la base de datos, los contratos asociados permanecen en la cadena de bloques y los datos sobre esos contratos permanecen en la base de datos. |
| WorkflowId                   | int           | No          | El identificador único del flujo de trabajo |
| WorkflowName                 | nvarchar(50)  | No          | El nombre del flujo de trabajo |
| WorkflowDisplayName          | nvarchar(255) | No          | El nombre del flujo de trabajo que aparecerá en la interfaz de usuario |
| WorkflowDescription          | nvarchar(255) | Sí         | La descripción del flujo de trabajo |
| WorkflowStateID              | int           | No          | El identificador único del estado |
| WorkflowStateName            | nvarchar(50)  | No          | Nombre del estado |
| WorkflowStateDisplayName     | nvarchar(255) | No          | El nombre del estado que aparecerá en la interfaz de usuario |
| WorkflowStateDescription     | nvarchar(255) | Sí         | La descripción del estado de flujo de trabajo |
| WorkflowStatePercentComplete | int           | No          | Este valor permite identificar el porcentaje completado del flujo de trabajo cuando se encuentra en este estado |
| WorkflowStateValue           | nvarchar(50)  | No          | Valor del estado |
| WorkflowStateStyle           | nvarchar(50)  | No          | Una descripción de texto que proporciona una sugerencia a los clientes sobre cómo representar este estado en la interfaz de usuario. Entre los estados admitidos se incluyen *Correcto* y *Error* |
