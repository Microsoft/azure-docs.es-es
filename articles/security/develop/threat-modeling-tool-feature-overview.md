---
title: 'Introducción a la característica Microsoft Threat Modeling Tool: Azure'
description: Más información acerca de todas las características disponibles en Threat Modeling Tool como, por ejemplo, la vista de análisis y los informes.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 08/17/2017
ms.openlocfilehash: e405ea35dcaa33ac7c8227385e4aaff86d9244cb
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87540009"
---
# <a name="threat-modeling-tool-feature-overview"></a>Información general de las características de Threat Modeling Tool

Threat Modeling Tool puede ayudarle con sus necesidades de modelado de amenazas. Para una introducción básica a la herramienta, consulte [Introducción a Threat Modeling Tool](threat-modeling-tool-getting-started.md).

> [!NOTE]
>Threat Modeling Tool se actualiza con frecuencia, de modo que revise esta guía a menudo para ver nuestras últimas características y mejoras.

Para abrir una página en blanco, seleccione **Crear un modelo**.

![Página en blanco](./media/threat-modeling-tool-feature-overview/tmtstart.png)

Para ver las características actualmente disponibles en la herramienta, utilice el modelo de amenazas creado por nuestro equipo en el ejemplo de [Introducción](threat-modeling-tool-getting-started.md).

![Modelo de amenaza básico](./media/threat-modeling-tool-feature-overview/basictmt.png)

## <a name="navigation"></a>Navegación

Antes de profundizar en las características integradas, veamos los componentes principales de la herramienta.

### <a name="menu-items"></a>Elementos de menú

La experiencia es similar a la de otros productos de Microsoft. Vamos a revisar los elementos del menú de nivel superior.

![Elementos de menú](./media/threat-modeling-tool-feature-overview/menuitems.png)

| Etiqueta                               | Detalles      |
| --------------------------------------- | ------------ |
| **Archivo** | <ul><li>Abrir, guardar y cerrar archivos</li><li>Iniciar y cerrar sesión en cuentas de OneDrive.</li><li>Compartir vínculos (vista y edición).</li><li>Ver información de archivo.</li><li>Aplicar una plantilla nueva a modelos existentes.</li></ul> |
| **Edición** | Acciones de deshacer y rehacer, además de copiar, pegar y eliminar. |
| **Vista** | <ul><li>Cambiar entre las vistas **Análisis** y **Diseño**.</li><li>Abrir las ventanas cerradas (por ejemplo, galerías de símbolos, propiedades de elementos y mensajes).</li><li>Restablecer el diseño a la configuración predeterminada.</li></ul> |
| **Diagrama** | Agregar y eliminar diagramas y navegar a través de pestañas de diagramas. |
| **Informes** | Crear informes HTML para compartir con otros usuarios. |
| **Ayuda** | Encontrar guías para ayudarle a usar la herramienta. |

Los símbolos son accesos directos para los menús de nivel superior:

| Símbolo                               | Detalles      |
| --------------------------------------- | ------------ |
| **Abrir** | Abre un nuevo archivo. |
| **Guardar** | Guarda el archivo actual. |
| **Diseño** | Abre la vista **Diseño**, donde puede crear modelos. |
| **Análisis** | Muestra las amenazas generadas y sus propiedades. |
| **Agregar diagrama** | Agrega un nuevo diagrama (similar a las nuevas pestañas de Excel). |
| **Eliminar diagrama** | Elimina el diagrama actual. |
| **Copiar/Cortar/Pegar** | Copia, corta y pega elementos. |
| **Deshacer/Rehacer** | Deshace y rehace acciones. |
| **Acercar/Alejar** | Acerca y aleja el diagrama para obtener una mejor vista. |
| **Comentarios** | Abre el Foro de MSDN. |

### <a name="canvas"></a>Lienzo

El lienzo es el espacio donde arrastrar y colocar elementos. Arrastrar y colocar es la manera más rápida y eficaz para crear modelos. También puede hacer clic con el botón derecho y seleccionar elementos en el menú para agregar versiones genéricas de elementos, como se muestra:

#### <a name="drop-the-stencil-on-the-canvas"></a>Colocar la galería de símbolos en el lienzo

![Colocar en el lienzo](./media/threat-modeling-tool-feature-overview/canvasdrop1.png)

#### <a name="select-the-stencil"></a>Seleccionar la galería de símbolos

