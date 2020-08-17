---
title: Aplicación de una versión mínima necesaria de Seguridad de la capa de transporte (TLS) para las solicitudes entrantes
titleSuffix: Azure Storage
description: Configure una cuenta de almacenamiento para que exija una versión mínima de Seguridad de la capa de transporte (TLS) para los clientes que realizan solicitudes en Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/29/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: e7bb996b3d42e2db2b4fa65d050ec1cb6a935bc6
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533383"
---
# <a name="enforce-a-minimum-required-version-of-transport-layer-security-tls-for-requests-to-a-storage-account"></a>Aplicación de una versión mínima necesaria de Seguridad de la capa de transporte (TLS) para las solicitudes a una cuenta de almacenamiento

La comunicación entre una aplicación cliente y una cuenta de Azure Storage se cifra mediante la Seguridad de la capa de transporte (TLS). TLS es un protocolo criptográfico estándar que garantiza la privacidad y la integridad de los datos entre los clientes y los servicios a través de Internet. Para más información acerca de TLS, consulte [Seguridad de la capa de transporte](https://en.wikipedia.org/wiki/Transport_Layer_Security).

Actualmente, Azure Storage admite tres versiones del protocolo TLS: 1.0, 1.1 y 1.2. Azure Storage usa TLS 1.2 en puntos de conexión HTTP públicos. Sin embargo, TLS 1.0 y TLS 1.1 se siguen admitiendo gracias a la compatibilidad con versiones anteriores.

De forma predeterminada, las cuentas de Azure Storage permiten a los clientes enviar y recibir datos con la versión más antigua de TLS (TLS 1.0) y con las versiones posteriores. Para aplicar medidas de seguridad más estrictas, puede configurar la cuenta de almacenamiento para que los clientes deban enviar y recibir datos con una versión más reciente de TLS. Si una cuenta de almacenamiento requiere una versión mínima de TLS, se producirá un error en todas las solicitudes realizadas con una versión anterior.

En este artículo se describe cómo usar un marco de tipo DRAG (detección-corrección-auditoría-gobernanza) para administrar continuamente una Seguridad de la capa de transporte segura para las cuentas de almacenamiento.

Para obtener información acerca de cómo especificar una versión concreta de TLS al enviar una solicitud desde una aplicación cliente, consulte [Configuración de la Seguridad de la capa de transporte (TLS) para una aplicación cliente](transport-layer-security-configure-client-version.md).

## <a name="detect-the-tls-version-used-by-client-applications"></a>Detección de la versión de TLS que usan las aplicaciones cliente

Cuando se aplica una versión mínima de TLS para la cuenta de almacenamiento, se corre el riesgo de rechazar las solicitudes de los clientes que envían datos con una versión anterior de TLS. Para comprender cómo la configuración de la versión mínima de TLS puede afectar a las aplicaciones cliente, Microsoft recomienda que habilite el registro para su cuenta de Azure Storage y analice los registros después de un intervalo de tiempo para detectar qué versiones de TLS están usando las aplicaciones cliente.

Para registrar las solicitudes a su cuenta de Azure Storage y determinar qué versión de TLS usa el cliente, puede usar el registro de Azure Storage en Azure Monitor (versión preliminar). Para más información, consulte [Supervisión de Azure Storage](monitor-storage.md).

El registro de Azure Storage en Azure Monitor admite el uso de consultas de registro para analizar los datos de registro. Para consultar los registros, puede usar un área de trabajo de Azure Log Analytics. Para más información sobre las consultas de registro, consulte el [Tutorial: Introducción a las consultas de Log Analytics](../../azure-monitor/log-query/get-started-portal.md).

Para registrar datos de Azure Storage con Azure Monitor y analizarlos con Azure Log Analytics, primero debe crear una configuración de diagnóstico que indique qué tipos de solicitudes y para qué servicios de almacenamiento quiere registrar los datos. Para crear una configuración de diagnóstico en Azure Portal, siga estos pasos:

1. Inscríbase en la [versión preliminar de registro de Azure Storage en Azure Monitor](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUM0E0MFdPRFpOVTRYVklDSE1WUTcyTVAwOC4u).
1. Cree un área de trabajo de Log Analytics en la suscripción que contenga la cuenta de Azure Storage. Después de configurar el registro de la cuenta de almacenamiento, los registros estarán disponibles en el área de trabajo de Log Analytics. Para obtener más información, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../../azure-monitor/learn/quick-create-workspace.md).
1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En la sección Supervisión, seleccione **Configuración de diagnóstico (versión preliminar)** .
1. Seleccione el servicio de Azure Storage cuyas solicitudes quiere registrar. Por ejemplo, elija **Blob** para registrar las solicitudes a Blob Storage.
1. Seleccione **Agregar configuración de diagnóstico**.
1. Proporcione un nombre para la configuración de diagnóstico.
1. En **Detalles de la categoría**, en la sección **registro**, elija qué tipos de solicitudes registrar. Puede registrar las solicitudes de lectura, escritura y eliminación. Por ejemplo, al elegir **StorageRead** y **StorageWrite** se registrarán las solicitudes de lectura y escritura del servicio seleccionado.
1. En **Detalles del destino**, seleccione **Enviar a Log Analytics**. Seleccione la suscripción y el área de trabajo de Log Analytics que creó anteriormente, como se muestra en la siguiente imagen.

    :::image type="content" source="media/transport-layer-security-configure-minimum-version/create-diagnostic-setting-logs.png" alt-text="Captura de pantalla que muestra cómo crear una configuración de diagnóstico para el registro de las solicitudes":::

