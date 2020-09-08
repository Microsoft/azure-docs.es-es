---
title: Modelos de consumo del vehículo para enrutamiento | Microsoft Azure Maps
description: 'Obtenga información acerca de los modelos de consumo que Azure Maps admite: combustión y electricidad. Vea qué parámetros usa cada modelo y sus restricciones.'
author: subbarayudukamma
ms.author: skamma
ms.date: 05/08/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: b44186d783a249192a8c13ee97063034ee319df7
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036766"
---
# <a name="consumption-model"></a>Modelo de consumo

El servicio de enrutamiento proporciona un conjunto de parámetros para tener una descripción detallada del modelo de consumo específico del vehículo.
Según el valor de **vehicleEngineType**, se admiten dos modelos principales de consumo: _Combustion_ y _Electric_. Es incorrecto especificar parámetros que pertenecen a diferentes modelos en la misma solicitud. Además, los parámetros del modelo de consumo no se pueden usar con los siguientes valores de **travelMode**: _bicycle_ y _pedestrian_.

## <a name="parameter-constraints-for-consumption-model"></a>Restricciones de parámetros en el modelo de consumo

En ambos modelos de consumo, existen algunas dependencias cuando se especifican los parámetros. Esto significa que, si se especifican explícitamente algunos parámetros, puede que sea necesario especificar otros. Estas son las dependencias que se deben tener en cuenta:

* Todos los parámetros requieren que el usuario especifique **constantSpeedConsumption**. Es un error especificar cualquier otro parámetro del modelo de consumo si no se ha especificado **constantSpeedConsumption**. El parámetro **vehicleWeight** es una excepción de este requisito.
* **accelerationEfficiency** y **decelerationEfficiency** se deben especificar siempre como un par (es decir, ambos o ninguno).
* Si se especifican **accelerationEfficiency** y **decelerationEfficiency**, el producto de los valores no debe ser mayor que 1 (para evitar el movimiento perpetuo).
* **uphillEfficiency** y **downhillEfficiency** se deben especificar siempre como un par (es decir, ambos o ninguno).
* Si se especifican **uphillEfficiency** y **downhillEfficiency**, el producto de los valores no debe ser mayor que 1 (para evitar el movimiento perpetuo).
* Si el usuario especifica los parámetros de \*__Efficiency__, también debe especificarse **vehicleWeight**. Cuando **vehicleEngineType** es _combustion_, se debe especificar también **fuelEnergyDensityInMJoulesPerLiter**.
* **maxChargeInkWh** y **currentChargeInkWh** se deben especificar siempre como un par (es decir, ambos o ninguno).

> [!NOTE]
> Si solo se especifica **constantSpeedConsumption**, no se tiene en cuenta ningún otro aspecto de consumo, como pendientes o aceleración del vehículo, en los cálculos de consumo.

## <a name="combustion-consumption-model"></a>Modelo de consumo de combustión

El modelo de consumo de combustión se usa cuando **vehicleEngineType** se establece en _combustion_.
A continuación se muestra la lista de parámetros que pertenecen a este modelo. Consulte la sección Parámetros para ver una descripción más detallada.

* constantSpeedConsumptionInLitersPerHundredkm
* vehicleWeight
* currentFuelInLiters
* auxiliaryPowerInLitersPerHour
* fuelEnergyDensityInMJoulesPerLiter
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="electric-consumption-model"></a>Modelo de consumo eléctrico

El modelo de consumo eléctrico se usa cuando **vehicleEngineType** se establece en _electric_.
A continuación se muestra la lista de parámetros que pertenecen a este modelo. Consulte la sección Parámetros para ver una descripción más detallada.

* constantSpeedConsumptionInkWhPerHundredkm
* vehicleWeight
* currentChargeInkWh
* maxChargeInkWh
* auxiliaryPowerInkW
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="sensible-values-of-consumption-parameters"></a>Valores razonables de parámetros de consumo

Se puede rechazar un determinado conjunto de parámetros de consumo, incluso aunque puedan satisfacer todos los requisitos explícitos. Esto sucede cuando el valor de un parámetro concreto, o una combinación de valores de varios parámetros, se considera que conduce a magnitudes irrazonables de valores de consumo. En este caso, probablemente sea indicio de un error en la entrada, dado que se tiene el debido cuidado para admitir todos los valores razonables de los parámetros de consumo. Si un determinado conjunto de parámetros de consumo se rechace, el mensaje de error acompañante contendrá una explicación textual de los motivos.
En las descripciones detalladas de los parámetros se incluyen de ejemplos de valores razonables para ambos modelos.
