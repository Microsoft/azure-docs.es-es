---
title: Creación de directivas de Configuración de invitado para Linux
description: Aprenda a crear una directiva de Configuración de invitado de Azure Policy para Linux.
ms.date: 08/17/2020
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4f49732aa2be50b0d8be6f1f3af974121dc9f363
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89076368"
---
# <a name="how-to-create-guest-configuration-policies-for-linux"></a>Creación de directivas de Configuración de invitado para Linux

Antes de crear directivas personalizadas, lea la información general en [Configuración de invitado de Azure Policy](../concepts/guest-configuration.md).
 
Para obtener información sobre cómo crear directivas de Configuración de invitado para Windows, consulte la página [Cómo crear una directiva de configuración de invitados para Windows](./guest-configuration-create.md).

Al auditar Linux, Configuración de invitado usa [Chef InSpec](https://www.inspec.io/). El perfil de InSpec define la condición en la que debe encontrarse la máquina. Si se produce un error en la evaluación de la configuración, se desencadena el efecto **auditIfNotExists** de la directiva y se considera que la máquina **no es compatible**.

Solo se puede usar la [configuración de invitados de Azure Policy](../concepts/guest-configuration.md) para auditar la configuración dentro de las máquinas. Todavía no está disponible la opción para corregir la configuración dentro de las máquinas.

Use las siguientes acciones para crear su propia configuración para validar el estado de una máquina de Azure o que no sea de Azure.

> [!IMPORTANT]
> Las directivas personalizadas con la configuración de invitados son una característica en vista previa (GB).
>
> La extensión de configuración de invitado es necesaria para realizar auditorías en las máquinas virtuales de Azure. Para implementar la extensión a gran escala en todas las máquinas Linux, asigne las siguientes definiciones de directiva: `Deploy prerequisites to enable Guest Configuration Policy on Linux VMs`

## <a name="install-the-powershell-module"></a>Instalación del módulo de PowerShell

El módulo Configuración de invitado automatiza el proceso de creación de contenido personalizado, entre los que se incluyen lo siguiente:

- Creación de un artefacto de contenido de Configuración de invitado (.zip)
- Pruebas automatizadas del artefacto
- Creación de una definición de directiva
- Publicación de la directiva

El módulo se puede instalar en una máquina que ejecute Windows, macOS o Linux con PowerShell 6.2 o una versión posterior que se ejecute localmente, con [Azure Cloud Shell](https://shell.azure.com) o con la [imagen de Azure PowerShell de Docker](https://hub.docker.com/r/azuresdk/azure-powershell-core).

> [!NOTE]
> No se admite la compilación de configuraciones en Linux.

### <a name="base-requirements"></a>Requisitos básicos

Sistemas operativos donde se puede instalar el módulo:

- Linux
- macOS
- Windows

> [!NOTE]
> El cmdlet "test-GuestConfigurationPackage" requiere OpenSSL versión 1.0, debido a una dependencia en OMI. Esto produce un error en cualquier entorno con OpenSSL 1.1 o posterior.

El módulo de recursos de configuración de invitados requiere el siguiente software:

- PowerShell 6.2 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/scripting/install/installing-powershell).
- Azure PowerShell 1.5.0 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/azure/install-az-ps).
  - Solo se requieren los módulos Az "Az.Accounts" y "Az.Resources".

### <a name="install-the-module"></a>Instalación del módulo

Para instalar el módulo **GuestConfiguration** en PowerShell:

1. En una secuencia de comandos de PowerShell, ejecute el siguiente comando:

   ```azurepowershell-interactive
   # Install the Guest Configuration DSC resource module from PowerShell Gallery
   Install-Module -Name GuestConfiguration
   ```

1. Asegúrese de que el módulo se haya importado:

   ```azurepowershell-interactive
   # Get a list of commands for the imported GuestConfiguration module
   Get-Command -Module 'GuestConfiguration'
   ```

## <a name="guest-configuration-artifacts-and-policy-for-linux"></a>Artefactos y directivas de Configuración de invitado para Linux

