---
title: Importación y exportación de planos técnicos con PowerShell
description: Obtenga información sobre cómo trabajar con sus definiciones de planos técnicos como código. Compártalas, controle su código fuente y adminístrelas mediante los comandos de exportación e importación.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 87e8f00013fec3abb8132f92c1c717f152d1e264
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051447"
---
# <a name="import-and-export-blueprint-definitions-with-powershell"></a>Importación y exportación de definiciones de planos técnicos con PowerShell

Azure Blueprints se puede administrar totalmente a través de Azure Portal. A medida que las organizaciones avanzan en su uso de Azure Blueprints, deben empezar a pensar en las definiciones de planos técnicos como código administrado. Este concepto suele conocerse como Infraestructura como código (IaC). Tratar sus definiciones de planos técnicos como código ofrece ventajas adicionales más allá de lo que ofrece Azure Portal. Entre las ventajas se incluye lo siguiente:

- Uso compartido de las definiciones de planos técnicos
- Copia de seguridad de las definiciones de planos técnicos
- Reutilización de las definiciones de planos técnicos en diferentes inquilinos o suscripciones
- Colocación de las definiciones de planos técnicos en el control de código fuente
  - Pruebas automatizadas de las definiciones de planos técnicos en entornos de prueba
  - Compatibilidad con las canalizaciones de integración e implementación continuas (CI/CD)

Independientemente de las razones, la administración de sus definiciones de planos técnicos como código tiene ventajas. En este artículo se muestra cómo usar los comandos `Import-AzBlueprintWithArtifact` y `Export-AzBlueprintWithArtifact` en el módulo [Az.Blueprint](https://powershellgallery.com/packages/Az.Blueprint/).

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se presuponen conocimientos prácticos moderados de Azure Blueprints. Si aún no lo ha hecho, trabaje en los siguientes artículos:

- [Creación de un plano técnico en el portal](../create-blueprint-portal.md)
- Obtenga información sobre las [fases de implementación](../concepts/deployment-stages.md) y [el ciclo de vida del plano técnico](../concepts/lifecycle.md)
- [Creación](../create-blueprint-powershell.md) y [administración](./manage-assignments-ps.md) de asignaciones y definiciones de planos técnicos con PowerShell

Si aún no se ha instalado, siga las instrucciones en [Adición del módulo Az.Blueprint](./manage-assignments-ps.md#add-the-azblueprint-module) para instalar y validar el módulo **Az.Blueprint** desde la Galería de PowerShell.

## <a name="folder-structure-of-a-blueprint-definition"></a>Estructura de carpetas de una definición de plano técnico

Antes de examinar la exportación e importación de planos técnicos, echemos un vistazo a cómo se estructuran los archivos que constituyen la definición de plano técnico. Una definición de plano técnico se debe almacenar en su propia carpeta.

> [!IMPORTANT]
> Si no se pasa ningún valor al parámetro **Name** del cmdlet `Import-AzBlueprintWithArtifact`, se usará el nombre de la carpeta donde se almacena la definición de plano técnico.

Junto con la definición de plano técnico, que debe denominarse `blueprint.json`, se encuentran los artefactos de los que se compone la definición de plano técnico. Cada artefacto debe incluirse en la subcarpeta denominada `artifacts`.
Resumiendo, la estructura de su definición de plano técnico como archivos JSON en carpetas debe tener el siguiente aspecto:

```text
.
|
|- MyBlueprint/  _______________ # Root folder name becomes default name of blueprint definition
|  |- blueprint.json  __________ # The blueprint definition. Fixed name.
|
|  |- artifacts/  ______________ # Subfolder for all blueprint artifacts. Fixed name.
|     |- artifact.json  ________ # Blueprint artifact as JSON file. Artifact named from file.
|     |- ...
|     |- more-artifacts.json

```

## <a name="export-your-blueprint-definition"></a>Exportación de su definición de plano técnico

Los pasos para exportar su definición de plano técnico son sencillos. La exportación de la definición de plano técnico puede ser útil para el uso compartido, la realización de copia de seguridad o la colocación en el control de código fuente.

- **Blueprint** [obligatorio]
  - Especifica la definición de plano técnico
  - Use `Get-AzBlueprint` para obtener el objeto de referencia.
- **OutputPath** [obligatorio]
  - Especifica la ruta de acceso en la que guardar los archivos JSON de definición de plano técnico
  - Los archivos de salida están en una subcarpeta con el nombre de la definición de plano técnico
- **Versión** (opcional)
  - Especifica la versión que se va a producir si el objeto de referencia **Blueprint** contiene referencias a más de una versión.

1. Obtenga una referencia a la definición de plano técnico que se va a exportar desde la suscripción representada como `{subId}`:

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   # Get version '1.1' of the blueprint definition in the specified subscription
   $bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'MyBlueprint' -Version '1.1'
   ```

1. Use el cmdlet `Export-AzBlueprintWithArtifact` para exportar la definición de plano técnico especificada:

   ```azurepowershell-interactive
   Export-AzBlueprintWithArtifact -Blueprint $bpDefinition -OutputPath 'C:\Blueprints'
   ```

## <a name="import-your-blueprint-definition"></a>Importación de su definición de plano técnico

Una vez que tenga una [definición de plano técnico exportada](#export-your-blueprint-definition) o tenga una definición de plano técnico creada manualmente en la [estructura de carpetas necesaria](#folder-structure-of-a-blueprint-definition), puede importar esa definición de plano técnico a una suscripción o grupo de administración diferentes.

Para obtener ejemplos de definiciones de planos técnicos integradas, consulte el [repositorio de GitHub de Azure Blueprints](https://github.com/Azure/azure-blueprints/tree/master/samples/001-builtins).

- **Name** [obligatorio]
  - Especifica el nombre para la nueva definición de plano técnico.
- **InputPath** [obligatorio]
  - Especifica la ruta de acceso a partir de la que crear la definición de plano técnico
  - Debe coincidir con la [estructura de carpetas necesaria](#folder-structure-of-a-blueprint-definition)
- **ManagementGroupId** (opcional)
  - El identificador de grupo de administración en el que guardar la definición de plano técnico si no se trata del valor predeterminado del contexto actual
  - **ManagementGroupId** o **SubscriptionId** deben especificarse
- **SubscriptionId** (opcional)
  - El identificador de suscripción en el que guardar la definición de plano técnico si no se trata del valor predeterminado del contexto actual
  - **ManagementGroupId** o **SubscriptionId** deben especificarse

1. Use el cmdlet `Import-AzBlueprintWithArtifact` para importar la definición de plano técnico especificada:

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   Import-AzBlueprintWithArtifact -Name 'MyBlueprint' -ManagementGroupId 'DevMG' -InputPath 'C:\Blueprints\MyBlueprint'
   ```

Una vez importada la definición de plano técnico, [asígnela con PowerShell](./manage-assignments-ps.md#create-blueprint-assignments).

Para obtener información sobre cómo crear definiciones de planos técnicos avanzadas, consulte los siguientes artículos:

- Use [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Personalice el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Proteja las implementaciones con el [bloqueo de recursos de plano técnico](../concepts/resource-locking.md).
- [Administre planos técnicos como código](https://github.com/Azure/azure-blueprints/blob/master/README.md).

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.