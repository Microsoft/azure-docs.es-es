---
title: Configuración de materiales de PBR en 3ds Max
description: En este artículo se explica cómo configurar materiales con representación basada en la física en 3ds Max y exportarlos al formato FBX.
author: muxanickms
ms.author: misams
ms.date: 06/16/2020
ms.topic: tutorial
ms.openlocfilehash: ac0f4ee8f06982126d2ae30bed01716b287e8993
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078046"
---
# <a name="tutorial-set-up-physically-based-rendering-materials-in-3ds-max"></a>Tutorial: Configuración de materiales con representación basada en la física en 3ds Max

## <a name="overview"></a>Información general
En este tutorial, aprenderá a:

>[!div class="checklist"]
>
> * Asignar materiales con iluminación avanzada a los objetos de una escena.
> * Administrar la creación de instancias de objetos y materiales.
> * Exportar una escena a formato FBX y seleccionar las opciones importantes.

Crear [materiales con representación basada en la física (PBR)](../../overview/features/pbr-materials.md) en 3ds Max es una tarea relativamente sencilla. Es similar de muchas maneras a la configuración de PBR en otras aplicaciones de creación de contenido como Maya. Este tutorial es una guía para la configuración básica del sombreador de PBR y la exportación a FBX de proyectos de Azure Remote Rendering.

La escena de ejemplo de este tutorial contiene varios objetos de cuadro poligonal. Se les han asignado distintos materiales, como madera, metal, metal pintado, plástico y caucho. En términos generales, cada material contiene todas o la mayoría de las texturas siguientes:

* **Albedo**, que es el mapa de colores del material y también se llama **Diffuse** (Difuso) y **BaseColor** (Color de base).
* **Metálico**, que determina si un material es metálico y qué partes son metálicas. 
* **Rugosidad**, que determina cómo de rugosa o suave es una superficie.
También afecta a la nitidez o al desenfoque de los reflejos y los resaltados en una superficie.
* **Normal**, que agrega detalles a una superficie sin agregar más polígonos. Algunos ejemplos de detalle son las picaduras y las abolladuras en una superficie de metal o el grano en la madera.
* **Oclusión de ambiente**, que se usa para agregar matices suaves y sombras de contacto a un modelo. Se trata de un mapa de escala de grises que indica qué áreas del modelo reciben una iluminación completa (blanco) o una sombra completa (negro).

## <a name="prepare-the-scene"></a>Preparación de la escena
En 3dx Max, este es el proceso para configurar un material de PBR:

Para empezar, vamos a crear una serie de objetos de cuadro, cada uno de los cuales representa un tipo de material diferente.

>[!TIP]
>Antes de empezar a crear recursos para Remote Rendering, es importante advertir que se usan metros para las medidas.  
>
>Por lo tanto, es conveniente establecer las unidades del sistema de la escena en metros. También es una buena idea establecer **Units** (Unidades) en metros en la configuración de exportación de FBX cuando se exporta una escena.

En la captura de pantalla siguiente se muestran los pasos para establecer las unidades del sistema en metros en 3ds Max. 

1. En el menú principal, vaya a **Customize** > **Units Setup** > **System Units Setup** (Personalizar > Configuración de unidades > Configuración de unidades del sistema). En **System Unit Scale** (Escala de unidades del sistema), seleccione **Meters** (Metros): ![Captura de pantalla que muestra cómo establecer las unidades del sistema.](media/3dsmax/system-units.jpg)

1. Ahora podemos empezar a crear los modelos. En la escena de ejemplo, vamos a crear varios objetos de cuadro, cada uno de los cuales representa un tipo de material diferente. Por ejemplo, metal, caucho y plástico. 

   >[!TIP]
   >Al crear recursos, es aconsejable asignarles un nombre adecuado a medida que avanza. De esta forma, será más fácil encontrarlos más adelante si la escena contiene muchos objetos.

