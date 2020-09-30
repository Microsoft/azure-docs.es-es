---
title: Interfaz de usuario de StorSimple Snapshot Manager | Microsoft Docs
description: Describe la interfaz de Administrador de instantáneas StorSimple y explica cómo usarla para administrar los trabajos de copia de seguridad y el catálogo de copia de seguridad.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: c7d91892-2881-41a2-a7a2-908dc3646493
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.custom: ''
ms.openlocfilehash: ead24b89dba889a4bb8f11d57eb687e7790aebfb
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055918"
---
# <a name="use-storsimple-snapshot-manager-user-interface-to-manage-backup-jobs-and-backup-catalog"></a>Uso de la interfaz de usuario de StorSimple Snapshot Manager para administrar trabajos de copia de seguridad y el catálogo de copia de seguridad

## <a name="overview"></a>Información general
Snapshot Manager de StorSimple tiene una interfaz de usuario intuitiva que puede usar para realizar y administrar copias de seguridad. Este tutorial proporciona una introducción a la interfaz de usuario y, después, explica cómo usar cada uno de los componentes. Para obtener una descripción detallada de StorSimple Snapshot Manager, consulte [¿Qué es StorSimple Snapshot Manager?](storsimple-what-is-snapshot-manager.md)

### <a name="console-description"></a>Descripción de la consola
Para ver la interfaz de usuario, haga clic en el icono de Administrador de instantáneas StorSimple que se encuentra en su escritorio. Aparece la ventana de consola, tal como se muestra en la siguiente ilustración.

![Paneles de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_gui_panes.png)

La ventana de consola tiene cinco elementos principales. Haga clic en el vínculo correspondiente para obtener una descripción completa de cada elemento.

