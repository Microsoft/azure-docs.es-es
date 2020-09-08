---
title: 'Información general: automatización de la implementación para Azure Logic Apps'
description: Obtenga información sobre las plantillas de Azure Resource Manager para automatizar la implementación de Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: 9d3c5a914fe472dd7e4f797cb633e65951bf07e7
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871469"
---
# <a name="overview-automate-deployment-for-azure-logic-apps-by-using-azure-resource-manager-templates"></a>Introducción: Automatización de la implementación para Azure Logic Apps mediante plantillas de Azure Resource Manager

Cuando esté listo para automatizar la creación e implementación de la aplicación lógica, puede expandir la definición de flujo de trabajo subyacente de la aplicación lógica en una [plantilla de Azure Resource Manager](../azure-resource-manager/management/overview.md). Esta plantilla define la infraestructura, los recursos, los parámetros y otra información para aprovisionar e implementar la aplicación lógica. Al definir los parámetros para los valores que varían en la implementación, lo que también se conoce como *parametrización*, puede implementar de forma repetida y coherente aplicaciones lógicas según las diferentes necesidades de implementación.

Por ejemplo, si realiza implementaciones en entornos de desarrollo, prueba y producción, probablemente usa cadenas de conexión diferentes para cada entorno. Puede declarar parámetros de plantilla que acepten cadenas de conexión diferentes y, a continuación, almacenar dichas cadenas en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md) independiente. De este modo, puede cambiar esos valores sin tener que actualizar y volver a implementar la plantilla. En los casos en los que tenga valores de parámetros que sean confidenciales o deban protegerse, como contraseñas y secretos, puede almacenarlos en [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) y hacer que el archivo de parámetros recupere esos valores. Sin embargo, en estos casos, debería hacer la implementación de nuevo para recuperar los valores actuales.

