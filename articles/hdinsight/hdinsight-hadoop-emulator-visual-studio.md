---
title: Herramientas de Data Lake para Visual Studio con Sandbox de Hortonworks en Azure HDInsight
description: Aprenda a usar las herramientas de Azure Data Lake para Visual Studio con Sandbox de Hortonworks en ejecución en una máquina virtual local. Con estas herramientas puede crear y ejecutar trabajos de Hive y Pig en el espacio aislado, así como ver la salida de los trabajos y el historial.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: hrasheed
ms.openlocfilehash: 3286ca3b9c85236ff322eb19324bc5ac7a904e22
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65605453"
---
# <a name="use-the-azure-data-lake-tools-for-visual-studio-with-the-hortonworks-sandbox"></a>Uso de las herramientas de Azure Data Lake para Visual Studio con Sandbox de Hortonworks

Azure Data Lake incluye herramientas para trabajar con clústeres de Apache Hadoop genéricos. En este documento se proporcionan los pasos necesarios para usar las herramientas de Data Lake con Sandbox de Hortonworks en una máquina virtual local.

Mediante Sandbox de Hortonworks puede trabajar con Hadoop localmente en su entorno de desarrollo. Una vez que haya desarrollado una solución y quiera implementarla a escala, puede desplazarse a un clúster de HDInsight.

## <a name="prerequisites"></a>Requisitos previos

* Sandbox de Hortonworks en ejecución en una máquina virtual en el entorno de desarrollo. Este documento se ha escrito y probado con el espacio aislado ejecutado en Oracle VirtualBox, Para información sobre cómo configurar el espacio aislado, consulte el documento de [introducción al espacio aislado de Hortonworks](hadoop/apache-hadoop-emulator-get-started.md) .

* Visual Studio.

