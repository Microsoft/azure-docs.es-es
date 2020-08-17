---
title: 'Procedimiento: Creación de una aplicación con comandos simples: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: En este artículo, obtendrá información sobre cómo crear y probar una aplicación de Comandos personalizados hospedada con comandos simples.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: d166257dd28773d89a4f1fd56de3cb1a22242523
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284173"
---
# <a name="create-application-with-simple-commands"></a>Creación de una aplicación con comandos simples

En este artículo aprenderá a:
 - Crear una aplicación vacía
 - Actualizar recursos de LUIS
 - Agregar algunos comandos básicos a la aplicación de Comandos personalizados

## <a name="create-empty-application"></a>Creación de una aplicación vacía
Cree una aplicación de Comandos personalizados vacía. Para obtener más información, consulte la [guía de inicio rápido](quickstart-custom-commands-application.md). Esta vez, en lugar de importar un proyecto, se crea un proyecto en blanco.

1. En el cuadro **Nombre**, escriba el nombre del proyecto `Smart-Room-Lite` (u otro de su elección).
1. En la lista **Idioma**, seleccione **Inglés (Estados Unidos)** .
1. Seleccione o cree el recurso de LUIS que prefiera.

   > [!div class="mx-imgBorder"]
   > ![Creación de un proyecto](media/custom-commands/create-new-project.png)

## <a name="update-luis-resources-optional"></a>Actualización de recursos de LUIS (opcional)

Puede actualizar el recurso de creación que seleccionó en la ventana **Nuevo proyecto**, así como establecer un recurso de predicción. El recurso de predicción se usa para el reconocimiento cuando se publica la aplicación de Comandos personalizados. No necesita un recurso de predicción durante las fases de desarrollo y pruebas.

## <a name="add-turnon-command"></a>Adición del comando TurnOn

En la aplicación Comandos personalizados **Smart-Room-Lite** que acaba de crear, agregue un comando simple que procese una expresión, `turn on the tv`, y responda con el mensaje `Ok, turning the tv on`.

1. Cree un nuevo comando; para ello, seleccione **Nuevo comando** en la parte superior del panel izquierdo. Se abre la ventana **Nuevo comando**.
1. Especifique el valor del campo **Nombre** en **TurnOn**.
1. Seleccione **Crear**.

El panel central muestra las distintas propiedades del comando. Le corresponde configurar las siguientes propiedades del comando. Para obtener una explicación de todas las propiedades de configuración de un comando, vaya a [referencias](./custom-commands-references.md).

| Configuración            | Descripción                                                                                                                 |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Oraciones de ejemplo** | Expresiones de ejemplo que el usuario puede decir para desencadenar este comando.                                                                 |
| **Parámetros**       | Información necesaria para completar el comando.                                                                                |
| **Reglas de finalización** | Acciones que se van a realizar para finalizar el comando. Por ejemplo, para responder al usuario o comunicarse con otro servicio web. |
| **Reglas de interacción**   | Reglas adicionales para controlar situaciones más específicas o complejas.                                                              |


> [!div class="mx-imgBorder"]
> ![Creación de un comando](media/custom-commands/add-new-command.png)

### <a name="add-example-sentences"></a>Adición de oraciones de ejemplo

Empecemos por la sección **Oraciones de ejemplo** y especifiquemos un ejemplo de lo que el usuario puede decir.

1. Seleccione la sección **Oraciones de ejemplo** en el panel central.
1. En el panel del extremo derecho, agregue los ejemplos:

    ```
    turn on the tv
    ```

1.  En la parte superior del panel, seleccione **Guardar**.

Por ahora, no hay ningún parámetro, así que podemos continuar con la sección **Completion rules** (Reglas de finalización).

### <a name="add-a-completion-rule"></a>Adición de una regla de finalización

A continuación, el comando debe contener una regla de finalización. Esta regla informa al usuario que se está llevando a cabo una acción de cumplimiento. Para obtener más información sobre las reglas y las reglas de finalización, vaya a las [referencias](./custom-commands-references.md).