En esta información general se describen los atributos de una plantilla de Resource Manager que incluye una definición de flujo de trabajo de una aplicación lógica. Tanto la plantilla como la definición de flujo de trabajo usan la sintaxis JSON, pero existen algunas diferencias porque la definición de flujo de trabajo también sigue el [esquema del lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md). Por ejemplo, las expresiones de plantilla y las de definición de flujo de trabajo difieren en la manera en que [hacen referencia a los parámetros](#parameter-references) y los valores que pueden aceptar.

> [!TIP]
> Para obtener de manera fácil una plantilla de aplicación lógica parametrizada válida que esté principalmente lista para la implementación, use Visual Studio (la edición Community gratis o superior) y las herramientas de Azure Logic Apps para Visual Studio. Después, puede [crear la aplicación lógica en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) o [buscar y descargar una aplicación lógica existente de Azure en Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).
>
> O bien, puede crear plantillas de aplicaciones lógicas mediante [Azure PowerShell con el módulo LogicAppTemplate](../logic-apps/logic-apps-create-azure-resource-manager-templates.md#azure-powershell).

En la aplicación lógica de ejemplo de este tema se usa un [desencadenador de Office 365 Outlook](/connectors/office365/), que se activa cuando llega un nuevo correo electrónico, y una [acción de Azure Blob Storage](/connectors/azureblob/) que crea un blob para el cuerpo del correo electrónico y carga dicho blob en un contenedor de almacenamiento de Azure. En el ejemplo también se muestra cómo se deben parametrizar los valores que varían en la implementación.

Para obtener más información sobre las plantillas de Resource Manager, consulte los temas siguientes:

* [Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Azure Resource Manager template best practices](../azure-resource-manager/templates/template-best-practices.md) (Procedimientos recomendados de plantillas de Azure Resource Manager)
* [Desarrollo de plantillas de Azure Resource Manager para mantener la coherencia en la nube](../azure-resource-manager/templates/templates-cloud-consistency.md)

Para ver muestras de las plantillas de aplicaciones lógicas, consulte estos ejemplos:

* [Plantilla completa](#full-example-template) que se usa para los ejemplos de este tema
* [Ejemplo de la plantilla de aplicación lógica de inicio rápido](https://github.com/Azure/azure-quickstart-templates/blob/master/101-logic-app-create) en GitHub

Para obtener información sobre los recursos de plantilla específicos de Logic Apps, las cuentas de integración y los artefactos de la cuenta de integración, consulte [Tipos de recursos de Microsoft.Logic](/azure/templates/microsoft.logic/allversions).

<a name="template-structure"></a>

## <a name="template-structure"></a>Estructura de plantilla

En el nivel superior, una plantilla de Resource Manager sigue esta estructura, que se describe detalladamente en el tema [Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md):

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

En el caso de una plantilla de aplicación lógica, principalmente debe trabajar con estos objetos de plantilla:

| Atributo | Descripción |
|-----------|-------------|
| `parameters` | Declara los [parámetros de plantilla](../azure-resource-manager/templates/template-syntax.md#parameters) para aceptar los valores que se van a usar al crear y personalizar los recursos para la implementación en Azure. Por ejemplo, estos parámetros aceptan los valores del nombre y la ubicación de la aplicación lógica, las conexiones y otros recursos necesarios para la implementación. Puede almacenar estos valores de parámetros en un [archivo de parámetros](#template-parameter-files), que se describe más adelante en este tema. Para obtener información general, consulte la sección [Parámetros del artículo Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#parameters). |
| `resources` | Define los [recursos](../azure-resource-manager/templates/template-syntax.md#resources) que se crean o actualizan e implementan en un grupo de recursos de Azure, como la aplicación lógica, las conexiones, las cuentas de Azure Storage, etc. Para obtener información general, consulte la sección [Recursos del artículo Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#resources). |
||||

La plantilla de la aplicación lógica usa este formato de nombre de archivo:

**<*logic-app-name*>.json**

> [!IMPORTANT]
> La sintaxis de la plantilla distingue mayúsculas de minúsculas, por lo que debe asegurarse de que hace un uso correcto de las mayúsculas y minúsculas. 

<a name="template-parameters"></a>

## <a name="template-parameters"></a>Parámetros de plantilla

Una plantilla de aplicación lógica tiene varios objetos `parameters` que existen en diferentes niveles y realizan diferentes funciones. Por ejemplo, en el nivel superior, puede declarar [parámetros de plantilla](../azure-resource-manager/templates/template-syntax.md#parameters) para los valores que se deben aceptar y usar en la implementación al crear e implementar recursos en Azure, por ejemplo:

* Su aplicación lógica
* Conexiones que usa la lógica para acceder a otros servicios y sistemas a través de [conectores administrados](../connectors/apis-list.md)
* Otros recursos que necesita su aplicación lógica para la implementación

  Por ejemplo, si la aplicación lógica usa una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) para escenarios negocio a negocio (B2B), el objeto `parameters` de nivel superior de la plantilla declara el parámetro que acepta el identificador del recurso para esa cuenta de integración.

A continuación, se muestra la estructura y la sintaxis generales de una definición de parámetro, que se describe detalladamente en la sección [Parámetros del artículo Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#parameters):

```json
"<parameter-name>": {
   "type": "<parameter-type>",
   "defaultValue": <default-parameter-value>,
   <other-parameter-attributes>,
   "metadata": {
      "description": "<parameter-description>"
   }
},
```

En este ejemplo solo se muestran los parámetros de plantilla de los valores usados para crear e implementar estos recursos en Azure:

* Nombre y ubicación de la aplicación lógica
* Identificador que se usará para una cuenta de integración que esté vinculada a la aplicación lógica

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location for the logic app"
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

Excepto en el caso de los parámetros que controlan valores que son confidenciales o deben protegerse, como nombres de usuario, contraseñas y secretos, todos estos parámetros incluyen atributos `defaultValue`, aunque en algunos casos, los valores predeterminados son valores vacíos. Los valores de implementación que se usarán para estos parámetros de plantilla se proporcionan mediante el [archivo de parámetros](#template-parameter-files) de ejemplo que se describe más adelante en este tema.

Para obtener más información sobre la protección de los parámetros de plantilla, vea estos temas:

* [Recomendaciones de seguridad para parámetros de plantilla](../azure-resource-manager/templates/template-best-practices.md#parameters)
* [Mejora de la seguridad de los parámetros de plantilla](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)
* [Pasar valores de parámetros seguros con Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

Otros objetos de plantilla a menudo hacen referencia a los parámetros de plantilla a fin de poder usar los valores que pasan a través de los parámetros de plantilla, por ejemplo:

* El [objeto de recursos de la plantilla](#template-resources), que se describe más adelante en este tema, define cada recurso de Azure que quiere crear e implementar, como la [definición de recursos de la aplicación lógica](#logic-app-resource-definition). Estos recursos suelen usar valores de parámetros de plantilla, como el nombre y la ubicación de la aplicación lógica, y la información de conexión.

* En un nivel más profundo de la definición de recursos de la aplicación lógica, el [objeto de parámetros de la definición de flujo de trabajo](#workflow-definition-parameters) declara los parámetros de los valores que se van a usar en el tiempo de ejecución de la aplicación lógica. Por ejemplo, puede declarar parámetros de definición de flujo de trabajo para el nombre de usuario y la contraseña que un desencadenador HTTP utiliza para la autenticación. Para especificar los valores de los parámetros de definición de flujo de trabajo, use el objeto `parameters` que está *fuera* de la definición de flujo de trabajo, pero que sigue *dentro* de la definición de recursos de la aplicación lógica. En este objeto `parameters` externo, puede hacer referencia a los parámetros de plantilla declarados previamente, que pueden aceptar valores en la implementación de un archivo de parámetros.

Cuando se hace referencia a los parámetros, las expresiones y las funciones de plantilla usan una sintaxis diferente y se comportan de manera distinta a la de las expresiones y funciones de la definición de flujo de trabajo. Para obtener más información sobre estas diferencias, consulte [Referencias a parámetros](#parameter-references) más adelante en este tema.

<a name="best-practices-template-parameters"></a>

## <a name="best-practices---template-parameters"></a>Procedimientos recomendados: parámetros de plantilla

Estos son algunos procedimientos recomendados para definir parámetros:

* Declare los parámetros solo para los valores que varían, en función de sus necesidades de implementación. No declare parámetros para los valores que se mantengan iguales para los diferentes requisitos de implementación.

* Incluya el atributo `defaultValue`, que puede especificar valores vacíos, para todos los parámetros excepto los valores que sean confidenciales o que se deban proteger. Use siempre parámetros seguros para los nombres de usuario, las contraseñas y los secretos. Para ocultar o proteger los valores de parámetros confidenciales, siga las instrucciones de estos temas:

  * [Recomendaciones de seguridad para parámetros de plantilla](../azure-resource-manager/templates/template-best-practices.md#parameters)

  * [Mejora de la seguridad de los parámetros de plantilla](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)

  * [Pasar valores de parámetros seguros con Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

* Para diferenciar los nombres de los parámetros de plantilla de los nombres de los parámetros de la definición de flujo de trabajo, puede usar nombres de parámetros de plantilla descriptivos, por ejemplo: `TemplateFabrikamPassword`.

Para obtener más procedimientos recomendados de plantillas, consulte [Procedimientos recomendados para parámetros de plantilla](../azure-resource-manager/templates/template-best-practices.md#parameters).

<a name="template-parameter-files"></a>

## <a name="template-parameters-file"></a>Archivo de parámetros de plantilla

Para proporcionar los valores de los parámetros de plantilla, almacénelos en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md). De este modo, puede usar diferentes archivos de parámetros en función de sus necesidades de implementación. Este es el formato de nombre de archivo que se va a usar:

* Nombre de archivo de la plantilla de la aplicación lógica: **<*logic-app-name*>.json**
* Nombre del archivo de parámetros: **<*logic-app-name*>.parameters.json**

Esta es la estructura del interior del archivo de parámetros, que incluye una referencia al almacén de claves para [pasar un valor de parámetro seguro con Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md):

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "<parameter-name-1>": {
         "value": "<parameter-value>"
      },
      "<parameter-name-2>": {
         "value": "<parameter-value>"
      },
      "<secured-parameter-name>": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/<key-vault-name>",
            },
            "secretName: "<secret-name>"
         }
      },
      <other-parameter-values>
   }
}
```

Este archivo de parámetros de ejemplo especifica los valores de los parámetros de plantilla declarados anteriormente en este tema:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      }
   }
}
```

<a name="template-resources"></a>

## <a name="template-resources"></a>Recursos de plantilla

La plantilla tiene un objeto `resources`, que es una matriz que contiene las definiciones de cada recurso que se va a crear e implementar en Azure, como la [definición de recursos de la aplicación lógica](#logic-app-resource-definition), [cualquier definición de recursos de conexión](#connection-resource-definitions) y cualquier otro recurso que necesite la aplicación lógica para la implementación.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

> [!NOTE]
> Las plantillas pueden incluir definiciones de recursos para varias aplicaciones lógicas, por lo que debe asegurarse de que todos los recursos de la aplicación lógica especifican el mismo grupo de recursos de Azure. Al implementar la plantilla en un grupo de recursos de Azure mediante Visual Studio, se le pedirá que especifique la aplicación lógica que quiere abrir. Además, el proyecto del grupo de recursos de Azure puede contener más de una plantilla, por lo que debe asegurarse de seleccionar el archivo de parámetros correcto cuando se le solicite.

<a name="view-resource-definitions"></a>

### <a name="view-resource-definitions"></a>Visualización de definiciones de recursos

Para revisar las definiciones de recurso de todos los recursos de un grupo de recursos de Azure, [descargue la aplicación lógica de Azure en Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md), que es la forma más sencilla de crear una plantilla válida de aplicación lógica con parámetros que está prácticamente lista para la implementación.

Para obtener información general sobre los recursos de plantilla y sus atributos, consulte estos temas:

* [Recursos en Estructura y sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#resources)
* [Procedimientos recomendados para los recursos de plantilla](../azure-resource-manager/templates/template-best-practices.md#resources)

<a name="logic-app-resource-definition"></a>

### <a name="logic-app-resource-definition"></a>Definición de recursos de la aplicación lógica

La definición de recursos de la aplicación lógica se inicia con el objeto `properties`, que incluye esta información:

* El estado de la aplicación lógica en la implementación
* El identificador de cualquier cuenta de integración que use la aplicación lógica
* La definición de flujo de trabajo de la aplicación lógica
* Un objeto `parameters` que establece los valores que se van a usar en tiempo de ejecución
* Otra información sobre los recursos de la aplicación lógica, como el nombre, el tipo, la ubicación, etc.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            "state": "<Enabled or Disabled>",
            "integrationAccount": {
               "id": "[parameters('LogicAppIntegrationAccount')]" // Template parameter reference
            },
            "definition": {<workflow-definition>},
            "parameters": {<workflow-definition-parameter-values>},
            "accessControl": {}
         },
         "name": "[parameters('LogicAppName')]", // Template parameter reference
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]", // Template parameter reference
         "tags": {
           "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
         ]
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

Estos son los atributos que son específicos de la definición de recursos de la aplicación lógica:

| Atributo | Obligatorio | Tipo | Descripción |
|-----------|----------|------|-------------|
| `state` | Sí | String | El estado de la aplicación lógica en la implementación, en que `Enabled` significa que la aplicación lógica está activa y `Disabled` significa que la aplicación lógica está inactiva. Por ejemplo, si no está listo para que la aplicación lógica pase a estar activa, pero quiere implementar una versión de borrador, puede usar la opción `Disabled`. |
| `integrationAccount` | No | Object | Si la aplicación lógica usa una cuenta de integración, que almacena artefactos para escenarios negocio a negocio (B2B), este objeto incluye el atributo `id`, que especifica el identificador de la cuenta de integración. |
| `definition` | Sí | Object | La definición de flujo de trabajo subyacente de la aplicación lógica, que es el mismo objeto que aparece en la vista de código y se describe detalladamente en el tema [Referencia de esquema del lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md). En esta definición de flujo de trabajo, el objeto `parameters` declara los parámetros de los valores que se van a usar en el tiempo de ejecución de la aplicación lógica. Para obtener más información, consulte [Definición y parámetros del flujo de trabajo](#workflow-definition-parameters). <p><p>Para ver los atributos de la definición de flujo de trabajo de la aplicación lógica, cambie de la "vista de diseño" a la "vista de código" en Azure Portal o Visual Studio, o mediante una herramienta como [Azure Resource Explorer](https://resources.azure.com). |
| `parameters` | No | Object | [Valores del parámetro de definición de flujo de trabajo](#workflow-definition-parameters) que se usarán en el tiempo de ejecución de la aplicación lógica. Las definiciones de los parámetros de estos valores se muestran dentro del [objeto de parámetros de la definición de flujo de trabajo](#workflow-definition-parameters). Además, si la aplicación lógica usa [conectores administrados](../connectors/apis-list.md) para acceder a otros servicios y sistemas, este objeto incluye un objeto `$connections` que establece los valores de conexión que se usarán en tiempo de ejecución. |
| `accessControl` | No | Object | Se usa para especificar los atributos de seguridad de la aplicación lógica, por ejemplo, para restringir el acceso IP a los desencadenadores de solicitud o ejecutar entradas y salidas del historial. Para obtener más información, consulte [Protección del acceso a las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md). |
||||

Para obtener información sobre los recursos de plantilla específicos de Logic Apps, las cuentas de integración y los artefactos de la cuenta de integración, consulte [Tipos de recursos de Microsoft.Logic](/azure/templates/microsoft.logic/allversions).

<a name="workflow-definition-parameters"></a>

## <a name="workflow-definition-and-parameters"></a>Definición y parámetros del flujo de trabajo

La definición de flujo de trabajo de la aplicación lógica aparece en el objeto `definition`, que se muestra en el objeto `properties` que hay dentro de la definición de recursos de la aplicación lógica. Este objeto `definition` es el mismo que se muestra en la vista de código y se describe detalladamente en el tema [Referencia de esquema del lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md). La definición de flujo de trabajo incluye un objeto de declaración `parameters` interno en el que puede definir nuevos parámetros o modificar los existentes para los valores que usa la definición de flujo de trabajo en tiempo de ejecución. Después, puede hacer referencia a estos parámetros en el desencadenador o las acciones del flujo de trabajo. De forma predeterminada, este objeto `parameters` está vacío a menos que la aplicación lógica cree conexiones a otros servicios y sistemas mediante [conectores administrados](../connectors/apis-list.md).

Para establecer los valores de los parámetros de la definición de flujo de trabajo, use el objeto `parameters` que está *fuera* de la definición de flujo de trabajo, pero que sigue *dentro* de la definición de recursos de la aplicación lógica. Luego, en este objeto `parameters` externo, puede hacer referencia a los parámetros de plantilla declarados previamente, que pueden aceptar valores en la implementación de un archivo de parámetros.

> [!TIP]
>
> Como procedimiento recomendado, no haga referencia directamente a los parámetros de plantilla, que se evalúan en la implementación, desde dentro de la definición de flujo de trabajo. En su lugar, declare un parámetro de definición de flujo de trabajo, que puede establecer en el objeto `parameters` que está *fuera* de la definición de flujo de trabajo, pero que sigue *dentro* de la definición de recursos de la aplicación lógica. Para obtener más información, consulte la sección [Referencias a parámetros](#parameter-references).

En esta sintaxis se muestra dónde puede declarar parámetros en los niveles de definición de plantilla y flujo de trabajo, así como dónde puede establecer esos valores de parámetros haciendo referencia a los parámetros de definición de flujo de trabajo y plantilla:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "<template-parameter-name>": {
         "type": "<parameter-type>",
         "defaultValue": "<parameter-default-value>",
         "metadata": {
            "description": "<parameter-description>"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "<workflow-definition-parameter-name>": {
                     "type": "<parameter-type>",
                     "defaultValue": "<parameter-default-value>",
                     "metadata": {
                        "description": "<parameter-description>"
                     }
                  }
               },
               "triggers": {
                  "<trigger-name>": {
                     "type": "<trigger-type>",
                     "inputs": {
                         // Workflow definition parameter reference
                         "<attribute-name>": "@parameters('<workflow-definition-parameter-name')"
                     }
                  }
               },
               <...>
            },
            // Workflow definition parameter value
            "parameters": {
               "<workflow-definition-parameter-name>": { 
                  "value": "[parameters('<template-parameter-name>')]"
               }
            },
            "accessControl": {}
         },
         <other-logic-app-resource-definition-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

<a name="secure-workflow-definition-parmameters"></a>

### <a name="secure-workflow-definition-parameters"></a>Protección de los parámetros de definición de flujo de trabajo

Si tiene un parámetro de definición de flujo de trabajo que controla información confidencial, contraseñas, claves de acceso o secretos en tiempo de ejecución, declárelo o edítelo para usar el tipo de parámetro `securestring` o `secureobject`. Puede hacer referencia a este parámetro en la definición de flujo de trabajo y mediante esta. En el nivel superior de la plantilla, declare un parámetro que tenga el mismo tipo a fin de controlar esta información en la implementación.

Para establecer el valor del parámetro de la definición de flujo de trabajo, use el objeto `parameters` que está *fuera* de la definición de flujo de trabajo, pero que sigue *dentro* de la definición de recursos de la aplicación lógica, con el fin de hacer referencia al parámetro de la plantilla. Por último, para pasar el valor al parámetro de plantilla en la implementación, almacénelo en [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) y haga referencia a ese almacén de claves en el [archivo de parámetros](#template-parameter-files) que utiliza la plantilla en la implementación.

En esta plantilla de ejemplo, se muestra cómo puede completar estas tareas mediante la definición de parámetros seguros cuando sea necesario a fin de almacenar sus valores en Azure Key Vault:

* Declare parámetros seguros para los valores usados para autenticar el acceso.
* Use estos valores en los niveles de definición de plantilla y flujo de trabajo.
* Proporcione estos valores mediante un archivo de parámetros.

**Plantilla**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      <previously-defined-template-parameters>,
      // Additional template parameters for passing values to use in workflow definition
      "TemplateAuthenticationType": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The type of authentication used for the Fabrikam portal"
         }
      },
      "TemplateFabrikamPassword": {
         "type": "securestring",
         "metadata": {
            "description": "The password for the Fabrikam portal"
         }
      },
      "TemplateFabrikamUserName": {
         "type": "securestring",
         "metadata": {
            "description": "The username for the Fabrikam portal"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            // Start workflow definition
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "authenticationType": {
                     "type": "string",
                     "defaultValue": "",
                     "metadata": {
                        "description": "The type of authentication used for the Fabrikam portal"
                     }
                  },
                  "fabrikamPassword": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The password for the Fabrikam portal"
                     }
                  },
                  "fabrikamUserName": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The username for the Fabrikam portal"
                     }
                  }
               },
               "triggers": {
                  "HTTP": {
                     "inputs": {
                        "authentication": {
                           // Reference workflow definition parameters
                           "password": "@parameters('fabrikamPassword')",
                           "type": "@parameters('authenticationType')",
                           "username": "@parameters('fabrikamUserName')"
                        }
                     },
                     "recurrence": {<...>},
                     "type": "Http"
                  }
               },
               <...>
            },
            // End workflow definition
            // Start workflow definition parameter values
            "parameters": {
               "authenticationType": "[parameters('TemplateAuthenticationType')]", // Template parameter reference
               "fabrikamPassword": "[parameters('TemplateFabrikamPassword')]", // Template parameter reference
               "fabrikamUserName": "[parameters('TemplateFabrikamUserName')]" // Template parameter reference
            },
            "accessControl": {}
         },
         <other-logic-app-resource-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

**Archivo de parámetros**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      <previously-defined-template-parameter-values>,
     "TemplateAuthenticationType": {
        "value": "Basic"
     },
     "TemplateFabrikamPassword": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamPassword"
        }
     },
     "TemplateFabrikamUserName": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamUserName"
        }
     }
   }
}
```

<a name="best-practices-workflow-definition-parameters"></a>

## <a name="best-practices---workflow-definition-parameters"></a>Procedimientos recomendados: parámetros de definición de flujo de trabajo

Para asegurarse de que el Diseñador de aplicaciones lógicas puede mostrar correctamente los parámetros de definición de flujo de trabajo, siga estos procedimientos recomendados:

* Incluya el atributo `defaultValue`, que puede especificar valores vacíos, para todos los parámetros excepto los valores que sean confidenciales o que se deban proteger.

* Use siempre parámetros seguros para los nombres de usuario, las contraseñas y los secretos. Para ocultar o proteger los valores de parámetros confidenciales, siga las instrucciones de estos temas:

  * [Recomendaciones de seguridad para los parámetros de acción](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)

  * [Recomendaciones de seguridad para los parámetros de las definiciones de flujo de trabajo](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-workflow)

  * [Pasar valores de parámetros seguros con Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

Para obtener más información sobre los parámetros de definición de flujo de trabajo,consulte la sección [Parámetros en el artículo Lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md#parameters).

<a name="connection-resource-definitions"></a>

## <a name="connection-resource-definitions"></a>Definiciones de recursos de conexión

Cuando la aplicación lógica crea y usa conexiones a otros servicios y sistemas mediante el uso de [conectores administrados](../connectors/apis-list.md), el objeto `resources` de la plantilla contiene las definiciones de recursos para esas conexiones. Aunque cree conexiones desde una aplicación lógica, estas son recursos de Azure independientes con sus propias definiciones de recursos. Para revisar estas definiciones de recursos de conexión, puede [descargar la aplicación lógica desde Azure en Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md), que es la forma más sencilla de crear una plantilla válida de aplicación lógica con parámetros prácticamente lista para la implementación.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

Las definiciones de recursos de conexión hacen referencia a los parámetros de nivel superior de la plantilla de sus valores, lo que significa que puede proporcionar estos valores durante la implementación mediante un archivo de parámetros. Asegúrese de que las conexiones usen el mismo grupo de recursos y la misma ubicación de Azure que la aplicación lógica.

Este es un ejemplo de una definición de recursos para una conexión de Office 365 Outlook y los parámetros de plantilla correspondientes:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name for the Office 365 Outlook connection"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

La definición de recursos de la aplicación lógica también funciona con las definiciones de recursos de conexión de las siguientes maneras:

* En esta definición de flujo de trabajo, el objeto `parameters` declara un parámetro `$connections` de los valores de conexión que se van a usar en el tiempo de ejecución de la aplicación lógica. Además, el desencadenador o la acción que crea una conexión utiliza los valores correspondientes que pasan a través de este parámetro `$connections`.

* *Fuera* de la definición de flujo de trabajo, pero *dentro* de la definición de recursos de la aplicación lógica, otro objeto `parameters` establece los valores que se van a usar en tiempo de ejecución para el parámetro `$connections` haciendo referencia a los parámetros de la plantilla correspondientes. Estos valores usan expresiones de plantilla para hacer referencia a los recursos que almacenan de forma segura los metadatos de las conexiones en la aplicación lógica.

  Por ejemplo, los metadatos pueden incluir cadenas de conexión y tokens de acceso, que puede almacenar en [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md). Para pasar esos valores a los parámetros de plantilla, debe hacer referencia a ese almacén de claves en el [archivo de parámetros](#template-parameter-files) que utiliza su plantilla en la implementación. Para obtener más información sobre estas diferencias en la referencia de parámetros, consulte [Referencias a parámetros](#parameter-references) más adelante en este tema.

  Al abrir la definición de flujo de trabajo de la aplicación lógica en la vista de código mediante Azure Portal o Visual Studio, el objeto `$connections` aparece fuera de la definición de flujo de trabajo, pero en el mismo nivel. Esta ordenación en la vista de código permite hacer referencia a estos parámetros de forma más fácil al actualizar manualmente la definición de flujo de trabajo:

  ```json
  {
     "$connections": {<workflow-definition-parameter-connection-values-runtime},
     "definition": {<workflow-definition>}
  }
  ```

* La definición de recursos de la aplicación lógica tiene un objeto `dependsOn` que especifica las dependencias en las conexiones que usa la aplicación lógica.

Cada conexión que crea tiene un nombre único en Azure. Cuando crea varias conexiones en el mismo servicio o sistema, cada nombre de conexión se anexa con un número, que se incrementa con cada nueva conexión que se crea, por ejemplo, `office365`, `office365-1`, etc.

En este ejemplo se muestran las interacciones entre la definición de recursos de la aplicación lógica y una definición de recursos de conexión para Office 365 Outlook:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {<parameter-definition>},
      "office365_1_Connection_DisplayName": {<parameter-definition>}
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <...>,
            "definition": {
               <...>,
               "parameters": {
                  // Workflow definition "$connections" parameter
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               <...>
            },
            "parameters": {
               // Workflow definition "$connections" parameter values to use at runtime
               "$connections": {
                  "value": {
                     "office365": {
                        // Template parameter references
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            },
            <other-logic-app-resource-information>,
            "dependsOn": [
               "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
            ]
         }
         // End logic app resource definition
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-connection-parameters"></a>

### <a name="secure-connection-parameters"></a>Protección de los parámetros de conexión

Si tiene un parámetro de conexión que controla la información confidencial, contraseñas, claves de acceso o secretos, la definición de recursos de la conexión incluye un objeto `parameterValues` que especifica estos valores en el formato de par nombre-valor. Para ocultar esta información, puede declarar o editar los parámetros de plantilla de estos valores mediante los tipos de parámetro `securestring` o `secureobject`. Después, puede almacenar esa información en [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md). Para pasar esos valores a los parámetros de plantilla, debe hacer referencia a ese almacén de claves en el [archivo de parámetros](#template-parameter-files) que utiliza su plantilla en la implementación.

En el ejemplo siguiente, se proporciona el nombre de la cuenta y la clave de acceso de una conexión de Azure Blob Storage:

**Archivo de parámetros**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      },
      "azureblob_1_Connection_Name": {
         "value": "Fabrikam-Azure-Blob-Storage-Connection"
      },
      "azureblob_1_Connection_DisplayName": {
         "value": "Fabrikam-Storage"
      },
      "azureblob_1_accountName": {
         "value": "Fabrikam-Storage-Account"
      },
      "azureblob_1_accessKey": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
            },
            "secretName": "FabrikamStorageKey"
         }
      }
   }
}
```

**Plantilla**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "azureblob_1_Connection_Name": {<parameter-definition>},
      "azureblob_1_Connection_DisplayName": {<parameter-definition>},
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "secureobject",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  // Azure Blob Storage action
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              // Workflow definition parameter reference for values to use at runtime
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  // Workflow definition parameter for values to use at runtime
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {<trigger-definition>},
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     // Template parameter references for values to use at runtime
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                    }
                  }
               }
            },
            "name": "[parameters('LogicAppName')]",
            "type": "Microsoft.Logic/workflows",
            "location": "[parameters('LogicAppLocation')]",
            "tags": {
               "displayName": "LogicApp"
            },
            "apiVersion": "2016-06-01",
            // Template parameter reference for value to use at deployment
            "dependsOn": [
               "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]"
            ]
         }
      },
      // Azure Blob Storage API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            // Template parameter reference for values to use at deployment
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="authenticate-connections"></a>

