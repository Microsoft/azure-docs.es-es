---
title: Configuración de las claves administradas por el cliente para la cuenta de Azure Cosmos DB
description: Aprenda a configurar las claves administradas por el cliente para su cuenta de Azure Cosmos DB con Azure Key Vault.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 08/05/2020
ms.author: thweiss
ms.openlocfilehash: 9fa899e0f0de3b263baad7e44ed24d32d735b001
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836518"
---
# <a name="configure-customer-managed-keys-for-your-azure-cosmos-account-with-azure-key-vault"></a>Configuración de claves administradas por el cliente para una cuenta de Azure Cosmos con Azure Key Vault

Los datos almacenados en su cuenta de Azure Cosmos se cifran de forma automática y sin problemas con claves administradas por Microsoft (**claves administradas por el servicio**). También puede optar por agregar una segunda capa de cifrado con las claves administradas (**claves administradas por el cliente**).

:::image type="content" source="./media/how-to-setup-cmk/cmk-intro.png" alt-text="Capas de cifrado en torno a los datos de clientes":::

Debe almacenar las claves administradas por el cliente en [Azure Key Vault](../key-vault/general/overview.md) y proporcionar una clave para cada cuenta de Azure Cosmos que tenga habilitadas las claves administradas por el cliente. Esta clave se usa para cifrar todos los datos almacenados en esa cuenta.

> [!NOTE]
> Actualmente, las claves administradas por el cliente solo están disponibles para las nuevas cuentas de Azure Cosmos. Debe configurarlas durante la creación de la cuenta.

## <a name="register-the-azure-cosmos-db-resource-provider-for-your-azure-subscription"></a><a id="register-resource-provider"></a> Registro del proveedor de recursos de Azure Cosmos DB para su suscripción a Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com/), vaya a la suscripción de Azure y seleccione **Proveedores de recursos** en la pestaña **Configuración**:

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp.png" alt-text="Entrada Proveedores de recursos en el menú izquierdo":::

1. Busque el proveedor de recursos **Microsoft.DocumentDB**. Compruebe si el proveedor de recursos ya está marcado como registrado. Si no es así, elija el proveedor de recursos y seleccione **Registrar**:

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp-register.png" alt-text="Registro del proveedor de recursos Microsoft.DocumentDB":::

## <a name="configure-your-azure-key-vault-instance"></a>Configuración de la instancia de Azure Key Vault

El uso de claves administradas por el cliente con Azure Cosmos DB requiere que establezca dos propiedades en la instancia de Azure Key Vault que planea usar para hospedar las claves de cifrado: **Eliminación temporal** y **Protección de purga**.

Si crea una nueva instancia de Azure Key Vault, habilite estas propiedades durante la creación:

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-prop.png" alt-text="Habilitación de la eliminación temporal y la protección de purga para una nueva instancia de Azure Key Vault":::

Si usa una instancia de Azure Key Vault existente y desea verificar si estas propiedades estén habilitadas, puede consultar la sección **Propiedades** en Azure Portal. Si alguna de estas propiedades no está habilitada, consulte las secciones "Habilitar la eliminación temporal" y "Habilitación de la protección de purgas" en cualquiera de los siguientes artículos:

- [Uso de la eliminación temporal con PowerShell](../key-vault/general/soft-delete-powershell.md)
- [Uso de la eliminación temporal con la CLI de Azure](../key-vault/general/soft-delete-cli.md)

## <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Adición de una directiva de acceso a la instancia de Azure Key Vault

1. En Azure Portal, vaya a la instancia de Azure Key Vault que planea usar para hospedar las claves de cifrado. Seleccione **Directivas de acceso** en el menú izquierdo:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-ap.png" alt-text="Directivas de acceso en el menú izquierdo":::

1. Seleccione **+ Agregar directiva de acceso**.

1. En el menú desplegable **Permisos de claves**, seleccione los permisos **Obtener**, **Desencapsular clave** y **Encapsular clave**:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap-perm2.png" alt-text="Selección de los permisos correctos":::

