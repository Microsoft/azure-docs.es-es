---
title: 'Plantillas de libros de Azure Monitor: traslado de regiones'
description: Traslado de una plantilla de libro a una región diferente
services: azure-monitor
author: gardnerjr
manager: acearun
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/12/2020
ms.author: jgardner
ms.openlocfilehash: f37550d914cf7efb0c75ed3dfa8854e1ec7be7e0
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875468"
---
# <a name="move-an-azure-workbook-template-to-another-region"></a>Traslado de una plantilla de libro de Azure a otra región

En este artículo, se explica cómo pueden trasladarse los recursos de plantillas de libros de Azure a otra región. Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para implementar características o servicios que solo están disponibles en regiones concretas, para cumplir los requisitos de gobernanza y las directivas internas o en respuesta a los requisitos de planeamiento de capacidad.

Actualmente no hay ninguna interfaz de usuario en el portal para crear recursos de plantillas de libro; la única manera de crearlos es [a través de implementaciones de plantilla de Azure Resource Manager (plantilla de Resource Manager)](./workbooks-automate.md). Por eso, la manera más fácil de trasladar una plantilla es reutilizar la plantilla de ARM anterior y actualizarla para que se implemente en la nueva región.

## <a name="prerequisites"></a>Requisitos previos

* Asegúrese de que las plantillas de libro se admiten en la región de destino.

## <a name="prepare"></a>Preparación

* Identifique la plantilla de ARM usada anteriormente para la plantilla de libro.

## <a name="move"></a>Mover

1. Actualice la plantilla usada previamente para que haga referencia a la nueva región.

   > [!NOTE]
   > Es posible que tenga que usar un nuevo nombre para la plantilla de libro con el fin de que no haya nombres duplicados.

2. Implemente la plantilla actualizada a través de la implementación de plantilla de Resource Manager para crear una nueva plantilla de libro en la región deseada.

## <a name="verify"></a>Comprobar

Use la interfaz de usuario de exploración de libros de Azure para buscar la plantilla de libro recién implementada. Asegúrese de que la ubicación es la ubicación de destino.

## <a name="clean-up"></a>Limpieza

Una vez creada la plantilla del libro en la nueva región, elimine la plantilla del libro original de la región anterior.
1. Busque la plantilla de libro en la interfaz de usuario de exploración de libros de Azure.
2. Seleccione la plantilla de libro que desea eliminar.
3. Seleccione el comando "Eliminar".

Si cambió el nombre de la plantilla de libro para importarla en una nueva región, puede cambiar el nombre de la plantilla de libro al nombre anterior después de eliminar el elemento original mediante el comando "Cambiar nombre" en la vista de recursos de la plantilla de libro de Azure.

## <a name="next-steps"></a>Pasos siguientes

¿Necesita trasladar un libro en lugar de una plantilla? Consulte cómo [trasladar un libro de Azure a otra región](./workbooks-move-region.md).

