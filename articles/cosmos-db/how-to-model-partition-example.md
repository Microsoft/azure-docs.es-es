---
title: Modelado y creación de particiones de datos en Azure Cosmos DB mediante un ejemplo real
description: Aprenda a modelar y crear particiones en un ejemplo real mediante Core API de Azure Cosmos DB
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/23/2019
ms.author: thweiss
ms.custom: devx-track-javascript
ms.openlocfilehash: d5809d7475759450a513153abf641f7943163d98
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422222"
---
# <a name="how-to-model-and-partition-data-on-azure-cosmos-db-using-a-real-world-example"></a>Procedimientos para modelar y crear particiones de datos en Azure Cosmos DB mediante un ejemplo real

Este artículo se basa en varios conceptos de Azure Cosmos DB como el [modelado de datos](modeling-data.md), la [creación de particiones](partitioning-overview.md) y [rendimiento aprovisionado](request-units.md) para demostrar cómo abordar un ejercicio de diseño de datos reales.

Si suele trabajar con bases de datos relacionales, es probable que haya desarrollado hábitos e intuiciones acerca de cómo diseñar un modelo de datos. Dadas no solo las restricciones específicas, sino también los puntos fuertes exclusivos de Azure Cosmos DB, la mayoría de estos procedimientos recomendados no se traduce bien y es posible que le lleve a soluciones que no llegan a ser óptimas. El objetivo de este artículo es guiarle por todo el proceso de modelado de un caso de uso real en Azure Cosmos DB, desde el modelado de elementos a la colocación de entidades y la creación de particiones en contenedores.

## <a name="the-scenario"></a>El escenario

Para este ejercicio, vamos a tener en cuenta el dominio de una plataforma de blogs en las que los *usuarios* puede crear *publicaciones*. Los usuarios también pueden indicar que dichas publicaciones les *gustan* y agregarles *comentarios*.

> [!TIP]
> Hemos resaltado algunas palabras en *cursiva*; dichas palabras identifican el tipo de "cosas" que nuestro modelo va a tener que manipular.

Incorporación de más requisitos a la especificación:

- Una página frontal muestra una fuente de publicaciones recientemente creadas.
- Podemos capturar todas las publicaciones de un usuario, todos los comentarios de una publicación y todos los "Me gusta" de una publicación.
- Las publicaciones se devuelven con el nombre de usuario de sus autores y el número de comentarios y "Me gusta" que tienen.
- Los comentarios y "Me gusta" también se devuelven con el nombre de usuario de los usuarios que los han creado.
- Cuando se muestran en forma de listas, las publicaciones solo tienen que presentar un resumen truncado de su contenido.

## <a name="identify-the-main-access-patterns"></a>Identificación de los patrones de acceso principales

Para empezar, proporcionamos cierta estructura a nuestra especificación inicial mediante la identificación de los patrones de acceso de nuestra solución. Al diseñar un modelo de datos para Azure Cosmos DB, es importante saber qué solicitudes tendrá que atender nuestro modelo para tener la certeza de que el modelo va a hacerlo de manera eficiente.

Para facilitar el seguimiento del proceso completo, clasificamos las diferentes solicitudes como comandos o consultas, para lo que usamos la terminología de [CQRS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation#Command_query_responsibility_segregation), en la que los comandos son solicitudes de escritura (es decir, intenciones de actualizar el sistema) y las consultas son solicitudes de solo lectura.

Esta es la lista de solicitudes que se deberá exponer nuestra plataforma:

- **[C1]**  Crear o editar un usuario
- **[Q1]** Recuperar un usuario
- **[C2]** Crear o editar una publicación
- **[Q2]** Recuperar una publicación
- **[Q3]**  Enumerar las publicaciones de un usuario en forma abreviada
- **[C3]**  Crear un comentario
- **[P4]**  Enumerar los comentarios de una publicación
- **[C4]**  Gustar una publicación
- **[Q5]**  Enumerar los "Me gusta" de una publicación
- **[P6]**  Enumerar las *x* publicaciones más recientes creadas en formato abreviado (fuente)

En esta fase aún no hemos pensado en los detalles de lo que va a contener cada entidad (usuario, publicación, etc.) Este paso suele ser de los primeros en abordarse al diseñar en un almacén relacional, ya que tenemos que averiguar cómo se traducirán dichas entidades en términos de tablas, columnas, claves externas, etc. El problema es mucho menor cuando se trabaja con una base de datos documental que no exige ningún esquema en la escritura.

El motivo principal por el que es importante identificar los patrones de acceso desde el principio, es que esta lista de solicitudes va a ser nuestro conjunto de pruebas. Cada vez que recorramos el modelo de datos, vamos a pasar por todas y cada una de las solicitudes y a comprobar su rendimiento y escalabilidad.

## <a name="v1-a-first-version"></a>V1: una primera versión

Comenzamos con dos contenedores: `users` y `posts`.

### <a name="users-container"></a>Contenedor users

Este contenedor solo almacena elementos de usuario:

```json
{
    "id": "<user-id>",
    "username": "<username>"
}
```

La partición de este contenedor la realizamos por `id`, lo que significa que cada partición lógica del contenedor solo contendrá un elemento.

### <a name="posts-container"></a>Contenedor posts

Este contenedor hospeda publicaciones, comentarios y "Me gusta":

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "title": "<post-title>",
    "content": "<post-content>",
    "creationDate": "<post-creation-date>"
}