1. Cambie el nombre de los objetos, como se muestra en la siguiente captura de pantalla: 

   ![Captura de pantalla que muestra cómo cambiar el nombre de los objetos.](media/3dsmax/rename-objects.jpg)

## <a name="assign-materials"></a>Asignación de materiales

Ahora que tiene algunos objetos en la escena, en este caso, varios cubos, podemos iniciar la configuración de PBR:

1. En la barra de herramientas principal, seleccione el icono **Material Editor** (Editor de materiales), como se muestra en la siguiente captura de pantalla. También puede presionar la tecla **M** en el teclado para abrir el editor. El editor de materiales tiene dos modos que puede seleccionar en la lista **Modes** (Modos): **Compact Material Editor** (Editor de materiales compacto) y **Slate Material Editor** (Editor de materiales de pizarra). Dado que esta escena es relativamente sencilla, usaremos el modo compacto.

1. En el editor de materiales, verá una serie de esferas. Estas esferas son los materiales. Asignaremos uno de estos materiales a cada objeto (cuadro) de la escena. Para asignar los materiales, seleccione primero uno de los objetos en la ventanilla principal. A continuación, seleccione la primera esfera en el editor de materiales. Cuando se haya asignado a un objeto, se resaltará el material seleccionado, tal como se muestra en la siguiente imagen.

1. Seleccione **Assign Material to Selection** (Asignar material a la selección), como se muestra. El material se asigna ahora al objeto seleccionado.

   ![Captura de pantalla que muestra cómo asignar materiales.](media/3dsmax/assign-material.jpg)

    En el editor de materiales, puede elegir entre una amplia selección de tipos de materiales, en función de sus necesidades. Normalmente, el tipo de material se establece en **Estándar** de forma predeterminada. Este material es un material básico que no es adecuado para la instalación de PBR. Por lo tanto, es necesario cambiar el tipo de material por uno de PBR. Physical Material (Material físico) es el material de 3ds Max preferido para proyectos de Azure Remote Rendering.

1. En el editor de materiales, seleccione la pestaña **Standard** (Estándar). En **Material/Map Browser** (Explorador de materiales y mapas), seleccione **Physical Material** (Material físico). Esta acción convertirá el material **Standard** (Estándar) asignado en un material físico de PBR.

   ![Captura de pantalla que muestra cómo cambiar el material.](media/3dsmax/physical-material.jpg)

    En el editor de materiales, ahora verá las propiedades del material físico, como se muestra en la siguiente captura de pantalla. Ahora puede empezar a asignar texturas al recurso.

   ![Captura de pantalla que muestra la lista de texturas.](media/3dsmax/textures-list.jpg)

Como puede ver, hay una amplia variedad de mapas y texturas que puede agregar al material. En este tutorial, usamos solo cinco espacios de textura en el material.

>[!TIP]
>Es recomendable asignar un nombre adecuado a los materiales, como se muestra en la captura de pantalla anterior.

La forma de generar las texturas puede variar en función de la preferencia o el uso. Por ejemplo, puede que quiera usar texturas de mosaico que se puedan aplicar a cualquier recurso. O bien, puede que necesite que determinadas partes de un proyecto o recurso tengan sus propios conjuntos personalizados de texturas. También, es posible que quiera usar texturas de mosaicos genéricas que puede obtener en línea. Además, puede crearlas por su cuenta en aplicaciones como Photoshop, Quixel Suite y Substance Suite.

Antes de empezar a asignar texturas, debemos tener en cuenta las coordenadas de textura del recurso (UVW). Se recomienda que, al aplicar texturas a un modelo, esté desplegado. (Las texturas no se mostrarán correctamente sin un despligue UV adecuado). Es especialmente importante para nuestros fines, ya que queremos usar un mapa de oclusión de ambiente (AO) en nuestro modelo. A diferencia del sombreador Stingray de Maya, el material físico de 3ds Max no tiene un espacio de textura de AO dedicada. Por lo tanto, aplicaremos el mapa de AO a otro espacio. Para permitir que se use de forma independiente de las otras texturas (por ejemplo, las texturas de mosaicos), la asignaremos a un canal de mapa UVW propio. 

