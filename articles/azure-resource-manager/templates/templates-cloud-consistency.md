---
title: Reutilización de plantillas entre nubes
description: Desarrollar plantillas de Azure Resource Manager que funcionen de forma coherente para distintos entornos en la nube. Cree o actualice plantillas existentes de Azure Stack.
author: marcvaneijk
ms.topic: conceptual
ms.date: 12/09/2018
ms.author: mavane
ms.custom: seodec18
ms.openlocfilehash: 9355482c26cabb96fc6292bab5d542f36aec6a8c
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509763"
---
# <a name="develop-arm-templates-for-cloud-consistency"></a>Desarrollo de plantillas de ARM para la coherencia en la nube

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

Una ventaja clave de Azure es la coherencia. Las inversiones de desarrollo para una ubicación son reutilizables en otra. Una plantilla de Azure Resource Manager (ARM) hace que las implementaciones sean coherentes y repetibles entre los entornos, incluidos Azure global, las nubes soberanas de Azure y Azure Stack. Pero para reutilizar plantillas en las nubes, debe tener en cuenta las dependencias específicas de la nube como se explica en esta guía.

Microsoft ofrece servicios en la nube inteligentes y preparados para la empresa en varias ubicaciones, incluidas:

* La plataforma Azure global admitida por una red cada vez mayor de centros de datos administrados por Microsoft en regiones de todo el mundo.
* Nubes soberanas aisladas como Azure Alemania, Azure Government y Azure China 21Vianet. Las nubes soberanas proporcionan una plataforma coherente con la mayoría de las mismas características de calidad a las que tienen acceso los clientes de Azure global.
* Azure Stack, una plataforma en la nube híbrida que permite proporcionar servicios de Azure desde el centro de datos de la organización. Las empresas pueden configurar Azure Stack en sus propios centros de datos o consumir servicios de Azure de proveedores de servicios, que ejecutan Azure Stack en sus instalaciones (a veces conocidas como regiones hospedadas).

En el núcleo de todas estas nubes, Azure Resource Manager proporciona una API que permite que una amplia variedad de interfaces de usuario se comunique con la plataforma Azure. Esta API ofrece funcionalidades de infraestructura como código eficaces. Cualquier tipo de recurso que esté disponible en la plataforma de nube de Azure se puede implementar y configurar con Azure Resource Manager. Con una única plantilla, puede implementar y configurar la aplicación completa a un estado operativo final.

![Entornos de Azure](./media/templates-cloud-consistency/environments.png)

La coherencia de Azure global, las nubes soberanas, las nubes hospedadas y una nube en el centro de datos ayuda a beneficiarse de Azure Resource Manager. Puede reutilizar las inversiones de desarrollo en todas estas nubes al configurar la implementación y configuración de recursos basados en plantillas.

Pero aunque las nubes global, soberanas, hospedadas e híbridas proporcionan servicios coherentes, no todas son idénticas. Como consecuencia, puede crear una plantilla con dependencias de las características disponibles solo en una nube específica.

En el resto de esta guía se describen las áreas que se deben tener en cuenta cuando se planea el desarrollo de plantillas nuevas de ARM o la actualización de las existentes para Azure Stack. En general, la lista de comprobación debe incluir los elementos siguientes:

* Compruebe que las funciones, los puntos de conexión, los servicios y otros recursos de la plantilla están disponibles en las ubicaciones de implementación de destino.
* Almacene las plantillas anidadas y los artefactos de configuración en ubicaciones accesibles, lo que garantiza el acceso entre las nubes.
* Use referencias dinámicas en lugar de vínculos y elementos codificados de forma rígida.
* Asegúrese de que los parámetros de plantilla que usa funcionan en las nubes de destino.
* Compruebe que las propiedades específicas de los recursos están disponibles en las nubes de destino.

Para ver una introducción a las plantillas de ARM, consulte [Template Deployment](overview.md).

## <a name="ensure-template-functions-work"></a>Garantizar el funcionamiento de la plantilla

La sintaxis básica de la plantilla de ARM es JSON. Las plantillas usan un superconjunto de JSON, que amplía la sintaxis con expresiones y funciones. El procesador de lenguaje de plantilla se actualiza con frecuencia para admitir funciones de plantilla adicionales. Para obtener una explicación detallada de las funciones de plantilla disponibles, consulte [Funciones de plantilla de ARM](template-functions.md).

Las funciones de plantilla nuevas que se agregan a Azure Resource Manager no están disponibles inmediatamente en las nubes soberanas o Azure Stack. Para implementar correctamente una plantilla, todas las funciones a las que se hace referencia en la plantilla deben estar disponibles en la nube de destino.

Las funcionalidades de Azure Resource Manager siempre se introducirán primero en Azure global. Puede usar el script de PowerShell siguiente para comprobar si las funciones de plantilla recién introducidas también están disponibles en Azure Stack:

1. Clone el repositorio de GitHub: [https://github.com/marcvaneijk/arm-template-functions](https://github.com/marcvaneijk/arm-template-functions).

1. Una vez que tenga un clon del repositorio local, conéctese a la instancia de Azure Resource Manager de destino con PowerShell.

1. Importe el módulo psm1 y ejecute el cmdlet Test-AzureRmTemplateFunctions:

   ```powershell
   # Import the module
   Import-module <path to local clone>\AzTemplateFunctions.psm1

   # Execute the Test-AzureRmTemplateFunctions cmdlet
   Test-AzureRmTemplateFunctions -path <path to local clone>
   ```

El script implementa varias plantillas minimizadas, cada una solo con funciones de plantilla únicas. En la salida del script se informa de las funciones de plantilla admitidas y las que no están disponibles.

## <a name="working-with-linked-artifacts"></a>Trabajar con artefactos vinculados

Una plantilla puede contener referencias a artefactos vinculados y contiene un recurso de implementación que se vincula a otra plantilla. Las plantillas vinculadas (también denominadas plantillas anidadas) se recuperan mediante Resource Manager en tiempo de ejecución. Una plantilla también puede contener referencias a artefactos para extensiones de máquina virtual (VM). Estos artefactos se recuperan mediante la extensión de máquina virtual que se ejecuta dentro de la máquina virtual para la configuración de la extensión de máquina virtual durante la implementación de plantilla.

En las secciones siguientes se describen las consideraciones para mantener la coherencia en la nube al desarrollar plantillas que incluyen artefactos externos a la plantilla de implementación principal.

### <a name="use-nested-templates-across-regions"></a>Usar plantillas anidadas entre regiones

Las plantillas se pueden descomponer en pequeñas plantillas reutilizables, cada una con un propósito específico, y se pueden reutilizar en escenarios de implementación. Para ejecutar una implementación, especifique una sola plantilla conocida como plantilla principal o maestra. Especifica los recursos que se van a implementar, como redes virtuales, máquinas virtuales y aplicaciones web. La plantilla principal también puede contener un vínculo a otra plantilla, lo que significa que las plantillas se pueden anidar. Del mismo modo, una plantilla anidada puede contener vínculos a otras plantillas. Se pueden anidar hasta cinco niveles de profundidad.

En el código siguiente se muestra cómo el parámetro templateLink hace referencia a una plantilla anidada:

```json
"resources": [
  {
     "type": "Microsoft.Resources/deployments",
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/vNet.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

Azure Resource Manager evalúa la plantilla principal en tiempo de ejecución y recupera y evalúa cada plantilla anidada. Después de recuperar todas las plantillas anidadas, la plantilla se simplifica y se inicia el procesamiento posterior.

### <a name="make-linked-templates-accessible-across-clouds"></a>Hacer que las plantillas vinculadas sean accesibles entre las nubes

Tenga en cuenta dónde y cómo almacenar las plantillas vinculadas que usa. En tiempo de ejecución, Azure Resource Manager captura todas las plantillas vinculadas y, por tanto, requiere acceso directo a ellas. Una práctica común consiste en usar GitHub para almacenar las plantillas anidadas. Un repositorio de GitHub puede contener archivos que son accesibles públicamente a través de una dirección URL. Aunque esta técnica funciona bien en la nube pública y las nubes soberanas, es posible que un entorno de Azure Stack se encuentre en una red corporativa o en una ubicación remota desconectada, sin ningún acceso de salida a Internet. En esos casos, Azure Resource Manager no podría recuperar las plantillas anidadas.

Un procedimiento mejor para las implementaciones entre nubes consiste en almacenar las plantillas vinculadas en una ubicación que sea accesible para la nube de destino. Lo ideal es que todos los artefactos de implementación se mantengan y se implementen desde una canalización de integración continua y desarrollo continuo (CI/CD). Como alternativa, puede almacenar las plantillas anidadas en un contenedor de Blob Storage, desde el que Azure Resource Manager puede recuperarlas.

Como Blob Storage usa un nombre de dominio completo (FQDN) de punto de conexión distinto en cada nube, debe configurar la plantilla con la ubicación de las plantillas vinculadas con dos parámetros. Los parámetros pueden aceptar la entrada del usuario en tiempo de implementación. Normalmente, varias personas crean y comparten las plantillas, por lo que un procedimiento recomendado consiste en usar un nombre estándar para estos parámetros. Las convenciones de nomenclatura ayudan a que las plantillas sean más reutilizables entre regiones, nubes y autores.

En el código siguiente, se usa `_artifactsLocation` para apuntar a una única ubicación, que contiene todos los artefactos relacionados con la implementación. Tenga en cuenta que se proporciona un valor predeterminado. En tiempo de implementación, si no se especifica ningún valor de entrada para `_artifactsLocation`, se usa el valor predeterminado. Se usa `_artifactsLocationSasToken` como entrada para `sasToken`. El valor predeterminado debe ser una cadena vacía para escenarios donde `_artifactsLocation` no es seguro, por ejemplo, un repositorio público de GitHub.

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-custom-script-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

A lo largo de la plantilla, se generan vínculos mediante la combinación del URI base (del parámetro `_artifactsLocation`) con una ruta de acceso relativa del artefacto y `_artifactsLocationSasToken`. En el código siguiente se muestra cómo especificar el vínculo a la plantilla anidada con la función de plantilla uri:

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "shared",
    "properties": {
      "mode": "Incremental",
      "templateLink": {
        "uri": "[uri(parameters('_artifactsLocation'), concat('nested/vnet.json', parameters('_artifactsLocationSasToken')))]",
        "contentVersion": "1.0.0.0"
      }
    }
  }
]
```

Con este enfoque, se usa el valor predeterminado para el parámetro `_artifactsLocation`. Si es necesario recuperar las plantillas vinculadas desde otra ubicación, se puede usar el parámetro de entrada en tiempo de implementación para reemplazar el valor predeterminado: no se necesita ningún cambio en la propia plantilla.

### <a name="use-_artifactslocation-instead-of-hardcoding-links"></a>Usar _artifactsLocation en lugar de vínculos codificados de forma rígida

Además de usarse en las plantillas anidadas, la dirección URL del parámetro `_artifactsLocation` se usa como base para todos los artefactos relacionados de una plantilla de implementación. Algunas extensiones de máquina virtual incluyen un vínculo a un script almacenado fuera de la plantilla. Para estas extensiones, los vínculos no se deben codificar de forma rígida. Por ejemplo, las extensiones Custom Script y PowerShell DSC pueden vincular a un script externo en GitHub, como se muestra a continuación:

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
    ]
  }
}
```

Codificar de forma rígida los vínculos en el script impide potencialmente que la plantilla se implemente correctamente en otra ubicación. Durante la configuración del recurso de máquina virtual, el agente de máquina virtual que se ejecuta dentro de la máquina virtual inicia una descarga de todos los scripts vinculados en la extensión de máquina virtual y, después, almacena los scripts en el disco local de la máquina virtual. Este enfoque funciona como los vínculos de la plantilla anidada que se explicaron anteriormente en la sección "Usar plantillas anidadas entre regiones".

Resource Manager recupera las plantillas anidadas en tiempo de ejecución. Para las extensiones de máquina virtual, la recuperación de los artefactos externos se realiza mediante el agente de máquina virtual. Aparte del iniciador diferente de la recuperación de artefactos, la solución en la definición de plantilla es la misma. Use el parámetro _artifactsLocation con un valor predeterminado de la ruta de acceso base donde se almacenan todos los artefactos (incluidos los scripts de extensión de máquina virtual) y el parámetro `_artifactsLocationSasToken` para la entrada para sasToken.

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

Para construir el URI absoluto de un artefacto, el método preferido es usar la función de plantilla uri, en lugar de la función de plantilla concat. Al reemplazar vínculos codificados de forma rígida en los scripts de la extensión de máquina virtual con la función de plantilla uri, esta funcionalidad de la plantilla se configura para mantener la coherencia en la nube.

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "[uri(parameters('_artifactsLocation'), concat('scripts/configure-music-app.ps1', parameters('_artifactsLocationSasToken')))]"
    ]
  }
}
```

