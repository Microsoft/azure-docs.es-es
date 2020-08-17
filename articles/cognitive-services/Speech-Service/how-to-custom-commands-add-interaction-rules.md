---
title: 'Procedimientos: Adición de una confirmación a un comando personalizado'
titleSuffix: Azure Cognitive Services
description: En este artículo, obtendrá información sobre cómo implementar confirmaciones para un comando en Comandos personalizados.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: 7d6c0928196c9e8e1abf6aa7f724a58753ce3d2a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289051"
---
# <a name="add-interaction-rules"></a>Adición de reglas de interacción

En este artículo, aprenderá acerca de las **reglas de interacción**. Estas son reglas adicionales para controlar situaciones más específicas o complejas. Aunque tiene la libertad de crear sus propias reglas de interacción personalizadas, en este artículo, podrá usarlas para los siguientes escenarios objetivo:

* Confirmación de comandos
* Adición de correcciones de un paso a los comandos

Para obtener más información sobre las reglas de interacción, vaya a la sección de [referencia](./custom-commands-references.md).

## <a name="prerequisites"></a>Requisitos previos

Debe haber completado los pasos descritos en los siguientes artículos:
> [!div class="checklist"]
> * [Cómo: Creación de una aplicación con comandos simples](./how-to-custom-commands-create-application-with-simple-commands.md)
> * [Cómo: Adición de parámetros a comandos](./how-to-custom-commands-add-parameters-to-commands.md)

## <a name="add-confirmations-to-a-command"></a>Adición de confirmaciones a un comando

Para agregar una confirmación, use el comando **SetTemperature**. Para conseguir la confirmación, use los pasos siguientes para crear reglas de interacción.

1. Seleccione el comando **SetTemperature** en el panel izquierdo.
1. Seleccione **Agregar** en el panel central para agregar reglas de interacción. A continuación, seleccione **Reglas de interacción** > **Confirm command** (Confirmar comando).

    Esta acción agrega tres reglas de interacción; que pedirán al usuario que confirme la fecha y la hora de la alarma y esperarán una confirmación (sí/no) para el siguiente turno.

    1. Modifique la regla de interacción **Confirm Command** (Confirmar comando) según la configuración siguiente:
        1. Cambie el **Nombre** a **Confirm temperature** (Confirmar temperatura).
        1. Agregue una nueva condición como **Parámetros obligatorios** > **Temperature** (Temperatura).
        1. Agregue una nueva acción como **Tipo** > **Enviar respuesta de voz** > **Are you sure you want to set the temperature as {Temperature} degrees?** (¿Está seguro de que quiere definir la temperatura en {Temperature} grados?)
        1. Deje el valor predeterminado de **Expecting confirmation from user** (A la espera de confirmación del usuario) en la sección **Expectativas**.
      
         > [!div class="mx-imgBorder"]
         > ![Crear respuesta de parámetro requerida](media/custom-speech-commands/add-validation-set-temperature.png)
    

    1. Modifique la regla de interacción **Confirmation succeeded** (confirmación correcta) para controlar una confirmación correcta (el usuario dijo sí).
      
          1. Cambie el **Nombre** a **Confirmation temperature succeeded**.
          1. Deje la condición **La confirmación se realizó correctamente** existente.
          1. Agregue una nueva condición como **Tipo** > **Parámetros obligatorios** > **Temperature**.
          1. Deje el valor predeterminado del **Post-execution state** (estado posterior a la ejecución) como **Execute completion rules** (ejecutar reglas de finalización).

    1. Modifique la regla de interacción **Confirmation denied** (Confirmación denegada) para controlar escenarios en que se deniega la confirmación (el usuario dijo que no).

          1. Cambie el **Nombre** a **Confirmation temperature denied** (Confirmación de temperatura denegada).
          1. Deje la condición **La confirmación se denegó** existente.
          1. Agregue una nueva condición como **Tipo** > **Parámetros obligatorios** > **Temperature** (Temperatura).
          1. Agregue una nueva acción como **Tipo** > **Enviar respuesta de voz** > **No problem. What temperature then? (No hay problema. Entonces, ¿qué temperatura?)** .
          1. Deje el valor predeterminado del **Post-execution state** (estado posterior a la ejecución) como **Wait for user's input** (esperar la entrada del usuario).

> [!IMPORTANT]
> En este artículo, se usó la funcionalidad de confirmación integrada. También puede agregar manualmente reglas de interacción de una en una.
   

### <a name="try-out-the-changes"></a>Prueba de los cambios

Seleccione **Entrenar**, espere a que se complete el entrenamiento y elija **Probar**.

- **Entrada**: Set temperature to 80 degrees (Establece la temperatura en 80 grados)
- **Salida**: ¿está seguro de que quiere establecer la temperatura en 80 grados?
- **Entrada**: No
- **Salida**: No se preocupe. What temperature then? (Entonces, ¿qué temperatura?)
- **Entrada**: 72 degrees (72 grados).
- **Salida**: ¿está seguro de que quiere establecer la temperatura en 72 grados?
- **Entrada**: Sí
- **Salida**: Ok, setting temperature to 83 degrees (De acuerdo, definiré la temperatura en 83 grados)


## <a name="implement-corrections-in-a-command"></a>Implementación de correcciones en un comando