{
    "id": "<comment-id>",
    "type": "comment",
    "postId": "<post-id>",
    "userId": "<comment-author-id>",
    "content": "<comment-content>",
    "creationDate": "<comment-creation-date>"
}

{
    "id": "<like-id>",
    "type": "like",
    "postId": "<post-id>",
    "userId": "<liker-id>",
    "creationDate": "<like-creation-date>"
}
```

La partición de este contenedor la realizamos por `postId`, lo que significa que cada partición lógica de dicho contenedor contendrá no solo una publicación, sino también todos los comentarios y "Me gusta" de la misma.

Tenga en cuenta que hemos introducido una propiedad `type` en los elementos almacenados en este contenedor para establecer una distinción entre los tres tipos de entidades que este contenedor hospeda.

Además, hemos elegido hacer referencia a los datos relacionados, en lugar de incrustarlo (consulte [esta sección](modeling-data.md) para más información acerca de estos conceptos) porque:

- no hay límite superior en el número de publicaciones que puede crear un usuario,
- las publicaciones pueden ser arbitrariamente largas,
- no límite superior con respecto al número de comentarios y "Me gusta" que puede tener una publicación
- queremos poder agregar un comentario o un "Me gusta" a una publicación sin tener que actualizar la propia publicación.

## <a name="how-well-does-our-model-perform"></a>¿Hasta qué punto funciona bien nuestro modelo?

Ahora es el momento de evaluar el rendimiento y la escalabilidad de nuestra primera versión. En cada una de las solicitudes que ha identificado anteriormente, medimos su latencia y el número de unidades de solicitud que consume. Esta medida se realiza en un conjunto de datos ficticio que contiene 100 000 usuarios con entre 5 y 50 publicaciones por usuario y hasta 25 comentarios y 100 "Me gusta" por publicación.

### <a name="c1-createedit-a-user"></a>[C1] Crear o editar un usuario

Esta solicitud es fácil de implementar, ya que acabamos de crear o actualizar un elemento en el contenedor `users`. Las solicitudes se esparcirán entre todas las particiones gracias a la clave de partición `id`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C1.png" alt-text="Escritura de un elemento individual en el contenedor users" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 7 ms | 5,71 RU | ✅ |

### <a name="q1-retrieve-a-user"></a>[Q1] Recuperar un usuario

La recuperación de los usuarios se realiza mediante la lectura del elemento correspondiente del contenedor `users`.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q1.png" alt-text="Recuperación de un elemento individual del contenedor users" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 2 ms | 1 RU | ✅ |

### <a name="c2-createedit-a-post"></a>[C2] Crear o editar una publicación

Del mismo modo que **[C1]** , solo tenemos que escribir en el contenedor `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Escritura de un elemento individual en el contenedor posts" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 9 ms | 8,76 RU | ✅ |

### <a name="q2-retrieve-a-post"></a>[Q2] Recuperar una publicación

Empezaremos por recuperar el documento correspondiente del contenedor `posts`. Pero eso no es suficiente, ya que, de acuerdo a nuestra especificación, tenemos que agregar el nombre de usuario del autor de la publicación y el número de comentarios y "Me gusta" que tiene la publicación, lo que requiere la emisión de 3 consultas SQL adicionales.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q2.png" alt-text="Recuperación de una publicación e incorporación de datos adicionales" border="false":::

