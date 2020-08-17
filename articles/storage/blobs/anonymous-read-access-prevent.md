---
title: Impedir el acceso de lectura público anónimo a contenedores y blobs
titleSuffix: Azure Storage
description: Obtenga información sobre cómo analizar las solicitudes anónimas a una cuenta de almacenamiento y cómo evitar el acceso anónimo a toda la cuenta de almacenamiento o a alguno de sus contenedores.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/02/2020
ms.author: tamram
ms.reviewer: fryu
ms.openlocfilehash: f46a7927c149009eaf5baddbad2758732d4da758
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534295"
---
# <a name="prevent-anonymous-public-read-access-to-containers-and-blobs"></a>Impedir el acceso de lectura público anónimo a contenedores y blobs

El acceso de lectura público anónimo a contenedores y blobs en Azure Storage es una manera cómoda de compartir datos, pero también puede suponer un riesgo para la seguridad. Es importante obrar con prudencia al administrar el acceso anónimo y entender cómo evaluar el acceso anónimo a los datos. La complejidad operativa, el error humano o el ataque malintencionado contra los datos a los que se puede tener acceso público pueden dar lugar a costosas vulnerabilidades de los datos. Microsoft recomienda permitir el acceso anónimo solo cuando sea necesario para el escenario de la aplicación.

De forma predeterminada, siempre se prohíbe el acceso público a los datos del blob. Sin embargo, la configuración predeterminada de una cuenta de almacenamiento permite a un usuario con los permisos adecuados configurar el acceso público a los contenedores y blobs en una cuenta de almacenamiento. Para mejorar la seguridad, puede deshabilitar todo el acceso público a la cuenta de almacenamiento, independientemente de la configuración de acceso público que haya establecido en un contenedor individual. No permitir el acceso público a la cuenta de almacenamiento impide que un usuario habilite el acceso público a un contenedor de la cuenta. Microsoft recomienda que no permita el acceso público a una cuenta de almacenamiento a menos que su escenario lo requiera. No permitir el acceso público ayuda a evitar las vulneraciones de datos que se producen debido a los accesos anónimos no deseados.

Cuando se deniega el acceso público a blobs en la cuenta de almacenamiento, Azure Storage rechaza todas las solicitudes anónimas a esa cuenta. Una vez que no se permite el acceso público a una cuenta, los contenedores de la misma no se pueden configurar posteriormente para el acceso público. Los contenedores ya configurados para el acceso público ya no aceptarán solicitudes anónimas. Para más información, consulte [Configuración del acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-configure.md).

En este artículo se describe cómo usar un marco de tipo DRAG (detección-corrección-auditoría-gobernanza) para administrar continuamente el acceso público a las cuentas de almacenamiento.

## <a name="detect-anonymous-requests-from-client-applications"></a>Detección de solicitudes anónimas desde aplicaciones cliente

Al no permitir el acceso de lectura público a una cuenta de almacenamiento, se arriesga a rechazar las solicitudes a contenedores y blobs que están configurados actualmente para el acceso público. Cuando no se permite el acceso público a una cuenta de almacenamiento, se invalida la configuración de acceso público de cada contenedor de esta. Además, todas las solicitudes anónimas futuras a esa cuenta generarán un error.

Para comprender de qué forma el impedimento del acceso público puede afectar a las aplicaciones cliente, Microsoft recomienda habilitar el registro y las métricas de esa cuenta y analizar los patrones de solicitudes anónimas a lo largo de un intervalo de tiempo. Use métricas para determinar el número de solicitudes anónimas a la cuenta de almacenamiento y utilice registros para determinar a qué contenedores se tiene acceso anónimo.

### <a name="monitor-anonymous-requests-with-metrics-explorer"></a>Supervisión de las solicitudes anónimas con el Explorador de métricas

Para realizar el seguimiento de las solicitudes anónimas a una cuenta de almacenamiento, use el Explorador de métricas de Azure situado en Azure Portal. Para más información sobre el Explorador de métricas, consulte [Introducción al Explorador de métricas de Azure](../../azure-monitor/platform/metrics-getting-started.md).

Siga estos pasos para crear una métrica que realice el seguimiento de las solicitudes anónimas:

