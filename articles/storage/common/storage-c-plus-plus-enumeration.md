---
title: Enumeración de los recursos de Azure Storage con la biblioteca cliente de C++
description: Obtenga información acerca de cómo usar las API de enumeración en la biblioteca de cliente de Microsoft Azure Storage para C++ para enumerar los contenedores, blobs, colas, tablas y entidades.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 01/23/2017
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.reviewer: dineshm
ms.openlocfilehash: b9ae42bb29d1273e4f0f9c25803e8cc97b56ad95
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462418"
---
# <a name="list-azure-storage-resources-in-c"></a>Enumeración de los recursos de Azure Storage en C++

Las operaciones de enumeración son clave para muchos escenarios de desarrollo con Azure Storage. En este artículo se describe cómo enumerar los objetos de Azure Storage de manera eficaz con las API de enumeración proporcionadas en la biblioteca de cliente de Microsoft Azure Storage para C++.

> [!NOTE]
> Esta guía tiene como destino la biblioteca de cliente de Azure Storage para C++ versión 2.x, que está disponible a través de [NuGet](https://www.nuget.org/packages/wastorage) o [GitHub](https://github.com/Azure/azure-storage-cpp).

La biblioteca de cliente de almacenamiento proporciona una variedad de métodos para enumerar o consultar objetos en Azure Storage. En este artículo se tratan los siguientes escenarios:

* Enumerar contenedores de una cuenta
* Enumerar blobs de un contenedor o un directorio virtual de blobs
* Enumerar colas de una cuenta
* Enumerar tablas en una cuenta
* Query Entities de una tabla

Cada uno de estos métodos se muestra con diferentes sobrecargas para diferentes escenarios.

## <a name="asynchronous-versus-synchronous"></a>Asincrónica frente sincrónica

Puesto que la biblioteca de cliente de almacenamiento para C++ está integrada en la [biblioteca de REST de C++](https://github.com/Microsoft/cpprestsdk), admitimos inherentemente operaciones asincrónicas usando [pplx::task](https://microsoft.github.io/cpprestsdk/classpplx_1_1task.html). Por ejemplo:

```cpp
pplx::task<list_blob_item_segment> list_blobs_segmented_async(continuation_token& token) const;
```

Las operaciones sincrónicas contienen las operaciones asincrónicas correspondientes:

```cpp
list_blob_item_segment list_blobs_segmented(const continuation_token& token) const
{
    return list_blobs_segmented_async(token).get();
}
```

Si está trabajando con varias aplicaciones o servicios de subprocesos, es recomendable usar las API asincrónicas directamente en lugar de crear un subproceso para llamar a las API sincrónicas, lo que afecta de forma significativa a su rendimiento.

## <a name="segmented-listing"></a>Enumeración segmentada

El escalamiento del almacenamiento en la nube requiere la enumeración segmentada. Por ejemplo, puede tener más de un millón de blobs en un contenedor de blobs de Azure o más de mil millones de entidades en una tabla de Azure. Estos no son números teóricos, sino casos de uso reales de clientes.

Por lo tanto, no resulta práctico enumerar todos los objetos en una sola respuesta. En su lugar, puede enumerar objetos mediante la paginación. Cada una de las API de enumeración tienen una sobrecarga *segmentada* .

La respuesta para una operación de enumeración segmentada incluye:

* *_segment*, que contiene el conjunto de resultados devueltos para una única llamada a la API de enumeración.
* *continuation_token*, que se pasa a la siguiente llamada con el fin de obtener la siguiente página de resultados. Cuando no hay más resultados para devolver, el token de continuación es nulo.

Por ejemplo, es posible que una llamada típica para enumerar todos los blobs de un contenedor tenga un aspecto similar al siguiente fragmento de código. El código está disponible en nuestros [ejemplos](https://github.com/Azure/azure-storage-cpp/blob/master/Microsoft.WindowsAzure.Storage/samples/BlobsGettingStarted.cpp):

```cpp
// List blobs in the blob container
azure::storage::continuation_token token;
do
{
    azure::storage::list_blob_item_segment segment = container.list_blobs_segmented(token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}

    token = segment.continuation_token();
}
while (!token.empty());
```

Tenga en cuenta que el número de resultados devueltos en una página puede controlarse mediante el parámetro *max_results* en la sobrecarga de cada API, por ejemplo:

```cpp
list_blob_item_segment list_blobs_segmented(const utility::string_t& prefix, bool use_flat_blob_listing,
    blob_listing_details::values includes, int max_results, const continuation_token& token,
    const blob_request_options& options, operation_context context)
```

Si no se especifica el parámetro *max_results*, se devolverá el valor máximo predeterminado de hasta 5000 resultados en una sola página.

Tenga en cuenta también que una consulta en Azure Table Storage puede no devolver ningún registro o menos registros que el valor del parámetro *max_results* especificado, incluso si el token de continuación no está vacío. Una razón podría ser que la consulta no se pudo completar en cinco segundos. Siempre y cuando el token de continuación no esté vacío, la consulta debe continuar y el código no debe suponer el tamaño del resultado del segmento.

El patrón de codificación recomendado para la mayoría de los escenarios es la enumeración segmentada, que proporciona información de progreso explícita de la enumeración o la consulta y sobre cómo responde el servicio a cada solicitud. Especialmente para las aplicaciones o servicios de C++, es posible que el control de bajo nivel de la enumeración ayude a controlar la memoria y el rendimiento.

## <a name="greedy-listing"></a>Enumeración expansiva

Las versiones anteriores de la biblioteca de cliente de almacenamiento de C++ (versiones 0.5.0 de vista previa y versiones anteriores) incluyen API de enumeración no segmentadas para las tablas y colas, como en el ejemplo siguiente:

```cpp
std::vector<cloud_table> list_tables(const utility::string_t& prefix) const;
std::vector<table_entity> execute_query(const table_query& query) const;
std::vector<cloud_queue> list_queues() const;
```

Estos métodos se implementaron como contenedores de API segmentadas. Para cada respuesta de la enumeración segmentada, el código anexa los resultados a un vector y devuelve todos los resultados después de analizar los contenedores completos.

Este enfoque puede funcionar si la cuenta de almacenamiento o la tabla contiene un pequeño número de objetos. Sin embargo, con un aumento en el número de objetos, la memoria necesaria podría aumentar sin límite, porque todos los resultados permanecen en memoria. Una operación de enumeración puede tardar mucho tiempo, durante el cual el autor de la llamada no tenía ninguna información sobre su progreso.

Estas API enumeración expansiva del SDK no existen en C#, Java o el entorno de JavaScript Node.js. Para evitar los posibles problemas con el uso de estas API expansivas, las hemos eliminado en la versión 0.6.0 de vista previa.

Si el código llama a estas API expansivas:

```cpp
std::vector<azure::storage::table_entity> entities = table.execute_query(query);
for (auto it = entities.cbegin(); it != entities.cend(); ++it)
{
    process_entity(*it);
}
```

Deberá modificar el código para usar las API de enumeración segmentada:

```cpp
azure::storage::continuation_token token;
do
{
    azure::storage::table_query_segment segment = table.execute_query_segmented(query, token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
    {
        process_entity(*it);
    }

    token = segment.continuation_token();
} while (!token.empty());
```

Mediante la especificación del parámetro *max_results* del segmento, puede obtener equilibrio entre los números de solicitudes y el uso de memoria para cumplir con las consideraciones de rendimiento de la aplicación.

Además, si usa API de enumeración segmentadas, pero almacena los datos en una colección local de manera "expansiva", también es altamente recomendable refactorizar el código para controlar el almacenamiento de datos en una colección local cuidadosamente a escala.

## <a name="lazy-listing"></a>Enumeración diferida

Aunque la enumeración expansiva produjo posibles problemas, es recomendable si no hay demasiados objetos en el contenedor.

Si también SDK de Java de Oracle o C#, debería estar familiarizado con el modelo de programación Enumerable, que ofrece una enumeración de estilo diferido, en la que los datos con un determinado desplazamiento solo se capturan si es necesario. En C++, la plantilla de iterador también proporciona un enfoque similar.

Una API de enumeración diferida normal, con **list_blobs** como ejemplo, tiene el siguiente aspecto:

```cpp
list_blob_item_iterator list_blobs() const;
```

Un fragmento de código típico que usa el patrón de lista diferida podría tener este aspecto:

```cpp
// List blobs in the blob container
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}
```

Tenga en cuenta que la enumeración diferida solo está disponible en modo sincrónico.

En comparación con la enumeración expansiva, la diferida captura los datos solo cuando es necesario. En segundo plano, recupera los datos de Azure Storage solo cuando se mueve el iterador siguiente al siguiente segmento. Por lo tanto, el uso de la memoria se controla con un tamaño límitado y la operación es rápida.

Las API de enumeración diferida se incluyen en la biblioteca de cliente de almacenamiento de C++ en la versión 2.2.0.

## <a name="conclusion"></a>Conclusión

En este artículo, hemos tratado diferentes sobrecargas para API de enumeración de varios objetos de la biblioteca de cliente de almacenamiento de C++. Resumiendo:

* Se recomiendan encarecidamente las API asincrónicas en escenarios de varios subprocesos.
* La enumeración segmentada es recomendable para la mayoría de los escenarios.
* La enumeración diferida se proporciona en la biblioteca como un contenedor conveniente en escenarios sincrónicos.
* La enumeración expansiva no se recomienda y se ha eliminado de la biblioteca.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Azure Storage y la biblioteca de cliente de C++, consulte los siguientes recursos.

* [Cómo usar Blob Storage de C++](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [Cómo usar Table Storage de C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [Cómo usar Queue Storage de C++](../storage-c-plus-plus-how-to-use-queues.md)
* [Documentación de la Biblioteca de cliente de Azure Storage para la API de C++.](https://azure.github.io/azure-storage-cpp/)
* [Blog del equipo de Azure Storage](https://docs.microsoft.com/archive/blogs/windowsazurestorage/)
* [Documentación de Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
