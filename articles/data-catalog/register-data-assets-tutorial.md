---
title: 'Tutorial: Registro de recursos de datos en Azure Data Catalog'
description: Este tutorial describe cómo registrar recursos de datos en Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: tutorial
ms.date: 08/01/2019
ms.openlocfilehash: 66bacdf27b2ef9cf624b645cb6e4c123d9c3fa2a
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88135885"
---
# <a name="tutorial-register-data-assets-in-azure-data-catalog"></a>Tutorial: Registro de recursos de datos en Azure Data Catalog

En este tutorial, se usa la herramienta de registro para registrar recursos de datos de la base de datos de ejemplo con el catálogo. El registro es el proceso de extraer metadatos estructurales clave (como nombres, tipos y ubicaciones) del origen de datos y los recursos que contiene, y copiar dichos metadatos en el catálogo. El origen de datos y los recursos de datos permanecen donde están, pero el catálogo usa los metadatos para que se puedan detectar y comprender más fácilmente.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Registro de los recursos de datos 
> * Buscar recursos de datos
> * Anotación de los recursos de datos
> * Conexión con los recursos de datos
> * Administración de recursos de datos
> * Eliminar recursos de datos

## <a name="prerequisites"></a>Requisitos previos

Para empezar, debe completar el [inicio rápido](register-data-assets-tutorial.md).

