---
title: Información de rendimiento de consultas
description: La supervisión del rendimiento de consultas identifica las consultas que consumen más CPU y tardan más en ejecutarse en bases de datos únicas y agrupadas en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: jrasnik, carlrab
ms.date: 03/10/2020
ms.openlocfilehash: 76be966d8202bb56a6762a261be5adeaf2d58d6b
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926406"
---
# <a name="query-performance-insight-for-azure-sql-database"></a>Información de rendimiento de consultas para Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Información de rendimiento de consultas proporciona un análisis inteligente de las consultas para bases de datos únicas y agrupadas. Ayuda a identificar las consultas que consumen más recursos y las que más tardan ej ejecutarse en la carga de trabajo. Esto le ayudará a encontrar consultas que optimizan el rendimiento general de la carga de trabajo y que usan de forma eficaz el recurso por el que se paga. Información de rendimiento de consultas ayuda a dedicar menos tiempo a solucionar problemas de rendimiento de bases de datos, ya que proporciona:

* Información más detallada sobre el consumo de recursos (DTU) de las bases de datos
* Detalles sobre las principales consultas de una base de datos por CPU, duración y recuento de ejecuciones (las posibles candidatas a un ajuste para mejorar el rendimiento)
* La posibilidad de explorar en profundidad los detalles de una consulta, ver su texto y el historial de uso de recursos
* Anotaciones que muestran recomendaciones de rendimiento de [instancias de Database Advisor](database-advisor-implement-performance-recommendations.md)

![Información de rendimiento de consultas](./media/query-performance-insight-use/opening-title.png)

## <a name="prerequisites"></a>Requisitos previos

