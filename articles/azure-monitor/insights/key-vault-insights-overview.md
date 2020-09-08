---
title: Supervisión de Key Vault con Azure Monitor para Key Vault (versión preliminar) | Microsoft Docs
description: En este artículo se describe Azure Monitor para Key Vault.
services: azure-monitor
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 04/13/2019
ms.openlocfilehash: c669946ab76fcaeaaa6fd681f521408643c5a63c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88531266"
---
# <a name="monitoring-your-key-vault-service-with-azure-monitor-for-key-vault-preview"></a>Supervisión del servicio de almacén de claves con Azure Monitor para Key Vault (versión preliminar)
Azure Monitor para Key Vault (versión preliminar) proporciona una supervisión completa de los almacenes de claves proporcionando una vista unificada del rendimiento, los errores, la latencia y las solicitudes de Key Vault.
Este artículo le ayudará a comprender cómo incorporar y personalizar la experiencia de Azure Monitor para Key Vault (versión preliminar).

## <a name="introduction-to-azure-monitor-for-key-vault-preview"></a>Introducción a Azure Monitor para Key Vault (versión preliminar)

Antes de saltar a la experiencia, debe entender cómo se presenta y se visualiza la información.
-    **Perspectiva a gran escala**, mostrando una vista de instantánea del rendimiento en función de las solicitudes, el desglose de errores y una visión general de las operaciones y la latencia.
-   **Análisis en profundidad** de un almacén de claves determinado para realizar un análisis detallado.
-    **Personalizable**, que le permite cambiar las métricas que desea ver y modificar, o establecer umbrales en consonancia con sus límites y guardar su propio libro. Los gráficos del libro se pueden anclar a los paneles de Azure.

Azure Monitor para Key Vault combina los registros y las métricas para proporcionar una solución de supervisión global. Todos los usuarios pueden acceder a los datos de supervisión basados en métricas; sin embargo, la inclusión de visualizaciones basadas en registros puede requerir que los usuarios [habiliten el registro de su instancia de Azure Key Vault](../../key-vault/general/logging.md).

## <a name="configuring-your-key-vaults-for-monitoring"></a>Configuración de los almacenes de claves para la supervisión

> [!NOTE]
> La habilitación de registros es un servicio de pago que proporciona capacidades de supervisión adicionales.

1. La pestaña Operations & Latency (Operaciones y latencia) le ayuda a determinar cuántos y qué almacenes de claves están habilitados. Para iniciar la recopilación, seleccione el botón **Habilitar**, que le llevará a un libro independiente en el que se enumeran los almacenes de claves que requieren la habilitación de los registros de diagnóstico.

    ![Captura de pantalla de la pestaña Operaciones y latencia donde se muestra el botón Habilitar azul](./media/key-vaults-insights-overview/enable-logging.png)

2. Para habilitar los registros de diagnóstico, haga clic en el vínculo **Habilitar** debajo de la columna de acciones y cree una nueva configuración de diagnóstico que envíe los registros a un área de trabajo de Log Analytics. Se recomienda enviar todos los registros a la misma área de trabajo.

3. Una vez guardada la configuración de diagnóstico, podrá ver todos los gráficos y visualizaciones basados en registros debajo de las conclusiones de Key Vault. Tenga en cuenta que se puede tardar entre varios minutos y horas en empezar a rellenar los registros.

4. Para obtener ayuda adicional sobre cómo habilitar los registros de diagnóstico para el servicio Key Vault, lea la [guía completa](../../key-vault/general/logging.md).

## <a name="view-from-azure-monitor"></a>Vista desde Azure Monitor

En Azure Monitor, puede ver los detalles de solicitud, latencia y errores de varios almacenes de claves de su suscripción, así como ayudar a identificar problemas de rendimiento y escenarios de limitación.

Para ver el uso y las operaciones de los almacenes de claves en todas las suscripciones, realice los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione **Monitor** en el panel izquierdo y en Azure Portal, bajo la sección Conclusiones, seleccione **Almacenes de claves (versión preliminar)** .

![Captura de pantalla de la experiencia de información general con varios gráficos](./media/key-vaults-insights-overview/overview.png)

## <a name="overview-workbook"></a>Libro de información general

En el libro Información general de la suscripción seleccionada, la tabla muestra las métricas de almacén de claves interactivas para almacenes de claves agrupados dentro de la suscripción. Puede filtrar los resultados en función de las opciones que seleccione en las siguientes listas desplegables:

* Suscripciones: solo se muestran las suscripciones que tienen almacenes de clave.

* Almacenes de claves: de forma predeterminada, solo se preseleccionan cinco almacenes de claves como máximo. Si selecciona todos los almacenes de claves o varios de ellos en el selector de ámbito, se devolverán como máximo doscientos almacenes de claves. Por ejemplo, si tenía en total 573 almacenes de claves entre tres suscripciones que ha seleccionado, solo se mostrarán doscientos.

* Intervalo de tiempo: de forma predeterminada, muestra las últimas veinticuatro horas de información en función de las selecciones correspondientes realizadas.

El icono de contador, que se encuentra bajo la lista desplegable, acumula el número total de almacenes de claves de la suscripción seleccionada y refleja cuántos se seleccionan. Hay mapas térmicos de código de colores condicionales para las columnas del libro que notifican las métricas de solicitud, error y latencia. El color más profundo tiene el valor más alto y un color más claro indica los valores más bajos.

## <a name="failures-workbook"></a>Libro de errores

Seleccione **Errores** en la parte superior de la página; se abrirá la pestaña Errores. Muestra el número total de visitas de la API y la frecuencia en el tiempo, junto con la cantidad de ciertos códigos de respuesta.

![Captura de pantalla del libro de errores](./media/key-vaults-insights-overview/failures.png)

La codificación de color condicional o los mapas térmicos de las columnas del libro notifican las métricas del número total de visitas de la API con un valor azul. El color más profundo tiene el valor más alto y un color más claro indica los valores más bajos.

El libro muestra Correctos (códigos de estado 2xx), Authentication Errors (Errores de autenticación) (códigos de estado 401/403), Limitación (códigos de estado 429) y Other Failures (Otros errores) (códigos de estado 4xx).

Para comprender mejor lo que representan cada uno de los códigos de estado, se recomienda leer la documentación sobre los [códigos de estado y respuesta de Azure Key Vault](../../key-vault/general/authentication-requests-and-responses.md).

## <a name="operations--latency-workbook"></a>Libro de operaciones y latencia