En esta sección, configurará la corrección de un paso, que se usa después de ejecutar la acción de cumplimiento. También verá un ejemplo de cómo la corrección se habilita de forma predeterminada en caso de que el comando aún no se haya completado. Para agregar una corrección cuando no se haya completado el comando, agregue el nuevo parámetro **AlarmTone**.

Seleccione el comando **SetAlarm** en el panel izquierdo y agregue el nuevo parámetro **AlarmTone**.
        
- **Nombre** > **AlarmTone**
- **Tipo** > **Cadena**
- **Valor predeterminado** > **Campanillas**
- **Configuración** > **Aceptar valores de entrada predefinidos del catálogo interno**
- **Valores de entrada predefinidos** > **Campanillas**, **Tintineo** y **Eco** como entradas individuales predefinidas


A continuación, actualice la respuesta del parámetro **DateTime** como **Ready to set alarm with tone as {AlarmTone}. For what time? (Listo para definir la alarma con el tono {AlarmTone} ¿A qué hora?)** . A continuación, modifique la regla de finalización como se indica a continuación:

1. Seleccione la regla de finalización existente **ConfirmationResponse**.
1. En el panel derecho, mantenga el mouse sobre la acción existente y seleccione **Editar**.
1. Actualice la respuesta de voz a **OK, alarm set for {DateTime}. The alarm tone is {AlarmTone} (De acuerdo, alarma definida para {DateTime}. El tono de alarma es {AlarmTone}).**

### <a name="try-out-the-changes"></a>Prueba de los cambios

Seleccione **Entrenar**, espere a que se complete el entrenamiento y elija **Probar**.
Pruebe las siguientes expresiones:

- **Entrada**: Set an alarm (Define una alarma).
- **Salida**: Ready to set alarm with tone as Chimes. (Listo para establecer una alarma con el tono campanillas). For what time? (¿A qué hora?)
- **Entrada**: Set an alarm with the tone as Jingle for 9 am tomorrow (Establece una alarma con el tono Tintineo para mañana a las 9:00).
- **Salida**: OK, alarm set for 2020-05-30 09:00:00 (De acuerdo, alarma establecida para el 30/05/2020 a las 9:00:00) The alarm tone is Jingle. (El tono de alarma es Tintineo).

> [!IMPORTANT]
> Se puede cambiar el tono de alarma sin ninguna configuración explícita en un comando en curso; por ejemplo, cuando el comando no se ha completado todavía. *Hay una corrección habilitada de forma predeterminada para todos los parámetros del comando, independientemente del número de turno si el comando aún no se ha completado.*

### <a name="correction-when-command-is-completed"></a>Corrección cuando el comando se ha completado

La plataforma de Comandos personalizados también proporciona la funcionalidad de corrección en un paso, incluso cuando se ha completado el comando. Esta característica no está habilitada de forma predeterminada. Debe configurarse explícitamente. Siga estos pasos para configurar una corrección de un paso.

1. En el comando **SetAlarm**, agregue una regla de interacción del tipo **Update previous command** (Actualizar el comando anterior) para actualizar la alarma establecida previamente. Cambie el **nombre** predeterminado de la regla de interacción a **Update previous alarm** (actualizar la alarma anterior).
1. Deje la condición predeterminada **Previous command needs to be updated** (se debe actualizar el comando anterior) tal como está.
1. Agregue una nueva condición como **Tipo** > **Parámetros obligatorios** > **DateTime**.
1. Agregue una nueva acción como **Tipo** > **Enviar respuesta de voz** > **Editor simple** > **Updating previous alarm time to {DateTime} (Actualizando hora de alarma anterior a {DateTime}).**
1. Deje el valor predeterminado del **estado posterior a la ejecución** como **Comando finalizado**.

### <a name="try-out-the-changes"></a>Prueba de los cambios

Seleccione **Entrenar**, espere a que se complete el entrenamiento y elija **Probar**.

- **Entrada**: Set an alarm (Define una alarma).
- **Salida**: Ready to set alarm with tone as Chimes. (Listo para establecer una alarma con el tono campanillas). For what time? (¿A qué hora?)
- **Entrada**: Set an alarm with the tone as Jingle for 9 am tomorrow (Establece una alarma con el tono Tintineo para mañana a las 9:00).
- **Salida**: OK, alarm set for 2020-05-21 09:00:00 (De acuerdo, alarma establecida para el 21/05/2020 a las 9:00:00) The alarm tone is Jingle. (El tono de alarma es Tintineo).
- **Entrada**: No, 8 am (No, 08:00).
- **Salida**: Updating previous alarm time to 2020-05-29 08:00 (actualizaré la hora de la alarma anterior al 29/05/2020 a las 8:00:00).

> [!NOTE]
> En una aplicación real, en la sección **Acciones** de esta regla de corrección, también deberá enviar una actividad al cliente o llamar a un punto de conexión HTTP para actualizar la hora de la alarma en el sistema. Esta acción solo debe ser responsable de actualizar la hora de la alarma y ningún otro atributo del comando. En este caso, sería el tono de alarma.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo: Adición de plantillas de generación de idioma para respuestas de voz](./how-to-custom-commands-add-language-generation-templates.md)
