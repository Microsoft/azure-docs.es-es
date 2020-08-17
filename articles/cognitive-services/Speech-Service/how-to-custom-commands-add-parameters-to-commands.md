---
title: 'Instrucciones: Adición de comandos simples a una aplicación de Comandos personalizados: servicios de Voz'
titleSuffix: Azure Cognitive Services
description: En este artículo, obtendrá información sobre cómo agregar parámetros a Comandos personalizados
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: 0ed237debc2395ed307658b2d57a541574f9478a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284156"
---
# <a name="add-parameters-to-commands"></a>Adición de parámetros a comandos

En este artículo, obtendrá información sobre cómo agregar parámetros a Comandos personalizados. Los parámetros son información que los comandos requieren para completar una tarea. En los escenarios complejos, los parámetros también se pueden usar para definir las condiciones que desencadenan acciones personalizadas.

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * [Cómo: Creación de una aplicación con comandos simples](./how-to-custom-commands-create-application-with-simple-commands.md)

## <a name="configure-parameters-for-turnon-command"></a>Configuración de parámetros para el comando TurnOn

Edite el comando **TurnOn** existente para encender y apagar varios dispositivos.

1. Dado que el comando controlará ahora los escenarios de encendido y apagado, cambie el nombre del comando a **TurnOnOff**.
   1. En el panel izquierdo, seleccione el comando **TurnOn** y, después, el botón de puntos suspensivos (...) junto a **Nuevo comando** en la parte superior del panel.
   
   1. Seleccione **Rename** (Cambiar nombre). En las ventanas **Rename command** (cambiar nombre del comando), cambie el **Nombre** por **TurnOnOff**.

1. A continuación, agregue un nuevo parámetro a este comando para indicar si el usuario quiere encender o apagar el dispositivo.
   1. Seleccione la opción **Agregar**, presente en la parte superior del panel. En el menú desplegable, seleccione **Parámetro**.
   1. En el panel derecho, en la sección **Parámetros**, en el cuadro **Nombre**, agregue el valor **OnOff**.
   1. Seleccione **Requerido**. En la ventana **Add response for a required parameter** (Agregar respuesta para un parámetro requerido), seleccione **Simple editor** (Editor sencillo). En **First variation** (primera variación), agregue
        ```
        On or Off?
        ```
   1. Seleccione **Actualizar**.

       > [!div class="mx-imgBorder"]
       > ![Crear respuesta de parámetro requerida](media/custom-commands/add-required-on-off-parameter-response.png)
   
   1. Ahora, se configurarán las propiedades de los parámetros. Para obtener una explicación de todas las propiedades de configuración de un comando, vaya a [referencias](./custom-commands-references.md). Configure las propiedades del parámetro como se indica a continuación:
      

       | Configuración      | Valor sugerido     | Descripción                                                      |
       | ------------------ | ----------------| ---------------------------------------------------------------------|
       | Nombre               | `OnOff`           | Nombre descriptivo para el parámetro                                                                           |
       | Es global          | Desactivado       | Casilla que indica si el valor de este parámetro se aplica globalmente a todos los comandos de la aplicación.|
       | Obligatorio           | Activado         | Casilla que indica si es obligatorio especificar un valor para este parámetro antes de completar el comando. |
       | Respuesta para el parámetro requerido      |Editor sencillo > `On or Off?`      | Pregunta para solicitar el valor de este parámetro cuando no se conoce. |
       | Tipo               | String          | Tipo de parámetro; por ejemplo, Number, String, Date Time o Geography   |
       | Configuración      | Aceptar valores de entrada predefinidos del catálogo interno | En el caso de las cadenas, esto limita las entradas a un conjunto de valores posibles. |
       | Valores de entrada predefinidos     | `on`, `off`           | Conjunto de valores posibles y sus alias         |
       
        
   1. Para agregar valores de entrada predefinidos, seleccione **Agregar una entrada predefinida** y, en la ventana **Nuevo elemento**, escriba **Nombre** como se indica en la tabla anterior. En este caso, no usamos alias, por lo que puede dejarlo en blanco.
   
      > [!div class="mx-imgBorder"]
      > ![Crear parámetro](media/custom-commands/create-on-off-parameter.png)

   1. Seleccione **Guardar** para guardar todas las configuraciones del parámetro.
 
 ### <a name="add-subjectdevice-parameter"></a>Adición del parámetro SubjectDevice 

   1. A continuación, vuelva a seleccionar **Agregar** para agregar un segundo parámetro que represente el nombre de los dispositivos que se podrán controlar con este comando. Use la configuración siguiente.
   

       | Configuración            | Valor sugerido       |
       | ------------------ | --------------------- |
       | Nombre               | `SubjectDevice`         |
       | Es global          | Desactivado             |
       | Obligatorio           | Activado               |
       | Respuesta para el parámetro requerido     | Editor sencillo > `Which device do you want to control?`    | 
       | Tipo               | String                |          |
       | Configuración      | Aceptar valores de entrada predefinidos del catálogo interno | 
       | Valores de entrada predefinidos | `tv`, `fan`               |
       | Alias (`tv`)      | `television`, `telly`     |

   1. Seleccione **Guardar**.

