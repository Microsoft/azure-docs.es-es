---
title: Tipos de clase de ejemplo en Azure Lab Services | Microsoft Docs
description: Proporciona algunos tipos de clases para los que puede configurar laboratorios mediante Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 21159aacfcd87bb929eaeaf9884b0275c7005e1a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001630"
---
# <a name="class-types-overview---azure-lab-services"></a>Información general sobre tipos de clase: Azure Lab Services

Azure Lab Services le permite configurar rápidamente un entorno de laboratorio educativo en la nube. En los artículos de esta sección se proporcionan instrucciones sobre cómo configurar varios tipos de laboratorios de clase mediante Azure Lab Services.

## <a name="deep-learning-in-natural-language-processing"></a>Aprendizaje profundo en el procesamiento del lenguaje natural

Puede configurar un laboratorio centrado en el aprendizaje profundo en el procesamiento de lenguaje natural (NLP) mediante Azure Lab Services. El procesamiento de lenguaje natural (NLP) es una forma de inteligencia artificial (AI) que permite a los equipos utilizar la traducción, el reconocimiento de voz y otras capacidades de comprensión de lenguajes. Los alumnos que sigan una clase de NLP obtienen una máquina virtual Linux para aprender a aplicar algoritmos de red neuronal para desarrollar modelos de aprendizaje profundo que se utilizan para analizar el lenguaje humano escrito.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio centrado en el aprendizaje profundo en el procesamiento de lenguaje natural mediante Azure Lab Services](class-type-deep-learning-natural-language-processing.md).

## <a name="shell-scripting-on-linux"></a>Generación de scripts en shell en Linux

Puede configurar un laboratorio para enseñar el scripting de shell en Linux. El scripting es una parte útil de la administración del sistema que permite a los administradores evitar tareas repetitivas. En este escenario de ejemplo, la clase cubre scripts bash tradicionales y scripts mejorados. Los scripts mejorados son scripts que combinan comandos bash y Ruby. Este enfoque permite a Ruby pasar datos y comandos bash para interactuar con el shell.

Los alumnos que sigan estas clases de scripting obtienen una máquina virtual Linux para conocer los aspectos básicos de Linux y también familiarizarse con el scripting de shell de bash. La máquina virtual Linux incluye el acceso al escritorio remoto habilitado y con los editores de texto [gedit](https://help.gnome.org/users/gedit/stable/) y [Visual Studio Code](https://code.visualstudio.com/) instalados.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Scripting de shell en Linux](class-type-shell-scripting-linux.md).

## <a name="ethical-hacking"></a>Piratería ética

Puede configurar un laboratorio para una clase que se centre en la parte forense de la piratería ética. Las pruebas de penetración son una práctica que usa la comunidad de piratería ética y que se producen cuando alguien intenta obtener acceso al sistema o a la red para mostrar los puntos vulnerables que un atacante malintencionado podría aprovechar.

En una clase de piratería ética, los alumnos pueden aprender técnicas modernas para protegerse frente a los puntos vulnerables. Cada alumno obtiene una máquina virtual host de Windows Server que tiene dos máquinas virtuales anidadas: una máquina virtual con una imagen de [Metasploitable3](https://github.com/rapid7/metasploitable3) y otra con una imagen de [Kali Linux](https://www.kali.org/). La máquina virtual Metasploitable se usa para la explotación.  La máquina virtual Kali de Linux proporciona acceso a las herramientas necesarias para ejecutar tareas forenses.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para impartir una clase de piratería ética](class-type-ethical-hacking.md).

## <a name="database-management"></a>Administración de bases de datos
Los conceptos de las bases de datos son uno de los cursos introductorios en la mayoría de los departamentos de informática de las universidades. Puede configurar un laboratorio para una clase de administración de bases de datos básica en Azure Lab Services. Por ejemplo, puede configurar una plantilla de máquina virtual en un laboratorio con un servidor de base de datos [MySQL](https://www.mysql.com/) o un servidor de [SQL Server 2019](https://www.microsoft.com/sql-server/sql-server-2019).

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para enseñar administración de bases de datos relacionales](class-type-database-management.md).

## <a name="python-and-jupyter-notebooks"></a>Python y cuadernos de Jupyter Notebook
Puede configurar una máquina de plantilla en Azure Lab Services con las herramientas necesarias para enseñar a los alumnos cómo usar [cuadernos de Jupyter Notebook](http://jupyter-notebook.readthedocs.io). Jupyter Notebooks es un proyecto de código abierto que le permite combinar fácilmente texto enriquecido y código fuente [Python](https://www.python.org/) ejecutable en un solo lienzo denominado cuaderno. Al ejecutar un cuaderno, se genera un registro lineal de entradas y salidas.  Dichas salidas pueden incluir texto, tablas de información, gráficos de dispersión y mucho más.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para enseñar ciencia de datos con Python y cuadernos de Jupyter Notebook](class-type-jupyter-notebook.md).

## <a name="big-data-analytics"></a>Análisis de macrodatos
Puede configurar un laboratorio de GPU para enseñar una clase de análisis de macrodatos. Con este tipo de clase, los alumnos aprenden a manipular grandes volúmenes de datos y a aplicar algoritmos de aprendizaje automático y estadístico para obtener conclusiones sobre los datos. Un objetivo clave para los alumnos es aprender a usar herramientas de análisis de datos, como el paquete de software de código abierto de Apache Hadoop, que proporciona herramientas para almacenar, administrar y procesar macrodatos. 

Para obtener información detallada sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para el análisis de macrodatos mediante una implementación de Docker de HortonWorks Data Platform](class-type-big-data-analytics.md).

## <a name="matlab"></a>MATLAB
[MATLAB](https://www.mathworks.com/products/matlab.html), que es la abreviatura de laboratorio de matrices, es una plataforma de programación de [MathWorks](https://www.mathworks.com/).  Combina la potencia del cálculo con una buena visualización, lo que lo convierte en una herramienta popular en los campos de matemáticas, ingeniería, física y química.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para enseñar MATLAB](class-type-matlab.md).

## <a name="solidworks-computer-aided-design-cad"></a>Diseño asistido por ordenador (CAD) de SolidWorks
Puede configurar un laboratorio de GPU que proporcione a los estudiantes de ingeniería acceso a [SolidWorks](https://www.solidworks.com/).  SolidWorks proporciona un entorno CAD 3D para el modelado de objetos sólidos.  Con SolidWorks, los ingenieros pueden crear, visualizar, simular y documentar sus diseños fácilmente.

Para más información sobre cómo configurar este tipo de laboratorio, consulte [Configuración de un laboratorio para clases de ingeniería mediante SolidWorks](class-type-solidworks.md)

## <a name="next-steps"></a>Pasos siguientes

Vea los artículos siguientes:

- [Configuración de un laboratorio centrado en el aprendizaje profundo en el procesamiento de lenguaje natural mediante Azure Lab Services](class-type-deep-learning-natural-language-processing.md)
- [Generación de scripts de shell en Linux](class-type-shell-scripting-linux.md)
- [Piratería ética](class-type-ethical-hacking.md)