Con este enfoque, todos los artefactos de implementación, incluidos los scripts de configuración, se pueden almacenar en la misma ubicación con la plantilla propiamente dicha. Para cambiar la ubicación de todos los vínculos, basta con especificar otra dirección URL base para los _parámetros artifactsLocation_.

## <a name="factor-in-differing-regional-capabilities"></a>Tener en cuenta la diferencia de las funcionalidades regionales

Con el desarrollo ágil y el flujo continuo de actualizaciones y servicios nuevos que se introducen en Azure, la disponibilidad de los servicios o las actualizaciones en [las regiones puede diferir](https://azure.microsoft.com/regions/services/). Después de rigurosas pruebas internas, los servicios nuevos o las actualizaciones de los servicios existentes se presentan normalmente a un pequeño grupo de clientes que participan en un programa de validación. Después de la validación correcta de los clientes, los servicios o las actualizaciones se ponen a disposición en un subconjunto de regiones de Azure, después se introducen en más regiones, se implementan en las nubes soberanas y, potencialmente, también se ponen a disposición de los clientes de Azure Stack.

Sabiendo que las nubes y las regiones de Azure pueden diferir en sus servicios disponibles, puede tomar algunas decisiones proactivas con respecto a las plantillas. Un buen lugar para comenzar consiste en examinar los proveedores de recursos disponibles para una nube. Un proveedor de recursos indica el conjunto de recursos y operaciones que están disponibles para un servicio de Azure.

Una plantilla implementa y configura los recursos. Un proveedor de recursos proporciona un tipo de recurso. Por ejemplo, el proveedor de recursos de proceso (Microsoft.Compute), proporciona varios tipos de recursos como virtualMachines y availabilitySets. Cada proveedor de recursos proporciona una API a Azure Resource Manager definida por un contrato común, lo que permite una experiencia de autoría coherente y unificada entre todos los proveedores de recursos. Pero un proveedor de recursos que esté disponible en Azure global puede no estarlo en una nube soberana o una región de Azure Stack.

![Proveedores de recursos](./media/templates-cloud-consistency/resource-providers.png)

Para comprobar los proveedores de recursos que están disponibles en una nube determinada, ejecute el script siguiente en la interfaz de la línea de comandos de Azure ([CLI](/cli/azure/install-azure-cli)):

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

También puede usar el cmdlet de PowerShell siguiente para ver los proveedores de recursos disponibles:

```azurepowershell-interactive
Get-AzureRmResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

### <a name="verify-the-version-of-all-resource-types"></a>Comprobar la versión de todos los tipos de recursos

Un conjunto de propiedades es común para todos los tipos de recursos, pero cada recurso también tiene sus propias propiedades específicas. Las características nuevas y las propiedades relacionadas se agregan a los tipos de recursos existentes a veces a través de una versión de API nueva. Un recurso en una plantilla tiene su propia propiedad de versión de API: `apiVersion`. Este control de versiones garantiza que una configuración de recurso existente en una plantilla no se vea afectada por los cambios en la plataforma.

Es posible que las versiones de API nuevas que se introducen para los tipos de recursos existentes en Azure global no estén inmediatamente disponibles en todas las regiones, las nubes soberanas o Azure Stack. Para ver una lista de los proveedores de recursos disponibles, los tipos de recursos y las versiones de API para una nube, puede usar el Explorador de recursos en Azure Portal. Busque el Explorador de recursos en el menú Todos los servicios. Expanda el nodo Proveedores en el Explorador de recursos para devolver todos los proveedores de recursos disponibles, sus tipos de recursos y las versiones de API de esa nube.

Para obtener la lista de la versión de API disponible para todos los tipos de recursos en una nube determinada en la CLI de Azure, ejecute el script siguiente:

```azurecli-interactive
az provider list --query "[].{namespace:namespace, resourceType:resourceType[]}"
```

También puede usar el cmdlet de PowerShell siguiente:

```azurepowershell-interactive
Get-AzureRmResourceProvider | select-object ProviderNamespace -ExpandProperty ResourceTypes | ft ProviderNamespace, ResourceTypeName, ApiVersions
```

### <a name="refer-to-resource-locations-with-a-parameter"></a>Hacer referencia a ubicaciones de recursos con un parámetro

Una plantilla siempre se implementa en un grupo de recursos que se encuentra en una región. Además de la propia implementación, cada recurso de una plantilla también tiene una propiedad de ubicación que se usa para especificar la región en la que se implementa. Para desarrollar la plantilla para mantener la coherencia en la nube, necesita una forma dinámica de hacer referencia a las ubicaciones de los recursos, dado que cada instancia de Azure Stack pueden contener nombres de ubicación únicos. Normalmente, los recursos se implementan en la misma región que el grupo de recursos, pero para admitir escenarios como la disponibilidad de las aplicaciones entre regiones, puede ser útil distribuir los recursos entre las regiones.

Aunque los nombres de región se podrían codificar de forma rígida al especificar las propiedades del recurso en una plantilla, este enfoque no garantiza que la plantilla se pueda implementar en otros entornos de Azure Stack, porque lo más probable es que el nombre de la región no exista allí.

Para dar cabida a otras regiones, agregue una ubicación de parámetro de entrada a la plantilla con un valor predeterminado. Si no se especifica ningún valor durante la implementación, se usará el valor predeterminado.

La función de plantilla `[resourceGroup()]` devuelve un objeto que contiene los siguientes pares clave/valor:

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

Al hacer referencia a la clave de ubicación del objeto en el valor defaultValue del parámetro de entrada, Azure Resource Manager, en tiempo de ejecución, reemplazará la función de plantilla `[resourceGroup().location]` con el nombre de la ubicación del grupo de recursos en el que se implementa la plantilla.

```json
"parameters": {
  "location": {
    "type": "string",
    "metadata": {
      "description": "Location the resources will be deployed to."
    },
    "defaultValue": "[resourceGroup().location]"
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2015-06-15",
    "name": "storageaccount1",
    "location": "[parameters('location')]",
    ...
```

Con esta función de plantilla, puede implementar la plantilla en cualquier nube sin ni siquiera conocer los nombres de región con antelación. Además, una ubicación para un recurso específico en la plantilla puede diferir de la ubicación del grupo de recursos. En este caso, se puede configurar mediante parámetros de entrada adicionales para ese recurso concreto, mientras que los otros recursos de la misma plantilla siguen usando el parámetro de entrada de ubicación inicial.

### <a name="track-versions-using-api-profiles"></a>Seguimiento de versiones mediante perfiles de API

Puede ser muy difícil realizar el seguimiento de todos los proveedores de recursos disponibles y las versiones de API relacionadas que se encuentran en Azure Stack. Por ejemplo, en el momento de redactar este texto, la versión de API más reciente para **Microsoft.Compute/availabilitySets** en Azure es `2018-04-01`, mientras que la versión de API disponible común para Azure y Azure Stack es `2016-03-30`. La versión de API común para **Microsoft.Storage/storageAccounts** que se comparte entre todas las ubicaciones de Azure y Azure Stack es `2016-01-01`, mientras que la versión de API más reciente en Azure es `2018-02-01`.

Por este motivo, Resource Manager introdujo el concepto de perfiles de API para las plantillas. Sin los perfiles de API, cada recurso de una plantilla se configura con un elemento `apiVersion` que describe la versión de API para ese recurso concreto.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "metadata": {
          "description": "Location the resources will be deployed to."
      },
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2016-01-01",
      "name": "mystorageaccount",
      "location": "[parameters('location')]",
      "properties": {
        "accountType": "Standard_LRS"
      }
    },
    {
      "type": "Microsoft.Compute/availabilitySets",
      "apiVersion": "2016-03-30",
      "name": "myavailabilityset",
      "location": "[parameters('location')]",
      "properties": {
        "platformFaultDomainCount": 2,
        "platformUpdateDomainCount": 2
      }
    }
  ],
  "outputs": {}
}
```

Una versión de perfil de API actúa como un alias para una única versión de API por tipo de recurso común para Azure y Azure Stack. En lugar de especificar una versión de API para cada recurso de una plantilla, especifique solo la versión del perfil de API en un elemento raíz nuevo denominado `apiProfile` y omita el elemento `apiVersion` para los recursos individuales.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "mystorageaccount",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "type": "Microsoft.Compute/availabilitySets",
            "name": "myavailabilityset",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

El perfil de API garantiza que las versiones de API están disponibles en todas las ubicaciones, por lo que no tiene que comprobar de forma manual las versiones de API que están disponibles en una ubicación específica. Para asegurarse de que las versiones de API a las que hace referencia el perfil de API están presentes en un entorno de Azure Stack, los operadores de Azure Stack deben mantener la solución actualizada según la directiva de soporte técnico. Si un sistema lleva más de seis meses desactualizado, se considera fuera de cumplimiento, y el entorno se debe actualizar.

El perfil de API no es un elemento necesario en una plantilla. Incluso si agrega el elemento, solo se usará para los recursos para los que no se especifica `apiVersion`. Este elemento permite realizar cambios graduales, pero no requiere ningún cambio en las plantillas existentes.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2016-01-01",
            "name": "mystorageaccount",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "type": "Microsoft.Compute/availabilitySets",
            "name": "myavailabilityset",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

## <a name="check-endpoint-references"></a>Comprobar las referencias de punto de conexión

Los recursos pueden tener referencias a otros servicios en la plataforma. Por ejemplo, una dirección IP pública puede tener un nombre DNS público asignado. La nube pública, las nubes soberanas y las soluciones de Azure Stack tienen sus propios espacios de nombres de punto de conexión diferentes. En la mayoría de los casos, un recurso solo requiere un prefijo como entrada en la plantilla. En tiempo de ejecución, Azure Resource Manager le anexa el valor de punto de conexión. Algunos valores de punto de conexión se deben especificar de forma explícita en la plantilla.

> [!NOTE]
> Para desarrollar plantillas para mantener la coherencia en la nube, no codifique de forma rígida los espacios de nombres de punto de conexión.

Los dos ejemplos siguientes son espacios de nombres de punto de conexión comunes que se deben especificar de forma explícita al crear un recurso:

* Cuentas de almacenamiento (blob, cola, tabla y archivo)
* Cadenas de conexión para las bases de datos y Azure Cache for Redis

Los espacios de nombres de punto de conexión también se pueden usar en la salida de una plantilla como información para el usuario cuando se complete la implementación. A continuación se muestran ejemplos comunes:

* Cuentas de almacenamiento (blob, cola, tabla y archivo)
* Cadenas de conexión (MySql, SQLServer, SQLAzure, Custom, NotificationHub, ServiceBus, EventHub, ApiHub, DocDb, RedisCache, PostgreSQL)
* Traffic Manager
* etiquetaDeNombreDeDominio de una dirección IP pública
* Servicios en la nube

En general, evite los puntos de conexión codificados de forma rígida en una plantilla. El procedimiento recomendado consiste en usar la función de plantilla de referencia para recuperar los puntos de conexión de forma dinámica. Por ejemplo, el punto de conexión que se codifica de forma rígida con más frecuencia es el espacio de nombres de punto de conexión para las cuentas de almacenamiento. Cada cuenta de almacenamiento tiene un FQDN único que se crea concatenando el nombre de la cuenta de almacenamiento con el espacio de nombres de punto de conexión. Una cuenta de Blob Storage con el nombre mystorageaccount1 da como resultado otro FQDN en función de la nube:

* **mystorageaccount1.blob.core.windows.net** cuando se crea en la nube de Azure global.
* **mystorageaccount1.blob.core.chinacloudapi.cn** cuando se crea en la nube de Azure China 21Vianet.

La función de plantilla de referencia siguiente recupera el espacio de nombres de punto de conexión del proveedor de recursos de almacenamiento:

```json
"diskUri":"[concat(reference(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))).primaryEndpoints.blob, 'container/myosdisk.vhd')]"
```

Si se reemplaza el valor codificado de forma rígida del punto de conexión de cuenta de almacenamiento con la función de plantilla `reference`, se puede usar la misma plantilla para implementar correctamente en otros entornos sin realizar ningún cambio en la referencia de punto de conexión.

### <a name="refer-to-existing-resources-by-unique-id"></a>Hacer referencia a recursos existentes con el identificador único

También puede hacer referencia a un recurso existente del mismo grupo de recursos o de otro, y dentro de la misma suscripción o en otra, en el mismo inquilino en la misma nube. Para recuperar las propiedades del recurso, debe usar el identificador único para el recurso propiamente dicho. La función de plantilla `resourceId` recupera el identificador único de un recurso, como SQL Server como se muestra en el código siguiente:

```json
"outputs": {
  "resourceId":{
    "type": "string",
    "value": "[resourceId('otherResourceGroup', 'Microsoft.Sql/servers', parameters('serverName'))]"
  }
}
```

Después, puede usar la función `resourceId` dentro de la función de plantilla `reference` para recuperar las propiedades de una base de datos. El objeto devuelto contiene la propiedad `fullyQualifiedDomainName` que almacena el valor de punto de conexión completo. Este valor se recupera en tiempo de ejecución y proporciona el espacio de nombres de punto de conexión específico del entorno de nube. Para definir la cadena de conexión sin codificar de forma rígida el espacio de nombres de punto de conexión, se puede hacer referencia a la propiedad del objeto devuelto directamente en la cadena de conexión como se muestra a continuación:

```json
"[concat('Server=tcp:', reference(resourceId('sql', 'Microsoft.Sql/servers', parameters('test')), '2015-05-01-preview').fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('database'),';User ID=', parameters('username'), ';Password=', parameters('pass'), ';Encrypt=True;')]"
```

## <a name="consider-resource-properties"></a>Tener en cuenta las propiedades del recurso

Los recursos específicos dentro de entornos de Azure Stack tienen propiedades únicas que se deben tener en cuenta en la plantilla.

### <a name="ensure-vm-images-are-available"></a>Asegurarse de que las imágenes de máquina virtual están disponibles

Azure proporciona una amplia selección de imágenes de máquina virtual. Microsoft y asociados crean y preparan estas imágenes para la implementación. Las imágenes forman la base de las máquinas virtuales en la plataforma. Pero una plantilla coherente para la nube solo debe hacer referencia a los parámetros disponibles, en particular, el editor, la oferta y SKU de las imágenes de máquina virtual disponibles para Azure global, las nubes soberanas de Azure o una solución de Azure Stack.

Para recuperar una lista de las imágenes de máquina virtual disponibles en una ubicación, ejecute el comando de la CLI de Azure siguiente:

```azurecli-interactive
az vm image list -all
```

Puede recuperar la misma lista con el cmdlet de PowerShell de Azure [Get-AzureRmVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) y especificar la ubicación que quiera con el parámetro `-Location`. Por ejemplo:

```azurepowershell-interactive
Get-AzureRmVMImagePublisher -Location "West Europe" | Get-AzureRmVMImageOffer | Get-AzureRmVMImageSku | Get-AzureRmVMImage
```

Este comando tarda unos minutos en devolver todas las imágenes disponibles en la región de Oeste de Europa de la nube de Azure global.

Si puso estas imágenes de máquina virtual a disposición de Azure Stack, se podría consumir todo el almacenamiento disponible. Para dar cabida incluso a la unidad de escala más pequeña, Azure Stack permite seleccionar las imágenes que quiere agregar a un entorno.

En el ejemplo de código siguiente se muestra un enfoque coherente para hacer referencia a los parámetros de editor, oferta y SKU en las plantillas de ARM:

```json
"storageProfile": {
    "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "2016-Datacenter",
    "version": "latest"
    }
}
```

### <a name="check-local-vm-sizes"></a>Comprobar los tamaños de las máquinas virtuales locales

Para desarrollar la plantilla para mantener la coherencia en la nube, debe asegurarse de que el tamaño de máquina virtual que quiere está disponible en todos los entornos de destino. Los tamaños de máquina virtual son una agrupación de funcionalidades y características de rendimiento. Algunos tamaños de máquina virtual dependen del hardware en el que se ejecuta la máquina virtual. Por ejemplo, si quiere implementar una máquina virtual optimizada para GPU, el hardware que ejecuta el hipervisor debe tener GPU de hardware.

Cuando Microsoft presenta un tamaño de máquina virtual nuevo que tiene determinadas dependencias de hardware, por lo general primero se pone a disposición de un pequeño subconjunto de regiones en la nube de Azure. Más adelante, se pone a disposición de otras regiones y nubes. Para asegurarse de que el tamaño de máquina virtual existe en todas las nubes en las que se implementa, puede recuperar los tamaños disponibles con el comando siguiente de la CLI de Azure:

```azurecli-interactive
az vm list-sizes --location "West Europe"
```

Para Azure PowerShell, use:

```azurepowershell-interactive
Get-AzureRmVMSize -Location "West Europe"
```

Para obtener una lista completa de los servicios disponibles, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?cdn=disable).

### <a name="check-use-of-azure-managed-disks-in-azure-stack"></a>Comprobar el uso de Azure Managed Disks en Azure Stack

Los discos administrados controlan el almacenamiento para un inquilino de Azure. En lugar de crear de forma explícita una cuenta de almacenamiento y especificar el URI para un disco duro virtual (VHD), puede usar discos administrados para realizar estas acciones de forma implícita al implementar una máquina virtual. Los discos administrados mejoran la disponibilidad mediante la colocación de todos los discos de las máquinas virtuales del mismo conjunto de disponibilidad en otras unidades de almacenamiento. Además, los discos duros virtuales existentes se pueden convertir de almacenamiento Estándar a Premium con mucho menos tiempo de inactividad.

Aunque los discos administrados se encuentran en la hoja de ruta de Azure Stack, en la actualidad no se admiten. Hasta que se admitan, puede desarrollar plantillas coherentes con la nube para Azure Stack si especifica de forma explícita los discos duros virtuales mediante el elemento `vhd` de la plantilla para el recurso de máquina virtual tal como se muestra:

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "name": "osdisk",
    "vhd": {
      "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
    },
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

En cambio, para especificar una configuración de disco administrado en una plantilla, quite el elemento `vhd` de la configuración del disco.

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

Los mismos cambios también se aplican a los [discos de datos](../../virtual-machines/using-managed-disks-template-deployments.md).

### <a name="verify-that-vm-extensions-are-available-in-azure-stack"></a>Comprobar que las extensiones de máquina virtual están disponibles en Azure Stack

Otra consideración para mantener la coherencia en la nube es el uso de [extensiones de máquina virtual](../../virtual-machines/extensions/features-windows.md) para configurar los recursos dentro de una máquina virtual. No todas las extensiones de máquina virtual están disponibles en Azure Stack. Una plantilla puede especificar los recursos dedicados a la extensión de máquina virtual, y crear dependencias y condiciones dentro de la plantilla.

Por ejemplo, si quiere configurar una máquina virtual que ejecute Microsoft SQL Server, la extensión de máquina virtual puede configurar SQL Server como parte de la implementación de plantilla. Tenga en cuenta qué sucede si la plantilla de implementación también contiene un servidor de aplicaciones configurado para crear una base de datos en la máquina virtual que ejecuta SQL Server. Además de usar una extensión de máquina virtual para los servidores de aplicaciones, puede configurar la dependencia del servidor de aplicaciones tras la devolución correcta del recurso de extensión de máquina virtual de SQL Server. Este enfoque garantiza que la máquina virtual que ejecuta SQL Server está configurada y disponible cuando se indique al servidor de aplicaciones que cree la base de datos.

El enfoque declarativo de la plantilla permite definir el estado final de los recursos y sus interdependencias, mientras que la plataforma se encarga de la lógica necesaria para las dependencias.

#### <a name="check-that-vm-extensions-are-available"></a>Comprobar que las extensiones de máquina virtual están disponibles

Existen muchos tipos de extensiones de máquina virtual. Al desarrollar plantillas para mantener la coherencia en la nube, asegúrese de usar solo las extensiones que están disponibles en todas las regiones de destino de la plantilla.

Para recuperar una lista de las extensiones de máquina virtual que están disponibles para una región específica (en este ejemplo, `myLocation`), ejecute el comando siguiente de la CLI de Azure:

```azurecli-interactive
az vm extension image list --location myLocation
```

También puede ejecutar el cmdlet de Azure PowerShell [Get-AzureRmVmImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) y usar `-Location` para especificar la ubicación de la imagen de máquina virtual. Por ejemplo:

```azurepowershell-interactive
Get-AzureRmVmImagePublisher -Location myLocation | Get-AzureRmVMExtensionImageType | Get-AzureRmVMExtensionImage | Select Type, Version
```

#### <a name="ensure-that-versions-are-available"></a>Asegurarse de que hay versiones disponibles

Como las extensiones de máquina virtual son recursos propios de Resource Manager, tienen sus propias versiones de API. Como se muestra en el código siguiente, el tipo de extensión de máquina virtual es un recurso anidado en el proveedor de recursos Microsoft.Compute.

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "apiVersion": "2015-06-15",
    "name": "myExtension",
    "location": "[parameters('location')]",
    ...
```

La versión de API del recurso de extensión de máquina virtual debe estar presente en todas las ubicaciones de destino de la plantilla. La dependencia de ubicación funciona como la disponibilidad del proveedor de recursos de versión de API mencionado anteriormente en la sección "Comprobar la versión de todos los tipos de recursos".

Para recuperar una lista de las versiones de API disponibles para el recurso de extensión de máquina virtual, use el cmdlet [Get-AzureRmResourceProvider](/powershell/module/az.resources/get-azresourceprovider) con el proveedor de recursos **Microsoft.Compute**, como se muestra:

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachines/extensions"}
```

También puede usar las extensiones de máquina virtual en conjuntos de escalado de máquinas virtuales. Se aplican las mismas condiciones de ubicación. Para desarrollar la plantilla para mantener la coherencia en la nube, asegúrese de que las versiones de API están disponibles en todas las ubicaciones en las que planea implementar. Para recuperar las versiones de API del recurso de extensión de máquina virtual para conjuntos de escalado, use el mismo cmdlet que antes, pero especifique el tipo de recurso de conjuntos de escalado de máquinas virtuales tal como se muestra:

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachineScaleSets/extensions"}
```

Cada extensión específica también tiene una versión. Esta versión se muestra en la propiedad `typeHandlerVersion` de la extensión de máquina virtual. Asegúrese de que la versión especificada en el elemento `typeHandlerVersion` de las extensiones de máquina virtual de la plantilla está disponible en las ubicaciones donde planea implementar la plantilla. Por ejemplo, en el código siguiente se especifica la versión 1.7:

```json
{
    "type": "extensions",
    "apiVersion": "2016-03-30",
    "name": "MyCustomScriptExtension",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
    ],
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        ...
```

Para recuperar una lista de las versiones disponibles para una extensión de máquina virtual específica, use el cmdlet [Get-AzureRmVMExtensionImage](/powershell/module/az.compute/get-azvmextensionimage). En el ejemplo siguiente se recuperan las versiones disponibles para la extensión PowerShell DSC (Desired State Configuration) desde **myLocation**:

```azurepowershell-interactive
Get-AzureRmVMExtensionImage -Location myLocation -PublisherName Microsoft.PowerShell -Type DSC | FT
```

Para obtener una lista de los editores, use el comando [Get-AzureRmVmImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher). Para el tipo de solicitud, use el comando [Get-AzureRmVMExtensionImageType](/powershell/module/az.compute/get-azvmextensionimagetype).

## <a name="tips-for-testing-and-automation"></a>Sugerencias para las pruebas y la automatización

Es un desafío realizar el seguimiento de todas las configuraciones relacionadas, funcionalidades y limitaciones durante la creación de una plantilla. El enfoque común consiste en desarrollar y probar las plantillas en una sola nube antes de seleccionar otras ubicaciones como destino. Pero cuanto antes se realicen las pruebas en el proceso de creación, menos solución de problemas y reescritura de código tendrá que realizar el equipo de desarrollo. Los errores de implementación debidos a las dependencias de ubicación pueden llevar mucho tiempo en solucionarse. Por este motivo se recomienda realizar las pruebas automatizadas tan pronto como sea posible en el ciclo de creación. En última instancia, necesitará menos tiempo de desarrollo y menos recursos, y los artefactos coherentes para la nube serán incluso más valiosos.

En la imagen siguiente se muestra un ejemplo típico de un proceso de desarrollo de un equipo en el que se usa un entorno de desarrollo integrado (IDE). Los distintos tipos de pruebas se ejecutan en diferentes etapas de la escala de tiempo. En este caso, dos desarrolladores trabajan en la misma solución, pero este escenario se aplica igualmente a un único desarrollador o un equipo grande. Normalmente, cada desarrollador crea una copia local de un repositorio central, lo que permite a cada uno de ellos trabajar en la copia local sin afectar a otros que pueden estar trabajando en los mismos archivos.

![Flujo de trabajo](./media/templates-cloud-consistency/workflow.png)

Tenga en cuenta las sugerencias siguientes para las pruebas y la automatización:

* Asegúrese de usar de herramientas de prueba. Por ejemplo, en Visual Studio Code y Visual Studio se incluye IntelliSense y otras características que pueden ayudarle a validar las plantillas.
* Para mejorar la calidad del código durante el desarrollo en el IDE local, realice el análisis de código estático con pruebas unitarias y de integración.
* Para obtener una experiencia incluso mejor durante el desarrollo inicial, las pruebas unitarias y las de integración solo deben advertir cuando se encuentre un problema y continuar con las pruebas. De este modo, puede identificar los problemas que se deben solucionar y priorizar el orden de los cambios, lo que también se denomina desarrollo controlado por pruebas (TDD).
* Tenga en cuenta que algunas pruebas se pueden realizar sin estar conectado a Azure Resource Manager. Otras, como probar la implementación de plantilla, requieren Resource Manager para llevar a cabo determinadas acciones que no se pueden realizar sin conexión.
* Probar una plantilla de implementación sobre la API de validación no es lo mismo que una implementación real. Además, incluso si una plantilla se implementa desde un archivo local, todas las referencias a las plantillas anidadas en la plantilla se recuperan de forma directa mediante Resource Manager, y los artefactos a los que hacen referencia las extensiones de máquina virtual se recuperan mediante el agente de máquina virtual que se ejecuta dentro de la máquina virtual implementada.

## <a name="next-steps"></a>Pasos siguientes

* [Consideraciones de la plantilla de Azure Resource Manager](/azure-stack/user/azure-stack-develop-templates)
* [Procedimientos recomendados para las plantillas de ARM](template-syntax.md)