1. Vaya a la cuenta de almacenamiento en Azure Portal. En la sección **Supervisión**, seleccione **Métricas**.
1. Seleccione **Agregar métrica**. En el cuadro de diálogo **Métrica**, especifique los valores siguientes:
    1. Deje el campo Ámbito establecido en el nombre de la cuenta de almacenamiento.
    1. Establezca **Espacio de nombres de métricas** en *Blob*. Esta métrica informará de las solicitudes realizadas solo al almacenamiento de blobs.
    1. Establezca el campo **Métrica** en *Transacciones*.
    1. Establezca el campo **Agregación** en *Suma*.

    La nueva métrica mostrará la suma del número de transacciones en el almacenamiento de blobs a lo largo de un intervalo de tiempo determinado. La métrica resultante aparece como se muestra en la siguiente imagen:

    :::image type="content" source="media/anonymous-read-access-prevent/configure-metric-blob-transactions.png" alt-text="Captura de pantalla que muestra cómo configurar la métrica para sumar transacciones de blobs":::

1. A continuación, seleccione el botón **Agregar filtro** para filtrar la métrica de las solicitudes anónimas.
1. En el cuadro de diálogo **Filtro**, especifique los valores siguientes:
    1. Establezca el valor **Propiedad** en *Autenticación*.
    1. Establezca el campo **Operador** en el signo igual (=).
    1. Establezca el campo **Valores** en *Anónimos*.
1. En la esquina superior derecha, seleccione el intervalo de tiempo del que desea ver la métrica. También puede indicar lo pormenorizada que debe ser la agregación de las solicitudes, especificando intervalos en cualquier punto entre 1 minuto y 1 mes.

Una vez configurada la métrica, las solicitudes anónimas comenzarán a aparecer en el gráfico. En la imagen siguiente se muestran las solicitudes anónimas agregadas en los últimos treinta minutos.

:::image type="content" source="media/anonymous-read-access-prevent/metric-anonymous-blob-requests.png" alt-text="Captura de pantalla que muestra las solicitudes anónimas agregadas al almacenamiento de blobs":::

También puede configurar una regla de alerta para recibir una notificación cuando se realice un determinado número de solicitudes anónimas a su cuenta de almacenamiento. Para más información, vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

### <a name="analyze-logs-to-identify-containers-receiving-anonymous-requests"></a>Análisis de los registros para identificar los contenedores que reciben solicitudes anónimas

Los registros de Azure Storage capturan detalles sobre las solicitudes realizadas a la cuenta de almacenamiento, como el modo en que se autorizó una solicitud. Puede analizar los registros para determinar qué contenedores reciben solicitudes anónimas.

Para registrar las solicitudes a su cuenta de Azure Storage con el fin de evaluar las solicitudes anónimas, puede usar el registro de Azure Storage en Azure Monitor (versión preliminar). Para más información, consulte [Supervisión de Azure Storage](../common/monitor-storage.md).

El registro de Azure Storage en Azure Monitor admite el uso de consultas de registro para analizar los datos de registro. Para consultar los registros, puede usar un área de trabajo de Azure Log Analytics. Para más información sobre las consultas de registro, consulte el [Tutorial: Introducción a las consultas de Log Analytics](../../azure-monitor/log-query/get-started-portal.md).

#### <a name="create-a-diagnostic-setting-in-the-azure-portal"></a>Creación de una configuración de diagnóstico en Azure Portal

Para registrar datos de Azure Storage con Azure Monitor y analizarlos con Azure Log Analytics, primero debe crear una configuración de diagnóstico que indique qué tipos de solicitudes y para qué servicios de almacenamiento quiere registrar los datos. Para crear una configuración de diagnóstico en Azure Portal, siga estos pasos:

1. Inscríbase en la [versión preliminar de registro de Azure Storage en Azure Monitor](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUM0E0MFdPRFpOVTRYVklDSE1WUTcyTVAwOC4u).
1. Cree un área de trabajo de Log Analytics en la suscripción que contenga la cuenta de Azure Storage. Después de configurar el registro de la cuenta de almacenamiento, los registros estarán disponibles en el área de trabajo de Log Analytics. Para obtener más información, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../../azure-monitor/learn/quick-create-workspace.md).
1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En la sección Supervisión, seleccione **Configuración de diagnóstico (versión preliminar)** .
1. Seleccione **Blob** para registrar las solicitudes realizadas al almacenamiento de blobs.
1. Seleccione **Agregar configuración de diagnóstico**.
1. Proporcione un nombre para la configuración de diagnóstico.
1. En **Detalles de la categoría**, en la sección **registro**, elija qué tipos de solicitudes registrar. Todas las solicitudes anónimas serán solicitudes de lectura, así que seleccione **StorageRead** para capturar las solicitudes anónimas.
1. En **Detalles del destino**, seleccione **Enviar a Log Analytics**. Seleccione la suscripción y el área de trabajo de Log Analytics que creó anteriormente, como se muestra en la siguiente imagen.

    :::image type="content" source="media/anonymous-read-access-prevent/create-diagnostic-setting-logs.png" alt-text="Captura de pantalla que muestra cómo crear una configuración de diagnóstico para el registro de las solicitudes":::

