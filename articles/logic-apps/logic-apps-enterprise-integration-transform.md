---
title: Transformación de XML entre formatos
description: Crear transformaciones o asignaciones para convertir XML de un formato a otro en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 07/08/2016
ms.openlocfilehash: da583a1c884ddcae0815849c43dc0eb335005e53
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832746"
---
# <a name="create-maps-that-transform-xml-between-formats-in-azure-logic-apps-with-enterprise-integration-pack"></a>Creación de asignaciones para transformar XML de un formato a otro en Azure Logic Apps con Enterprise Integration Pack

El conector de transformación de integración de empresas convierte los datos de un formato a otro. Por ejemplo, puede darse el caso de que tenga un mensaje entrante en el que la fecha actual tenga el formato añoMesDía. Puede utilizar una transformación para cambiar el formato de la fecha a mesDíaAño.

## <a name="what-does-a-transform-do"></a>¿Para qué sirve una transformación?
Una Transformación, que se conoce también como asignación, está formada por un esquema XML de origen (la entrada) y un esquema XML de destino (la salida). Puede utilizar las diferentes funciones integradas para administrar o controlar los datos, incluidos aspectos como las manipulaciones de cadenas, las asignaciones condicionales, las expresiones aritméticas, los formateadores de tiempo y fecha e, incluso, las construcciones en bucle.

