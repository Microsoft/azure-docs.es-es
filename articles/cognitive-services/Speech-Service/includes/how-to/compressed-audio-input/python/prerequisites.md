---
author: amitkumarshukla
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: amishu
ms.openlocfilehash: a715d0ece88cf2caf1cb3d20d703f550353094c5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282756"
---
El control del audio comprimido se implementa mediante [`GStreamer`](https://gstreamer.freedesktop.org). Por motivos de licencia, los binarios de `GStreamer` no se compilan ni se vinculan con el SDK de Voz. Los desarrolladores deben instalar varias dependencias y complementos; vea [Instalación en Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c). Los archivos binarios de `GStreamer` deben estar en la ruta de acceso del sistema, con el fin de que el SDK de Voz pueda cargarlos durante el tiempo de ejecución. Si el SDK de Voz es capaz de encontrar `libgstreamer-1.0-0.dll` durante el tiempo de ejecución, significa que los archivos binarios se encuentran en la ruta de acceso del sistema.