* Una suscripción de [Microsoft Azure](https://azure.microsoft.com/).
* Debe tener su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

Para configurar Data Catalog, debe ser propietario o copropietario de una suscripción de Azure.

## <a name="register-data-assets"></a>Registro de los recursos de datos

### <a name="register-a-data-source"></a>Registro de un origen de datos

Los recursos de datos (tablas) que se registran son de una [base de datos de ejemplo](../azure-sql/database/single-database-create-quickstart.md) para Azure SQL Database, pero puede usar cualquier origen de datos compatible si prefiere trabajar con datos que conozca y que sean relevantes para su rol. Para ver una lista de los orígenes de datos compatibles, consulte [Orígenes de datos compatibles con Azure Data Catalog](data-catalog-dsr.md).

El nombre de la base de datos que se usa en este tutorial es *RLSTest*.

Ahora puede registrar recursos de datos de la base de datos ejemplo mediante Azure Data Catalog.

1. Vaya a la [página principal de Azure Data Catalog](http://azuredatacatalog.com) y seleccione **Publicar datos**.

   ![Azure Data Catalog: botón Publicar datos](media/register-data-assets-tutorial/data-catalog-publish-data.png)

2. Seleccione **Iniciar la aplicación** para descargar, instalar y ejecutar la herramienta de registro en el equipo.

   ![Azure Data Catalog: botón Iniciar](media/register-data-assets-tutorial/data-catalog-launch-application.png)

3. En la página **principal**, seleccione **Iniciar sesión** y escriba sus credenciales.

    ![Azure Data Catalog: página principal](media/register-data-assets-tutorial/data-catalog-welcome-dialog.png)

4. En la página **Microsoft Azure Data Catalog**, seleccione **SQL Server** y **Siguiente**.

    ![Azure Data Catalog: orígenes de datos](media/register-data-assets-tutorial/data-catalog-data-sources.png)

5. Especifique las propiedades de la conexión de SQL Server de la base de datos de ejemplo de Azure SQL Database y seleccione **CONNECT**.

   ![Azure Data Catalog: configuración de conexión con SQL Server](media/register-data-assets-tutorial/data-catalog-sql-server-connection.png)

6. Registre los metadatos del recurso de datos. En este ejemplo, se registran los objetos **Product** del espacio de nombres de ejemplo:

   1. En el árbol **Jerarquía de servidor**, expanda la base de datos de ejemplo y seleccione **SalesLT**.

   2. Seleccione **Product**, **ProductCategory**, **ProductDescription** y **ProductModel** mediante Ctrl + seleccionar.

   3. Seleccione la **flecha Mover seleccionados** ( **>** ). Esta acción mueve todos los objetos seleccionados a la lista **Objetos que se registrarán** .

      ![Azure Data Catalog: examen o selección de objetos](media/register-data-assets-tutorial/data-catalog-server-hierarchy.png)

   4. Seleccione **Include a Preview** (Incluir vista previa) para incluir una vista previa de instantánea de los datos. La instantánea incluye un máximo de 20 registros de cada tabla y se copia en el catálogo.

   5. Seleccione **Incluir perfil de datos** para incluir una instantánea de las estadísticas del objeto del perfil de datos (por ejemplo: los valores mínimo, máximo y medio de una columna, o el número de filas).

   6. En el campo **Agregar etiquetas**, escriba **ventas, producto, azure sql**. Esta acción agrega etiquetas de búsqueda a estos recursos de datos. Las etiquetas son una excelente manera de ayudar a los usuarios a buscar un origen de datos registrados.

   7. Especifique el nombre de un **experto** en estos datos (opcional).

      ![Azure Data Catalog: objetos que se registran](media/register-data-assets-tutorial/data-catalog-objects-register.png)

   8. Seleccione **REGISTRAR**. Azure Data Catalog registra los objetos seleccionados. En este ejercicio, se registran los objetos seleccionados de la base de datos de ejemplo. La herramienta de registro extrae metadatos del recurso de datos y los copia en el servicio Azure Data Catalog. Los datos permanecen donde están actualmente. Los datos permanecen bajo el control de los administradores y directivas del sistema de origen.

      ![Azure Data Catalog: objetos registrados](media/register-data-assets-tutorial/data-catalog-registered-objects.png)

    9. Para ver los objetos de origen de datos registrados, seleccione **Ver portal**. En el portal de Azure Data Catalog, confirme que ve las cuatro tablas y la base de datos en la vista de cuadrícula (compruebe que la barra de búsqueda ha desaparecido).

       ![Objetos del portal de Azure Data Catalog](media/register-data-assets-tutorial/data-catalog-view-portal.png)

En este ejercicio, ha registrado objetos de la base de datos de ejemplo para Azure SQL Database de forma que los usuarios de la organización puedan detectarlos fácilmente.

En el siguiente ejercicio aprenderá a detectar recursos de datos registrados.

## <a name="discover-data-assets"></a>Detección de los recursos de datos

La detección en Azure Data Catalog usa dos mecanismos principales: la búsqueda y el filtrado.

La búsqueda está diseñada para ser intuitiva y eficaz. De forma predeterminada, los términos de la búsqueda se comparan todas las propiedades del catálogo, incluidas las anotaciones proporcionadas por el usuario.

El filtrado está diseñado para complementar la búsqueda. Puede seleccionar características específicas como expertos, tipos de origen de datos, tipos de objeto y etiquetas, para ver los recursos de datos que coincidan y para restringir los resultados de la búsqueda a los recursos correspondientes.

Mediante una combinación de búsqueda y filtrado puede desplazarse rápidamente por los orígenes de datos registrados en Azure Data Catalog.

En este ejercicio se usa el portal de Azure Data Catalog para detectar los recursos de datos que registró en el ejercicio anterior. Para más información acerca de la sintaxis de búsqueda, consulte [Referencia de sintaxis de búsqueda en Data Catalog](/rest/api/datacatalog/#search-syntax-reference) .

A continuación hay varios ejemplos de detección de recursos de datos en el catálogo.  

### <a name="discover-data-assets-with-basic-search"></a>Detección de recursos de datos con la búsqueda básica

La búsqueda básica permite buscar en un catálogo con uno o varios términos de búsqueda. Los resultados son los recursos que tienen alguna propiedad que coincide con uno o varios de los términos especificados.

1. Seleccione **Inicio** en el portal de Azure Data Catalog. Si ha cerrado el explorador web, vaya a la [página principal de Azure Data Catalog](https://www.azuredatacatalog.com).

2. En el cuadro de búsqueda, escriba `product` y presione **ENTRAR**.

   ![Azure Data Catalog: búsqueda de texto básica](media/register-data-assets-tutorial/data-catalog-basic-text-search.png)

3. Confirme que ve las cuatro tablas y la base de datos en los resultados. Para alternar entre la **vista de cuadrícula** y la **vista de lista**, seleccione los botones de la barra de herramientas, como se muestra en la siguiente imagen. Observe que la palabra clave de la búsqueda aparece resaltada en los resultados de la búsqueda, ya que la opción **Resaltar** está **activada**. También puede especificar el número de **resultados por página** .

   ![Azure Data Catalog: resultados de búsqueda de texto básica](media/register-data-assets-tutorial/data-catalog-basic-text-search-results.png)

   El panel **Búsquedas** está a la izquierda y el panel **Propiedades**, a la derecha. En panel **Búsquedas** se pueden cambiar los criterios de búsqueda y filtrar los resultados. El panel **Propiedades** muestra las propiedades de un objeto seleccionado en la cuadrícula o en la lista.

4. Seleccione **Producto** en los resultados de la búsqueda. Seleccione las pestañas **Vista previa**, **Columnas**, **Perfil de datos** y **Documentación**, o bien la flecha para expandir el panel inferior.  

   ![Azure Data Catalog: panel inferior](media/register-data-assets-tutorial/data-catalog-data-asset-preview.png)

   En la pestaña **Vista previa**, se muestra una vista previa de los datos de la tabla **Product**.

5. Seleccione la pestaña **Columnas** para buscar detalles de las columnas (como el **nombre** y el **tipo de datos**) en el recurso de datos.

6. Seleccione la pestaña **Perfil de datos** para ver la generación de perfiles de los datos (por ejemplo: el número de filas, el tamaño de los datos o el valor mínimo de una columna) del recurso de datos.

### <a name="discover-data-assets-with-property-scoping"></a>Detección de recursos de datos con ámbito de propiedad

El ámbito de una propiedad le ayuda a detectar recursos de datos en los que el término de búsqueda coincide con la propiedad especificada.

1. Desactive el filtro **Tabla** en **Tipo de objeto** en **Filtros**.  

2. En el cuadro de búsqueda, escriba `tags:product` y presione **ENTRAR**. En [Referencia de sintaxis de búsqueda en Data Catalog](/rest/api/datacatalog/#search-syntax-reference) encontrará todas las propiedades que se pueden usar en las búsquedas en el catálogo de datos.

3. Confirme que ve las tablas y la base de datos en los resultados.  

   ![Catálogo de datos: resultados de búsqueda de ámbito de propiedad](media/register-data-assets-tutorial/data-catalog-property-scoping-results.png)

### <a name="save-the-search"></a>Guardado de una búsqueda

1. En el panel **Búsquedas** de la sección **Búsqueda actual**, escriba el nombre de la búsqueda y seleccione **Guardar**.

   ![Azure Data Catalog: guardado de búsqueda](media/register-data-assets-tutorial/data-catalog-save-search.png)

2. Confirme que la búsqueda guardada se muestra en **Búsquedas guardadas**.

3. Seleccione una de las acciones que puede realizar en la búsqueda guardada (**Cambiar nombre**, **Eliminar**, **Guardar como predeterminado**).

### <a name="grouping-with-parentheses"></a>Agrupación con paréntesis

Mediante los paréntesis se pueden agrupar partes de la consulta para conseguir el aislamiento lógico, especialmente junto con los operadores booleanos.

1. En el cuadro de búsqueda, escriba `name:product AND (tags:product AND objectType:table)` y presione **ENTRAR**.

2. Confirme que en los resultados de la búsqueda solo ve la tabla **Product** .

   ![Azure Data Catalog: búsqueda con agrupación](media/register-data-assets-tutorial/data-catalog-grouping-search.png)

### <a name="comparison-operators"></a>Operadores de comparación

Con los operadores de comparación puede usar comparaciones diferentes de la igualdad de propiedades que tengan tipos de datos numéricos y de fechas.

1. En el cuadro de búsqueda, escriba `lastRegisteredTime:>"06/09/2016"`.

2. Desactive el filtro **Tabla** en **Tipo de objeto**.

3. Presione **ENTRAR**.

4. Confirme que ve las tablas **Product**, **ProductCategory** y **ProductDescription** y la base de datos de SQL que registró en los resultados de la búsqueda.

   ![Azure Data Catalog: resultados de búsqueda con comparación](media/register-data-assets-tutorial/data-catalog-comparison-operator-results.png)

Consulte [Detección de orígenes de datos en Azure Data Catalog](data-catalog-how-to-discover.md) para ver información detallada acerca de la detección de recursos de datos. Para más información acerca de la sintaxis de búsqueda, consulte la [referencia de la sintaxis de búsqueda en Data Catalog](/rest/api/datacatalog/#search-syntax-reference).

## <a name="annotate-data-assets"></a>Anotación de los recursos de datos

En este ejercicio, usará el portal de Azure Data Catalog para anotar (agregar información como descripciones, etiquetas o expertos) los recursos de datos existentes en el catálogo. Las anotaciones complementan los metadatos estructurales extraídos del origen de datos durante el registro. También facilitan la detección y comprensión de los recursos de datos.

En este ejercicio, se anota un único recurso de datos (ProductPhoto). y se agrega un nombre descriptivo y una descripción al recurso de datos ProductPhoto.  

1. Vaya a la [página principal de Azure Data Catalog](https://www.azuredatacatalog.com) y busque con `tags:product` para encontrar los recursos de datos que ha registrado.

2. Seleccione **ProductModel** en los resultados de la búsqueda.  

3. Escriba **Imágenes de productos** en **Nombre descriptivo** y **Fotos de productos para materiales de marketing** en **Descripción**.

    ![Azure Data Catalog: descripción de ProductPhoto](media/register-data-assets-tutorial/data-catalog-productmodel-description.png)

    El contenido del campo **Descripción** ayuda a otros usuarios a detectar el recurso de datos seleccionado y entender por qué y cómo se usa. También se pueden agregar otras etiquetas y ver columnas. Para filtrar y realizar búsquedas en los orígenes de datos, utilice los metadatos descriptivos que ha agregado al catálogo.

También puede dar los siguientes pasos en esta página:

* Agregar a expertos para el recurso de datos. Seleccione **Agregar** en el área **Expertos**.

* Agregar etiquetas en el nivel de conjunto de datos. Seleccione **Agregar** en el área **Etiquetas**. Una etiqueta puede ser una etiqueta de usuario o una etiqueta de glosario. La edición estándar del Catálogo de datos incluye un glosario empresarial que ayuda a los administradores del catálogo a definir una taxonomía empresarial central. Después, los usuarios del catálogo pueden anotar los recursos de datos con los términos del glosario. Para más información, consulte [Configuración del glosario empresarial para el etiquetado regulado](data-catalog-how-to-business-glossary.md)

* Agregar etiquetas a nivel de columna. Seleccione **Agregar** en **Etiquetas** en la columna en la que desee realizar anotaciones.

* Agregar descripción en el nivel de columna. En **Descripción** , escriba la descripción de la columna. También puede ver los metadatos de descripción extraídos del origen de datos.

* Agregue la información de **Solicitar acceso** que muestra a los usuarios cómo solicitar acceso al recurso de datos.
  
* Elija la pestaña **Documentación** y proporcione la documentación del recurso de datos. Con la documentación de Azure Data Catalog se puede usar un catálogo de datos como repositorio de contenido para crear una descripción completa de los recursos de datos.
  
También puede agregar una anotación a varios recursos de datos. Por ejemplo, puede seleccionar todos los recursos de datos que ha registrado y especificar un experto para ellos.

![Azure Data Catalog: anotación de varios recursos de datos](media/register-data-assets-tutorial/data-catalog-multi-select-annotate.png)

Azure Data Catalog admite un enfoque de colaboración abierta distribuida con respecto a las anotaciones. Todos los usuarios de Data Catalog pueden agregar etiquetas (usuario o glosario), descripciones y otros metadatos. Al hacerlo, los usuarios agregan perspectiva a un recurso de datos y su uso, y compartan dicha perspectiva con otros usuarios.

Consulte [Anotación de recursos de datos](data-catalog-how-to-annotate.md) para ver información detallada acerca de cómo anotar recursos de datos.

## <a name="connect-to-data-assets"></a>Conexión con los recursos de datos

En este ejercicio, se abren recursos de datos en una herramienta cliente integrada (Excel) y en una herramienta no integrada (SQL Server Management Studio) mediante la información de conexión.

> [!NOTE]
> Es importante recordar que Azure Data Catalog no brinda acceso al origen de datos en sí, simplemente facilita su detección y comprensión. Al conectarse a un origen de datos, la aplicación cliente que elija usa sus credenciales de Windows o le solicita las credenciales cuando sea necesario. Si no se le ha otorgado previamente acceso al origen de datos, será preciso que se le otorgue para que se pueda conectar.

### <a name="connect-to-a-data-asset-from-excel"></a>Conexión a un recurso de datos desde Excel

1. Seleccione **Product** en los resultados de la búsqueda. Seleccione **Abrir en** en la barra de herramientas y, después, seleccione **Excel**.

   ![Azure Data Catalog: conexión a recurso de datos](media/register-data-assets-tutorial/data-catalog-connect1.png)

2. Seleccione **Abrir** en la ventana emergente de descarga. Este proceso puede variar en función del explorador.

3. En la ventana **Aviso de seguridad de Microsoft Excel**, seleccione **Habilitar**.

   ![Azure Data Catalog: mensaje emergente de seguridad de Excel](media/register-data-assets-tutorial/data-catalog-excel-security-popup.png)

4. Mantenga los valores predeterminados del cuadro de diálogo **Importar datos** y seleccione **Aceptar**.

   ![Azure Data Catalog: datos de importación de Excel](media/register-data-assets-tutorial/data-catalog-excel-import-data.png)

5. Vea el origen de datos en Excel.

   ![Azure Data Catalog: tabla de producto en Excel](media/register-data-assets-tutorial/data-catalog-connect2.png)

### <a name="sql-server-management-studio"></a>SQL Server Management Studio

En este ejercicio se ha conectado a los recursos de datos detectados mediante Azure Data Catalog. Con el portal de Azure Data Catalog, puede conectarse directamente mediante las aplicaciones cliente integradas en el menú **Abrir en** . También puede conectarse con cualquier aplicación que elija mediante la información de la ubicación de la conexión incluida en los metadatos del recurso. Por ejemplo, puede usar SQL Server Management Studio para conectarse a Azure SQL Database para acceder a los datos de los recursos de datos registrados en este tutorial.

1. Abra **SQL Server Management Studio**.

2. En el cuadro de diálogo **Conectar con el servidor**, escriba el nombre del servidor del panel **Propiedades** del portal de Azure Data Catalog.

3. Use la autenticación y las credenciales apropiadas para acceder al recurso de datos. Si no tiene acceso, utilice la información del campo **Solicitar acceso** para obtenerla.

   ![Azure Data Catalog: solicitud de acceso](media/register-data-assets-tutorial/data-catalog-request-access.png)

Seleccione **Ver cadenas de conexión** para ver y copiar las cadenas de conexión ADO.NET, ODBC y OLEDB en el Portapapeles para utilizarlas en la aplicación.

## <a name="manage-data-assets"></a>Administración de recursos de datos

En este paso, verá cómo se configura la seguridad de los recursos de datos. Data Catalog no ofrece a los usuarios acceso a los propios datos. El propietario del origen de datos controla el acceso a los datos.

Catálogo de datos se puede usar para detectar los orígenes de datos y ver los metadatos relacionados con los orígenes registrados en el catálogo. Sin embargo, puede haber situaciones en las que los orígenes de datos solo deben verlos determinados usuarios o miembros de grupos concretos. En estos escenarios se puede usar Data Catalog para tomar propiedad de los recursos de datos registrados y controlar la visibilidad de los recursos que se poseen.

> [!NOTE]
> Las funcionalidades de administración descritas en este ejercicio solo están disponibles en la edición estándar de Azure Data Catalog, no en la edición gratuita.
> En Azure Data Catalog, puede tomar propiedad de los recursos de datos, agregar copropietarios a los recursos de datos y establecer la visibilidad de los recursos de datos.

### <a name="take-ownership-of-data-assets-and-restrict-visibility"></a>Toma de propiedad de los recursos de datos y restricción de la visibilidad

1. Vaya a la [página principal de Azure Data Catalog](https://www.azuredatacatalog.com). En el cuadro de texto **Buscar**, escriba `tags:cycles` y presione **ENTRAR**.

2. Seleccione un elemento de la lista de resultados y, después, seleccione **Asumir propiedad** en la barra de herramientas.

3. En la sección **Administración** del panel **Propiedades**, seleccione **Tomar posesión**.

    ![Azure Data Catalog: toma de propiedad](media/register-data-assets-tutorial/data-catalog-take-ownership.png)

4. Para restringir la visibilidad, elija **Propietarios y estos usuarios** en la sección **Visibilidad** y seleccione **Agregar**. Escriba las direcciones de correo electrónico de los usuarios en el cuadro de texto y presione **ENTRAR**.

    ![Azure Data Catalog: restricción de acceso](media/register-data-assets-tutorial/data-catalog-ownership.png)

## <a name="remove-data-assets"></a>Eliminación de los recursos de datos

En este ejercicio se usa el portal de Azure Data Catalog para quitar los datos de vista previa de los recursos de datos registrados y eliminar los recursos de datos del catálogo.

En Azure Data Catalog se pueden eliminar uno o varios recursos.

1. Vaya a la [página principal de Azure Data Catalog](https://www.azuredatacatalog.com).

2. En el cuadro de texto **Buscar**, escriba `tags:cycles` y seleccione **ENTRAR**.

3. Seleccione un elemento de la lista de resultados y, después, seleccione **Eliminar** en la barra de herramientas, como se muestra en la siguiente imagen:

   ![Azure Data Catalog: eliminación de elemento de cuadrícula](media/register-data-assets-tutorial/data-catalog-delete-grid-item.png)

   Si utiliza la vista de lista, la casilla está a la izquierda del elemento, como se muestra en la siguiente imagen:

   ![Azure Data Catalog: eliminación de elemento de lista](media/register-data-assets-tutorial/data-catalog-delete-list-item.png)

   También puede seleccionar varios recursos de datos y eliminarlos, según se muestra en la siguiente imagen:

   ![Azure Data Catalog: eliminación de varios recursos de datos](media/register-data-assets-tutorial/data-catalog-delete-assets.png)

> [!NOTE]
> El comportamiento predeterminado del catálogo es permitir que cualquier usuario registre cualquier origen de datos y permitir que cualquier usuario elimine cualquier recurso de datos que se haya registrado. Las funcionalidades de administración que se incluyen en la edición estándar de Azure Data Catalog proporcionan más opciones para la toma de propiedad de los recursos, ya que restringen quiénes puede detectar y eliminar los recursos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Siga los pasos de [Eliminación de los recursos de datos](#remove-data-assets) para limpiar los recursos utilizados al seguir este tutorial.

## <a name="summary"></a>Resumen

En este tutorial ha explorado las funcionalidades esenciales de Azure Data Catalog, entre las que se incluyen el registro, la anotación, la detección y la administración de recursos de datos empresariales. Ahora que ha completado el tutorial, ha llegado el momento de comenzar. Puede empezar hoy mismo por registrar los orígenes de datos en los que confían usted y su equipo, e invitando a compañeros a usar el catálogo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Orígenes de datos admitidos](data-catalog-dsr.md)