Para comenzar, asignaremos un modificador para desajustar UVW en el modelo, tal como se muestra en la captura de pantalla siguiente. 

- En el editor de propiedades de los objetos seleccionados, seleccione la lista de modificadores. En la lista desplegable que aparece, desplácese hacia abajo y seleccione **Unwrap UVW** (Desajustar UVW). Esta acción aplica al recurso un modificador para desajustar UVW.
![Captura de pantalla que muestra cómo seleccionar Unwrap UVW (Desajustar UVW).](media/3dsmax/unwrap-modifier.jpg)

  El canal del mapa se establece en 1. Normalmente, el despliegue principal se realiza en el canal del mapa 1. En nuestro caso, el objeto se ha desplegado sin superposición de las coordenadas de textura (UV).
![Captura de pantalla que muestra las coordenadas de textura (UVW) desplegadas.](media/3dsmax/unwrapped-uvw.jpg)

El siguiente paso consiste en crear un segundo canal de mapa UV.

1. Cierre el editor de UV si está abierto. En la sección **Channel** (Canal) del menú **Edit UVs** (Editar UV), cambie el número de canal a **2**. El canal de mapa 2 es el canal esperado para los mapas de AO. 

1. En el cuadro de diálogo **Channel Change Warning** (Advertencia de cambio de canal), puede **mover** los UV existentes en el canal 1 al nuevo canal 2 o bien **abandonar** los UV existentes, lo que creará un nuevo desajuste de UV automáticamente. Seleccione **Abandon** (Abandonar) solo si tiene previsto crear un nuevo desajuste de UV para el mapa de AO que difiera de los UV del canal de mapa 1. (Por ejemplo, si quiere usar texturas de mosaicos en el canal 1). En este tutorial, moveremos los UV del canal 1 al canal 2, ya que no es necesario editar el nuevo canal UV.

   >[!NOTE]
   >Incluso si ha copiado (movido) el desajuste UV del canal de mapa 1 en el canal de mapa 2, puede realizar las modificaciones necesarias en los nuevos UV del canal sin que ello afecte al canal de mapa original.

   ![Captura de pantalla que muestra la advertencia de cambio de canal.](media/3dsmax/channel-change.jpg)

Ahora que hemos creado el canal de mapa, podemos volver al material físico en el editor de materiales y empezar a agregarle las texturas. En primer lugar, vamos a agregar el mapa de AO y, luego, realizaremos otro paso para que funcione correctamente. Una vez que el mapa de AO está conectado a nuestro material, es necesario configurarlo para que use el canal de mapa 2.

Como se mencionó anteriormente, no hay ningún espacio dedicado para mapas de AO en el material físico de 3ds Max. Se aplicará, en cambio, el mapa de AO al espacio **Diffuse Roughness** (Rugosidad difusa).

1. En la lista **Generic Maps** (Mapas genéricos) del material físico, seleccione el espacio **No Map** (Sin mapa) de **Diffuse Roughness** (Rugosidad difusa) y cargue el mapa de AO.

1. En las propiedades de las texturas de AO, el canal de mapa se establece en **1** de forma predeterminada. Cambie este valor a **2**. Esta acción pone punto y final a los pasos necesarios para agregar el mapa de AO.

   >[!IMPORTANT]
   >Este es un paso importante, especialmente si los ejes UV del canal 2 son diferentes de los del canal 1, ya que el AO no se asignará correctamente si se selecciona el canal equivocado.

   ![Captura de pantalla que muestra cómo asignar un mapa de AO.](media/3dsmax/assign-ao-map.jpg)