1. En **Seleccionar entidad de seguridad**, seleccione **Ninguna seleccionada**. A continuación, busque la entidad de seguridad de **Azure Cosmos DB** y selecciónela (para que sea más fácil de encontrar, también puede buscar por el identificador de la entidad de seguridad: `a232010e-820c-4083-83bb-3ace5fc29d0b` en cualquier región de Azure, excepto en las regiones de Azure Government donde el identificador de la entidad de seguridad es `57506a73-e302-42a9-b869-6f12d9ec29e9`). Por último, elija **Seleccionar** en la parte inferior. Si la entidad de seguridad **Azure Cosmos DB** no está en la lista, es posible que tenga que volver a registrar el proveedor de recursos **Microsoft.DocumentDB** como se describe en la sección [Registro del proveedor de recursos](#register-resource-provider) de este artículo.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap.png" alt-text="Selección de la entidad de seguridad de Azure Cosmos DB":::

1. Seleccione **Agregar** para agregar la nueva directiva de acceso.

1. Seleccione **Guardar** en la instancia de Key Vault para guardar todos los cambios.

## <a name="generate-a-key-in-azure-key-vault"></a>Generación de una clave en Azure Key Vault

1. En Azure Portal, vaya a la instancia de Azure Key Vault que planea usar para hospedar las claves de cifrado. A continuación, seleccione **Claves** en el menú izquierdo:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keys.png" alt-text="Entrada Claves en el menú izquierdo":::

1. Seleccione **Generar/Importar**, proporcione un nombre para la nueva clave y seleccione un tamaño de clave RSA. Para mejorar la seguridad, se recomienda un mínimo de 3072. A continuación, seleccione **Crear**:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-gen.png" alt-text="Creación de una nueva clave":::

1. Una vez creada la clave, seleccione la clave que se acaba de crear y, a continuación, su versión actual.

1. Copie el **Identificador de clave** de la clave, excepto la parte que se encuentra después de la última barra diagonal:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keyid.png" alt-text="Copia del identificador de clave de la clave":::

## <a name="create-a-new-azure-cosmos-account"></a>Creación de una cuenta de Azure Cosmos

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Al crear una nueva cuenta de Azure Cosmos DB desde Azure Portal, elija **Clave administrada por el cliente** en el paso **Cifrado**. En el campo **URI de clave**, pegue el URI o identificador de clave de la clave de Azure Key Vault que copió del paso anterior:

:::image type="content" source="./media/how-to-setup-cmk/portal-cosmos-enc.png" alt-text="Configuración de los parámetros de CMK en Azure Portal":::

### <a name="using-azure-powershell"></a><a id="using-powershell"></a> Con Azure PowerShell

Al crear una nueva cuenta de Azure Cosmos DB mediante PowerShell:

- Pase el identificador URI de la clave de Azure Key Vault que copió anteriormente de la propiedad **keyVaultKeyUri** en **PropertyObject**.

- Use **2019-12-12** o posterior como versión de la API.

> [!IMPORTANT]
> Debe establecer la propiedad `locations` de manera explícita para que la cuenta se cree correctamente con claves administradas por el cliente.

```powershell
$resourceGroupName = "myResourceGroup"
$accountLocation = "West US 2"
$accountName = "mycosmosaccount"

$failoverLocations = @(
    @{ "locationName"="West US 2"; "failoverPriority"=0 }
)

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$failoverLocations;
    "keyVaultKeyUri" = "https://<my-vault>.vault.azure.net/keys/<my-key>";
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2019-12-12" -ResourceGroupName $resourceGroupName `
    -Location $accountLocation -Name $accountName -PropertyObject $CosmosDBProperties
```

Una vez que cree la cuenta, puede verificar que las claves administradas por el cliente se hayan habilitado mediante la captura del URI de la clave de Azure Key Vault:

```powershell
Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
    -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    | Select-Object -ExpandProperty Properties `
    | Select-Object -ExpandProperty keyVaultKeyUri
```

### <a name="using-an-azure-resource-manager-template"></a>Uso de una plantilla de Azure Resource Manager

Al crear una nueva cuenta de Azure Cosmos con una plantilla de Azure Resource Manager:

- Pase el URI de la clave de Azure Key Vault que copió anteriormente en la propiedad **keyVaultKeyUri** en el objeto **properties**.

- Use **2019-12-12** o posterior como versión de la API.

> [!IMPORTANT]
> Debe establecer la propiedad `locations` de manera explícita para que la cuenta se cree correctamente con claves administradas por el cliente.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "keyVaultKeyUri": {
            "type": "string"
        }
    },
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[parameters('accountName')]",
            "apiVersion": "2019-12-12",
            "kind": "GlobalDocumentDB",
            "location": "[parameters('location')]",
            "properties": {
                "locations": [ 
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    }
                ],
                "databaseAccountOfferType": "Standard",
                "keyVaultKeyUri": "[parameters('keyVaultKeyUri')]"
            }
        }
    ]
}
```

Implemente la plantilla con el siguiente script de PowerShell:

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

### <a name="using-the-azure-cli"></a><a id="using-azure-cli"></a> Con la CLI de Azure

Al crear una nueva cuenta de Azure Cosmos a través de la CLI de Azure, pase el URI de la clave de Azure Key Vault que copió anteriormente en el parámetro `--key-uri`.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
keyVaultKeyUri = 'https://<my-vault>.vault.azure.net/keys/<my-key>'

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --key-uri $keyVaultKeyUri
```

