---
title: Retirada de las API de implementación para métricas de Azure Monitor y escalado automático
description: Las API de métricas y escalabilidad automática, también llamadas Azure Service Management (ASM) o modelo de implementación RDFE se van a retirar
ms.subservice: ''
ms.topic: conceptual
ms.date: 11/19/2018
ms.openlocfilehash: 38a30f2adc0de7ccb9a9a3a4ba7ed53fd5fda3f7
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317389"
---
# <a name="azure-monitor-retirement-of-classic-deployment-model-apis-for-metrics-and-autoscale"></a>Retirada de Azure Monitor de las API del modelo de implementación clásica para métricas y escalabilidad automática

Azure Monitor (anteriormente Azure Insights cuando se publicó por primera vez) ofrece actualmente la funcionalidad para crear y administrar configuraciones de escalabilidad automática y métricas de uso de las máquinas virtuales clásicas y de las instancias de Cloud Services clásicas. El conjunto original de API basadas en modelos de implementación clásica se **retirará después del 30 de junio de 2019** en todas las nubes públicas y privadas de Azure en todas las regiones.   

Se han admitido las mismas operaciones a través de un conjunto de API basadas en Azure Resource Manager durante más de un año. Azure Portal usa las nuevas API REST para la escalabilidad automática y las métricas. También hay disponible un nuevo SDK, PowerShell y CLI basados en estas API de Resource Manager. Nuestros servicios de asociados para la supervisión usan el nuevo Resource Manager que se basa en las API REST de Azure Monitor.  

## <a name="who-is-not-affected"></a>A quiénes no afecta

Si va a administrar la escalabilidad automática mediante Azure Portal, el [nuevo SDK de Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/), PowerShell, la CLI o mediante plantillas de Resource Manager, no se requiere ninguna acción.  

Si va a usar métricas a través de Azure Portal o a través de diversos [servicios de supervisión de asociados](./partners.md), no se requiere ninguna acción. Microsoft está trabajando con asociados de supervisión para migrar a las nuevas API.

## <a name="who-is-affected"></a>A quién afecta

Este artículo es aplicable a usted si usa los siguientes componentes:

- **SDK de Azure Insights clásico**: si usa el [SDK de Azure Insights clásico](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), empiece a utilizar el nuevo SDK de Azure Monitor para [.NET](https://github.com/azure/azure-libraries-for-net#download) o [Java](https://github.com/azure/azure-libraries-for-java#download). Descargue el [paquete NuGet del SDK de Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/).

- **Escalabilidad automática clásica**: si llama a [las API de configuración de escalabilidad automática clásicas](/previous-versions/azure/reference/mt348562(v=azure.100)) desde las herramientas de compilación personalizada o mediante el [SDK de Azure Insights clásico](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), empiece a usar la [API REST de Resource Manager para Azure Monitor](/rest/api/monitor/autoscalesettings).

- **Métricas clásicas**: si usa métricas mediante las [API REST clásicas](/previous-versions/azure/reference/dn510374(v=azure.100)) o el [SDK de Azure Insights clásico](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/) desde herramientas de compilación personalizada, empiece a usar la [API REST de Resource Manager para Azure Monitor](/rest/api/monitor/autoscalesettings). 

Si no está seguro de si el código o las herramientas personalizadas están llamando a las API clásicas, observe lo siguiente:

- Revise el identificador URI al que se hace referencia en el código o la herramienta. Las API clásicas usan el URI https://management.core.windows.net. Debería utilizar el URI más reciente para las API basadas en Resource Manager que empiezan por `https://management.azure.com/`.

- Compare el nombre del ensamblado en la máquina. El ensamblado clásico anterior se encuentra en https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/.

- Si usa la autenticación de certificados para acceder a las API de métricas o escalabilidad automática, está usando un punto de conexión y una biblioteca clásicos. Las API de Resource Manager más recientes requieren la autenticación de Azure Active Directory a través de una entidad de servicio o una entidad de seguridad de usuario.

- Si utiliza las llamadas a las que se hace referencia en la documentación en cualquiera de los siguientes vínculos, está usando las API clásicas antiguas.

  - [Biblioteca de clases Windows.Azure.Management.Monitoring](/previous-versions/azure/dn510414(v=azure.100))

  - [Supervisión (clásica) de .NET](/previous-versions/azure/reference/mt348562(v%3dazure.100))

  - [Interfaz IMetricOperations](/previous-versions/azure/reference/dn802395(v%3dazure.100))

## <a name="why-you-should-switch"></a>¿Por qué debe cambiar?

Todas las funcionalidades existentes para la escalabilidad automática y las métricas seguirán funcionando a través de las nuevas API.  

La migración a las API más recientes incluye funcionalidades basadas en Resource Manager como, por ejemplo, el control de acceso basado en rol (RBAC) coherente en todos los servicios de supervisión. También obtendrá una funcionalidad adicional para las métricas: 

- compatibilidad con las dimensiones
- granularidad de las métricas de 1 minuto coherente en todos los servicios 
- mejor capacidad de consulta
- retención de datos mayor (93 días de métricas en lugar de 30 días) 

En general, al igual que en todos los demás servicios de Azure, las API de Azure Monitor basadas en Resource Manager incluyen un mejor rendimiento, escalabilidad y confiabilidad. 

## <a name="what-happens-if-you-do-not-migrate"></a>¿Qué sucede si no realiza la migración?

### <a name="before-retirement"></a>Antes de la retirada

No habrá ningún efecto directo en los servicios de Azure ni en sus cargas de trabajo.  

### <a name="after-retirement"></a>Después de la retirada

Todas las llamadas a las API clásicas que se enumeraron anteriormente devolverán mensajes de error parecidos a estos:

Para el escalado automático: *Esta API está en desuso. Use Azure Portal, el SDK de Azure Monitor, PowerShell, la CLI o las plantillas de Resource Manager para administrar la configuración de la escalabilidad automática*.  

Para las métricas: *Esta API está en desuso. Use Azure Portal, el SDK de Azure Monitor, PowerShell o la CLI para consultar las métricas*.

## <a name="email-notifications"></a>Notificaciones por correo electrónico

Se envió una notificación de retirada a las direcciones de correo electrónico de los siguientes roles de cuenta: 

- Administradores de cuentas y servicios
- Coadministradores  

Si tiene alguna duda, póngase en contacto con nosotros en MonitorClassicAPIhelp@microsoft.com.  

## <a name="references"></a>Referencias

- [Nuevas API REST de Azure Monitor](/rest/api/monitor/) 
- [Nuevo SDK de Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/)