1. Seleccione la regla de finalización predeterminada **Listo** y edítela de la siguiente manera:

    
    | Configuración    | Valor sugerido                          | Descripción                                        |
    | ---------- | ---------------------------------------- | -------------------------------------------------- |
    | **Nombre**       | ConfirmationResponse                  | Nombre que describe el propósito de la regla.          |
    | **Condiciones** | None                                     | Condiciones que determinan cuándo se puede ejecutar la regla.    |
    | **Acciones**    | Enviar respuesta de voz > Editor sencillo > Primera variación > `Ok, turning the tv on` | Acción que se realizará cuando la condición de la regla sea true. |
    



   > [!div class="mx-imgBorder"]
   > ![Creación de una respuesta de voz](media/custom-commands/create-speech-response-action.png)

1. Seleccione **Guardar** para guardar la acción.
1. De nuevo en la sección **Completion rules** (reglas de finalización), seleccione **Guardar** para guardar todos los cambios. 


    > [!NOTE]
    > No es necesario usar la regla de finalización predeterminada que viene con el comando. Si es preciso, puede eliminar la regla de finalización predeterminada existente y agregar su propia regla.

### <a name="try-it-out"></a>Prueba

Use el panel de conversación de prueba para comprobar el comportamiento.
1. Seleccione el icono **Entrenar** presente en la parte superior del panel derecho.
1. Una vez completado el entrenamiento, seleccione **Probar**. Pruebe la siguiente expresión a través de voz/texto:
    - Entrada: turn on the tv (enciende el televisor)
    - Salida: Ok, turning the tv on (de acuerdo, encenderé el televisor)


> [!div class="mx-imgBorder"]
> ![Prueba con chat en web](media/custom-commands/create-basic-test-chat.png)

> [!TIP]
> En el panel de pruebas, puede seleccionar la opción **Turn details** (detalles del turno) para obtener información sobre cómo se procesó esta entrada de voz/texto.  

## <a name="add-settemperature-command"></a>Adición del comando SetTemperature

Ahora, agregue otro comando **SetTemperature** que usará una sola expresión (`set the temperature to 40 degrees`) y responderá con el mensaje `Ok, setting temperature to 40 degrees`.

Siga los pasos que se muestran para el comando **TurnOn** para crear un nuevo comando con la oración de ejemplo "**set the temperature to 40 degrees**" (establece la temperatura en 40 grados).

A continuación, edite las reglas de finalización **Listo** de la siguiente manera:

| Configuración    | Valor sugerido                          |
| ---------- | ---------------------------------------- |
| Nombre  | ConfirmationResponse                  |
| Condiciones | None                                     |
| Acciones    | Enviar respuesta de voz > Editor sencillo > Primera variación > `Ok, setting temperature to 40 degrees` |

Seleccione **Guardar** para guardar todos los cambios del comando.

## <a name="add-setalarm-command"></a>Adición de un comando SetAlarm
Cree un nuevo comando **SetAlarm** con la oración de ejemplo "**set an alarm for 9 am tomorrow**" (establece una alarma para mañana a las 9:00). A continuación, edite las reglas de finalización **Listo** de la siguiente manera:

| Configuración    | Valor sugerido                          |
| ---------- | ---------------------------------------- |
| Nombre de la regla  | ConfirmationResponse                  |
| Condiciones | None                                     |
| Acciones    | Enviar respuesta de voz > Editor sencillo > Primera variación > `Ok, setting an alarm for 9 am tomorrow` |

Seleccione **Guardar** para guardar todos los cambios del comando.

## <a name="try-it-out"></a>Prueba

Use el panel de conversación de prueba para comprobar el comportamiento.
1. Seleccione **Entrenar**. Una vez realizado el entrenamiento, seleccione **Probar** y pruebe lo siguiente:
    - Escriba: establece la temperatura en 40 grados
    - La respuesta esperada: Ok, setting temperature to 40 degrees (de acuerdo, definiré la temperatura en 40 grados)
    - Si escribe: turn on the tv (enciende el televisor)
    - La respuesta esperada: Ok, turning the tv on (de acuerdo, encenderé el televisor)
    - Escriba: set an alarm for 9 am tomorrow (establece una alarma para mañana a las 9:00)
    - La respuesta esperada: Ok, setting an alarm for 9 am tomorrow (de acuerdo, estableceré una alarma para mañana a las 9:00)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo: Adición de parámetros a comandos](./how-to-custom-commands-add-parameters-to-commands.md)
