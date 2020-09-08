---
title: Introducción a la compatibilidad integrada con cuadernos de Jupyter Notebook en Azure Cosmos DB (versión preliminar)
description: Aprenda a usar la compatibilidad integrada con los cuadernos de Jupyter Notebook en Azure Cosmos DB para ejecutar consultas de forma interactiva.
ms.service: cosmos-db
ms.topic: overview
ms.date: 05/19/2020
author: deborahc
ms.author: dech
ms.openlocfilehash: 0299ab1e76e742adb8d2653b8b5d6923aff2f11f
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588163"
---
# <a name="built-in-jupyter-notebooks-support-in-azure-cosmos-db-preview"></a>Compatibilidad integrada con cuadernos de Jupyter Notebook en Azure Cosmos DB (versión preliminar)

Jupyter Notebook es una aplicación web de código abierto que le permite crear y compartir documentos que contienen código, ecuaciones, visualizaciones y texto narrativo dinámicos. 

Los cuadernos de Jupyter Notebook integrados en Azure Cosmos DB se integran directamente en Azure Portal y en las cuentas de Azure Cosmos DB, lo que hace que sean cómodos y fáciles de usar. Los desarrolladores, científicos de datos, ingenieros y analistas pueden usar la conocida experiencia de los cuadernos de Jupyter Notebook para explorar, limpiar, visualizar y transformar los datos, así como para simulaciones numéricas, modelado estadístico y aprendizaje automático.

:::image type="content" source="./media/cosmosdb-jupyter-notebooks/cosmos-notebooks-overview.png" alt-text="Visualizaciones de cuadernos de Jupyter Notebook en Azure Cosmos DB":::

Azure Cosmos DB admite cuadernos de C# y de Python para todas las API, lo que incluye Core (SQL), Cassandra, Gremlin, Table y API para MongoDB. Dentro del cuaderno, puede aprovechar las características y comandos integrados que facilitan la creación de recursos de Azure Cosmos DB, así como la carga, la consulta y la visualización de los datos en Azure Cosmos DB. 

:::image type="content" source="./media/cosmosdb-jupyter-notebooks/jupyter-notebooks-portal.png" alt-text="Compatibilidad con cuadernos de Jupyter Notebook en Azure Cosmos DB":::

## <a name="benefits-of-jupyter-notebooks"></a>Ventajas de los cuadernos de Jupyter Notebook

Los cuadernos de Jupyter Notebook se desarrollaron originalmente para aplicaciones de ciencia de datos escritas en Python y R. Sin embargo, se pueden usar de varias maneras para diferentes tipos de proyectos, como:

**Visualización de datos:** los cuadernos de Jupyter Notebook permiten visualizar los datos en forma de cuaderno compartido que representa algún conjunto de datos como un gráfico. Puede crear visualizaciones, realizar cambios interactivos en el código compartido y el conjunto de datos, y compartir los resultados.

**Uso compartido de código:** los servicios como GitHub proporcionan maneras de compartir el código, pero en su mayoría no son interactivos. Con un cuaderno de Jupyter Notebook, puede ver el código, ejecutarlo y mostrar los resultados directamente en Azure Portal.

**Interacciones activas con código:** el código de un cuaderno de Jupyter Notebook es dinámico; puede editarlo y ejecutar las actualizaciones de manera incremental en tiempo real. También puede insertar controles de usuario (por ejemplo, controles deslizantes o campos de entrada de texto) que se pueden usar como orígenes de entrada para código, demostraciones o prueba de conceptos (POC).

**Documentación de ejemplos de código y resultados de la exploración de datos:** si tiene un fragmento de código y quiere explicar línea a línea cómo funciona, puede insertarlo en un cuaderno de Jupyter. Puede agregar interactividad junto con la documentación al mismo tiempo.

**Comandos integrados para Azure Cosmos DB:** los comandos mágicos integrados de Azure Cosmos DB facilitan la interacción con su cuenta. Puede usar comandos como %%upload y %%sql para cargar datos en un contenedor y consultarlos mediante [sintaxis de la API de SQL](sql-query-getting-started.md). No es necesario escribir código personalizado adicional.

**Todo en un solo lugar:** Los cuadernos de Jupyter Notebook combinan código, texto enriquecido, imágenes, vídeos, animaciones, ecuaciones matemáticas, trazados, mapas, figuras interactivas, widgets e interfaces de usuario gráficas en un solo documento.

## <a name="components-of-a-jupyter-notebook"></a>Componentes de un cuaderno de Jupyter Notebook

Los cuadernos de Jupyter Notebook pueden incluir varios tipos de componentes y cada uno de ellos se organizan en celdas o bloques discretos:

**Texto y HTML:** El texto sin formato, o texto anotado en la sintaxis de Markdown para generar HTML, se puede insertar en el documento en cualquier momento. Los estilos de CSS también pueden insertarse o agregarse a la plantilla que se usa para generar el cuaderno.

**Entrada y salida:** los cuadernos de Jupyter Notebook admiten código de Python y de C#. Los resultados del código ejecutado aparecen inmediatamente después de los bloques de código, y estos se pueden ejecutar varias veces en cualquier orden que desee.

**Visualizaciones:** puede generar gráficos a partir del código mediante módulos como Matplotlib, Plotly, Bokeh, etc. De forma similar a la salida, estas visualizaciones aparecen insertadas junto al código que las genera. De forma similar a la salida, estas visualizaciones aparecen insertadas junto al código que las genera.

**Multimedia:** como los cuadernos de Jupyter Notebook se basan en tecnología web, pueden mostrar todos los tipos de contenido multimedia que se admiten en una página web. Puede incluirlos en un cuaderno como elementos HTML o puede generarlos mediante programación con el módulo `IPython.display`.

**Datos:** puede importar los datos mediante programación desde contenedores de Azure Cosmos o los resultados de las consultas a un cuaderno de Jupyter Notebook. Use comandos mágicos integrados para cargar o consultar datos en Azure Cosmos DB. 

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con cuadernos de Jupyter Notebook integrados en Azure Cosmos DB, consulte los siguientes artículos:

* [Habilitación de cuadernos en una cuenta de Azure Cosmos](enable-notebooks.md)
* [Uso de características y comandos de cuadernos de Python](use-python-notebook-features-and-commands.md)
* [Uso de características y comandos de cuadernos de C#](use-csharp-notebook-features-and-commands.md)