Después de crear la configuración de diagnóstico, las solicitudes a la cuenta de almacenamiento se registran posteriormente según esa configuración. Para más información, consulte [Creación de una configuración de diagnóstico para recopilar registros y métricas en Azure](../../azure-monitor/platform/diagnostic-settings.md).

Puede encontrar una referencia de los campos disponibles en los registros de Azure Storage en Azure Monitor en [Registros de recursos (versión preliminar)](../common/monitor-storage-reference.md#resource-logs-preview).

#### <a name="query-logs-for-anonymous-requests"></a>Consulta de registros de solicitudes anónimas

Los registros de Azure Storage en Azure Monitor incluyen el tipo de autorización que se usó para hacer una solicitud a una cuenta de almacenamiento. En la consulta de registro, filtre por la propiedad **AuthenticationType** para ver las solicitudes anónimas.

Para recuperar los registros de los últimos siete días de solicitudes anónimas al almacenamiento de blobs, abra el área de trabajo de Log Analytics. Luego, pegue la siguiente consulta en una nueva consulta de registro y ejecútela:

```kusto
StorageBlobLogs
| where TimeGenerated > ago(7d) and AuthenticationType == "Anonymous"
| project TimeGenerated, AccountName, AuthenticationType, Uri
```

También puede configurar una regla de alerta basada en esta consulta para que le informe sobre las solicitudes anónimas. Para más información, consulte [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../../azure-monitor/platform/alerts-log.md).

## <a name="remediate-anonymous-public-access"></a>Medidas para el acceso público anónimo

Después de haber evaluado las solicitudes anónimas a los contenedores y blobs de la cuenta de almacenamiento, puede tomar medidas para limitar o evitar el acceso público. Si algunos contenedores de la cuenta de almacenamiento deben estar disponibles para el acceso público, puede configurar el valor de acceso público para cada contenedor de la cuenta de almacenamiento. Esta opción proporciona el control más pormenorizado sobre el acceso público. Para más información, consulte [Configuración del nivel de acceso público a un contenedor](anonymous-read-access-configure.md#set-the-public-access-level-for-a-container).

Para mejorar la seguridad, puede impedir el acceso público a toda la cuenta de almacenamiento. La configuración de acceso público de una cuenta de almacenamiento invalida la configuración individual de los contenedores de esa cuenta. Al no permitir el acceso público a una cuenta de almacenamiento, los contenedores que estén configurados para permitirlo ya no estarán accesibles de forma anónima. Para más información, consulte la sección sobre [permitir o no permitir el acceso de lectura público a una cuenta de almacenamiento](anonymous-read-access-configure.md#allow-or-disallow-public-read-access-for-a-storage-account).

Si en su escenario es preciso que determinados contenedores estén disponibles para el acceso público, puede ser aconsejable mover esos contenedores y sus blobs a cuentas de almacenamiento reservadas para el acceso público. Luego, puede impedir el acceso público a cualquier otra cuenta de almacenamiento.

### <a name="verify-that-public-access-to-a-blob-is-not-permitted"></a>Comprobación de que no se permite el acceso público a un blob

Para comprobar que se impide el acceso público a un blob específico, puede intentar descargar el blob a través de su dirección URL. Si la descarga se realiza correctamente, el blob sigue estando disponible públicamente. Si el blob no está accesible públicamente porque se ha impedido el acceso público a la cuenta de almacenamiento, verá un mensaje de error que indica que no se permite el acceso público en esta cuenta de almacenamiento.

En el ejemplo siguiente se muestra cómo usar PowerShell para intentar descargar un blob a través de su dirección URL. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$url = "<absolute-url-to-blob>"
$downloadTo = "<file-path-for-download>"
Invoke-WebRequest -Uri $url -OutFile $downloadTo -ErrorAction Stop
```

### <a name="verify-that-modifying-the-containers-public-access-setting-is-not-permitted"></a>Comprobación de que no se permite modificar la configuración de acceso público del contenedor

Para comprobar que la configuración de acceso público de un contenedor no se puede modificar una vez impedido el acceso público a la cuenta de almacenamiento, puede intentar modificar la configuración. El cambio de la configuración de acceso público del contenedor producirá un error si el acceso público no se permite en la cuenta de almacenamiento.

En el ejemplo siguiente se muestra cómo usar PowerShell para intentar cambiar la configuración de acceso público de un contenedor. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$containerName = "<container-name>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

Set-AzStorageContainerAcl -Context $ctx -Container $containerName -Permission Blob
```

### <a name="verify-that-creating-a-container-with-public-access-enabled-is-not-permitted"></a>Comprobación de que no se permite la creación de un contenedor con acceso público habilitado

Si el acceso público no se permite en la cuenta de almacenamiento, no podrá crear un contenedor con acceso público habilitado. Para comprobarlo, puede intentar crear un contenedor con acceso público habilitado.

En el ejemplo siguiente se muestra cómo usar PowerShell para intentar crear un contenedor con acceso público habilitado. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$containerName = "<container-name>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

New-AzStorageContainer -Name $containerName -Permission Blob -Context $ctx
```

### <a name="check-the-public-access-setting-for-multiple-accounts"></a>Comprobar la configuración de acceso público en varias cuentas

Para comprobar el valor del acceso público en un conjunto de cuentas de almacenamiento con un rendimiento óptimo, puede usar Azure Resource Graph Explorer en Azure Portal. Para más información sobre el uso de Resource Graph Explorer, consulte [Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante Azure Resource Graph Explorer](/azure/governance/resource-graph/first-query-portal).

Si la siguiente consulta se ejecuta en el explorador de Resource Graph, este devuelve una lista de cuentas de almacenamiento y muestra la configuración de acceso público de cada cuenta:

```kusto
resources
| where type =~ 'Microsoft.Storage/storageAccounts'
| extend allowBlobPublicAccess = parse_json(properties).allowBlobPublicAccess
| project subscriptionId, resourceGroup, name, allowBlobPublicAccess
```

## <a name="use-azure-policy-to-audit-for-compliance"></a>Uso de Azure Policy para auditar el cumplimiento

Si tiene un gran número de cuentas de almacenamiento, tal vez quiera realizar una auditoría para asegurarse de que esas cuentas están configuradas para impedir el acceso público. Para auditar un conjunto de cuentas de almacenamiento para su cumplimiento, use Azure Policy. Azure Policy es un servicio que puede usar para crear, asignar y administrar directivas que aplican reglas a los recursos de Azure. Azure Policy le permite mantener esos recursos conforme a lo establecido en los estándares corporativos y los contratos de nivel de servicio. Para más información, consulte la [Introducción a Azure Policy](../../governance/policy/overview.md).

### <a name="create-a-policy-with-an-audit-effect"></a>Creación de una directiva con un efecto de auditoría

Azure Policy admite efectos que determinan lo que sucede cuando una regla de directiva se evalúa con respecto a un recurso. El efecto de auditoría crea una advertencia cuando un recurso no cumple los requisitos, pero no detiene la solicitud. Para obtener más información, consulte [Comprender los efectos de Azure Policy](../../governance/policy/concepts/effects.md).

Para crear una directiva con un efecto de auditoría para la configuración de acceso público de una cuenta de almacenamiento con Azure Portal, siga estos pasos:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Definiciones** en la sección **Creación**.
1. Seleccione **Agregar definición de directiva** para crear una nueva definición de directiva.
1. En el campo donde se indica la **ubicación de la definición**, seleccione el botón **Más** para especificar dónde se encuentra el recurso de directiva de auditoría.
1. Escriba un nombre para la directiva. Si lo desea, puede escribir también una descripción y la categoría.
1. En la **regla de directiva**, agregue la siguiente definición de directiva a la sección **policyRule**.

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
              "field":"Microsoft.Storage/storageAccounts/allowBlobPublicAccess",
              "equals": "false"
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

A continuación, asigne la directiva a un recurso. El ámbito de la directiva corresponde a ese recurso y a todos los recursos que hay debajo del mismo. Para obtener más información sobre la asignación de directivas, consulte [estructura de asignaciones de Azure Policy](../../governance/policy/concepts/assignment-structure.md).

Para asignar la directiva con Azure Portal, haga lo siguiente:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Asignaciones** en la sección **Creación**.
1. Seleccione **Asignar directiva** para crear una nueva asignación de directiva.
1. En el campo del **ámbito**, seleccione el ámbito de la asignación de directiva.
1. En el campo de la **definición de directiva**, seleccione el botón **Más** y, a continuación, seleccione la directiva que definió en la sección anterior de la lista.
1. Escriba un nombre para la asignación de directiva. La descripción es opcional.
1. Deje la opción **Cumplimiento de directivas** como *Habilitada*. Esta configuración no tiene ningún efecto en la directiva de auditoría.
1. Seleccione **Revisar y crear** para crear la asignación.

### <a name="view-compliance-report"></a>Ver el informe de cumplimiento

Una vez asignada la Directiva, puede ver el informe de cumplimiento. El informe de cumplimiento de una directiva de auditoría proporciona información sobre las cuentas de almacenamiento que no cumplen con esa directiva. Para obtener más información, consulte [Obtención de datos de cumplimiento de directivas](../../governance/policy/how-to/get-compliance-data.md).

El informe de cumplimiento puede tardar varios minutos en estar disponible después de que se cree la asignación de directiva.

Para ver el informe de cumplimiento en Azure Portal, siga estos pasos:

1. En Azure Portal, vaya al servicio Azure Policy.
1. Seleccione **Cumplimiento**.
1. Filtre los resultados en función del nombre de la asignación de directiva que creó en el paso anterior. El informe muestra el número de recursos que no cumplen con la directiva.
1. Puede explorar en profundidad el informe para obtener más detalles, incluida una lista de cuentas de almacenamiento que no cumplen los requisitos.

    :::image type="content" source="media/anonymous-read-access-prevent/compliance-report-policy-portal.png" alt-text="Captura de pantalla que muestra el informe de cumplimiento de la directiva de auditoría para el acceso público de blobs":::

## <a name="use-azure-policy-to-enforce-authorized-access"></a>Uso de Azure Policy para aplicar el acceso autorizado

Azure Policy admite la gobernanza en la nube, asegurándose así de que los recursos de Azure cumplen los requisitos y los estándares establecidos. Para asegurarse de que las cuentas de almacenamiento de su organización permiten solo solicitudes autorizadas, puede crear una directiva que impida la creación de una nueva cuenta de almacenamiento con una configuración de acceso público que permita las solicitudes anónimas. Esta directiva también impedirá todos los cambios de configuración en una cuenta existente si la configuración de acceso público de esa cuenta no es compatible con la directiva.

La directiva de cumplimiento usa el efecto de denegación para evitar una solicitud que podría crear o modificar una cuenta de almacenamiento para permitir el acceso público. Para obtener más información, consulte [Comprender los efectos de Azure Policy](../../governance/policy/concepts/effects.md).

Para crear una directiva con un efecto de denegación para una configuración de acceso público que permita solicitudes anónimas, siga los mismos pasos descritos en [Uso de Azure Policy para la auditoría de cumplimiento](#use-azure-policy-to-audit-for-compliance), pero proporcione el siguiente código JSON en la sección **policyRule** de la definición de directivas:

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
          "field":"Microsoft.Storage/storageAccounts/allowBlobPublicAccess",
          "equals": "false"
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

Después de crear la directiva con el efecto de denegación y asignarla a un ámbito, un usuario no puede crear una cuenta de almacenamiento que permita el acceso público. Asimismo, los usuarios tampoco pueden realizar cambios de configuración en una cuenta de almacenamiento existente que actualmente permita el acceso público. Si intentan hacerlo, se producirá un error. La configuración de acceso público de la cuenta de almacenamiento debe establecerse en **falso** para continuar con la creación o la configuración de la cuenta.

En la siguiente imagen se muestra el error que se produce si se intenta crear una cuenta de almacenamiento que permita el acceso público (el valor predeterminado de una nueva cuenta) cuando una directiva con un efecto de denegación requiere que no se permita el acceso público.

:::image type="content" source="media/anonymous-read-access-prevent/deny-policy-error.png" alt-text="Captura de pantalla que muestra el error que se produce al crear una cuenta de almacenamiento que infringe la directiva":::

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-configure.md).
- [Acceso anónimo a contenedores y blobs públicos con .NET](anonymous-read-access-client.md).
