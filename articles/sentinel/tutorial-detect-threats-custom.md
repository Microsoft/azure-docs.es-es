---
title: Creación de reglas de análisis personalizadas para detectar amenazas con Azure Sentinel | Microsoft Docs
description: Use este tutorial para aprender a crear reglas de análisis personalizadas para detectar amenazas de seguridad con Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2020
ms.author: yelevin
ms.openlocfilehash: 0e5989490603e22745a8bc972b16ed016c894893
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605901"
---
# <a name="tutorial-create-custom-analytics-rules-to-detect-threats"></a>Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas

Cuando haya [conectado los orígenes de datos](quickstart-onboard.md) con Azure Sentinel, puede crear reglas personalizadas que busquen criterios específicos en el entorno y generen incidentes si se detectan estos criterios para que pueda investigarlos. Este tutorial le ayuda a crear reglas personalizadas para detectar amenazas con Azure Sentinel.

Este tutorial ayuda a detectar amenazas con Azure Sentinel.
> [!div class="checklist"]
> * Creación de reglas de análisis
> * Automatizar las respuestas frente a amenazas

## <a name="create-custom-analytics-rules"></a>Creación de reglas de análisis personalizados

Puede crear reglas de análisis personalizadas que le ayuden a buscar los tipos de amenazas y anomalías que sean sospechosas en el entorno. La regla garantiza que se le notificará de inmediato, de modo que pueda evaluar, investigar y corregir las amenazas.

1. En Azure Portal en Azure Sentinel, seleccione **Analytics**.

1. En la barra de menús superior, seleccione **+Crear** y seleccione**Regla de consulta programada**. Así se abre el **Asistente para reglas de Analytics**.

    :::image type="content" source="media/tutorial-detect-threats-custom/create-scheduled-query.png" alt-text="Crear una consulta programada":::

1. En la pestaña **General**, especifique unos valores únicos en los campos **Nombre** y **Descripción**. En el campo **Tactics** (Táctica), puede elegir cualquiera de las categorías de ataques por las que se clasifica la regla. Establezca la **gravedad** según sea necesario. Cuando cree la regla, el valor predeterminado del campo **Status** (Estado) es **Enabled** (Habilitado), lo que significa que se ejecutará inmediatamente después de que termine de crearla. Si no desea ejecutarla de inmediato, seleccione **Disabled** (Deshabilitado) para agregar la regla a la pestaña **Active rules** (Reglas activas), desde donde podrá habilitarla cuando sea necesario.

    ![Inicio de la creación de una regla de análisis personalizada](media/tutorial-detect-threats-custom/general-tab.png)

