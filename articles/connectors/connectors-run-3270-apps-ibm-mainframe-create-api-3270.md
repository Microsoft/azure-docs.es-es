---
title: Conexión a aplicaciones de 3270 en sistemas centrales de IBM
description: Integración y automatización de aplicaciones controladas por pantallas 3270 con Azure mediante Azure Logic Apps y el conector de IBM 3270
services: logic-apps
ms.suite: integration
author: ChristopherHouser
ms.author: chrishou
ms.reviewer: estfan, valthom
ms.topic: article
ms.date: 03/06/2019
tags: connectors
ms.openlocfilehash: 41e3f1ff430293ebc7b3828a0fd7090923fc209c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281487"
---
# <a name="integrate-3270-screen-driven-apps-on-ibm-mainframes-with-azure-by-using-azure-logic-apps-and-ibm-3270-connector"></a>Integración de aplicaciones controladas por pantallas 3270 en sistemas centrales de IBM con Azure mediante Azure Logic Apps y el conector de IBM 3270

> [!NOTE]
> Este conector está en [*versión preliminar pública*](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 

Con Azure Logic Apps y el conector de IBM 3270, puede ejecutar y tener acceso a aplicaciones del sistema central de IBM que suele impulsar navegando a través de pantallas del emulador 3270. De este modo, puede integrar sus aplicaciones del sistema central de IBM con Azure, Microsoft y otras aplicaciones, servicios y sistemas creando flujos de trabajo automatizados con Azure Logic Apps. El conector se comunica con los sistemas centrales de IBM mediante el protocolo TN3270 y está disponible en todas las regiones de Azure Logic Apps, salvo para Azure Government y Azure China 21Vianet. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

En este artículo se describen estos aspectos para usar el conector 3270: 

* Por qué se usa el conector de IBM 3270 en Azure Logic Apps y cómo ejecuta dicho conector aplicaciones controladas por pantallas 3270

* Requisitos previos y configuración de uso del conector 3270

* Los pasos para agregar acciones del conector 3270 a su aplicación lógica

## <a name="why-use-this-connector"></a>¿Por qué se debe usar este conector?

Para obtener acceso a aplicaciones en sistemas centrales de IBM, normalmente usa un emulador de terminal 3270, al que se suele llamar "pantalla verde". Este método es una forma demostrada con el tiempo, pero tiene limitaciones. Aunque Host Integration Server (HIS) le ayuda a trabajar directamente con estas aplicaciones, en ocasiones, es posible que la pantalla y la lógica de negocios no se puedan separar. O bien, puede que ya no tenga información sobre el funcionamiento de las aplicaciones host.

Para ampliar estos escenarios, el conector de IBM 3270 de Azure Logic Apps funciona con la herramienta de diseño 3270, la cual usa para registrar o "capturar" las pantallas de host utilizadas para una tarea concreta, definir el flujo de navegación para dicha tarea a través de su aplicación del sistema central y definir los métodos con parámetros de entrada y salida para esa tarea. La herramienta de diseño convierte dicha información en metadatos que usa el conector 3270 al llamar a una acción que representa esa tarea de su aplicación lógica.

Una vez que genere el archivo de metadatos a partir de la herramienta de diseño, agregará ese archivo a una cuenta de integración de Azure. De ese modo, su aplicación lógica podrá obtener acceso a los metadatos de su aplicación al agregar una acción del conector 3270. El conector lee el archivo de metadatos en su cuenta de integración, controla la navegación a través de las pantallas 3270 y presenta de forma dinámica los parámetros para la acción del conector 3270. A partir de ese momento, puede proporcionar datos a la aplicación host y el conector devuelve los resultados a su aplicación lógica. De este modo, puede integrar sus aplicaciones heredadas con Azure, Microsoft y otras aplicaciones, servicios y sistemas que Azure Logic Apps admite.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Se recomienda: Un [Entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md) 

  Puede seleccionar este entorno como ubicación para crear y ejecutar su aplicación lógica. Un ISE proporciona acceso desde su aplicación lógica a recursos que se encuentran protegidos dentro de redes virtuales de Azure.

* La aplicación lógica que se va a usar para automatizar y ejecutar su aplicación controlada por pantallas 3270

  El conector de IBM 3270 no tiene desencadenadores, por lo que debe usar otro desencadenador para iniciar la aplicación lógica, como **Periodicidad**. A continuación, puede agregar acciones del conector 3270. Para empezar, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). 
  Si usa un ISE, selecciónelo como ubicación de la aplicación lógica.