Información de rendimiento de consultas requiere que el [Almacén de consultas](https://msdn.microsoft.com/library/dn817826.aspx) esté activo en la base de datos. Se habilita automáticamente para todas las bases de datos de Azure SQL Database de forma predeterminada. Si el Almacén de consultas no está en ejecución, Azure Portal le pedirá que lo habilite.

> [!NOTE]
> Si aparece un mensaje en el portal que dice que el Almacén de consultas no está configurado correctamente en esta base de datos, consulte [Optimización de la configuración del almacén de consultas](#optimize-the-query-store-configuration).

## <a name="permissions"></a>Permisos

Necesitará los siguientes permisos de [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md) para usar Información de rendimiento de consultas:

* Se requieren permisos de **lector**, **propietario**, **colaborador**, **colaborador de base de datos SQL** o **colaborador de SQL Server** para ver las consultas y los gráficos que más recursos consumen.
* Se requieren permisos de **propietario**, **colaborador**, **colaborador de base de datos SQL** o **colaborador de SQL Server** para ver el texto de la consulta.

## <a name="use-query-performance-insight"></a>Uso de Información de rendimiento de consultas

Query Performance Insight es fácil de usar:

1. Abra [Azure Portal](https://portal.azure.com/) y busque la base de datos que quiere examinar.
2. En el menú izquierdo, abra **Rendimiento inteligente** > **Información de rendimiento de consultas**.
  
   ![Información de rendimiento de consultas en el menú](./media/query-performance-insight-use/tile.png)

3. En la primera pestaña, revise la lista de consultas que más recursos consumen.
4. Seleccione una consulta individual para ver sus detalles.
5. Abra **Rendimiento inteligente** > **Recomendaciones de rendimiento** y compruebe si existe alguna recomendación sobre el rendimiento. Para obtener más información sobre las recomendaciones de rendimiento integradas, vea [Azure SQL Database Advisor](database-advisor-implement-performance-recommendations.md).
6. Use los controles deslizantes o los iconos de zoom para cambiar el intervalo observado.

   ![Panel de rendimiento](./media/query-performance-insight-use/performance.png)

> [!NOTE]
> Para que Azure SQL Database represente la información de Información de rendimiento de consultas, el Almacén de consultas debe capturar un par de horas de datos. Si la base de datos no tiene actividad o si el Almacén de consultas no ha estado activo durante un período determinado, los gráficos estarán vacíos cuando Información de rendimiento de consultas muestre ese intervalo de tiempo. Puede habilitar el Almacén de consultas en cualquier momento si no se está ejecutando. Para más información, consulte [Procedimiento recomendado con el Almacén de consultas](https://docs.microsoft.com/sql/relational-databases/performance/best-practice-with-the-query-store).
>

Para ver las recomendaciones de rendimiento de base de datos, seleccione [Recommendations](database-advisor-implement-performance-recommendations.md) (Recomendaciones) en la hoja de navegación de Información de rendimiento de consultas.

![La pestaña Recommendations (Recomendaciones)](./media/query-performance-insight-use/ia.png)

## <a name="review-top-cpu-consuming-queries"></a>Revisión de las consultas que más CPU consumen

De forma predeterminada, Información de rendimiento de consultas muestra las cinco consultas que más CPU consumen cuando se abre por primera vez.

1. Use las casillas para seleccionar o deseleccionar las distintas consultas para incluirlas o excluirlas del gráfico.

   La línea superior muestra el porcentaje general de DTU para la base de datos. Las barras muestran el porcentaje de CPU que consumen las consultas seleccionadas durante el intervalo seleccionado. Por ejemplo, si se selecciona **Semana anterior**, cada barra representa un solo día.

   ![Principales consultas](./media/query-performance-insight-use/top-queries.png)

   > [!IMPORTANT]
   > La línea de DTU mostrada se agrega a un valor de consumo máximo en períodos de una hora. Se utiliza para la comparación de alto nivel solo con estadísticas de ejecución de consultas. En algunos casos, la utilización de DTU puede parecer demasiado alta en comparación con las consultas ejecutadas, pero podría no ser el caso.
   >
   > Por ejemplo, si una consulta aumenta la DTU al 100 % durante solo unos minutos, la línea de DTU en Información de rendimiento de consultas mostrará la hora completa de consumo como 100 % (el resultado del valor agregado máximo).
   >
   > Para una comparación más precisa (hasta un minuto), considere la posibilidad de crear un gráfico de utilización de DTU personalizado:
   >
   > 1. En Azure Portal, seleccione **Azure SQL Database** > **Supervisión**.
   > 2. Seleccione **Métricas**.
   > 3. Seleccione **+Agregar un gráfico**.
   > 4. Seleccione el porcentaje de DTU en el gráfico.
   > 5. Además, seleccione **Últimas 24 horas** en el menú superior izquierdo y cámbielo a un minuto.
   >
   > Use el gráfico de DTU personalizado con un mayor nivel de detalles para compararlo con el gráfico de ejecución de consultas.

   La cuadrícula inferior muestra información agregada para las consultas visibles:

   * El identificador de consulta, que es un identificador único para la consulta en la base de datos.
   * La CPU por consulta durante un intervalo observable, que depende de la función de agregación.
   * La duración por consulta, que también depende de la función de agregación.
   * El número total de ejecuciones de una consulta específica.

2. Si los datos se vuelven obsoletos, haga clic en el botón **Actualizar**.

3. Puede usar los controles deslizantes y botones de zoom para cambiar el intervalo de observación e investigar los picos de consumo:

   ![Controles deslizantes y botones de zoom para cambiar el intervalo](./media/query-performance-insight-use/zoom.png)

4. También, puede seleccionar la pestaña **Personalizado** para personalizar la vista para:

   * Métrica (CPU, duración, recuento de ejecuciones).
   * Intervalo de tiempo (últimas 24 horas, semana anterior o mes anterior).
   * Número de consultas.
   * Función de agregación.
  
   ![Pestaña Personalizado](./media/query-performance-insight-use/custom-tab.png)
  
5. Seleccione el botón **Go >** (Ir >) para ver la vista personalizada.

   > [!IMPORTANT]
   > Información de rendimiento de consultas está limitado a mostrar entre las 5 y las 20 consultas que más consumen, según su selección. La base de datos puede ejecutar muchas más consultas más allá de las principales que se muestran, pero estas no se incluirán en el gráfico.
   >
   > Podría haber un tipo de carga de trabajo de base de datos en la que muchas consultas más pequeñas, más allá de las principales mostradas, se ejecutan con frecuencia y usan la mayoría de DTU. Estas consultas no aparecen en el gráfico de rendimiento.
   >
   > Por ejemplo, una consulta podría haber consumido una cantidad importante de DTU durante un tiempo, aunque su consumo total en el período observado sea inferior al de las otras consultas que más consumen. En tal caso, el uso de recursos de esta consulta no aparecería en el gráfico.
   >
   > Si necesita comprender la ejecución de consultas principales más allá de las limitaciones de Información de rendimiento de consultas, considere el uso de [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md) para la supervisión y la solución de problemas del rendimiento de la base de datos de manera avanzada.
   >

## <a name="view-individual-query-details"></a>Visualización de detalles de consultas individuales

Para ver los detalles de una consulta:

1. Seleccione cualquier consulta de la lista de consultas principales.

    ![Lista de consultas principales](./media/query-performance-insight-use/details.png)

   Se abre una vista detallada. En ella se muestran el consumo de CPU, la duración y el recuento de ejecuciones con el tiempo.

2. Seleccione las características del gráfico para ver los detalles.

   * El gráfico superior muestra una línea con el porcentaje general de DTU de base de datos. Las barras son el porcentaje de CPU que consume la consulta seleccionada.
   * El segundo gráfico muestra la duración total de la consulta seleccionada.
   * El gráfico inferior muestra el número total de ejecuciones por la consulta seleccionada.

   ![Detalles de la consulta](./media/query-performance-insight-use/query-details.png)

3. Opcionalmente, puede usar controles deslizantes, botones de zoom o seleccionar **Configuración** para personalizar la forma en que se muestran los datos de consulta o para seleccionar otro intervalo de tiempo.

   > [!IMPORTANT]
   > Información de rendimiento de consultas no captura las consultas DDL. En algunos casos, no puede capturar todas las consultas ad hoc.
   >

## <a name="review-top-queries-per-duration"></a>Revisión de las principales consultas por duración

Dos métricas de Información de rendimiento de consultas pueden ayudarle a encontrar posibles cuellos de botella: duración y recuento de ejecuciones.

Las consultas de larga ejecución tienen el máximo potencial para bloquear recursos durante más tiempo, bloquear otros usuarios y limitar la escalabilidad. También son las mejores candidatas para la optimización.

Para identificar consultas de larga ejecución:

1. Abra la pestaña **Custom** (Personalizado) en Información de rendimiento de consultas para la base de datos seleccionada.
2. Cambie las métricas a **duración**.
3. Seleccione el número de consultas y el intervalo de observación.
4. Seleccione la función de agregación:

   * **Sum** aumenta el tiempo de ejecución de todas las consultas durante todo el intervalo de observación.
   * **Max** busca consultas cuyo tiempo de ejecución era máximo en todo el intervalo de observación.
   * **Avg** busca el tiempo medio de ejecución de todas las ejecuciones de consulta y le muestra las principales para estos promedios.

   ![Duración de la consulta](./media/query-performance-insight-use/top-duration.png)

5. Seleccione el botón **Go >** (Ir >) para ver la vista personalizada.

   > [!IMPORTANT]
   > Al ajustar la vista de consulta no se actualiza la línea de DTU. La línea de DTU muestra siempre el valor de consumo máximo para el intervalo.
   >
   > Para comprender el consumo de DTU de la base de datos con más detalle (hasta un minuto), considere la posibilidad de crear un gráfico personalizado en Azure Portal:
   >
   > 1. Seleccione **Azure SQL Database** > **Supervisión**.
   > 2. Seleccione **Métricas**.
   > 3. Seleccione **+Agregar un gráfico**.
   > 4. Seleccione el porcentaje de DTU en el gráfico.
   > 5. Además, seleccione **Últimas 24 horas** en el menú superior izquierdo y cámbielo a un minuto.
   >
   > Se recomienda usar el gráfico de DTU personalizado para compararlo con el gráfico de rendimiento de consultas.
   >

## <a name="review-top-queries-per-execution-count"></a>Revisión de las principales consultas por recuento de ejecuciones

Una aplicación de usuario que usa la base de datos podría ralentizarse, incluso aunque puede que un elevado número de ejecuciones no afecte a la base de datos propiamente dicha y el uso de recursos sea bajo.

En algunos casos, un recuento de ejecuciones alto puede provocar más recorridos de ida y vuelta de red. Los recorridos de ida y vuelta afectan al rendimiento. Están sujetos a la latencia de red y a la latencia del servidor que sigue en la cadena.

Por ejemplo, muchos sitios web controlados por datos tienen acceso en gran medida a la base de datos para cada solicitud de usuario. Aunque la agrupación de conexiones ayuda, el mayor tráfico de red y el procesamiento de la carga en el servidor pueden ralentizar el rendimiento. En general, reduzca los recorridos de ida y vuelta al mínimo.

Para identificar las consultas que se ejecutan con frecuencia ("fragmentadas"):

1. Abra la pestaña **Custom** (Personalizado) en Información de rendimiento de consultas para la base de datos seleccionada.
2. Cambiar las métricas a **recuento de ejecuciones**.
3. Seleccione el número de consultas y el intervalo de observación.
4. Seleccione el botón **Go >** (Ir >) para ver la vista personalizada.

   ![Recuento de ejecuciones de consultas](./media/query-performance-insight-use/top-execution.png)

## <a name="understand-performance-tuning-annotations"></a>Anotaciones de ajuste del rendimiento

Al explorar su carga de trabajo en Información de rendimiento de consultas, puede que observe iconos con una línea vertical en la parte superior del gráfico.

Estos iconos son anotaciones. Muestran las recomendaciones de rendimiento de [Azure SQL Database Advisor](database-advisor-implement-performance-recommendations.md). Si mantiene el puntero sobre una anotación, puede obtener información resumida sobre las recomendaciones de rendimiento.

   ![Anotación de consulta](./media/query-performance-insight-use/annotation.png)

Si desea más información o quiere aplicar la recomendación del asesor, seleccione el icono para abrir los detalles de la acción recomendada. Si se trata de una recomendación activa, puede aplicarla inmediatamente desde el portal.

   ![Detalles de la anotación de consulta](./media/query-performance-insight-use/annotation-details.png)

En algunos casos, y debido al nivel de zoom, es posible que las anotaciones que se encuentran unas al lado de otras se contraigan en una sola. Información de rendimiento de consultas las representa como un icono de grupo de anotaciones. Al seleccionar el icono de grupo de anotaciones se abre una nueva hoja en la que se enumeran las anotaciones.

La correlación de consultas y las acciones de ajuste del rendimiento pueden ayudarle a comprender mejor la carga de trabajo.

## <a name="optimize-the-query-store-configuration"></a>Optimización de la configuración del Almacén de consultas

Al usar Información de rendimiento de consultas, es posible que vea los siguientes mensajes de error del Almacén de consultas:

* "El Almacén de consultas no está configurado correctamente en esta base de datos. Haga clic aquí para más información".
* "El Almacén de consultas no está configurado correctamente en esta base de datos. Haga clic aquí para cambiar la configuración".

Estos mensajes suelen aparecer cuando el Almacén de consultas no puede recopilar nuevos datos.

El primer caso sucede si el Almacén de consultas está en estado de solo lectura y los parámetros están óptimamente establecidos. Para solucionar este problema, puede aumentar el tamaño del almacén de datos o desactivar el Almacén de consultas. (Si desactiva el Almacén de consultas, todos los datos de telemetría recopilados previamente se perderán).

   ![Detalles del Almacén de consultas](./media/query-performance-insight-use/qds-off.png)

El segundo caso sucede cuando el Almacén de consultas no está habilitado o los parámetros no están establecidos de manera óptima. Puede cambiar la directiva de retención y captura y también habilitar el Almacén de consultas mediante la ejecución de los siguientes comandos proporcionados en [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) o Azure Portal.

### <a name="recommended-retention-and-capture-policy"></a>Directiva de retención y captura recomendada

Hay dos tipos de directivas de retención:

* **Basada en el tamaño**: si se establece en **AUTOMÁTICA**, los datos se borran automáticamente cuando se haya alcanzado casi el tamaño máximo.
* **Basada en el tiempo**: de forma predeterminada, esta directiva se establece en 30 días. Si el Almacén de consultas se queda sin espacio, eliminará la información de consultas que tenga más de 30 días.

Puede establecer la directiva de captura en:

* **Todos**: el Almacén de consultas captura todas las consultas.
* **Automático**: el Almacén de consultas ignora las consultas poco frecuentes y las consultas con una duración de ejecución y compilación insignificante. Los umbrales del recuento de ejecuciones, la duración de compilación y la duración del tiempo de ejecución se determinan internamente. Ésta es la opción predeterminada.
* **Ninguna**: el Almacén de consultas deja de capturar nuevas consultas, pero se siguen recopilando estadísticas del tiempo de ejecución de las consultas ya capturadas.

Se recomienda establecer todas las directivas en **Automática** y la directiva de limpieza en 30 días mediante la ejecución de los siguientes comandos desde [SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) o Azure Portal. (Reemplace `YourDB` por el nombre de la base de datos).

```sql
    ALTER DATABASE [YourDB]
    SET QUERY_STORE (SIZE_BASED_CLEANUP_MODE = AUTO);

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (QUERY_CAPTURE_MODE = AUTO);
```

Aumente el tamaño del Almacén de consultas; para ello, conéctese a una base de datos mediante [SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) o Azure Portal y ejecute la consulta siguiente. (Reemplace `YourDB` por el nombre de la base de datos).

```SQL
    ALTER DATABASE [YourDB]
    SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 1024);
```

Al aplicar esta configuración, el Almacén de consultas recopilará finalmente la telemetría de las nuevas consultas. Si necesita que el Almacén de consultas esté operativo enseguida, tiene la opción de borrarlo; para ello, ejecute la siguiente consulta mediante SSMS o Azure Portal. (Reemplace `YourDB` por el nombre de la base de datos).

> [!NOTE]
> Al ejecutar la siguiente consulta se elimina toda la telemetría supervisada recopilada previamente en el Almacén de consultas.

```SQL
    ALTER DATABASE [YourDB] SET QUERY_STORE CLEAR;
```

## <a name="next-steps"></a>Pasos siguientes

Plantéese la posibilidad de usar [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md) para la supervisión avanzada del rendimiento de una gran flota de bases de datos únicas y agrupadas, grupos elásticos, instancias administradas y bases de datos de instancias.