Después de crear la configuración de diagnóstico, las solicitudes a la cuenta de almacenamiento se registran posteriormente según esa configuración. Para más información, consulte [Creación de una configuración de diagnóstico para recopilar registros y métricas en Azure](../../azure-monitor/platform/diagnostic-settings.md).

Puede encontrar una referencia sobre los campos disponibles en los registros de Azure Storage en Azure Monitor en el artículo [Registros de recursos (versión preliminar)](monitor-storage-reference.md#resource-logs-preview).

### <a name="query-logged-requests-by-tls-version"></a>Consulta de las solicitudes registradas por versión de TLS

Los registros de Azure Storage en Azure Monitor incluyen la versión de TLS que se usó para hacer una solicitud a una cuenta de almacenamiento. Use la propiedad **TlsVersion** para comprobar la versión de TLS de una solicitud registrada.

Para determinar cuántas solicitudes se realizaron a Blob Storage con cada versión de TLS durante los últimos siete días, abra su área de trabajo de Log Analytics. Luego, pegue la siguiente consulta en una nueva consulta de registro y ejecútela. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```kusto
StorageBlobLogs
| where TimeGenerated > ago(7d) and AccountName == "<account-name>"
| summarize count() by TlsVersion
```

Los resultados muestran el número de solicitudes realizadas con cada versión de TLS:

:::image type="content" source="media/transport-layer-security-configure-minimum-version/log-analytics-query-version.png" alt-text="Captura de pantalla que muestra los resultados de la consulta de Log Analytics para devolver la versión de TLS":::

### <a name="query-logged-requests-by-caller-ip-address-and-user-agent-header"></a>Consulta de solicitudes registradas por dirección IP del autor de la llamada y por encabezado del agente de usuario

Los registros de Azure Storage en Azure Monitor también incluyen la dirección IP del autor de la llamada y el encabezado del agente de usuario para ayudarle a evaluar qué aplicaciones cliente tienen acceso a la cuenta de almacenamiento. Puede analizar estos valores para determinar si las aplicaciones cliente deben actualizarse para usar una versión más reciente de TLS, o si es aceptable que se produzca un error en la solicitud de un cliente si no se envía con la versión mínima de TLS.

Para determinar qué clientes realizaron solicitudes con una versión de TLS anterior a TLS 1.2 en los últimos siete días, pegue la siguiente consulta en una nueva consulta de registro y ejecútela. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```kusto
StorageBlobLogs
| where TimeGenerated > ago(7d) and AccountName == "<account-name>" and TlsVersion != "TLS 1.2"
| project TlsVersion, CallerIpAddress, UserAgentHeader
```

## <a name="remediate-security-risks-with-a-minimum-version-of-tls"></a>Corrección de riesgos de seguridad con una versión mínima de TLS

Si está seguro de que el tráfico de los clientes que usan versiones anteriores de TLS es mínimo, o que es aceptable que se produzcan errores en las solicitudes realizadas con una versión anterior de TLS, puede empezar a aplicar una versión de TLS mínima en la cuenta de almacenamiento. Requerir que los clientes usen una versión mínima de TLS para realizar solicitudes en una cuenta de almacenamiento forma parte de una estrategia para minimizar los riesgos de seguridad de los datos.

### <a name="configure-the-minimum-tls-version-for-a-storage-account"></a>Configuración de la versión mínima de TLS para una cuenta de almacenamiento

Para configurar la versión mínima de TLS para una cuenta de almacenamiento, establezca la versión **MinimumTlsVersion** de la cuenta. Esta propiedad está disponible para todas las cuentas de almacenamiento que se crean con el modelo de implementación de Azure Resource Manager. Para más información sobre el modelo de implementación de Azure Resource Manager, consulte [Introducción a las cuentas de almacenamiento](storage-account-overview.md).

> [!NOTE]
> La propiedad **minimumTlsVersion** no se establece de forma predeterminada y no devuelve un valor hasta que se establece de forma explícita. La cuenta de almacenamiento permite las solicitudes enviadas con la versión 1.0 o posterior de TLS si el valor de la propiedad es **NULL**.

# <a name="portal"></a>[Portal](#tab/portal)

Para configurar la versión mínima de TLS de una cuenta de almacenamiento con Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. Seleccione el valor de **Configuración**.
1. En **Versión mínima de TLS**, use la lista desplegable para seleccionar la versión mínima de TLS necesaria para acceder a los datos de esta cuenta de almacenamiento, tal como se muestra en la siguiente imagen.

    :::image type="content" source="media/transport-layer-security-configure-minimum-version/configure-minimum-version-portal.png" alt-text="Captura de pantalla que muestra cómo configurar la versión mínima de TLS en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para configurar la versión mínima de TLS de una cuenta de almacenamiento con PowerShell, instale [Azure PowerShell, versión 4.4.0](https://www.powershellgallery.com/packages/Az/4.4.0) o posterior. A continuación, configure la propiedad **MinimumTLSVersion** en una cuenta de almacenamiento nueva o existente. Los valores válidos para **MinimumTLSVersion** son `TLS1_0`, `TLS1_1` y `TLS1_2`.

En el ejemplo siguiente se crea una cuenta de almacenamiento y se establece la propiedad **MinimumTLSVersion** en TLS 1.1 y, a continuación, se actualiza la cuenta y se establece **MinimumTLSVersion** en TLS 1.2. En este ejemplo también se recupera el valor de propiedad en cada caso. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$location = "<location>"

# Create a storage account with MinimumTlsVersion set to TLS 1.1.
New-AzStorageAccount -ResourceGroupName $rgName \
    -AccountName $accountName \
    -Location $location \
    -SkuName Standard_GRS \
    -MinimumTlsVersion TLS1_1

# Read the MinimumTlsVersion property.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).MinimumTlsVersion

# Update the MinimumTlsVersion version for the storage account to TLS 1.2.
Set-AzStorageAccount -ResourceGroupName $rgName \
    -AccountName $accountName \
    -MinimumTlsVersion TLS1_2

# Read the MinimumTlsVersion property.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).MinimumTlsVersion
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para configurar la versión mínima de TLS de una cuenta de almacenamiento con la CLI de Azure, instale la CLI de Azure, versión 2.9.0 o posterior. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). A continuación, configure la propiedad **MinimumTLSVersion** en una cuenta de almacenamiento nueva o existente. Los valores válidos para **MinimumTLSVersion** son `TLS1_0`, `TLS1_1` y `TLS1_2`.