Cada uno de los filtros de consultas adicionales de la clave de partición de su respectivo contenedor, que es exactamente lo que deseamos para maximizar el rendimiento y la escalabilidad. Pero eventualmente tenemos que realizar cuatro operaciones para devolver una publicación individual, lo que mejoraremos en una iteración posterior.

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 9 ms | 19,54 RU | ⚠ |

### <a name="q3-list-a-users-posts-in-short-form"></a>[Q3] Enumerar las publicaciones de un usuario en forma abreviada

En primer lugar, tenemos que recuperar las publicaciones deseadas con una consulta SQL que captura las publicaciones correspondientes al usuario concreto. Pero también tenemos que emitir consultas adicionales para agregar el nombre de usuario del creador y el número de comentarios y "Me gusta".

:::image type="content" source="./media/how-to-model-partition-example/V1-Q3.png" alt-text="Recuperación de todas las publicaciones de un usuario e incorporación de sus datos adicionales" border="false":::

Esta implementación presenta muchas desventajas:

- las consultas que agregan el número de comentarios y "Me gusta" deben emitirse para cada publicación que devuelve la primera consulta,
- la consulta principal no se filtra por la clave de partición del contenedor `posts`, lo que provoca una distribución ramificada y un examen de las particiones del contenedor.

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 130 ms | 619,41 RU | ⚠ |

### <a name="c3-create-a-comment"></a>[C3] Crear un comentario

Los comentarios se crean mediante la escritura del elemento correspondiente en el contenedor `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Escritura de un elemento individual en el contenedor posts" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 7 ms | 8,57 RU | ✅ |

### <a name="q4-list-a-posts-comments"></a>[Q4] Enumerar los comentarios de una publicación

Comenzamos con una consulta que captura todos los comentarios de la publicación y, una vez más, es preciso agregar los nombres de usuario agregados por separado para cada comentario.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q4.png" alt-text="Recuperación de todos los comentarios de una publicación y agregación de sus datos adicionales" border="false":::

Aunque la consulta principal filtrar por la clave de partición del contenedor, agregar los nombres de usuario por separado penaliza el rendimiento general. Eso lo mejoraremos más adelante.

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 23 ms | 27,72 RU | ⚠ |

### <a name="c4-like-a-post"></a>[C4] Gustar una publicación

Al igual que **[C3]** , creamos el elemento correspondiente en el contenedor `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Escritura de un elemento individual en el contenedor posts" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 6 ms | 7,05 RU | ✅ |

### <a name="q5-list-a-posts-likes"></a>[Q5] Enumerar los "Me gusta" de una publicación

Al igual que **[Q4]** , se consulta los "Me gusta" para la publicación y, después, se agregan sus nombres de usuario.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q5.png" alt-text="Recuperación de todos los Me gusta de una publicación y agregación de sus datos adicionales" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 59 ms | 58,92 RU | ⚠ |

### <a name="q6-list-the-x-most-recent-posts-created-in-short-form-feed"></a>[Q6] Enumerar las x publicaciones más recientes creadas en formato abreviado (fuente)

Para capturar las publicaciones más recientes, consultamos el contenedor `posts` ordenado por fecha de creación orden, de forma descendente, y, después, los nombres de usuario agregados y el número de comentarios y "Me gusta" de cada una de las publicaciones.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q6.png" alt-text="Recuperación de las publicaciones más recientes e incorporación de sus datos adicionales" border="false":::

Una vez más, la consulta inicial no filtra por la clave de partición del contenedor `posts`, lo que desencadena una costosa distribución ramificada. Esta es incluso peor, ya que el destino es un conjunto de resultados mucho mayor y los resultados se ordenan con una cláusula `ORDER BY`, lo que hace que sea más cara en términos de unidades de solicitud.

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 306 ms | 2063,54 RU | ⚠ |

## <a name="reflecting-on-the-performance-of-v1"></a>Reflexión en el rendimiento de V1

Al examinar los problemas de rendimiento que nos encontramos en la sección anterior, podemos identificar dos clases principales:

- algunas solicitudes requieren que se emitan varias consultas para recopilar todos los datos que hay que devolver,
- algunas consultas no filtran por la clave de partición de los contenedores a los que van dirigidas, lo que da lugar a una distribución ramificada que impide la escalabilidad.

