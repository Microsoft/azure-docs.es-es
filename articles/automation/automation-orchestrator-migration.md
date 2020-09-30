---
title: Migración de Orchestrator a Azure Automation (versión beta)
description: En este artículo se describe cómo migrar runbooks y paquetes de integración de Orchestrator a Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: a47f720344a16d0f77559d6aabfb2b0245e62976
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426340"
---
# <a name="migrate-from-orchestrator-to-azure-automation-beta"></a>Migración de Orchestrator a Azure Automation (versión beta)

Los runbooks de [System Center 2012: Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh237242(v=sc.12)) se basan en actividades provenientes de paquetes de integración escritos específicamente para Orchestrator, mientras que los runbooks de Azure Automation se basan en Windows PowerShell. [Runbooks gráficos](automation-runbook-types.md#graphical-runbooks) de Azure Automation tienen una apariencia similar a los runbooks de Orchestrator, donde sus actividades representan cmdlets de PowerShell, runbooks secundarios y recursos. Además de convertir los runbooks, debe convertir los paquetes de integración con las actividades que los runbooks usan en módulos de integración con cmdlets de Windows PowerShell. 

[Service Management Automation](/previous-versions/system-center/system-center-2012-R2/dn469260(v=sc.12)) (SMA) almacena y ejecuta runbooks en el centro de datos local como Orchestrator y usa los mismos módulos de integración que Azure Automation. El Convertidor de runbooks convierte los runbooks de Orchestrator en runbooks gráficos, que no son compatibles con SMA. Aún así puede instalar el módulo Actividades estándar y los módulos de integración de System Center Orchestrator en SMA, pero tendrá que [reescribir los runbooks manualmente](/system-center/sma/authoring-automation-runbooks).

## <a name="download-the-orchestrator-migration-toolkit"></a>Descarga del kit de herramientas de migración de Orchestrator

El primer paso de la migración consiste en descargar el [kit de herramientas para migración de System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=47323). Este kit incluye herramientas que le ayudan a convertir runbooks de Orchestrator a Azure Automation.  

## <a name="import-the-standard-activities-module"></a>Importación del módulo Actividades estándar

Importe el [módulo Actividades estándar](/system-center/orchestrator/standard-activities?view=sc-orch-2019) a Azure Automation. Esto incluye versiones convertidas de las actividades estándar de Orchestrator que los runbooks gráficos convertidos pueden usar.

## <a name="import-orchestrator-integration-modules"></a>Importación de los módulos de integración de System Center Orchestrator

Microsoft proporciona [paquetes de integración](/previous-versions/system-center/packs/hh295851(v=technet.10)) para crear runbooks a fin de automatizar componentes de System Center y otros productos. Algunos de estos paquetes de integración se basan en OIT pero actualmente no se pueden convertir en módulos de integración debido a problemas conocidos. Importe los [Módulos de integración de System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=49555) a Azure Automation correspondientes a los paquetes de integración que usan los runbooks que acceden a System Center. Este paquete incluye las versiones convertidas de los paquetes de integración que se pueden importar a Azure Automation y a Service Management Automation.  

## <a name="convert-integration-packs"></a>Conversión de los paquetes de integración

Use el [Convertidor de paquetes de integración](/system-center/orchestrator/orch-integration-toolkit/integration-pack-wizard?view=sc-orch-2019) para convertir los paquetes de integración creados con [Orchestrator Integration Toolkit (OIT)](/previous-versions/system-center/developer/hh855853(v=msdn.10)) en módulos de integración basados en PowerShell que se pueden importar a Azure Automation o a Service Management Automation. Al ejecutar el Convertidor de paquetes de integración, aparece un asistente que le permite seleccionar un archivo de paquete de integración (.oip). Después, el asistente muestra las actividades incluidas en ese paquete de integración y le permite seleccionar las que quiere migrar. Al completar el asistente, se crea un módulo de integración que incluye un cmdlet correspondiente para cada una de las actividades del paquete de integración original.

> [!NOTE]
> No puede usar el Convertidor de paquetes de integración para convertir paquetes de integración que no se han creado con OIT. También hay algunos módulos de integración ofrecidos por Microsoft que actualmente no se pueden convertir con esta herramienta. Las versiones convertidas de estos paquetes de integración se ofrecen para descarga a fin de que se puedan instalar en Azure Automation o Service Management Automation.

### <a name="parameters"></a>Parámetros

Las propiedades de una actividad en el paquete de integración se convierten en parámetros del cmdlet correspondiente en el módulo de integración.  Los cmdlets de Windows PowerShell tienen un conjunto de [parámetros comunes](/powershell/module/microsoft.powershell.core/about/about_commonparameters) que se pueden usar con todos los cmdlets. Por ejemplo, el parámetro -Verbose hace que un cmdlet genere información detallada acerca de su operación.  Ningún cmdlet puede tener un parámetro con el mismo nombre que un parámetro común. Si una actividad sí tiene una propiedad con el mismo nombre que un parámetro común, el asistente le solicita que cambie el nombre del parámetro.

### <a name="monitor-activities"></a>Actividad de supervisión

Los runbooks de supervisión en Orchestrator comienzan con una [actividad de supervisión](/previous-versions/system-center/system-center-2012-R2/hh403827(v=sc.12)) y se ejecutan de manera constante, mientras esperan que un evento determinado los invoque. Azure Automation no admite los runbooks de supervisión, por lo que no se convierte ninguna actividad de supervisión del paquete de integración. En su lugar, se crea un cmdlet de marcador de posición en el módulo de integración para la actividad de supervisión.  Este cmdlet no tiene ninguna funcionalidad, sino que permite la instalación de cualquier runbook convertir que lo use. Este runbook no se puede ejecutar en Azure Automation, pero sí se puede instalar para que el usuario pueda modificarlo.

Orchestrator incluye un conjunto de [actividades estándar](/previous-versions/system-center/system-center-2012-R2/hh403832(v=sc.12)) que no están incluidas en un paquete de integración, pero que son usadas por muchos runbooks.  El módulo Actividades estándar es un módulo de integración que incluye un cmdlet equivalente para cada una de estas actividades. Debe instalar este módulo de integración en Azure Automation antes de importar cualquier runbook convertido que usa una actividad estándar.

Además de admitir los runbooks convertidos, es posible que un usuario familiarizado con Orchestrator pueda usar los cmdlets del módulo de actividades estándar para crear nuevos runbooks en Azure Automation. A pesar de que la funcionalidad de todas las actividades estándar se puede realizar con cmdlets, es posible que funcionen de manera diferente. Los cmdlets del módulo Actividades estándar convertidas funcionan de la misma manera que sus actividades correspondientes y usan los mismos parámetros. Esto puede ayudarle a realizar la transición a runbooks de Azure Automation.

## <a name="convert-orchestrator-runbooks"></a>Conversión de runbooks de Orchestrator

El Convertidor de runbooks de Orchestrator convierte los runbooks de Orchestrator en [runbooks gráficos](automation-runbook-types.md#graphical-runbooks) que se pueden importar a Azure Automation. El Convertidor de runbooks se implementa como módulo de PowerShell con el cmdlet `ConvertFrom-SCORunbook` que realiza la conversión. Al instalarlo, el convertidor crea un acceso directo a una sesión de PowerShell que carga el cmdlet.   

Estos son los pasos básicos para convertir un runbook e importarlo a Azure Automation. Más adelante en esta sección se ofrecen detalles sobre el uso del cmdlet.

1. Exporte uno o varios runbooks desde Orchestrator.
2. Obtenga los módulos de integración para todas las actividades del runbook.
3. Convierta los runbooks de Orchestrator en el archivo exportado.
4. Revise la información de los registros para validar la conversión y determinar las tareas manuales necesarias.
5. Importe los runbooks convertidos a Azure Automation.
6. Cree todos los recursos necesarios en Azure Automation.
7. Edite el runbook en Azure Automation para modificar las actividades necesarias.

La sintaxis de `ConvertFrom-SCORunbook` es:

```powershell
ConvertFrom-SCORunbook -RunbookPath <string> -Module <string[]> -OutputFolder <string>
```

* RunbookPath: ruta de acceso al archivo de exportación que contiene los runbooks que se convertirá.
* Module: lista delimitada por comas de los módulos de integración que contienen las actividades de los runbooks.
* OutputFolder: ruta de acceso a la carpeta para crear runbooks gráficos convertidos.

El siguiente comando de ejemplo convierte los runbooks en un archivo de exportación llamado **MyRunbooks.ois_export**.  Estos runbooks usan los paquetes de integración de Active Directory y Data Protection Manager.

```powershell
ConvertFrom-SCORunbook -RunbookPath "c:\runbooks\MyRunbooks.ois_export" -Module c:\ip\SystemCenter_IntegrationModule_ActiveDirectory.zip,c:\ip\SystemCenter_IntegrationModule_DPM.zip -OutputFolder "c:\runbooks"
```

### <a name="use-runbook-converter-log-files"></a>Uso de archivos de registro del Convertidor de runbooks

El Convertidor de runbooks crea los siguientes archivos de registro en la misma ubicación que el runbook convertido.  Si los archivos ya existen, se sobrescriben con la información de la última conversión.

| Archivo | Contenido |
|:--- |:--- |
| Runbook Converter - Progress.log |Pasos detallados de la conversión, incluidas información de cada actividad convertida correctamente y advertencias para cada actividad no convertida. |
| Runbook Converter - Summary.log |Resumen de la última conversión, incluidas todas las advertencias y realizar tareas de seguimiento que debe realizar, como la creación de una variable necesaria para el runbook convertido. |

### <a name="export-runbooks-from-orchestrator"></a>Exportación de runbooks desde Orchestrator

El Convertidor de runbooks funciona con un archivo de exportación de Orchestrator que contiene uno o más runbooks.  Crea un runbook de Azure Automation correspondiente para cada runbook de Orchestrator en el archivo de exportación.  

Para exportar un runbook de Orchestrator, haga clic con el botón derecho en el nombre del runbook en Runbook Designer y seleccione **Exportar**.  Para exportar todos los runbooks a una carpeta, haga clic con el botón derecho en el nombre de la carpeta y seleccione **Exportar**.

### <a name="convert-runbook-activities"></a>Conversión de actividades de runbook

El Convertidor de runbooks convierte cada actividad del runbook de Orchestrator a una actividad correspondiente en Azure Automation.  Para las actividades que no se puede convertir, se crea una actividad de marcador de posición en el runbook con el texto de la advertencia.  Después de importar el runbook convertido a Azure Automation, debe reemplazar cualquiera de estas actividades por actividades válidas que realizan la funcionalidad requerida.

Las actividades de Orchestrator en el módulo Actividades estándar se convierten. Sin embargo, hay algunas actividades de Orchestrator estándar que no están en este módulo y no se convierten. Por ejemplo, `Send Platform Event` no tiene ningún equivalente de Azure Automation, ya que es un evento específico de Orchestrator.

[Actividad de supervisión](/previous-versions/system-center/system-center-2012-R2/hh403827(v=sc.12)) no se convierten porque no hay ningún equivalente para ellas en Azure Automation. La excepción son las actividades de supervisión de paquetes de integración convertidos, que se convierten en la actividad de marcador de posición.

Todas las actividades de un paquete de integración convertido se convertirán si proporciona la ruta de acceso al módulo de integración con el parámetro `modules`. En los paquetes de integración de System Center, puede usar los Módulos de integración de System Center Orchestrator.

### <a name="manage-orchestrator-resources"></a>Administración de recursos de Orchestrator

El Convertidor de runbooks solo convierte runbooks, no otros recursos de Orchestrator como contadores, variables o conexiones.  Los contadores no se admiten en Azure Automation.  Las variables y conexiones se admiten, pero es necesario crearlas manualmente. Los archivos de registro le indican si el runbook requiere estos recursos y especifican los recursos correspondientes que se deben crear en Azure Automation para que el runbook convertido funcione correctamente.

Por ejemplo, un runbook puede usar una variable para rellenar un determinado valor en una actividad.  El runbook convertido convierte la actividad y especifica un recurso de variable en Azure Automation con el mismo nombre que la variable de Orchestrator. Esta acción se anota en el archivo **Runbook Converter - Summary.log** que se crea después de la conversión. Tiene que crear manualmente este recurso de variable en Azure Automation para poder usar el runbook.

### <a name="work-with-orchestrator-input-parameters"></a>Trabajo con parámetros de entrada de Orchestrator

Los runbooks de Orchestrator aceptan parámetros de entrada con la actividad `Initialize Data`.  Si el runbook que se convierte incluye esta actividad, se crea un [parámetro de entrada](automation-graphical-authoring-intro.md#handle-runbook-input) en el runbook de Azure Automation para cada parámetro de la actividad.  Se crea una actividad [Control de Script de flujo de trabajo](automation-graphical-authoring-intro.md#use-activities) en el runbook convertido que recupera y devuelve cada parámetro. Las actividades del runbook que usan un parámetro de entrada hacen referencia a la salida de esta actividad.

La razón por la que se usa esta estrategia es para reflejar mejor la funcionalidad del runbook de Orchestrator.  Las actividades de los nuevos runbooks gráficos deben hacer referencia directamente a parámetros de entrada que usan un origen de datos de entrada de Runbook.

### <a name="invoke-runbook-activity"></a>Actividad Invocar Runbook

Los runbooks de Orchestrator inician otros runbooks con la actividad `Invoke Runbook`. Si el runbook que se convierte incluye esta actividad y la opción `Wait for completion` está establecida, se crea una actividad de runbook para ella en el runbook convertido.  Si la opción `Wait for completion` no está establecida, se crea una actividad de script de flujo de trabajo que usa [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.7.0) para iniciar el runbook. Después de importar el runbook convertido a Azure Automation, debe modificar esta actividad con la información especificada en la actividad.

## <a name="create-orchestrator-assets"></a>Creación de recursos de Orchestrator

El Convertidor de runbooks no convierte recursos de Orchestrator. Debe crear manualmente los recursos de Orchestrator necesarios en Azure Automation.

## <a name="configure-hybrid-runbook-worker"></a>Configuración de Hybrid Runbook Worker

Orchestrator almacena los runbooks en un servidor de base de datos y los ejecuta en servidores de runbooks, ambas cosas en el centro de datos local. Los runbooks de Azure Automation se almacenan en la nube de Azure y se pueden ejecutar en el centro de datos local con una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md). Configure un rol de trabajo que ejecute los runbooks convertidos desde Orchestrator, dado que están diseñados para ejecutarse en servidores locales y acceder a recursos locales.

## <a name="related-articles"></a>Artículos relacionados

* Para más información sobre Orchestrator, consulte [System Center 2012: Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh237242(v=sc.12)).
* Obtenga más información sobre la automatización de la administración de servicios en [Service Management Automation](/previous-versions/system-center/system-center-2012-R2/dn469260(v=sc.12)).
* Puede encontrar detalles de las actividades de Orchestrator en [Actividades estándar de Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh403832(v=sc.12)).
* Para obtener el kit de herramientas de migración de Orchestrator, consulte [Descarga del kit de herramientas para migración de System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=47323).
* Para obtener información general sobre Hybrid Runbook Worker de Azure Automation, consulte [Introducción a Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).