En el ejemplo siguiente se crea una cuenta de almacenamiento y se establece **MinimumTLSVersion** en TLS 1.1. A continuación, se actualiza la cuenta y se establece la propiedad **MinimumTLSVersion** en TLS 1.2. En este ejemplo también se recupera el valor de propiedad en cada caso. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --kind StorageV2 \
    --location <location> \
    --min-tls-version TLS1_1

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query minimumTlsVersion \
    --output tsv

az storage account update \
    --name <storage-account> \
    --resource-group <resource-group> \
    --min-tls-version TLS1_2

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query minimumTlsVersion \
    --output tsv
```

# <a name="template"></a>[Plantilla](#tab/template)

Para configurar la versión mínima de TLS para una cuenta de almacenamiento con una plantilla, cree una plantilla con la propiedad **MinimumTLSVersion** establecida en `TLS1_0`, `TLS1_1`o `TLS1_2`. En los siguientes pasos se muestra cómo crear una plantilla en Azure Portal.

1. En Azure Portal, elija **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
1. Elija la opción de **Template Deployment (implementar mediante plantillas personalizadas) (versión preliminar)** , seleccione **Crear** y, a continuación, elija **Cree su propia plantilla en el editor.** .
1. En el editor de plantillas, pegue el siguiente código JSON para crear una cuenta nueva y establecer la versión mínima de TLS en TLS 1.2. No olvide reemplazar los valores de los marcadores de posición entre corchetes angulares por sus propios valores.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {
            "storageAccountName": "[concat(uniqueString(subscription().subscriptionId), 'tls')]"
        },
        "resources": [
            {
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "location": "<location>",
            "properties": {
                "minimumTlsVersion": "TLS1_2"
            },
            "dependsOn": [],
            "sku": {
              "name": "Standard_GRS"
            },
            "kind": "StorageV2",
            "tags": {}
            }
        ]
    }
    ```

