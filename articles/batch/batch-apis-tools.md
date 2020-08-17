---
title: API y herramientas para desarrolladores
description: Obtenga información acerca de las API y herramientas disponibles para desarrollar soluciones con el servicio Azure Batch.
ms.topic: conceptual
ms.date: 05/22/2020
ms.custom: seodec18
ms.openlocfilehash: c7484e59391be8f4853c2394bff20c6d9676dec9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032199"
---
# <a name="overview-of-batch-apis-and-tools"></a>Información general sobre las API y herramientas de Batch

El procesamiento de cargas de trabajo paralelas con Azure Batch se suele realizar mediante programación con una de las API de Batch. Un servicio o una aplicación cliente pueden usar las API de Batch para comunicarse con el servicio Batch. Con las API de Batch se pueden crear y administrar grupos de nodos de proceso, tanto máquinas virtuales como servicios en la nube. A continuación, se puede programar que se ejecuten trabajos y tareas en dichos nodos.

Puede procesar de forma eficiente cargas de trabajo a gran escala para su organización, o bien proporcionar un front-end del servicio a los clientes para que puedan ejecutar trabajos y tareas (a petición o de forma programada) en uno, cientos, o incluso, miles de nodos. Azure Batch también se puede usar como parte de un flujo de trabajo mayor, administrado mediante herramientas como [Azure Data Factory](../data-factory/transform-data-using-dotnet-custom-activity.md?toc=%2fazure%2fbatch%2ftoc.json).

> [!TIP]
> Para más información sobre las características y el flujo de trabajo que se usan en Azure Batch, consulte [Flujo de trabajo y recursos del servicio Batch](batch-service-workflow-features.md).

## <a name="azure-accounts-for-batch-development"></a>Cuentas de Azure para desarrollo con Batch

Cuando se desarrollan soluciones de Batch, se usan las siguientes cuentas de la suscripción de Azure:

- **Cuenta de Batch**: los recursos de Azure Batch, entre los que se incluyen grupos, nodos de proceso, trabajos y tareas, están asociados a una cuenta de [Azure Batch](accounts.md). Cuando la aplicación realiza una solicitud en el servicio Batch, autentica la solicitud mediante el nombre de la cuenta de Azure Batch, la dirección URL de la cuenta y una clave de acceso<o un token de Azure Active Directory. Puede [crear una cuenta de Batch](batch-account-create-portal.md) en Azure Portal o mediante programación.
- **Cuenta de Storage**: Batch incluye compatibilidad integrada para trabajar con archivos en [Azure Storage](../storage/index.yml). Casi todos los escenarios de Batch usan Azure Blob Storage tanto para el almacenamiento provisional de los programas que ejecutan las tareas como de los datos que procesan, así como para el almacenamiento de los datos de salida que generan. Normalmente, cada cuenta de Batch está asociada a una cuenta de almacenamiento correspondiente.

## <a name="service-level-and-management-level-apis"></a>API de nivel de servicio y de administración

Azure Batch tiene dos conjuntos de API, uno para el nivel de servicio y otro para el nivel de administración. La nomenclatura suele ser similar, pero devuelven resultados diferentes.

Solo se realiza un seguimiento de las acciones de las API de administración en el registro de actividad. Las API de nivel de servicio omiten la capa de administración de recursos de Azure (management.azure.com) y no se registran.

Por ejemplo, el destino de la [API del servicio Batch para eliminar un grupo](/rest/api/batchservice/pool/delete) se establece directamente en la cuenta de Batch: `DELETE {batchUrl}/pools/{poolId}`

Sin embargo, el destino de la [API de administración de Batch para eliminar un grupo](/rest/api/batchmanagement/pool/delete) se establece en la capa management.azure.com: `DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Batch/batchAccounts/{accountName}/pools/{poolName}`

## <a name="batch-service-apis"></a>API del servicio Batch

Las aplicaciones y servicios pueden emitir llamadas directas de la API de REST o usar una o varias de las siguientes bibliotecas de cliente para ejecutar y administrar las cargas de trabajo de Azure Batch.

