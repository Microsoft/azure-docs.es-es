---
title: Información sobre cómo auditar el contenido de máquinas virtuales
description: Obtenga información sobre cómo Azure Policy usa Guest Configuration para auditar la configuración dentro de las máquinas virtuales.
ms.date: 05/20/2020
ms.topic: conceptual
ms.openlocfilehash: f2f07a3e88984a84ca1529052d5899ad8570a268
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072828"
---
# <a name="understand-azure-policys-guest-configuration"></a>Información sobre Guest Configuration de Azure Policy

Azure Policy puede auditar la configuración dentro de un equipo, tanto para las máquinas que se ejecutan en Azure como para las [conectadas a Arc](../../../azure-arc/servers/overview.md).
La validación se realiza mediante el cliente y la extensión Guest Configuration. La extensión, a través del cliente, valida la configuración como:

- La configuración del sistema operativo
- Configuración de la aplicación o presencia
- Configuración del entorno

En este momento, la mayoría de directivas de configuración de invitado de Azure Policy solo realiza la auditoría de la configuración dentro de la máquina.
No se aplica a configuraciones. La excepción es una directiva integrada [a la que se hace referencia a continuación](#applying-configurations-using-guest-configuration).

## <a name="enable-guest-configuration"></a>Habilitar configuración de invitado

Para auditar el estado de las máquinas de su entorno, incluidas las de Azure y las conectadas a Arc, revise los detalles siguientes.

## <a name="resource-provider"></a>Proveedor de recursos

Para poder usar Guest Configuration debe registrar el proveedor de recursos. El proveedor de recursos se registra automáticamente si la asignación de una directiva de configuración de invitado se realiza a través del portal. Puede registrarse manualmente mediante el [portal](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal), [Azure PowerShell](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell) o la [CLI de Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).

## <a name="deploy-requirements-for-azure-virtual-machines"></a>Requisitos de implementación de Azure Virtual Machines

Para auditar la configuración en una máquina, se habilita una [extensión de máquina virtual](../../../virtual-machines/extensions/overview.md) y la máquina debe tener una identidad administrada por el sistema. La extensión descarga la asignación de directiva aplicable y la definición de configuración correspondiente. La identidad se usa para autenticar la máquina a medida que lee y escribe en el servicio de configuración de invitado. La extensión no es necesaria para las máquinas conectadas a Arc porque se incluye en el agente Connected Machine de Arc.

> [!IMPORTANT]
> La extensión de configuración de invitado y una identidad administrada son necesarias para realizar auditorías en las máquinas virtuales de Azure. La extensión de configuración de invitado es necesaria para realizar auditorías en las máquinas virtuales de Azure. Para implementar la extensión a escala, asigne la siguiente iniciativa de directiva: > implementar la extensión a escala, asigne las siguientes definiciones de directiva: 
>  - [Implementar los requisitos previos para habilitar directivas de configuración de invitado en máquinas virtuales](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F12794019-7a00-42cf-95c2-882eed337cc8)

### <a name="limits-set-on-the-extension"></a>Límites establecidos en la extensión

Para limitar que la extensión afecte a las aplicaciones que se ejecutan en la máquina, no se permite que la configuración de invitado supere el 5 % de la CPU. Esta limitación existe tanto para definiciones integradas como personalizadas. Lo mismo se aplica al servicio de configuración de invitado en el agente Connected Machine de Arc.

### <a name="validation-tools"></a>Herramientas de validación

Dentro de la máquina, el cliente de Guest Configuration usa herramientas locales para ejecutar la auditoría.

En la tabla siguiente se muestra una lista de las herramientas locales usadas en cada sistema operativo compatible. En el caso de contenido integrado, la configuración de invitado controla la carga de estas herramientas automáticamente.

|Sistema operativo|Herramienta de validación|Notas|
|-|-|-|
|Windows|[Configuración de estado deseado de PowerShell](/powershell/scripting/dsc/overview/overview) v2| Instalado de prueba en una carpeta que solo usa Azure Policy. No entrará en conflicto con DSC de Windows PowerShell. PowerShell Core no se agrega a la ruta de acceso del sistema.|
|Linux|[Chef InSpec](https://www.chef.io/inspec/)| Instala Chef InSpec versión 2.2.61 en la ubicación predeterminada y lo agrega a la ruta de acceso del sistema. También se instalan las dependencias del paquete InSpec, que incluyen Ruby y Python. |

### <a name="validation-frequency"></a>Frecuencia de validación

El cliente de Guest Configuration busca contenido nuevo cada 5 minutos. Una vez que se recibe una asignación de invitado, se comprueban los valores de esa configuración en un intervalo de 15 minutos. Los resultados se envían al proveedor de recursos de Guest Configuration cuando finaliza la auditoría. Cuando se produce una directiva del tipo [desencadenador evaluación](../how-to/get-compliance-data.md#evaluation-triggers), el estado de la máquina se escribe en el proveedor de recursos de Guest Configuration. Esta actualización hace que Azure Policy evalúe las propiedades de Azure Resource Manager. Una evaluación de Azure Policy a petición recupera el valor más reciente del proveedor de recursos de la configuración de invitado. Sin embargo, no desencadena una nueva auditoría de la configuración en la máquina.

## <a name="supported-client-types"></a>Tipos de cliente admitidos

Las directivas de configuración de invitado son inclusivas de nuevas versiones. Las versiones anteriores de los sistemas operativos disponibles en Azure Marketplace se excluyen si el agente de configuración de invitado no es compatible.
En la tabla siguiente se muestra una lista de sistemas operativos compatibles en imágenes de Azure:

|Publicador|Nombre|Versiones|
|-|-|-|
|Canonical|Ubuntu Server|14.04 y versiones posteriores|
|Credativ|Debian|8 y versiones posteriores|
|Microsoft|Windows Server|2012 y versiones posteriores|
|Microsoft|Cliente Windows|Windows 10|
|OpenLogic|CentOS|7.3 y versiones posteriores|
|Red Hat|Red Hat Enterprise Linux|7.4 - 7.8, 9.0 y posteriores|
|Suse|SLES|12 SP3 y versiones posteriores|

Las directivas de configuración de invitado admiten imágenes de máquina virtual personalizadas, siempre y cuando se trate de uno de los sistemas operativos de la tabla anterior.

## <a name="guest-configuration-extension-network-requirements"></a>Requisitos de red de la extensión de configuración de invitado

Para comunicarse con el proveedor de recursos de la configuración de invitado en Azure, las máquinas requieren acceso de salida a los centros de datos Azure en el puerto **443**. Si una red en Azure no permite el tráfico saliente, las excepciones deben configurarse con las reglas del [grupo de seguridad de red](../../../virtual-network/manage-network-security-group.md#create-a-security-rule). La [etiqueta del servicio](../../../virtual-network/service-tags-overview.md) "GuestAndHybridManagement" se puede usar para hacer referencia al servicio de Guest Configuration.

## <a name="managed-identity-requirements"></a>Requisitos de identidad administrada

Las directivas de la iniciativa [Implementar los requisitos previos para habilitar directivas de configuración de invitado en máquinas virtuales](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F12794019-7a00-42cf-95c2-882eed337cc8) habilitan una identidad administrada asignada por el sistema, si no existe ninguna. Hay dos definiciones de directivas en la iniciativa que administran la creación de identidades. Las condiciones IF en las definiciones de directiva garantizan el comportamiento correcto en función del estado actual del recurso de máquina en Azure.

Si la máquina no tiene ninguna identidad administrada actualmente, la directiva efectiva será: [\[Versión preliminar\]: Agregar una identidad administrada asignada por el sistema para habilitar las asignaciones de configuración de invitado en máquinas virtuales sin identidades](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cf2ab00-13f1-4d0c-8971-2ac904541a7e)

Si la maquina tiene actualmente una identidad del sistema asignada por el usuario, la directiva efectiva será: [\[Versión preliminar\]: Agregar una identidad administrada asignada por el sistema para habilitar las asignaciones de configuración de invitado en máquinas virtuales con una identidad asignada por el usuario](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F497dff13-db2a-4c0f-8603-28fa3b331ab6)

## <a name="guest-configuration-definition-requirements"></a>Requisitos de definición de Guest Configuration

Cada auditoría que ejecuta la configuración de invitado requiere dos definiciones de directiva, una definición **DeployIfNotExists** y otra **AuditIfNotExists**. Las definiciones de la directiva **DeployIfNotExists** administran las dependencias para realizar auditorías en cada máquina.

La definición de directiva **DeployIfNotExists** valida y corrige los siguientes elementos:

- Valide que a la máquina se le ha asignado una configuración para evaluar. Si actualmente no hay ninguna asignación, obtiene la asignación y prepara la máquina:
  - Autenticando la máquina con una [identidad administrada](../../../active-directory/managed-identities-azure-resources/overview.md)
  - Instalando la versión más reciente de la extensión **Microsoft.GuestConfiguration**
  - Instalando las [herramientas de validación](#validation-tools) y dependencias, si es necesario

Si la asignación **DeployIfNotExists** no es conforme, se puede usar una [tarea de corrección](../how-to/remediate-resources.md#create-a-remediation-task).

Una vez que la asignación **DeployIfNotExists** es compatible, la asignación de la directiva **AuditIfNotExists** determina si la asignación de invitado es compatible o no compatible. La herramienta de validación proporciona los resultados al cliente de Guest Configuration. El cliente envía los resultados a la extensión de Guest, que hace que estén disponibles a través del proveedor de recursos de Guest Configuration.

Azure Policy usa la propiedad **complianceStatus** de los proveedores de recursos de Guest Configuration para notificar el cumplimiento en el nodo **Compliance**. Para más información, vea [Obtención de datos de cumplimiento](../how-to/get-compliance-data.md).

> [!NOTE]
> La directiva **DeployIfNotExists** es necesaria para que la directiva **AuditIfNotExists** devuelva resultados. Sin la directiva **DeployIfNotExists**, **AuditIfNotExists** muestra "0 de 0" recursos como estado.

Se incluyen todas las directivas integradas para Guest Configuration en una iniciativa para agrupar las definiciones para su uso en las asignaciones. La iniciativa integrada denominada _\[Versión preliminar\]: La seguridad de la contraseña de la auditoría dentro de máquinas Linux y Windows_ contiene 18 directivas. Hay seis pares **DeployIfNotExists** y **AuditIfNotExists** para Windows y tres pares para Linux. La lógica de [definición de directiva](definition-structure.md#policy-rule) valida que solo se evalúa el sistema operativo de destino.

#### <a name="auditing-operating-system-settings-following-industry-baselines"></a>Auditoría de la configuración del sistema operativo siguiendo las líneas de base del sector

Una iniciativa de Azure Policy proporciona la capacidad de auditar la configuración del sistema operativo después de una "línea de base". La definición, _\[Versión preliminar\]: Auditar las VM Windows que no coinciden con la configuración de línea de base de seguridad de Azure_ incluye un conjunto de reglas basadas en la directiva de grupo de Active Directory.

La mayoría de las opciones de configuración están disponibles como parámetros. Los parámetros le permiten personalizar lo que se audita.
Alinee la directiva con sus requisitos o asigne la directiva a información de terceros, como estándares normativos del sector.

Algunos parámetros admiten un intervalo de valores enteros. Por ejemplo, el valor de Vigencia máxima de la contraseña puede auditar la configuración de directiva de grupo efectiva. Un intervalo "1,70" confirmaría que los usuarios deben cambiar sus contraseñas al menos cada 70 días, pero no menos de un día.

Si asigna la directiva mediante una plantilla de Azure Resource Manager (plantilla de ARM), use un archivo de parámetros para administrar excepciones. Proteja los archivos en un sistema de control de versiones, como Git. Los comentarios sobre los cambios en los archivos proporcionan evidencias sobre el motivo por el que una asignación es una excepción al valor esperado.

#### <a name="applying-configurations-using-guest-configuration"></a>Aplicación de configuraciones mediante la configuración de invitado

La última característica de Azure Policy establece la configuración dentro de las máquinas. La definición _Configurar la zona horaria de las máquinas Windows_ realiza cambios en la máquina mediante la configuración de la zona horaria.

Al asignar definiciones que empiezan por _Configurar_, también debe asignar la definición _Implementar los requisitos previos para habilitar la directiva de configuración de invitado en máquinas virtuales Windows._ . Puede combinar estas definiciones en una iniciativa si así lo decide.

#### <a name="assigning-policies-to-machines-outside-of-azure"></a>Asignación de directivas a máquinas fuera de Azure

Las directivas de auditoría disponibles para la configuración de invitado incluyen el tipo de recurso **Microsoft.HybridCompute/machines**. Todas las máquinas incorporadas a [Azure Arc para servidores](../../../azure-arc/servers/overview.md) que se encuentran en el ámbito de asignación de directivas se incluyen automáticamente.

### <a name="multiple-assignments"></a>Asignaciones múltiples

Actualmente, las directivas de configuración de invitado solo admiten la asignación de la misma asignación de invitado una vez por cada máquina, incluso si la asignación de directiva usa parámetros diferentes.

## <a name="client-log-files"></a>Archivos de registro de cliente

La extensión de la configuración de invitado escribe archivos de registro en las siguientes ubicaciones:

Windows: `C:\ProgramData\GuestConfig\gc_agent_logs\gc_agent.log`

Linux: `/var/lib/GuestConfig/gc_agent_logs/gc_agent.log`

Donde `<version>` se refiere al número de versión actual.

### <a name="collecting-logs-remotely"></a>Recopilar registros de forma remota

El primer paso para solucionar problemas de configuraciones o módulos de Configuración de invitado debe ser usar el cmdlet `Test-GuestConfigurationPackage` siguiendo los pasos para [crear una directiva de auditoría de Configuración de invitado para Windows](../how-to/guest-configuration-create.md#step-by-step-creating-a-custom-guest-configuration-audit-policy-for-windows).
Si no se realiza correctamente, la recopilación de registros de cliente puede ayudar a diagnosticar problemas.

#### <a name="windows"></a>Windows

Capture información de archivos de registro con el [comando de ejecución de VM de Azure](../../../virtual-machines/windows/run-command.md). El siguiente script de PowerShell de ejemplo puede ser útil.

```powershell
$linesToIncludeBeforeMatch = 0
$linesToIncludeAfterMatch = 10
$logPath = 'C:\ProgramData\GuestConfig\gc_agent_logs\gc_agent.log'
Select-String -Path $logPath -pattern 'DSCEngine','DSCManagedEngine' -CaseSensitive -Context $linesToIncludeBeforeMatch,$linesToIncludeAfterMatch | Select-Object -Last 10
```

#### <a name="linux"></a>Linux

Capture información de archivos de registro con el [comando de ejecución de VM de Azure](../../../virtual-machines/linux/run-command.md). El siguiente script de Bash de ejemplo puede ser útil.

```Bash
linesToIncludeBeforeMatch=0
linesToIncludeAfterMatch=10
logPath=/var/lib/GuestConfig/gc_agent_logs/gc_agent.log
egrep -B $linesToIncludeBeforeMatch -A $linesToIncludeAfterMatch 'DSCEngine|DSCManagedEngine' $logPath | tail
```

## <a name="guest-configuration-samples"></a>Ejemplos de configuración de invitado

Hay ejemplos de directivas integradas de la configuración de invitado disponibles en las siguientes ubicaciones:

- [Definiciones de directivas integradas: configuración de invitado](../samples/built-in-policies.md#guest-configuration)
- [Iniciativas integradas: configuración de invitado](../samples/built-in-initiatives.md#guest-configuration)
- [Repositorio de GitHub de ejemplos de Azure Policy](https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Guest%20Configuration)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda cómo ver los detalles de cada opción de configuración en la [vista de compatibilidad de la configuración de invitado](../how-to/determine-non-compliance.md#compliance-details-for-guest-configuration).
- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](definition-structure.md).
- Vea la [Descripción de los efectos de directivas](effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
