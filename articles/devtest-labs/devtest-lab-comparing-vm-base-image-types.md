---
title: Comparación de imágenes personalizadas y fórmulas en DevTest Labs | Microsoft Docs
description: Obtenga más información sobre las diferencias entre las imágenes personalizadas y las fórmulas como bases de máquina virtual para que pueda decidir cuál se adapta mejor a su entorno.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: df7a8e6209f8033eb5a29c65079e9c2f4cbbe544
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287526"
---
# <a name="comparing-custom-images-and-formulas-in-devtest-labs"></a>Comparación de imágenes personalizadas y fórmulas en DevTest Labs
Se pueden usar [imágenes personalizadas](devtest-lab-create-template.md) y [fórmulas](devtest-lab-manage-formulas.md) como base para las [nuevas máquinas virtuales creadas](devtest-lab-add-vm.md). No obstante, la diferencia clave entre las fórmulas y las imágenes personalizadas estriba en que estas últimas son simplemente una imagen basada en un disco duro virtual, mientras que una fórmula es una imagen basada en un disco duro virtual *además de* opciones preconfiguradas, como el tamaño de la máquina virtual, la red virtual, la subred y los artefactos. Estas opciones preconfiguradas se configuran con valores predeterminados que se pueden reemplazar en el momento de creación de máquinas virtuales. En este artículo se explican algunas de las ventajas y desventajas del uso de imágenes personalizadas en comparación con el uso de fórmulas.

## <a name="custom-image-pros-and-cons"></a>Ventajas y desventajas de las imágenes personalizadas
Las imágenes personalizadas ofrecen una manera estática e inmutable de crear máquinas virtuales a partir de un entorno deseado. 

**Ventajas**

* El aprovisionamiento de máquinas virtuales a partir de imágenes personalizadas es rápido, ya que nada cambia después de poner en marcha la máquina virtual desde la imagen. En otras palabras, no hay ninguna configuración que aplicar, pues la imagen personalizada es tan solo una imagen sin configuración. 
* Las máquinas virtuales creadas a partir de una sola imagen personalizada son idénticas.

**Desventajas**

* Si necesita actualizar algún aspecto de la imagen personalizada, hay que volver a crear la imagen.  

## <a name="formula-pros-and-cons"></a>Ventajas y desventajas de las fórmulas
Las fórmulas ofrecen una manera dinámica de crear máquinas virtuales a partir de la configuración deseada.

**Ventajas**

* Los cambios de entorno se pueden capturar sobre la marcha mediante artefactos. Por ejemplo, si quiere que una máquina virtual se instale con los bits más recientes de la canalización de entrega de versiones o desea dar de alta el código más reciente del repositorio, basta con especificar un artefacto que implemente los bits más recientes o dé de alta el código más reciente en la fórmula junto con una imagen base de destino. Siempre que se use esta fórmula para crear máquinas virtuales, se implementa o se da de alta el código o los bits más recientes en la máquina virtual. 
* Las fórmulas pueden definir la configuración predeterminada que las imágenes personalizadas no pueden proporcionar; por ejemplo, tamaños de máquina virtual y configuración de red virtual. 
* La configuración guardada en una fórmula se muestra como valores predeterminados, pero se puede modificar al crear la máquina virtual. 

**Desventajas**

* La creación de una máquina virtual a partir de una fórmula puede tardar más que la creación de una máquina virtual a partir de una imagen personalizada.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>Entradas blogs relacionadas
* [Custom images or formulas? (¿Imágenes personalizadas o fórmulas?)](./devtest-lab-faq.md#blog-post)

## <a name="next-steps"></a>Pasos siguientes
- [Preguntas frecuentes sobre DevTest Labs](devtest-lab-faq.md)