### <a name="modify-example-sentences"></a>Modificación de oraciones de ejemplo

En el caso de los comandos con parámetros, resulta útil agregar oraciones de ejemplo que cubran todas las combinaciones posibles. Por ejemplo:

* Información del parámetro completa: `turn {OnOff} the {SubjectDevice}`
* Información del parámetro parcial: `turn it {OnOff}`.
* Información del parámetro faltante: `turn something`.

Las oraciones de ejemplo con diferentes niveles de información permiten que la aplicación de comandos personalizados resuelva las resoluciones de una sola captura y las resoluciones multigiro con información parcial.

Teniendo esto en cuenta, edite las oraciones de ejemplo para usar los parámetros como se sugiere a continuación:

```
turn {OnOff} the {SubjectDevice}
{SubjectDevice} {OnOff}
turn it {OnOff}
turn something {OnOff}
turn something
```

Seleccione **Guardar**.

> [!TIP]
> En el editor de oraciones de ejemplo, use llaves para hacer referencia a los parámetros. - `turn {OnOff} the {SubjectDevice}` Use la tecla TAB para la finalización automática respaldada por parámetros creados previamente.

### <a name="modify-completion-rules-to-include-parameters"></a>Modificación de reglas de finalización para incluir parámetros

Modifique la regla de finalización existente **ConfirmationResponse**.

1. En la sección **Conditions** (Condiciones), seleccione **Add a condition** (Agregar una condición).
1. En la ventana **New Condition** (Nueva condición), en la lista **Type** (Tipo), seleccione **Required parameters** (Parámetros requeridos). En la lista de comprobación siguiente, marque las opciones **OnOff** y **SubjectDevice**.
1. Deje **IsGlobal** sin activar.
1. Seleccione **Crear**.
1. En la sección **Actions** (Acciones), edite la acción de **envío de respuesta de voz** existente. Para ello, mantenga el puntero sobre la acción y seleccione el botón de edición. Esta vez, haga uso de los parámetros **OnOff** y **SubjectDevice** que se acaban de crear.

    ```
    Ok, turning the {SubjectDevice} {OnOff}
    ```
1. Seleccione **Guardar**.

### <a name="try-it-out"></a>Prueba
1. Seleccione el icono **Entrenar** que se encuentra en la parte superior del panel derecho.

1. Cuando se haya completado el entrenamiento, seleccione **Probar**. Aparecerá una nueva ventana **Test your application** (Prueba de la aplicación).
 Pruebe algunas interacciones.

    - Entrada: Turn off the tv (apagar el televisor)
    - Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)
    - Entrada: turn off the television (apaga el televisor)
    - Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)
    - Entrada: turn it off (apágalo)
    - Salida: Which device do you want to control? (¿Qué dispositivo quiere controlar?)
    - Entrada: the tv (el televisor)
    - Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)

## <a name="configure-parameters-for-settemperature-command"></a>Configuración de parámetros para el comando SetTemperature

