---
title: 'Herramientas de Azure HDInsight: entorno interactivo de PySpark'
description: Aprenda a usar Obtenga información sobre cómo usar Herramientas de Azure HDInsight para Visual Studio Code para crear y enviar consultas y scripts.
keywords: VScode,Herramientas de Azure HDInsight,Hive,Python,PySpark,Spark,HDInsight,Hadoop,LLAP,Hive interactivo,Consulta interactiva
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-python
ms.date: 04/23/2020
ms.openlocfilehash: 14f0c7177953e43b0c58b4f5432d0c08c5f03f45
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876621"
---
# <a name="set-up-the-pyspark-interactive-environment-for-visual-studio-code"></a>Configuración del entorno interactivo de PySpark para Visual Studio Code

Los pasos siguientes muestran cómo configurar el entorno interactivo de PySpark en VSCode. Este paso es solo para usuarios que no son de Windows.

Usamos el comando **python/pip** para crear un entorno virtual en la ruta de acceso principal. Si quiere usar otra versión, deberá cambiar la versión predeterminada del comando **python/pip** manualmente. Para más información, consulte [update-alternatives](https://linux.die.net/man/8/update-alternatives).

1. Instale [Python](https://www.python.org/downloads/) y [pip](https://pip.pypa.io/en/stable/installing/).

   * Instale Python desde [https://www.python.org/downloads/](https://www.python.org/downloads/). 
   * Instale pip desde [https://pip.pypa.io/en/stable/installing](https://pip.pypa.io/en/stable/installing/) (si no se instaló desde la instalación de Python).
   * Compruebe que Python y pip se han instalado correctamente mediante los siguientes comandos. (Opcional)

        ![Comando para comprobar la versión de pip para Python](./media/set-up-pyspark-interactive-environment/check-python-pip-version.png)

     > [!NOTE]
     > Se recomienda instalar Python manualmente, en lugar de usar la versión predeterminada de macOS.

2. Instale **virtualenv** con el comando siguiente.

   ```bash
   pip install virtualenv
   ```

## <a name="other-packages"></a>Otros paquetes

En Linux, si aparece el mensaje de error siguiente, ejecute los dos comandos siguientes para instalar los paquetes necesarios.

   ![Instalación del paquete libkrb5 para Python](./media/set-up-pyspark-interactive-environment/install-libkrb5-package.png)

```bash
sudo apt-get install libkrb5-dev
```

```bash
sudo apt-get install python-dev
```

Reinicie VSCode y, a continuación, vuelva al editor de VSCode y ejecute el comando **Spark: PySPark Interactive**.

## <a name="next-steps"></a>Pasos siguientes

### <a name="demo"></a>Demostración

* HDInsight para VS Code: [Vídeo](https://go.microsoft.com/fwlink/?linkid=858706)

### <a name="tools-and-extensions"></a>Herramientas y extensiones

* [Uso de Herramientas de Azure HDInsight para Visual Studio Code](hdinsight-for-vscode.md)
* [Uso de Azure Toolkit for IntelliJ para crear y enviar aplicaciones de Scala para Apache Spark](spark/apache-spark-intellij-tool-plugin.md)
* [Instalación de un cuaderno de Jupyter Notebook en el equipo y conexión al clúster de Apache Spark en HDInsight de Azure](spark/apache-spark-jupyter-notebook-install-locally.md)
