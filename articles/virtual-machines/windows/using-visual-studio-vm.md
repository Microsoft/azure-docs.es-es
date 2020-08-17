---
title: Uso de Visual Studio en una máquina virtual de Azure
description: Uso de Visual Studio en una máquina virtual de Azure.
author: cathysull
manager: cathys
ms.service: virtual-machines-windows
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 04/23/2020
ms.author: cathys
keywords: visualstudio
ms.openlocfilehash: 5ec8e0f6448e725998628e41b917942966d3ef2a
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836163"
---
# <a name="visual-studio-images-on-azure"></a>Imágenes de Visual Studio en Azure
El uso de Visual Studio en una máquina virtual (VM) de Azure preconfigurada es la manera más fácil y rápida de tener un entorno de desarrollo que funcione correctamente desde el principio. En [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?filters=virtual-machine-images%3Bmicrosoft%3Bwindows&page=1&subcategories=application-infrastructure) encontrará varias imágenes del sistema con distintas configuraciones de Visual Studio.

¿Acaba de llegar a Azure? [Creación de una cuenta de Azure gratis](https://azure.microsoft.com/free).

> [!NOTE]
> No todas las suscripciones son aptas para implementar imágenes de Windows 10. Para obtener más información, consulte [Uso del cliente Windows en Azure para escenarios de desarrollo y pruebas](./client-images.md).

## <a name="what-configurations-and-versions-are-available"></a>¿Qué configuraciones y versiones están disponibles?
En Azure Marketplace se pueden encontrar imágenes de las versiones principales más recientes: Visual Studio 2019, Visual Studio 2017 y Visual Studio 2015.  Igualmente, para cada versión principal publicada podrá ver la "versión de lanzamiento original en la Web" (RTW) y las últimas versiones actualizadas.  Cada una de estas versiones ofrece las ediciones Visual Studio Enterprise y Visual Studio Community.  Estas imágenes se actualizan al menos cada mes para incluir las actualizaciones más recientes de Visual Studio y Windows.  Aunque los nombres de las imágenes siguen siendo los mismos, la descripción de cada imagen incluye la versión del producto instalada y la fecha inicial de la imagen.

| Versión de lanzamiento                                                                                                                                                | Ediciones              | Versión del producto   |
|:--------------------------------------------------------------------------------------------------------------------------------------------------------------:|:---------------------:|:-----------------:|
| [Visual Studio 2019: versión más reciente (16.5)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftvisualstudio.visualstudio2019latest?tab=Overview) | Enterprise, Community | Versión 16.5.4    |
| Visual Studio 2019: RTW                         | Enterprise | Versión 16.0.13    |
| Visual Studio 2017: Más reciente (versión 15.9)           | Enterprise, Community | Versión 15.9.22   |
| Visual Studio 2017: RTW                             | Enterprise, Community | Versión 15.0.28  |
| Visual Studio 2015: Versión más reciente (Actualización 3)               | Enterprise, Community | Versión 14.0.25431.01 |

> [!NOTE]
> De acuerdo con la directiva de mantenimiento de Microsoft, ha expirado el mantenimiento de la versión de lanzamiento original (RTW) de Visual Studio 2015. Visual Studio 2015 Update 3 es la única versión que queda que se ofrece en la línea de productos de Visual Studio 2015.

Para obtener más información, consulte la [Directiva de mantenimiento de Visual Studio](https://www.visualstudio.com/productinfo/vs-servicing-vs).

## <a name="what-features-are-installed"></a>¿Qué características están instaladas?
Cada imagen contiene el conjunto de características recomendado para esa edición de Visual Studio. Por lo general, la instalación incluye:

* Todas las cargas de trabajo disponibles, incluidos los componentes opcionales recomendados de cada carga de trabajo
* SDK de .NET 4.6.2 y .NET 4.7, paquetes de destino y herramientas de desarrollo
* Visual F#
* Extensión de GitHub para Visual Studio
* Herramientas de LINQ to SQL

La línea de comandos usada para instalar Visual Studio al compilar las imágenes es la siguiente:

```
    vs_enterprise.exe --allWorkloads --includeRecommended --passive ^
       add Microsoft.Net.Component.4.7.SDK ^
       add Microsoft.Net.Component.4.7.TargetingPack ^ 
       add Microsoft.Net.Component.4.6.2.SDK ^
       add Microsoft.Net.Component.4.6.2.TargetingPack ^
       add Microsoft.Net.ComponentGroup.4.7.DeveloperTools ^
       add Microsoft.VisualStudio.Component.FSharp ^
       add Component.GitHub.VisualStudio ^
       add Microsoft.VisualStudio.Component.LinqToSql
```

Si las imágenes no incluyen la característica de Visual Studio que necesita, proporcione comentarios mediante la herramienta para crear comentarios que se encuentra en la esquina superior derecha de la página.

## <a name="what-size-vm-should-i-choose"></a>¿Qué tamaño de VM debería elegir?
Azure ofrece una amplia gama de tamaños de máquina virtual. Dado que Visual Studio es una eficaz aplicación multiproceso, es recomendable que la máquina virtual tenga un tamaño que pueda incluir al menos dos procesadores y 7 GB de memoria. Se recomiendan los siguientes tamaños de máquina virtual para las imágenes de Visual Studio:

   * Standard_D2_v3
   * Standard_D2s_v3
   * Standard_D4_v3
   * Standard_D4s_v3
   * Standard_D2_v2
   * Standard_D2S_v2
   * Standard_D3_v2
    
Para obtener más información acerca de los tamaños más recientes de máquinas virtuales, consulte [Tamaños de las máquinas virtuales Windows en Azure](../sizes.md).

Con Azure, puede volver a equilibrar su elección inicial mediante la modificación del tamaño de la máquina virtual. Puede aprovisionar una nueva máquina virtual con un tamaño más adecuado o cambiar el tamaño de la máquina virtual existente para otro hardware subyacente. Para más información, consulte [Cambio de tamaño de una máquina virtual Windows](./resize-vm.md).

## <a name="after-the-vm-is-running-whats-next"></a>Una vez que la máquina virtual ya está en ejecución, ¿qué es lo siguiente?
Visual Studio sigue el modelo "traiga su propia licencia" en Azure. Igual que sucede con la instalación en hardware propietario, uno de los primeros pasos es obtener una licencia para la instalación de Visual Studio. Para desbloquear Visual Studio, tiene dos opciones:
- Iniciar sesión con una cuenta de Microsoft que esté asociada con una suscripción de Visual Studio 
- Desbloquear Visual Studio con la clave de producto suministrada con la compra inicial

Para más información, consulte [Iniciar sesión en Visual Studio](/visualstudio/ide/signing-in-to-visual-studio) y [Cómo desbloquear Visual Studio](/visualstudio/ide/how-to-unlock-visual-studio).

## <a name="how-do-i-save-the-development-vm-for-future-or-team-use"></a>¿Cómo puedo guardar la máquina virtual de desarrollo para usarla en un futuro o trabajar en equipo?

Hay una amplia gama de entornos de desarrollo y, si quiere compilar uno de los entornos más complejos, esto le supondrá un costo significativo. Con independencia de la configuración del entorno, puede guardar o capturar la máquina virtual configurada como una "imagen base" para usarla en un futuro o con otros miembros de su equipo. Luego, al arrancar una nueva máquina virtual, puede aprovisionarla a partir de la imagen base en lugar de la imagen de Azure Marketplace.

Resumen rápido: use la herramienta de preparación del sistema (Sysprep) y apague la máquina virtual en ejecución. A continuación, capture *(Figura 1)* la máquina virtual como una imagen mediante la interfaz de usuario de Azure Portal. Azure guarda el archivo `.vhd` que contiene la imagen en la cuenta de almacenamiento de su elección. La nueva imagen se muestra entonces como un recurso de imagen en la lista de recursos de la suscripción.

<img src="media/using-visual-studio-vm/capture-vm.png" alt="Capture an image through the Azure portal UI" style="border:3px solid Silver; display: block; margin: auto;"><center> *(Figura 1) Captura de una imagen mediante la interfaz de usuario de Azure Portal.* </center>

Para más información, consulte [Captura de una imagen administrada de una máquina virtual generalizada en Azure](./capture-image-resource.md).

> [!IMPORTANT]
> No olvide usar Sysprep para preparar la máquina virtual. Si se salta este paso, Azure no podrá aprovisionar la VM desde la imagen.

> [!NOTE]
> Almacenar las imágenes le supondrá cierto costo, pero ese costo incremental puede ser insignificante en comparación con los costos generales de recompilar la máquina virtual desde cero para cada miembro del equipo que necesite una. Por ejemplo, puede crear y almacenar una imagen de 127 GB durante un mes para que la use el equipo entero y solo le costará una pequeña cantidad de dinero. Sin embargo, este costo es insignificante si lo comparamos con las horas que debe invertir cada empleado en compilar y validar un cuadro de desarrollo que esté configurado correctamente y que se pueda usar de forma individual.

Además, las tareas o tecnologías dedicadas al desarrollo necesitan más escalado, como las variedades referentes a la configuración de desarrollo y a la configuración de varias máquinas. Puede usar Azure DevTest Labs para crear _recetas_ que automaticen la construcción de la "imagen maestra". También puede usar DevTest Labs para administrar directivas de las máquinas virtuales en ejecución de su equipo. Si quiere obtener más información acerca de DevTest Labs, consulte [Uso de Azure DevTest Labs para desarrolladores](../../devtest-labs/devtest-lab-developer-lab.md).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ya conoce las imágenes preconfiguradas de Visual Studio, el siguiente paso es crear una nueva máquina virtual:

* [Cree una máquina virtual Windows en Azure Portal](quick-create-portal.md)
* [Información general sobre las máquinas virtuales Windows](overview.md)