Modifique el comando **SetTemperature** para que pueda establecer la temperatura según las indicaciones del usuario.

Agregue un nuevo parámetro **Temperature** con la siguiente configuración.

| Configuración      | Valor sugerido     |
| ------------------ | ----------------|
| Nombre               | `Temperature`           |
| Obligatorio           | Activado         |
| Respuesta para el parámetro requerido      | Editor sencillo > `What temperature would you like?`
| Tipo               | Number          |


Edite las expresiones de ejemplo con los siguientes valores.

```
set the temperature to {Temperature} degrees
change the temperature to {Temperature}
set the temperature
change the temperature
```

Edite las reglas de finalización existentes según la configuración siguiente.

| Configuración      | Valor sugerido     |
| ------------------ | ----------------|
| Condiciones         | Parámetro obligatorio > Temperature           |
| Acciones           | Enviar respuesta de voz > `Ok, setting temperature to {Temperature} degrees` |

### <a name="try-it-out"></a>Prueba

**Entrene** y **pruebe** los cambios con algunas interacciones.

- Entrada: Set temperature (establece la temperatura)
- Salida: What temperature would you like? (¿Qué temperatura le gustaría?)
- Entrada: 50 degrees (50 grados)
- Salida: Ok, setting temperature to 50 degrees (de acuerdo, definiré la temperatura en 50 grados)

## <a name="configure-parameters-to-the-setalarm-command"></a>Configuración de parámetros para el comando SetAlarm

Agregue un parámetro denominado **DateTime** con la siguiente configuración.

   | Configuración                           | Valor sugerido                     | 
   | --------------------------------- | ----------------------------------------|
   | Nombre                              | `DateTime`                               |
   | Obligatorio                          | Activado                                 |
   | Respuesta para el parámetro requerido   | Editor sencillo > `For what time?`            | 
   | Tipo                              | DateTime                                |
   | Valores predeterminados de fecha                     | Si falta la fecha, use hoy            |
   | Valores predeterminados de hora                     | Si falta la hora, use inicio del día     |


> [!NOTE]
> En este artículo, en su mayor parte usamos los tipos de parámetros de cadena, número y DateTime. Para obtener una lista de todos los tipos de parámetro admitidos y sus propiedades, vaya a las [referencias](./custom-commands-references.md).


Edite las expresiones de ejemplo con los siguientes valores.

```
set an alarm for {DateTime}
set alarm {DateTime}
alarm for {DateTime}
```

Edite las reglas de finalización existentes según la configuración siguiente.

   | Configuración    | Valor sugerido                               |
   | ---------- | ------------------------------------------------------- |
   | Acciones    | Enviar respuesta de voz: `Ok, alarm set for {DateTime}`  |


### <a name="try-it-out"></a>Prueba

**Entrene** y **pruebe** los cambios.
- Entrada: Establecer alarma para mañana a mediodía
- Salida: Ok, alarm set for 2020-05-02 12:00:00 (De acuerdo, alarma establecida para el 05/02/2020 a las 12:00:00)
- Entrada: Establecer una alarma
- Salida: What time? (¿A qué hora?)
- Entrada: 17:00
- Salida: Ok, alarm set for 2020-05-01 17:00:00 (De acuerdo, alarma establecida para el 05/01/2020 a las 17:00:00)


## <a name="try-out-all-the-commands"></a>Prueba de todos los comandos

Pruebe los tres comandos juntos mediante expresiones relacionados con distintos comandos. Tenga en cuenta que puede cambiar entre los distintos comandos.

- Entrada: Establecer una alarma
- Salida: For what time? (¿A qué hora?)
- Entrada: Turn on the tv (enciende el televisor)
- Salida: Ok, turning the tv on (de acuerdo, encenderé el televisor)
- Entrada: Establecer una alarma
- Salida: For what time? (¿A qué hora?)
- Entrada: 17:00
- Salida: Ok, alarm set for 2020-05-01 17:00:00 (De acuerdo, alarma establecida para el 05/01/2020 a las 17:00:00)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo: Adición de configuraciones a parámetros de comandos](./how-to-custom-commands-add-parameter-configuration.md)
