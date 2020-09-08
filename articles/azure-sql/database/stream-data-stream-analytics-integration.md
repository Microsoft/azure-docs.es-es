---
title: Transmisión de datos mediante la integración de Azure Stream Analytics (versión preliminar)
description: Use la integración de Azure Stream Analytics para transmitir datos a Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: ajetasin
ms.author: ajetasi
ms.reviewer: sstein
ms.date: 11/04/2019
ms.openlocfilehash: 945e9019d75f9597d89a63c9322cbd4a8d502f15
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869871"
---
# <a name="stream-data-into-azure-sql-database-using-azure-stream-analytics-integration-preview"></a>Transmisión de datos en Azure SQL Database mediante la integración de Azure Stream Analytics (versión preliminar)

Ahora los usuarios pueden ingerir, procesar, ver y analizar datos de transmisión en tiempo real en una tabla directamente desde una base de datos en Azure SQL Database. Lo hacen en Azure Portal mediante [Azure Stream Analytics](../../stream-analytics/stream-analytics-introduction.md). Esta experiencia permite una amplia variedad de escenarios, como automóviles conectados, supervisión remota, detección de fraudes y muchos más. En Azure Portal, puede seleccionar un origen de eventos (Event Hub o IoT Hub), ver eventos entrantes en tiempo real y seleccionar una tabla para almacenar eventos. También puede escribir consultas de lenguaje de consulta de Azure Stream Analytics en el portal para transformar los eventos entrantes y almacenarlos en la tabla seleccionada. Este nuevo punto de entrada se suma a las experiencias de creación y configuración que ya existen en Stream Analytics. Esta experiencia se inicia en el contexto de la base de datos, lo que le permite configurar rápidamente un trabajo de Stream Analytics y navegar sin problemas entre la base de datos de Azure SQL Database y las experiencias de Stream Analytics.

![Flujo de Stream Analytics](./media/stream-data-stream-analytics-integration/stream-analytics-flow.png)

## <a name="key-benefits"></a>Ventajas principales

- Cambio de contexto mínimo: puede empezar a partir de una base de datos en Azure SQL Database en el portal y comenzar a ingerir datos en tiempo real en una tabla sin cambiar a ningún otro servicio.
- Número reducido de pasos: el contexto de la base de datos y la tabla se usa para configurar previamente un trabajo de Stream Analytics.
- Facilidad de uso adicional con vista previa de los datos: obtenga una vista previa de los datos entrantes desde el origen de eventos (Event Hub o IoT Hub) en el contexto de la tabla seleccionada

> [!IMPORTANT]
> Un trabajo de Azure Stream Analytics puede generar Azure SQL Database, Azure SQL Managed Instance o Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse). Para más información, vea [Salidas](../../stream-analytics/sql-database-output.md).

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo, necesitará los siguientes recursos:

- Suscripción a Azure. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- Una base de datos de Azure SQL Database. Para más información, vea [Creación de una base de datos única en Azure SQL Database](single-database-create-quickstart.md).
- Una regla de firewall que permita que el equipo se conecte al servidor. Para más información, vea [Creación de una regla de firewall de nivel de servidor](firewall-create-server-level-portal-quickstart.md).

## <a name="configure-stream-analytics-integration"></a>Configuración de la integración de Stream Analytics

1. Inicie sesión en Azure Portal.
2. Vaya a la base de datos en la que quiera ingerir los datos de transmisión. Seleccione **Stream Analytics (versión preliminar)** .

    ![Stream Analytics](./media/stream-data-stream-analytics-integration/stream-analytics.png)

3. Para empezar a ingerir los datos de transmisión en esta base de datos, seleccione **Crear**, asigne un nombre al trabajo de transmisión y, después, seleccione **Siguiente: Entrada**.

    ![configuración de los aspectos básicos del trabajo de Stream Analytics](./media/stream-data-stream-analytics-integration/create-job.png)

