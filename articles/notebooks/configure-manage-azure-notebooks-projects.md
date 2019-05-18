---
title: Configuración y administración de proyectos de Azure Notebook
description: Administración de los metadatos de proyecto, los archivos de proyecto, el entorno del proyecto y los pasos de configuración a través tanto de la interfaz de usuario de Azure Notebooks como del acceso al terminal directo.
services: app-service
documentationcenter: ''
author: kraigb
manager: douge
ms.assetid: 35dd6ff1-a14a-4a2e-b173-6d8467de3e89
ms.service: azure-notebooks
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/13/2019
ms.author: kraigb
ms.openlocfilehash: 0440e498451ee141fa03851b78418caf911d0e32
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596743"
---
# <a name="manage-and-configure-projects"></a>Administración y configuración de proyectos

Esencialmente, un proyecto en Azure Notebooks es una configuración de la máquina virtual Linux subyacente donde se ejecutan las instancias de Jupyter Notebook, junto con metadatos descriptivos y una carpeta de archivos. El panel del proyecto de Azure Notebooks permite administrar los archivos y configurar las características del proyecto:

- Nivel de proceso en el que se ejecuta el proyecto, que puede ser el nivel Gratis o una máquina virtual de Azure.
- Los metadatos del proyecto, que incluyen un nombre, una descripción, un identificador que se usa al compartir el proyecto e información de si el proyecto es público o privado.
- El cuaderno del proyecto, los datos y otros archivos que el usuario administra tal como lo hace con cualquier otro sistema de archivos.
- El entorno de un proyecto, que configura ya sea a través de los scripts de inicio o directamente a través del terminal.
- Los registros, a los que obtiene acceso a través de la terminal.

> [!Note]
> Las características de administración y configuración descritas aquí solo están disponibles para el propietario del proyecto que lo creó inicialmente. Sin embargo, puede clonar el proyecto en su propia cuenta, en cuyo caso se convierte en propietario y puede configurar el proyecto según sea necesario.

Azure Notebooks inicia la máquina virtual subyacente cada vez que ejecuta un cuaderno u otro archivo. El servidor guarda automáticamente los archivos y se apaga después de 60 minutos de inactividad. También puede detener el servidor en cualquier momento con el comando **Shutdown** (Apagar) (método abreviado de teclado: h).

## <a name="compute-tier"></a>Nivel de proceso

De forma predeterminada, los proyectos se ejecutan el **gratuitos Compute** nivel, que está limitado a 4 GB de memoria y 1 GB de datos para evitar abusos. Puede omitir estas limitaciones y aumentar la capacidad de proceso mediante el uso de una máquina virtual diferente que se ha aprovisionado en una suscripción de Azure. Para obtener más información, consulte [cómo usar Data Science Virtual Machines](use-data-science-virtual-machine.md).

## <a name="edit-project-metadata"></a>Edición de los metadatos del proyecto

En el panel del proyecto, seleccione **Configuración del proyecto** y la pestaña **Información**, que contiene los metadatos del proyecto que se describen en la tabla siguiente. Puede cambiar los metadatos del proyecto en cualquier momento.

