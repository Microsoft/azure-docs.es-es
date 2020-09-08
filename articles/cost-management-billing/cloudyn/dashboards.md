---
title: Visualización de las métricas principales con los paneles de Cloudyn en Azure
description: Este artículo describe cómo ver las métricas principales desde los paneles de Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: vitavor
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 03d5e3959b85a2b0bd9ae30744051895ff026b77
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691231"
---
# <a name="view-key-cost-metrics-with-dashboards"></a>Vista de las métricas clave de costos con paneles

Los paneles en Cloudyn proporcionan una visión general de los informes. Los paneles le permiten ver las métricas clave de costos en una sola vista. También proporcionan aspectos destacados sobre las tendencias comerciales para ayudarle a tomar decisiones empresariales importantes.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

Los paneles también se utilizan para crear vistas para los usuarios con responsabilidades diferentes en su organización, incluidos:

- Interventor
- Propietario de la aplicación o del proyecto
- Ingeniero de DevOps
- Ejecutivos

Los paneles se componen de widgets, y cada widget es básicamente una miniatura del informe. Haga clic en un widget para abrir su informe. Al personalizar los informes, los guarda en Mis informes y se agregan al panel.

Las versiones de los paneles difieren para los usuarios de Management (MSP), Enterprise y Cloudyn Premium. Las diferencias vienen determinadas por los niveles de acceso de la entidad. Para obtener más información sobre los niveles de acceso, consulte [Niveles de acceso de entidades](tutorial-user-access.md#entity-access-levels).

La disponibilidad de los paneles depende del tipo de cuenta de proveedor de servicios de nube que se utilice al ver los paneles. El tipo de información disponible y recopilado por Cloudyn afecta a los informes en los paneles. Por ejemplo, si no tiene una cuenta de AWS, no verá el panel S3 Tracker (Seguimiento de S3). De forma similar, si no habilita el acceso de Azure Resource Manager a Cloudyn, no verán ninguna información específica de Azure en los widgets del panel Optimizer (Optimizador).

Puede usar cualquiera de los paneles predefinidos o puede crear su propio panel con informes personalizados. Si no está familiarizado con los informes de Cloudyn, consulte [Uso de los informes de Cloudyn](use-reports.md).

## <a name="create-a-custom-dashboard"></a>Creación de un panel personalizado

Para comenzar a trabajar rápidamente con un panel personalizado, puede duplicar uno existente para usar sus propiedades. Luego, puede modificarlo para satisfacer sus necesidades. En el panel que desea copiar, haga clic en **Save As** (Guardar como). Solo puede duplicar paneles personalizados, no puede duplicar los paneles que se incluyen con Cloudyn.

Para crear un panel personalizado:

1. En la página principal, haga clic en **Add New +** (Agregar nuevo +). Se abrirá la página My Dashboard (Mi panel).  
    ![Página Mi panel en la que se agregan nuevos informes](./media/dashboards/my-dashboard.png)
2. Haga clic en **Add New Report** (Agregar nuevo informe). Se muestra el cuadro Add Report (Agregar informe).
3. Seleccione el informe que desea agregar al widget del panel. El widget se agrega al panel.
4. Repita los pasos anteriores hasta que se complete el panel.
5. Para cambiar el nombre del panel, haga clic en el nombre del panel en la página principal Dashboard (Panel) y escriba el nuevo nombre.

## <a name="modify-a-custom-dashboard"></a>Modificación de un panel personalizado

Al igual que con la creación de un panel personalizado, no se pueden modificar los paneles incluidos con Cloudyn. Para modificar un informe de panel personalizado:

1. En el panel, busque el informe que desee modificar y haga clic en **Edit** (Editar). Se muestra el informe.
2. Realice los cambios que desee para el informe y haga clic en **Save** (Guardar). El informe se actualiza y muestra los cambios.

## <a name="share-a-custom-dashboard"></a>Uso compartido de un panel personalizado

Puede compartir un panel personalizado con otros usuarios como _Public_ (Público) o _My Entity_ (Mi entidad). Cuando se comparte en Public, todos los usuarios pueden ver el panel. Solo los usuarios con acceso a la entidad actual pueden ver el panel cuando se comparte en My Entity. Los pasos para compartir un panel personalizado con Public y My Entity son similares.

Para compartir un panel personalizado en Public:

1. En un panel, haga clic en **Dashboard Settings** (Configuración del panel). Se muestra el cuadro Dashboard Settings.  
    ![configuración del panel para un panel personalizado](./media/dashboards/dashboard-options.png)
2. En el cuadro Dashboard Settings, haga clic en el símbolo de flecha y, a continuación, haga clic en **Public**. Aparece el cuadro de diálogo de confirmación Public Dashboard (Panel público).
3. Haga clic en **Yes** (Sí). El panel ahora está disponible para otros usuarios.

## <a name="delete-a-custom-dashboard-report"></a>Eliminación de un informe de panel personalizado

Puede eliminar un componente de informe personalizado del panel. Al eliminar el informe del panel, no elimina el informe de la lista de informes. Sino que, al eliminar el informe, solo se quita del panel.

Para eliminar un componente de panel, en el componente de panel, haga clic en **Delete** (Eliminar). Al hacer clic en **Delete**, el componente de panel se elimina de inmediato.

## <a name="share-a-dashboard-enterprise"></a>Uso compartido de un panel (Enterprise)

Puede compartir paneles personalizados para todos los usuarios de su organización o con los usuarios de la entidad actual. Compartir un panel puede conceder a otros usuarios una vista rápida de alto nivel de los KPI. Cuando se comparte un panel, el panel se replica automáticamente en todas sus entidades o clientes de Cloudyn. Los cambios en el panel compartido se actualizan automáticamente.

Para compartir un panel con todos los usuarios, incluidas las subentidades:

1. En la página principal del panel, haga clic en **Edit** (Editar).
2. Haga clic en **Share** (Compartir) y, a continuación, seleccione **Public** (Público).
3. Aparece el cuadro de confirmación Global Public Dashboard (Panel público global).
4. Haga clic en **Yes** (Sí) para establecer el panel como panel público global.

Para compartir un panel con todos los usuarios de una entidad actual:

1. En la página principal Dashboard (Panel), haga clic en **Edit** (Editar).
2. Haga clic en **Share** (Compartir) y, a continuación, seleccione **My Entity** (Mi entidad).
3. Haga clic en **Yes** (Sí) para establecer el panel como panel público.

## <a name="duplicate-a-custom-dashboard"></a>Duplicación de un panel personalizado

Cuando se crea un nuevo panel, puede querer usar propiedades similares de un panel existente. Puede duplicar el panel para crear otro.

Solo puede duplicar paneles personalizados. No se pueden duplicar paneles estándares.

Para duplicar (clonar) un panel personalizado:

1. En el panel que desea duplicar, haga clic en **Save As** (Guardar como). Se abre un nuevo panel con el mismo nombre y un número.
2. Cambie el nombre del panel duplicado y modifíquelo como quiera.

-O bien-

1. En Dashboard Settings (Configuración del panel), haga clic en **Save As** (Guardar como) en la línea del panel que desea duplicar.
2. Se abre el panel duplicado.
3. Cambie el nombre del panel y modifíquelo como quiera.

## <a name="set-a-default-dashboard"></a>Configuración de un panel como predeterminado

Puede establecer cualquier panel como predeterminado. Si lo establece como predeterminado, aparecerá como la pestaña del extremo izquierdo en la lista de pestañas de paneles. El panel predeterminado se muestra al abrir el portal de Cloudyn.

- Haga clic en la pestaña del panel que desea establecer como predeterminado, a continuación, haga clic en **Default** (Predeterminado) a la derecha.

-O bien-

1. Haga clic en **Dashboard Settings** (Configuración del panel) para ver la lista de paneles disponibles, y seleccione el panel que desea establecer como predeterminado.  
    ![opciones del panel para un panel predeterminado](./media/dashboards/dashboard-options.png)
2. Haga clic en **Default** (Predeterminado) en la línea del panel. Aparece el cuadro de confirmación Default Dashboard (Panel predeterminado).
3. Haga clic en **Sí**. El panel se establece como predeterminado.

## <a name="management-dashboard"></a>Panel de administración
El panel Management (Administración) (o panel MSP para los usuarios de MSP) incluye aspectos destacados de los tipos de informe principales.  
![Panel de administración que muestra los distintos informes](./media/dashboards/management-dash.png)

### <a name="cost-entity-summary-enterprise-only"></a>Cost Entity Summary (Resumen de entidad de costos) (solo en Enterprise)
Este widget resume las entidades de costo administradas, incluido el número de entidades y el número de cuentas.
- Haga clic en el widget para abrir el informe Enterprise Details (Detalles de la empresa).

### <a name="cost-over-time"></a>Cost Over Time (Costo en el tiempo)
Este widget puede ayudarle a detectar las tendencias de los costos. Indica el costo para el último día, en función de la tendencia de los últimos 30 días.
- Haga clic en el widget para abrir el informe Actual Cost Over Time (Costo real en el tiempo) para ver y filtrar los detalles adicionales.

### <a name="asset-controller"></a>Asset Controller (Controlador de recursos)
Este widget indica el número de instancias en ejecución desde el día anterior, por encima de las tendencias de uso durante los últimos 30 días.
- Haga clic en el widget para abrir el panel Asset Controller.

### <a name="unused-ri-detector"></a>Unused RI Detector (Detector de RI sin usar)
Este widget indica el número de reservas sin usar de Amazon EC2.
- Haga clic en el widget para abrir el informe Currently Unused Reservations (Reservas actualmente sin usar), donde puede ver las reservas sin usar que puede modificar.

### <a name="cost-by-service"></a>Cost by Service (Costo por servicio)
Este widget indica los costos amortizados por servicio para los últimos 30 días. Pase el mouse por el gráfico circular para ver los costos por servicio.
- Haga clic en el widget para abrir el informe Actual Cost Analysis (Análisis de costos reales).

### <a name="potential-savings"></a>Potential savings (Posibles ahorros)
Este widget muestra recomendaciones de precios por tipo de instancia para Amazon EC2 y Amazon RDS.
- Haga clic en el widget para abrir el informe Savings Analysis (Análisis de ahorro). Enumera los costos por tipo de instancia que tiene potencial de ahorro.

### <a name="compute-instances---daily-trend"></a>Compute Instances - Daily Trend (Instancias de proceso: tendencia diaria)
Este widget muestra las instancias activas por tipo, para los últimos 30 días.
- Haga clic en el widget para abrir el informe Instances Over Time (Instancias en el tiempo), donde puede ver un desglose de todas las instancias en ejecución durante los últimos 30 días.

### <a name="storage-by-department"></a>Storage by department (Almacenamiento por departamento)
Este widget muestra los servicios de almacenamiento utilizados por los departamentos. Pase el mouse por el gráfico circular para ver el consumo de almacenamiento por departamento.
- Haga clic en el widget para abrir el panel S3 Tracker (Seguimiento de S3).

## <a name="cost-controller-dashboard"></a>Panel Cost Controller (Controlador de costos)
El panel Cost Controller muestra aspectos destacados de la asignación de costos preestablecidos.  
![Panel del controlador de costos que muestra los distintos informes](./media/dashboards/cost-controller-dashboard.png)

### <a name="cost-over-time"></a>Cost Over Time (Costo en el tiempo)
Este widget le ayuda a detectar las tendencias de los costos. Indica el costo para el último día, en función de la tendencia de los últimos 30 días.
- Haga clic en el widget para abrir el informe Actual Cost Over Time (Costo real en el tiempo) para ver y filtrar los detalles adicionales.

### <a name="monthly-cost-trends"></a>Monthly Cost Trends (Tendencias de costos mensuales)
Este widget indica el gasto amortizado previsto y el gasto real desde el principio del mes.
- Haga clic en el widget para abrir el informe Current Month Projected Cost (Costo previsto del mes actual), que proporciona un resumen de costos del mes hasta la fecha.

Este informe muestra el costo desde el principio del mes, el costo del mes anterior y el costo previsto del mes actual. El costo previsto del mes actual se calcula sumando el costo mensual hasta la fecha y la proyección. La proyección se basa en el costo observado durante los últimos 30 días.

### <a name="12-month-planner"></a>12 Month Planner (Planificador de 12 meses)
Este widget indica los costos previstos para los próximos 12 meses y el ahorro potencial.
- Haga clic en el widget para abrir el informe Annual Projected Cost (Costo anual previsto).

### <a name="cost-by-service"></a>Cost by Service (Costo por servicio)
Este widget indica los costos amortizados por servicio para los últimos 30 días.
- Pase el mouse por el gráfico circular para ver los costos por servicio.
- Haga clic en el widget para abrir el informe Actual Cost Analysis (Análisis de costos reales).

### <a name="cost-by-account"></a>Cost by Account (Costo por cuenta)
Este widget indica los costos amortizados por cuenta para los últimos 30 días.
- Pase el mouse por el gráfico circular para ver los costos por cuenta.
- Haga clic en el widget para abrir el informe Actual Cost Analysis (Análisis de costos reales).

### <a name="cost-trend-by-day"></a>Cost Trend by Day (Tendencia de costos por día)
Este widget indica el gasto durante los últimos 30 días.
- Pase el mouse por el gráfico de barras para ver los costos por día.
- Haga clic en el widget para abrir el informe Actual Cost Over Time (Costo real en el tiempo).

### <a name="cost-trend-by-month---last-6-months"></a>Cost Trend by Month - Last 6 months (Tendencia de costos por mes: últimos 6 meses)

Este widget indica el gasto durante los últimos seis meses.
- Pase el mouse por el gráfico de barras para ver los costos por mes.
- Haga clic en el widget para abrir el informe Actual Cost Over Time (Costo real en el tiempo).

## <a name="asset-controller-dashboard"></a>Panel Asset Controller (Controlador de recursos)

Este panel muestra el número de instancias en ejecución, los discos disponibles y en uso, la distribución de los tipos de instancia y la información de almacenamiento.  
![Panel del controlador de recursos que muestra los distintos informes](./media/dashboards/asset-controller-dashboard.png)

### <a name="compute-instances"></a>Compute Instances (Instancias de proceso)
Este widget muestra el número de instancias en ejecución según las tendencias de uso durante los últimos 30 días.
- Haga clic en el widget para abrir el informe Instances Over Time (Instancias en el tiempo).

### <a name="disks"></a>Discos
Este widget indica el número total y el volumen de discos que están en uso y disponibles.
- Haga clic en el widget para abrir el informe Active Disks (Discos activos).

### <a name="instance-type-distribution"></a>Instance Type Distribution (Distribución del tipo de instancia)
Este widget indica los tipos de instancia en un gráfico circular.
- Haga clic en el widget para abrir el informe Instance Distribution (distribución de instancias), que proporciona un desglose de las instancias activas mediante la agregación seleccionada.

### <a name="compute-instances---daily-trend"></a>Compute Instances - Daily Trend (Instancias de proceso: tendencia diaria)
Este widget indica las instancias de proceso (puntual, reservadas y a petición) por día durante los últimos 30 días.
- Pase el mouse por el gráfico para ver el número de instancias de proceso, por tipo por día.
- Haga clic en el widget para abrir el informe Instances Over Time (Instancias en el tiempo).

### <a name="all-buckets-s3"></a>All Buckets (S3) (Todos los cubos (S3))
Este widget indica el almacenamiento S3 total y el número de objetos almacenados.
- Haga clic en el widget para abrir el panel S3 Tracker (Seguimiento de S3). El panel le ayuda a buscar, analizar y mostrar el uso y las tendencias de almacenamiento actuales.

### <a name="sql-db-instances-rds"></a>SQL DB Instances (RDS) (Instancias de SQL DB (RDS))
Este widget indica el número de instancias de Amazon RDS en ejecución, en función de la tendencia de los últimos 30 días.
- Haga clic en el widget para abrir el informe RDS Instance Over Time (Instancias RDS en el tiempo).

## <a name="optimizer-dashboard"></a>Panel Optimizer (Optimizador)
Este panel muestra las recomendaciones de reducción de tamaños, los recursos sin utilizar y ahorros potenciales.  
![Panel de optimización que muestra los distintos informes](./media/dashboards/optimizer-dashboard.png)

### <a name="ri-calculator"></a>RI Calculator (Calculadora de RI)
Este widget muestra el número de recomendaciones de compra de RI e indica el ahorro anual potencial.
- Haga clic en el widget para abrir Reserved Instance Calculator (Calculadora de instancias reservadas), donde puede determinar cuándo se deben utilizar planes de precios a petición frente a reservados.

### <a name="sizing"></a>Ajuste de tamaño
Este widget indica los tamaños recomendados y los ahorros potenciales, si se implementan.
- Haga clic en el widget para abrir el informe EC2 Cost Effective Sizing Recommendations (Recomendaciones de ajuste de tamaño de EC2 rentable).

### <a name="unused-ri-detector"></a>Unused RI Detector (Detector de RI sin usar)
Este widget indica el número de reservas sin usar de Amazon EC2.
- Haga clic en el widget para abrir el informe Currently Unused Reservations (Reservas actualmente sin usar), donde puede ver las reservas sin usar que puede modificar.

###  <a name="available-disks"></a>Available Disks (Discos disponibles)
Este widget indica el número de discos sin conectar a su implementación.
- Haga clic en el widget para abrir el informe Unattached Disks (Discos sin conectar).

### <a name="rds-ri-calculator"></a>RDS RI Calculator (Calculadora de RI de RDS)
Este widget indica el número de recomendaciones de reserva para las instancias de Amazon RDS y el ahorro potencial.
- Haga clic en el widget para abrir el informe RDS RI Buying Recommendations (Recomendaciones de compra de RI de RDS), donde puede ver las recomendaciones de Cloudyn para utilizar instancias reservadas, en lugar de instancias a petición.

### <a name="rds-sizing"></a>RDS Sizing (Ajuste del tamaño de RDS)
Este widget muestra el número de recomendaciones de ajuste de tamaño y el ahorro potencial.
- Haga clic en el widget para abrir el informe RDS Sizing Recommendations (Recomendaciones de ajuste de tamaño de RDS), que muestra recomendaciones detalladas de ajuste de tamaño de Amazon RDS.

Las recomendaciones de optimización se basan en los datos de uso y rendimiento observados en el último mes.

## <a name="s3-tracker-dashboard"></a>Panel S3 Tracker (Seguimiento de S3)
El panel S3 Tracker le ayuda a buscar, analizar y mostrar el uso y las tendencias de almacenamiento actuales.  
![Panel S3 Tracker que muestra los distintos informes](./media/dashboards/s3-tracker-dashboard.png)

### <a name="all-buckets"></a>All Buckets (Todos los cubos)
Este widget indica el tamaño total de todos los cubos, en GB, y el número total de objetos en los cubos.
- Haga clic en el widget para abrir el informe Distribution of S3 Size (Distribución del tamaño de S3). El informe le ayuda a analizar el tamaño de S3 por cubo, carpeta de nivel superior, clase de almacenamiento y estado de control de versiones.

### <a name="bucket-properties"></a>Bucket Properties (Propiedades de cubos)
Este widget indica el número total de cubos de almacenamiento.
- Haga clic en el widget para ver el informe S3 Bucket Properties (Propiedades de cubos de S3).

### <a name="scan-status"></a>Scan Status (Estado de examen)
Este widget indica cuándo se realizó el último examen de S3 y cuándo se iniciará el siguiente.
- Haga clic en el widget para abrir el informe S3 Scan Status (Estado de examen de S3).

### <a name="storage-by-bucket"></a>Storage by Bucket (Almacenamiento por cubo)
Este widget indica el porcentaje que usa cada clase de almacenamiento en cubo.
- Haga clic en el widget para abrir el informe Distribution of S3 Size (Distribución del tamaño de S3). El informe le ayuda a analizar el tamaño de S3 por cubo, carpeta de nivel superior, clase de almacenamiento y estado de control de versiones.

### <a name="number-of-objects-by-bucket"></a>Number of Objects by Bucket (Número de objetos por cubo)
Este widget indica el número de objetos por cubo en el número y porcentaje reales. Pase el mouse por el cubo para ver el total de objetos.
- Haga clic en el widget para abrir el informe Distribution of S3 Size (Distribución del tamaño de S3) (basado en el examen).

## <a name="cloud-comparison-dashboard"></a>Panel Cloud Comparison (Comparación de la nube)
El panel Cloud Comparison le ayuda a comparar los costos de distintos proveedores de nube en función de los precios, el tipo de CPU y el tamaño de RAM.  
![Panel de comparación en la nube que muestra los distintos informes](./media/dashboards/cloud-comparison-dashboard.png)

### <a name="ec2-cost-in-azure-by-instance-type"></a>EC2 Cost in Azure by Instance Type (Costo de EC2 en Azure por tipo de instancia)
Este widget indica los últimos 30 días de uso en las tasas a petición. Compara el costo actual de Amazon EC2 con al costo posible en Azure.
- Pase el mouse por las barras para comparar los costos por tipo de instancia.
- Haga clic en el widget para abrir el informe Porting Your Deployment – Cost Analysis (Traslado del análisis de implementación-costos).

### <a name="ec2-cost-in-azure"></a>EC2 Cost in Azure (Costo de EC2 en Azure)
Este widget muestra los costos actuales de Amazon EC2 y los compara con Azure. La comparación se basa en los últimos 30 días de uso en las tasas a petición.
- Haga clic en el widget para abrir el informe Porting Your Deployment – Cost Analysis (Traslado del análisis de implementación-costos).

### <a name="ec2azure-instance-type-mapping"></a>EC2/Azure Instance Type Mapping (Asignación de tipo de instancia EC2/Azure)
Este widget indica la mejor asignación de unidades de proceso elásticas entre Amazon EC2 y Azure.
- Haga clic en el widget para abrir el informe Instances Type Mapping (Asignación de tipo de instancia).

## <a name="next-steps"></a>Pasos siguientes
- Lea el artículo [Uso de los informes de Cloudyn](use-reports.md) para más información sobre los informes.
