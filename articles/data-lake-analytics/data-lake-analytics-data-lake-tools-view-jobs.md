---
title: 'Uso del explorador de trabajos y la vista de trabajo: Azure Data Lake Analytics'
description: En este artículo se describe cómo usar el explorador de trabajos y la vista de trabajo para trabajos de Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.assetid: bdf27b4d-6f58-4093-ab83-4fa3a99b5650
ms.topic: how-to
ms.date: 08/02/2017
ms.openlocfilehash: 32684ea72df63de5b82941b3ef44e9d579d09eb4
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131895"
---
# <a name="use-job-browser-and-job-view-for-azure-data-lake-analytics"></a>Usar el explorador de trabajos y la vista de trabajo para Azure Data Lake Analytics
El servicio Azure Data Lake Analytics archiva los trabajos enviados en un almacén de consultas. En este artículo aprenderá a utilizar el explorador de trabajos y la vista de trabajo de Azure Data Lake Tools para Visual Studio para encontrar la información histórica del trabajo. 

De forma predeterminada, el servicio Data Lake Analytics archiva los trabajos durante 30 días. El período de expiración se puede configurar desde Azure Portal configurando la directiva de caducidad personalizada. No podrá acceder a la información de trabajo tras la expiración. 

## <a name="prerequisites"></a>Prerrequisitos
Vea [Data Lake Tools for Visual Studio prerequisites](data-lake-analytics-data-lake-tools-get-started.md#prerequisites) (Requisitos previos de Data Lake Tools para Visual Studio).

## <a name="open-the-job-browser"></a>Abra el explorador de trabajos
Puede acceder al Explorador de trabajos a través de **Explorador de servidores>Azure>Data Lake Analytics>Trabajos** en Visual Studio.  Con el Explorador de trabajos puede acceder al almacén de consultas de una cuenta de Data Lake Analytics. El Explorador de trabajos muestra el Almacén de consultas a la izquierda, que muestra información básica sobre los trabajos y a la derecha la Vista de trabajos que muestra información detallada de los trabajos.

## <a name="job-view"></a>Vista de trabajo
La vista de trabajo muestra la información detallada de un trabajo. Para abrir un trabajo, puede hacer doble clic en un trabajo en el explorador de trabajos o abrirlo en el menú Data Lake haciendo clic en la vista de trabajo. Debe ver un cuadro de diálogo que se rellena con la dirección URL del trabajo.

![Explorador de trabajos de Data Lake Tools para Visual Studio](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view.png)

La vista de trabajo contiene:

* Resumen del trabajo
  
    Actualice la vista de trabajo para ver la información más reciente de trabajos en ejecución.
  
  * Estado del trabajo (gráfico):
    
      En Estado del trabajo se describen las fases del trabajo:
    
      ![Estado de las fases del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-phases.png)
    
    * Preparando: carga del script en la nube, compilación y optimización del script mediante el servicio de compilación.
    * En cola: los trabajos se ponen en cola cuando esperan recursos suficientes o superan el número máximo de trabajos simultáneos por limitación de cuenta. La configuración de prioridad determina la secuencia de trabajos en cola: cuanto menor sea el número, mayor será la prioridad.
    * En ejecución: el trabajo se ejecuta realmente en la cuenta de Data Lake Analytics.
    * Finalizando: el trabajo se está completando (por ejemplo, finalizando el archivo).
      
      Se pueden producir errores en todas las fases del trabajo. Por ejemplo, errores de compilación en la fase de preparación, errores de tiempo de espera en la fase de puesta en cola y errores de ejecución en la fase de ejecución, etc.
  * Información básica
    
      La información básica del trabajo se muestra en la parte inferior del panel de resumen del trabajo.
    
      ![Estado de las fases del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-info.png)
    
    * Resultado del trabajo: correcto o con error. Puede que se produzcan errores en todas las fases.
    * Duración total: tiempo de reloj (duración) entre la hora de envío y la hora de finalización.
    * Tiempo de proceso total: suma del tiempo de ejecución de cada vértice, puede considerarse como el tiempo en que el trabajo se ejecuta en un solo vértice. Consulte Total de vértices para obtener más información sobre vértices.
    * Hora de envío/inicio/finalización: hora en que el servicio Data Lake Analytics recibe el envío del trabajo/empieza a ejecutar el trabajo/finaliza el trabajo correctamente o no.
    * Compilación/En cola/En ejecución: tiempo de reloj empleado durante la fase de puesta en cola, de preparación y de ejecución.
    * Cuenta: cuenta de Data Lake Analytics utilizada para ejecutar el trabajo.
    * Autor: usuario que envió el trabajo, puede ser una cuenta del sistema o la cuenta de una persona real.
    * Prioridad: prioridad del trabajo. Cuanto menor sea el número, mayor será la prioridad. Solo afecta a la secuencia de los trabajos en la cola. Al establecer una prioridad mayor no se adelantan trabajos en ejecución.
    * Paralelismo: número máximo solicitado de unidades de Azure Data Lake Analytics (ADLAU) simultáneas, también llamadas vértices. Actualmente, un vértice es igual a una máquina virtual con dos núcleos virtuales y seis GB de RAM, aunque esto podría aumentar en futuras actualizaciones de Data Lake Analytics.
    * Bytes restantes: bytes que deben procesarse hasta que se complete el trabajo.
    * Bytes leídos/escritos: bytes que se han leído/escrito desde que se inició la ejecución del trabajo.
    * Vértices totales: el trabajo se divide en muchos elementos de trabajo, cada elemento de trabajo se denomina un vértice. Este valor describe de cuántos elementos de trabajo está formado el trabajo. Un vértice puede considerarse como una unidad de proceso básico, también denominada unidad de Azure Data Lake Analytics (ADLAU), y los vértices se pueden ejecutar en paralelismo. 
    * Completado/En ejecución/Con error: número de vértices completados, en ejecución o con errores. Los vértices pueden no ejecutarse debido a errores del sistema y de código de usuario, pero el sistema intenta automáticamente ejecutar los vértices con errores varias veces. Si el vértice sigue dando errores después de intentar ejecutarlo varias veces, se producirá un error en todo el trabajo.
* Gráfico del trabajo
  
    Un script U-SQL representa la lógica de transformación de datos de entrada en datos de salida. El script se compila y optimiza para un plan de ejecución física en la fase de preparación. El objeto del gráfico del trabajo es mostrar el plan de ejecución física.  En el siguiente diagrama se ilustra este proceso:
  
    ![Estado de las fases del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-logical-to-physical-plan.png)
  
    Un trabajo se divide en muchos elementos de trabajo. Cada elemento de trabajo se denomina vértice. Los vértices se agrupan como supervértice (también llamado fase) y se visualizan como gráfico del trabajo. Los paneles de fase verdes del gráfico del trabajo muestran las fases.
  
    Cada vértice de una fase realiza el mismo tipo de trabajo con diferentes partes de los mismos datos. Por ejemplo, si tiene un archivo con un TB de datos y hay cientos de vértices leyendo en ellos, cada uno lee un fragmento. Los vértices se agrupan en la misma fase y hacen el mismo trabajo en distintas partes del mismo archivo de entrada.
  
  * <a name="state-information"></a>Información de fase
    
      En una fase determinada, se muestran algunos números en el panel.
    
      ![Fase del gráfico del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-stage.png)
    
    * SV1 Extract (Extracción SV1): nombre de una fase, denominada mediante un número y el método de operación.
    * 84 vertices (84 vértices): recuento total de vértices en esta fase. La cifra indica en cuántos elementos de trabajo se divide esta fase.
    * 12.90 s/vertex (12,90 s/vértice): tiempo promedio de ejecución de los vértices de esta fase. La cifra se calcula mediante SUM (tiempo de ejecución de cada vértice)/(recuento total de vértices). Lo que significa que si se pudiera asignar que todos los vértices se ejecuten en paralelismo, la fase entera se completaría en 12,90 segundos. También significa que si todo el trabajo de esta fase se realiza en serie, el coste sería n.º de vértices*tiempo medio.
    * 850,895 rows written (850 895 filas escritas): número total de filas escritas en esta fase.
    * R (lectura) y W (escritura): cantidad de datos leídos/escritos en esta fase, en bytes.
    * Colores: colores que se usan en la fase para indicar distintos estados de vértice.
      
      * El color verde indica que el vértice se ha ejecutado correctamente.
      * El color naranja indica que el vértice se ha intentado ejecutar de nuevo. El reintento de ejecución del vértice produjo errores pero el sistema vuelve a intentarlo y lo consigue, y la fase general se completa correctamente. Si se reintenta ejecutar el vértice pero sigue dando errores, el color cambia a rojo y se producirá un error en todo el trabajo.
      * El color rojo indica errores, lo que significa que el sistema intentó ejecutar varias veces un vértice determinado, pero no lo consiguió. Este escenario provoca un error todo el trabajo.
      * El color azul indica que un vértice determinado está en ejecución.
      * El color blanco indica que el vértice está en espera. Es posible que el vértice esté esperando a ser programado en cuanto una ADLAU pase a estar disponible, o bien, puede que esté en espera de entrada porque sus datos de entrada no estén preparados.
      
      Puede encontrar más detalles de la fase si mueve el cursor del mouse sobre un estado:
      
      ![Detalles de la fase del gráfico del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-stage-details.png)
  * Vértices: describe los detalles de los vértices; por ejemplo, cuántos vértices hay en total, cuántos vértices se completaron, si tienen errores o siguen en ejecución, en espera, etc.
  * Datos leídos en un pod o entre pods: los archivos y datos se almacenan en varios pods en un sistema de archivos distribuido. Este valor describe la cantidad de datos leídos en un mismo pod o entre pods.
  * Tiempo de proceso total: suma del tiempo de ejecución de cada vértice de la fase, puede considerarse como el tiempo que se tardaría si todo el trabajo de la fase se ejecutase en un solo vértice.
  * Datos y filas escritos/leídos: indica cuantos datos o filas se han leído/escrito o hay que leer.
  * Errores de lectura de vértice: describe cuántos vértices produjeron errores al leer datos.
  * Descartes de duplicado de vértice: si un vértice se ejecuta demasiado despacio, puede que el sistema programe que varios vértices ejecuten el mismo elemento de trabajo. Los vértices redundantes se descartarán cuando uno de los vértices se complete correctamente. Vertex duplicate discards: (Descartes de duplicados de vértice) registra el número de vértices que se descartan como duplicados en la fase.
  * Revocaciones de vértice: el vértice se ejecutó correctamente, pero por algún motivo se vuelven a ejecutar después. Por ejemplo, si un vértice de canal de bajada pierde datos de entrada intermedios, le pedirá al vértice de canal de subida que vuelva a ejecutarse.
  * Ejecuciones programadas del vértice: tiempo total que se han programado los vértices.
  * Lectura de datos mín./media/máx. de vértice: cantidad mínima, media y máxima de datos leídos en cada vértice.
  * Duración: tiempo de reloj que tarda una fase; debe cargar el perfil para ver este valor.
  * Reproducción del trabajo
    
      Data Lake Analytics ejecuta los trabajos y archiva la información de ejecución de vértices de los trabajos, como cuándo se inician los vértices, se detienen o generan errores, y cómo se reintentan, etc. Toda la información se registra automáticamente en el almacén de consultas y se almacena en su perfil del trabajo. Puede descargar el perfil del trabajo a través de "Perfil de carga" en la vista de trabajo y ver la reproducción del trabajo después de descargar el perfil del trabajo.
    
      Reproducción de trabajo es una visualización perfecta de lo que sucedió en el clúster. Permite ver el progreso de la ejecución del trabajo y detectar visualmente los cuellos de botella y las anomalías de rendimiento en muy poco tiempo (normalmente, menos de 30 segundos).
  * Presentación del mapa térmico del trabajo 
    
      Mapa térmico del trabajo se puede seleccionar a través de la lista desplegable Mostrar del gráfico del trabajo. 
    
      ![Presentación del mapa de montón de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-heat-map-display.png)
    
      Muestra el mapa térmico de E/S, tiempo y rendimiento de un trabajo, mediante el que puede encontrar donde pasa el trabajo la mayor parte del tiempo o si se trata de un trabajo con límite de E/S, etc.
    
      ![Ejemplo del mapa de montón de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-heat-map-example.png)
    
    * Progreso: progreso de la ejecución del trabajo, consulte la Información de fase.
    * Datos leídos/escritos: mapa térmico del total de datos leídos o escritos en cada fase.
    * Tiempo de proceso: mapa térmico de SUM (tiempo de ejecución de cada vértice), puede considerarse como el tiempo que se tardaría si todo el trabajo de la fase se ejecutase en un solo vértice.
    * Tiempo de ejecución promedio por nodo: mapa térmico de SUM (tiempo de ejecución de cada vértice)/(número de vértices). Lo que significa que si se pudiera asignar que todos los vértices se ejecuten en paralelismo, la fase entera se hará en este período de tiempo.
    * Rendimiento de entrada/salida: mapa térmico de rendimiento de entrada y salida de cada fase, mediante el que puede confirmar si se trata de un trabajo con límite de E/S.
* Operaciones de metadatos
  
    Puede realizar algunas operaciones de metadatos en el script U-SQL, como crear una base de datos, eliminar una tabla, etc. Estas operaciones se muestran en la operación de metadatos después de la compilación. Aquí puede encontrar aserciones, crear entidades y eliminar entidades.
  
    ![Operaciones de metadatos de vista de trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-metadata-operations.png)
* Historial de los estados
  
    El historial de los estados también se visualiza en el resumen del trabajo, pero aquí se ofrecen más detalles. Puede encontrar la información detallada, como, por ejemplo, cuándo el trabajo está preparado, en cola, iniciado, en ejecución o finalizado. También puede consultar cuántas veces se ha compilado el trabajo (línea CcsAttempts: 1), cuando se envía realmente el trabajo al clúster (línea con el texto Detail: Dispatching job to cluster [Detalles: Enviando trabajo al clúster]), etc.
  
    ![Historial de los estados de vista de trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-state-history.png)
* Diagnóstico
  
    La herramienta diagnostica automáticamente la ejecución del trabajo. Recibirá alertas cuando haya errores o problemas de rendimiento en sus trabajos. Tenga en cuenta que debe descargar el perfil para obtener toda la información aquí. 
  
    ![Diagnóstico de vista de trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-diagnostics.png)
  
  * Advertencias: aquí aparece una alerta con una advertencia del compilador. Puede hacer clic en el vínculo "x problemas" para obtener más detalles cuando aparezca la alerta.
  * Vertex run too long (Ejecución de vértice demasiado larga): si algún vértice agota el tiempo (digamos, 5 horas), aquí se encontrarán los problemas.
  * Uso de recursos: si ha asignado más paralelismo del necesario o insuficiente, aquí se encontrarán problemas. También puede hacer clic en Uso de recursos para ver más detalles y ejecutar escenarios hipotéticos para encontrar una mejor asignación de recursos (si desea más información, vea esta guía).
  * Comprobación de memoria: si algún vértice utiliza más de 5 GB de memoria, aquí se encontrarán problemas. Puede que el sistema termine la ejecución del trabajo si este usa más memoria que la limitación del sistema.

## <a name="job-detail"></a>Detalles del trabajo
Muestra la información detallada del trabajo, que incluye el script, los recursos y la vista de ejecución de vértices.

![Detalles del trabajo de Azure Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-details.png)

* Script
  
    El script U-SQL del trabajo se almacena en el almacén de consultas. Puede ver el script U-SQL original y volver a enviarlo si es necesario.
* Recursos
  
    Aquí puede encontrar los resultados de compilación del trabajo almacenados en el almacén de consultas. Por ejemplo, puede encontrar "algebra.xml", que se utiliza para mostrar el gráfico del trabajo, los ensamblados que se han registrado, etc.
* Vista de ejecución de vértices
  
    Muestra detalles de la ejecución de vértices. El perfil del trabajo archiva cada registro de ejecución de vértice, como el total de los datos leídos o escritos, el tiempo de ejecución, el estado, etc. Mediante esta vista, puede obtener más detalles sobre cómo se ejecutó un trabajo. Para más información, vea [Use the Vertex Execution View in Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md) (Usar la vista de ejecución de vértices de Data Lake Tools para Visual Studio).

## <a name="next-steps"></a>Pasos siguientes
* Para registrar información de diagnóstico, consulte [Accessing Diagnostics logs for Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md)
* Para ver una consulta más compleja, consulte la página sobre el [análisis de registros de sitio web mediante Análisis de Azure Data Lake](data-lake-analytics-analyze-weblogs.md).
* Para usar la vista de ejecución de vértices, vea [Use the Vertex Execution View in Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md) (Uso de la vista de ejecución de vértices de Data Lake Tools para Visual Studio).

