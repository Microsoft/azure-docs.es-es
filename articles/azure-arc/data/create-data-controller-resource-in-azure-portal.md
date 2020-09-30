---
title: Creación de un controlador de datos de Azure Arc en Azure Portal
description: Creación de un controlador de datos de Azure Arc en Azure Portal
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 297efa83fb1563e3a360f652a6ac1bc2b1679998
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932386"
---
# <a name="create-an-azure-arc-data-controller-in-the-azure-portal"></a>Creación de un controlador de datos de Azure Arc en Azure Portal

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="introduction"></a>Introducción

Puede usar Azure Portal para crear un controlador de datos de Azure Arc.

Muchas de las experiencias de creación de Azure Arc comienzan en Azure Portal, incluso aunque el recurso que se va a crear o administrar se encuentre fuera de la infraestructura de Azure. El patrón de la experiencia de usuario en estos casos, especialmente cuando no hay conectividad directa entre Azure y su entorno, consiste en usar Azure Portal para generar un script que se puede descargar y ejecutar en su entorno para establecer una conexión segura con Azure. Por ejemplo, los servidores habilitados para Azure Arc siguen este patrón para [crear servidores habilitados para Arc](../servers/onboard-portal.md).

Por ahora, dado que la versión preliminar solo admite el modo Conectado indirectamente de los servicios de datos habilitados para Azure Arc, puede usar Azure Portal para generar un cuaderno que puede descargar y ejecutar con Azure Data Studio en el clúster de Kubernetes. En el futuro, cuando esté disponible el modo Conectado directamente, podrá aprovisionar el controlador de datos directamente desde Azure Portal. Más información acerca de los [Modos de conectividad](connectivity.md).

## <a name="use-the-azure-portal-to-create-an-azure-arc-data-controller"></a>Uso de Azure Portal para crear un controlador de datos de Azure Arc

Siga los pasos que se indican a continuación para crear un controlador de datos de Azure Arc con Azure Portal y Azure Data Studio.

1. En primer lugar, inicie sesión en [Marketplace de Azure Portal](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/MarketplaceOffersBlade/selectedMenuItemId/home/searchQuery/azure%20arc%20data%20controller).  Los resultados de la búsqueda de Marketplace se filtrarán para mostrar el elemento "Azure Arc data controller" (Controlador de datos de Azure Arc).
2. Si el primer paso no ha especificado los criterios de búsqueda. Escriba en los resultados de la búsqueda y haga clic en "Azure Arc data controller" (Controlador de datos de Azure Arc).
3. Seleccione el icono del controlador de datos de Azure en Marketplace.
4. Haga clic en el botón **Crear**.
5. Revise los requisitos para crear un controlador de datos de Azure Arc e instale el software necesario que pueda faltar, como Azure Data Studio y kubectl.
6. Haga clic en el botón **Data controller details** (Detalles del controlador de datos).
7. Elija una suscripción, un grupo de recursos y una ubicación de Azure igual que haría con cualquier otro recurso que se crea en Azure Portal. En este caso, la ubicación de Azure que seleccione será donde se almacenarán los metadatos sobre el recurso.  El recurso se creará en la infraestructura que elija. No es necesario que esté en la infraestructura de Azure.
8. Escriba un nombre para el controlador de datos.
9. Actualmente, en la versión preliminar solo se admite el modo Conectado indirectamente.
10. Seleccione un perfil de configuración de implementación.
11. Haga clic en el botón **Open in Azure Data Studio** (Abrir en Azure Data Studio).
12. En la siguiente pantalla, verá un resumen de las selecciones y un cuaderno que se ha generado.  Puede hacer clic en el botón **Download provisioning notebook** (Descargar cuaderno de aprovisionamiento) para descargar el cuaderno.
13. Abra el cuaderno en Azure Data Studio y haga clic en el botón **Run All** (Ejecutar todo) situado en la parte superior.
14. Siga las indicaciones y las instrucciones del cuaderno para completar la creación del controlador de datos.

## <a name="monitoring-the-creation-status"></a>Supervisión del estado de creación

La creación del controlador tardará unos minutos en completarse. Puede supervisar el progreso en otra ventana de terminal con los siguientes comandos:

> [!NOTE]
>  En los siguientes comandos de ejemplo se da por hecho que ha creado un controlador de datos y un espacio de nombres de Kubernetes con el nombre "arc".  Si ha usado otro un nombre de controlador de datos o de espacio de nombres, puede reemplazar "arc" por ese nombre.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

También puede comprobar el estado de creación de un pod determinado ejecutando un comando como el siguiente.  Esto es especialmente útil para solucionar problemas.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Solución de problemas de creación

Si tiene problemas con la creación, consulte la [guía de solución de problemas](troubleshoot-guide.md).
