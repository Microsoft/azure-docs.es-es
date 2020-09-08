---
title: Información sobre los informes de administración de costos de Cloudyn en Azure
description: Este artículo le ayuda a entender la estructura y las funciones básicas de los informes de administración de costos de Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 9f42359cc843a934cf1258576d13eec8af2279cc
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687661"
---
# <a name="understanding-cloudyn-cost-management-reports"></a>Descripción de los informes de administración de costos de Cloudyn

Este artículo le ayuda a entender la estructura y las funciones básicas de los informes de administración de costos de Cloudyn. La mayoría de los informes de Cloudyn son intuitivos y tienen un aspecto uniforme. Después de leer este artículo, está listo para usar todos los informes de administración de costos. Muchas características estándar están disponibles en los distintos informes, lo que le permite navegar por ellos con facilidad. Los informes son personalizables y puede seleccionar entre varias opciones para calcular y mostrar los resultados.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="report-fields-and-options"></a>Campos y opciones de informes

A continuación se muestra un ejemplo del informe de costo a lo largo del tiempo. La mayoría de los informes de Cloudyn tienen un diseño similar.

![Ejemplo del informe de costo a lo largo del tiempo con áreas numeradas que se corresponden con las descripciones](./media/understanding-cost-reports/sample-report.png)

Cada área numerada de la imagen anterior se describe detalladamente en la siguiente información:

1. **Date Range** (Intervalo de fechas)

    Utilice la lista Date Range para definir un intervalo de tiempo del informe mediante una configuración prefijada o personalizada.
2. **Saved Filter** (Filtro guardado)

    Utilice la lista Saved Filter para guardar los grupos y filtros actuales que se aplican al informe. Los filtros guardados están disponibles en todos los informes de costos y rendimiento, incluidos los siguientes:

      - Análisis de costos
      - Asignación
      - Administración de recursos
      - Optimization

   Escriba un nombre de filtro y haga clic en **Guardar**.

3. **Etiquetas**

    Utilice el área Tags (Etiquetas) para agrupar por categorías de etiquetas. Las etiquetas que aparecen en el menú son etiquetas de departamento o centro de costo de Azure, o bien, son la entidad de costo y las etiquetas de suscripción de Cloudyn. Seleccione las etiquetas para filtrar los resultados. También puede escribir un nombre de etiqueta (palabra clave) para filtrar los resultados.

    ![Ejemplo de una lista de etiquetas para filtrar los resultados](./media/understanding-cost-reports/select-options.png)

    Haga clic en **Add** (Agregar) para agregar un filtro nuevo.

    ![Agregar un cuadro de filtro que muestre las opciones y las condiciones para filtrar](./media/understanding-cost-reports/add-filter.png)

    La agrupación o el filtrado de etiquetas no se refiere a los recursos de Azure ni a las etiquetas de grupos de recursos.

    La agrupación y el filtrado de etiquetas de asignación de costos están disponibles en la opción del menú **Groups** (Grupos).