| Configuración | DESCRIPCIÓN |
| --- | --- |
| Nombre del proyecto | Un nombre descriptivo para el proyecto que usa Azure Notebooks para fines de presentación. Por ejemplo, "Hola mundo en Python". |
| Id. de proyecto | Un identificador personalizado que se vuelve parte de la dirección URL que usa para compartir un proyecto. Este identificador puede usar solo letras, números y guiones, se limita a 30 caracteres y no puede ser un [reservado Id. de proyecto](create-clone-jupyter-notebooks.md#reserved-project-ids). Si no está seguro de qué usar, una convención común es usar una versión en minúsculas del nombre del proyecto, donde los espacios se convierten en guiones, como "mi-proyecto-de-cuaderno" (truncado si es necesario para que se ajuste al límite de longitud). |
| Proyecto público | Si se establece, permite a cualquier usuario con el vínculo acceder al proyecto. Al crear un proyecto privado, desactive esta opción. |
| Ocultar clones | Si se establece, otros usuario no podrán ver una lista de los clones que se han realizado para este proyecto. Ocultar clones resulta útil para los proyectos que se comparten con muchas personas que no forman parte de la misma organización, como cuando se usa un cuaderno para impartir una clase. |

> [!Important]
>
> Al cambiar el identificador del proyecto se invalida cualquier vínculo al proyecto que haya compartido anteriormente.

## <a name="manage-project-files"></a>Administración de archivos del proyecto

El panel del proyecto muestra el contenido del sistema de carpetas del proyecto. Puede usar varios comandos para administrar esos archivos.

### <a name="create-new-files-and-folders"></a>Creación de archivos y carpetas

El comando **+ Nuevo** (método abreviado de teclado: n) crea archivos o carpetas. Cuando use el comando, primero seleccione el tipo de elemento que se va a crear:

| Tipo de elemento | DESCRIPCIÓN | Comportamiento del comando |
| --- | --- | --- |
| **Cuaderno** | Un cuaderno de Jupyter | Muestra un elemento emergente en el que se especifica el nombre de archivo y el lenguaje del cuaderno. |
| **Carpeta** | Una subcarpeta | Crea un campo de edición en la lista de archivos del proyecto donde se escribe el nombre de la carpeta. |
| **Archivo en blanco** | Un archivo donde puede almacenar cualquier contenido, como texto, datos, etc. | Crea un campo de edición en la lista de archivos del proyecto donde se escribe el nombre del archivo. |
| **Markdown** | Un archivo Markdown. | Crea un campo de edición en la lista de archivos del proyecto donde se escribe el nombre del archivo. |

### <a name="upload-files"></a>Cargar archivos

El comando **Cargar** proporciona dos opciones para importar datos desde otras ubicaciones: **From URL** (Desde dirección URL) y **From Computer** (Desde equipo). Para más información, consulte [Trabajar con archivos de datos en proyectos de Azure Notebooks](work-with-project-data-files.md).

### <a name="select-file-specific-commands"></a>Selección de los comandos específicos de archivo

Cada elemento de la lista de archivos del proyecto proporciona comandos a través del menú contextual que aparece con el botón derecho:

![Comandos en el menú contextual de un archivo](media/project-file-commands.png)

| Get-Help | Métodos abreviados de teclado | . |
| --- | --- | --- |
| Ejecutar | r (o clic) | Ejecuta un archivo de Notebook. Otros tipos de archivo se abren para su visualización.  |
| Copiar vínculo | y | Copia un vínculo al archivo en el Portapapeles. |
| Ejecutar en JupyterLab | j | Ejecuta un cuaderno en JupyterLab, que es una interfaz más orientada al desarrollador que la que Jupyter proporciona habitualmente. |
| Preview | p | Abre una vista previa en HTML del archivo. En Notebooks, la vista previa es una representación de solo lectura del cuaderno. Para más información, consulte la sección [Vista previa](#preview). |
| Editar archivo | i | Abre el archivo para edición. |
| Descargar | d | Descarga un archivo ZIP que contiene el archivo o el contenido de una carpeta. |
| Cambiar nombre | un | Solicita un nombre nuevo para el archivo o carpeta. |
| Eliminar | x | Solicita confirmación y luego quita de manera permanente el archivo del proyecto. Las eliminaciones no se pueden deshacer. |
| Mover | min | Mueve un archivo a otra carpeta del mismo proyecto. |

#### <a name="preview"></a>Preview

La vista previa de un archivo o cuaderno es una vista de solo lectura del contenido; la ejecución de las celdas del cuaderno está deshabilitada. Se muestra una vista previa a cualquier persona que tenga un vínculo al archivo o al cuaderno y que no haya iniciado sesión en Azure Notebooks. Una vez que haya iniciado sesión, un usuario puede clonar el cuaderno en su propia cuenta, o bien descargar el cuaderno a su equipo local.

La página de vista previa admite varios comandos de barra de herramientas con métodos abreviados de teclado:

| Get-Help | Métodos abreviados de teclado | . |
| --- | --- | --- |
| Compartir | s | Muestra la ventana emergente de uso compartido donde puede obtener un vínculo, compartir en las redes sociales, obtener el código HTML para insertarlo y enviar un correo electrónico. |
| Clonar | c  | Clone el cuaderno en su cuenta. |
| Ejecutar | r | Ejecute el cuaderno si tiene permiso para hacerlo. |
| Descargar | d | Descarga una copia del cuaderno. |

## <a name="configure-the-project-environment"></a>Configuración del entorno del proyecto

Hay tres formas de configurar el entorno de la máquina virtual subyacente donde se ejecutan los cuadernos:

- Incluir un script de inicialización único
- Usar la configuración del entorno del proyecto para especificar los pasos de configuración
- Acceder a la máquina virtual a través de un terminal.

Todas las formas de configuración de proyectos se aplican cada vez que se inicia la máquina virtual, lo que afecta a todos los cuadernos dentro del proyecto.

### <a name="one-time-initialization-script"></a>Script de inicialización único

La primera vez que Azure Notebooks crea un servidor para el proyecto, busca un archivo llamado *aznbsetup.sh* en el proyecto. Si este archivo existe, Azure Notebooks lo ejecuta. La salida del script se almacena en la carpeta del proyecto como *.aznbsetup.log*.

### <a name="environment-setup-steps"></a>Pasos para la configuración del entorno

Puede usar la configuración del entorno del proyecto para crear pasos individuales que configuren el entorno.

En el panel del proyecto, seleccione **Configuración del proyecto** y luego la pestaña **Entorno** donde puede agregar, quitar y modificar los pasos para la configuración del proyecto:

![Ventana emergente de la configuración del proyecto con la pestaña Entorno seleccionada](media/project-settings-environment-steps.png)

Para agregar un paso, primero seleccione **+ Agregar** y, luego, seleccione un tipo de paso en la lista desplegable **Operaciones**:

![Selector de operaciones para un nuevo paso de configuración del entorno](media/project-settings-environment-details.png)

La información del proyecto depende del tipo de operación que eligió:

- **Requirements.txt**: En la segunda lista desplegable, seleccione un archivo *requirements.txt* que ya esté en el proyecto. Luego, seleccione una versión de Python de la tercera lista desplegable que aparece. Mediante el uso de un archivo *requirements.txt*, Azure Notebooks ejecuta `pip install -r` con el archivo *requirements.txt* al iniciar un servidor de cuaderno. No es necesario instalar explícitamente los paquetes desde el mismo cuaderno.

- **Script de shell**: En la segunda lista desplegable, seleccione un script de shell de bash del proyecto (habitualmente, un archivo con la extensión *.sh*) que contiene cualquier comando que quiere ejecutar para inicializar el entorno.

- **Environment.yml**: En la segunda lista desplegable, seleccione un archivo *environments.yml* para los proyectos de Python mediante un entorno de Conda.

Una vez que agregue los pasos, seleccione **Guardar**.

### <a name="use-the-terminal"></a>Uso del terminal

En el panel del proyecto, el comando **Terminal** abre un terminal Linux que le da acceso directo al servidor. Dentro del terminal, puede descargar datos, editar o administrar archivos, inspeccionar procesos e, incluso, usar herramientas como vi y nano.

> [!Note]
> Si tiene scripts de inicio en el entorno del proyecto, al abrir el terminal puede aparecer un mensaje que indique que la instalación está en curso.

Puede emitir cualquier comando estándar de Linux en el terminal. También puede usar `ls` en la carpeta de inicio para ver los distintos entornos que existen en la máquina virtual, como *anaconda2_501*, *anaconda3_420*, *anaconda3_501*, *IfSharp* y *R*, junto con una carpeta *project* que contiene el proyecto:

![Terminal del proyecto en Azure Notebooks](media/project-terminal.png)

Para afectar un entorno específico, cambie los directorios a esa carpeta de entorno primero.

En el caso de los entornos de Python, puede encontrar `pip` y `conda` en la carpeta *bin* de cada entorno. También puede usar alias integrados para los entornos:

```bash
# Anaconda 2 5.3.0/Python 2.7: python27
python27 -m pip install <package>

# Anaconda 3 4.2.0/Python 3.5: python35
python35 -m pip install <package>

# Anaconda 3 5.3.0/Python 3.6: python36
python36 -m pip install <package>
```

Los cambios que se hagan en el servidor solo se aplican a la sesión actual, excepto en el caso de los archivos y carpetas que crea en la carpeta *project* misma. Por ejemplo, la edición de un archivo en la carpeta del proyecto se mantiene entre las sesiones, pero los paquetes con `pip install` no.

> [!Note]
> Si usa `python` o `python3`, invoca las versiones de Python instaladas por el sistema, las que no se usan para los cuadernos. Tampoco tiene permisos para operaciones como `pip install`, por lo que debe asegurarse de usar los alias específicos para la versión.

## <a name="access-notebook-logs"></a>Acceso a los registros de cuadernos

Si encuentra problemas al ejecutar un cuaderno, la salida de Jupyter se almacena en una carpeta denominada *.nb.log*. Puede acceder a estos registros a través del comando **Terminal** o del panel del proyecto.

Por lo general, cuando ejecuta Jupyter de manera local, puede iniciarlo desde una ventana del terminal. La ventana del terminal muestra la salida como un estado de kernel.

Para ver los registros, use el comando siguiente en el terminal:

```bash
cat .nb.log
```

También puede usar el comando desde una celda de código de un cuaderno de Python:

```bash
!cat .nb.log
```

## <a name="next-steps"></a>Pasos siguientes

- [Procedimientos: Work with project data files](work-with-project-data-files.md) (Trabajo con archivos de datos de proyecto)
- [Access cloud data in a notebook](access-data-resources-jupyter-notebooks.md) (Acceso a los datos en la nube en un cuaderno)