Una vez que cree la cuenta, puede verificar que las claves administradas por el cliente se hayan habilitado mediante la captura del URI de la clave de Azure Key Vault:

```azurecli-interactive
az cosmosdb show \
    -n $accountName \
    -g $resourceGroupName \
    --query keyVaultKeyUri
```

## <a name="key-rotation"></a>Rotación de claves

La rotación de la clave administrada por el cliente que usa la cuenta de Azure Cosmos se puede realizar de dos maneras.

- Cree una nueva versión de la clave que se usa actualmente en Azure Key Vault:

  :::image type="content" source="./media/how-to-setup-cmk/portal-akv-rot.png" alt-text="Creación de una nueva versión de la clave":::

- Intercambie la clave que se usa en este momento con una totalmente diferente mediante la actualización del URI de la clave en la cuenta. En Azure Portal, vaya a su cuenta de Azure Cosmos y seleccione **Cifrado de datos** en el menú de la izquierda:

    :::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Entrada de menú de cifrado de datos":::

    A continuación, reemplace el **URI de clave** por la nueva clave que desee usar y seleccione **Guardar**:

    :::image type="content" source="./media/how-to-setup-cmk/portal-key-swap.png" alt-text="Actualización del URI de la clave":::

    Aquí se muestra cómo lograr el mismo resultado en PowerShell:

    ```powershell
    $resourceGroupName = "myResourceGroup"
    $accountName = "mycosmosaccount"
    $newKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-new-key>"
    
    $account = Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
        -ResourceType "Microsoft.DocumentDb/databaseAccounts"
    
    $account.Properties.keyVaultKeyUri = $newKeyUri
    
    $account | Set-AzResource -Force
    ```

La versión de la clave o la clave anterior pueden deshabilitarse después de 24 horas, o bien después de que los [registros de auditoría de Azure Key Vault](../key-vault/general/logging.md) no muestren actividad de Azure Cosmos DB en esa versión.
    
## <a name="error-handling"></a>Control de errores

Al usar claves administradas por el cliente (CMK) en Azure Cosmos DB, si hay algún error, Azure Cosmos DB devuelve los detalles del error junto con un código de subestado HTTP en la respuesta. Puede usar este código de subestado para depurar la causa principal del problema. Consulte el artículo [Códigos de estado HTTP para Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) para obtener la lista de códigos de subestado HTTP admitidos.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="is-there-an-additional-charge-to-enable-customer-managed-keys"></a>¿Existe algún cargo adicional al habilitar las claves administradas por el cliente?

No, no hay ningún cargo por habilitar esta característica.

### <a name="how-do-customer-managed-keys-impact-capacity-planning"></a>¿De qué manera las claves administradas por el cliente afectan al planeamiento de capacidad?

Al usar claves administradas por el cliente, las [Unidades de solicitud](./request-units.md) utilizadas por las operaciones de base de datos ven un aumento para reflejar el procesamiento adicional necesario para realizar el cifrado y descifrado de los datos. Esto puede dar lugar a un uso ligeramente mayor de la capacidad aprovisionada. Emplee la tabla siguiente como guía:

| Tipo de operación | Aumento de las Unidades de solicitud |
|---|---|
| Lecturas de punto (captura de elementos por su identificador) | + 5 % por operación |
| Cualquier operación de escritura | + 6 % por operación<br/>aprox. + 0,06 RU por propiedad indexada |
| Consultas, lectura de la fuente de cambios o fuente de conflictos | + 15 % por operación |

### <a name="what-data-gets-encrypted-with-the-customer-managed-keys"></a>¿Qué datos se cifran con las claves administradas por el cliente?

Todos los datos almacenados en la cuenta de Azure Cosmos se cifran con las claves administradas por el cliente, excepto los metadatos siguientes:

- Los nombres de las [cuentas, bases de datos y contenedores](./account-overview.md#elements-in-an-azure-cosmos-account) de Azure Cosmos DB

- Los nombres de los [procedimientos almacenados](./stored-procedures-triggers-udfs.md)

- Las rutas de acceso a las propiedades declaradas en las [directivas de indexación](./index-policy.md)

- Los valores de la [clave de partición](./partitioning-overview.md) de los contenedores

### <a name="are-customer-managed-keys-supported-for-existing-azure-cosmos-accounts"></a>¿Son compatibles las claves administradas por el cliente con las cuentas existentes de Azure Cosmos?

Esta característica solo está disponible actualmente para las cuentas nuevas.

### <a name="is-there-a-plan-to-support-finer-granularity-than-account-level-keys"></a>¿Existe un plan para admitir una granularidad más fina que las claves de nivel de cuenta?

Actualmente, no. Sin embargo, se está analizando la posibilidad de incluir claves de nivel de contenedor.

### <a name="how-can-i-tell-if-customer-managed-keys-are-enabled-on-my-azure-cosmos-account"></a>¿Cómo se puede saber si las claves administradas por el cliente están habilitadas en mi cuenta de Azure Cosmos?

En Azure Portal, vaya a su cuenta de Azure Cosmos y observe la entrada de **Cifrado de datos** del menú de la izquierda. Si esta entrada existe, las claves administradas por el cliente estarán habilitadas en su cuenta:

:::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Entrada de menú de cifrado de datos":::

También puede recuperar los detalles de la cuenta de Azure Cosmos mediante programación y buscar la presencia de la propiedad `keyVaultKeyUri`. Consulte anteriormente para conocer formas de hacerlo [en PowerShell](#using-powershell) y [mediante la CLI de Azure](#using-azure-cli).

### <a name="how-do-customer-managed-keys-affect-a-backup"></a>¿Cómo afectan las claves administradas por el cliente a una copia de seguridad?

Azure Cosmos DB realiza [copias de seguridad periódicas y automáticas](./online-backup-and-restore.md) de los datos almacenados en su cuenta. Esta operación realiza una copia de seguridad de los datos cifrados. Para usar la copia de seguridad restaurada, se requiere la clave de cifrado que se usó en el momento de la copia de seguridad. Esto significa que no se ha realizado ninguna revocación y que la versión de la clave que se usó en el momento de la copia de seguridad aún estará habilitada.

### <a name="how-do-i-revoke-an-encryption-key"></a>¿Cómo se revoca una clave de cifrado?

Para revocar una clave, se debe deshabilitar la versión más reciente de la clave:

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev2.png" alt-text="Deshabilitación de la versión de una clave":::

Como alternativa, para revocar todas las claves de una instancia de Azure Key Vault, puede eliminar la directiva de acceso concedida a la entidad de seguridad de Azure Cosmos DB:

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev.png" alt-text="Eliminación de la directiva de acceso para la entidad de seguridad de Azure Cosmos DB":::

### <a name="what-operations-are-available-after-a-customer-managed-key-is-revoked"></a>¿Qué operaciones hay disponibles después de la revocación de una clave administrada por el cliente?

La única operación posible cuando se ha revocado la clave de cifrado es la eliminación de la cuenta.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [cifrado de datos en Azure Cosmos DB](./database-encryption-at-rest.md).
- Obtenga información general sobre el [acceso seguro a los datos en Cosmos DB](secure-access-to-data.md).
