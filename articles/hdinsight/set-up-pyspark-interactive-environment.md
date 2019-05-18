---
title: 'Herramientas de Azure HDInsight: configuración de un entorno interactivo de PySpark para Visual Studio Code'
description: Aprenda a usar Obtenga información sobre cómo usar Herramientas de Azure HDInsight para Visual Studio Code para crear y enviar consultas y scripts.
keywords: VScode,Herramientas de Azure HDInsight,Hive,Python,PySpark,Spark,HDInsight,Hadoop,LLAP,Hive interactivo,Consulta interactiva
ms.service: hdinsight
author: jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 1/17/2019
ms.openlocfilehash: d7d6bf45a8cdf22fbff0d3fda8175e8007a612d9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789280"
---
# <a name="set-up-the-pyspark-interactive-environment-for-visual-studio-code"></a>Configuración del entorno interactivo de PySpark para Visual Studio Code

Los pasos siguientes muestran cómo configurar el entorno interactivo de PySpark en VS Code.

Usamos el comando **python/pip** para crear un entorno virtual en la ruta de acceso principal. Si quiere usar otra versión, deberá cambiar la versión predeterminada del comando **python/pip** manualmente. Para más información, consulte [update-alternatives](https://linux.die.net/man/8/update-alternatives).

1. Instale [Python](https://www.python.org/downloads/) y [pip](https://pip.pypa.io/en/stable/installing/).
   
   + Instale Python desde [https://www.python.org/downloads/](https://www.python.org/downloads/).
   + Instale pip desde [https://pip.pypa.io/en/stable/installing](https://pip.pypa.io/en/stable/installing/). (Si no se ha instalado desde la instalación de Python)
   + Compruebe que Python y pip se han instalado correctamente mediante los siguientes comandos. (Opcional)
 
        ![Versión de pip de Python](./media/set-up-pyspark-interactive-environment/check-python-pip-version.png)

     > [!NOTE]
     > Se recomienda instalar Python manualmente, en lugar de usar la versión predeterminada de MacOS.


2. Instale **virtualenv** con el comando siguiente.
   
   ```
   pip install virtualenv
   ```

3. Solo para Linux, instale los paquetes necesarios ejecutando los comandos siguientes si aparece algún mensaje de error.
   
    ![Versión de pip de Python](./media/set-up-pyspark-interactive-environment/install-libkrb5-package.png)
       
   ```
   sudo apt-get install libkrb5-dev 
   ```

   ```
   sudo apt-get install python-dev
   ```

4. Reinicie Visual Studio Code y vuelva al editor de scripts que ejecuta **HDInsight: PySpark interactivo**.

## <a name="next-steps"></a>Pasos siguientes

### <a name="demo"></a>Demostración
* HDInsight para VS Code: [Vídeo](https://go.microsoft.com/fwlink/?linkid=858706)

### <a name="tools-and-extensions"></a>Herramientas y extensiones
* [Uso de Herramientas de Azure HDInsight para Visual Studio Code](hdinsight-for-vscode.md)
* [Uso de Azure Toolkit for IntelliJ para crear y enviar aplicaciones de Scala para Apache Spark](spark/apache-spark-intellij-tool-plugin.md)
* [Uso de Azure Toolkit for IntelliJ para depurar de forma remota aplicaciones de Apache Spark mediante SSH](spark/apache-spark-intellij-tool-debug-remotely-through-ssh.md)
* [Uso de Azure Toolkit for IntelliJ para depurar de forma remota aplicaciones de Apache Spark mediante VPN](spark/apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Uso de las herramientas de HDInsight de Azure Toolkit for Eclipse con el fin de crear aplicaciones Apache Spark](spark/apache-spark-eclipse-tool-plugin.md)
* [Uso de las herramientas de HDInsight para IntelliJ con Hortonworks Sandbox](hadoop/hdinsight-tools-for-intellij-with-hortonworks-sandbox.md)
* [Uso de cuadernos de Apache Zeppelin con un clúster de Apache Spark en HDInsight](spark/apache-spark-zeppelin-notebook.md)
* [Kernels disponibles para Jupyter Notebook en un clúster de Apache Spark para HDInsight](spark/apache-spark-jupyter-notebook-kernels.md)
* [Uso de paquetes externos con cuadernos de Jupyter Notebook](spark/apache-spark-jupyter-notebook-use-external-packages.md)
* [Instalación de un cuaderno de Jupyter Notebook en el equipo y conexión al clúster de Apache Spark en HDInsight de Azure](spark/apache-spark-jupyter-notebook-install-locally.md)
* [Visualización de datos de Apache Hive con Microsoft Power BI en Azure HDInsight](hadoop/apache-hadoop-connect-hive-power-bi.md)
* [Usar Apache Zeppeling para ejecutar consultas de Apache Hive en HDInsight de Azure](./interactive-query/hdinsight-connect-hive-zeppelin.md)