Incluso en entornos de Linux, Configuración de invitado usa Desired State Configuration como abstracción de lenguaje. La implementación se basa en código nativo (C++), por lo que no es necesario cargar PowerShell. Sin embargo, sí requiere un MOF de configuración que describa los detalles sobre el entorno.
DSC actúa como contenedor de InSpec para normalizar cómo se ejecuta, cómo se proporcionan los parámetros y cómo se devuelve el resultado al servicio. Se requieren pocos conocimientos de DSC al trabajar con contenido personalizado de InSpec.

#### <a name="configuration-requirements"></a>Requisitos de configuración

El nombre de la configuración personalizada debe ser coherente en todas partes. El nombre del archivo .zip para el paquete de contenido, el nombre de la configuración en el archivo MOF y el nombre de la asignación de invitado en la plantilla de Azure Resource Manager (plantilla de ARM) deben ser el mismo.

### <a name="custom-guest-configuration-configuration-on-linux"></a>Opciones personalizadas de la configuración de invitados en Linux

Configuración de invitado en Linux usa el recurso `ChefInSpecResource` para proporcionar al motor el nombre del [perfil de InSpec](https://www.inspec.io/docs/reference/profiles/). **Name** es la única propiedad de recurso requerida. Cree un archivo YaML y un archivo de script de Ruby, tal como se detalla a continuación.

En primer lugar, cree el archivo YaML que usa InSpec. El archivo ofrece información básica sobre el entorno. A continuación encontrará un ejemplo:

```YaML
name: linux-path
title: Linux path
maintainer: Test
summary: Test profile
license: MIT
version: 1.0.0
supports:
    - os-family: unix
```

Guarde este archivo con el nombre `inspec.yml` en una carpeta denominada `linux-path` en el directorio del proyecto.

A continuación, cree el archivo de Ruby con la abstracción de lenguaje de InSpec usada para auditar la máquina.

```Ruby
describe file('/tmp') do
    it { should exist }
end
```

Guarde este archivo con el nombre `linux-path.rb` en una nueva carpeta denominada `controls` dentro del directorio `linux-path`.

Por último, cree una configuración, importe el módulo de recursos **PSDesiredStateConfiguration** y compile la configuración.

```powershell
# Define the configuration and import GuestConfiguration
Configuration AuditFilePathExists
{
    Import-DscResource -ModuleName 'GuestConfiguration'

    Node AuditFilePathExists
    {
        ChefInSpecResource 'Audit Linux path exists'
        {
            Name = 'linux-path'
        }
    }
}

# Compile the configuration to create the MOF files
import-module PSDesiredStateConfiguration
AuditFilePathExists -out ./Config
```

Guarde este archivo con el nombre `config.ps1` en la carpeta del proyecto. Ejecútelo en PowerShell mediante la ejecución de `./config.ps1` en el terminal. Se creará un nuevo archivo MOF.

El comando `Node AuditFilePathExists` no es técnicamente necesario, pero genera un archivo denominado `AuditFilePathExists.mof`, en lugar del valor predeterminado, `localhost.mof`. El hecho de que el nombre de archivo. mof siga la configuración facilita la organización de muchos archivos cuando se trabaja a escala.

Ahora debería tener una estructura de proyecto como la siguiente:

```file
/ AuditFilePathExists
    / Config
        AuditFilePathExists.mof
    / linux-path
        linux-path.yml
        / controls
            linux-path.rb 
```

Los archivos auxiliares deben empaquetarse juntos. La configuración de invitados usa el paquete completado para crear las definiciones de Azure Policy.

El cmdlet `New-GuestConfigurationPackage` crea el paquete. Parámetros del cmdlet `New-GuestConfigurationPackage` al crear el contenido de Linux:

- **Name**: nombre del paquete de configuración de invitados.
- **Configuración**: Ruta de acceso completa del documento de configuración compilado.
- **Ruta de acceso**: ruta de acceso de la carpeta de salida. Este parámetro es opcional. Si no se especifica, el paquete se crea en el directorio actual.
- **ChefProfilePath**: ruta de acceso completa al perfil de InSpec. Este parámetro solo se admite cuando se crea contenido para auditar Linux.

Ejecute el siguiente comando para crear un paquete con la configuración proporcionada en el paso anterior:

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'AuditFilePathExists' `
  -Configuration './Config/AuditFilePathExists.mof' `
  -ChefInSpecProfilePath './'
```

Después de crear el paquete de configuración, pero antes de publicarlo en Azure, puede probar el paquete desde la estación de trabajo o el entorno de CI/CD. El cmdlet `Test-GuestConfigurationPackage` de GuestConfiguration incluye el mismo agente en el entorno de desarrollo que se usa en las máquinas de Azure. Con esta solución, puede realizar pruebas de integración localmente antes de la publicación en entornos de nube con facturación.

Dado que el agente en realidad está evaluando el entorno local, en la mayoría de los casos es necesario ejecutar el cmdlet Test- en la misma plataforma del sistema operativo que se va a auditar.

Parámetros del cmdlet `Test-GuestConfigurationPackage`:

- **Name**: Nombre de la directiva de Configuración de invitado.
- **Parámetro**: Parámetros de directiva proporcionados en formato de tabla hash.
- **Ruta de acceso**: ruta de acceso completa del paquete de configuración de invitados.

Ejecute el siguiente comando para probar el paquete creado en el paso anterior:

```azurepowershell-interactive
Test-GuestConfigurationPackage `
  -Path ./AuditFilePathExists/AuditFilePathExists.zip
```

El cmdlet también admite la entrada de la canalización de PowerShell. Canaliza la salida del cmdlet `New-GuestConfigurationPackage` al cmdlet `Test-GuestConfigurationPackage`.

```azurepowershell-interactive
New-GuestConfigurationPackage -Name AuditFilePathExists -Configuration ./Config/AuditFilePathExists.mof -ChefProfilePath './' | Test-GuestConfigurationPackage
```

El siguiente paso consiste en publicar el archivo en el almacenamiento de blobs. El script siguiente contiene una función que puede usar para automatizar esta tarea. Los comandos usados en la función `publish` requieren el módulo `Az.Storage`.

```azurepowershell-interactive
function publish {
    param(
    [Parameter(Mandatory=$true)]
    $resourceGroup,
    [Parameter(Mandatory=$true)]
    $storageAccountName,
    [Parameter(Mandatory=$true)]
    $storageContainerName,
    [Parameter(Mandatory=$true)]
    $filePath,
    [Parameter(Mandatory=$true)]
    $blobName
    )

    # Get Storage Context
    $Context = Get-AzStorageAccount -ResourceGroupName $resourceGroup `
        -Name $storageAccountName | `
        ForEach-Object { $_.Context }

    # Upload file
    $Blob = Set-AzStorageBlobContent -Context $Context `
        -Container $storageContainerName `
        -File $filePath `
        -Blob $blobName `
        -Force

    # Get url with SAS token
    $StartTime = (Get-Date)
    $ExpiryTime = $StartTime.AddYears('3')  # THREE YEAR EXPIRATION
    $SAS = New-AzStorageBlobSASToken -Context $Context `
        -Container $storageContainerName `
        -Blob $blobName `
        -StartTime $StartTime `
        -ExpiryTime $ExpiryTime `
        -Permission rl `
        -FullUri

    # Output
    return $SAS
}

# replace the $storageAccountName value below, it must be globally unique
$resourceGroup        = 'policyfiles'
$storageAccountName   = 'youraccountname'
$storageContainerName = 'artifacts'

$uri = publish `
  -resourceGroup $resourceGroup `
  -storageAccountName $storageAccountName `
  -storageContainerName $storageContainerName `
  -filePath ./AuditFilePathExists.zip `
  -blobName 'AuditFilePathExists'
```
Una vez que se ha creado y cargado un paquete de directivas personalizadas de Configuración de invitado, cree la definición de la directiva de Configuración de invitado. El cmdlet `New-GuestConfigurationPolicy` toma un paquete de directivas personalizado y crea una definición de directivas.

Parámetros del cmdlet `New-GuestConfigurationPolicy`:

- **ContentUri**: Uri de http(s) público del paquete de contenido de configuración de invitados.
- **DisplayName**: Nombre para mostrar de la directiva.
- **Descripción**: Descripción de la directiva.
- **Parámetro**: Parámetros de directiva proporcionados en formato de tabla hash.
- **Versión**: Versión de la directiva.
- **Ruta de acceso**: Ruta de acceso de destino donde se crean las definiciones de directiva.
- **Plataforma**: Plataforma de destino (Windows/Linux) para la directiva de configuración de invitados y el paquete de contenido.
- **Tag** agrega uno o varios filtros de etiquetas a la definición de directiva.
- **Category** establece el campo de metadatos de categoría en la definición de directiva.

En el ejemplo siguiente se crean las definiciones de directivas en una ruta de acceso especificada desde un paquete de directivas personalizado:

```azurepowershell-interactive
New-GuestConfigurationPolicy `
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Linux file path.' `
    -Description 'Audit that a file path exists on a Linux machine.' `
    -Path './policies' `
    -Platform 'Linux' `
    -Version 1.0.0 `
    -Verbose
```

`New-GuestConfigurationPolicy` se encarga de crear los siguientes archivos:

- **auditIfNotExists.json**
- **deployIfNotExists.json**
- **Initiative.json**

La salida del cmdlet devuelve un objeto que contiene el nombre para mostrar de la iniciativa y la ruta de acceso de los archivos de directiva.

Por último, publique las definiciones de directivas con el cmdlet `Publish-GuestConfigurationPolicy`. El cmdlet solo tiene el parámetro **Path** que apunta a la ubicación de los archivos JSON que creó `New-GuestConfigurationPolicy`.

Para ejecutar el comando Publish, necesita acceso para crear directivas en Azure. Los requisitos de autorización específicos se documentan en la página [Información general de Azure Policy](../overview.md). El mejor rol integrado es **Colaborador de la directiva de recursos**.

```azurepowershell-interactive
Publish-GuestConfigurationPolicy `
  -Path '.\policyDefinitions'
```

 El cmdlet `Publish-GuestConfigurationPolicy` acepta la ruta de acceso de la canalización de PowerShell. Esta característica significa que puede crear los archivos de directiva y publicarlos en un único conjunto de comandos canalizados.

 ```azurepowershell-interactive
 New-GuestConfigurationPolicy `
  -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
  -DisplayName 'Audit Linux file path.' `
  -Description 'Audit that a file path exists on a Linux machine.' `
  -Path './policies' `
 | Publish-GuestConfigurationPolicy
 ```

Con la directiva creada en Azure, el último paso es asignar la iniciativa. Consulte cómo puede asignar la iniciativa con el [portal](../assign-policy-portal.md), la [CLI de Azure](../assign-policy-azurecli.md) y [Azure PowerShell](../assign-policy-powershell.md).

> [!IMPORTANT]
> Las directivas de configuración de invitados **siempre** se deben asignar mediante la iniciativa que combina las directivas _AuditIfNotExists_ y _DeployIfNotExists_. Si solo se asigna la directiva _AuditIfNotExists_, los requisitos previos no se implementan y la directiva siempre muestra que los servidores "0" son compatibles.

La asignación de una definición de directiva con el efecto _DeployIfNotExists_ requiere un nivel de acceso adicional. Para conceder el privilegio mínimo, puede crear una definición de rol personalizada que amplíe **Colaborador de la directiva de recursos**. En el ejemplo siguiente se crea un rol denominado **Resource Policy Contributor DINE** con el permiso adicional _Microsoft.Authorization/roleAssignments/write_.

```azurepowershell-interactive
$subscriptionid = '00000000-0000-0000-0000-000000000000'
$role = Get-AzRoleDefinition "Resource Policy Contributor"
$role.Id = $null
$role.Name = "Resource Policy Contributor DINE"
$role.Description = "Can assign Policies that require remediation."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Authorization/roleAssignments/write")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/$subscriptionid")
New-AzRoleDefinition -Role $role
```

### <a name="using-parameters-in-custom-guest-configuration-policies"></a>Usar parámetros en directivas de configuración de invitados personalizadas

La configuración de invitados admite la invalidación de propiedades de una configuración en tiempo de ejecución. Esta característica significa que los valores del archivo MOF del paquete no tienen que considerarse estáticos. Los valores de invalidación se proporcionan a través de Azure Policy y no afectan al modo en que se crean o se compilan las configuraciones.

Con InSpec, los parámetros se suelen administrar como entrada en tiempo de ejecución o como código mediante atributos. Configuración de invitado ofusca este proceso, por lo que se puede proporcionar la entrada cuando se asigna la directiva. Un archivo de atributos se crea automáticamente dentro de la máquina. No es necesario crear y agregar un archivo al proyecto. Existen dos pasos para agregar parámetros al proyecto de auditoría de Linux.

Defina la entrada en el archivo de Ruby donde cree el script de lo que se va a auditar en la máquina. A continuación encontrará un ejemplo.

```Ruby
attr_path = attribute('path', description: 'The file path to validate.')

describe file(attr_path) do
    it { should exist }
end
```

Los cmdlets `New-GuestConfigurationPolicy` e `Test-GuestConfigurationPolicyPackage` incluyen un parámetro denominado **Parámetros**. Este parámetro toma una tabla hash que incluye todos los detalles sobre cada parámetro y crea automáticamente todas las secciones necesarias de los archivos que se usan para crear cada definición de Azure Policy.

En el ejemplo siguiente se crea una definición de directiva para auditar una ruta de acceso de archivo, donde el usuario proporciona la ruta de acceso en el momento de asignación de la directiva.

```azurepowershell-interactive
$PolicyParameterInfo = @(
    @{
        Name = 'FilePath'                             # Policy parameter name (mandatory)
        DisplayName = 'File path.'                    # Policy parameter display name (mandatory)
        Description = "File path to be audited."      # Policy parameter description (optional)
        ResourceType = "ChefInSpecResource"           # Configuration resource type (mandatory)
        ResourceId = 'Audit Linux path exists'        # Configuration resource property name (mandatory)
        ResourcePropertyName = "AttributesYmlContent" # Configuration resource property name (mandatory)
        DefaultValue = '/tmp'                         # Policy parameter default value (optional)
    }
)

# The hashtable also supports a property named 'AllowedValues' with an array of strings to limit input to a list

New-GuestConfigurationPolicy
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Linux file path.' `
    -Description 'Audit that a file path exists on a Linux machine.' `
    -Path './policies' `
    -Parameter $PolicyParameterInfo `
    -Version 1.0.0
```

En el caso de las directivas de Linux, incluya la propiedad **AttributesYmlContent** en la configuración y sobrescriba los valores según sea necesario. El agente de configuración de invitados crea automáticamente el archivo YAML que usa InSpec para almacenar los atributos. Observe el ejemplo siguiente.

```powershell
Configuration AuditFilePathExists
{
    Import-DscResource -ModuleName 'GuestConfiguration'

    Node AuditFilePathExists
    {
        ChefInSpecResource 'Audit Linux path exists'
        {
            Name = 'linux-path'
            AttributesYmlContent = "path: /tmp"
        }
    }
}
```

## <a name="policy-lifecycle"></a>Ciclo de vida de la directiva

Para publicar una actualización de la definición de la directiva, hay dos campos que requieren atención.

- **Versión**: al ejecutar el cmdlet `New-GuestConfigurationPolicy`, debe especificar un número de versión mayor que el que se ha publicado actualmente. La propiedad actualiza la versión de la asignación de Configuración de invitado para que el agente reconozca el paquete actualizado.
- **contentHash**: el cmdlet `New-GuestConfigurationPolicy` actualiza automáticamente esta propiedad. Es un valor hash del paquete que creó `New-GuestConfigurationPackage`. La propiedad debe ser correcta para el archivo `.zip` que se publica. Si solo se actualiza la propiedad **contentUri**, la extensión no aceptará el paquete de contenido.

La manera más fácil de publicar un paquete actualizado es repetir el proceso que se describe en este artículo y proporcionar un número de versión actualizado. Este proceso garantiza que todas las propiedades se hayan actualizado correctamente.


### <a name="filtering-guest-configuration-policies-using-tags"></a>Filtrado de directivas de Configuración de invitado mediante etiquetas

De forma opcional, las definiciones de directiva que crean los cmdlets en el módulo Configuración de invitado pueden incluir un filtro para las etiquetas. El parámetro **Tag** de `New-GuestConfigurationPolicy` admite una matriz de tablas hash que contiene entradas de etiquetas individuales. Las etiquetas se agregan a la sección `If` de la definición de directiva y no se pueden modificar mediante una asignación de directiva.

A continuación se muestra un fragmento de código de ejemplo de una definición de directiva que filtra por etiquetas.

```json
"if": {
  "allOf" : [
    {
      "allOf": [
        {
          "field": "tags.Owner",
          "equals": "BusinessUnit"
        },
        {
          "field": "tags.Role",
          "equals": "Web"
        }
      ]
    },
    {
      // Original Guest Configuration content will follow
    }
  ]
}
```

## <a name="optional-signing-guest-configuration-packages"></a>Opcional: Firma de paquetes de configuración de invitados

Las directivas personalizadas de Configuración de invitado usan el hash SHA256 para validar que el paquete de directivas no haya cambiado.
Opcionalmente, los clientes también pueden usar un certificado para firmar paquetes y forzar a la extensión de configuración de invitados a permitir solo el contenido firmado.

Para habilitar este escenario, hay dos pasos que debe completar. Ejecute el cmdlet para firmar el paquete de contenido y anexe una etiqueta a las máquinas que deben solicitar la firma del código.

Para usar la característica de validación de firmas, ejecute el cmdlet `Protect-GuestConfigurationPackage` para firmar el paquete antes de publicarlo. Este cmdlet requiere un certificado de "firma de código".

Parámetros del cmdlet `Protect-GuestConfigurationPackage`:

- **Ruta de acceso**: ruta de acceso completa del paquete de configuración de invitados.
- **PublicGpgKeyPath**: ruta de acceso de la clave GPG pública. Este parámetro solo se admite cuando se firma contenido para Linux.

Una buena referencia para la creación de claves GPG para usarlas con máquinas Linux se proporciona en el artículo de Github [Generar una clave GPG nueva](https://help.github.com/en/articles/generating-a-new-gpg-key).

El agente GuestConfiguration espera que la clave pública del certificado esté presente en la ruta de acceso `/usr/local/share/ca-certificates/extra` de las máquinas Linux. Para que el nodo compruebe el contenido firmado, instale la clave pública del certificado en la máquina antes de aplicar la directiva personalizada. Este proceso se puede realizar con cualquier técnica dentro de la máquina virtual o mediante Azure Policy. [Aquí se proporciona](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows) una plantilla de ejemplo.
La directiva de acceso de Key Vault debe permitir que el proveedor de recursos del proceso obtenga acceso a los certificados durante las implementaciones. Para obtener los pasos detallados, consulte [Configuración de Key Vault para máquinas virtuales en Azure Resource Manager](../../../virtual-machines/windows/key-vault-setup.md#use-templates-to-set-up-key-vault).

Una vez publicado el contenido, anexe una etiqueta con el nombre `GuestConfigPolicyCertificateValidation` y el valor `enabled` a todas las máquinas virtuales en las que se debe solicitar la firma de código. Consulte los [ejemplos de etiqueta](../samples/built-in-policies.md#tags) sobre cómo se pueden entregar etiquetas a escala mediante Azure Policy. Una vez que esta etiqueta esté en su lugar, la definición de la directiva que se genera mediante el cmdlet `New-GuestConfigurationPolicy` habilita el requisito a través de la extensión de configuración de invitados.

## <a name="troubleshooting-guest-configuration-policy-assignments-preview"></a>Solución de problemas de asignaciones de directivas de configuración de invitados (versión preliminar)

Hay una herramienta disponible en versión preliminar para ayudarle a solucionar problemas en las asignaciones de configuración de invitados de Azure Policy. La herramienta se encuentra en versión preliminar y se ha publicado en la Galería de PowerShell con el nombre de módulo [Guest Configuration Troubleshooter](https://www.powershellgallery.com/packages/GuestConfigurationTroubleshooter/) (Solucionador de problemas de configuración de invitados).

Para más información sobre los cmdlets de esta herramienta, use el comando Get-Help en PowerShell para mostrar la guía integrada. Como la herramienta tiene actualizaciones frecuentes, es la mejor manera de obtener la información más reciente.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la auditoría de VM con la [configuración de invitados](../concepts/guest-configuration.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](./programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](./get-compliance-data.md).