### <a name="authenticate-connections"></a>Autenticar conexiones

Después de la implementación, la aplicación lógica funciona por completo con parámetros válidos. Aun así, igualmente debe autorizar las conexiones de OAuth para generar tokens de acceso válidos a fin de [autenticar las credenciales](../active-directory/develop/authentication-vs-authorization.md). Para obtener más información, consulte el artículo [Autorización de conexiones de OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections).

Algunas conexiones admiten el uso de una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md) de Azure Active Directory (Azure AD) para autorizar conexiones para una aplicación lógica [registrada en Azure AD](../active-directory/develop/quickstart-register-app.md). Por ejemplo, en esta definición de recursos de conexión de Azure Data Lake se muestra cómo se debe referencia a los parámetros de plantilla que controlan la información de la entidad de servicio y cómo la plantilla declara estos parámetros:

**Definición de recursos de conexión**

```json
{
   <other-template-objects>
   "type": "MICROSOFT.WEB/CONNECTIONS",
   "apiVersion": "2016-06-01",
   "name": "[parameters('azuredatalake_1_Connection_Name')]",
   "location": "[parameters('LogicAppLocation')]",
   "properties": {
      "api": {
         "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', 'resourceGroup().location', '/managedApis/', 'azuredatalake')]"
      },
      "displayName": "[parameters('azuredatalake_1_Connection_DisplayName')]",
      "parameterValues": {
         "token:clientId": "[parameters('azuredatalake_1_token:clientId')]",
         "token:clientSecret": "[parameters('azuredatalake_1_token:clientSecret')]",
         "token:TenantId": "[parameters('azuredatalake_1_token:TenantId')]",
         "token:grantType": "[parameters('azuredatalake_1_token:grantType')]"
      }
   }
}
```