4. Escriba los detalles del origen del evento y, después, seleccione **Siguiente: Salida**.

   - **Tipo de entrada**: Event Hub/IoT Hub
   - **Alias de entrada**: escriba un nombre para identificar el origen del evento
   - **Suscripción**: igual que la suscripción a Azure SQL Database
   - **Espacio de nombres del centro de eventos**: nombre del espacio de nombres
   - **Nombre del centro de eventos**: nombre del centro de eventos en el espacio de nombres seleccionado
   - **Nombre de directiva de centro de eventos** (de forma predeterminada se crea uno): proporcione un nombre de directiva
   - **Grupo de consumidores del Centro de eventos** (de forma predeterminada se crea uno): proporcione un nombre para el grupo de consumidores  

      Se recomienda crear un grupo de consumidores y una directiva para cada trabajo de Azure Stream Analytics que cree desde aquí. Los grupos de consumidores solo permiten cinco lectores simultáneos, por lo que, si se proporciona un grupo de consumidores dedicado para cada trabajo, se evitarán los errores que podrían surgir en caso de que se superara ese límite. Las directivas dedicadas le permiten cambiar la clave o revocar permisos sin afectar a otros recursos.

     ![configuración de la salida del trabajo de Stream Analytics](./media/stream-data-stream-analytics-integration/create-job-output.png)

5. Seleccione la tabla en la que quiera ingerir los datos de transmisión. Cuando haya terminado, seleccione **Crear**.

   - **Nombre de usuario**, **Contraseña**: escriba las credenciales para la autenticación de SQL Server. Seleccione **Validar**.
   - **Tabla**: seleccione **Crear nueva** o **Usar existente**. En este flujo, se seleccionará **Crear**. Esto creará una tabla al iniciar el trabajo de Stream Analytics.

     ![Creación del trabajo de Stream Analytics](./media/stream-data-stream-analytics-integration/create.png)

6. Se abre una página de consulta con los detalles siguientes:

   - La **Entrada** (origen del evento de entrada) desde la que se ingerirán los datos  
   - La **Salida** (tabla de salida) que almacenará los datos transformados
   - Una [consulta SAQL](../../stream-analytics/stream-analytics-stream-analytics-query-patterns.md) de ejemplo con la instrucción SELECT.
   - **Vista previa de entrada**: muestra la instantánea de los datos entrantes más recientes del origen del evento de entrada.
     - El tipo de serialización de los datos se detecta automáticamente (JSON o CSV). Puede cambiarlo manualmente a JSON, CSV o AVRO.
     - Puede obtener una vista previa de los datos de entrada en Formato de tabla o en Formato sin procesar.
     - Si los datos que aparecen no están actualizados, seleccione **Actualizar** para ver los eventos más recientes.
     - Seleccione **Seleccionar intervalo de tiempo** para probar la consulta con un intervalo de tiempo específico de eventos entrantes.
     - Seleccione **Cargar entrada de muestra** para probar la consulta mediante la carga de un archivo JSON o CSV de ejemplo. Para más información sobre cómo probar una consulta de SAQL, vea [Prueba de un trabajo de Azure Stream Analytics con datos de ejemplo](../../stream-analytics/stream-analytics-test-query.md).

     ![Consulta de prueba](./media/stream-data-stream-analytics-integration/test-query.png)

   - **Resultados de la prueba**: seleccione **Consulta de prueba** y podrá ver los resultados de la consulta de streaming.

     ![Resultados de la prueba](./media/stream-data-stream-analytics-integration/test-results.png)

   - **Esquema de resultados de la prueba**: muestra el esquema de los resultados de la consulta de streaming después de la prueba. Asegúrese de que el esquema de resultados de la prueba coincide con el esquema de salida.

     ![Esquema de resultados de la prueba](./media/stream-data-stream-analytics-integration/test-results-schema.png)

   - **Esquema de salida**: contiene el esquema de la tabla que ha seleccionado en el paso 5 (nueva o existente).

      - Crear nuevo: si ha seleccionado esta opción en el paso 5, no verá el esquema hasta que inicie el trabajo de streaming. Al crear una tabla, seleccione el índice de tabla adecuado. Para más información sobre la indexación de tablas, vea [Índices agrupados y no agrupados descritos](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described/).
      - Usar existente: si ha seleccionado esta opción en el paso 5, verá el esquema de la tabla seleccionada.

