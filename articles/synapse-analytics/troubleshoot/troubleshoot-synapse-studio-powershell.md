---
title: Solución de problemas de conectividad de Synapse Studio (versión preliminar) con PowerShell
description: Solución de problemas de conectividad de Azure Synapse Studio con PowerShell
author: julieMSFT
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: f4afaf536a9c65758ad030e5cdeeee5fb97074d7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046666"
---
# <a name="diagnose-azure-synapse-studio-preview-connectivity-issues-with-powershell-script"></a>Diagnóstico de problemas de conectividad de Azure Synapse Studio (versión preliminar) con un script de PowerShell

Azure Synapse Studio (versión preliminar) depende de un conjunto de puntos de conexión de API web para funcionar correctamente. Esta guía le ayudará a identificar las causas de los problemas de conectividad al:
- configurar la red local (por ejemplo, la red detrás de un firewall corporativo) para acceder a Azure Synapse Studio;
- experimentar problemas de conectividad con Azure Synapse Studio.

## <a name="prerequisite"></a>Requisito previo

* PowerShell 5.0 o una versión posterior en Windows, o
* PowerShell 6.0 o una versión posterior en Windows o Linux.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

Haga clic con el botón derecho en el siguiente vínculo y haga clic en "Guardar destino como":

- [Test-AzureSynapse.ps1](https://go.microsoft.com/fwlink/?linkid=2119734)

Como alternativa, puede abrir el vínculo directamente y guardar el archivo de script abierto. No guarde la dirección del vínculo anterior, ya que puede cambiar en el futuro.

En el Explorador de archivos, haga clic con el botón derecho en el archivo de script descargado y haga clic en "Ejecutar con PowerShell".

![Ejecución del archivo de script descargado con PowerShell](media/troubleshooting-synapse-studio-powershell/run-with-powershell.png)

Cuando se le solicite, escriba el nombre del área de trabajo de Azure Synapse que está experimentando un problema o cuya conectividad desea probar y presione Entrar.

![Escribir el nombre del área de trabajo](media/troubleshooting-synapse-studio-powershell/enter-workspace-name.png)

Se iniciará la sesión de diagnóstico. Espere hasta que se complete la operación.

![Esperar a que se complete el diagnóstico](media/troubleshooting-synapse-studio-powershell/wait-for-diagnosis.png)

Al final, se mostrará un resumen del diagnóstico. Si el equipo no se puede conectar a uno o varios de los puntos de conexión, mostrará algunas sugerencias en la sección "Summary" (Resumen).

![Revisar el resumen de diagnóstico](media/troubleshooting-synapse-studio-powershell/diagnosis-summary.png)

Además, se generará un archivo de registro de diagnóstico para esta sesión en la misma carpeta que el script de solución de problemas. Su ubicación se muestra en la sección "General tips" (Recomendaciones generales) (`D:\TestAzureSynapse_2020....log`). Si es necesario, puede enviar este archivo al servicio de soporte técnico.

Si es administrador de red y está ajustando la configuración del firewall para Azure Synapse Studio, los detalles técnicos que se muestran sobre la sección "Summary" pueden ayudarle.

* Todos los elementos de prueba (solicitudes) marcados con "Passed" significan que han superado las pruebas de conectividad, independientemente del código de estado HTTP.
 En el caso de las solicitudes con error, el motivo se muestra en amarillo, como `NamedResolutionFailure` o `ConnectFailure`. Estas razones pueden ayudarle a averiguar si hay errores de configuración en el entorno de red.


## <a name="next-steps"></a>Pasos siguientes
Si los pasos anteriores no le ayudan a resolver el problema, [cree una incidencia de soporte técnico](../../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md).
