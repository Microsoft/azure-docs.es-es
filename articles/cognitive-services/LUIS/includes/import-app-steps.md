---
title: Pasos para importar una aplicación
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 06/22/2020
ms.author: diberry
ms.openlocfilehash: 37f1b85b4ce8510d5e288df985a55dba659f0c9b
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86035861"
---
1. En el [portal de LUIS](https://www.luis.ai), en la página **Mis aplicaciones**, seleccione **+ New app for conversation** (Nueva aplicación para conversación) y, a continuación, **Importar como JSON**. Busque el archivo JSON guardado en el paso anterior. No es necesario cambiar el nombre de la aplicación. Seleccione **Listo**

1. En la sección **Administrar**, en la pestaña **Versiones**, seleccione la versión `0.1`, luego seleccione **Clonar** para clonar la versión y asígnele el nuevo nombre de `ml-entity`. A continuación, seleccione **Listo** para finalizar el proceso de clonación. Dado que el nombre de la versión se usa como parte de la ruta de la dirección URL, el nombre no puede contener ningún carácter que no sea válido en una dirección URL.

    > [!TIP]
    > La clonación en una nueva versión es un procedimiento recomendado antes de modificar la aplicación. Cuando finalice un cambio en una versión, exporte la versión (como archivo .json o .lu), y compruebe el archivo en el sistema de control de código fuente.

1. Seleccione **Compilar** y después **Intenciones** para ver las intenciones, los principales bloques de creación de una aplicación de LUIS.

    ![Cambie de la página Versiones a la página Intenciones.](../media/tutorial-machine-learned-entity/new-version-imported-app.png)