Ahora, asignaremos el mapa normal al material de PBR. Esta acción difiere ligeramente del proceso de Maya. El mapa normal no se aplica directamente al espacio del mapa de rugosidad. (No hay ningún espacio de mapa normal en el material físico de 3ds Max). En cambio, el mapa normal se agrega a un modificador de mapa normal, que se conecta al espacio del mapa normal.

1. En la sección **Special Maps** (Mapas especiales) de las propiedades del material físico (en el editor de materiales), seleccione el espacio **No Map** (Sin mapa) junto a **Bump Map** (Mapa de rugosidad). 

1. En **Material/Map Browser** (Explorador de materiales y mapas), busque y seleccione **Normal Bump** (Rugosidad normal). Esta acción agrega un modificador **Normal Bump** (Rugosidad normal) al material.

1. En el modificador **Normal Bump** (Rugosidad normal), seleccione **No Map** (Sin mapa) junto a **Normal**. Busque y cargue el mapa normal.

1. Asegúrese de que el método está establecido en **Tangen** (Tangente). (De forma predeterminada, debería estarlo). Si es necesario, alterne **Flip Green (Y)** (Girar verde [Y]).

   ![Captura de pantalla que muestra cómo seleccionar la rugosidad normal.](media/3dsmax/normal-bump.jpg)
   ![Captura de pantalla que muestra la carga del mapa normal.](media/3dsmax/load-normal-map.jpg)

Con el mapa normal asignado correctamente, podemos asignar el resto de texturas para completar la configuración del material físico. El proceso es sencillo. No hay que tener en cuenta ninguna configuración especial. En la captura de pantalla siguiente se muestra el conjunto completo de texturas asignadas al material: 

![Captura de pantalla que muestra el conjunto completo de texturas asignadas al material.](media/3dsmax/all-textures.jpg)

Ahora que se han creado y configurado los materiales de PBR, conviene pensar en crear instancias de los objetos de la escena. Cree instancias de objetos similares de la escena, como tuercas, pernos, tornillos y arandelas. Los objetos que sean iguales pueden producir ahorros considerables en términos de tamaño de archivo. Las instancias de un objeto maestro pueden tener su propia escala, rotación y transformaciones, por lo que se pueden colocar donde sean necesarios en la escena. En 3ds Max, el proceso de creación de instancias es sencillo.

1. En la ventanilla principal, seleccione los objetos que quiere exportar.

1. Mantenga presionada la tecla **Mayús** y arrastre los recursos hacia arriba con la herramienta de transformación (movimiento). 

1. En el cuadro de diálogo **Clone Options** (Opciones de clonación), establezca **Object** (Objeto) en **Instance** (Instancia) y, luego, haga clic en **OK** (Aceptar).

   ![Captura de pantalla del cuadro de diálogo Clone Options (Opciones de clonación).](media/3dsmax/instance-object.jpg)

Esta acción crea una instancia del objeto que puede mover, girar o escalar con independencia de su elemento primario y de otras instancias de este.

>[!IMPORTANT]
>Los cambios que realice en una instancia mientras está en el modo de objeto secundario se transmiten a todas las instancias del objeto. Por lo tanto, si va a trabajar con los componentes de un objeto con instancias, como vértices y caras de un polígono, asegúrese de que quiere que los cambios que realice afecten a todas las instancias. Recuerde que cualquier objeto creado como una instancia se puede convertir en un objeto único en cualquier momento. 

>[!TIP]
>Al crear instancias en la escena, es una buena idea hacerlo a medida que avanza. Reemplazar las copias por objetos con instancias más adelante es difícil. 