| Atributo | Descripción |
|-----------|-------------|
| `token:clientId` | Identificador de cliente o aplicación asociado a la entidad de servicio |
| `token:clientSecret` | Valor de clave asociado a la entidad de servicio |
| `token:TenantId` | Identificador del directorio de su inquilino de Azure AD |
| `token:grantType` | Tipo de concesión solicitado, que debe ser `client_credentials`. Para obtener más información, consulte [La Plataforma de identidad de Microsoft y el flujo de credenciales de cliente de OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). |
|||

**Definiciones de parámetros de plantilla**

El objeto `parameters` de nivel superior de la plantilla declara estos parámetros para la conexión de ejemplo:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "azuredatalake_1_Connection_Name": {
        "type": "string",
        "defaultValue": "azuredatalake"
      },
      "azuredatalake_1_Connection_DisplayName": {
        "type": "string",
        "defaultValue": "<connection-name>"
      },
      "azuredatalake_1_token:clientId": {
        "type": "securestring",
        "metadata": {
          "description": "Client (or Application) ID of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:clientSecret": {
        "type": "securestring",
        "metadata": {
          "description": "Client secret of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:TenantId": {
        "type": "securestring",
        "metadata": {
          "description": "The tenant ID of for the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:resourceUri": {
        "type": "string",
        "metadata": {
          "description": "The resource you are requesting authorization to use."
        }
      },
      "azuredatalake_1_token:grantType": {
        "type": "string",
        "metadata": {
          "description": "Grant type"
        },
        "defaultValue": "client_credentials",
        "allowedValues": [
          "client_credentials"
        ]
      },
      // Other template parameters
   }
   // Other template objects
}
```

Para obtener más información sobre cómo trabajar con entidades de servicio, consulte estos temas:

* [Create a service principal by using the Azure portal](../active-directory/develop/howto-create-service-principal-portal.md) (Uso de Azure Portal para crear una entidad de servicio)
* [Creación de una entidad de servicio de Azure con Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps)
* [Creación de una entidad de servicio con un certificado mediante Azure PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

<a name="parameter-references"></a>

## <a name="references-to-parameters"></a>Referencias a parámetros

Para hacer referencia a los parámetros de plantilla, puede usar expresiones de plantilla con [funciones de plantilla](../azure-resource-manager/templates/template-functions.md), que se evalúan en la implementación. Las expresiones de plantilla usan corchetes (**[]**):

`"<attribute-name>": "[parameters('<template-parameter-name>')]"`

Para hacer referencia a parámetros de definición de flujo de trabajo, debe usar [funciones y expresiones del lenguaje de definición de flujo de trabajo](../logic-apps/workflow-definition-language-functions-reference.md), que se evalúan en tiempo de ejecución. Es posible que observe que algunas funciones de plantilla y de definición de flujo de trabajo tienen el mismo nombre. Las expresiones de definición de flujo de trabajo comienzan por el símbolo "arroba" ( **@** ):

`"<attribute-name>": "@parameters('<workflow-definition-parameter-name>')"`

Puede pasar valores de parámetro de plantilla a la definición de flujo de trabajo para que la aplicación lógica los use en tiempo de ejecución. Sin embargo, evite usar parámetros de plantilla, expresiones y sintaxis en la definición de flujo de trabajo, ya que el Diseñador de aplicaciones lógicas no admite elementos de plantilla. Así mismo, las expresiones y la sintaxis de las plantillas pueden complicar el código debido a las diferencias en el momento en que se evalúan las expresiones.

En su lugar, siga estos pasos generales para declarar los parámetros de definición de flujo de trabajo que se van a usar en tiempo de ejecución, así como hacerles referencia; para declarar los parámetros de plantilla que se van a usar en la implementación, así como hacerles referencia; y para especificar los valores que se van a pasar en la implementación mediante un archivo de parámetros. Para obtener todos los detalles, consulte la sección [Definición y parámetros del flujo de trabajo](#workflow-definition-parameters) incluida anteriormente en este tema.

1. Cree la plantilla y declare los parámetros de plantilla de los valores que aceptará y usará en la implementación.

1. En la definición de flujo de trabajo, declare los parámetros de los valores que se van a aceptar y usar en tiempo de ejecución. Puede hacer referencia a estos valores en la definición de flujo de trabajo y mediante esta.

1. En el objeto `parameters` que está *fuera* de la definición de flujo de trabajo, pero que todavía está *dentro* de la definición de recursos de la aplicación lógica, establezca los valores de los parámetros de la definición de flujo de trabajo haciendo referencia los parámetros de plantilla correspondientes. De este modo, podrá pasar valores de parámetros de plantilla a los parámetros de definición de flujo de trabajo.

1. En el archivo de parámetros, especifique los valores de la plantilla que se usarán en la implementación.

<a name="full-example-template"></a>

## <a name="full-example-template"></a>Ejemplo completo de una plantilla

Esta es la plantilla de ejemplo parametrizada que se usa en los ejemplos de este tema:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name to use for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location to use for the logic app"
         }
      },
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name to use for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name to use for the Office 365 Outlook connection"
         }
      },
      "azureblob_1_Connection_Name": {
         "type": "string",
         "defaultValue": "azureblob",
         "metadata": {
            "description": "The resource name to use for the Azure Blob storage account connection"
         }
      },
      "azureblob_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         },

      },
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "securestring",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "integrationAccount": {
              "id": "[parameters('LogicAppIntegrationAccount')]"
            },
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {
                  "When_a_new_email_arrives": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['office365']['connectionId']"
                           }
                        },
                        "method": "get",
                        "path": "/Mail/OnNewEmail",
                        "queries": {
                           "folderPath": "Inbox",
                           "importance": "Any",
                           "fetchOnlyWithAttachment": false,
                           "includeAttachments": false
                        }
                     },
                     "recurrence": {
                        "frequency": "Day",
                        "interval": 1
                     },
                     "splitOn": "@triggerBody()?['value']"
                  }
               },
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                     },
                     "office365": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            },
            "accessControl": {}
         },
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
            "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
            "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
         ]
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('office365_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
                "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de plantillas de aplicaciones lógicas](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