1. Guarde la plantilla.
1. Especifique el parámetro del grupo de recursos y, a continuación, seleccione el botón **Revisar y crear** para implementar la plantilla y crear una cuenta de almacenamiento con la propiedad **MinimumTLSVersion** configurada.

---

> [!NOTE]
> Después de actualizar la versión mínima de TLS para la cuenta de almacenamiento, el cambio puede tardar hasta 30 segundos en propagarse por completo.

La configuración de la versión de TLS mínima requiere la versión 2019-04-01 o posterior del proveedor de recursos de Azure Storage. Para más información, consulte la [API REST del proveedor de recursos de Azure Storage](/rest/api/storagerp/).

### <a name="check-the-minimum-required-tls-version-for-multiple-accounts"></a>Comprobación de la versión mínima de TLS requerida para varias cuentas

Para comprobar la versión mínima de TLS necesaria en un conjunto de cuentas de almacenamiento con un rendimiento óptimo, puede usar Azure Resource Graph Explorer en Azure Portal. Para más información sobre el uso de Resource Graph Explorer, consulte [Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante Azure Resource Graph Explorer](/azure/governance/resource-graph/first-query-portal).

Si la siguiente consulta se ejecuta en Resource Graph Explorer, este devuelve una lista de cuentas de almacenamiento y muestra la versión mínima de TLS de cada cuenta:

```kusto
resources
| where type =~ 'Microsoft.Storage/storageAccounts'
| extend minimumTlsVersion = parse_json(properties).minimumTlsVersion
| project subscriptionId, resourceGroup, name, minimumTlsVersion
```

### <a name="test-the-minimum-tls-version-from-a-client"></a>Prueba de la versión mínima de TLS desde un cliente