| API | Referencia de API | Descargar | Tutorial | Ejemplos de código | Más información |
| --- | --- | --- | --- | --- | --- |
| **REST de Batch** |[API de REST de Azure - Docs](/rest/api/batchservice/) |N/D |- |- | [Versiones compatibles](/rest/api/batchservice/batch-service-rest-api-versioning) |
| **.NET de Batch** |[Azure SDK para .NET - Docs](/dotnet/api/overview/azure/batch?view=azure-dotnet) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Batch/) |[Tutorial](tutorial-parallel-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) | [Notas de la versión](https://aka.ms/batch-net-dataplane-changelog) |
| **Batch Python** |[Azure SDK para Python - Docs](/python/api/overview/azure/batch/client?view=azure-python) |[PyPI](https://pypi.org/project/azure-batch/) |[Tutorial](tutorial-parallel-python.md)|[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Python/Batch) | [Léame](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/batch/azure-batch/README.md) |
| **Batch Node.js** |[Azure SDK para JavaScript - Docs](/javascript/api/overview/azure/batch/client?view=azure-node-latest) |[npm](https://www.npmjs.com/package/azure-batch) |[Tutorial](batch-nodejs-get-started.md) |- | [Léame](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/batch) |
| **Batch Java** |[Azure SDK para Java - Docs](/java/api/overview/azure/batch?view=azure-java-stable) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Java) | [Léame](https://github.com/Azure/azure-batch-sdk-for-java)|

## <a name="batch-management-apis"></a>API de administración de Batch

Las API de Azure Resource Manager para Batch proporcionan acceso mediante programación a las cuentas de Batch. Gracias a estas API, puede administrar mediante programación las cuentas de Batch, las cuotas, los paquetes de aplicaciones y otros recursos mediante el proveedor Microsoft.Batch.  

| API | Referencia de API | Descargar | Tutorial | Ejemplos de código |
| --- | --- | --- | --- | --- |
| **Batch Management REST** |[API de REST de Azure - Docs](/rest/api/batchmanagement/) |- |- |[GitHub](https://github.com/Azure-Samples/batch-dotnet-manage-batch-accounts) |
| **Batch Management .NET** |[Azure SDK para .NET - Docs](/dotnet/api/overview/azure/batch/management?view=azure-dotnet) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/) | [Tutorial](batch-management-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) |
| **Batch Management Python** |[Azure SDK para Python - Docs](/python/api/overview/azure/batch/management?view=azure-python) |[PyPI](https://pypi.org/project/azure-mgmt-batch/) |- |- |
| **Batch Management Node.js** |[Azure SDK para JavaScript - Docs](/javascript/api/overview/azure/batch/management?view=azure-node-latest) |[npm](https://www.npmjs.com/package/azure-arm-batch) |- |- | 
| **Batch Management Java** |[Azure SDK para Java - Docs](/java/api/overview/azure/batch/management?view=azure-java-stable) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |- |

## <a name="batch-command-line-tools"></a>Herramientas de línea de comandos de Batch

Estas herramientas de línea de comandos proporcionan la misma funcionalidad que el servicio Batch y las API de administración de Batch: 

- [Cmdlets de PowerShell para Batch](/powershell/module/az.batch/): los cmdlets del módulo [Azure PowerShell](/powershell/azure/) para Azure Batch permiten administrar los recursos de Batch con PowerShell.
- [CLI de Azure](/cli/azure): la CLI de Azure es un conjunto de herramientas multiplataforma que proporciona comandos de shell para interactuar con muchos de los servicios de Azure, incluidos el servicio Batch y el servicio de administración de Batch. Consulte [Administración de recursos de Batch con la CLI de Azure](batch-cli-get-started.md) para más información sobre el uso de la CLI de Azure con Batch.

## <a name="other-tools-for-application-development"></a>Otras herramientas de desarrollo de aplicaciones

Estas herramientas adicionales pueden ser útiles para crear y depurar los servicios y las aplicaciones de Batch.

- [Portal de Azure](https://portal.azure.com/): puede crear, supervisar y eliminar grupos, trabajos y tareas de Batch en Azure Portal. La información relativa al estado de estos y otros recursos se puede ver mientras se ejecutan trabajos, e incluso se pueden descargar archivos desde los nodos de proceso de los grupos. Por ejemplo, puede descargar el archivo `stderr.txt` de una tarea con errores mientras soluciona problemas. También puede descargar los archivos de Escritorio remoto (RDP) que puede usar para iniciar sesión en los nodos de proceso.
- [Azure Batch Explorer](https://azure.github.io/BatchExplorer/): Batch Explorer (anteriormente BatchLabs) es una herramienta de cliente independiente, completa y gratuita que puede ayudarle a crear, depurar y supervisar las aplicaciones de Azure Batch. Descargue un [paquete de instalación](https://azure.github.io/BatchExplorer/) para Mac, Linux o Windows.
- [Azure Batch Shipyard](https://github.com/Azure/batch-shipyard): Batch Shipyard es una herramienta para ayudar a aprovisionar, ejecutar y supervisar el procesamiento por lotes basado en contenedores y las cargas de trabajo HPC en Azure Batch.
- [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) aunque no es estrictamente una herramienta de Azure Batch, el Explorador de Storage es otra herramienta muy útil para desarrollar y depurar soluciones de Batch.

## <a name="additional-resources"></a>Recursos adicionales

- Para más información sobre el registro de eventos desde la aplicación de Batch, consulte [Métricas, alertas y registros de Batch para evaluación de diagnóstico y supervisión](batch-diagnostics.md).
- Puede encontrar información de referencia sobre los eventos generados por el servicio Batch en [Análisis de Batch](batch-analytics.md).
- Para más información sobre las variables de entorno para nodos de proceso, consulte [Variables de entorno de tiempo de ejecución de Azure Batch](batch-compute-node-environment-variables.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- [Introducción a la biblioteca de Azure Batch para .NET](tutorial-parallel-dotnet.md) . También hay disponible una versión del tutorial para [Python](tutorial-parallel-python.md) y [Node.js](batch-nodejs-get-started.md).
- Descargue los [ejemplos de código en GitHub](https://github.com/Azure-Samples/azure-batch-samples) para ver la forma en que C# y Python pueden comunicarse con Batch para programar y procesar las cargas de trabajo de ejemplo.
