---
title: Introducción a Microsoft Azure StorSimple Data Manager | Microsoft Docs
description: Obtenga información sobre la solución StorSimple Data Manager y cómo puede usar este servicio para escribir aplicaciones que usan datos de StorSimple y otros servicios de Azure.
services: storsimple
documentationcenter: NA
author: vidarmsft
manager: syadav
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/21/2018
ms.author: alkohli
ms.openlocfilehash: d683f49cadb384ef59d3bae819156733691813cd
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183503"
---
# <a name="storsimple-data-manager-solution-overview"></a>Introducción a la solución StorSimple Data Manager

## <a name="overview"></a>Información general

Microsoft Azure StorSimple usa el almacenamiento en la nube como una extensión de la solución local y organiza automáticamente los datos en niveles entre el almacenamiento local y el almacenamiento en la nube. Los datos se almacenan en la nube en un formato comprimido y desduplicado para obtener la máxima eficacia y reducir los costos. Como los datos se almacenan en formato de StorSimple, no es fácilmente compatible con otras aplicaciones en la nube que pueda querer usar.

StorSimple Data Manager le permite acceder y utilizar sin problemas los datos en formato de StorSimple en la nube. Para ello, transforma el formato de StorSimple en archivos y blobs nativos que puede usar con otros servicios como Azure Media Services, Azure HDInsights y Azure Machine Learning.

Es artículo proporciona una introducción a la solución StorSimple Data Manager. También se explica cómo puede usar este servicio para escribir aplicaciones que usen datos de StorSimple y otros servicios de Azure en la nube.

## <a name="how-it-works"></a>¿Cómo funciona?

El servicio StorSimple Data Manager identifica los datos de StorSimple en la nube desde un dispositivo local StorSimple serie 8000. Los datos de StorSimple en la nube están desduplicados y comprimidos en formato de StorSimple. El servicio Data Manager proporciona las API para extraer los datos en formato de StorSimple y transformarlos a otros formatos, como Azure Blobs y Azure Files. Estos datos transformados, a continuación, se usan con facilidad en Azure HDInsight y Azure Media Services. La transformación de datos, por tanto, permite a estos servicios operar en los datos de StorSimple transformados desde el dispositivo local StorSimple serie 8000. Esto se ilustra en el diagrama siguiente.

![Diagrama de alto nivel](./media/storsimple-data-manager-overview/storsimple-data-manager-overview2.png)


## <a name="data-manager-use-cases"></a>Casos de uso de Data Manager

Puede utilizar Data Manager con Azure Functions, Azure Automation y Azure Data Factory para tener flujos de trabajo en ejecución en los datos tal como llega en StorSimple. Puede procesar el contenido multimedia que almacena en StorSimple con Azure Media Services o ejecutar un algoritmo de aprendizaje automático en los datos o utilizar un clúster de Hadoop para analizar los datos que se almacenan en StorSimple. Con la amplia gama de servicios disponibles en Azure combinada con los datos de StorSimple, puede desbloquear la eficacia de los datos.


## <a name="region-availability"></a>Disponibilidad en regiones

StorSimple Data Manager está disponible en las siguientes 7 regiones:

 - Sudeste de Asia
 - Este de EE. UU.
 - Oeste de EE. UU.
 - Oeste de EE. UU. 2
 - Centro-Oeste de EE. UU.
 - Norte de Europa
 - Oeste de Europa

Sin embargo, StorSimple Data Manager se puede usar para transformar datos en las siguientes regiones. 

![Regiones disponibles para datos](./media/storsimple-data-manager-overview/data-manager-job-definition-different-regions-m.png)

Este conjunto es mayor porque la implementación de recursos en cualquiera de las regiones anteriores es capaz de abrir el proceso de transformación en las regiones siguientes. Por lo tanto, siempre y cuando los datos residan en cualquiera de las 19 regiones, puede transformar los datos mediante este servicio.


## <a name="choosing-a-region"></a>Elección de una región

Es recomendable que:
 - La cuenta de almacenamiento de origen (la asociada con el dispositivo de StorSimple) y la cuenta de almacenamiento de destino (donde quiere los datos en formato nativo) estén en la misma región de Azure.
 - Incorpore Data Manager y la definición del trabajo en la región que contiene la cuenta de almacenamiento de StorSimple. Si esto no es posible, incorpore Data Manager en la región de Azure más próxima y, a continuación, cree la definición del trabajo en la misma región que la cuenta de almacenamiento de StorSimple. 

    Si la cuenta de almacenamiento de StorSimple no está en una de las 26 regiones que admiten la creación de la definición de trabajo, se recomienda que no ejecute StorSimple Data Manager, puesto que tendrá largas latencias y cargos de salida potenciales.
    
Microsoft se esfuerza por garantizar que los servicios de Azure siempre estén disponibles en todas las regiones. Sin embargo, pueden producirse interrupciones de servicio no planeadas durante breves períodos en una región determinada. En tales casos, puede abrir una definición de trabajos y Data Manager en una región que no se esté afectada por la interrupción y ejecutar el trabajo de transformación. Podría encontrar alguna latencia adicional en este escenario, pero podría ser la estrategia de recuperación en el caso excepcional de una interrupción regional.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

StorSimple Data Manager necesita la clave de cifrado de datos del servicio para transformar desde el formato de StorSimple al formato nativo. La clave de cifrado de datos del servicio se genera solo cuando se registra el primer dispositivo en el servicio StorSimple. Consulte [Seguridad de StorSimple](storsimple-8000-security.md) para más información sobre esta clave.

La clave de cifrado de datos del servicio proporcionada como entrada se almacena en un almacén de claves que se crea cuando se crea una instancia de Data Manager. El almacén se encuentra en la misma región de Azure que StorSimple Data Manager. Esta clave se elimina al eliminar el servicio Data Manager.

Los recursos de proceso utilizan esta clave para realizar la transformación. Estos recursos de proceso se encuentran en la misma región de Azure que la definición del trabajo. Esta región puede o no ser la misma que la región donde se incorpora Data Manager.

Si la región de Data Manager es diferente de la región de la definición del trabajo, es importante que sepa qué datos y metadatos residen en cada una de estas regiones. El siguiente diagrama muestra el efecto de tener diferentes regiones para Data Manager y para la definición del trabajo.

![Servicio y definición del trabajo en distintas regiones](./media/storsimple-data-manager-overview/data-manager-job-different-regions.png)

## <a name="managing-personal-information"></a>Administración de información personal

StorSimple Data Manager no recopila ni muestra información personal. Para obtener más información, revise la directiva de privacidad de Microsoft en el [Centro de confianza](https://www.microsoft.com/trustcenter).

## <a name="known-limitations"></a>Limitaciones conocidas

Actualmente el servicio tiene las siguientes limitaciones:
- StorSimple Data Manager actualmente no funciona con los volúmenes que están cifrados con BitLocker. Si intenta ejecutar el servicio con una unidad cifrada, verá errores en el trabajo.
- No se conservarán algunos metadatos de archivos (como las ACL) en los datos transformados.
- Este servicio solo funciona con volúmenes NTFS.
- Las longitudes de ruta de acceso de archivo deben contener menos de 256 caracteres, de lo contrario se producirá un error en el trabajo.

## <a name="next-steps"></a>Pasos siguientes

[Manage using the StorSimple Data Manager service UI](storsimple-data-manager-ui.md) (Administración del uso de la interfaz de usuario del servicio StorSimple Data Manager).
