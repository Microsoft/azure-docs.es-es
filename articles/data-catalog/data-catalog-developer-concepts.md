---
title: Conceptos para desarrolladores de Azure Data Catalog
description: Introducción a los conceptos clave en el modelo conceptual de Azure Data Catalog, como se expone mediante la API de REST del catálogo.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: b65697c224f612a1bc9d5bfa193355832cafd73f
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799254"
---
# <a name="azure-data-catalog-developer-concepts"></a>Conceptos para desarrolladores de Azure Data Catalog
**Microsoft Azure Data Catalog** es un servicio en la nube totalmente administrado que proporciona capacidades de detección de origen de datos y para metadatos de origen de datos de micromecenazgo. Los desarrolladores pueden usar el servicio a través de sus API de REST. Comprender los conceptos que se implementa en el servicio es importante para los desarrolladores para integrarse correctamente con **Azure Data Catalog**.

## <a name="key-concepts"></a>Conceptos clave 
El modelo conceptual **Azure Data Catalog** se basa en cuatro conceptos clave: el **catálogo**, los **usuarios**, los **recursos** y las **anotaciones**.

![Ilustración del modelo conceptual de Azure Data Catalog](./media/data-catalog-developer-concepts/concept2.png)

### <a name="catalog"></a>Catálogo
Un **catálogo** es el contenedor de nivel superior para todos los metadatos que una organización almacena. Se permite un **catálogo** por cuenta de Azure. Los catálogos están vinculados a una suscripción de Azure, pero solo se puede crear un **catálogo** para una sola cuenta de Azure, aunque una cuenta puede tener varias suscripciones.

Un catálogo contiene **usuarios** y **recursos**.

### <a name="users"></a>Usuarios
Los usuarios son entidades de seguridad que tienen permisos para realizar acciones (buscar en el catálogo, agregar, editar o eliminar elementos, etc...) en el catálogo.

Existen diferentes roles que un usuario puede tener. Para más información sobre los roles, consulte la sección Roles y autorización.

Se pueden agregar usuarios individuales y grupos de seguridad.

Azure Data Catalog usa Azure Active Directory para la administración de identidades y acceso. Cada usuario de catálogo debe ser un miembro de Active Directory para la cuenta.

### <a name="assets"></a>Recursos
Un **catálogo** contiene recursos de datos. **recursos** son la unidad de granularidad que administra el catálogo.

La granularidad de un recurso varía según el origen de datos. Para una o Base de datos SQL Server o de Oracle, un recurso puede ser una tabla o una vista. Para SQL Server Analysis Services, un recurso puede ser una medida, una dimensión o un indicador clave de rendimiento (KPI). Para SQL Server Reporting Services, un recurso es un informe.

Un **Recurso** es el elemento que se agrega o quita de un Catálogo. Es la unidad del resultado que recibimos de la **Búsqueda**.

Un **Recurso** está compuesto por su nombre, ubicación y tipo, así como de las anotaciones que lo describen.

### <a name="annotations"></a>anotaciones
Las anotaciones son elementos que representan los metadatos acerca de los recursos.

