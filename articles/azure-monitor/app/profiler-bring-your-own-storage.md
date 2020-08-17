---
title: Configuración de BYOS (Traiga su propio almacenamiento) para Profiler y Snapshot Debugger
description: Configuración de BYOS (Traiga su propio almacenamiento) para Profiler y Snapshot Debugger
ms.topic: conceptual
author: renatosalas
ms.author: regutier
ms.date: 04/14/2020
ms.reviewer: mbullwin
ms.openlocfilehash: 719f0cfa0a1f80568acf3231ce3ffab441e5f6b7
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87117392"
---
# <a name="configure-bring-your-own-storage-byos-for-application-insights-profiler-and-snapshot-debugger"></a>Configuración de Traiga su propio almacenamiento (BYOS) para Application Insights Profiler y Snapshot Debugger

## <a name="what-is-bring-your-own-storage-byos-and-why-might-i-need-it"></a>¿Qué es Traiga su propio almacenamiento (BYOS) y por qué podría resultarme últil? 
Cuando se usa Application Insights Profiler o Snapshot Debugger, los artefactos generados por la aplicación se cargan en cuentas de Azure Storage a través de la red pública de Internet. Microsoft paga y controla esas cuentas para su procesamiento y análisis. Microsoft controla las directivas de administración de la vigencia y el cifrado en reposo para esos artefactos.

Con Traiga su propio almacenamiento, estos artefactos se cargan en una cuenta de almacenamiento que usted controla. Esto significa que controla la directiva de cifrado en reposo, la directiva de administración de la vigencia y el acceso a la red. Sin embargo, será responsable de los costos asociados a esa cuenta de almacenamiento.

> [!NOTE]
> Si está habilitando Private Link, Traiga su propio almacenamiento es obligatorio. Para obtener más información acerca de Private Link para Application Insights, [consulte la documentación](../platform/private-link-security.md).
>
> Si está habilitando las claves administradas de cliente, Traiga su propio almacenamiento es obligatorio. Para obtener más información acerca de las claves administradas de cliente, [consulte la documentación](../platform/customer-managed-keys.md).

## <a name="how-will-my-storage-account-be-accessed"></a>¿Cómo se tiene acceso a la cuenta de almacenamiento?
1. Los agentes que se ejecutan en Virtual Machines o App Service cargarán artefactos (perfiles, instantáneas y símbolos) en los contenedores de blobs de su cuenta. Este proceso implica ponerse en contacto con el servicio Application Insights Profiler o Snapshot Debugger para obtener un token de SAS (firma de acceso compartido) para un nuevo blob en la cuenta de almacenamiento.
1. El servicio Application Insights Profiler o Snapshot Debugger analizará el blob entrante y volverá a escribir los resultados del análisis y los archivos de registro en el almacenamiento de blobs. Dependiendo de la capacidad de proceso disponible, este proceso puede producirse en cualquier momento después de la carga.
1. Al ver los seguimientos de Profiler o el análisis de Snapshot Debugger, el servicio capturará los resultados del análisis del almacenamiento de blobs.

## <a name="prerequisites"></a>Requisitos previos
* Asegúrese de crear la cuenta de almacenamiento en la misma ubicación que el recurso de Application Insights. Por ejemplo, Si el recurso de Application Insights está en Oeste de EE. UU. 2, la cuenta de almacenamiento debe estar también en Oeste de EE. UU. 2. 
* Conceda el rol "Colaborador de datos de blobs de almacenamiento" al "Acceso a almacenamiento de confianza de servicios de diagnóstico" de la aplicación de AAD en la cuenta de almacenamiento a través de la interfaz de usuario de Access Control (IAM).
* Si está habilitado Private Link, configure la opción adicional para permitir la conexión a nuestro servicio de confianza de Microsoft desde Virtual Network. 

## <a name="how-to-enable-byos"></a>Habilitación de BYOS

### <a name="create-storage-account"></a>Crear cuenta de almacenamiento
Cree una nueva cuenta de almacenamiento (si no la tiene) en la misma ubicación que el recurso de Application Insights.
Si el recurso de Application Insights está en `West US 2`, la cuenta de almacenamiento debe estar en `West US 2`.

