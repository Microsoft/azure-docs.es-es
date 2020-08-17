---
title: 'Instrucciones: Configuración de parámetros como entidades externas'
titleSuffix: Azure Cognitive Services
description: En este artículo, se explica cómo configurar un parámetro de cadena para hacer referencia al catálogo expuesto a través de un punto de conexión web.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/09/2019
ms.author: sausin
ms.openlocfilehash: 45dba3b7f46ec558c46b8505da26fd3ef4de4cbc
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284190"
---
# <a name="add-configurations-to-commands-parameters"></a>Adición de configuraciones a parámetros de comandos

En este artículo, obtendrá más información sobre la configuración avanzada de parámetros, incluido lo siguiente:

 - Cómo los valores de parámetro pueden pertenecer a un conjunto definido de forma externa a la aplicación de Comandos personalizados.
 - Cómo agregar cláusulas de validación según el valor de los parámetros.

## <a name="prerequisites"></a>Requisitos previos

Debe haber completado los pasos descritos en los siguientes artículos:

> [!div class="checklist"]
> * [Cómo: Creación de una aplicación con comandos simples](./how-to-custom-commands-create-application-with-simple-commands.md)
> * [Cómo: Adición de parámetros a comandos](./how-to-custom-commands-add-parameters-to-commands.md)


## <a name="configure-parameter-as-external-catalog-entity"></a>Configuración de parámetros como entidades de catálogo externas

En esta sección, configurará parámetros de tipo cadena para hacer referencia a los catálogos externos hospedados a través de un punto de conexión web. Esto le permite actualizar el catálogo externo de forma independiente sin realizar modificaciones en la aplicación de Comandos personalizados. Este enfoque resulta útil en los casos en los que puede haber un gran número de entradas de catálogo.

Vuelva a usar el parámetro **SubjectDevice** del comando **TurnOnOff**. La configuración actual de este parámetro es **Accept predefined inputs from internal catalog** (aceptar valores de entrada predefinidos del catálogo interno). Se refiere a una lista estática de dispositivos, tal y como se definió en la configuración de parámetros. Queremos trasladar este contenido a un origen de datos externo que se puede actualizar de forma independiente.

Para ello, empiece por agregar un nuevo punto de conexión web. Vaya a la sección **Puntos de conexión web** del panel izquierdo y agregue un nuevo punto de conexión web con la siguiente configuración.

| Configuración | Valor sugerido |
|----|----|
| Nombre | `getDevices` |
| URL | `https://aka.ms/speech/cc-sampledevices` |
| Método | GET |


Si el valor sugerido para la dirección URL no funciona, debe configurar y hospedar un punto de conexión web simple que devuelva un JSON con una lista de los dispositivos que se pueden controlar. El punto de conexión web debe devolver un JSON con el formato siguiente:
    
```json
{
    "fan" : [],
    "refrigerator" : [
        "fridge"
    ],
    "lights" : [
        "bulb",
        "bulbs",
        "light"
        "light bulb"
    ],
    "tv" : [
        "telly",
        "television"
        ]
}

```


Después, vaya a la página de configuración del parámetro **SubjectDevice** y cambie las propiedades a lo siguiente.

| Configuración | Valor sugerido |
| ----| ---- |
| Configuración | Aceptar entradas predefinidas del catálogo externo |                               
| Punto de conexión del catálogo | getDevices |
| Método | GET |

Después, seleccione **Guardar**.

> [!IMPORTANT]
> No verá la opción para configurar un parámetro que acepte entradas de un catálogo externo, a menos que el punto de conexión web esté configurado en la sección **Punto de conexión web** del panel izquierdo.

### <a name="try-it-out"></a>Prueba

Seleccione **Entrenar** y espere a que se complete el entrenamiento. Una vez que se complete el entrenamiento, seleccione **Probar** y pruebe algunas interacciones.

* Entrada: apagar
* Salida: Which device do you want to control? (¿Qué dispositivo quiere controlar?)
* Entrada: las luces
* Salida: De acuerdo, apagaré las luces

> [!NOTE]
> Observe cómo ya puede controlar ahora todos los dispositivos hospedados en el punto de conexión web. Aún debe entrenar la aplicación para probar los cambios nuevos y volver a publicar la aplicación.

## <a name="add-validation-to-parameters"></a>Adición de validación a los parámetros

Las **validaciones** son construcciones que se aplican a determinados tipos de parámetro; estas permiten configurar restricciones para el valor del parámetro y solicitar correcciones si los valores no se encuentran dentro de las restricciones. Para obtener una lista completa de los tipos de parámetro que amplían la construcción de validación, diríjase a las [referencias](./custom-commands-references.md)

Pruebe las validaciones con el comando **SetTemperature**. Use los siguientes pasos para agregar una validación al parámetro **Temperature**.

1. Seleccione el comando **SetTemperature** en el panel izquierdo.
1. Seleccione **Temperature** en el panel central.
1. Seleccione **Add a validation** (agregar una validación), presente en el panel derecho.
1. En la ventana **Nueva validación**, configure la validación de la siguiente manera y seleccione **Crear**.


    | Configuración de parámetros | Valor sugerido | Descripción |
    | ---- | ---- | ---- |
    | Valor mínimo | `60` | En el caso de los parámetros numéricos, el valor mínimo que puede asumir este parámetro |
    | Valor máximo | `80` | En el caso de los parámetros numéricos, el valor máximo que puede asumir este parámetro |
    | Error de respuesta |  Editor sencillo > Primera variación > `Sorry, I can only set temperature between 60 and 80 degrees. What temperature do you want?` | Mensaje para solicitar un valor nuevo si se produce un error en la validación |

    > [!div class="mx-imgBorder"]
    > ![Agregue una validación de intervalo](media/custom-commands/add-validations-temperature.png).

### <a name="try-it-out"></a>Prueba

1. Seleccione el icono **Entrenar** presente en la parte superior del panel derecho.

1. Una vez que se complete el entrenamiento, seleccione **Probar** y pruebe algunas interacciones:

    - Entrada: establezca la temperatura en 72 grados.
    - Salida: Ok, setting temperature to 72 degrees (De acuerdo, definiendo la temperatura en 72 grados)
    - Entrada: establezca la temperatura en 45 grados.
    - Salida: Sorry, I can only set temperature between 60 and 80 degrees (el valor de temperatura solo puede establecerse entre 60 y 80 grados)
    - Entrada: cámbiala a 72 grados en su lugar.
    - Salida: Ok, setting temperature to 72 degrees (De acuerdo, definiendo la temperatura en 72 grados)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo: Adición de reglas de interacción](./how-to-custom-commands-add-interaction-rules.md)