Para probar que la versión mínima de TLS necesaria para una cuenta de almacenamiento impida las llamadas realizadas con una versión anterior, puede configurar un cliente para que use una versión anterior de TLS. Para obtener más información acerca de cómo configurar un cliente para que use una versión específica de TLS, consulte [Configuración de la Seguridad de la capa de transporte (TLS) para una aplicación cliente](transport-layer-security-configure-client-version.md).

Cuando un cliente acceda a una cuenta de almacenamiento mediante una versión de TLS que no cumpla con la versión mínima de TLS configurada para la cuenta, Azure Storage devolverá el código de error 400 (solicitud incorrecta) y un mensaje indicando que la versión de TLS que se usó no está permitida para realizar solicitudes en esta cuenta de almacenamiento.

## <a name="use-azure-policy-to-audit-for-compliance"></a>Uso de Azure Policy para auditar el cumplimiento

Si tiene un gran número de cuentas de almacenamiento, tal vez quiera realizar una auditoría para asegurarse de que todas las cuentas están configuradas para la versión mínima de TLS que requiere la organización. Para auditar un conjunto de cuentas de almacenamiento para su cumplimiento, use Azure Policy. Azure Policy es un servicio que puede usar para crear, asignar y administrar directivas que aplican reglas a los recursos de Azure. Azure Policy le permite mantener esos recursos conforme a lo establecido en los estándares corporativos y los contratos de nivel de servicio. Para más información, consulte la [Introducción a Azure Policy](../../governance/policy/overview.md).

### <a name="create-a-policy-with-an-audit-effect"></a>Creación de una directiva con un efecto de auditoría

Azure Policy admite efectos que determinan lo que sucede cuando una regla de directiva se evalúa con respecto a un recurso. El efecto de auditoría crea una advertencia cuando un recurso no cumple los requisitos, pero no detiene la solicitud. Para más información, consulte [Comprender los efectos de Azure Policy](../../governance/policy/concepts/effects.md).

Para crear una directiva con un efecto de auditoría para la versión mínima de TLS con Azure Portal, siga estos pasos:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Definiciones** en la sección **Creación**.
1. Seleccione **Agregar definición de directiva** para crear una nueva definición de directiva.
1. En el campo donde se indica la **ubicación de la definición**, seleccione el botón **Más** para especificar dónde se encuentra el recurso de directiva de auditoría.
1. Escriba un nombre para la directiva. Si lo desea, puede escribir también una descripción y la categoría.
1. En **Regla de directivas**, agregue la siguiente definición de directiva a la sección **policyRule**.

    ```json
    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "not": {
              "field":"Microsoft.Storage/storageAccounts/minimumTlsVersion",
              "equals": "TLS1_2"
            }
          }
        ]
      },
      "then": {
        "effect": "audit"
      }
    }
    ```

1. Guarde la directiva.

### <a name="assign-the-policy"></a>Asignación de la directiva

A continuación, asigne la directiva a un recurso. El ámbito de la directiva corresponde a ese recurso y a todos los recursos que hay debajo del mismo. Para más información sobre la asignación de directivas, consulte [Estructura de asignaciones de Azure Policy](../../governance/policy/concepts/assignment-structure.md).

Para asignar la directiva con Azure Portal, haga lo siguiente:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Asignaciones** en la sección **Creación**.
1. Seleccione **Asignar directiva** para crear una nueva asignación de directiva.
1. En el campo **Ámbito**, seleccione el ámbito de la asignación de directiva.
1. En el campo **Definición de directiva**, seleccione el botón **Más** y, a continuación, seleccione la directiva que definió en la sección anterior de la lista.
1. Escriba un nombre para la asignación de directiva. La descripción es opcional.
1. Deje la opción **Cumplimiento de directivas** como *Habilitada*. Esta configuración no tiene ningún efecto en la directiva de auditoría.
1. Seleccione **Revisar y crear** para crear la asignación.

### <a name="view-compliance-report"></a>Ver el informe de cumplimiento