Ejemplos de las anotaciones incluyen descripciones, etiquetas, esquemas, documentación, etc. Consulte la [sección Modelo de objetos de recurso](#asset-object-model) para obtener una lista completa de los tipos de recursos y de anotaciones.

## <a name="crowdsourcing-annotations-and-user-perspective-multiplicity-of-opinion"></a>Anotaciones de micromecenazgo y perspectiva del usuario (multiplicidad de opinión)
Un aspecto clave de Azure Data Catalog es cómo admite el micromecenazgo de los metadatos en el sistema. En contraposición con un enfoque de wiki (en el que solo hay una opinión y el último escritor es el que gana), el modelo de Azure Data Catalog permite escribir varias opiniones en el sistema.

Este enfoque refleja el mundo real de los datos empresariales, en el que distintos usuarios pueden tener distintas perspectivas en un recurso:

* Un administrador de base de datos puede proporcionar información sobre los acuerdos de nivel de servicio o la ventana de procesamiento disponible para las operaciones de ETL masivas
* Un administrador de datos puede proporcionar información sobre los procesos de negocio a los que se aplica el recurso, o las clasificaciones que la empresa le ha aplicado
* Un analista financiero puede proporcionar información acerca de cómo se usan los datos durante las tareas de informes de final de período

Para admitir este ejemplo, cada usuario (el DBA, el ddministrador de datos y el analista) puede agregar una descripción a una única tabla que se ha registrado en el catálogo. Todas las descripciones se mantienen en el sistema y en el portal de Azure Data Catalog se muestran todas las descripciones.

Este patrón se aplica a la mayoría de los artículos en el modelo de objetos, por lo que los tipos de objetos de la carga de JSON son a menudo matrices de las propiedades en las que se puede esperar un singleton.

Por ejemplo, en la raíz del recurso se encuentra una matriz de objetos de descripción. La propiedad de matriz se denomina "descriptions". Un objeto de descripción tiene una propiedad: description. El patrón es que cada usuario que escribe la descripción obtiene un objeto de descripción creado para el valor proporcionado por el usuario.

A continuación, la experiencia de usuario puede elegir cómo mostrar la combinación. A continuación se muestran tres modelos diferentes para mostrar.

* El modelo más simple es "Show All". En este patrón, se muestran todos los objetos en una vista de lista. El portal de Azure Data Catalog UX utiliza este patrón para la descripción.
* Otro patrón es "Merge". En este patrón, todos los valores de los distintos usuarios se combinan, y se eliminan los duplicados. Ejemplos de este patrón en la experiencia del usuario del portal de Azure Data Catalog son las etiquetas y las propiedades de expertos.
* Un tercer patrón es "el último escritor gana". En este patrón, solo se muestra el valor más reciente escrito. friendlyName es un ejemplo de este patrón.

## <a name="asset-object-model"></a>Modelo de objeto de recurso
Como se mencionó en la sección Conceptos clave, el modelo de objetos **Azure Data Catalog** incluye elementos, que pueden ser recursos o anotaciones. Los elementos tienen propiedades, que pueden ser optional o required. Algunas propiedades se aplican a todos los elementos. Algunas propiedades se aplican a todos los recursos. Algunas propiedades se aplican solo a tipos de recursos específicos.

### <a name="system-properties"></a>Propiedades del sistema
<table><tr><td><b>Nombre de la propiedad</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr><tr><td>timestamp</td><td>DateTime</td><td>La última vez que se modificó el elemento. El servidor genera este campo cuando se inserta un elemento y cada vez que se actualiza. El valor de esta propiedad se omite en la entrada de las operaciones de publicación.</td></tr><tr><td>Id.</td><td>Identificador URI</td><td>Dirección URL absoluta del elemento (solo lectura). Es el identificador URI direccionable único para el elemento.  El valor de esta propiedad se omite en la entrada de las operaciones de publicación.</td></tr><tr><td>type</td><td>String</td><td>El tipo de recurso (solo lectura).</td></tr><tr><td>ETag</td><td>String</td><td>Una cadena correspondiente a la versión del elemento que puede utilizarse para el control de simultaneidad optimista al realizar operaciones que actualizan elementos en el catálogo. "*" puede usarse para coincidir con cualquier valor.</td></tr></table>

### <a name="common-properties"></a>Propiedades comunes
Estas propiedades se aplican a todos los tipos de recursos de raíz y a todos los tipos de anotación.

<table>
<tr><td><b>Nombre de la propiedad</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr>
<tr><td>fromSourceSystem</td><td>Boolean</td><td>Indica si los datos del elemento derivan de un sistema de origen (como una base de datos de SQL Server, o de Oracle Database) o los creó un usuario.</td></tr>
</table>

### <a name="common-root-properties"></a>Propiedades de raíz comunes
<p>
Estas propiedades se aplican a todos los tipos de recursos de raíz.

<table><tr><td><b>Nombre de la propiedad</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr><tr><td>name</td><td>String</td><td>Un nombre derivado de la información de ubicación del origen de datos</td></tr><tr><td>dsl</td><td>DataSourceLocation</td><td>Describe el origen de datos de forma exclusiva y es uno de los identificadores del recurso. (Consulte la sección de identidad dual).  La estructura del dsl varía según el protocolo y el tipo de origen.</td></tr><tr><td>dataSource</td><td>DataSourceInfo</td><td>Más información sobre el tipo de recurso.</td></tr><tr><td>lastRegisteredBy</td><td>SecurityPrincipal</td><td>Describe el usuario que registró más recientemente este recurso.  Contiene el id. único para el usuario (upn) y un nombre para mostrar (lastName y firstName).</td></tr><tr><td>containerID</td><td>String</td><td>Id. del recurso de contenedor para el origen de datos. Esta propiedad no se admite para el tipo de contenedor.</td></tr></table>

### <a name="common-non-singleton-annotation-properties"></a>Propiedades comunes de anotación no de singleton
Estas propiedades se aplican a todos los tipos de anotación no singleton (anotaciones que pueden ser múltiples por recurso).

<table>
<tr><td><b>Nombre de la propiedad</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr>
<tr><td>key</td><td>String</td><td>Clave especificada por el usuario que identifica de forma única la anotación en la colección actual. La longitud de la clave no puede superar los 256 caracteres.</td></tr>
</table>

### <a name="root-asset-types"></a>Tipos de recursos de raíz
Los tipos de recursos de raíz son aquellos que representan los distintos tipos de recursos de datos que se pueden registrar en el catálogo. Para cada tipo de raíz hay una vista definida que describe el recurso y las anotaciones que se incluyen en la vista. El nombre de la vista debe usarse en el segmento de dirección URL {view_name} correspondiente al publicar un recurso mediante la API de REST.

<table><tr><td><b>Tipo de recurso (nombre de la vista)</b></td><td><b>Propiedades adicionales</b></td><td><b>Tipo de datos</b></td><td><b>Anotaciones permitidas</b></td><td><b>Comentarios</b></td></tr><tr><td>Tabla ("tables")</td><td></td><td></td><td>Descripción<p>FriendlyName<p>Etiqueta<p>Schema<p>ColumnDescription<p>ColumnTag<p> Experto<p>Versión preliminar<p>AccessInstruction<p>TableDataProfile<p>ColumnDataProfile<p>ColumnDataClassification<p>Documentación<p></td><td>Una tabla representa datos tabulares.  Por ejemplo, una tabla SQL, una vista SQL, una tabla del modelo tabular de Analysis Services, una dimensión multidimensional de Analysis Services, una tabla de Oracle, etc.   </td></tr><tr><td>Medidas ("measures")</td><td></td><td></td><td>Descripción<p>FriendlyName<p>Etiqueta<p>Experto<p>AccessInstruction<p>Documentación<p></td><td>Este tipo representa una medida de Analysis Services.</td></tr><tr><td></td><td>measure</td><td>Columna</td><td></td><td>Metadatos que describen la medida</td></tr><tr><td></td><td>isCalculated </td><td>Boolean</td><td></td><td>Especifica si se calcula la medida o no.</td></tr><tr><td></td><td>measureGroup</td><td>String</td><td></td><td>Contenedor físico de medida</td></tr><td>KPI ("kpis")</td><td></td><td></td><td>Descripción<p>FriendlyName<p>Etiqueta<p>Experto<p>AccessInstruction<p>Documentación</td><td></td></tr><tr><td></td><td>measureGroup</td><td>String</td><td></td><td>Contenedor físico de medida</td></tr><tr><td></td><td>goalExpression</td><td>String</td><td></td><td>Expresión numérica MDX o cálculo que devuelve el valor de destino del KPI.</td></tr><tr><td></td><td>valueExpression</td><td>String</td><td></td><td>Expresión númerica MDX que devuelve el valor real del KPI.</td></tr><tr><td></td><td>statusExpression</td><td>String</td><td></td><td>Expresión MDX que representa el estado del KPI en un punto temporal específico.</td></tr><tr><td></td><td>trendExpression</td><td>String</td><td></td><td>Una expresión MDX que evalúa el valor del KPI en el tiempo. La tendencia puede ser cualquier criterio basado en el tiempo que sea útil en determinado contexto empresarial.</td>
<tr><td>Informe ("reports")</td><td></td><td></td><td>Descripción<p>FriendlyName<p>Etiqueta<p>Experto<p>AccessInstruction<p>Documentación<p></td><td>Este tipo representa un informe de SQL Server Reporting Services </td></tr><tr><td></td><td>assetCreatedDate</td><td>String</td><td></td><td></td></tr><tr><td></td><td>assetCreatedBy</td><td>String</td><td></td><td></td></tr><tr><td></td><td>assetModifiedDate</td><td>String</td><td></td><td></td></tr><tr><td></td><td>assetModifiedBy</td><td>String</td><td></td><td></td></tr><tr><td>Contenedor ("containers")</td><td></td><td></td><td>Descripción<p>FriendlyName<p>Etiqueta<p>Experto<p>AccessInstruction<p>Documentación<p></td><td>Este tipo representa un contenedor de otros activos, como una base de datos SQL, un contenedor de blobs de Azure o un modelo de Analysis Services.</td></tr></table>

### <a name="annotation-types"></a>Tipos de anotación
Los tipos de anotación representan tipos de metadatos que se pueden asignar a otros tipos dentro del catálogo.

<table>
<tr><td><b>Tipo de anotación (nombre de la vista anidada)</b></td><td><b>Propiedades adicionales</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr>

<tr><td>Descripción ("descriptions")</td><td></td><td></td><td>Esta propiedad contiene una descripción para un recurso. Cada usuario del sistema puede agregar sus propia descripción.  Solo ese usuario puede editar el objeto de la descripción.  (Los administradores y los propietarios de recursos pueden eliminar el objeto Description, pero no editarlo). El sistema mantiene las descripciones de los usuarios por separado.  Por lo tanto, hay una matriz de descripciones en cada recurso (una para cada usuario que ha contribuido con sus conocimientos sobre el recurso, además de posiblemente uno que contenga información derivada del origen de datos).</td></tr>
<tr><td></td><td>description</td><td>string</td><td>Una descripción breve (de entre 2 y 3 líneas) del recurso</td></tr>

<tr><td>Etiqueta ("tags")</td><td></td><td></td><td>Esta propiedad define una etiqueta para un recurso. Cada usuario del sistema puede agregar varias etiquetas para un recurso.  Solo el usuario que ha creado objetos Tag puede modificarlos.  (Los administradores y los propietarios de recursos pueden eliminar el objeto Tag, pero no editarlo). El sistema mantiene las etiquetas de los usuarios por separado.  Por lo tanto, hay una matriz de objetos Tag en cada recurso.</td></tr>
<tr><td></td><td>etiqueta</td><td>string</td><td>Una etiqueta que describe el recurso.</td></tr>

<tr><td>FriendlyName ("friendlyName")</td><td></td><td></td><td>Esta propiedad contiene un nombre descriptivo para un recurso. FriendlyName es una anotación de singleton: solo un objeto FriendlyName puede agregarse a un recurso.  Solo el usuario que creó el objeto FriendlyName puede editarlo. (Los administradores y los propietarios de recursos pueden eliminar el objeto FriendlyName, pero no editarlo). El sistema mantiene los nombres descriptivos de los usuarios por separado.</td></tr>
<tr><td></td><td>friendlyName</td><td>string</td><td>Nombre descriptivo del recurso.</td></tr>

<tr><td>Esquema ("schema")</td><td></td><td></td><td>El esquema describe la estructura de los datos.  Enumera los nombres de los atributos (columna, atributo, campo, etc.), así como otros metadatos.  Esta información se deriva del origen de datos.  El esquema es una anotación de singleton: solo puede agregarse un esquema a un recurso.</td></tr>
<tr><td></td><td>columnas</td><td>Column[]</td><td>Una matriz de objetos de columna. Describen la columna con información obtenida a partir del origen de datos.</td></tr>

<tr><td>ColumnDescription ("columnDescriptions")</td><td></td><td></td><td>Esta propiedad contiene una descripción para una columna.  Cada usuario del sistema puede agregar sus propias descripciones para varias columnas (a lo sumo uno por columna). Solo el usuario que ha creado objetos ColumnDescription puede modificarlos.  (Los administradores y los propietarios de recursos pueden eliminar el objeto ColumnDescription, pero no editarlo). El sistema mantiene las descripciones de las columnas de usuario por separado.  Por lo tanto, hay una matriz de objetos ColumnDescription en cada recurso (uno por columna para cada usuario que ha contribuido con sus conocimientos acerca de la columna, además de posiblemente uno que contiene información derivada del origen de datos).  ColumnDescription está enlazado de forma flexible con el esquema, por lo que puede dejar de estar sincronizado. ColumnDescription puede describir una columna que ya no existe en el esquema.  El escritor puede decidir si desea mantener la descripción y el esquema sincronizados.  El origen de datos también puede tener información de descripción de las columnas, y hay objetos ColumnDescription adicionales que se crean al ejecutar la herramienta.</td></tr>
<tr><td></td><td>columnName</td><td>String</td><td>Nombre de la columna a la que hace referencia esta descripción.</td></tr>
<tr><td></td><td>description</td><td>String</td><td>Una descripción breve (de entre 2 y 3 líneas) de la columna.</td></tr>

<tr><td>ColumnTag ("columnTags")</td><td></td><td></td><td>Esta propiedad contiene una etiqueta para una columna. Todos los usuarios del sistema pueden agregar varias etiquetas a una columna determinada y agregar etiquetas de varias columnas. Solo el usuario que ha creado objetos ColumnTag puede modificarlos. (Los administradores y los propietarios de recursos pueden eliminar el objeto ColumnTag, pero no editarlo). El sistema mantiene las etiquetas de columna de estos usuarios por separado.  Por lo tanto, hay una matriz de objetos ColumnTag en cada recurso.  ColumnTag flexible está enlazado de forma flexible con el esquema, por lo que puede dejar de estar sincronizado. ColumnTag puede describir una columna que ya no existe en el esquema.  El escritor puede decidir si desea mantener la etiqueta de la columna y el esquema sincronizados.</td></tr>
<tr><td></td><td>columnName</td><td>String</td><td>Nombre de la columna a la que hace referencia esta etiqueta.</td></tr>
<tr><td></td><td>etiqueta</td><td>String</td><td>Una etiqueta que describe la columna.</td></tr>

<tr><td>Experto ("experts")</td><td></td><td></td><td>Esta propiedad contiene un usuario que se considera un experto en el conjunto de datos. Las opiniones de los expertos (descripciones) se muestran en la parte superior de la experiencia de usuario al enumerar las descripciones. Cada usuario puede especificar sus propios expertos. Solo ese usuario podrá modificar el objeto de expertos. (Los administradores y los propietarios de recursos pueden eliminar estos objeto Expert, pero no editarlos).</td></tr>
<tr><td></td><td>expert</td><td>SecurityPrincipal</td><td></td></tr>

<tr><td>Vista previa ("previews")</td><td></td><td></td><td>La vista previa contiene una instantánea de las 20 filas principales de datos para el recurso. La vista previa solo tiene sentido para algunos tipos de recursos (tiene sentido para las tablas, pero no para las medidas).</td></tr>
<tr><td></td><td>Vista previa</td><td>object[]</td><td>Matriz de objetos que representan una columna.  Cada objeto tiene una asignación de propiedad para una columna con un valor para la columna de la fila.</td></tr>

<tr><td>AccessInstruction ("accessInstructions")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>mimeType</td><td>string</td><td>El tipo MIME del contenido.</td></tr>
<tr><td></td><td>contenido</td><td>string</td><td>Las instrucciones sobre cómo obtener acceso a este recurso de datos. El contenido podría ser una dirección URL, una dirección de correo electrónico o un conjunto de instrucciones.</td></tr>

<tr><td>TableDataProfile ("tableDataProfiles")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>numberOfRows</td></td><td>int</td><td>El número de filas del conjunto de datos</td></tr>
<tr><td></td><td>tamaño</td><td>long</td><td>El tamaño en bytes del conjunto de datos.  </td></tr>
<tr><td></td><td>schemaModifiedTime</td><td>string</td><td>La última vez que se modificó el esquema.</td></tr>
<tr><td></td><td>dataModifiedTime</td><td>string</td><td>La última vez que se modificó el conjunto de datos (se han agregado, modificado o eliminado datos).</td></tr>

<tr><td>ColumnsDataProfile ("columnsDataProfiles")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columnas</td></td><td>ColumnDataProfile[]</td><td>Matriz de perfiles de datos de columna.</td></tr>

<tr><td>ColumnDataClassification ("columnDataClassifications")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columnName</td><td>String</td><td>Nombre de la columna a la que hace referencia esta clasificación.</td></tr>
<tr><td></td><td>clasificación</td><td>String</td><td>La clasificación de los datos en esta columna.</td></tr>

<tr><td>Documentación ("documentation")</td><td></td><td></td><td>Un activo dado solo puede tener una documentación asociada con él.</td></tr>
<tr><td></td><td>mimeType</td><td>string</td><td>El tipo MIME del contenido.</td></tr>
<tr><td></td><td>contenido</td><td>string</td><td>El contenido de la documentación.</td></tr>

</table>

### <a name="common-types"></a>Tipos comunes
Tipos comunes pueden usarse como tipos de propiedades, pero no son elementos.

<table>
<tr><td><b>Tipo común</b></td><td><b>Propiedades</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr>
<tr><td>DataSourceInfo</td><td></td><td></td><td></td></tr>
<tr><td></td><td>sourceType</td><td>string</td><td>Describe el tipo de origen de datos.  Por ejemplo: SQL Server, Base de datos de Oracle, etc.  </td></tr>
<tr><td></td><td>objectType</td><td>string</td><td>Describe el tipo de objeto en el origen de datos. Por ejemplo, tabla, vista de SQL Server.</td></tr>

<tr><td>DataSourceLocation</td><td></td><td></td><td></td></tr>
<tr><td></td><td>protocol</td><td>string</td><td>Necesario. Describe un protocolo utilizado para comunicarse con el origen de datos. Por ejemplo, `tds` (para SQL Server), `oracle` (para Oracle), etc. Consulte [Especificación de referencia de origen de datos: estructura de DSL](data-catalog-dsr.md) para la lista de protocolos admitidos actualmente.</td></tr>
<tr><td></td><td>address</td><td>Diccionario&lt;string, objeto&gt;</td><td>Necesario. La dirección es un conjunto de datos específicos del protocolo que se utiliza para identificar el origen de datos al que se hace referencia. Los datos de direcciones con ámbito en un protocolo determinado, lo que significa que no tiene sentido sin conocer el protocolo.</td></tr>
<tr><td></td><td>autenticación</td><td>string</td><td>Opcional. El esquema de autenticación que se utiliza para comunicarse con el origen de datos. Por ejemplo, windows, oauth, etc.</td></tr>
<tr><td></td><td>connectionProperties</td><td>Diccionario&lt;string, objeto&gt;</td><td>Opcional. Información adicional sobre cómo conectarse a un origen de datos.</td></tr>

<tr><td>SecurityPrincipal</td><td></td><td></td><td>El servidor back-end no realiza ninguna validación de las propiedades proporcionadas en Azure Active Directory durante la publicación.</td></tr>
<tr><td></td><td>upn</td><td>string</td><td>Dirección de correo electrónico única del usuario. Debe especificarse si no se proporciona el identificador de objeto o en el contexto de la propiedad "lastRegisteredBy"; de lo contrario, es opcional.</td></tr>
<tr><td></td><td>objectId</td><td>Guid</td><td>Identidad de Azure Active Directory de un grupo de seguridad o usuario. Opcional. Debe especificarse si no se proporciona UPN; de lo contrario, es opcional.</td></tr>
<tr><td></td><td>firstName</td><td>string</td><td>Nombre de usuario (para fines de presentación). Opcional. Solo es válido en el contexto de la propiedad "lastRegisteredBy". No se puede especificar al proporcionar la entidad de seguridad para "roles", "permisos" y "expertos".</td></tr>
<tr><td></td><td>lastName</td><td>string</td><td>Apellidos del usuario (para fines de presentación). Opcional. Solo es válido en el contexto de la propiedad "lastRegisteredBy". No se puede especificar al proporcionar la entidad de seguridad para "roles", "permisos" y "expertos".</td></tr>

<tr><td>Columna</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>string</td><td>Nombre de la columna o atributo.</td></tr>
<tr><td></td><td>type</td><td>string</td><td>Tipo de datos de la columna o atributo. Los tipos permitidos dependen del sourceType de datos del recurso.  Solo se admite un subconjunto de tipos.</td></tr>
<tr><td></td><td>maxLength</td><td>int</td><td>Longitud máxima permitida para la columna o el atributo. Derivada del origen de datos. Solo se aplica a algunos tipos de origen.</td></tr>
<tr><td></td><td>Precisión</td><td>byte</td><td>Precisión de la columna o atributo. Derivada del origen de datos. Solo se aplica a algunos tipos de origen.</td></tr>
<tr><td></td><td>isNullable</td><td>Boolean</td><td>Si se permite que la columna tenga un valor null o no. Derivada del origen de datos. Solo se aplica a algunos tipos de origen.</td></tr>
<tr><td></td><td>expresión</td><td>string</td><td>Si el valor es una columna calculada, este campo contiene la expresión que expresa el valor. Derivada del origen de datos. Solo se aplica a algunos tipos de origen.</td></tr>

<tr><td>ColumnDataProfile</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columnName </td><td>string</td><td>Nombre de la columna</td></tr>
<tr><td></td><td>type </td><td>string</td><td>El tipo de la columna</td></tr>
<tr><td></td><td>min. </td><td>string</td><td>El valor mínimo del conjunto de datos</td></tr>
<tr><td></td><td>máx. </td><td>string</td><td>El valor máximo del conjunto de datos</td></tr>
<tr><td></td><td>avg </td><td>double</td><td>El valor promedio del conjunto de datos</td></tr>
<tr><td></td><td>stdev </td><td>double</td><td>La desviación estándar del conjunto de datos</td></tr>
<tr><td></td><td>nullCount </td><td>int</td><td>El número de valores null del conjunto de datos</td></tr>
<tr><td></td><td>distinctCount  </td><td>int</td><td>El número de valores distinct del conjunto de datos</td></tr>
</table>

## <a name="asset-identity"></a>Identidad de recursos
Azure Data Catalog utiliza las propiedades de identidad y "protocolo" desde el contenedor de propiedades "address" de la propiedad "dsl" DataSourceLocation para generar la identidad del recurso que se usa para dirigir el recurso dentro del Catálogo.
Por ejemplo, el protocolo de flujo TDS tiene las propiedades de identidad "server", "database", "schema" y "object". Las combinaciones de las propiedades de identidad y protocolo se utilizan para generar la identidad del recurso de la tabla de SQL Server.
Azure Data Catalog proporciona varios protocolos de origen de datos integrados que se enumeran en [Especificación de referencia de origen de datos: estructura de DSL](data-catalog-dsr.md).
El conjunto de protocolos admitidos se puede extender mediante programación (consulte la referencia de API de REST de Catálogo de datos). Los administradores del Catálogo pueden registrar los protocolos de orígenes de datos personalizados. En la tabla siguiente se describen las propiedades necesarias para registrar un protocolo personalizado.

### <a name="custom-data-source-protocol-specification"></a>Especificación del protocolo de orígenes de datos personalizados
<table>
<tr><td><b>Tipo</b></td><td><b>Propiedades</b></td><td><b>Tipo de datos</b></td><td><b>Comentarios</b></td></tr>

<tr><td>DataSourceProtocol</td><td></td><td></td><td></td></tr>
<tr><td></td><td>espacio de nombres</td><td>string</td><td>Espacio de nombres del protocolo El espacio de nombres debe tener entre 1 y 255 caracteres, contener una o más partes no vacías separadas por un punto (.). Cada parte debe estar entre 1 y 255 caracteres de longitud, comenzar por una letra y contener solo letras y números.</td></tr>
<tr><td></td><td>name</td><td>string</td><td>El nombre del protocolo El nombre debe tener de 1 a 255 caracteres, comenzar por una letra o contener solo letras, números y el carácter de guion (-).</td></tr>
<tr><td></td><td>identityProperties</td><td>DataSourceProtocolIdentityProperty[]</td><td>Lista de propiedades de identidad, debe contener al menos una, pero no más de 20 propiedades. Por ejemplo: "server", "database", "schema", "object" son propiedades de identidad del protocolo "tds".</td></tr>
<tr><td></td><td>identitySets</td><td>DataSourceProtocolIdentitySet[]</td><td>Lista de conjuntos de identidades. Define conjuntos de propiedades de identidad que representan la identidad del recurso válido. Debe contener al menos una, pero no más de 20 propiedades. Por ejemplo: {"server", "database", "schema" y "object"} es un conjunto de identidades para el protocolo TDS, que define la identidad del recurso de tabla de SQL Server.</td></tr>

<tr><td>DataSourceProtocolIdentityProperty</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>string</td><td>Nombre de la propiedad. El nombre debe tener entre 1 y 100 caracteres, comenzar por una letra y puede contener solo letras y números.</td></tr>
<tr><td></td><td>type</td><td>string</td><td>Tipo de la propiedad. Valores admitidos: "bool", "boolean", "byte", "guid", "int", "integer", "long", "string", "url"</td></tr>
<tr><td></td><td>ignoreCase</td><td>bool</td><td>Indica si se ignoran las mayúsculas y minúsculas al usar el valor de la propiedad. Solo puede especificarse para las propiedades de tipo "string". El valor predeterminado es false.</td></tr>
<tr><td></td><td>urlPathSegmentsIgnoreCase</td><td>bool[]</td><td>Indica si se ignoran las mayúsculas y minúsculas para cada segmento de las ruta de acceso de la dirección URL. Solo puede especificarse para las propiedades de tipo "url". El valor predeterminado es [false].</td></tr>

<tr><td>DataSourceProtocolIdentitySet</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>string</td><td>El nombre del conjunto de identidades.</td></tr>
<tr><td></td><td>properties</td><td>string[]</td><td>La lista de propiedades de identidad que se incluyen en este conjunto de identidades. No puede contener duplicados. Cada propiedad a la que hace referencia el conjunto de identidades debe definirse en la lista de "identityProperties" del protocolo.</td></tr>

</table>

## <a name="roles-and-authorization"></a>Roles y autorización
Microsoft Azure Data Catalog proporciona capacidades de autorización para las operaciones CRUD en recursos y anotaciones.

Azure Data Catalog usa dos mecanismos de autorización:

* Autorización basada en roles
* Autorización basada en permisos

### <a name="roles"></a>Roles
Existen tres roles: **Administrador**, **Propietario** y **Colaborador**.  Cada rol tiene su ámbito y sus derechos, que se resumen en la tabla siguiente.

<table><tr><td><b>Rol</b></td><td><b>Ámbito</b></td><td><b>Derechos</b></td></tr><tr><td>Administrador</td><td>Catálogo (todos los recursos/anotaciones del catálogo)</td><td>Read Delete ViewRoles

ChangeOwnership ChangeVisibility ViewPermissions</td></tr><tr><td>Propietario</td><td>Cada recurso (elemento raíz)</td><td>Read Delete ViewRoles

ChangeOwnership ChangeVisibility ViewPermissions</td></tr><tr><td>Colaborador</td><td>Cada recurso y anotación individual</td><td>Read Update Delete ViewRoles Nota: todos los derechos se revocarán si la operación de lectura en el elemento se ha revocado desde el colaborador</td></tr></table>

> [!NOTE]
> Los derechos **Read**, **Update**, **Delete** y **ViewRoles** se pueden aplicar a cualquier elemento (recurso o anotación), mientras que **TakeOwnership**, **ChangeOwnership**, **ChangeVisibility** y **ViewPermissions** solo se pueden aplicar al recurso de raíz.
> 
> **Delete** se aplica a un elemento y a los elementos secundarios o únicos situados debajo de él. Por ejemplo, la eliminación de un recurso también eliminará todas las anotaciones de ese recurso.
> 

### <a name="permissions"></a>Permisos
Un permiso es un lista de entradas de control de acceso. Cada entrada de control de acceso asigna un conjunto de derechos a una entidad de seguridad. Los permisos solo se pueden especificar en un recurso (es decir, el elemento raíz) y se aplican al recurso y a los elementos secundarios.

Durante la vista previa del **Azure Data Catalog**, solo se admite el derecho **Read** en la lista de permisos para habilitar el escenario que restringe la visibilidad de un recurso.

De forma predeterminada, cualquier usuario autenticado tiene derecho **Read** para cualquier elemento del catálogo, a menos que la visibilidad esté restringida al conjunto de entidades de los permisos.

## <a name="rest-api"></a>API DE REST
Las solicitudes de elementos de visualización **PUT** y **POST** pueden usarse para controlar roles y permisos; además de la carga de elementos, pueden especificarse dos propiedades del sistema: **roles** y **permissions**.

> [!NOTE]
> **permissions** solo se aplica a un elemento de raíz.
> 
> **Propietario** solo es aplicable a un elemento de raíz.
> 
> De forma predeterminada, cuando se crea un elemento en el catálogo, su **Colaborador** se establece como el usuario autenticado actualmente. Si el elemento debe poder ser actualizable por todo el mundo, el **Colaborador** debe establecerse como entidad de seguridad especial &lt;Todos&gt; en la propiedad **roles** cuando se publique la propiedad por primera vez (consulte el ejemplo siguiente). El **Colaborador** no se puede cambiar y permanece igual mientras dura un elemento (incluso **Administrador** o **Propietario** no tienen derecho para cambiar el **Colaborador**). El único valor que se admite para la configuración explícita de **Colaborador** es &lt;Todos&gt;: es decir, **Colaborador** solo puede ser un usuario que haya creado un elemento o &lt;Todos&gt;.
> 

### <a name="examples"></a>Ejemplos
**Establecer Colaborador como &lt;Todos&gt; al publicar un elemento.**
La entidad de seguridad especial &lt;Todos&gt; tiene el elemento objectId "00000000-0000-0000-0000-000000000201".
  **POST** https:\//api.azuredatacatalog.com/catalogs/default/views/tables/?api-version=2016-03-30

> [!NOTE]
> Algunas implementaciones de cliente HTTP pueden volver a emitir automáticamente solicitudes como respuesta a HTTP 302 desde el servidor, pero normalmente eliminan encabezados Authorization de la solicitud. Dado que el encabezado Authorization se requiere para realizar solicitudes a Azure Data Catalog, es preciso asegurarse de que todavía se proporciona el encabezado Authorization al volver a emitir una solicitud a una ubicación de redireccionamiento especificada por Azure Data Catalog. El ejemplo de código siguiente lo demuestra mediante el objeto HttpWebRequest. NET.
> 

**Cuerpo**
```json
    {
        "roles": [
            {
                "role": "Contributor",
                "members": [
                    {
                        "objectId": "00000000-0000-0000-0000-000000000201"
                    }
                ]
            }
        ]
    }
```

  **Asignar propietarios y restringir la visibilidad de un elemento raíz existente**: **PUT** https:\//api.azuredatacatalog.com/catalogs/default/views/tables/042297b0...1be45ecd462a?api-version=2016-03-30

```json
    {
        "roles": [
            {
                "role": "Owner",
                "members": [
                    {
                        "objectId": "c4159539-846a-45af-bdfb-58efd3772b43",
                        "upn": "user1@contoso.com"
                    },
                    {
                        "objectId": "fdabd95b-7c56-47d6-a6ba-a7c5f264533f",
                        "upn": "user2@contoso.com"
                    }
                ]
            }
        ],
        "permissions": [
            {
                "principal": {
                    "objectId": "27b9a0eb-bb71-4297-9f1f-c462dab7192a",
                    "upn": "user3@contoso.com"
                },
                "rights": [
                    {
                        "right": "Read"
                    }
                ]
            },
            {
                "principal": {
                    "objectId": "4c8bc8ce-225c-4fcf-b09a-047030baab31",
                    "upn": "user4@contoso.com"
                },
                "rights": [
                    {
                        "right": "Read"
                    }
                ]
            }
        ]
    }
```

> [!NOTE]
> En PUT no es necesario especificar una carga de elementos en el cuerpo: PUT puede usarse para actualizar solo roles y permisos.
> 

## <a name="next-steps"></a>Pasos siguientes
[Referencia de la API REST de Azure Data Catalog](/rest/api/datacatalog/)