![Propiedades del elemento](./media/threat-modeling-tool-feature-overview/canvasdrop2.png)

### <a name="stencils"></a>Galerías de símbolos

En función de la plantilla que seleccione, puede encontrar todas las galerías de símbolos disponibles para su uso. Si no encuentra los elementos adecuados, utilice otra plantilla. O bien, puede modificar una plantilla para ajustarla a sus necesidades. Por lo general, puede encontrar una combinación de categorías como las siguientes:

| Nombre de la galería de símbolos                               | Detalles      |
| --------------------------------------- | ------------ |
| **Proceso** | Aplicaciones, complementos de explorador, subprocesos, máquinas virtuales |
| **Interactivo externo** | Proveedores de autenticación, exploradores, usuarios, aplicaciones web |
| **Almacén de datos** | Caché, almacenamiento, archivos de configuración, bases de datos, registro |
| **Flujo de datos** | Binario, ALPC, HTTP, HTTPS/TLS/SSL, IOCTL, IPSec, Canalización con nombre, RPC/DCOM, SMB, UDP |
| **Línea de confianza/Límite de borde** | Redes corporativas, Internet, máquina, espacio aislado, modo Kernel/Usuario |

### <a name="notesmessages"></a>Notas/Mensajes

| Componente                               | Detalles      |
| --------------------------------------- | ------------ |
| **Mensajes** | Lógica de herramienta interna que alerta a los usuarios cada vez que hay un error, por ejemplo, cuando no hay flujos de datos entre los elementos. |
| **Notas** | Notas manuales que los equipos de ingeniería agregan al archivo durante el proceso de diseño y revisión. |

### <a name="element-properties"></a>Propiedades del elemento

Las propiedades del elemento varían en función de los elementos seleccionados. A excepción de los límites de confianza, todos los demás elementos contienen tres selecciones generales:

| Propiedad del elemento                               | Detalles      |
| --------------------------------------- | ------------ |
| **Nombre** | Resulta útil para asignar nombres a los procesos, almacenes, elementos interactivos y flujos para que se reconozcan con facilidad. |
| **Fuera de ámbito** | Si se selecciona, el elemento se saca de la matriz de generación de amenazas (no se recomienda). |
| **Motivo de elegir Fuera de ámbito** | Campo de justificación para que los usuarios sepan por qué se seleccionó la opción de fuera de ámbito. |

Se cambian las propiedades de cada categoría de elemento. Seleccione cada elemento para inspeccionar las opciones disponibles. O bien, puede abrir la plantilla para más información. Vamos a revisar las características.

## <a name="welcome-screen"></a>Pantalla principal

Cuando se abre la aplicación, verá la pantalla de **Bienvenida**.

### <a name="open-a-model"></a>Abrir un modelo

Mantenga el ratón sobre **Abrir un modelo** para mostrar dos opciones: **Abrir desde este equipo** y **Abrir desde OneDrive**. La primera opción abre la pantalla **Abrir archivo**. La segunda opción le guiará por el proceso de inicio de sesión en OneDrive. Tras la autenticación correcta, puede seleccionar los archivos y carpetas.

![Abrir modelo](./media/threat-modeling-tool-feature-overview/openmodel.png)

![Abrir desde el equipo o desde OneDrive](./media/threat-modeling-tool-feature-overview/openmodel2.png)

### <a name="feedback-suggestions-and-issues"></a>Comentarios, sugerencias y problemas

Cuando se selecciona **Comentarios, sugerencias y problemas**, se abre el foro de MSDN para herramientas de SDL. Puede leer lo que dicen otros usuarios acerca de la herramienta, incluidas nuevas ideas y soluciones.

![Comentarios](./media/threat-modeling-tool-feature-overview/feedback.png)

## <a name="design-view"></a>Vista de diseño

Al abrir o crear un nuevo modelo, se abre la vista **Diseño**.

### <a name="add-elements"></a>Agregar elementos

Puede agregar elementos en la cuadrícula de dos maneras:

- **Arrastrar y colocar**: arrastre el elemento deseado a la cuadrícula. A continuación, utilice las propiedades del elemento para proporcionar información adicional.
- **Hacer clic con el botón derecho**: haga clic con el botón derecho en cualquier parte de la cuadrícula y seleccione elementos en el menú desplegable. Aparecerá una representación genérica del elemento seleccionado en la pantalla.

