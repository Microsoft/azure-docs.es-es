---
title: Uso de las características de accesibilidad del diseñador (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre los métodos abreviados de teclado y las características de accesibilidad del lector de pantalla disponibles en el diseñador.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: peterlu
author: peterclu
ms.date: 01/09/2020
ms.custom: designer
ms.openlocfilehash: 5d91dc212eb2f856f0252261f2ddfc8a5a4d70ee
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88210366"
---
# <a name="use-a-keyboard-to-use-azure-machine-learning-designer-preview"></a>Uso del diseñador de Azure Machine Learning mediante el teclado (versión preliminar)

Aprenda a usar el diseñador de Azure Machine Learning con un teclado y un lector de pantalla. Para obtener una lista de métodos abreviados de teclado que funcionen en cualquier lugar de Azure Portal, consulte [Métodos abreviados de teclado de Azure Portal](../azure-portal/azure-portal-keyboard-shortcuts.md)

Este flujo de trabajo se ha probado con [Narrator](https://support.microsoft.com/help/22798/windows-10-complete-guide-to-narrator) y [JAWS](https://www.freedomscientific.com/products/software/jaws/), pero debería funcionar con otros lectores de pantalla estándar.

## <a name="navigate-the-pipeline-graph"></a>Desplazamiento por el gráfico de canalización

El gráfico de canalización se organiza como lista anidada. La lista externa es una lista de módulos, que describe todos los módulos del gráfico de canalización. La lista interna es una lista de conexiones, que describe todas las conexiones de un módulo concreto.  

1. En la lista de módulos, use la tecla de dirección para cambiar los módulos.
1. Use la pestaña para abrir la lista de conexiones del módulo de destino.
1. Use la tecla de dirección para cambiar entre los distintos puertos de conexión del módulo.
1. Use “G” para ir al módulo de destino.

## <a name="edit-the-pipeline-graph"></a>Edición del gráfico de canalización

### <a name="add-a-module-to-the-graph"></a>Adición de un módulo al gráfico

1. Use Ctrl+F6 para cambiar el foco del lienzo al árbol de módulos.
1. Busque el módulo deseado en el árbol de módulos mediante el control TreeView estándar.

### <a name="edit-a-module"></a>Edición de un módulo

Para conectar un módulo a otro:

1. Use Ctrl + Mayús + H cuando el destino sea un módulo de la lista de módulos para abrir el asistente de conexión.
1. Edite los puertos de conexión del módulo.

Para ajustar las propiedades del módulo:

1. Use Ctrl + Mayús + E cuando el destino sea un módulo para abrir las propiedades del módulo.
1. Edite las propiedades del módulo.

## <a name="navigation-shortcuts"></a>Métodos abreviados de navegación

| Pulsación de tecla | Descripción |
|-|-|
| Ctrl + F6 | Alterna el foco entre el lienzo y el árbol de módulos |
| Ctrl + F1   | Abre la tarjeta de información cuando se centra en un nodo del árbol de módulos |
| Ctrl + Mayús + H | Abre el asistente de conexión cuando el foco está en un nodo |
| Ctrl + Mayús + E | Abre las propiedades del módulo cuando el foco está en un nodo |
| Ctrl + G | Mueve el foco al primer nodo con error si se produjo algún error al ejecutar la canalización |

## <a name="action-shortcuts"></a>Accesos directos de acciones

Use los siguientes accesos directo con la clave de acceso. Para más información sobre las claves de acceso, consulte https://en.wikipedia.org/wiki/Access_key.

| Pulsación de tecla | Acción |
|-|-|
| Clave de acceso + R | Ejecute |
| Clave de acceso + P | Publicar |
| Clave de acceso + C | Clonar |
| Clave de acceso + D | Implementación |
| Clave de acceso + I | Crear o actualizar canalización de inferencia |
| Clave de acceso + B | Crear o actualizar canalización de inferencias por lotes |
| Clave de acceso + K | Abrir la lista desplegable "Create inference pipeline" (Crear canalización de inferencia) |
| Clave de acceso + U | Abrir la lista desplegable "Update inference pipeline" (Actualizar canalización de inferencia) |
| Clave de acceso + M | Abrir la lista desplegable más (...) |

## <a name="next-steps"></a>Pasos siguientes

- [Activación del contraste alto o cambio de tema](../azure-portal/set-preferences.md#choose-a-theme-or-enable-high-contrast)
- [Herramientas relacionadas con la accesibilidad en Microsoft](https://www.microsoft.com/accessibility)