Un aspecto final que se debe tener en cuenta antes de pasar al proceso de exportación es cómo desea empaquetar la escena o el recurso para compartirlos. Idealmente, si pasa el recurso a un cliente o a un miembro del equipo, querrá que puedan abrirlo y visualizarlo con una cantidad mínima de complicaciones. Por lo tanto, es importante mantener las rutas de acceso de las texturas de los recursos relativas al archivo de la escena. Si las rutas de acceso de las texturas del recurso apuntan a una unidad local o a una ruta o ubicación absolutas, no se cargarán en la escena si se abren en otro equipo, incluso si el archivo .max está en la misma carpeta que las texturas. El uso de rutas de acceso de texturas relativas en 3ds Max resuelve este problema y es bastante sencillo.

1. En la barra de herramientas principal, vaya a **File** > **Reference** > **Asset Tracking Toggle** (Archivo > Referencia > Activar seguimiento de recursos). 

1. En la ventana de seguimiento de recursos, verá todas o la mayoría de las texturas que ha aplicado a los materiales de PBR que aparecen en la columna **Maps/Shaders** (Mapas/Sombreadores).

1. Junto a ellas, en la columna **Full Path** (Ruta de acceso completa), verá la ruta de acceso de la ubicación de las texturas, que probablemente sea su ubicación en la máquina local.

1. Por último, verá una columna llamada **Status** (Estado). Esta columna indica si se ha encontrado una textura determinada y se ha aplicado a la escena. La textura se marca con uno de estos términos: **Ok**, **Found** o **File Missing** (Correcto > Encontrado o Falta un archivo). Los dos primeros indican que el archivo se ha encontrado y cargado. El último obviamente significa que el rastreador no encontró el archivo.
 
   ![Captura de pantalla que muestra la ventana de seguimiento de recursos.](media/3dsmax/texture-paths.jpg)

Es posible que observe que no se enumeran todas las texturas en la ventana Asset Tracking (Seguimiento de recursos) cuando se abre por primera vez. No hay nada de lo que preocuparse. Al ejecutar el proceso de búsqueda de rutas de acceso, una o dos veces, se suelen encontrar todas las texturas de una escena. El proceso de búsqueda de rutas de acceso es el siguiente: 

1. En la ventana Asset Tracking (Seguimiento de recursos), mantenga pulsada la tecla **Mayús** y seleccione la textura superior de la lista **Maps/Shaders** (Mapas/Sombreadores) y, sin soltar esta tecla, seleccione la última textura de la lista. Esta acción selecciona todas las texturas de la lista. Las texturas seleccionadas se resaltan en azul. (Consulte la captura de pantalla anterior).

1. Haga clic con el botón derecho en la selección y elija **Set Path** (Establecer ruta de acceso).