Seleccione **Operations & Latency** (Operaciones y latencia) en la parte superior de la página; se abre la pestaña **Operaciones y latencia** (Operaciones y latencia). Esta pestaña le permite incorporar los almacenes de claves para la supervisión. Para obtener pasos más detallados, consulte la sección [Configuración de los almacenes de claves para la supervisión](#configuring-your-key-vaults-for-monitoring).

Puede ver cuántos almacenes de claves están habilitados para el registro. Si al menos un almacén se ha configurado correctamente, podrá ver las tablas que muestran las operaciones y los códigos de estado de cada uno de los almacenes de claves. Puede hacer clic en la sección de detalles de una fila para obtener información adicional sobre la operación individual.

![Captura de pantalla de los gráficos de operaciones y latencia](./media/key-vaults-insights-overview/logs.png)

Si no ve ningún dato en esta sección, consulte la sección superior sobre cómo habilitar los registros de Azure Key Vault o consulte la sección de solución de problemas a continuación.

## <a name="view-from-a-key-vault-resource"></a>Vista de un recurso de Key Vault

Para acceder a Azure Monitor para Key Vault directamente desde un almacén de claves:

1. En Azure Portal, seleccione Almacenes de claves.

2. En la lista, elija un almacén de claves. En la sección Supervisión, seleccione Conclusiones (versión preliminar).

También se puede acceder a estas vistas seleccionando el nombre del recurso de un almacén de claves en el libro de nivel de Azure Monitor.

![Captura de pantalla de vista de un recurso de almacén de claves](./media/key-vaults-insights-overview/key-vault-resource-view.png)

En el libro **Información general** del almacén de claves, se muestran varias métricas de rendimiento que le ayudan a evaluar rápidamente:

- Los gráficos de rendimiento interactivos que muestran los detalles más esenciales relacionados con la disponibilidad, las transacciones y la latencia del almacén de claves.

- Los iconos de métricas y estado que resaltan la disponibilidad del servicio, el recuento total de transacciones con el recurso del almacén de claves la latencia global.

Al seleccionar cualquiera de las demás pestañas para **Errores** u **Operaciones** se abren los libros correspondientes.

![Captura de pantalla de la vista de errores](./media/key-vaults-insights-overview/resource-failures.png)

El libro de errores desglosa los resultados de todas las solicitudes del almacén de claves en el período de tiempo seleccionado y proporciona categorización en Correctos (2xx), Authentication Errors (Errores de autenticación) (401/403), Limitación (429) y Other Failures (Otros errores).

![Captura de pantalla de la vista de operaciones](./media/key-vaults-insights-overview/operations.png)

El libro Operaciones permite a los usuarios profundizar en los detalles completos de todas las transacciones, que se pueden filtrar por el estado del resultado mediante los iconos de nivel superior.

![Captura de pantalla de la vista de operaciones](./media/key-vaults-insights-overview/info.png)

Los usuarios también pueden ver las vistas en función de tipos de transacciones específicos en la tabla superior, que actualiza dinámicamente la tabla inferior, donde los usuarios pueden ver los detalles de las operaciones completas en un panel de contexto emergente.

>[!NOTE]
> Tenga en cuenta que los usuarios deben tener habilitada la configuración de diagnóstico para ver este libro. Para obtener más información sobre cómo habilitar la configuración de diagnóstico, lea [Registro de Azure Key Vault](../../key-vault/general/logging.md).

## <a name="pin-and-export"></a>Anclar y exportar

Cualquiera de las secciones de métricas se puede anclar a un panel de Azure; para ello, seleccione el icono de chincheta que se encuentra en la parte superior derecha de la sección.

Los libros de información general o errores de varias suscripciones y almacenes de claves admiten la exportación de los resultados en formato Excel; para ello, seleccione el icono de descarga situado a la izquierda del icono de chincheta.

![Captura de pantalla del icono de chincheta seleccionado](./media/key-vaults-insights-overview/pin.png)

## <a name="customize-azure-monitor-for-key-vault"></a>Personalización de Azure Monitor para Key Vault

En esta sección se resaltan los escenarios comunes para editar el libro y personalizarlo de acuerdo con sus necesidades de análisis de datos:
*  Definir el ámbito del libro para seleccionar siempre una suscripción o unos almacenes de claves determinados
* Cambiar las métricas de la cuadrícula
* Cambiar el umbral de solicitudes
* Cambiar la representación de color

Puede iniciar las personalizaciones habilitando el modo de edición; para ello, seleccione el botón **Personalizar** de la barra de herramientas superior.

![Captura de pantalla del botón Personalizar](./media/key-vaults-insights-overview/customize.png)

Las personalizaciones se guardan en un libro personalizado para evitar sobrescribir la configuración predeterminada del libro publicado. Los libros se guardan en un recurso compartido, bien en la sección Mis informes, que es privada para usted, o en la sección Informes compartidos, a la que puede acceder cualquiera con acceso al grupo de recursos. Después de guardar el libro personalizado, debe ir a la galería de libros para iniciarlo.

![Captura de pantalla de la galería de libros](./media/key-vaults-insights-overview/gallery.png)

### <a name="specifying-a-subscription-or-key-vault"></a>Especificación de una suscripción o un almacén de claves

Puede configurar los libros Información general o Errores de varias suscripciones y almacenes de claves para definir el ámbito a una o varias suscripciones o almacenes de claves en concreto en cada ejecución. Para ello, realice los siguientes pasos:

1. Seleccione **Monitor** en el portal y, luego, **Almacenes de claves (versión preliminar)** en el panel izquierdo.
2. En el libro **Overview** (Información general), elija **Edit** (Editar) en la barra de comandos.
3. En la lista desplegable **Suscripciones**, seleccione una o varias suscripciones que quiera que sean las predeterminadas. Recuerde que el libro admite la selección de hasta 10 suscripciones.
4. En la lista desplegable **Almacenes de claves**, seleccione una o varias cuentas que quiera usar como las predeterminadas. Recuerde que el libro admite la selección de hasta 200 cuentas de almacenamiento.
5. Seleccione **Save as** (Guardar como) en la barra de comandos para guardar una copia del libro con las personalizaciones y, luego, haga clic en **Done editing** (Edición finalizada) para regresar el modo de lectura.

## <a name="troubleshooting"></a>Solución de problemas

Para obtener instrucciones generales para la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.

Esta sección le ayudará con el diagnóstico y la solución de algunos de los problemas habituales que pueden producirse en la aplicación al usar Azure Monitor para Almacén de claves (versión preliminar). Utilice la siguiente lista para buscar la información relacionada con el problema específico.

### <a name="resolving-performance-issues-or-failures"></a>Resolución de problemas de rendimiento o errores

Para ayudar a solucionar los problemas relacionados con el almacén de claves que identifique con Azure Monitor para Key Vault (versión preliminar), consulte la [documentación de Azure Key Vault](../../key-vault/index.yml).

### <a name="why-can-i-only-see-200-key-vaults"></a>¿Por qué solo puedo ver doscientos almacenes de claves?

Hay un límite de doscientos almacenes de claves que se pueden seleccionar y ver. Independientemente del número de suscripciones seleccionadas, el número de almacenes de claves seleccionados tiene un límite de doscientos.

### <a name="why-dont-i-see-all-my-subscriptions-in-the-subscription-picker"></a>¿Por qué veo todas mis suscripciones en el selector de suscripciones?

Solo se muestran las suscripciones que contienen almacenes de claves, que se eligen desde el filtro de suscripción seleccionado, que se seleccionan en la sección "Directory + Subscription" (Directorio + Suscripción) en el encabezado de Azure Portal.

![Captura de pantalla del filtro de suscripción](./media/key-vaults-insights-overview/Subscriptions.png)

### <a name="i-am-getting-an-error-message-that-the-query-exceeds-the-maximum-number-of-workspacesregions-allowed-what-to-do-now"></a>Recibo un mensaje de error que indica que la "consulta supera el número máximo de áreas de trabajo o regiones permitidas", ¿qué hago ahora?

Actualmente, hay un límite de veinticinco regiones y doscientas áreas de trabajo. Para ver los datos, debe reducir el número de suscripciones o grupos de recursos.

### <a name="i-want-to-make-changes-or-add-additional-visualizations-to-key-vault-insights-how-do-i-do-so"></a>Quiero realizar cambios o agregar visualizaciones adicionales a las conclusiones de Key Vault, ¿cómo lo hago?

Para realizar cambios, seleccione "Modo de edición" para modificar el libro y después puede guardar el trabajo como un libro nuevo que esté asociado a una suscripción y un grupo de recursos designados.

### <a name="what-is-the-time-grain-once-we-pin-any-part-of-the-workbooks"></a>¿Cuál es el intervalo de agregación una vez que se ancla cualquier parte de los libros?

Se usa el intervalo de agregación "automático", por lo que depende del intervalo de agregación seleccionado.

### <a name="what-is-the-time-range-when-any-part-of-the-workbook-is-pinned"></a>¿Cuál es el intervalo de agregación cuando se ancla cualquier parte del libro?

El intervalo de agregación dependerá de la configuración del panel.

### <a name="why-do-i-not-see-any-data-for-my-key-vault-under-the-operations--latency-sections"></a>¿Por qué no veo ningún dato para mi Key Vault en las secciones Operaciones y latencia?

Para ver los datos basados en registros, deberá habilitar los registros para cada uno de los almacenes de claves que desee supervisar. Esto puede realizarse en la configuración de diagnóstico de cada almacén de claves. Tendrá que enviar los datos a un área de trabajo de Log Analytics designada.

### <a name="i-have-already-enabled-logs-for-my-key-vault-why-am-i-still-unable-to-see-my-data-under-operations--latency"></a>Ya he habilitado los registros para mi Key Vault. ¿Por qué sigo sin poder ver mis datos en Operaciones y latencia?

Actualmente, los registros de diagnóstico no funcionan de forma retroactiva, por lo que los datos solo comenzarán a aparecer una vez que se hayan realizado acciones en los almacenes de claves. Por lo tanto, pueden tardar algún tiempo, desde horas hasta un día, en función de la actividad que presente el almacén de claves.

Además, si tiene un gran número de almacenes de claves y suscripciones seleccionados, es posible que no pueda ver los datos debido a las limitaciones de la consulta. Para ver los datos, es posible que tenga que reducir el número de suscripciones o almacenes de claves seleccionados. 

### <a name="what-if-i-want-to-see-other-data-or-make-my-own-visualizations-how-can-i-make-changes-to-the-key-vault-insights"></a>¿Qué ocurre si deseo ver otros datos o crear mis propias visualizaciones? ¿Cómo puedo realizar cambios en las conclusiones de Key Vault?

Puede editar el libro existente mediante el uso del modo de edición y luego guardar el trabajo como un libro nuevo que tendrá todos los cambios nuevos.

## <a name="next-steps"></a>Pasos siguientes

Conozca los escenarios para los que están concebidos los libros, cómo crear informes y personalizar los ya existentes y otros muchos temas en el artículo [Crear informes interactivos con libros de Azure Monitor](../platform/workbooks-overview.md).