1. En la pestaña **Set rule logic** (Establecer lógica de regla), puede escribir una consulta directamente en el campo **Rule query** (Consulta de búsqueda), o bien crearla en Log Analytics y, después, copiarla y pegarla ahí.
 
   ![Crear una consulta en Azure Sentinel](media/tutorial-detect-threats-custom/settings-tab.png)

   - Vea el área **Results preview** (Vista previa de resultados) de la derecha, donde Azure Sentinel muestra el número de resultados (eventos de registro) que va a generar la consulta, lo que cambia sobre la marcha mientras se escribe y configura la consulta. El gráfico muestra el número de resultados en un periodo definido, lo que determina la configuración de la sección **Query scheduling** (Programación de consultas).
    - Si ve que su consulta desencadena demasiadas alertas o alertas demasiado frecuentes, puede establecer una base de referencia en la sección **Alert threshold** (Umbral de alerta).

      Aquí tiene una consulta de ejemplo que le alertará cuando se cree una cantidad anómala de recursos en Azure Activity.

      `AzureActivity
     \| where OperationName == "Create or Update Virtual Machine" or OperationName =="Create Deployment"
     \| where ActivityStatus == "Succeeded"
     \| make-series dcount(ResourceId)  default=0 on EventSubmissionTimestamp in range(ago(7d), now(), 1d) by Caller`

      > [!NOTE]
      > La longitud de la consulta debe estar entre 1 y 10 000 caracteres y no puede contener las palabras "buscar \*" ni "unión \*".

    1. Use la sección **Map entities** (Asignar entidades) para vincular parámetros de los resultados de las consultas a entidades reconocidas de Azure Sentinel. Estas entidades forman la base del posterior análisis, lo que incluye la agrupación de alertas en incidentes en la pestaña **Incident settings** (Configuración de incidentes).
  
    1. En la sección **Query scheduling** (Programación de consultas), establezca los siguientes parámetros:

       1. Establezca la opción **Run query every** (Ejecutar consulta cada) para controlar la frecuencia de ejecución de la consulta (puede establecer frecuencias de 5 minutos o de una vez al día).

       1. Set **Lookup data from the last** (Datos de la búsqueda a partir del último) para determinar el periodo de los datos que cubre la consulta (por ejemplo, puede consultar los 10 últimos minutos de datos o las 6 últimas horas de datos).

       > [!NOTE]
       > Estos dos valores son independientes entre sí, hasta cierto punto. Puede ejecutar una consulta a un intervalo corto que abarque un periodo mayor que el intervalo (teniendo en efecto consultas que se solapan), pero no puede ejecutar una consulta a un intervalo que supere el periodo de cobertura, ya que, de lo contrario tendrá lagunas en la cobertura general de la consulta.

    1. Use la sección **Alert threshold** (Umbral de alerta) para definir una base de referencia. Por ejemplo, en **Generar alerta cuando el número de resultados de consulta**, seleccione **Es mayor que** y escriba el número 1000 si desea que la regla genere una alerta solo si la consulta genera más de 1000 resultados cada vez que se ejecuta. Este es un campo obligatorio, por lo que si no desea establecer una base de referencia (es decir, si desea que su alerta registre todos los eventos) escriba 0 en el campo numérico.
    
    1. En **Agrupación de eventos**, elija una de las dos formas de controlar la agrupación de **eventos** en **alertas**: 

       - **Agrupar todos los eventos en una misma alerta** (la configuración predeterminada). La regla genera una única alerta cada vez que se ejecuta, siempre y cuando la consulta devuelva más resultados de los especificados en el **umbral de alerta** anterior. La alerta incluye un resumen de todos los eventos que se devuelven en los resultados. 

       - **Desencadenar una alerta para cada evento**. La regla genera una alerta única para cada evento devuelto por la consulta. Esto resulta útil si desea que los eventos se muestren individualmente, o si desea agruparlos por determinados parámetros por usuario, nombre de host o cualquier otro elemento. Puede definir estos parámetros en la consulta.
    
       En la actualidad, el número de alertas que puede generar una regla se limita a veinte. Si en una regla determinada, **Agrupación de eventos** se establece en **Desencadenar una alerta para cada evento** y la consulta de la regla devuelve más de 20 eventos, cada uno de los primeros 19 eventos generará una alerta única y la vigésima alerta resumirá todo el conjunto de eventos devueltos. En otras palabras, la alerta vigésima es lo que se habría generado en la opción **Agrupar todos los eventos en una misma alerta**.

       > [!NOTE]
       > ¿Cuál es la diferencia entre **eventos** y **alertas**?
       >
       > - Un **eventos** es una descripción de una única aparición. Por ejemplo, una sola entrada de un archivo de registro podría contar como un evento. En este contexto, un evento hace referencia a un único resultado devuelto por una consulta en una regla de análisis.
       >
       > - Una **alerta** es una colección de eventos que, en conjunto, son importantes desde el punto de vista de la seguridad. Una alerta podría contener un solo evento si el evento tuviera implicaciones de seguridad importantes (por ejemplo, un inicio de sesión administrativo de un país extranjero fuera de horas de oficina).
       >
       > - Por cierto, ¿qué son los **incidentes**? La lógica interna de Azure Sentinel crea **incidentes** a partir de **alertas** o de grupos de alertas. La cola de incidentes es el punto focal del trabajo: evaluación de prioridades, investigación y corrección del analista.
       > 
       > Azure Sentinel ingiere eventos sin procesar de algunos orígenes de datos y alertas ya procesadas de otros usuarios. Es importante tener en cuenta con cuál es el que se está tratando en cualquier momento.

       > [!IMPORTANT]
       > La agrupación de eventos se encuentra actualmente en versión preliminar pública. Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
    
    1. In the **Suppression** (Supresión), en el valor **Stop running query after alert is generated** (Detener la ejecución de la consulta después de que se genere una alerta), seleccione **On** (Activar) si, una vez que recibe la alerta, desea suspender la operación de esta reglar durante un periodo que supere el intervalo de consulta. Si lo activa, en **Stop running query for** (Detener la ejecución de la consulta durante), seleccione el periodo durante el que debe detenerse la consulta, con un máximo de 24 horas.