* [Azure SDK para .NET](https://azure.microsoft.com/downloads/) 2.7.1 o una versión posterior.

* Las [herramientas de Azure Data Lake para Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504).

## <a name="configure-passwords-for-the-sandbox"></a>Configuración de contraseñas para el espacio aislado

Asegúrese de que Sandbox de Hortonworks se esté ejecutando. A continuación, siga los pasos descritos en el documento de [introducción al espacio aislado de Hortonworks](hadoop/apache-hadoop-emulator-get-started.md#set-sandbox-passwords). Estos pasos configuran la contraseña de la cuenta `root` de SSH y de la cuenta `admin` de Apache Ambari. Estas contraseñas se usan al conectarse al espacio aislado desde Visual Studio.

## <a name="connect-the-tools-to-the-sandbox"></a>Conexión de las herramientas al espacio aislado

1. Abra Visual Studio, seleccione **Ver** y, luego, **Explorador de servidores**.

2. En el **Explorador de servidores**, haga clic con el botón derecho en la entrada **HDInsight** y seleccione **Conectar con el emulador de HDInsight**.

    ![Captura de pantalla del Explorador de servidores, con la opción Conectar al emulador de HDInsight resaltada](./media/hdinsight-hadoop-emulator-visual-studio/connect-emulator.png)

3. En el cuadro de diálogo **Conectar al emulador de HDInsight**, escriba la contraseña que ha configurado para Ambari.

    ![Captura de pantalla del cuadro de diálogo, con el cuadro de texto de contraseña resaltado](./media/hdinsight-hadoop-emulator-visual-studio/enter-ambari-password.png)

    Seleccione **Next** (Siguiente) para continuar.

4. Use el campo **Contraseña** para escribir la contraseña configurada para la cuenta `root`. Deje los demás campos con los valores predeterminados.

    ![Captura de pantalla del cuadro de diálogo, con el cuadro de texto de contraseña resaltado](./media/hdinsight-hadoop-emulator-visual-studio/enter-root-password.png)

    Seleccione **Next** (Siguiente) para continuar.

5. Espere a que finalice la validación de los servicios. En algunos casos, se puede producir un error de validación y se le pedirá que actualice la configuración. Si se produce un error de validación, seleccione **Actualizar** y espere a que finalicen la configuración y la comprobación del servicio.

    ![Captura de pantalla del cuadro de diálogo, con el botón Actualizar resaltado](./media/hdinsight-hadoop-emulator-visual-studio/fail-and-update.png)

    > [!NOTE]  
    > El proceso de actualización usa Ambari para modificar la configuración de Sandbox de Hortonworks en función de lo que esperan las herramientas de Data Lake para Visual Studio.

6. Una vez finalizada la validación, seleccione **Finalizar** para concluir la configuración.
    ![Captura de pantalla del cuadro de diálogo, con el botón Finalizar resaltado](./media/hdinsight-hadoop-emulator-visual-studio/finished-connect.png)

     >[!NOTE]  
     > Según la velocidad del entorno de desarrollo y de la cantidad de memoria asignada a la máquina virtual, la configuración y validación de los servicios puede tardar varios minutos.

Después de seguir estos pasos, tendrá una entrada **Clúster local de HDInsight** en la sección **HDInsight** del Explorador de servidores.

## <a name="write-an-apache-hive-query"></a>Escritura de una consulta de Apache Hive

Hive proporciona un lenguaje de consultas de tipo SQL (HiveQL) para trabajar con datos estructurados. Use los pasos siguientes para aprender a ejecutar consultas a petición en el clúster local.

1. En el **Explorador de servidores**, haga clic con el botón derecho en la entrada del clúster local agregado anteriormente y, luego, seleccione **Escribir una consulta de Hive**.

    ![Captura de pantalla del Explorador de servidores, con la opción Escribir una consulta de Hive resaltada](./media/hdinsight-hadoop-emulator-visual-studio/write-hive-query.png)

    Aparece una nueva ventana de consulta. Aquí puede escribir y enviar rápidamente una consulta al clúster local.

2. En la nueva ventana de consulta, escriba el siguiente comando:

        select count(*) from sample_08;

    Para ejecutar la consulta, seleccione **Enviar** en la parte superior de la ventana. Deje las demás opciones (**Batch** y el nombre del servidor) en los valores predeterminados.

    ![Captura de pantalla de la ventana de consulta, con el botón Enviar resaltado](./media/hdinsight-hadoop-emulator-visual-studio/submit-hive.png)

    También puede usar el menú desplegable junto a **Enviar** para seleccionar **Avanzadas**. Las opciones avanzadas permiten proporcionar opciones adicionales al enviar el trabajo.

    ![Captura de pantalla del cuadro de diálogo Enviar script](./media/hdinsight-hadoop-emulator-visual-studio/advanced-hive.png)

3. Una vez enviada la consulta, aparece el estado del trabajo. El estado del trabajo muestra información sobre el trabajo mientras lo procesa Hadoop. En **Estado del trabajo** se proporciona el estado actual del trabajo. El estado se actualiza periódicamente, aunque también se puede usar el icono de actualización para actualizarlo manualmente.

    ![Captura de pantalla del cuadro de diálogo Vista de trabajos, con la opción Estado del trabajo resaltada](./media/hdinsight-hadoop-emulator-visual-studio/job-state.png)

    Cuando el **estado del trabajo** cambie a **Finalizado**, se mostrará un grafo acíclico dirigido (DAG). Este diagrama describe la ruta de acceso de ejecución que determina Tez al procesar la consulta de Hive. Tez es el motor de ejecución predeterminado para Hive en el clúster local.

    > [!NOTE]  
    > Apache Tez también es el motor predeterminado cuando se usan clústeres de HDInsight basados en Linux. No es el motor predeterminado en los clústeres de HDInsight basados en Windows. Para usarlo en Windows, debe agregar la línea `set hive.execution.engine = tez;` al principio de la consulta de Hive.

    Use el vínculo **Salida de trabajo** para ver la salida. En este caso es 823, el número de filas de la tabla sample_08. Puede ver información de diagnóstico sobre el trabajo mediante los vínculos **Registro de trabajo** y **Descargar registro YARN**.

4. También puede ejecutar trabajos de Hive de forma interactiva cambiando el campo **Batch** a **Interactivo**. Luego, seleccione **Ejecutar**.

    ![Captura de pantalla de los botones Interactivo y Ejecutar resaltados](./media/hdinsight-hadoop-emulator-visual-studio/interactive-query.png)

    Una consulta interactiva transmite el registro de salida generado durante el procesamiento a la ventana **Salida de HiveServer2**.

    > [!NOTE]  
    > Es la misma información que está disponible en el vínculo **Registro de trabajo** una vez concluido un trabajo.

    ![Captura de pantalla del registro de salida](./media/hdinsight-hadoop-emulator-visual-studio/hiveserver2-output.png)

## <a name="create-a-hive-project"></a>Creación de un proyecto de Hive

También puede crear un proyecto que incluya varios scripts de Hive. Cuando tenga scripts relacionados o desee almacenar scripts en un sistema de control de versiones, use un proyecto.

1. En Visual Studio, seleccione **Archivo**, **Nuevo** y **Proyecto**.

2. En la lista de proyectos, expanda **Plantillas** y **Azure Data Lake** y, luego, seleccione **HIVE (HDInsight)**. En la lista de plantillas, seleccione **Hive Sample**. Escriba un nombre y una ubicación y, luego, seleccione **Aceptar**.

    ![Captura de pantalla de la ventana Nuevo proyecto, con las opciones Azure Data Lake, HIVE, Ejemplo de Hive y Aceptar resaltadas](./media/hdinsight-hadoop-emulator-visual-studio/new-hive-project.png)

El proyecto **Hive Sample** contiene dos scripts, **WebLogAnalysis.hql** y **SensorDataAnalysis.hql**. Puede enviar estos scripts con el mismo botón **Enviar**, situado en la parte superior de la ventana.

## <a name="create-an-apache-pig-project"></a>Creación de un proyecto de Apache Pig

Aunque Hive proporciona un lenguaje similar a SQL para trabajar con datos estructurados, Pig funciona mediante la realización de transformaciones de datos. Pig proporciona un lenguaje (Pig Latin) que le permite desarrollar una canalización de transformaciones. Siga estos pasos para usar Pig con el clúster local:

1. Abra Visual Studio y seleccione **Archivo**, **Nuevo** y **Proyecto**. En la lista de proyectos, expanda **Plantillas** y **Azure Data Lake** y, luego, seleccione **Pig (HDInsight)**. En la lista de plantillas, seleccione **Pig Application**. Escriba un nombre y una ubicación, y seleccione **Aceptar**.

    ![Captura de pantalla de la ventana Nuevo proyecto, con las opciones Azure Data Lake, Pig, Aplicación Pig y Aceptar resaltadas](./media/hdinsight-hadoop-emulator-visual-studio/new-pig.png)

2. Escriba el siguiente texto como contenido del archivo **script.pig** que se creó con este proyecto.

        a = LOAD '/demo/data/Website/Website-Logs' AS (
            log_id:int,
            ip_address:chararray,
            date:chararray,
            time:chararray,
            landing_page:chararray,
            source:chararray);
        b = FILTER a BY (log_id > 100);
        c = GROUP b BY ip_address;
        DUMP c;

    Aunque Pig usa un lenguaje diferente que Hive, la forma de ejecutar los trabajos es la misma en ambos lenguajes mediante el botón **Enviar**. La selección de la lista desplegable situada junto a **Enviar** muestra un cuadro de diálogo de envío avanzado para Pig.

    ![Captura de pantalla del cuadro de diálogo Enviar script](./media/hdinsight-hadoop-emulator-visual-studio/advanced-pig.png)

3. El estado y la salida de trabajo también se muestran del mismo modo que en una consulta de Hive.

    ![Captura de pantalla de un trabajo de Pig finalizado](./media/hdinsight-hadoop-emulator-visual-studio/completed-pig.png)

## <a name="view-jobs"></a>Vista de trabajos

Las herramientas de Data Lake también le permiten ver fácilmente la información sobre los trabajos ejecutados en Hadoop. Siga estos pasos para ver los trabajos que se han ejecutado en el clúster local.

1. En el **Explorador de servidores**, haga clic con el botón derecho en el clúster local y seleccione **Ver trabajos**. Se muestra una lista de trabajos que se enviaron al clúster.

    ![Captura de pantalla del Explorador de servidores, con la opción Ver trabajos resaltada](./media/hdinsight-hadoop-emulator-visual-studio/view-jobs.png)

2. En la lista de trabajos, seleccione uno para ver los detalles del trabajo.

    ![Captura de pantalla del Explorador de trabajos, con uno de los trabajos resaltado](./media/hdinsight-hadoop-emulator-visual-studio/view-job-details.png)

    La información que aparece es similar a la que se ve después de ejecutar una consulta de Hive o Pig, junto con vínculos para ver la información de registro y de salida.

3. También puede modificar y volver a enviar el trabajo desde aquí.

## <a name="view-hive-databases"></a>Vista de bases de datos de Hive

1. En el **Explorador de servidores**, expanda la entrada **Clúster local de HDInsight** y, luego, **Bases de datos de Hive**. Se muestran las bases de datos **Predeterminada** y **xademo** en el clúster local. Al expandir una base de datos se muestran las tablas que hay dentro de la base de datos.

    ![Captura de pantalla del Explorador de servidores, con las bases de datos expandidas](./media/hdinsight-hadoop-emulator-visual-studio/expanded-databases.png)

2. La expansión de una tabla muestra las columnas de esa tabla. Si quiere ver los datos rápidamente, haga clic con el botón derecho en una tabla y seleccione **Ver las 100 primeras filas**.

    ![Captura de pantalla del Explorador de servidores, con una tabla expandida y la opción Ver las 100 primeras filas seleccionada](./media/hdinsight-hadoop-emulator-visual-studio/view-100.png)

### <a name="database-and-table-properties"></a>Propiedades de las bases de datos y de las tablas

Puede ver las propiedades de una base de datos o de una tabla. Al seleccionar **Propiedades** se muestran los detalles del elemento seleccionado en la ventana de propiedades. Por ejemplo, puede ver la información que se muestra en la siguiente captura de pantalla:

![Captura de pantalla de la ventana Propiedades](./media/hdinsight-hadoop-emulator-visual-studio/properties.png)

### <a name="create-a-table"></a>Creación de una tabla

Para crear una tabla, haga clic con el botón derecho en una base de datos y seleccione **Crear tabla**.

![Captura de pantalla del Explorador de servidores, con la opción Crear tabla resaltada](./media/hdinsight-hadoop-emulator-visual-studio/create-table.png)

Ya puede crear la tabla usando un formulario. En la parte inferior de la siguiente captura de pantalla puede ver el script HiveQL sin formato que se usa para crear la tabla.

![Captura de pantalla del formulario usado para crear una tabla](./media/hdinsight-hadoop-emulator-visual-studio/create-table-form.png)

## <a name="next-steps"></a>Pasos siguientes

* [Learning the ropes of the Hortonworks Sandbox](https://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
* [Tutorial de Apache Hadoop: introducción a HDP](https://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/)
