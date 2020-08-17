---
title: Prueba de rendimiento y carga con Azure Application Insights | Microsoft Docs
description: Configuración de pruebas de rendimiento y carga con Azure Application Insights
ms.topic: conceptual
ms.date: 06/19/2019
ms.reviewer: sdash
ms.openlocfilehash: 8885a2e511c11317f593706255437e52819adcae
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024357"
---
# <a name="performance-testing"></a>Pruebas de rendimiento

> [!NOTE]
> El servicio de prueba de carga basado en la nube ha quedado en desuso. Puede encontrar más información sobre el desuso, la disponibilidad del servicio y servicios alternativos [aquí](/azure/devops/test/load-test/overview?view=azure-devops).

Application Insights le permite generar pruebas de carga para sus sitios web. Al igual que las [pruebas de disponibilidad](monitor-web-app-availability.md), puede enviar solicitudes básicas o [solicitudes de varios pasos](availability-multistep.md) desde agentes de prueba de Azure de todo el mundo. Las pruebas de rendimiento le permiten simular hasta 20 000 usuarios simultáneos durante 60 minutos.

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Para crear una prueba de rendimiento, primero deberá crear un recurso de Application Insights. Si ya ha creado un recurso, continúe con la siguiente sección.

En Azure Portal, seleccione **Crear un recurso** > **Herramientas para desarrolladores** > **Application Insights** y cree un recurso de Application Insights.

## <a name="configure-performance-testing"></a>Configuración de pruebas de rendimiento

Si es la primera vez que crea una prueba de rendimiento, seleccione **Establecer la organización** y elija una organización de Azure DevOps como el origen para las pruebas de rendimiento.

En **Configurar**, vaya a **Pruebas de rendimiento** y haga clic en **Nuevo** para crear una prueba.

![Fill at least the URL of your website](./media/performance-testing/new-performance-test.png)

Para crear una prueba de rendimiento básica, seleccione un tipo de prueba de **Prueba manual** y rellene las opciones que desee para la prueba.

|Configuración| Valor máximo
|----------|------------|
| Carga de usuarios | 20.000 |
| Duración (minutos)  | 60 |  

Después de crea la prueba, haga clic en **Ejecutar prueba**.

Una vez completada la prueba, verá los resultados con un aspecto similares al siguiente:

![Resultados de pruebas](./media/performance-testing/test-results.png)

## <a name="configure-visual-studio-web-test"></a>Configuración de la prueba web de Visual Studio

Las capacidades de pruebas de rendimiento avanzadas de Application Insights se basan en los proyectos de prueba de rendimiento y carga de Visual Studio.

![Visual Studio ](./media/performance-testing/visual-studio-test.png)

## <a name="next-steps"></a>Pasos siguientes

* [Pruebas web de varios pasos](availability-multistep.md)
* [Pruebas de ping de dirección URL](monitor-web-app-availability.md)