1. En la pestaña **Incident Settings** (Configuración de incidentes), puede elegir si Azure Sentinel convierte las alertas en incidentes sobre los que se pueden realizar acciones y cómo lo hace. Si esta pestaña no se modifica, Azure Sentinel creará un solo incidente independiente a partir de todas y cada una de las alertas. Puede elegir que no se creen incidentes o agrupar varias alertas en un solo incidente. Para ello, solo debe cambiar el valor de esta pestaña.

   > [!IMPORTANT]
   > La pestaña Configuración de los incidentes se encuentra actualmente en versión preliminar pública. Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
    
    1. En la sección **Incident Settings** (Configuración de incidentes), el valor predeterminado de **Create incidents from alerts triggered by this analytics rule** (Crear incidentes a partir de alertas desencadenadas por esta regla de análisis) es **Enabled** (Habilitado), lo que significa que Azure Sentinel creará un solo incidente independiente por cada una de las alertas que desencadene la regla.
       - Si no desea que esta regla provoque la aparición de incidentes (por ejemplo, si esta regla es solo para recopilar información para su posterior análisis), seleccione **Disabled** (Deshabilitado).

    1. En la sección **Alert grouping** (Agrupación de alertas), si desea que se genere un solo incidente a partir de un grupo de hasta 150 alertas similares o recurrentes (consulte la nota), en **Group related alerts, triggered by this analytics rule, into incidents** (Agrupar en incidentes alertas relacionadas desencadenadas por esta regla de análisis) seleccione **Enabled** (Habilitado) y establezca los siguientes parámetros.

    - **Limite el grupo a las alertas creadas en el período de tiempo seleccionado**: Determine el período de tiempo en el que se agruparán las alertas similares o periódicas. Todas las alertas correspondientes que se encuentren dentro de este periodo de tiempo generarán colectivamente un incidente o un conjunto de incidentes (en función de la configuración de agrupación que encontrará a continuación). Las alertas que aparezcan fuera de este periodo de tiempo generarán un incidente, o conjunto de incidentes, independientes.

    - **Agrupe las alertas desencadenadas por esta regla de análisis en un único incidente por**: elija el motivo por el que se agruparán las alertas:

        - **Agrupar las alertas en un solo incidente si todas las entidades coinciden**: Las alertas se agrupan si comparten los mismos valores entre todas las entidades asignadas (definidas en la pestaña Set rule logic [Establecer lógica de regla] anterior). Esta es la configuración recomendada.

        - **Agrupar todas las alertas desencadenadas por esta regla en un único incidente**: Todas las alertas que genera esta regla se agrupan aunque no compartan valores idénticos.

        - **Agrupar las alertas en un solo incidente si las entidades seleccionadas coinciden**: Las alertas se agrupan si comparten valores idénticos en algunas de las entidades asignadas (que puede seleccionar en la lista desplegable). Es posible que desee usar esta opción si, por ejemplo, desea crear incidentes independientes basados en las direcciones IP de origen o de destino.

    - **Vuelva a abrir los incidentes coincidentes cerrados**: si se ha resuelto y cerrado un incidente y, posteriormente, se genera otra alerta que habría pertenecido a ese incidente, establezca esta opción en **Enabled** (Habilitado) si quiere que el incidente cerrado se vuelva a abrir, o bien déjela como **Disabled** (Deshabilitado) si quiere que la alerta cree un incidente nuevo.
    
        > [!NOTE]
        > Se pueden agrupar hasta 150 alertas en un solo incidente. Si hay más de 150 alertas generadas por una regla que las agrupa en un solo incidente, se generará un nuevo incidente con los mismos detalles del incidente que el original, y las alertas sobrantes se agruparán en el nuevo incidente.

1. En la pestaña **Automated responses** (Respuestas automatizadas), seleccione todos los cuadernos de estrategias que desee ejecutar automáticamente cuando la regla personalizada genere una alerta. Para más información sobre la creación y automatización de los cuadernos de estrategias, consulte [Respuesta a amenazas](tutorial-respond-threats-playbook.md).

1. Seleccione **Review and create** (Revisar y crear) para revisar todos los valores de la nueva regla de alertas y, después, seleccione **Create to initialize your alert rule** (Crear para inicializar la regla de alertas).
  
1. Una vez creada la alerta, se agrega una regla personalizada a la tabla en **Active rules** (Reglas activas). Desde esta lista puede habilitar, deshabilitar o eliminar cada regla.

1. Para ver los resultados de las reglas de alertas, vaya a la página de **incidentes**, donde puede evaluar las prioridades,  [investigar los incidentes](tutorial-investigate-cases.md) y solucionar las amenazas.


> [!NOTE]
> Las alertas generadas en Azure Sentinel están disponibles mediante  [Microsoft Graph Security](https://aka.ms/securitygraphdocs). Para más información, consulte la [documentación sobre alertas de Microsoft Graph Security](https://aka.ms/graphsecurityreferencebetadocs).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a detectar amenazas casos mediante Azure Sentinel.

Para aprender a automatizar las respuestas a las amenazas, consulte  [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md).

