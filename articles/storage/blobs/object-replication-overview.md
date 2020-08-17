---
title: Información general de replicación de objetos (versión preliminar)
titleSuffix: Azure Storage
description: La replicación de objetos (vista previa) copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una cuenta de destino. Utilice la replicación de objetos para minimizar la latencia de las solicitudes de lectura, aumentar la eficacia de las cargas de trabajo de proceso, optimizar la distribución de los datos y minimizar los costos.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/28/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: 097b3c71b01a8ad0e930d7aa0d7be46a1e835e1a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495172"
---
# <a name="object-replication-for-block-blobs-preview"></a>Replicación de objetos para blobs en bloques (versión preliminar)

La replicación de objetos (vista previa) copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una cuenta de destino. Algunos escenarios que admite la replicación de objetos son:

- **Minimización de latencia.** La replicación de objetos puede reducir la latencia de las solicitudes de lectura al permitir a los clientes consumir datos de una región que esté más cerca físicamente.
- **Aumentar la eficiencia de las cargas de trabajo de proceso.** Con la replicación de objetos, las cargas de trabajo de proceso pueden procesar los mismos conjuntos de blob en bloques en diferentes regiones.
- **Optimizar la distribución de datos.** Puede procesar o analizar los datos en una sola ubicación y luego replicar sólo los resultados en regiones adicionales.
- **Optimizar los costos.** Después de replicar los datos, puede reducir los costos moviéndolos al nivel de archivo mediante las directivas de administración del ciclo de vida.

El siguiente diagrama muestra cómo la replicación de objetos replica blob en bloques de una cuenta de almacenamiento de origen en una región a cuentas de destino en dos regiones diferentes.

:::image type="content" source="media/object-replication-overview/object-replication-diagram.svg" alt-text="Diagrama que muestra cómo funciona la replicación de objetos":::

Para obtener información sobre cómo configurar la replicación de objetos, consulte [Configurar la replicación de objetos (versión preliminar)](object-replication-configure.md).

## <a name="object-replication-policies-and-rules"></a>Reglas y directivas de replicación de objetos

Al configura la replicación de objetos, se crea una política de replicación que especifica la cuenta de almacenamiento de origen y la cuenta de destino. Una directiva de replicación incluye una o más reglas que especifican un contenedor de origen y un contenedor de destino e indican qué blob en bloques del contenedor de origen se replicarán.

Después de configurar la replicación de objetos, Azure Storage comprueba periódicamente la fuente de cambios de la cuenta de origen y replica asincrónicamente cualquier operación de escritura o borrada a la cuenta de destino. La latencia de replicación depende del tamaño del blob en bloques que se está replicando.

> [!IMPORTANT]
> Debido a que los datos de blob en bloques se replican asincrónicamente, la cuenta de origen y la de destino no están inmediatamente sincronizadas. Actualmente no hay ningún acuerdo de nivel de servicio sobre cuánto tiempo se tarda en replicar los datos en la cuenta de destino.

### <a name="replications-policies"></a>Directivas de replicación

Al configurar la replicación de objetos, se crea una directiva de replicación tanto en la cuenta de origen como en la de destino a través del proveedor de recursos de Azure Storage. La directiva de replicación se identifica mediante un id. de directiva. La directiva en las cuentas de origen y de destino debe tener el mismo id. de directiva para que tenga lugar la replicación.

Una cuenta de almacenamiento puede servir como cuenta de origen para un máximo dos cuentas de destino. Y una cuenta de destino puede tener como máximo dos cuentas de origen. Las cuentas de origen y destino pueden estar en diferentes regiones. Puede configurar directivas de replicación independientes para replicar los datos en cada una de las cuentas de destino.

### <a name="replication-rules"></a>Reglas de replicación

Las reglas de replicación especifican el modo en que Azure Storage replicará los blobs de un contenedor de origen a un contenedor de destino. Puede especificar un máximo de 10 reglas de replicación para cada directiva de replicación. Cada regla define un único contenedor de origen y de destino, y cada contenedor de origen y de destino puede utilizarse en una sola regla.

Al crear una regla de replicación, de forma predeterminada solo se copian los blobs en bloques nuevos que se agreguen posteriormente al contenedor de origen. También puede especificar que se copien tanto los blob en bloques nuevos como los ya existentes, o puede definir un ámbito de copia personalizado que copie los blob en bloques creados a partir de un momento determinado.

También puede especificar uno o varios filtros como parte de una regla de replicación para filtrar los blobs en bloques por prefijo. Cuando se especifica un prefijo, solo los blobs que coinciden con ese prefijo en el contenedor de origen se copiarán en el contenedor de destino.