### <a name="connect-elements"></a>Conectar elementos

Puede conectar elementos de dos maneras:

- **Arrastrar y colocar**: arrastre el flujo de datos deseado a la cuadrícula y conecte ambos extremos a los elementos correspondientes.
- **Hacer clic + Mayús**: haga clic en el primer elemento (enviar datos) y mantenga presionada la tecla Mayús y luego seleccione el segundo elemento (recibir datos). Haga clic con el botón derecho y seleccione **Conectar**. Si usa un flujo de datos bidireccional, el orden no es tan importante.

### <a name="properties"></a>Propiedades

 Para ver las propiedades que pueden modificarse en las galerías de símbolos, seleccione la galería de símbolos y la información se rellena en consecuencia. En el ejemplo siguiente se muestra la situación antes y después de arrastrar una galería de símbolos **Base de datos** al diagrama:

#### <a name="before"></a>Antes del

![Antes del](./media/threat-modeling-tool-feature-overview/properties1.png)

#### <a name="after"></a>Después

![Después](./media/threat-modeling-tool-feature-overview/properties2.png)

### <a name="messages"></a>error de Hadoop

Si crea un modelo de amenazas y olvida conectar los flujos de datos a los elementos, recibirá una notificación. Puede elegir omitirla o seguir las instrucciones para solucionar el problema. 

![error de Hadoop](./media/threat-modeling-tool-feature-overview/messages.png)

### <a name="notes"></a>Notas

Para agregar notas al diagrama, cambie de la pestaña **Mensajes** a la pestaña **Notas**.

## <a name="analysis-view"></a>Vista de análisis

Después de crear el diagrama, seleccione el símbolo **Análisis** (Lupa) en la barra de herramientas de accesos directos para cambiar a la vista **Análisis**.

![Vista de análisis](./media/threat-modeling-tool-feature-overview/analysisview.png)

### <a name="generated-threat-selection"></a>Selección de amenaza generada

Cuando se selecciona una amenaza, puede usar tres funciones distintas:

| Característica                               | Information      |
| --------------------------------------- | ------------ |
| **Indicador de leído** | <p>La amenaza se marca como leída, lo que ayuda a realizar un seguimiento de los elementos que ha revisado.</p><p>![Indicador de leído o no leído](./media/threat-modeling-tool-feature-overview/readmode.png)</p> |
| **Foco de interacción** | <p>Se resalta la interacción en el diagrama que pertenece a una amenaza.</p><p>![Foco de interacción](./media/threat-modeling-tool-feature-overview/interactionfocus.png)</p> |
| **Propiedades de la amenaza** | <p>Aparece información adicional sobre la amenaza en la ventana **Propiedades de la amenaza**.</p><p>![Propiedades de la amenaza](./media/threat-modeling-tool-feature-overview/threatproperties.png)</p> |

### <a name="priority-change"></a>Priority change (Cambio de prioridad)

Puede cambiar el nivel de prioridad de cada amenaza generada. Colores diferentes hacen fácil identificar las amenazas de prioridad alta, media y baja.

![Priority change (Cambio de prioridad)](./media/threat-modeling-tool-feature-overview/prioritychange.png)

### <a name="threat-properties-editable-fields"></a>Campos modificables de las propiedades de amenaza

Tal como se muestra en la imagen anterior, puede cambiar la información generada por la herramienta. También puede agregar información a algunos de los campos, como la justificación. Estos campos son generados por la plantilla. Si necesita más información para cada amenaza, puede realizar modificaciones.

![Propiedades de la amenaza](./media/threat-modeling-tool-feature-overview/threatproperties.png)

## <a name="reports"></a>Informes

Una vez que haya terminado de cambiar las prioridades y de actualizar el estado de cada amenaza generada, puede guardar el archivo o imprimir un informe. Vaya a **Informe** > **Crear informe completo**. Asigne nombre al informe y debería ver algo similar a la siguiente imagen:

![Informe](./media/threat-modeling-tool-feature-overview/report.png)

## <a name="next-steps"></a>Pasos siguientes

- Envíe sus preguntas, comentarios y preocupaciones a tmtextsupport@microsoft.com. **[Descargue](https://aka.ms/threatmodelingtool)** Threat Modeling Tool para empezar.
- Para contribuir con una plantilla para la comunidad, vaya a nuestra página de [GitHub](https://github.com/Microsoft/threat-modeling-templates).