Vamos a resolver cada uno de estos problemas, empezando por el primero.

## <a name="v2-introducing-denormalization-to-optimize-read-queries"></a>V2: presentación de la desnormalización para optimizar las consultas de lectura

El motivo por el qué en algunos casos es preciso emitir solicitudes adicionales es que los resultados de la solicitud inicial no contienen todos los datos que necesitamos devolver. Cuando se trabaja con un almacén de datos no relacionales como Azure Cosmos DB, este tipo de problema se soluciona normalmente mediante la desnormalización de los datos de nuestro conjunto de datos.

En nuestro ejemplo, modificamos los elementos de la publicación para agregar el nombre de usuario del autor de la publicación y el número de comentarios y "Me gusta":

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

También modificamos los elementos de comentarios y, "Me gusta" para agregar el nombre de usuario del usuario que los ha creado:

```json
{
    "id": "<comment-id>",
    "type": "comment",
    "postId": "<post-id>",
    "userId": "<comment-author-id>",
    "userUsername": "<comment-author-username>",
    "content": "<comment-content>",
    "creationDate": "<comment-creation-date>"
}

{
    "id": "<like-id>",
    "type": "like",
    "postId": "<post-id>",
    "userId": "<liker-id>",
    "userUsername": "<liker-username>",
    "creationDate": "<like-creation-date>"
}
```

### <a name="denormalizing-comment-and-like-counts"></a>Desnormalización del número de comentarios y, "Me gusta"

Lo que queremos conseguir es que cada vez que agregamos un comentario o un "Me gusta", también aumentamos `commentCount` o `likeCount` en la publicación correspondiente. Como en nuestro contenedor `posts``postId` ha creado particiones, el nuevo elemento (comentario o "Me gusta") y su publicación correspondiente se colocan en la misma partición lógica. Como resultado, podemos usar un [procedimiento almacenado](stored-procedures-triggers-udfs.md) para realizar dicha operación.

Ahora, al crear un comentario ( **[C3]** ), en lugar de simplemente agregar un nuevo elemento al contenedor `posts` llamamos al siguiente procedimiento almacenado de dicho contenedor:

```javascript
function createComment(postId, comment) {
  var collection = getContext().getCollection();

  collection.readDocument(
    `${collection.getAltLink()}/docs/${postId}`,
    function (err, post) {
      if (err) throw err;

      post.commentCount++;
      collection.replaceDocument(
        post._self,
        post,
        function (err) {
          if (err) throw err;

          comment.postId = postId;
          collection.createDocument(
            collection.getSelfLink(),
            comment
          );
        }
      );
    })
}
```

Este procedimiento almacenado toma el identificador de la publicación y el cuerpo del nuevo comentario como parámetros y luego:

- recupera la publicación
- incrementa el valor de `commentCount`
- reemplaza la publicación
- agrega el nuevo comentario

Dado que los procedimientos almacenados se ejecutan como transacciones atómicas, se garantiza que el valor de `commentCount` y el número real de comentarios siempre estarán sincronizados.

Obviamente llamamos a un procedimiento almacenado similar al agregar nuevos "Me gusta" para incrementar `likeCount`.

### <a name="denormalizing-usernames"></a>Desnormalización de nombres de usuario

Los nombres de usuario requieren un enfoque diferente, ya que los usuarios no solo se encuentran en particiones distintas, sino también en un contenedor diferente. Cuando tenemos que desnormalizar los datos en las particiones y contenedores, podemos usar la [fuente de cambios](change-feed.md) del contenedor de origen.

En nuestro ejemplo, usamos la fuente de cambios del contenedor `users` para reaccionar cuando los usuarios actualizan sus nombres de usuario. Cuando esto ocurre, propagamos el cambio llamando a otro procedimiento almacenado del contenedor `posts`:

:::image type="content" source="./media/how-to-model-partition-example/denormalization-1.png" alt-text="Desnormalización de los nombres de usuario en el contenedor posts" border="false":::

```javascript
function updateUsernames(userId, username) {
  var collection = getContext().getCollection();
  
  collection.queryDocuments(
    collection.getSelfLink(),
    `SELECT * FROM p WHERE p.userId = '${userId}'`,
    function (err, results) {
      if (err) throw err;

      for (var i in results) {
        var doc = results[i];
        doc.userUsername = username;

        collection.upsertDocument(
          collection.getSelfLink(),
          doc);
      }
    });
}
```

