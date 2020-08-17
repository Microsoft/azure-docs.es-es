---
title: Conexión de orígenes de datos en Azure Data Catalog
description: Artículo de procedimientos que indica cómo conectarse a orígenes de datos que se detectan con Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 53a861761f819c2127228e7ef688f04d91744d25
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081260"
---
# <a name="how-to-connect-to-data-sources"></a>Conexión a orígenes de datos
## <a name="introduction"></a>Introducción
**Microsoft Azure Data Catalog** es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección de orígenes de datos empresariales. En otras palabras, **Azure Data Catalog** consiste en ayudar a las personas a detectar, comprender y usar orígenes de datos, y en ayudar a las organizaciones a obtener más valor de sus datos. Un aspecto clave de este escenario es el uso de los datos: cuando un usuario detecta un origen de datos y comprende su propósito, el paso siguiente es la conexión al origen de datos para utilizar sus datos.

## <a name="data-source-locations"></a>Ubicaciones de orígenes de datos
Durante el registro del origen de datos, el **Azure Data Catalog** recibe los metadatos sobre el origen de datos. Estos metadatos incluyen los detalles de la ubicación del origen de datos. Los detalles de la ubicación varían según el origen de datos, pero siempre contendrán la información necesaria para conectarse. Por ejemplo, la ubicación de una tabla de SQL Server incluye el nombre del servidor, el nombre de la base de datos, el nombre del esquema y el nombre de la tabla; mientras que la ubicación de un informe de SQL Server Reporting Services incluye el nombre del servidor y la ruta de acceso al informe. Otros tipos de orígenes de datos tendrán ubicaciones que reflejan la estructura y las capacidades del sistema de origen.

## <a name="integrated-client-tools"></a>Herramientas de cliente integradas
La manera más sencilla de conectarse a un origen de datos es usar el menú "Abrir en…" del portal de **Azure Data Catalog**. En este menú se muestra una lista de opciones para conectarse al recurso de datos seleccionado.
Cuando se usa la vista de iconos predeterminada, este menú está disponible en cada icono.

 ![Apertura de una tabla de SQL Server en Excel desde el icono de recurso de datos](./media/data-catalog-how-to-connect/data-catalog-how-to-connect1.png)

Al utilizar la vista de lista, el menú está disponible en la barra de búsqueda de la parte superior de la ventana del portal.

 ![Apertura de un informe de SQL Server Reporting Services en el Administrador de informes](./media/data-catalog-how-to-connect/data-catalog-how-to-connect2.png)

## <a name="supported-client-applications"></a>Aplicaciones de cliente admitidas
Si se usa el menú "Abrir en…" en los orígenes de datos del portal de Azure Data Catalog, debe instalarse la aplicación de cliente correcta en el equipo cliente.

| Abrir en la aplicación | Extensión de archivo o protocolo | Versiones de aplicación admitidas |
| --- | --- | --- |
| Excel |.odc |Excel 2010 o posterior |
| Excel (primeros 1000) |.odc |Excel 2010 o posterior |
| Power Query |.xlsx |Excel 2016 o Excel 2010 o Excel 2013 con el complemento de Power Query para Excel instalado |
| Power BI Desktop |.pbix |Power BI Desktop de julio de 2016 o posterior |
| SQL Server Data Tools |vsweb:// |Visual Studio 2013 Update 4 o posterior con las herramientas de SQL Server instaladas |
| Administrador de informes |http:// |Consulte los [requisitos del explorador para SQL Server Reporting Services](https://technet.microsoft.com/library/ms156511.aspx) |

## <a name="your-data-your-tools"></a>Sus propios datos y herramientas
Las opciones disponibles en el menú dependerán del tipo de recurso de datos actualmente seleccionado. Por supuesto, en el menú "Abrir en…" no se incluirán todas las herramientas posibles, pero sigue siendo fácil conectarse al origen de datos mediante cualquier herramienta de cliente. Cuando se selecciona un recurso de datos en el portal de **Azure Data Catalog**, se muestra la ubicación completa en el panel de propiedades.

 ![Información de conexión de una tabla de SQL Server](./media/data-catalog-how-to-connect/data-catalog-how-to-connect3.png)

Los detalles de la información de conexión variarán según el tipo de origen de datos, pero la información incluida en el portal le proporcionará todo lo que necesita para conectarse al origen de datos en cualquier herramienta de cliente. Los usuarios pueden copiar los detalles de conexión de los orígenes de datos que hayan detectado con el **Azure Data Catalog**, lo que les permite trabajar con los datos en su herramienta preferida.

## <a name="connecting-and-data-source-permissions"></a>Permisos de origen de datos y conexión
Aunque **Azure Data Catalog** permite que los orígenes de datos sean detectables, el control del acceso a los propios datos sigue siendo responsabilidad del administrador o propietario del origen de datos. La detección de un origen de datos en **Azure Data Catalog** no otorga al usuario ningún permiso para tener acceso al propio origen de datos.

Con el objetivo de simplificar las cosas para los usuarios que detecten un origen de datos, pero no tengan permiso para obtener acceso a sus datos, dichos usuarios pueden proporcionar información en la propiedad de solicitud de acceso al anotar un origen de datos. La información que se incluye aquí (incluidos los vínculos al proceso o el punto de contacto para obtener acceso a los datos de origen) se presenta junto con la información de ubicación del origen de datos en el portal.

 ![Información de conexión con las instrucciones de solicitud de acceso proporcionadas](./media/data-catalog-how-to-connect/data-catalog-how-to-connect4.png)

## <a name="summary"></a>Resumen
Al registrar un origen de datos con **Azure Data Catalog**, se consigue que esos datos sean detectables mediante la copia de los metadatos descriptivos y estructurales del origen de datos en el servicio Catalog. Una vez que se registra y detecta un origen de datos, los usuarios pueden conectarse al origen de datos desde el menú "Abrir en…" del portal de **Azure Data Catalog** o con las herramientas de datos que prefieran.

## <a name="see-also"></a>Consulte también
* [Introducción a Azure Data Catalog](data-catalog-get-started.md) para obtener información paso a paso sobre cómo realizar una conexión a orígenes de datos.