## <a name="how-to-create-a-transform"></a>¿Cómo se crea una transformación?
Puede crear una asignación o una transformación mediante el [SDK de integración de empresas](https://aka.ms/vsmapsandschemas)de Visual Studio. Cuando haya terminado de crear y probar la transformación, cargue la transformación en la cuenta de integración. 

## <a name="how-to-use-a-transform"></a>Procedimiento para utilizar una transformación
Cuando cargue la transformación o la asignación en la cuenta de integración, podrá emplearla para crear una aplicación lógica. Esta ejecuta las transformaciones siempre que se desencadena la aplicación lógica (y que haya contenido de entrada que deba transformarse).

**Estos son los pasos para utilizar una transformación**:

### <a name="prerequisites"></a>Prerrequisitos

* Crear una cuenta de integración y agregarle una asignación.  

Ahora que ha tenido en cuenta los requisitos previos, tendrá que crear la Aplicación lógica:  

1. Cree una aplicación lógica y [vincúlela a su cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md "Aprenda a vincular una cuenta de integración a una aplicación lógica.") que contiene la asignación.
2. Agregue un desencadenador de tipo **Solicitud** a su aplicación lógica.  
   ![Captura de pantalla de la lista desplegable "Mostrar las API administradas por Microsoft" con el desencadenador de solicitud seleccionado. La lista desplegable está en una aplicación lógica creada con el SDK de integración de Visual Studio Enterprise.](./media/logic-apps-enterprise-integration-transforms/transform-1.png)    
3. Agregue la acción **Transform XML** (Transformar XML), pero seleccione antes **Agregar una acción**   
   ![Captura de pantalla que muestra el botón "Agregar una acción" seleccionado en la pantalla del desencadenador de solicitud.](./media/logic-apps-enterprise-integration-transforms/transform-2.png)   
4. Escriba la palabra *transform* en el cuadro de búsqueda para filtrar todas las acciones por la que desee usar.  
   ![Captura de pantalla que muestra cómo buscar la acción Transformar XML en la lista desplegable "Mostrar las API administradas por Microsoft" para que se pueda agregar al desencadenador de solicitud.](./media/logic-apps-enterprise-integration-transforms/transform-3.png)  
5. Seleccione la acción **Transform XML** (Transformar XML)   
6. Agregue el **CONTENIDO** XML que vaya a transformar. Puede usar cualquier dato XML que reciba en la solicitud HTTP como valor de **CONTENIDO**. En este ejemplo, seleccione el cuerpo de la solicitud HTTP que desencadenó la Aplicación lógica.

   > [!NOTE]
   > Asegúrese de que el contenido de **Transformar XML** sea XML. Si el contenido no está en formato XML o está codificado en base 64, debe especificar una expresión que procese el contenido. Por ejemplo, puede usar [funciones](logic-apps-workflow-definition-language.md#functions), como ```@base64ToBinary```, para descodificar el contenido o ```@xml``` para procesar el contenido como XML.
 

7. Seleccione el nombre de la **ASIGNACIÓN** que quiera usar para realizar la transformación. La asignación ya debe estar en la cuenta de integración. En el paso anterior, ya concedió a la Aplicación lógica acceso a la cuenta de integración que contiene la asignación.      
   ![Captura de pantalla que muestra los campos de contenido y de asignación en la pantalla Transformar XML para el desencadenador de solicitud.](./media/logic-apps-enterprise-integration-transforms/transform-4.png) 
8. Guarde el trabajo  
    ![Captura de pantalla que muestra el botón Guardar en el Diseñador de aplicaciones lógicas.](./media/logic-apps-enterprise-integration-transforms/transform-5.png) 

En este momento, ya ha terminado de configurar su asignación. En una aplicación real, puede almacenar los datos transformados en una aplicación LOB como SalesForce. Puede agregar fácilmente una acción para enviar el resultado de la transformación a Salesforce. 

Ahora puede probar la transformación realizando una solicitud al punto de conexión HTTP.  


## <a name="features-and-use-cases"></a>Características y casos de uso
* La transformación creada en una asignación puede ser simple, como copiar un nombre y una dirección de un documento a otro. O bien puede crear transformaciones más complejas mediante las operaciones de asignación integradas.  
* Hay varias operaciones de asignación o funciones a las que se puede acceder fácilmente, por ejemplo, cadenas, funciones de fecha hora, etc.  
* Puede realizar una copia de datos directa entre los esquemas. En el Asignador que incluye el SDK, es tan sencillo como dibujar una línea que conecte los elementos del esquema de origen con sus equivalentes en el de destino.  
* Al crear una asignación, verá una representación gráfica de esta, que muestra todos los vínculos y relaciones que cree.
* Utilice la característica Comprobar asignación para agregar un mensaje XML de ejemplo. Con un solo clic, puede probar la asignación creada y ver el resultado generado.  
* Cargue asignaciones que ya existan.  
* Es compatible con el formato XML.

## <a name="advanced-features"></a>Características avanzadas

### <a name="reference-assembly-or-custom-code-from-maps"></a>Ensamblado de referencia o código personalizado de asignaciones 
La acción de transformación también admite asignaciones o transformaciones con referencia a ensamblados externos. Esta funcionalidad permite llamadas a código .NET personalizado directamente desde asignaciones XSLT. Estos son los requisitos previos para usar ensamblados en asignaciones.

* La asignación y el ensamblado al que se hace referencia desde la asignación debe [cargarse en la cuenta de integración](./logic-apps-enterprise-integration-maps.md). 

  > [!NOTE]
  > La asignación y el ensamblado deben cargarse en un orden específico. Debe cargar el ensamblado antes de cargar la asignación que hace referencia al ensamblado.

* Además, la asignación debe tener estos atributos y una sección CDATA que contenga la llamada al código de ensamblado:

    * **name** es el nombre de ensamblado personalizado.
    * **namespace** es el espacio de nombres en el ensamblado que incluye el código personalizado.

  En este ejemplo se muestra una asignación que hace referencia a un ensamblado denominado "XslUtilitiesLib" y llama al método `circumreference` desde el ensamblado.

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="urn:my-scripts">
  <msxsl:script language="C#" implements-prefix="user">
    <msxsl:assembly name="XsltHelperLib"/>
    <msxsl:using namespace="XsltHelpers"/>
    <![CDATA[public double circumference(int radius){ XsltHelper helper = new XsltHelper(); return helper.circumference(radius); }]]>
  </msxsl:script>
  <xsl:template match="data">
     <circles>
        <xsl:for-each select="circle">
            <circle>
                <xsl:copy-of select="node()"/>
                    <circumference>
                        <xsl:value-of select="user:circumference(radius)"/>
                    </circumference>
            </circle>
        </xsl:for-each>
     </circles>
    </xsl:template>
    </xsl:stylesheet>
  ```


### <a name="byte-order-mark"></a>Marca BOM
De forma predeterminada, la respuesta de la transformación se inicia con la marca BOM. Puede tener acceso a esta funcionalidad solo mientras trabaja en el editor de la vista Código. Para deshabilitar esta funcionalidad, especifique `disableByteOrderMark` para la propiedad `transformOptions`:

```json
"Transform_XML": {
    "inputs": {
        "content": "@{triggerBody()}",
        "integrationAccount": {
            "map": {
                "name": "TestMap"
            }
        },
        "transformOptions": "disableByteOrderMark"
    },
    "runAfter": {},
    "type": "Xslt"
}
```





## <a name="learn-more"></a>Más información
* [Más información acerca de Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md "Información sobre Enterprise Integration Pack")  
* [Más información sobre las asignaciones](../logic-apps/logic-apps-enterprise-integration-maps.md "Información sobre las asignaciones de integración empresarial")  