* [Descargue e instale la herramienta de diseño 3270](https://aka.ms/3270-design-tool-download).
El único requisito previo es [Microsoft .NET Framework 4.6.1](https://aka.ms/net-framework-download).

  Esta herramienta le ayuda a registrar las pantallas, rutas de acceso de navegación, métodos y parámetros para las tareas de la aplicación que agrega y ejecuta como acciones del conector 3270. La herramienta genera un archivo Host Integration Designer XML (HIDX) que proporciona los metadatos necesarios para el conector que se va a usar para impulsar su aplicación del sistema central.
  
  Una vez descargada e instalada esta herramienta, siga estos pasos para conectarse a su host:

  1. Abra la herramienta de diseño 3270. En el menú **Sesión**, seleccione **Sesiones de host**.
  
  1. Proporcione información sobre el servidor host TN3270.

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md), que es el lugar donde almacena su archivo HIDX como asignación para que su aplicación lógica pueda tener acceso a los metadatos y las definiciones de métodos de ese archivo. 

  Asegúrese de que su cuenta de integración se vincula a la aplicación lógica que usa. Asimismo, si usa un ISE, asegúrese de que la ubicación de su cuenta de integración es el mismo ISE utilizado por su aplicación lógica.

* Acceso al servidor TN3270 que hospeda su aplicación del sistema central

<a name="define-app-metadata"></a>

## <a name="create-metadata-overview"></a>Información general sobre la creación de metadatos

En una aplicación controlada por pantallas 3270, las pantallas y los campos de datos son únicos para sus escenarios, por lo que el conector 3270 necesita esta información sobre su aplicación, que puede proporcionar como metadatos. Estos metadatos describen información que ayuda a la aplicación lógica a identificar y reconocer pantallas, describe cómo navegar entre pantallas, dónde escribir los datos y dónde esperar resultados. Para especificar y generar estos metadatos, debe usar la herramienta de diseño 3270, que le guía por estos *modos* específicos o fases, como se describe posteriormente de forma más detallada:

* **Captura**: en este modo, registra las pantallas necesarias para completar una tarea específica con su aplicación del sistema central, por ejemplo, obteniendo un saldo bancario.

* **Navegación**: en este modo, especifica el plan o la ruta de acceso para saber cómo navegar por las pantallas de su aplicación del sistema central para la tarea concreta.

* **Métodos**: en este modo, define el método, por ejemplo, `GetBalance`, que describe la ruta de acceso de navegación de la pantalla. También elige los campos de cada pantalla que pasan a ser los parámetros de entrada y salida del método.

### <a name="unsupported-elements"></a>Elementos no compatibles

La herramienta de diseño no admite estos elementos:

* Asignaciones Basic Mapping Support (BMS) de IBM parciales: si importa una asignación BMS, la herramienta de diseño omitirá las definiciones de pantalla parciales.
* Parámetros de entrada/salida: no puede definir los parámetros de entrada/salida.
* Procesamiento de menús: no admitido durante la versión preliminar
* Procesamiento de matrices: no admitido durante la versión preliminar

<a name="capture-screens"></a>

## <a name="capture-screens"></a>Captura de pantallas

En este modo, marca un elemento en cada pantalla 3270 que identifica dicha pantalla de forma exclusiva. Por ejemplo, podría especificar una línea de texto o un conjunto de condiciones más complejo, como texto específico y un campo no vacío. Puede registrar estas pantallas a través de una conexión dinámica al servidor host, o bien importar esta información desde una asignación Basic Mapping Support (BMS) de IBM. La conexión dinámica usa un emulador TN3270 para conectarse al host. Cada acción del conector debe asignarse a una sola tarea que se inicia con la conexión a su sesión y termina con la desconexión de su sesión.

1. Si aún no lo ha hecho, abra la herramienta de diseño 3270. En la barra de herramientas, elija **Captura** para tener acceso al modo Captura.

1. Para iniciar la grabación, presione la tecla F5 o, en el menú **Grabación**, seleccione **Iniciar grabación**. 

1. En el menú **Sesión**, seleccione **Conectar**.

1. En el panel **Captura**, a partir de la primera pantalla de su aplicación, consulte su aplicación para la tarea específica que está grabando.

1. Una vez que finalice la tarea, cierre la sesión de su aplicación como hace normalmente.

1. En el menú **Sesión**, seleccione **Desconectar**.

1. Para iniciar la grabación, presione las teclas Mayús + F5 o, en el menú **Grabación**, seleccione **Detener grabación**.

   Después de capturar las pantallas para una tarea, la herramienta de diseñador muestra miniaturas que representan esas pantallas. Algunas notas sobre estas miniaturas:

   * Incluida con sus pantallas capturadas, tiene una pantalla llamada "Vacía".

     La primera vez que se conecta a [CICS](https://www.ibm.com/it-infrastructure/z/cics), debe enviar la clave "Borrar" antes de poder escribir el nombre de la transacción que desea ejecutar. La pantalla donde envía la clave "Borrar" no tiene ningún *atributo de reconocimiento*, como un título, el cual puede agregar mediante el editor de reconocimiento de la pantalla. Para representar esta pantalla, las miniaturas incluyen una pantalla llamada "Vacía". Posteriormente podrá usar esta pantalla para representar la pantalla donde escribe el nombre de la transacción.

   * De forma predeterminada, el nombre de una pantalla capturada usa la primera palabra de la pantalla. Si ese nombre ya existe, la herramienta de diseño anexará el nombre con un guion bajo y un número, por ejemplo, "WBGB" y "WBGB_1".

1. Para asignar un nombre más significativo a una pantalla capturada, siga estos pasos:

   1. En el panel **Pantallas de host**, seleccione la pantalla cuyo nombre desee cambiar.

   1. En el mismo panel, cerca de la parte inferior, vea la propiedad **Nombre de pantalla**.

   1. Cambie el nombre actual de la pantalla por uno más descriptivo.

1. Ahora, especifique los campos para identificar cada pantalla.

   Con el flujo de datos de 3270, las pantallas no tienen identificadores predeterminados, por lo que debe seleccionar texto único en cada una de las pantallas. Para escenarios complejos, puede especificar varias condiciones, por ejemplo, texto único y un campo con una condición concreta.

Una vez que termine de seleccionar los campos de reconocimiento, pase al modo siguiente.

### <a name="conditions-for-identifying-repeated-screens"></a>Condiciones para identificar pantallas repetidas

Para que el conector navegue y distinga entre las pantallas, normalmente encontrará texto único en una pantalla que puede usar como identificador entre las pantallas capturadas. Para las pantallas repetidas, podrían ser necesarios más métodos de identificación. Por ejemplo, supongamos que tiene dos pantallas cuyo aspecto es similar, salvo que una devuelve un valor válido y la otra un mensaje de error.

En la herramienta de diseño, puede agregar *atributos de reconocimiento*, por ejemplo, un título de pantalla como "Obtener saldo de cuenta", mediante el editor de reconocimiento de la pantalla. Si tiene una ruta de acceso bifurcada y ambas ramas devuelven la misma pantalla, pero con otros resultados, necesitará otros atributos de reconocimiento. En tiempo de ejecución, el conector usa estos atributos para determinar la rama y la bifurcación actuales. Estas son las condiciones que puede usar:

* Valor específico: este valor coincide con la cadena especificada en la ubicación especificada.
* NO un valor específico: este valor no coincide con la cadena especificada en la ubicación especificada.
* Vacío: este campo está vacío.
* NO vacío: este campo no está vacío.

Para obtener más información, consulte el [plan de navegación de ejemplo](#example-plan) más adelante en este tema.

<a name="define-navigation"></a>

## <a name="define-navigation-plans"></a>Definición de planes de navegación

En este modo, define el flujo o los pasos para navegar por las pantallas de su aplicación del sistema central para su tarea específica. Por ejemplo, en ocasiones, podría tener más de una ruta de acceso que su aplicación puede tomar donde una ruta de acceso produce el resultado correcto y la otra un error. Para cada pantalla, especifique las pulsaciones de teclas necesarias para pasar a la pantalla siguiente, como `CICSPROD <enter>`.

> [!TIP]
> Si automatiza varias tareas que usan las mismas pantallas de conexión y desconexión, la herramienta de diseño proporcionará tipos de plan de conexión y desconexión especiales. Al definir estos planes, puede agregarlos al principio y al final de su plan de navegación.

### <a name="guidelines-for-plan-definitions"></a>Directrices para las definiciones de planes

* Incluya todas las pantallas, empezando con la conexión y terminando con la desconexión.

* Puede crear un plan independiente o usar los planes de conexión y desconexión, que le permiten reutilizar una serie de pantallas comunes a todas sus transacciones.

  * La última pantalla de su plan de conexión debe ser la misma pantalla que la primera pantalla de su plan de navegación.

  * La primera pantalla de su plan de desconexión debe ser la misma pantalla que la última pantalla de su plan de navegación.

* Sus pantallas capturadas podrían incluir muchas pantallas repetidas, de modo que seleccione y use solo una instancia de cualquier pantalla repetida de su plan. Estos son algunos ejemplos de pantallas repetidas:

  * La pantalla de inicio de sesión, por ejemplo, la pantalla **MSG-10**.
  * La pantalla principal para CICS
  * La pantalla "Borrar" o **Vacía**

<a name="create-plans"></a>

### <a name="create-plans"></a>Creación de planes

1. En la barra de herramientas de la herramienta de diseño 3270, elija **Navegación** para tener acceso al modo Navegación.

1. Para iniciar su plan, en el panel **Navegación**, elija **Nuevo plan**.

1. En **Elegir nuevo nombre del plan**, escriba un nombre para su plan. En la lista **Tipo**, seleccione el tipo de plan:

   | Tipo de plan | Descripción |
   |-----------|-------------|
   | **Proceso** | Para los planes independientes o combinados |
   | **Conexión** | Para los planes de conexión |
   | **Desconexión** | Para los planes de desconexión |
   |||

1. En el panel **Pantallas de host**, arrastre las miniaturas capturadas a la superficie del plan de navegación en el panel **Navegación**.

   Para representar la pantalla en blanco donde escribe el nombre de la transacción, use la pantalla "Vacía".

1. Organice las pantallas en el orden descrito por la tarea que está definiendo.

1. Para definir la ruta de acceso de flujo entre pantallas, incluidas bifurcaciones y combinaciones, en la barra de herramientas de la herramienta de diseño, elija **Flujo**.

1. Elija la primera pantalla del flujo. Arrástrela y establezca una relación entre esta y la siguiente pantalla del flujo.

1. Para cada pantalla, proporcione los valores de las propiedades **Clave AID** (Attention Identifier) y **Texto fijo**. Esta última mueve el flujo a la pantalla siguiente.

   Es posible que tenga solo la clave AID, o bien esta y el texto fijo.

Una vez finalizado su plan de navegación, puede [definir métodos en el siguiente modo](#define-method).

<a name="example-plan"></a>

### <a name="example"></a>Ejemplo

En este ejemplo, supongamos que ejecuta una transacción CICS llamada "WBGB" que tiene estos pasos: 

* En la primera pantalla, escribe un nombre y una cuenta.
* En la segunda pantalla, obtiene el saldo de cuenta.
* Sale a la pantalla "Vacía".
* Cierra la sesión de CICS en la pantalla "MSG-10".

Supongamos también que repite estos pasos, pero los datos que escribe son incorrectos para poder capturar la pantalla que muestra el error. Estas son las pantallas que captura:

* MSG-10
* Pantalla principal para CICS
* Vacío
* WBGB_1 (entrada)
* WBGB_2 (error)
* Empty_1
* MSG-10_1

Aunque muchas pantallas aquí obtienen nombres únicos, algunas son las mismas, por ejemplo, "MSG-10" y "Vacía". Para una pantalla repetida, use solo una instancia para esa pantalla de su plan. En estos ejemplos se muestra el aspecto que podrían tener un plan independiente, un plan de conexión, un plan de desconexión y un plan combinado:

* Plan independiente

  ![Plan de navegación independiente](./media/connectors-create-api-3270/standalone-plan.png)

* Plan de conexión

  ![Plan de conexión](./media/connectors-create-api-3270/connect-plan.png)

* Plan de desconexión

  ![Plan de desconexión](./media/connectors-create-api-3270/disconnect-plan.png)

* Plan combinado

  ![Plan combinado](./media/connectors-create-api-3270/combined-plan.png)

#### <a name="example-identify-repeated-screens"></a>Ejemplo: Identificación de pantallas repetidas

Para que el conector navegue y distinga entre las pantallas, normalmente encontrará texto único en una pantalla que puede usar como identificador entre las pantallas capturadas. Para las pantallas repetidas, podrían ser necesarios más métodos de identificación. El plan de ejemplo tiene una bifurcación donde puede obtener pantallas con un aspecto similar. Una pantalla devuelve el saldo de una cuenta, mientras que la otra devuelve un mensaje de error.

La herramienta de diseño le permite agregar atributos de reconocimiento, por ejemplo, el título de pantalla "Obtener saldo de cuenta", mediante el editor de reconocimiento de la pantalla. En el caso con pantallas similares, necesita otros atributos. En tiempo de ejecución, el conector usa estos atributos para determinar la rama y la bifurcación.

* En la rama que devuelve una entrada válida, que es la pantalla con el saldo de cuenta, puede agregar un campo con la condición "no vacía".

* En la rama que devuelve un error, puede agregar un campo con la condición "vacía".

<a name="define-method"></a>

## <a name="define-methods"></a>Definición de métodos

En este modo, define un método asociado a su plan de navegación. Para cada parámetro de método, especifica el tipo de datos, como cadena, entero, fecha u hora, etc. Cuando haya terminado, podrá probar su método en el host en vivo y confirmar que funciona según lo previsto. A continuación, genera el archivo de metadatos o el archivo Host Integration Designer XML (HIDX), que ahora tiene las definiciones de método que se van a usar para crear y ejecutar una acción para el conector de IBM 3270.

1. En la barra de herramientas de la herramienta de diseño 3270, elija **Métodos** para tener acceso al modo Métodos. 

1. En el panel **Navegación**, seleccione la pantalla que tenga los campos de entrada que desea.

1. Para agregar el primer parámetro de entrada del método, siga estos pasos:

   1. En el panel **Captura**, en la pantalla del emulador 3270, elija todo el campo, no solo el texto del campo, que desea como primera entrada.

      > [!TIP]
      > Para mostrar todos los campos y asegurarse de seleccionar el campo completo, en el menú **Ver**, seleccione **Todos los campos**.

   1. En la barra de herramientas de la herramienta de diseño, elija **Campo de entrada**. 

   Para agregar más parámetros de entrada, repita los pasos anteriores para cada parámetro.

1. Para agregar el primer parámetro de salida del método, siga estos pasos:

   1. En el panel **Captura**, en la pantalla del emulador 3270, elija todo el campo, no solo el texto del campo, que desea como primera salida.

      > [!TIP]
      > Para mostrar todos los campos y asegurarse de seleccionar el campo completo, en el menú **Ver**, seleccione **Todos los campos**.

   1. En la barra de herramientas de la herramienta de diseño, elija **Campo de salida**.

   Para agregar más parámetros de salida, repita los pasos anteriores para cada parámetro.

1. Una vez que agregue todos los parámetros del método, defina estas propiedades para cada parámetro:

   | Nombre de propiedad | Valores posibles | 
   |---------------|-----------------|
   | **Tipo de datos** | Byte, Date Time, Decimal, Int, Long, Short y String |
   | **Técnica de relleno de campos** | Los parámetros admiten estos tipos de relleno, usando espacios en blanco en caso necesario: <p><p>- **Type**: escriba caracteres de forma secuencial en el campo. <p>- **Fill**: reemplace el contenido del campo por caracteres, usando espacios en blanco en caso necesario. <p>- **EraseEofType**: borre el campo y, a continuación, escriba caracteres de forma secuencial en el campo. |
   | **Cadena de formato** | Algunos tipos de datos de parámetro usan una cadena de formato, que informa al conector 3270 de cómo se debe convertir el texto de la pantalla en un tipo de datos de .NET: <p><p>- **DateTime**: la cadena de formato DateTime sigue las [cadenas de formato de fecha y hora personalizadas de .NET](/dotnet/standard/base-types/custom-date-and-time-format-strings). Por ejemplo, la fecha `06/30/2019` usa la cadena de formato `MM/dd/yyyy`. <p>- **Decimal**: la cadena de formato decimal usa la [cláusula COBOL Picture](https://www.ibm.com/support/knowledgecenter/SS6SG3_5.2.0/com.ibm.cobol52.ent.doc/PGandLR/ref/rlddepic.html). Por ejemplo, el número `100.35` usa la cadena de formato `999V99`. |
   |||

## <a name="save-and-view-metadata"></a>Guardado y visualización de metadatos

Una vez que defina el método, pero antes de que lo pruebe, guarde toda la información que ha definido hasta el momento en un archivo RAP (.rap).
Puede guardarla en este archivo RAP en cualquier momento durante cualquier modo. La herramienta de diseño también incluye un archivo RAP de ejemplo que puede abrir y revisar yendo a la carpeta de instalación de esta en esta ubicación y abriendo el archivo "WoodgroveBank.rap":

`..\Program Files\Microsoft Host Integration Server - 3270 Design Tool\SDK\WoodgroveBank.rap`

Sin embargo, si intenta guardar los cambios en el archivo RAP de ejemplo o generar a partir de este un archivo HIDX mientras el archivo permanece en la carpeta de instalación de la herramienta de diseño, es posible que reciba el error "acceso denegado". La herramienta de diseño se instala de forma predeterminada en su carpeta Archivos de programa sin permisos elevados. Si recibe un error, pruebe una de estas soluciones:

* Copie el archivo de ejemplo en otra ubicación.
* Ejecute la herramienta de diseño como administrador.
* Conviértase en el propietario de la carpeta del SDK.

## <a name="test-your-method"></a>Comprobación del método

1. Para ejecutar su método en el host en vivo, mientras sigue en modo Métodos, presione la tecla F5 o, en la barra de herramientas de la herramienta de diseño, elija **Ejecutar**.

   > [!TIP]
   > Puede cambiar los modos en cualquier momento. En el menú **Archivo**, seleccione **Modo** y, a continuación, seleccione el modo que desee.

1. Escriba los valores de los parámetros y elija **Aceptar**.

1. Para continuar en la siguiente pantalla, elija **Siguiente**.

1. Cuando termine, elija **Listo**, que muestra sus valores del parámetro de salida.

<a name="add-metadata-integration-account"></a>

## <a name="generate-and-upload-hidx-file"></a>Generación y carga del archivo HIDX

Cuando esté listo, genere el archivo HIDX para que pueda cargarlo en su cuenta de integración. La herramienta de diseño 3270 crea el archivo HIDX en una nueva subcarpeta donde guardó su archivo RAP.

1. En la barra de herramientas de la herramienta de diseño 3270, elija **Generar código**.

1. Vaya a la carpeta que incluye su archivo RAP y abra la subcarpeta que la herramienta creó después de generar su archivo HIDX. Confirme que la herramienta creó el archivo HIDX.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y busque su cuenta de integración.

1. Agregue su archivo HIDX como asignación a su cuenta de integración [siguiendo estos pasos similares para agregar asignaciones](../logic-apps/logic-apps-enterprise-integration-liquid-transform.md), pero, cuando seleccione el tipo de asignación, seleccione **HIDX**.

Más adelante en este tema, al agregar una acción de IBM 3270 a su aplicación lógica por primera vez, se le pedirá que cree una conexión entre su aplicación lógica y el servidor host proporcionando información sobre la conexión, como los nombres de su cuenta de integración y servidor host. Una vez creada la conexión, puede seleccionar su archivo HIDX agregado anteriormente, el método que se va a ejecutar y los parámetros que se van a usar.

Al completar todos estos pasos, podrá usar la acción que crea en su aplicación lógica para conectarse a su sistema central de IBM, impulsar pantallas para su aplicación, especificar datos, devolver resultados, etc. También podrá seguir agregando otras acciones a su aplicación lógica para la integración con otras aplicaciones, servicios y sistemas.

<a name="run-action"></a>

## <a name="run-ibm-3270-actions"></a>Ejecución de acciones de IBM 3270

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. En el último paso donde desea agregar una acción, elija **Nuevo paso** y seleccione **Agregar una acción**. 

1. En el cuadro de búsqueda, elija **Enterprise**. En el cuadro de búsqueda, escriba "3270" como el filtro. En la lista de acciones, seleccione esta acción: **Ejecuta un programa del sistema central a través de una conexión TN3270**

   ![Selección de la acción 3270](./media/connectors-create-api-3270/select-3270-action.png)

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. 
   Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

1. Si aún no existe ninguna conexión, proporcione la información necesaria para su conexión y elija **Crear**.

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de la conexión** | Sí | <*connection-name*> | El nombre de la conexión |
   | **Id. de cuenta de integración** | Sí | <*integration-account-name*> | Nombre de su cuenta de integración |
   | **Dirección URL de SAS de la cuenta de integración** | Sí | <*integration-account-SAS-URL*> | Dirección URL de la Firma de acceso compartido (SAS) de su cuenta de integración, que puede generar a partir de la configuración de dicha cuenta en Azure Portal. <p>1. En el menú de la cuenta de integración, en **Configuración**, seleccione **Dirección URL de devolución de llamada**. <br>2. En el panel derecho, copie el valor **Dirección URL de devolución de llamada generada**. |
   | **Server** | Sí | <*TN3270-server-name*> | Nombre del servidor para su servicio TN3270 |
   | **Puerto** | No | <*TN3270-server-port*> | Puerto utilizado por el servidor TN3270. Si se deja en blanco, el conector usa `23` como valor predeterminado. |
   | **Tipo de dispositivo** | No | <*IBM-terminal-model*> | Nombre o número del modelo para el terminal de IBM que se va a emular. Si se deja en blanco, el conector usa valores predeterminados. |
   | **Página de códigos** | No | <*code-page-number*> | Número de la página de códigos para el host. Si se deja en blanco, el conector usa `37` como valor predeterminado. |
   | **Nombre de la unidad lógica** | No | <*logical-unit-name*> | Nombre de la unidad lógica específico que se va a solicitar desde el host |
   | **¿Quiere habilitar SSL?** | No | On (Activado) u Off (Desactivado) | Activar o desactivar el cifrado TLS. |
   | **¿Quiere validar un certificado SSL host?** | No | On (Activado) u Off (Desactivado) | Active o desactive la validación del certificado del servidor. |
   ||||

   Por ejemplo:

   ![Propiedades de la conexión](./media/connectors-create-api-3270/connection-properties.png)

1. Proporcione la información necesaria para la acción:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de HIDX** | Sí | <*HIDX-file-name*> | Seleccione el archivo HIDX 3270 que desea usar. |
   | **Nombre de método** | Sí | <*method-name*> | Seleccione el método en el archivo HIDX que desea usar. Una vez seleccionado un método, aparece la lista **Agregar nuevo parámetro** para que pueda seleccionar los parámetros que se van a usar con ese método. |
   ||||

   Por ejemplo:

   **Selección del archivo HIDX**

   ![Selección del archivo HIDX](./media/connectors-create-api-3270/select-hidx-file.png)

   **Selección del método**

   ![Selección del método](./media/connectors-create-api-3270/select-method.png)

   **Selección de los parámetros**

   ![Selección de parámetros](./media/connectors-create-api-3270/add-parameters.png)

1. Cuando haya terminado, guarde y ejecute la aplicación lógica.

   Cuando finalice la ejecución de la aplicación lógica, aparecerán los pasos de la ejecución. 
   Los pasos correctos muestran marcas de verificación, mientras que los pasos infructuosos muestran la letra "X".

1. Para revisar las entradas y salidas de cada paso, expanda ese paso.

1. Para revisar las salidas, elija **Ver salidas sin procesar**.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/si3270/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