Una vez asignada la directiva, puede ver el informe de cumplimiento. El informe de cumplimiento de una directiva de auditoría proporciona información sobre las cuentas de almacenamiento que no cumplen con esa directiva. Para más información, consulte [Obtención de datos de cumplimiento de directiva](../../governance/policy/how-to/get-compliance-data.md).

El informe de cumplimiento puede tardar varios minutos en estar disponible después de que se cree la asignación de directiva.

Para ver el informe de cumplimiento en Azure Portal, siga estos pasos:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Cumplimiento**.
1. Filtre los resultados por el nombre de la asignación de directiva que creó en el paso anterior. El informe muestra el número de recursos que no cumplen con la directiva.
1. Puede explorar en profundidad el informe para obtener más detalles, incluida una lista de cuentas de almacenamiento que no cumplen los requisitos.

    :::image type="content" source="media/transport-layer-security-configure-minimum-version/compliance-report-policy-portal.png" alt-text="Captura de pantalla que muestra el informe de cumplimiento de la directiva de auditoría para la versión mínima de TLS":::

## <a name="use-azure-policy-to-enforce-the-minimum-tls-version"></a>Uso de Azure Policy para aplicar la versión mínima de TLS

Azure Policy admite la gobernanza en la nube, asegurándose así de que los recursos de Azure cumplen los requisitos y los estándares establecidos. Para aplicar un requisito mínimo de versión de TLS para las cuentas de almacenamiento de la organización, puede crear una directiva que impida la creación de una nueva cuenta de almacenamiento que establezca un requisito mínimo de TLS en una versión anterior de TLS que la que se indica en la directiva. Esta directiva también impedirá todos los cambios de configuración en una cuenta existente si la configuración de la versión mínima de TLS no es compatible con la directiva.

La directiva de cumplimiento usa el efecto de denegación para impedir que una solicitud cree o modifique una cuenta de almacenamiento para que la versión de TLS mínima ya no se adhiera a los estándares de la organización. Para más información, consulte [Comprender los efectos de Azure Policy](../../governance/policy/concepts/effects.md).

Para crear una directiva con un efecto de denegación para una versión mínima de TLS que sea anterior a TLS 1.2, siga los mismos pasos que se indican en [Uso de Azure Policy para auditar el cumplimiento](#use-azure-policy-to-audit-for-compliance), pero proporcione el siguiente código JSON en la sección **policyRule** de la definición de directivas:

```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Storage/storageAccounts"
      },
      {
        "not": {
          "field":"Microsoft.Storage/storageAccounts/minimumTlsVersion",
          "equals": "TLS1_2"
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

Después de crear la directiva con el efecto de denegación y asignarla a un ámbito, un usuario no puede crear una cuenta de almacenamiento con una versión de TLS mínima que sea anterior a la 1.2. Asimismo, los usuarios tampoco pueden realizar cambios de configuración en una cuenta de almacenamiento existente que actualmente requiera una versión mínima de TLS que sea anterior a la 1.2. Si intentan hacerlo, se producirá un error. La versión mínima de TLS requerida de la cuenta de almacenamiento debe establecerse en 1.2 para continuar con la creación o la configuración de la cuenta.

En la siguiente imagen se muestra el error que se produce si se intenta crear una cuenta de almacenamiento con una versión mínima de TLS establecida en TLS 1.0 (la versión predeterminada para una nueva cuenta) cuando una directiva con un efecto de denegación requiere que la versión mínima de TLS se establezca en TLS 1.2.

:::image type="content" source="media/transport-layer-security-configure-minimum-version/deny-policy-error.png" alt-text="Captura de pantalla que muestra el error que se produce al crear una cuenta de almacenamiento que infringe la directiva":::

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de Seguridad de la capa de transporte (TLS) para una aplicación cliente](transport-layer-security-configure-client-version.md)
- [Recomendaciones de seguridad para Blob Storage](../blobs/security-recommendations.md)