Este procedimiento almacenado toma el identificador del usuario y el nuevo nombre de usuario del usuario como parámetros y luego:

- recupera todos los elementos que coinciden con `userId` (que puede ser publicaciones, comentarios, o "Me gusta")
- en cada uno de los elementos
  - reemplaza el valor de `userUsername`
  - reemplaza el elemento

> [!IMPORTANT]
> Esta operación es costosa porque requiere que este procedimiento almacenado se ejecute en todas las particiones del contenedor `posts`. Suponemos que la mayoría de los usuarios eligen un nombre de usuario adecuado en el registro y que nunca lo cambiará, por lo que esta actualización se ejecutará con muy poca frecuencia.

## <a name="what-are-the-performance-gains-of-v2"></a>¿Cuáles son las mejoras de rendimiento de V2?

### <a name="q2-retrieve-a-post"></a>[Q2] Recuperar una publicación

Ahora que la desnormalización está en vigor, solo tenemos que capturar un elemento para controlar la solicitud.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q2.png" alt-text="Recuperación de un elemento individual del contenedor posts" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 2 ms | 1 RU | ✅ |

### <a name="q4-list-a-posts-comments"></a>[Q4] Enumerar los comentarios de una publicación

Aquí podemos volver a compartir solicitudes adicionales que han capturado los el nombres de usuario y acabar con una sola consulta que filtra por la clave de partición.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q4.png" alt-text="Recuperación de los comentarios de una publicación" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 4 ms | 7,72 RU | ✅ |

### <a name="q5-list-a-posts-likes"></a>[Q5] Enumerar los "Me gusta" de una publicación

Exactamente la misma cuando se enumeran los "Me gusta".

:::image type="content" source="./media/how-to-model-partition-example/V2-Q5.png" alt-text="Recuperación de los Me gusta de una publicación" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 4 ms | 8,92 RU | ✅ |

## <a name="v3-making-sure-all-requests-are-scalable"></a>V3: asegurarse de que todas las solicitudes se pueden escalar

Si se examinan nuestras mejoras generales en el rendimiento, aún hay dos solicitudes que no hemos optimizado totalmente: **[Q3]** y **[P6]** . Son las solicitudes que implican consultas que no filtran por la clave de partición de los contenedores a que se dirige.

### <a name="q3-list-a-users-posts-in-short-form"></a>[Q3] Enumerar las publicaciones de un usuario en forma abreviada

Esta solicitud ya se beneficia de las mejoras introducidas en V2, que comparte consultas adicionales.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q3.png" alt-text="Recuperación de todas las publicaciones de un usuario" border="false":::

Pero la consulta restante no se filtra por la clave de partición del contenedor `posts`.

La manera de pensar en esta situación es realmente sencilla:

1. Esta solicitud *tiene* que filtrar por `userId`, ya que deseamos capturar todas las publicaciones de un usuario concreto
1. No funciona bien porque se ejecuta en el contenedor `posts`, en el que no se crean particiones mediante `userId`.
1. Empezando por lo obvio, podríamos resolver nuestro problema de rendimiento mediante la ejecución de esta solicitud en un contenedor en el que *se crean* particiones mediante `userId`.
1. Resulta que ya tenemos ese contenedor: el contenedor `users`.

Por tanto, introducimos un segundo nivel de desnormalización mediante la duplicación de publicaciones completas en el contenedor `users`. Al hacerlo, obtenemos una copia de nuestras publicaciones, en las que solo se crean particiones en dimensiones diferentes, lo que sea mucho más eficaz recuperarlas por `userId`.

El contenedor `users` ahora tiene dos tipos de elementos:

```json
{
    "id": "<user-id>",
    "type": "user",
    "userId": "<user-id>",
    "username": "<username>"
}

{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

Observe lo siguiente:

- hemos introducido un campo `type` en el elemento de usuario para distinguir a los usuarios de las publicaciones,
- también hemos agregado un campo `userId` en el elemento de usuario, que es redundante con el campo `id`, pero es obligatorio, ya que en el contenedor `users` ahora crea las particiones `userId` (no `id` como antes)

Para lograr dicha desnormalización, usamos una vez más la fuente de cambios. Esta vez reaccionamos ante la fuente de cambios del contenedor `posts` para enviar cualquier publicación nueva o actualizada al contenedor `users`. Y como la enumeración de publicaciones no requiere devolver todo su contenido, podemos truncarlas en el proceso.

:::image type="content" source="./media/how-to-model-partition-example/denormalization-2.png" alt-text="Desnormalización de publicaciones del contenedor users" border="false":::

Ahora podemos enrutar nuestra consulta al contenedor `users`, filtrando por la clave de partición del contenedor.

:::image type="content" source="./media/how-to-model-partition-example/V3-Q3.png" alt-text="Recuperación de todas las publicaciones de un usuario" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 4 ms | 6,46 RU | ✅ |

### <a name="q6-list-the-x-most-recent-posts-created-in-short-form-feed"></a>[Q6] Enumerar las x publicaciones más recientes creadas en formato abreviado (fuente)

Tenemos que tratar con una situación similar aquí: incluso después de compartir las consultas adicionales dejadas como innecesarias por la desnormalización introducida en la versión 2, la consulta restante no se filtra por la clave de partición del contenedor:

:::image type="content" source="./media/how-to-model-partition-example/V2-Q6.png" alt-text="Recuperación de las publicaciones más recientes" border="false":::

Siguiendo el mismo enfoque, la maximización del rendimiento y escalabilidad de esta solicitud requiere que solo acceda a una partición. Esto es concebible porque solo es necesario devolver un número de elementos limitado; con el fin de rellenar la página principal de nuestra plataforma de blogs, solo debemos obtener las cien publicaciones más recientes, sin necesidad de paginar en todo el conjunto de datos.

Por lo que para optimizar esta última solicitud, se introduce un tercer contenedor en nuestro diseño, completamente dedicado a atender esta solicitud. Desnormalizamos nuestras publicaciones en ese nuevo contenedor `feed`:

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

Las particiones de este contenedor las crea `type`, que siempre será `post` en nuestros elementos. Eso garantiza que todos los elementos de este contenedor se encontrarán en la misma partición.

Para lograr la desnormalización, solo tenemos que enlazar a la canalización de la fuente de cambios que hemos introducido anteriormente para enviar las publicaciones a ese nuevo contenedor. Hay algo importante que se debe tener en cuenta, que necesitamos asegurarnos de que solo almacenamos las 100 publicaciones más recientes; de lo contrario, el contenido del contenedor puede crecer más allá del tamaño máximo de una partición. Para hacerlo, se realiza una llamada a un [desencadenador posterior ](stored-procedures-triggers-udfs.md#triggers) cada vez que se agrega un documento en el contenedor:

:::image type="content" source="./media/how-to-model-partition-example/denormalization-3.png" alt-text="Desnormalización de publicaciones del contenedor feed" border="false":::

Este es el cuerpo del desencadenador posterior que trunca la colección:

```javascript
function truncateFeed() {
  const maxDocs = 100;
  var context = getContext();
  var collection = context.getCollection();

  collection.queryDocuments(
    collection.getSelfLink(),
    "SELECT VALUE COUNT(1) FROM f",
    function (err, results) {
      if (err) throw err;

      processCountResults(results);
    });

  function processCountResults(results) {
    // + 1 because the query didn't count the newly inserted doc
    if ((results[0] + 1) > maxDocs) {
      var docsToRemove = results[0] + 1 - maxDocs;
      collection.queryDocuments(
        collection.getSelfLink(),
        `SELECT TOP ${docsToRemove} * FROM f ORDER BY f.creationDate`,
        function (err, results) {
          if (err) throw err;

          processDocsToRemove(results, 0);
        });
    }
  }

  function processDocsToRemove(results, index) {
    var doc = results[index];
    if (doc) {
      collection.deleteDocument(
        doc._self,
        function (err) {
          if (err) throw err;

          processDocsToRemove(results, index + 1);
        });
    }
  }
}
```

El último paso es para volver a enrutar nuestra consulta a nuestro nuevo contenedor `feed`:

:::image type="content" source="./media/how-to-model-partition-example/V3-Q6.png" alt-text="Recuperación de las publicaciones más recientes" border="false":::

| **Latency** | **Carga de unidad de solicitud** | **Rendimiento** |
| --- | --- | --- |
| 9 ms | 16,97 RU | ✅ |

## <a name="conclusion"></a>Conclusión

Echemos un vistazo a las mejoras en el rendimiento y escalabilidad generales que hemos introducido en las distintas versiones de nuestro diseño.

| | V1 | V2 | V3 |
| --- | --- | --- | --- |
| **[C1]** | 7 ms/5,71 RU | 7 ms/5,71 RU | 7 ms/5,71 RU |
| **[Q1]** | 2 ms/1 RU | 2 ms/1 RU | 2 ms/1 RU |
| **[C2]** | 9 ms/8,76 RU | 9 ms/8,76 RU | 9 ms/8,76 RU |
| **[Q2]** | 9 ms/19,54 RU | 2 ms/1 RU | 2 ms/1 RU |
| **[Q3]** | 130 ms/619,41 RU | 28 ms/201,54 RU | 4 ms/6,46 RU |
| **[C3]** | 7 ms/8.57 RU | 7 ms/15,27 RU | 7 ms/15,27 RU |
| **[Q4]** | 23 ms/27,72 RU | 4 ms/7,72 RU | 4 ms/7,72 RU |
| **[C4]** | 6 ms/7,05 RU | 7 ms/14,67 RU | 7 ms/14,67 RU |
| **[Q5]** | 59 ms/58,92 RU | 4 ms/8,92 RU | 4 ms/8,92 RU |
| **[Q6]** | 306 ms/2063,54 RU | 83 ms/532,33 RU | 9 ms/16,97 RU |

### <a name="we-have-optimized-a-read-heavy-scenario"></a>Hemos optimizado un escenario en el que se realizan muchas lecturas

Es posible que haya observado que hemos concentrado nuestros esfuerzos en mejorar el rendimiento de las solicitudes de lectura (consultas) a costa de las solicitudes de escritura (comandos). En muchos casos, las operaciones de escritura desencadenan una desnormalización posteriores a través de fuentes de cambios, lo que hace que requieran más procesos computacionales y tarden más tiempo en materializarse.

Esto se justifica por el hecho de que en una plataforma de blogs (como la mayoría de plataformas sociales) se realiza mucha lectura, lo que significa que la cantidad de solicitudes de lectura que tiene que atender tener una magnitud mayor que la cantidad de solicitudes de escritura. Por consiguiente, tiene sentido realizar solicitudes de escritura cuya ejecución sea más costosas, con el fin de que las solicitudes de lectura sean mejores y más baratas.

Si examinamos la optimización más extrema que hemos realizado, **[P6]** pasó de más de 2000 RU a solo 17 RU, la hemos logrado mediante la desnormalización de publicaciones con un costo de aproximadamente 10 RU por elemento. Como se atenderían muchas más solicitudes de fuentes que de creación o actualizaciones de publicaciones, el costo de esta desnormalización es nimio, si se tiene en cuenta el ahorro general.

### <a name="denormalization-can-be-applied-incrementally"></a>La desnormalización se puede aplicar de forma incremental

Las mejoras de escalabilidad que se han analizado en este implican la desnormalización y duplicación de datos en el conjunto de datos. Debe tenerse en cuenta que estas optimizaciones no necesariamente deben entrar en vigor el día 1. Las consultas que filtran por claves de partición funcionan mejor a escala, pero las consultas entre particiones pueden ser totalmente aceptables si se llama muy de vez en cuando o en un conjunto de datos limitado. Si solo va a crear un prototipo o a lanzar un producto con una base de usuarios pequeña y controlada, es probable que pueda dejar dichas mejoras para más adelante; lo importante es [supervisar](use-metrics.md) el rendimiento del modelo para que pueda decidir si hay que incorporarlas y cuándo hay que hacerlo.

La fuente de cambios que usamos para distribuir las actualizaciones a otros contenedores almacena todas las actualizaciones sistemáticamente. Esto permite solicitar todas las actualizaciones desde la creación del contenedor y arrancar las vistas desnormalizadas como una operación de puesta al día que se realiza una sola vez, aunque el sistema ya tenga muchos datos.

## <a name="next-steps"></a>Pasos siguientes

Después de esta introducción práctica al modelado de datos y a la creación de particiones, es posible que desee consultar los artículos siguientes para revisar los conceptos que hemos tratado:

- [Uso de bases de datos, contenedores y elementos](databases-containers-items.md)
- [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
- [Fuente de cambios en Azure Cosmos DB](change-feed.md)
