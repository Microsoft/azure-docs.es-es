---
title: 'Tutorial: Exploración de los conceptos básicos con una aplicación cliente de ejemplo'
titleSuffix: Azure Digital Twins
description: Tutorial para explorar los SDK de Digital Twins mediante una aplicación de línea de comandos de ejemplo
author: baanders
ms.author: baanders
ms.date: 5/8/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: a1dc42815167da308fd87b541c0f21d02b47329b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022520"
---
# <a name="tutorial-explore-azure-digital-twins-with-a-sample-client-app"></a>Tutorial: Exploración de Azure Digital Twins con una aplicación cliente de ejemplo

En este tutorial se presenta una aplicación de ejemplo que implementa una aplicación cliente de línea de comandos, para interactuar con una instancia de Azure Digital Twins. La aplicación cliente es similar a la escrita en el [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md).

Puede usar este ejemplo para realizar acciones básicas de Azure Digital Twins, como cargar modelos, crear y modificar gemelos y crear relaciones. No obstante, si lo prefiere, también puede examinar el código del ejemplo para aprender sobre las API de Azure Digital Twins, y practicar la implementación de sus propios comandos mediante la modificación del proyecto de ejemplo.

En este tutorial, aprenderá lo siguiente:
> [!div class="checklist"]
> * Configuración de una instancia de Azure Digital Twins
> * Configuración de la aplicación de línea de comandos de ejemplo para interactuar con la instancia
> * Uso de la aplicación de línea de comandos para explorar Azure Digital Twins, por ejemplo, los **modelos**, los **gemelos digitales**, las **relaciones** y las **consultas**

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

## <a name="explore-with-the-sample-solution"></a>Exploración con la solución de ejemplo

Ahora que la instancia y la aplicación de ejemplo están configuradas, usará el proyecto de ejemplo y algunos códigos de ejemplo escritos previamente para crear y explorar una solución básica de Azure Digital Twins. Los componentes principales de la solución son **modelos**, **gemelos digitales** y **relaciones**, cuyo resultado es un **gráfico de gemelos** de un entorno.

### <a name="model-a-physical-environment-with-dtdl"></a>Modelado de un entorno físico con DTDL

El primer paso para crear una solución de Azure Digital Twins es definir los [**modelos**](concepts-models.md) de gemelos del entorno. 

Los modelos son parecidos a las clases de los lenguajes de programación orientados a objetos en el sentido de que proporcionan plantillas definidas por el usuario para [gemelos digitales](concepts-twins-graph.md), las cuales se siguen. Más tarde también se crean instancias de estas plantillas. Se escriben en un lenguaje tipo JSON llamado **lenguaje de definición de gemelos digitales (DTDL)** y pueden definir las *propiedades*, la *telemetría*, las *relaciones* y los *componentes* de un gemelo.

> [!NOTE]
> DTDL también permite la definición de *comandos* en digital twins. Sin embargo, en este momento no se admiten comandos en el servicio Azure Digital Twins.

En la ventana de Visual Studio donde está abierto el proyecto _**AdtE2ESample**_, use el panel *Explorador de soluciones* para ir a la carpeta *AdtSampleApp\SampleClientApp\Models*. Esta carpeta contiene modelos de ejemplo.

Seleccione *Room.json* para abrirlo en la ventana de edición y cámbielo de la siguiente manera:

* **Actualice el número de versión** para indicar que proporciona una versión más actualizada de este modelo. Para ello, cambie el *1* al final del valor `@id` por *2*. También servirá cualquier número mayor que el número de versión actual.
* **Edite una propiedad**. Cambie el nombre de la propiedad `Humidity` por *HumidityLevel* (o algo diferente que prefiera. Si usa algo diferente a *HumidityLevel*, recuerde lo que ha usado y no lo cambie por *HumidityLevel* a lo largo de todo el tutorial).
* **Agregue una propiedad**. Debajo de la propiedad `HumidityLevel` que termina en la línea 15, pegue el código siguiente para agregar una propiedad `RoomName` a la sala:

    ```json
    ,
    {
      "@type": "Property",
      "name": "RoomName",
      "schema": "string"
    }
    ```