### <a name="grant-access-to-diagnostic-services-to-your-storage-account"></a>Concesión de acceso a los servicios de diagnóstico a su cuenta de almacenamiento
Una cuenta de almacenamiento de BYOS se vinculará a un recurso de Application Insights. Puede haber solo una cuenta de almacenamiento por recurso de Application Insights y ambos deben estar en la misma ubicación. Puede usar la misma cuenta de almacenamiento con más de un recurso de Application Insights.

En primer lugar, se debe conceder acceso a la cuenta de almacenamiento al servicio Application Insights Profiler y Snapshot Debugger. Para conceder acceso, agregue el rol `Storage Blob Data Contributor` a la aplicación de AAD llamada `Diagnostic Services Trusted Storage Access` a través de la página de Access Control (IAM) de la cuenta de almacenamiento, tal como se muestra en la figura 1.0. 

Pasos: 
1. Haga clic en el botón "Agregar" de la sección "Agregar una asignación de roles". 
1. Seleccione rol "Colaborador de datos de blobs de almacenamiento" 
1. En la opción "Asignar acceso a", seleccione "Usuario, grupo o entidad de servicio de Azure AD". 
1. Búsqueda y selección de la aplicación "Acceso a almacenamiento de confianza de servicios de diagnóstico" 
1. Guardar cambios

_![Figura 1.0](media/profiler-bring-your-own-storage/figure-10.png)_
_Figura 1.0_ 

Después de agregar el rol, aparecerá en la sección "Asignaciones de roles", como la figura 1.1. 
_![Figura 1.1](media/profiler-bring-your-own-storage/figure-11.png)_
_Figura 1.1_ 