Los contenedores de origen y de destino deben existir antes de poder especificarlos en una regla. Después de crear la directiva de replicación, el contenedor de destino pasa a ser de solo lectura. Cualquier intento de escribir en el contenedor de destino producirá un error con el código de error 409 (Conflicto). Sin embargo, puede llamar a la operación [Establecer el nivel del blob](/rest/api/storageservices/set-blob-tier) en un blob en el contenedor de destino para moverla al nivel de archivo. Para más información sobre el nivel de archivo, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md#archive-access-tier).

## <a name="about-the-preview"></a>Acerca de la versión preliminar

La replicación de objetos sólo es compatible con las cuentas de almacenamiento v2 de uso general. La replicación de objetos está disponible en las siguientes regiones en vista preliminar:

- Centro de Francia
- Este de Canadá
- Centro de Canadá
- Este de EE. UU. 2
- Centro de EE. UU.

Tanto la cuenta de origen como la de destino deben residir en una de estas regiones para poder utilizar la replicación de objetos. Las cuentas pueden estar en dos regiones diferentes.

Durante la versión preliminar, no hay costos adicionales asociados con la replicación de datos entre cuentas de almacenamiento.

> [!IMPORTANT]
> Esta versión preliminar de replicación de objetos está pensada para usos distintos del de producción. En este momento no hay contratos de nivel de servicio de producción disponibles.

### <a name="prerequisites-for-object-replication"></a>Requisitos previos de la replicación de objetos

La replicación de objetos requiere que estén habilitadas las siguientes características de Azure Storage: 
- [Fuente de cambios](storage-blob-change-feed.md)
- [Control de versiones](versioning-overview.md)

Antes de configurar la replicación de objetos, habilite los requisitos previos. La fuente de cambios debe estar habilitada en la cuenta de origen y el control de versiones de blob debe estar habilitadas tanto en la cuenta de origen como en la de destino. Para obtener más información sobre cómo habilitar estas características, consulte estos artículos:

- [Habilitar y deshabilitar la fuente de cambios](storage-blob-change-feed.md#enable-and-disable-the-change-feed)
- [Habilitación y administración de las versiones de blob](versioning-enable.md)

Asegúrese de registrarse en la fuente de cambios y las versiones preliminares de los blob antes de habilitarlas.

Habilitar la fuente de cambios y el control de versiones de blob puede suponer costos adicionales. Para más información, consulte la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

### <a name="register-for-the-preview"></a>Registro para obtener la versión preliminar

Puede registrarse para la versión preliminar de la replicación de objetos mediante PowerShell o CLI de Azure. Asegúrese de registrarse también para la fuente de cambios y las vistas previas del control de versiones de blobs si aún no lo ha hecho.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para registrarse y obtener la versión preliminar con PowerShell, ejecute los siguientes comandos:

```powershell
# Register for the object replication preview
Register-AzProviderFeature -FeatureName AllowObjectReplication -ProviderNamespace Microsoft.Storage

# Register for change feed (preview)
Register-AzProviderFeature -FeatureName Changefeed -ProviderNamespace Microsoft.Storage

# Register for blob versioning (preview)
Register-AzProviderFeature -FeatureName Versioning -ProviderNamespace Microsoft.Storage

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para registrarse y obtener la versión preliminar con CLI de Azure, ejecute estos comandos en PowerShell:

```azurecli
az feature register --namespace Microsoft.Storage --name AllowObjectReplication
az feature register --namespace Microsoft.Storage --name Changefeed
az feature register --namespace Microsoft.Storage --name Versioning
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-registration-status"></a>Comprobación del estado de registro

Puede comprobar el estado de las solicitudes de registro con PowerShell o CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el estado de las solicitudes de registro con PowerShell, ejecute los siguientes comandos:

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowObjectReplication

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Changefeed

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Versioning
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar el estado de las solicitudes de registro mediante CLI de Azure, ejecute los siguientes comandos:

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/AllowObjectReplication')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Changefeed')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Versioning')].{Name:name,State:properties.state}"
```

---

## <a name="ask-questions-or-provide-feedback"></a>Formule preguntas o envíe comentarios

Para formular preguntas sobre la versión preliminar de la replicación de objetos o para escribir comentarios, póngase en contacto con Microsoft en AzureStorageFeedback@microsoft.com. Las ideas y sugerencias acerca de Azure Storage siempre están en el [Foro de comentarios de Azure Storage](https://feedback.azure.com/forums/217298-storage).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la replicación de objetos (versión preliminar)](object-replication-configure.md)
- [Compatibilidad con la fuente de cambios en Azure Blob Storage (versión preliminar)](storage-blob-change-feed.md)
- [Habilitar y administrar las versiones de blob](versioning-enable.md)