* **Agregue una relación**. Debajo de la propiedad `RoomName` que acaba de agregar, pegue el código siguiente para que este tipo de gemelo pueda formar relaciones *contiene* con otros gemelos:

    ```json
    ,
    {
      "@type": "Relationship",
      "name": "contains",
    }
    ```

Cuando haya terminado, el modelo actualizado se parecerá a este:

:::image type="content" source="media/tutorial-command-line-app/room-model.png" alt-text="Room.json editado con el número de versión actualizado, las propiedades HumidityLevel y RoomName, y la relación contiene" border="false":::

Asegúrese de guardar el archivo antes de continuar.

> [!TIP]
> Si quiere intentar crear su propio modelo, puede pegar el código del modelo *Room* en un nuevo archivo que guarde con una extensión *.json* en la carpeta *AdtSampleApp\SampleClientApp\Models*. Después, experimente con la adición de propiedades y relaciones para representar lo que quiera. Para encontrar ideas, puede examinar los otros modelos de ejemplo de esta carpeta.

> [!TIP] 
> Hay un [ejemplo de validador de DTDL](https://docs.microsoft.com/samples/azure-samples/dtdl-validator/dtdl-validator) independiente del lenguaje que puede usar para comprobar los documentos del modelo y asegurarse de que el DTDL sea válido. Se basa en la biblioteca del analizador de DTDL, sobre la que puede leer más en [*Procedimiento: Análisis y validación de modelos*](how-to-parse-models.md).

### <a name="get-started-with-the-command-line-app"></a>Introducción a la aplicación de línea de comandos

Ahora que ha definido un modelo, el resto de los pasos supone el uso de la aplicación de ejemplo para interactuar con la instancia de Azure Digital Twins. Ejecute el proyecto con este botón de la barra de herramientas:

:::image type="content" source="media/tutorial-command-line-app/start-button-sample.png" alt-text="Botón de inicio de Visual Studio (proyecto SampleClientApp)":::

Se abre una ventana de la consola, se lleva a cabo la autenticación y se espera un comando. 
* La autenticación se realiza a través del explorador: el explorador web predeterminado se abrirá con un aviso de autenticación. Use este aviso para iniciar sesión con sus credenciales de Azure. Luego, puede cerrar la pestaña o la ventana del explorador.

Esta es una captura de pantalla de la apariencia de la consola del proyecto:

:::image type="content" source="media/tutorial-command-line-app/command-line-app.png" alt-text="Mensaje de bienvenida de la aplicación de línea de comandos":::

> [!TIP]
> Si quiere una lista de todos los comandos posibles que puede usar con este proyecto, escriba `help` en la consola del proyecto y presione Entrar.
> :::image type="content" source="media/tutorial-command-line-app/command-line-app-help.png" alt-text="Salida del comando de ayuda":::

Mantenga la consola del proyecto en ejecución durante el resto de los pasos de este tutorial.

#### <a name="upload-models-to-azure-digital-twins"></a>Carga de modelos en Azure Digital Twins

Después de diseñar los modelos, debe cargarlos en la instancia de Azure Digital Twins. De esta forma se configura la instancia del servicio Azure Digital Twins con su propio vocabulario de dominio personalizado. Cuando haya cargado los modelos, puede crear instancias gemelas que los usen.

En la ventana de la consola del proyecto, ejecute el siguiente comando para cargar el modelo *Room* actualizado, así como un modelo *Floor* que también usará en la sección siguiente para crear distintos tipos de gemelos.

```cmd/sh
CreateModels Room Floor
```

La salida debe indicar que los modelos se crearon correctamente.

> [!TIP]
> Si ha diseñado su propio modelo anteriormente, también puede cargarlo aquí; para ello, agregue su nombre de archivo (puede omitir la extensión) a la lista `Room Floor` en el comando anterior.

Para comprobar que los modelos se han creado, ejecute el comando `GetModels true`. Se consultará la instancia de Azure Digital Twins para conocer todos los modelos que se han cargado y se imprimirá toda su información. Busque el modelo *Room* editado en los resultados:

:::image type="content" source="media/tutorial-command-line-app/output-get-models.png" alt-text="Resultados de GetModels, que muestra el modelo Room actualizado":::

#### <a name="errors"></a>Errors

La aplicación de ejemplo también controla los errores del servicio. 

Vuelva a ejecutar el comando `CreateModels` para intentar volver a cargar uno de los mismos modelos que acaba de cargar, por segunda vez:

```cmd/sh
CreateModels Room
```

Dado que los modelos no se pueden sobrescribir, esto le devolverá un error de servicio.
Para información detallada sobre cómo eliminar modelos existentes, consulte [*Procedimientos: Administración de modelos personalizados*](how-to-manage-model.md).
```cmd/sh
Response 409: Service request failed.
Status: 409 (Conflict)

Content:
{"error":{"code":"ModelAlreadyExists","message":"Could not add model dtmi:example:Room;2 as it already exists. Use Model_List API to view models that already exist. See the Swagger example.(http://aka.ms/ModelListSwSmpl)"}}

Headers:
Strict-Transport-Security: REDACTED
Date: Wed, 20 May 2020 00:53:49 GMT
Content-Length: 223
Content-Type: application/json; charset=utf-8
```

### <a name="create-digital-twins"></a>Creación de gemelos digitales

Ahora que algunos modelos se han cargado en la instancia de Azure Digital Twins, puede crear [**gemelos digitales**](concepts-twins-graph.md) basados en las definiciones de modelo. Los gemelos digitales representan las entidades del entorno empresarial, como los sensores de una granja, las salas de un edificio o las luces de un coche. 

Para crear un gemelo digital, se usa el comando `CreateDigitalTwin`. Se debe hacer referencia al modelo en el que se basa el gemelo y, opcionalmente, puede definir los valores iniciales de las propiedades del modelo. En esta fase, no es necesario pasar ninguna información de relación.

Ejecute este código en la consola del proyecto en ejecución para crear varios gemelos, según el modelo *Room* que ha actualizado anteriormente y otro modelo, *Floor*. Recuerde que *Room* tiene tres propiedades, por lo que puede proporcionar argumentos con los valores iniciales de estas.

```cmd/sh
CreateDigitalTwin dtmi:example:Room;2 room0 RoomName string Room0 Temperature double 70 HumidityLevel double 30
CreateDigitalTwin dtmi:example:Room;2 room1 RoomName string Room1 Temperature double 80 HumidityLevel double 60
CreateDigitalTwin dtmi:example:Floor;1 floor0
CreateDigitalTwin dtmi:example:Floor;1 floor1
```

> [!TIP]
> Si cargó su propio modelo anteriormente, pruebe a crear su propio comando `CreateDigitalTwin` según los comandos anteriores para agregar un gemelo de su propio tipo de modelo.

La salida de estos comandos debe indicar que los gemelos se crearon correctamente. 

:::image type="content" source="media/tutorial-command-line-app/output-create-digital-twin.png" alt-text="Extracto de los resultados de los comandos de CreateDigitalTwin, que muestra floor0, floor1, room0 y room1":::

También puede comprobar que se han creado los gemelos con el comando `Query`. Este comando consulta la instancia de Azure Digital Twins para conocer todos los gemelos digitales que contiene. Busque los gemelos *floor0*, *floor1*, *room0* y *room1* en los resultados.

#### <a name="modify-a-digital-twin"></a>Modificación de un gemelo digital

También puede modificar las propiedades de un gemelo que haya creado. Pruebe a ejecutar este comando para cambiar la propiedad RoomName de *room0* de *Room0* a *PresidentialSuite*:

```cmd/sh
UpdateDigitalTwin room0 add /RoomName string PresidentialSuite
```

La salida indicará que el gemelo se actualizó correctamente.

Otra forma de comprobarlo es ejecutar este comando para ver la información de *room0*:

```cmd/sh
GetDigitalTwin room0
```

La salida reflejará el nombre actualizado.

> [!NOTE]
> La API REST subyacente usa la revisión de JSON para definir las actualizaciones de un gemelo. La aplicación de línea de comandos refleja este formato, por lo que puede experimentar con lo que realmente esperan las API subyacentes.

### <a name="create-a-graph-by-adding-relationships"></a>Creación de un gráfico mediante la adición de relaciones

A continuación, puede crear algunas **relaciones** entre estos gemelos para conectarlos en un [**gráfico de gemelos**](concepts-twins-graph.md). Los gráficos de gemelos se utilizan para representar un entorno completo. 

Para agregar una relación, use el comando `CreateRelationship`. Especifique el gemelo del que procede la relación, el tipo de relación que se va a agregar y el gemelo con el que conecta la relación. Por último, proporcione un nombre (identificador) para la relación.

Ejecute el código siguiente para agregar una relación "contiene" desde cada uno de los gemelos *Floor* que creó anteriormente hasta un gemelo *Room* correspondiente. Tenga en cuenta que, para que esto sea posible, debe haber una relación *contiene* definida en el modelo *Floor*.

```cmd/sh
CreateRelationship floor0 contains room0 relationship0
CreateRelationship floor1 contains room1 relationship1
```

La salida de estos comandos confirma que las relaciones se crearon correctamente:

:::image type="content" source="media/tutorial-command-line-app/output-create-relationship.png" alt-text="Extracto de los resultados de los comandos CreateRelationship, que muestra relationship0 y relationship1":::

También puede comprobar las relaciones con cualquiera de los siguientes comandos, que consultan las relaciones en la instancia de Azure Digital Twins.
* Para ver todas las relaciones que proceden de cada planta (vista de las relaciones desde un lado):
    ```cmd/sh
    GetRelationships floor0
    GetRelationships floor1
    ```
* Para ver todas las relaciones que llegan a cada sala (vista de la relación desde el "otro" lado):
    ```cmd/sh
    GetIncomingRelationships room0
    ```
* Para consultar estas relaciones de forma individual: 
    ```cmd/sh
    GetRelationship floor0 contains relationship0
    GetRelationship floor1 contains relationship1
    ```

Los gemelos y las relaciones que ha configurado en este tutorial forman el siguiente gráfico conceptual:

:::image type="content" source="media/tutorial-command-line-app/sample-graph.png" alt-text="Un gráfico que muestra floor0 conectado a través de relationship0 con room0 y floor1 conectado a través de relationship1 con room1" border="false":::

### <a name="query-the-twin-graph-to-answer-environment-questions"></a>Consulta del gráfico de gemelos para responder a las preguntas del entorno

Una de las principales características de Azure Digital Twins es la posibilidad de [consultar](concepts-query-language.md) el gráfico de gemelos de forma fácil y eficaz para responder a las preguntas sobre el entorno. Ejecute los siguientes comandos en la consola del proyecto en ejecución para hacerse una idea del aspecto que tendrá.

* **¿Cuáles son todas las entidades de mi entorno que se representan en Azure Digital Twins?** (consulta de todo)

    ```cmd/sh
    Query
    ```

    Esto le permite evaluar su entorno de un vistazo y asegurarse de que todo se representa tal y como desea en Azure Digital Twins. El resultado es una salida que contiene cada gemelo digital con sus detalles. Aquí se muestra un extracto:

    :::image type="content" source="media/tutorial-command-line-app/output-query-all.png" alt-text="Resultados parciales de una consulta de gemelos, que muestra room0 y floor1":::

    >[!NOTE]
    >El comando `Query` sin ningún argumento adicional es el equivalente de `Query SELECT * FROM DIGITALTWINS`.

* **¿Cuáles son todas las salas de mi entorno?** (consulta por modelo)

    ```cmd/sh
    Query SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')
    ```

    Puede restringir la consulta a los gemelos de un tipo determinado para tener información más específica sobre lo que se representa. El resultado muestra *room0* y *room1*, pero **no** *floor0* o *floor1* (dado que son plantas, no salas).
    
    :::image type="content" source="media/tutorial-command-line-app/output-query-model.png" alt-text="Resultados de la consulta del modelo, que muestra solo room0 y room1":::

* **¿Cuáles son todas las salas de *floor0*?** (consulta por relación)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0'
    ```

    Puede realizar consultas en función de las relaciones del gráfico para tener información sobre cómo se conectan los gemelos o para restringir la consulta a un área determinada. Solo *room0* está en *floor0*, por lo que es la única sala del resultado.

    :::image type="content" source="media/tutorial-command-line-app/output-query-relationship.png" alt-text="Resultados de la consulta de relación, que muestra room0":::

* **¿Cuáles son todos los gemelos de mi entorno con una temperatura superior a 75?** (consulta por propiedad)

    ```cmd/sh
    Query SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
    ```

    Puede consultar el gráfico en función de las propiedades para responder a diversas preguntas, como buscar valores atípicos en el entorno que puedan necesitar atención. También se admiten otros operadores de comparación ( *<* , *>* , *=* o *!=* ). *room1* se muestra aquí en los resultados porque tiene una temperatura de 80.

    :::image type="content" source="media/tutorial-command-line-app/output-query-property.png" alt-text="Resultados de la consulta de propiedades, que muestra solo room1":::

* **¿Cuáles son todas las salas de *floor0* con una temperatura superior a 75?** (consulta compuesta)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75
    ```

    También puede combinar las consultas anteriores como lo haría en SQL, mediante operadores combinados como `AND`, `OR``NOT`. Esta consulta usa `AND` para que la consulta anterior sobre las temperaturas gemelas sea más específica. El resultado ahora solo incluye las salas con temperaturas superiores a 75 que se encuentran en *floor0*, que en este caso no es ninguna de ellas. El conjunto de resultados está vacío.

    :::image type="content" source="media/tutorial-command-line-app/output-query-compound.png" alt-text="Resultados de la consulta compuesta, que no muestra ningún resultado":::

## <a name="clean-up-resources"></a>Limpieza de recursos

El proyecto de este tutorial constituye la base del próximo tutorial, [*Tutorial: Conexión de una solución de un extremo a otro*](tutorial-end-to-end.md). Si tiene previsto continuar con el siguiente tutorial, puede mantener los recursos configurados aquí para seguir usando esta instancia de Azure Digital Twins y la aplicación de ejemplo configurada.
* En este caso, puede usar los comandos `DeleteAllTwins` y `DeleteAllModels` de la aplicación de ejemplo para borrar los gemelos y los modelos de la instancia, respectivamente. De esta forma, tendrá una pizarra en blanco para el siguiente tutorial.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Por último, elimine la carpeta de ejemplo del proyecto que descargó en la máquina local.

## <a name="next-steps"></a>Pasos siguientes 

En este tutorial, ha empezado a trabajar con Azure Digital Twins, para lo cual ha configurado una instancia y una aplicación cliente a fin de interactuar con la instancia. Ha usado la aplicación cliente para explorar Azure Digital Twins y crear modelos, gemelos digitales y relaciones. También ha ejecutado algunas consultas de la solución para hacerse una idea de los tipos de preguntas que Azure Digital Twins puede responder sobre un entorno.

Continúe con el siguiente tutorial para usar la aplicación de línea de comandos de ejemplo en combinación con otros servicios de Azure a fin de completar un escenario de un extremo a otro basado en datos:
> [!div class="nextstepaction"]
> [*Tutorial: Conexión de una solución de un extremo a otro*](tutorial-end-to-end.md).