* [Barra de menús](#menu-bar) 
* [Barra de herramientas](#tool-bar) 
* [Panel de Ámbito](#scope-pane) 
* [Panel de Resultados](#results-pane) 
* [Panel de Acciones](#actions-pane) 

Además, Administrador de instantáneas StorSimple es compatible con la [navegación mediante teclado y una serie de métodos abreviados de teclado](#keyboard-navigation-and-shortcuts).

### <a name="console-accessibility"></a>Accesibilidad de la consola
La interfaz de usuario de Administrador de instantáneas StorSimple es compatible con las características de accesibilidad proporcionadas por el sistema operativo Windows y Microsoft Management Console (MMC), así como con algunos métodos abreviados de teclado específicos de Administrador de instantáneas StorSimple. 

* Para obtener una descripción de las características de accesibilidad de Windows, vaya a [Métodos abreviados de teclado de Windows](https://support.microsoft.com/kb/126449). 
* Para obtener una descripción de las características de accesibilidad MMC, vaya a [Accesibilidad para MMC 3.0](https://technet.microsoft.com/library/cc766075.aspx)
* Para obtener una descripción de las características de accesibilidad de Administrador de instantáneas StorSimple, vaya a [Métodos abreviados y navegación mediante el teclado](#keyboard-navigation-and-shortcuts).

## <a name="menu-bar"></a>Barra de menús
La barra de menús de la parte superior de la ventana de la consola contiene los menús[ Archivo](#file-menu), [Acción](#action-menu), [Vista](#view-menu), [Favoritos](#favorites-menu), [Ventana](#window-menu) y [Ayuda](#help-menu).

Haga clic en cualquier elemento de la barra de menús para ver una lista de los comandos disponibles en ese menú. El ejemplo siguiente muestra el menú **Vista** seleccionado en la barra de menús.

![Menú Vista seleccionado](./media/storsimple-use-snapshot-manager/HCS_SSM_View_menu.png)

### <a name="file-menu"></a>Menú Archivo
El menú **Archivo** contiene comandos estándar de Microsoft Management Console (MMC).

#### <a name="menu-access"></a>Acceso al menú
Para ver el menú **Archivo**, haga clic en **Archivo** en la barra de menús. Aparecerá el siguiente menú.

![Menú Archivo de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_FileMenu.png) 

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente describe los elementos que aparecen en el menú **Archivo** .

| Elemento de menú | Descripción |
|:--- |:--- |
| Nuevo |Haga clic en **Nuevo** para crear una nueva consola basada en Administrador de instantáneas StorSimple. |
| Abrir |Haga clic en **Abrir** para abrir una consola existente. |
| Save |Haga clic en **Guardar** para guardar la consola actual. |
| Guardar como |Haga clic en **Guardar como** para crear una nueva instancia de la consola actual con un nombre diferente. Utilice la opción **Guardar como** para personalizar una vista y guardarla para su recuperación posterior. Por ejemplo, puede crear complementos de Administrador de instantáneas StorSimple que apunten a servidores específicos. |
| Agregar o quitar complemento |Haga clic en **Agregar o quitar complemento** para agregar o quitar complementos y organizar los nodos en el panel **Ámbito**. Para obtener más información, vaya a [Agregar, quitar y organizar complementos y extensiones en MMC 3.0](https://technet.microsoft.com/library/cc722035.aspx). |
| Opciones |Haga clic en **Opciones** para cambiar el icono de la consola, especificar los modos de acceso y permisos del usuario o eliminar archivos de consola para aumentar el espacio en disco disponible. |
| Lista de rutas de acceso |Haga clic en una ruta de acceso en la lista numerada para volver a abrir un archivo que haya abierto recientemente. |
| Salir |Haga clic en **Salir** para cerrar el menú **Archivo**. |

### <a name="action-menu"></a>Menú Acción
Use el menú **Acción** para seleccionar entre las acciones disponibles. Los elementos disponibles dependen de la selección que realice en el panel **Ámbito** o el panel **Resultados**.

#### <a name="menu-access"></a>Acceso al menú
Para ver el menú **Acción** , siga uno de estos procedimientos:

* Haga clic con el botón derecho en un elemento en el panel **Ámbito** o el panel **Resultados**.
* Seleccione un elemento en el panel **Ámbito** o en el panel **Resultados** y luego haga clic en **Acción** en la barra de menús. 

Por ejemplo, si selecciona el primer nodo en el panel **Ámbito** y luego hace clic con el botón derecho o el izquierdo en **Acción** en la barra de menús, aparecerá el siguiente menú.

![Menú Acción de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_Action_menu.png)

El panel **Acciones** (a la derecha de la consola) contiene la misma lista de acciones que el menú **Acción**. El panel **Acciones** contiene además las opciones del menú **Vista** que le permiten crear una vista personalizada del panel **Resultados**.

![Panel de Acciones con el menú Vista abierto](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane_Results.png)

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente contiene una lista alfabética de las acciones de Administrador de instantáneas StorSimple. 

* La columna **Acción** muestra las acciones que puede realizar en los nodos y los resultados. 
* La columna **Navegación** explica cómo mostrar el menú **Acción** apropiado para poder seleccionar la acción. Algunas acciones aparecen en varios menús **Acción** . Para estas acciones, seleccione una opción **Navegación** en la lista con viñetas. 
* La columna **Descripción** explica cómo usar cada acción en el menú **Acción** o el panel Acciones y explica lo que hace.

> [!NOTE]
> El panel **Acciones** y los menús **Acción** contienen opciones adicionales, como **Vista**, **Nueva ventana desde aquí**, **Actualizar**, **Exportar lista** y **Ayuda**. Estas opciones están disponibles como parte de MMC y no son específicas de Administrador de instantáneas StorSimple. La tabla incluye descripciones de estas opciones.
> 
> 

| Acción | Navegación | Descripción |
|:--- |:--- |:--- |
| Authenticate |Haga clic en el nodo **Dispositivos** y haga clic con el botón derecho en un dispositivo en el panel **Resultados**. |Haga clic en **Autenticar** para escribir la contraseña que configuró para el dispositivo. |
| Clonar |Expanda **Catálogo de copias de seguridad**, expanda **Instantáneas de nube**, haga clic en una copia de seguridad con fecha y luego seleccione un volumen en el panel **Resultados**. |Haga clic en **Clonar** para crear una copia de una instantánea de nube y almacenarla en la ubicación que designó. |
| Configurar un dispositivo |Haga clic con el botón derecho en el nodo **Dispositivos** . |Haga clic en **Configurar un dispositivo** para configurar uno o varios dispositivos para conectarlos con el host de Windows. |
| Crear directiva de copia de seguridad |Realice una de las siguientes acciones:<ul><li>Haga clic con el botón derecho en **Directivas de copia de seguridad**.</li><li>Haga clic o expanda **Grupos de volúmenes** y, a continuación, haga clic en un grupo de volúmenes.</li><li>Haga clic o expanda **Catálogo de copias de seguridad** y, a continuación, haga clic en un grupo de volúmenes.</li></ul> |Haga clic en **Crear directiva de copia de seguridad** para configurar una copia de seguridad programada para un grupo de volúmenes. |
| Crear grupo de volúmenes |Realice una de las siguientes acciones:<ul><li>Haga clic en el nodo **Volúmenes** y, a continuación, haga clic con el botón derecho en un dispositivo en el panel **Resultados**.</li><li>Haga clic con el botón derecho en el nodo **Grupos de volúmenes** .</li></ul> |Haga clic en **Crear grupo de volúmenes** para asignar volúmenes a un grupo de volúmenes. |
| Eliminar |Haga clic en un nodo o resultado (este elemento aparece en muchos menús **Acción** y paneles de **Acciones**). |Haga clic en **Eliminar** para eliminar el nodo o el resultado seleccionado. Cuando aparezca el cuadro de diálogo de confirmación, confirme o cancele la eliminación. |
| Detalles |Haga clic en el nodo **Dispositivos** y, a continuación, haga clic con el botón derecho en un dispositivo en el panel **Resultados**. |Haga clic en **Detalles** para ver los detalles de configuración de un dispositivo. |
| Editar |Haga clic en **Directivas de copia de seguridad** y luego haga clic con el botón derecho en una directiva en el panel **Resultados**. |Haga clic en **Editar** para cambiar la programación de copia de seguridad para un grupo de volúmenes. |
| Exportar lista |Haga clic en cualquier nodo o resultado (este elemento aparece en todos los menús **Acción** y paneles de **Acciones**). |Haga clic en **Exportar lista** para guardar una lista en un archivo de valores separados por comas (CSV). A continuación, puede importar este archivo en una aplicación de hoja de cálculo para su análisis. |
| Ayuda |Haga clic en cualquier nodo o resultado. (Este elemento aparece en todos los menús **Acción** y paneles de **Acciones**). |Haga clic en **Ayuda** para abrir la Ayuda en pantalla en una ventana independiente del explorador. |
| Nueva ventana desde aquí |Haga clic en cualquier nodo o resultado (este elemento aparece en todos los menús **Acción** y paneles de **Acciones**). |Haga clic en **Nueva ventana desde aquí** para abrir una nueva ventana de Administrador de instantáneas StorSimple. |
| Actualizar |Haga clic en cualquier nodo o resultado (este elemento aparece en todos los menús **Acción** y paneles de **Acciones**). |Haga clic en **Actualizar** para actualizar la ventana de Administrador de instantáneas StorSimple que se muestra en ese momento. |
| Actualizar dispositivo |Haga clic en el nodo **Dispositivos** y haga clic con el botón derecho en un dispositivo en el panel **Resultados**. |Haga clic en **Actualizar dispositivo** para sincronizar un dispositivo conectado específico con Administrador de instantáneas StorSimple. |
| Actualizar dispositivos |Haga clic con el botón derecho en el nodo **Dispositivos** . |Haga clic en **Actualizar dispositivos** para sincronizar la lista de dispositivos conectados con Administrador de instantáneas StorSimple. |
| Volver a examinar volúmenes |Haga clic con el botón derecho en el nodo **Volúmenes** . |Haga clic en **Volver a examinar volúmenes** para actualizar la lista de volúmenes que aparece en el panel **Resultados**. |
| Restauración |Expanda el **Catálogo de copias de seguridad**, un grupo de volúmenes y luego **Instantáneas locales** o **Instantáneas de nube**; a continuación, haga clic con el botón derecho en una copia de seguridad. |Haga clic en **Restaurar** para reemplazar los datos del grupo de volúmenes actual por los datos de la copia de seguridad seleccionada. |
| Realizar copia de seguridad |Realice una de las siguientes acciones:<ul><li>Expanda **Grupos de volúmenes** y, a continuación, haga clic en un grupo de volúmenes.</li><li>Expanda **Catálogo de copias de seguridad** y, a continuación, haga clic en un grupo de volúmenes.</li></ul> |Haga clic en **Realizar copia de seguridad** para iniciar un trabajo de copia de seguridad inmediatamente. |
| Alternar visualización de importaciones |Haga clic con el botón derecho en el primer nodo en el panel **Ámbito** (el nodo **Administrador de instantáneas StorSimple** en los ejemplos). |Haga clic en **Toggle Imports Display** (Alternar visualización de importaciones) para mostrar u ocultar los grupos de volúmenes y copias de seguridad asociadas que se importaron desde el panel del servicio StorSimple Device Manager. |

### <a name="view-menu"></a>Menú Vista
Use el menú **Vista** para crear una vista personalizada del contenido del panel **Resultados**. El menú **Vista** contiene las opciones **Agregar o quitar columnas** y **Personalizar**.

#### <a name="menu-access"></a>Acceso al menú
Puede acceder al menú **Vista** en la barra de menús o en el panel **Acciones**.

![Menú Vista de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_View_menu.png) 

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente describe los elementos que aparecen en el menú **Vista** .

| Elemento de menú | Descripción |
|:--- |:--- |
| Agregar o quitar columnas |Haga clic en **Agregar o quitar columnas** para agregar o quitar columnas en el panel **Resultados**. |
| Personalizar |Haga clic en **Personalizar** para mostrar u ocultar los elementos de la ventana de la consola de Administrador de instantáneas StorSimple. |

### <a name="favorites-menu"></a>Menú Favoritos
Use el menú **Favoritos** para agregar, quitar y organizar las vistas de página y las tareas que use con frecuencia. 

#### <a name="menu-access"></a>Acceso al menú
Puede acceder al menú **Favoritos** desde la barra de menús.

![Menú Favoritos de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_FavoritesMenu.png)

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente describe los elementos que aparecen en el menú **Favoritos** .

| Elemento de menú | Descripción |
|:--- |:--- |
| Agregar a Favoritos |Haga clic en **Agregar a favoritos** para agregar la vista actual a la lista de favoritos. |
| Organizar Favoritos |Haga clic en **Organizar favoritos** para organizar el contenido de la carpeta Favoritos. |

### <a name="window-menu"></a>Menú Ventana
Utilice el menú **Ventana** para agregar y organizar las ventanas de la consola de StorSimple Snapshot Manager.

#### <a name="menu-access"></a>Acceso al menú
Puede acceder al menú **Ventana** desde la barra de menús.

![Menú Ventana de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_WindowMenu.png)

La lista numerada en la parte inferior del menú muestra las ventanas que están actualmente abiertas. Haga clic en cualquier ventana de esa lista para sacarla a primer plano. 

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente describe los elementos que aparecen en el menú Ventana.

| Elemento de menú | Descripción |
|:--- |:--- |
| Nueva ventana |Haga clic en **Nueva ventana** para abrir una nueva ventana de consola (además de la ventana existente). |
| Cascade |Haga clic en **Cascada** para mostrar las ventanas de consola abiertas en cascada. |
| Mosaico horizontal |Haga clic en **Colocar en mosaico horizontal** para mostrar las ventanas de consola abiertas en un formato de mosaico (o cuadrícula). |
| Organizar iconos |Si tiene varias ventanas de consola abiertas y repartidas por el escritorio, minimícelas y luego haga clic en **Organizar iconos** para colocarlas en una fila horizontal en la parte inferior de la pantalla. |

### <a name="help-menu"></a>Menú Ayuda
Use el menú **Ayuda** para ver la ayuda en pantalla disponible para Administrador de instantáneas StorSimple y MMC. También puede ver información acerca de las versiones de software de MMC y Administrador de instantáneas StorSimple que están instaladas actualmente en el sistema. 

Puede acceder al menú **Ayuda** desde la barra de menús. También puede acceder a los temas de Ayuda de Administrador de instantáneas StorSimple desde el panel **Acciones** .

![Menú Ayuda de Administrador de instantáneas StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_HelpMenu.png)

#### <a name="menu-description"></a>Descripción del menú
La tabla siguiente describe los elementos que aparecen en el menú Ayuda.

| Elemento de menú | Descripción |
|:--- |:--- |
| Ayuda de Administrador de instantáneas StorSimple |Haga clic en **Ayuda en Administrador de instantáneas StorSimple** para abrir la Ayuda de Administrador de instantáneas StorSimple en una ventana independiente. |
| Temas de ayuda |Haga clic en **Temas de Ayuda** para abrir la Ayuda en pantalla de MMC en una ventana independiente. |
| Sitio Web de TechCenter |Haga clic en **Sitio web de TechCenter** para abrir la página principal de Microsoft TechNet Tech Center en una ventana independiente. |
| Acerca de Microsoft Management Console |Haga clic en **Acerca de Microsoft Management Console** para ver qué versión de Microsoft Management Console está instalada en el sistema. |
| Acerca de Administrador de instantáneas StorSimple |Haga clic en **Acerca de Administrador de instantáneas StorSimple** para ver qué versión del complemento está instalada en el sistema. |

## <a name="tool-bar"></a>Barra de herramientas
La barra de herramientas, que está debajo de la barra de menús, contiene iconos de navegación y de tareas. Cada icono es un acceso directo a una tarea específica.

### <a name="icon-descriptions"></a>Descripciones de los iconos
En la tabla siguiente se describe los iconos que aparecen en la barra de herramientas. 

| Icono | Descripción |
|:--- |:--- |
| ![Flecha izquierda](./media/storsimple-use-snapshot-manager/HCS_SSM_LeftArrow.png) |Haga clic en el icono de flecha izquierda para volver a la página anterior. |
| ![Flecha derecha](./media/storsimple-use-snapshot-manager/HCS_SSM_RightArrow.png) |Haga clic en la flecha derecha para ir a la página siguiente (si la flecha está atenuada, la acción no está disponible). |
| ![Icono de subir](./media/storsimple-use-snapshot-manager/HCS_SSM_Up.png) |Haga clic en el icono de subir para subir un nivel en el árbol de consola (panel **Ámbito** ). |
| ![Mostrar u ocultar el árbol de consola](./media/storsimple-use-snapshot-manager/HCS_SSM_ShowConsoleTree.png) |Haga clic en el icono de mostrar u ocultar en el árbol de consola para mostrar u ocultar el panel **Ámbito** . |
| ![Exportar lista](./media/storsimple-use-snapshot-manager/HCS_SSM_ExportListIcon.png) |Haga clic en el icono de la lista de exportación para exportar una lista a un archivo CSV que especifique. |
| ![Icono de ayuda](./media/storsimple-use-snapshot-manager/HCS_SSM_HelpIcon.png) |Haga clic en el icono de ayuda para abrir un tema de la ayuda en pantalla de MMC. |
| ![Mostrar u ocultar el panel Acciones](./media/storsimple-use-snapshot-manager/HCS_SSM_ShowAction.png) |Haga clic en el icono de mostrar u ocultar el panel **Acciones** para mostrar u ocultar el panel **Acciones**. |

## <a name="scope-pane"></a>Panel de Ámbito
El panel **Ámbito** es el panel situado más a la izquierda en la interfaz de usuario de Snapshot Manager de StorSimple. Contiene el árbol de consola (o nodo) y es el mecanismo de navegación principal para Administrador de instantáneas StorSimple. 

### <a name="scope-pane-structure"></a>Estructura del panel Ámbito
El panel **Ámbito** contiene una serie de objetos seleccionables (nodos) organizados en una estructura de árbol. 

![Panel de Ámbito](./media/storsimple-use-snapshot-manager/HCS_SSM_Scope_pane.png) 

* Para expandir o contraer un nodo, haga clic en el icono de flecha situado junto al nombre del nodo.
* Para ver el estado o el contenido de un nodo, haga clic en el nombre del nodo. La información aparece en el panel **Resultados** . 

El panel **Ámbito** contiene los siguientes nodos: 

* [Nodo Dispositivos](#devices-node) 
* [Nodo Volúmenes](#volumes-node) 
* [Nodo Grupos de volúmenes](#volume-groups-node) 
* [Nodo de Directivas de copia de seguridad](#backup-policies-node) 
* [Nodo Catálogo de copias de seguridad](#backup-catalog-node) 
* [Nodo Trabajos](#jobs-node) 

### <a name="scope-pane-tasks"></a>Tareas del panel Ámbito
Puede usar el panel **Ámbito** para completar una acción en un nodo específico. Para seleccionar una tarea, realice una de las siguientes acciones:

* Haga clic con el botón derecho en el nodo y, a continuación, seleccione la tarea en el menú que aparece.
* Haga clic en el nodo y, a continuación, haga clic en **Acción** en la barra de menús. Seleccione la tarea en el menú que aparece.
* Haga clic en el nodo y, a continuación, seleccione la acción en el panel **Acciones** .

Cuando se selecciona un nodo y se usa cualquiera de estos métodos para ver una lista de tareas, solo se muestran las acciones que se pueden realizar en ese nodo.

### <a name="devices-node"></a>Nodo Dispositivos
El nodo **Dispositivos** representa los dispositivos de StorSimple y los dispositivos virtuales de StorSimple que están conectados con Administrador de instantáneas StorSimple. Seleccione este nodo para conectar y configurar un dispositivo e importar sus volúmenes asociados, grupos de volúmenes y copias de seguridad existentes. Varios dispositivos pueden estar conectados a un solo host.

* Para expandir el nodo, haga clic en el icono de flecha junto a **Dispositivos**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Dispositivos** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de dispositivos configurados, haga clic en **Dispositivos** en el panel **Ámbito**. La lista de dispositivos, junto con información sobre cada dispositivo aparece en el panel **Resultados** .

### <a name="volumes-node"></a>Nodo Volúmenes
El nodo **Volúmenes** representa las unidades que corresponden a los volúmenes montados por el host, incluidos aquellos detectados a través de iSCSI y detectados a través de un dispositivo. Utilice este nodo para ver la lista de volúmenes disponibles y asignar volúmenes individuales a grupos de volúmenes.

* Para expandir el nodo, haga clic en el icono de flecha junto a **Volúmenes**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Volúmenes** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de volúmenes, haga clic en **Volúmenes** en el panel **Ámbito**. Aparece la lista de volúmenes, junto con información acerca de cada volumen en el panel **Resultados** .

### <a name="volume-groups-node"></a>Nodo Grupos de volúmenes
Los grupos de volúmenes también conocen como grupos de consistencia. Cada grupo de volúmenes es un conjunto de volúmenes relacionados con la aplicación que ayuda a asegurar la consistencia de las aplicaciones durante las operaciones de copia de seguridad. Use el nodo **Grupos de volúmenes** para configurar estos grupos y para realizar copias de seguridad interactivas o crear programaciones de copia de seguridad. 

* Para expandir el nodo, haga clic en el icono de flecha junto a **Grupos de volúmenes**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en **Grupos de volúmenes** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de grupos de volúmenes, haga clic en **Grupos de volúmenes** en el panel **Ámbito**. Aparece la lista de grupos de volúmenes, junto con información acerca de cada grupo de volúmenes en el panel **Resultados** .

### <a name="backup-policies-node"></a>Nodo de Directivas de copia de seguridad
Las directivas de copia de seguridad son las programaciones de trabajo para realizar instantáneas locales y en la nube. Use el nodo **Directivas de copia de seguridad** para especificar con qué frecuencia se creará una copia de seguridad y durante cuánto tiempo debe conservarse. 

* Para expandir el nodo, haga clic en el icono de flecha junto a **Directivas de copia de seguridad**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Directivas de copias de seguridad** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de directivas de copia de seguridad, haga clic en **Directivas de copias de seguridad** en el panel **Ámbito**. La lista de directrices de copias de seguridad, junto con información sobre cada directriz aparece en el panel **Resultados** .

> [!NOTE]
> Puede conservar un máximo de 64 copias de seguridad.


### <a name="backup-catalog-node"></a>Nodo Catálogo de copias de seguridad
El nodo **Catálogo de copia de seguridad** contiene listas de copias de seguridad locales y remotas de volúmenes de Azure StorSimple. Este nodo está organizado por grupo de volúmenes, y cada contenedor de grupo de volúmenes contiene estructuras separadas para instantáneas locales (el nodo **Instantáneas locales**) y de instantáneas de nube (el nodo **Instantáneas de nube**). Cuando se expande, cada contenedor de grupo de volúmenes enumera todas las copias de seguridad correctas que se tomaron de forma interactiva o por una directiva configurada.

* Para expandir el nodo, haga clic en el icono de flecha junto a **Catálogo de copias de seguridad**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Catálogo de copias de seguridad** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de las instantáneas de copia de seguridad, haga clic en **Catálogo de copias de seguridad** en el panel **Ámbito**. La lista de instantáneas, junto con información sobre cada instantánea, aparece en el panel **Resultados** .

### <a name="local-snapshots-node"></a>Nodo Instantáneas locales
El nodo **Instantáneas locales** ofrece una lista de las instantáneas locales para un grupo de volúmenes específico. El nodo se encuentra dentro del nodo **Catálogo de copia de seguridad** en el panel **Ámbito**. Las instantáneas locales son copias instantáneas de los datos del volumen que se almacenan en el dispositivo Azure StorSimple. Normalmente, este tipo de copia de seguridad se puede crear y restaurar rápidamente. Puede usar una instantánea local como lo haría con una copia de seguridad local.

* Para expandir el nodo, haga clic en el icono de flecha junto a **Instantáneas locales**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Instantáneas locales** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de instantáneas locales, haga clic en **Instantáneas locales** en el panel **Ámbito**. La lista de instantáneas, junto con información sobre cada instantánea, aparece en el panel **Resultados** .

### <a name="cloud-snapshots-node"></a>Nodo Instantáneas de nube
El nodo **Instantáneas de nube** ofrece una lista de las instantáneas de nube para un grupo de volúmenes específico. El nodo se encuentra dentro del nodo **Catálogo de copia de seguridad** en el panel **Ámbito**. Las instantáneas de nube son copias instantáneas de los datos del volumen que se almacenan en la nube. Una instantánea de nube es equivalente a una instantánea replicada en un sistema de almacenamiento externo. Las instantáneas en la nube son especialmente útiles en escenarios de recuperación ante desastres.

* Para expandir el nodo, haga clic en el icono de flecha junto a **Instantáneas de nube**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Instantáneas de nube** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de instantáneas de nube, haga clic en **Instantáneas de nube** en el panel **Ámbito**. La lista de instantáneas, junto con información sobre cada instantánea, aparece en el panel **Resultados** .

### <a name="jobs-node"></a>Nodo Trabajos
El nodo **Trabajos** contiene información acerca de los trabajos de copia de seguridad programados, en ejecución y completados recientemente. 

* Para expandir el nodo, haga clic en el icono de flecha situado junto a **Trabajos**.
* Para ver un menú de acciones disponibles, haga clic con el botón derecho en el nodo **Trabajos** o en cualquiera de los nodos que aparecen en la vista ampliada.
* Para ver una lista de los trabajos programados, expanda el nodo **Trabajos** y luego haga clic en **Programado**. La lista de trabajos configurados anteriormente e información sobre cada trabajo aparece en el panel **Resultados** . 
* Para ver una lista de los trabajos completados recientemente, expanda el nodo **Trabajos** y luego haga clic en **Últimas 24 horas**. En el panel **Resultados** aparecerá una lista de trabajos que se completaron en las últimas 24 horas. El panel **Resultados** también contiene información acerca de cada trabajo completado.
* Para ver una lista de trabajos que se están ejecutando, amplíe el nodo **Trabajos** y luego haga clic en **En ejecución**. La lista de trabajos que se están ejecutando e información sobre cada trabajo aparece en el panel **Resultados** .

## <a name="results-pane"></a>Panel de Resultados
El panel **Resultados** es el panel central en la interfaz de usuario de StorSimple Snapshot Manager. Contiene listas e información de estado detallada para el nodo que esté seleccionado en el panel **Ámbito** .

### <a name="example"></a>Ejemplo
Para ver el ejemplo siguiente, haga clic en el nodo **Grupos de volúmenes** en el panel **Ámbito**. El panel **Resultados** muestra una lista de grupos de volúmenes con detalles acerca de cada grupo.

![Panel de Resultados](./media/storsimple-use-snapshot-manager/HCS_SSM_Results_pane.png) 

Puede configurar los detalles que aparece en el panel **Resultados**: haga clic con el botón derecho en un nodo en el panel **Ámbito**, haga clic en **Vista** y luego haga clic en **Agregar o quitar columnas**.

## <a name="actions-pane"></a>Panel de Acciones
El panel **Acciones** es el panel que se encuentra a la derecha en la interfaz de usuario de Administrador de instantáneas StorSimple. Contiene un menú de las operaciones que puede realizar en el nodo, vista o datos que se seleccionan en el panel **Ámbito** o el panel **Resultados**. El panel **Acciones** contiene los mismos comandos que los menús **Acción** que están disponibles para los elementos del panel **Ámbito** y **Resultados**. Para obtener una descripción de cada acción, consulte la tabla en la sección sobre el menú **Acción** .

### <a name="examples"></a>Ejemplos
Para ver el ejemplo siguiente, en el panel **Ámbito**, expanda el nodo **Trabajos** y haga clic en **Programado**. El panel **Acciones** muestra las acciones disponibles para el nodo **Programado**.

![Ejemplo de los trabajos programados del panel Acciones](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane.png) 

Para ver más opciones, en panel **Ámbito**, expanda el nodo **Trabajos**, haga clic en **Programado** y, luego, haga clic en un trabajo programado en el panel **Resultados**. El panel **Acciones** muestra las acciones disponibles para el trabajo programado, tal como se muestra en el ejemplo siguiente.

![Ejemplo de acciones de trabajo del panel Acciones](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane_Results.png)

## <a name="keyboard-navigation-and-shortcuts"></a>Métodos abreviados y navegación mediante el teclado
Administrador de instantáneas StorSimple habilita las características de accesibilidad del sistema operativo Windows y Microsoft Management Console (MMC). También incluye algunas características de navegación con el teclado y métodos abreviados de teclado que son específicos de Administrador de instantáneas StorSimple, como se describe en las secciones siguientes.

* [Teclas de navegación](#keyboard-navigation-keys) 
* [Teclas de método abreviado de la barra de menús](#menu-bar-shortcut-keys) 
* [Teclas de método abreviado del panel Ámbito](#scope-pane-shortcut-keys) 

### <a name="keyboard-navigation-keys"></a>Teclas de navegación
La tabla siguiente describen las teclas que puede usar para navegar por la interfaz de usuario de Administrador de instantáneas StorSimple. 

| Tecla de navegación | Acción |
|:--- |:--- |
| Flecha abajo |Utilice la tecla flecha abajo para mover verticalmente al siguiente elemento de un menú o panel. |
| Escriba |Presione la tecla ENTRAR para completar una acción y, a continuación, vaya al paso siguiente. Por ejemplo, puede presionar ENTRAR para seleccionar **Siguiente**, **Aceptar** o **Crear** y luego vaya al paso siguiente en un asistente. |
| Esc |Presione la tecla Esc para cerrar un menú o para cancelar y cerrar una página. |
| F1 |Presione la tecla F1 para ver un tema de Ayuda de la ventana actualmente activa. |
| F5 |Presione la tecla F5 para actualizar un nodo. |
| F6 |Presione la tecla F6 para pasar desde el panel **Ámbito** al panel **Resultados**. |
| F10 |Presione la tecla F10 para ir a la barra de menús. |
| Tecla de flecha izquierda |Utilice la tecla de flecha izquierda para realizar movimientos horizontales de una opción de la barra de menú a la opción anterior. Cuando se desplaza al elemento anterior en la barra de menús, aparece el menú Acción (o contexto) para el elemento anterior. |
| Tecla de flecha derecha |Utilice la tecla de flecha derecha para realizar movimientos horizontales de una opción de la barra de menú a la siguiente. Cuando se desplaza al elemento siguiente en la barra de menús, aparece el menú Acción (o contexto) para el nuevo elemento. |
| Tecla TAB |Utilice la tecla Tab para desplazarse al siguiente panel de la consola o a la siguiente selección o cuadro de texto en una página. |
| Tecla de flecha arriba |Utilice la tecla flecha arriba para desplazarse verticalmente hasta el elemento anterior en un menú o panel. |

### <a name="menu-bar-shortcut-keys"></a>Teclas de método abreviado de la barra de menús
La tabla siguiente describe las combinaciones de teclas de método abreviado para la barra de menús. Después de presionar las teclas de método abreviado y abrir el menú, puede usar teclas de método abreviado de menú (las claves subrayadas en el menú). Para obtener más información acerca de la barra de menús, vaya a [Barra de menús](#menu-bar).

| Método abreviado | Resultado | Tecla de método abreviado de menú | Resultado |
|:--- |:--- |:--- |:--- |
| ALT + F |Abre el menú **Archivo** . |N |Abre una nueva instancia de la consola. |
|  |O |Abre la página **Herramientas administrativas** . | |
|  |S |Guarda la consola de Administrador de instantáneas StorSimple. | |
|  |A |Abre la página **Guardar como** . | |
|  |M |Abre la página **Agregar o quitar complemento** . | |
|  |P |Abre la página **Opciones** . | |
|  |H |Abre la Ayuda en pantalla. | |
| ALT + A |Abre el menú **Acción** . |I |Activa y desactiva la opción de visualización de importación . |
|  |W |Abre una nueva consola de Administrador de instantáneas StorSimple. | |
|  |F |Actualiza la consola de Administrador de instantáneas StorSimple. | |
|  |L |Abre la página **Exportar lista** . | |
|  |H |Abre la Ayuda en pantalla. | |
| ALT+V |Abre el menú **Vista** . |A |Abre la página **Agregar o quitar columnas** . |
|  |U |Abre la página **Personalizar vista** . | |
| ALT + O |Abre el menú **Favoritos** . |A |Abre la página **Agregar a favoritos** . |
|  |O |Abre la página **Organizar favoritos** . | |
| ALT + W |Abre el menú **Ventana** . |N |Abre otra ventana de Administrador de instantáneas StorSimple. |
|  |C |Muestra todas las ventanas de consola abiertas en un estilo en cascada. | |
|  |T |Muestra todas las ventanas de consola abiertas en una cuadrícula. | |
|  |I |Organiza los iconos en una fila horizontal en la parte inferior de la pantalla. | |
| ALT + H |Abre el menú **Ayuda** . |H |Abre la Ayuda en pantalla. |
|  |T |Abre la página web de Microsoft TechNet Tech Center. | |
|  |A |Abre la página **Acerca de Microsoft Management Console** . | |

### <a name="scope-pane-shortcut-keys"></a>Teclas de método abreviado del panel Ámbito
Las siguientes tablas muestran las combinaciones de teclas de método abreviado para cada nodo en el panel **Ámbito** . 

* [Teclas de método abreviado del nodo Dispositivos](#devices-node-shortcut-keys)
* [Teclas de método abreviado del nodo Volúmenes](#volumes-node-shortcut-keys)
* [Teclas de método abreviado del nodo Grupos de volúmenes](#volume-groups-node-shortcut-keys)
* [Teclas de método abreviado del nodo Directivas de copia de seguridad](#backup-policies-node-shortcut-keys)
* [Teclas de método abreviado del nodo Catálogo de copia de seguridad](#backup-catalog-node-shortcut-keys)
* [Teclas de método abreviado del nodo Trabajos](#jobs-node-shortcut-keys)

#### <a name="devices-node-shortcut-keys"></a>Teclas de método abreviado del nodo Dispositivos
| Método abreviado de menú | Resultado |
|:--- |:--- |
| C |Abre la página **Configurar un dispositivo** . |
| D |Actualiza la lista de dispositivos y los detalles del dispositivo. |
| V |Abre el menú **Vista** . |
| W |Abre una nueva consola de Administrador de instantáneas StorSimple centrada en el nodo **Detalles** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| L |Abre la página **Exportar lista** . |
| H |Abre la Ayuda en pantalla. |

#### <a name="volumes-node-shortcut-keys"></a>Teclas de método abreviado del nodo Volúmenes
| Método abreviado de menú | Resultado |
|:--- |:--- |
| V |Actualiza la lista de volúmenes. |
| V (presione dos veces) |Abre el menú **Vista** . |
| W |Abre una nueva consola de Administrador de instantáneas StorSimple centrada en el nodo **Volúmenes** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| L |Abre la página **Exportar lista** . |
| H |Abre la Ayuda en pantalla. |

#### <a name="volume-groups-node-shortcut-keys"></a>Teclas de método abreviado del nodo Grupos de volúmenes
| Método abreviado de menú | Resultado |
|:--- |:--- |
| G |Abre la página **Crear un grupo de volúmenes** . |
| V |Abre el menú **Vista** . |
| W |Abre una nueva consola de StorSimple Snapshot Manager centrada en el nodo **Grupos de volúmenes** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| L |Abre la página **Exportar lista** . |
| H |Abre la Ayuda en pantalla. |

#### <a name="backup-policies-node-shortcut-keys"></a>Teclas de método abreviado del nodo Directivas de copia de seguridad
| Método abreviado de menú | Resultado |
|:--- |:--- |
| B |Abre la página **Crear una directiva** . |
| V |Abre el menú **Vista** . |
| W |Abre una nueva consola de StorSimple Snapshot Manager centrada en el nodo **Grupos de volúmenes** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| L |Abre la página **Exportar lista** . |
| H |Abre la Ayuda en pantalla. |

#### <a name="backup-catalog-node-shortcut-keys"></a>Teclas de método abreviado del nodo Catálogo de copia de seguridad
| Método abreviado de menú | Resultado |
|:--- |:--- |
| W |Abre una nueva consola de StorSimple Snapshot Manager centrada en el nodo **Grupos de volúmenes** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| H |Abre la Ayuda en pantalla. |

#### <a name="jobs-node-shortcut-keys"></a>Teclas de método abreviado del nodo Trabajos
| Método abreviado de menú | Resultado |
|:--- |:--- |
| V |Abre el menú **Vista** . |
| W |Abre una nueva consola de Administrador de instantáneas StorSimple centrada en el nodo **Trabajos** . |
| F |Actualiza la consola de Administrador de instantáneas StorSimple. |
| L |Abre la página **Exportar lista** . |
| H |Abre la Ayuda en pantalla |

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple para administrar la solución de StorSimple](storsimple-snapshot-manager-admin.md).
* Aprenda a [usar StorSimple Snapshot Manager para conectarse y administrar dispositivos](storsimple-snapshot-manager-manage-devices.md).

