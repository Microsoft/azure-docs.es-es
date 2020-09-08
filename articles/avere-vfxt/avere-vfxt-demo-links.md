---
title: Proyectos de demostración de Avere vFXT para Azure
description: 'En estos ejemplos se muestran las características clave y los casos de uso de Avere vFXT for Azure: representación de vídeo, informática de alto rendimiento, rendimiento de vFXT y configuración de cliente.'
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 31e32bfc0a2c3279375148bdf3da7d4a4829af1c
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271080"
---
# <a name="avere-vfxt-demo-projects"></a>Proyectos de demostración de Avere vFXT

Los tutoriales de ejemplo se proporcionan en [GitHub](https://github.com/Azure/Avere). En estos pequeños proyectos se muestran las características y los casos de uso claves para Avere vFXT para Azure.

## <a name="video-rendering"></a>Representación de vídeo

* [Representación con Azure Batch y Avere vFXT](https://github.com/Azure/Avere/blob/master/docs/maya_azure_batch_avere_vfxt_demo.md): este es un proyecto de 60 minutos en el que se muestra cómo usar Autodesk Maya con Azure Batch y un clúster de Avere vFXT para generar una película animada.

* [¿Por qué usar Avere vFXT para la representación?](https://github.com/Azure/Avere/blob/master/docs/why_avere_for_rendering.md): esta es una demostración que compara los tiempos de representación de almacenamiento conectado a la red con y sin un clúster de Avere vFXT.

## <a name="high-performance-computing"></a>Informática de alto rendimiento

* [Procedimientos recomendados para mejorar el tiempo de inicio de la máquina virtual (VM) de Azure](https://github.com/Azure/Avere/blob/master/docs/azure_vm_provision_best_practices.md): esta es una prueba reproducible que usa Avere vFXT for Azure para mejorar los tiempos de inicio al generar miles de clientes informáticos rápidamente.

## <a name="vfxt-performance"></a>Rendimiento de vFXT

* [Medir el rendimiento de vFXT con vdbench](https://github.com/Azure/Avere/blob/master/docs/vdbench.md): esta es una configuración de prueba básica para generar cargas de trabajo pequeñas y medianas y así poder probar los subsistemas de disco y memoria de vFXT.

## <a name="client-setup"></a>Configuración del cliente

* [Estación de trabajo de Windows 10 para Avere vFXT](https://github.com/Azure/Avere/blob/master/docs/windows_10_avere_vfxt_mounted_workstation.md): muestra cómo configurar una estación de trabajo de Windows y montarla en un clúster de Avere vFXT.