7. Cuando haya terminado de crear y probar la consulta, seleccione **Guardar consulta**. Seleccione **Iniciar el trabajo de Stream Analytics** para empezar a ingerir los datos transformados en la tabla SQL. Una vez que haya finalizado los campos siguientes, **inicie** el trabajo.
   - **Hora de inicio de salida**: define la hora de la primera salida del trabajo.  
     - Ahora: el trabajo se iniciará ahora y procesará los datos entrantes nuevos.
     - Personalizado: el trabajo se iniciará ahora pero procesará los datos a partir de un momento determinado (que puede ser pasado o futuro). Para más información, vea [Procedimientos para iniciar un trabajo de Azure Stream Analytics](../../stream-analytics/start-job.md).
   - **Unidades de streaming**: Los precios de Azure Stream Analytics se basan en el número de unidades de streaming necesarias para procesar los datos en el servicio. Para más información, vea los [precios de Azure Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics/).
   - **Control de errores de datos de salida**:  
     - Reintento: Si se produce un error, Azure Stream Analytics volverá a intentar la escritura del evento indefinidamente hasta que esta se realice correctamente. No hay ningún tiempo de expiración para los reintentos. En última instancia, el evento que se está reintentando bloqueará todos los eventos posteriores del procesamiento. Esta opción es la directiva de control de errores de salida predeterminada.
     - Quitar: Azure Stream Analytics anulará todos los eventos de salida que produzcan un error de conversión de datos. No se pueden recuperar los eventos anulados para volver a procesarlos más adelante. Todos los errores transitorios (por ejemplo, errores de red) se reintentan independientemente de la configuración de la directiva de control de errores de salida.
   - **Configuración de salida de SQL Database**: Opción para heredar el esquema de partición del paso de consulta anterior a fin de habilitar la topología completamente paralela con múltiples escritores en la tabla. Para obtener más información, vea [Salida de Azure Stream Analytics a Azure SQL Database](../../stream-analytics/stream-analytics-sql-output-perf.md).
   - **Número máximo de lotes**: Número máximo de registros recomendado para enviarse con cada transacción de inserción masiva.  
    Para más información sobre el control de errores de salida, vea [Directivas de errores de salida en Azure Stream Analytics](../../stream-analytics/stream-analytics-output-error-policy.md).  

     ![Inicio del trabajo](./media/stream-data-stream-analytics-integration/start-job.png)

8. Una vez que haya iniciado el trabajo, verá el trabajo en ejecución en la lista y podrá realizar las acciones siguientes:
   - **Iniciar o detener el trabajo**: si el trabajo está en ejecución, lo puede detener. Si el trabajo está detenido, lo puede iniciar.
   - **Editar trabajo**: puede modificar la consulta. Si quiere realizar más cambios en el trabajo, agregar más entradas o salidas, y después abrirlo en Stream Analytics. La opción Editar está deshabilitada cuando el trabajo está en ejecución.
   - **Vista previa de la tabla de salida**: puede obtener una vista previa de la tabla en el editor de consultas SQL.
   - **Abrir en Stream Analytics**: abra el trabajo en Stream Analytics para ver, supervisar y depurar los detalles del trabajo.

     ![Trabajos de Stream Analytics](./media/stream-data-stream-analytics-integration/jobs.png)

## <a name="next-steps"></a>Pasos siguientes

- [Documentación de Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/)
- [Patrones de soluciones de Azure Stream Analytics](../../stream-analytics/stream-analytics-solution-patterns.md)
 