Si está usando también Private Link, se necesita una configuración adicional para permitir la conexión a nuestro servicio de confianza de Microsoft desde Virtual Network. Consulte la [documentación de Seguridad de red de almacenamiento](../../storage/common/storage-network-security.md#trusted-microsoft-services).

### <a name="link-your-storage-account-with-your-application-insights-resource"></a>Vinculación de la cuenta de almacenamiento con el recurso de Application Insights
Para configurar BYOS para el diagnóstico en el nivel de código (Profiler/Debugger), hay tres opciones:

* Uso de cmdlets de Azure PowerShell
* Uso de la interfaz de la línea de comandos de Azure (CLI)
* Uso de la plantilla de Azure Resource Manager

#### <a name="configure-using-azure-powershell-cmdlets"></a>Configuración mediante cmdlets de Azure PowerShell

1. Asegúrese de que ha instalado Az PowerShell 4.2.0 o posterior.

    Para instalar Azure PowerShell, consulte la [documentación oficial de Azure PowerShell](/powershell/azure/install-az-ps).

1. Instale la extensión de PowerShell para Application Insights.
    ```powershell
    Install-Module -Name Az.ApplicationInsights -Force
    ```

1. Inicie sesión con su cuenta de Azure.
    ```powershell
    Connect-AzAccount -Subscription "{subscription_id}"
    ```

    Para obtener más información sobre cómo iniciar sesión, consulte la [documentación de Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

1. Quite la cuenta de almacenamiento anterior vinculada al recurso de Application Insights.

    Patrón:
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{storage_account_name}"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

    Ejemplo:
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

1. Conecte la cuenta de almacenamiento con el recurso de Application Insights.
    
    Patrón:
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "{resource_group_name}" -Name "{storage_account_name}"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{application_insights_name}"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

    Ejemplo:
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "byos-test" -Name "byosteststoragewestus2"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

#### <a name="configure-using-azure-cli"></a>Configuración mediante la CLI de Azure

1. Asegúrese de que ha instalado la CLI de Azure.

    Para instalar la CLI de Azure, consulte la [documentación oficial de la CLI de Azure](/cli/azure/install-azure-cli).

1. Instale la extensión de la CLI de Application Insights.
    ```powershell
    az extension add -n application-insights
    ```

1. Conecte la cuenta de almacenamiento con el recurso de Application Insights.

    Patrón:
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "{resource_group_name}" --app "{application_insights_name}" --storage-account "{storage_account_name}"
    ```
    
    Ejemplo:
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "byos-test" --app "byos-test-westus2-ai" --storage-account "byosteststoragewestus2"
    ```
    
    Resultado esperado:
    ```powershell
    {
      "id": "/subscriptions/{subscription}/resourcegroups/byos-test/providers/microsoft.insights/components/byos-test-westus2-ai/linkedstorageaccounts/serviceprofiler",
      "linkedStorageAccount": "/subscriptions/{subscription}/resourceGroups/byos-test/providers/Microsoft.Storage/storageAccounts/byosteststoragewestus2",
      "name": "serviceprofiler",
      "resourceGroup": "byos-test",
      "type": "microsoft.insights/components/linkedstorageaccounts"
    }
    ```

    > [!NOTE]
    > Para realizar actualizaciones en las cuentas de almacenamiento vinculadas al recurso de Application Insights, consulte la [documentación de la CLI de Application Insights](/cli/azure/ext/application-insights/monitor/app-insights/component/linked-storage).

#### <a name="configure-using-azure-resource-manager-template"></a>Configuración de la plantilla de Azure Resource Manager

1. Cree un archivo de plantilla de Azure Resource Manager con el siguiente contenido (byos.template.json).
    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "applicationinsights_name": {
          "type": "String"
        },
        "storageaccount_name": {
          "type": "String"
        }
      },
      "variables": {},
      "resources": [
        {
          "name": "[concat(parameters('applicationinsights_name'), '/serviceprofiler')]",
          "type": "Microsoft.Insights/components/linkedStorageAccounts",
          "apiVersion": "2020-03-01-preview",
          "properties": {
            "linkedStorageAccount": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageaccount_name'))]"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Ejecute el siguiente comando de PowerShell para implementar la plantilla anterior (cree una cuenta de almacenamiento vinculada).

    Patrón:
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "{your_resource_name}" -TemplateFile "{local_path_to_arm_template}"
    ```

    Ejemplo:
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "byos-test" -TemplateFile "D:\Docs\byos.template.json"
    ```

1. Proporcione los siguientes parámetros cuando se le solicite en la consola de PowerShell:
    
    |           Parámetro           |                                Descripción                               |
    |-------------------------------|--------------------------------------------------------------------------|
    | application_insights_name     | Nombre del recurso de Application Insights para habilitar BYOS.            |
    | storage_account_name          | Nombre del recurso de la cuenta de almacenamiento que usará como BYOS. |
    
    Resultado esperado:
    ```powershell
    Supply values for the following parameters:
    (Type !? for Help.)
    application_insights_name: byos-test-westus2-ai
    storage_account_name: byosteststoragewestus2
    
    DeploymentName          : byos.template
    ResourceGroupName       : byos-test
    ProvisioningState       : Succeeded
    Timestamp               : 4/16/2020 1:24:57 AM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name                            Type                       Value
                              ==============================  =========================  ==========
                              application_insights_name        String                     byos-test-westus2-ai
                              storage_account_name             String                     byosteststoragewestus2
                              
    Outputs                 :
    DeploymentDebugLogLevel :
    ```

1. Habilite el diagnóstico en el nivel de código (Profiler/Debugger) en la carga de trabajo correspondiente a través de Azure Portal. (App Service > Application Insights) _![Figura 2.0](media/profiler-bring-your-own-storage/figure-20.png)_
_Figura 2.0_

## <a name="troubleshooting"></a>Solución de problemas
### <a name="template-schema-schema_uri-isnt-supported"></a>No se admite el esquema de plantilla "{schema_uri}".
* Asegúrese de que la propiedad `$schema` de la plantilla es válida. Debe seguir el patrón siguiente: `https://schema.management.azure.com/schemas/{schema_version}/deploymentTemplate.json#`
* Asegúrese de que `schema_version` en la plantilla esté dentro de los valores válidos: `2014-04-01-preview, 2015-01-01, 2018-05-01, 2019-04-01, 2019-08-01`.
    Mensaje de error:
    ```powershell
    New-AzResourceGroupDeployment : 11:53:49 AM - Error: Code=InvalidTemplate; Message=Deployment template validation failed: 'Template schema
    'https://schema.management.azure.com/schemas/2020-01-01/deploymentTemplate.json#' is not supported. Supported versions are
    '2014-04-01-preview,2015-01-01,2018-05-01,2019-04-01,2019-08-01'. Please see https://aka.ms/arm-template for usage details.'.
    ```

### <a name="no-registered-resource-provider-found-for-location-location"></a>No se encontró ningún proveedor de recursos registrado para la ubicación "{location}".
* Asegúrese de que `apiVersion` del recurso `microsoft.insights/components` es `2015-05-01`.
* Asegúrese de que `apiVersion` del recurso `linkedStorageAccount` es `2020-03-01-preview`.
    Mensaje de error:
    ```powershell
    New-AzResourceGroupDeployment : 6:18:03 PM - Resource microsoft.insights/components 'byos-test-westus2-ai' failed with message '{
      "error": {
        "code": "NoRegisteredProviderFound",
        "message": "No registered resource provider found for location 'westus2' and API version '2020-03-01-preview' for type 'components'. The supported api-versions are '2014-04-01,
    2014-08-01, 2014-12-01-preview, 2015-05-01, 2018-05-01-preview'. The supported locations are ', eastus, southcentralus, northeurope, westeurope, southeastasia, westus2, uksouth,
    canadacentral, centralindia, japaneast, australiaeast, koreacentral, francecentral, centralus, eastus2, eastasia, westus, southafricanorth, northcentralus, brazilsouth, switzerlandnorth,
    australiasoutheast'."
      }
    }'
    ```
### <a name="storage-account-location-should-match-ai-component-location"></a>La ubicación de la cuenta de almacenamiento debe coincidir con la ubicación del componente AI.
* Asegúrese de que la ubicación del recurso de Application Insights sea la misma que la de la cuenta de almacenamiento.
    Mensaje de error:
    ```powershell
    New-AzResourceGroupDeployment : 1:01:12 PM - Resource microsoft.insights/components/linkedStorageAccounts 'byos-test-centralus-ai/serviceprofiler' failed with message '{
      "error": {
        "code": "BadRequest",
        "message": "Storage account location should match AI component location",
        "innererror": {
          "trace": [
            "System.ArgumentException"
          ]
        }
      }
    }'
    ```

Para solucionar problemas generales de Profiler, consulte la [documentación de solución de problemas de Profiler](profiler-troubleshooting.md).

Para la solución de problemas generales de Snapshot Debugger, consulte la [documentación de solución de problemas de Snapshot Debugger](snapshot-debugger-troubleshoot.md). 

## <a name="faqs"></a>Preguntas más frecuentes
* Si tengo habilitado Profiler o Snapshot y he habilitado BYOS, ¿se migrarán mis datos a mi cuenta de almacenamiento?
    _No, no lo harán._

* ¿Funcionará BYOS con el cifrado en reposo y la clave administrada de cliente?
    _Sí, para ser precisos, BYOS es un requisito para tener Profiler/Debugger habilitado con las claves administradas por el cliente._

* ¿Funcionará BYOS en un entorno aislado de Internet?
    _Sí. De hecho, BYOS es un requisito para escenarios de redes aisladas._

* ¿Funcionará BYOS cuando se hayan habilitado tanto las claves administradas de cliente como Private Link? 
    _Sí, esa opción existe._

* Si he habilitado BYOS, ¿puedo volver a usar las cuentas de almacenamiento de servicios de diagnóstico para almacenar los datos recopilados? 
    _Sí, pero ya no se admite la migración de datos desde BYOS._

* Después de habilitar BYOS, ¿me haré cargo de todos los costos relacionados, que son los derivados del almacenamiento y las redes? 
    _Sí_