4. **Grupos en informes**

    Utilice grupos en los informes de análisis de costos para mostrar en su informe categorías desglosadas y estándar de los datos de facturación.  Sin embargo, los grupos en los informes de asignación de costos muestran categorías basadas en etiquetas. Las categorías basadas en etiquetas se definen en el modelo de asignación de costos y en las categorías desglosadas estándar de los datos de facturación.

    ![Primer ejemplo de lista de etiquetas por las que puede agrupar](./media/understanding-cost-reports/groups-tags01.png)

    ![Segundo ejemplo de lista de etiquetas por las que puede agrupar](./media/understanding-cost-reports/groups-tags02.png)

    En los informes de asignación de costos, los grupos en las categorías de grupos basados en etiquetas podrían incluir:
      - Etiquetas
      - etiquetas de grupos de recursos
      - Etiquetas de entidad de costo de Cloudyn
      - Categorías de etiquetas de suscripción para la asignación de costos

   Algunos ejemplos pueden incluir:
   - Centro de costos
   - department
   - Application
   - Entorno
   - Código de costo

     Esta es una lista de los grupos integrados disponibles en los informes:

     - **Tipo de costo**
     - Seleccione un tipo de costo, varios de ellos o todos. Estos tipos incluyen:
       - Precio único
       - Soporte técnico
       - Costo por uso
     - **Cliente**
       - Seleccione un cliente determinado, varios, o todos.
     - **Nombre de cuenta**
       - El nombre de la cuenta o de la suscripción. En Azure, es el nombre de la suscripción de Azure.
     - **Número de cuenta**
       - Seleccione una cuenta, varias o todas ellas. En Azure, es el identificador único de la suscripción de Azure.
     - **Cuenta primaria**
       - Seleccione una cuenta primaria, varias o todas ellas.
     - **Servicio**
       - Seleccione un servicio, varios o todos ellos.
     - **Proveedor**
       - El proveedor de la nube donde están asociados los recursos y los gastos.
     - **Región**
       - Región en la que se hospeda el recurso.
     - **Zona de disponibilidad**
       - Ubicaciones aisladas de AWS dentro de una región.
     - **Tipo de recurso**
       - El tipo de recurso en uso.
     - **Subtipo**
       - Seleccione el subtipo.
     - **operación**
       - Seleccione la operación o **Mostrar todo**.
     - **Modelo de precios**
       - Todo por adelantado
       - Sin gastos iniciales
       - En parte por adelantado
       - A petición
       - Reserva
       - Zona
     - **Tipo de gasto**
       - Seleccione el tipo de gasto, negativo o positivo, o ambos.
     - **Inquilinato**
       - Si una máquina se ejecuta como equipo dedicado o no.
     - **Tipo de uso**
       - El tipo de uso puede ser un pago único o pagos periódicos.

5. **Filtros**

    Utilice filtros de selección única o múltiple para establecer rangos a los valores seleccionados. Para establecer un filtro, haga clic en **Add** (Agregar) y seleccione las categorías y los valores del filtro.

