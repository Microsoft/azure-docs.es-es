---
title: El tamaño de la carpeta TEMP predeterminada es demasiado pequeño para un rol | Microsoft Docs
description: Un rol de servicio de nube tiene una cantidad limitada de espacio para la carpeta TEMP. En este artículo se ofrecen algunas sugerencias sobre cómo evitar quedarse sin espacio.
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 9f2af8dd-2012-4b36-9dd5-19bf6a67e47d
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 77c4f627668b91c4bc21ad73b3511c1775943229
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89460208"
---
# <a name="default-temp-folder-size-is-too-small-on-a-cloud-service-webworker-role"></a>El tamaño predeterminado de la carpeta TEMP es demasiado pequeño en un rol de trabajo o web de servicio en la nube.
El directorio temporal predeterminado de un rol web o de trabajo de servicio de nube tiene un tamaño máximo de 100 MB, que puede llenarse en algún momento. En este artículo se describe cómo evitar quedarse sin espacio para el directorio temporal.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-do-i-run-out-of-space"></a>¿Por qué me quedo sin espacio?
Las variables del entorno de Windows estándar TEMP y TMP están disponibles en el código que se ejecuta en su aplicación. Tanto TEMP como TMP apuntan a un único directorio que tiene un tamaño máximo de 100 MB. Los datos que están almacenados en este directorio no persisten en el ciclo de vida del servicio en la nube; si las instancias de rol de un servicio en la nube se reciclan, el directorio se limpia.

## <a name="suggestion-to-fix-the-problem"></a>Sugerencia para corregir el problema
Implemente una de las siguientes alternativas:

* Configure un recurso de almacenamiento local y acceda a él directamente, en lugar de usar TEMP o TMP. Para acceder a un recurso de almacenamiento local desde el código que se ejecuta en la aplicación, llame al método [RoleEnvironment.GetLocalResource](/previous-versions/azure/reference/ee772845(v=azure.100)) .
* Configure un recurso de almacenamiento local y apunte a los directorios TEMP y TMP para que apunten a la ruta de acceso del recurso de almacenamiento local. Esta modificación debe realizarse dentro del método [RoleEntryPoint.OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)) .

En el ejemplo de código siguiente se muestra cómo modificar los directorios de destino para TEMP y TMP desde dentro del método OnStart:

```csharp
using System;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override bool OnStart()
        {
            // The local resource declaration must have been added to the
            // service definition file for the role named WorkerRole1:
            //
            // <LocalResources>
            //    <LocalStorage name="CustomTempLocalStore"
            //                  cleanOnRoleRecycle="false"
            //                  sizeInMB="1024" />
            // </LocalResources>

            string customTempLocalResourcePath =
            RoleEnvironment.GetLocalResource("CustomTempLocalStore").RootPath;
            Environment.SetEnvironmentVariable("TMP", customTempLocalResourcePath);
            Environment.SetEnvironmentVariable("TEMP", customTempLocalResourcePath);

            // The rest of your startup code goes here…

            return base.OnStart();
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Lea el blog que describe [cómo aumentar el tamaño de la carpeta temporal de ASP.NET del rol web de Azure](https://docs.microsoft.com/archive/blogs/kwill/how-to-increase-the-size-of-the-windows-azure-web-role-asp-net-temporary-folder).

Vea más [artículos de solución de problemas](https://docs.microsoft.com/visualstudio/azure/vs-azure-tools-debugging-cloud-services-overview) para servicios en la nube.

Para más información acerca de cómo solucionar los problemas de los roles de los servicios en la nube mediante el uso de datos de diagnóstico de equipos de PaaS de Azure, consulte la [serie de blogs de Kevin Williamson](https://docs.microsoft.com/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).