1. En el cuadro **Specify Asset Path** (Especificar la ruta de acceso del recurso), seleccione la ruta de acceso local a las texturas y reemplácela por `.\`.  Seleccione **Aceptar**. 

    La ventana Asset Tracking (Seguimiento de recursos) se actualizará, como se muestra en la captura de pantalla siguiente. Esta actualización puede tardar un rato, según el número de texturas que haya en la escena y el tamaño de esta.
![Captura de pantalla que muestra la ventana Asset Tracking (Seguimiento de recursos) actualizada.](media/3dsmax/resolve-textures.jpg)

Observe que la columna **Full Path** (Ruta de acceso completa) ahora está en blanco. Esto significa que la escena ya no busca las texturas pertinentes en una ubicación específica (absoluta). Siempre se encontrarán mientras el archivo .max o el archivo FBX relacionado estén en la misma carpeta que las texturas. 

>[!NOTE]
>Es posible que tenga que repetir este proceso un par de veces para buscar y resolver todas las texturas y rutas de acceso. No hay nada de lo que preocuparse. Simplemente repita el proceso hasta que se tengan en cuenta todos los recursos pertinentes. En algunos casos, algunos archivos no se encuentran. En ese caso, solo tiene que seleccionar los recursos de la lista y, luego, elegir **Remove Missing Paths** (Eliminar las rutas de acceso que faltan). (Consulte la imagen anterior.)

## <a name="fbx-export"></a>Exportación de FBX

Ahora que hemos convertido en relativas las rutas de acceso de las texturas, podemos pasar a la exportación de FBX. De nuevo, el proceso es sencillo y se puede realizar de dos maneras. 

>[!TIP]
>A menos que quiera exportar toda la escena, es una buena idea seleccionar para exportar solo los recursos que necesite. En escenas con un uso intensivo de recursos, la exportación puede tardar mucho tiempo.
>
>Si ha usado modificadores como Turbosmooth u Open SubDiv, es una buena idea contraerlos antes de la exportación, ya que pueden causar problemas durante esta operación. Asegúrese de guardar la escena antes de contraerlos. 

1. En la escena, seleccione los recursos que quiere exportar. En la barra de herramientas principal, vaya a **File** > **Export** > **Export Selected** (Archivo > Exportar > Exportar la selección).

1. En el cuadro de diálogo **Select File to Export** (Seleccionar archivo para exportar), escriba o seleccione un nombre de archivo de salida. En la lista **Save as type** (Guardar como tipo), seleccione **Autodesk (*.fbx)** . Esta acción abre la ventana de exportación de FBX.

  >[!IMPORTANT] 
  >Si ha creado instancias en la escena, es importante seleccionar **Preserve Instances** (Conservar instancias) en la configuración de exportación de FBX. 

  ![Captura de pantalla que muestra cómo exportar a FBX.](media/3dsmax/fbx-export.jpg)

  Recuerde que hay un par de maneras de exportar el archivo. Si la intención es compartir el FBX junto con sus archivos de textura en una carpeta o directorio, la configuración que se muestra en la siguiente captura de pantalla funcionará correctamente. 

   Si prefiere no compartir carpetas o directorios grandes de texturas junto con el archivo FBX, puede elegir insertar las texturas en el FBX. Si inserta las texturas, todo el recurso, incluidas las texturas, se agrega a un único FBX. Al hacerlo, se combina la exportación en un único recurso, pero el archivo FBX será bastante más grande.

   >[!IMPORTANT]
   >Si el archivo FBX resultante es mayor que 2,4 GB, la versión mínima especificada en la configuración de exportación de FBX debe ser 2016 o posterior. (Consulte la captura de pantalla anterior). Las versiones más recientes tienen compatibilidad con 64 bits, por lo que admiten archivos más grandes.

1. Si quiere exportar la escena con las texturas incluidas, en la ventana de exportación de *FBX, seleccione **Embed Media** (Insertar elementos multimedia). 

1. Seleccione el resto de la configuración y, luego, elija **OK** (Aceptar):

    ![Captura de pantalla que muestra la configuración de exportación de FBX.](media/3dsmax/fbx-settings.jpg)


   Al exportar a FBX mientras usa un material físico, es probable que vea la siguiente advertencia después de seleccionar **OK** (Aceptar) en la ventana de exportación de FBX: 

   ![Captura de pantalla que muestra la advertencia de error de exportación del material.](media/3dsmax/export-warnings.jpg)

   Esta advertencia indica que los materiales exportados podrían no ser compatibles con otros paquetes de software. Dado que el material físico es compatible con Azure Remote Rendering, no es necesario preocuparse por esta advertencia. 

1. Simplemente haga clic en **OK** (Aceptar) para completar el proceso y cerrar la ventana.

## <a name="conclusion"></a>Conclusión

En general, este tipo de materiales es más realista, ya que se basa en la física de la luz del mundo real. Crea un efecto envolvente adicional para que la escena parezca ser del mundo real.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya sabe cómo configurar materiales con iluminación avanzada para los objetos de una escena. También sabe cómo exportar objetos al formato FBX, que es compatible con Azure Remote Rendering. El siguiente paso consiste en convertir el archivo FBX y visualizarlo en Azure Remote Rendering.

>[!div class="nextstepaction"]
>[Inicio rápido: Conversión de un modelo para su representación](../../quickstarts\convert-model.md)