6. **Cost Model** (Modelo de costo)

    Utilice Cost Model para seleccionar un modelo de costo creado anteriormente con la asignación de costos 360. Es posible que tenga varios modelos de costos de Cloudyn, dependiendo de los requisitos de asignación de costos. Algunos de los equipos de organización pueden tener requisitos de asignación de costos que difieren de otros. Cada equipo puede tener su propio modelo de costo dedicado.

    Para obtener información sobre la creación de una definición de modelo de asignación de costos, consulte la sección [Uso de etiquetas personalizadas para asignar costos](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

7. **Amortization** (Amortización)

    Utilice Amortization en los informes de asignación de costos para ver los cargos por servicios no basados en el uso o los costos pagaderos una sola vez y repartir su costo a lo largo del tiempo de manera uniforme durante su vida útil. Ejemplos de cargos únicos podrían incluir:
    - Cargos anuales por soporte técnico
    - Cargos anuales por componentes de seguridad
    - Cargos de compra de instancias reservadas
    - Algunos elementos de Azure Marketplace.

   En Amortización, seleccione **Amortized cost** (Costo amortizado) o **Actual Cost** (Costo real).

8. **Resolución**

    Utilice Resolution (Resolución) para seleccionar la resolución de tiempo dentro del intervalo de fechas seleccionado. La resolución de tiempo determina cómo se muestran las unidades en el informe y pueden ser:
    - Diario
    - Semanal
    - Mensual
    - Trimestral
    - Anual

9. **Allocation rules** (Reglas de asignación)

    Utilice Allocation rules para aplicar o deshabilitar el recálculo de los costos de asignación de costos. Se puede habilitar o deshabilitar el nuevo cálculo de la asignación de costos para los datos de facturación. El nuevo cálculo se aplica a las categorías seleccionadas en el informe. Permite evaluar el impacto del nuevo cálculo de la asignación de costos en comparación con los datos de facturación sin procesar.

10. **Uncategorized** (Sin categoría)

    Use Uncategorized para incluir o excluir los costos sin categoría en el informe.

11. **Show/hide fields** (Mostrar u ocultar campos)

    La opción Show/hide fields no tiene ningún efecto en los informes.

12. **Display formats** (Formatos de presentación)

    Utilice Display formats para seleccionar varias vistas de gráfico o tabla.

    ![Símbolos de los formatos de presentación que se pueden seleccionar](./media/understanding-cost-reports/display-formats.png)

13. **Multi-color** (Multicolor)

    Utilice la opción Multi-color para establecer el color de los gráficos en el informe.

14. **Acciones**

    Utilice Actions para guardar, exportar o programar el informe.

15. **Directiva**

    Aunque no aparecen en la imagen, algunos informes incluyen una directiva de cálculo de costos proyectados. En esos informes, la directiva **Consolidada** muestra recomendaciones para todas las cuentas y suscripciones de la entidad actual, como la inscripción a Microsoft o el pagador de AWS. La directiva **Independiente** muestra recomendaciones para una cuenta o suscripción como si no hubiera otra. La directiva que seleccione varía en función de la estrategia de optimización utilizada por la organización. Las proyecciones de costos se basan en los últimos 30 días de uso.

## <a name="save-and-schedule-reports"></a>Guardar y programar informes

Después de crear un informe, puede guardarlo para usarlo más adelante. Los informes guardados están disponibles en **Mis herramientas** > **Mis informes**. Si realiza cambios en un informe existente y lo guarda, el informe se guarda como una nueva versión. También puede guardarlo como un nuevo informe.

### <a name="save-a-report-to-the-cloudyn-portal"></a>Guardar un informe en el portal de Cloudyn

Mientras ve cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Save to my reports** (Guardar en Mis informes). Asigne un nombre al informe y, a continuación, agregue su propia dirección URL o use la dirección URL creada automáticamente. Opcionalmente puede **compartir** el informe públicamente con otras personas de la organización o con su entidad. Aunque no comparta el informe, sigue siendo un informe personal que solo puede ver. Guarde el informe.


### <a name="save-a-report-to-cloud-provider-storage"></a>Guardar un informe en el almacenamiento del proveedor de nube

Para guardar un informe en su proveedor de servicios en la nube, ya debe haber configurado una cuenta de almacenamiento. Mientras ve cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Schedule report** (Programar informe). Asigne un nombre al informe y, a continuación, agregue su propia dirección URL o use la dirección URL creada automáticamente. Seleccione **Save to storage** (Guardar en el almacenamiento) y, a continuación, seleccione la cuenta de almacenamiento o agregue una nueva. Escriba un prefijo que se anexe al nombre de archivo del informe. Seleccione el formato de archivo CSV o JSON y, a continuación, guarde el informe.

### <a name="schedule-a-report"></a>Programar un informe

Puede ejecutar informes a intervalos programados y puede enviarlos a una lista de destinatarios o a una cuenta de almacenamiento del proveedor de servicios en la nube. Mientras ve cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Schedule report** (Programar informe). Puede enviar el informe por correo electrónico y guardarlo en una cuenta de almacenamiento. En **Schedule** (Programar), seleccione el intervalo (diario, semanal, mensual). Para intervalos semanales o mensuales, seleccione el día o las fechas de entrega y seleccione la hora. Guarde el informe programado. Si selecciona el formato de informe de Excel, el informe se envía como datos adjuntos. Cuando se selecciona el formato de contenido de correo electrónico, los resultados del informe que se muestran en formato de gráfico se entregan como un gráfico.

### <a name="export-a-report-as-a-csv-file"></a>Exportar un informe como un archivo CSV

Mientras ve cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Export all report data** (Exportar todos los datos del informe). Aparece una ventana emergente y se descarga un archivo CSV.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda sobre los informes que se incluyen en Cloudyn en [Uso de los informes de Cloudyn](../../cost-management/use-reports.md).
- Obtenga información sobre cómo utilizar los informes para crear [paneles](../../cost-management/dashboards.md).
