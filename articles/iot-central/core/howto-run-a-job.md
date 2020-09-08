---
title: Crear y ejecutar trabajos en la aplicación de Azure IoT Central | Microsoft Docs
description: Los trabajos de Azure IoT Central permiten funcionalidades de administración de dispositivos de forma masiva, como actualizar propiedades o ejecutar un comando.
ms.service: iot-central
services: iot-central
author: sarahhubbard
ms.author: sahubbar
ms.date: 06/08/2020
ms.topic: how-to
ms.openlocfilehash: 406881a9131aae35b91dcab248745bb426cecf0e
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797843"
---
# <a name="create-and-run-a-job-in-your-azure-iot-central-application"></a>Crear y ejecutar trabajos en la aplicación de Azure IoT Central

Puede usar Microsoft Azure IoT Central para administrar los dispositivos conectados a escala mediante trabajos. Los trabajos le permiten realizar actualizaciones masivas de las propiedades de los dispositivos y ejecutar comandos. Este artículo le mostrará cómo empezar a usar trabajos en su propia aplicación.

## <a name="create-and-run-a-job"></a>Crear y ejecutar un trabajo

En esta sección se muestra cómo crear y ejecutar un trabajo con la configuración del umbral ligero de un grupo de dispositivos de puerta de enlace logística.

1. En el panel izquierdo, seleccione **Trabajos**.

2. Seleccione **+ Nuevo**.

   ![Captura de pantalla que muestra las selecciones para crear un trabajo.](./media/howto-run-a-job/create-new-job.png)

3. Escriba un nombre y una descripción para identificar el trabajo que va a crear.

4. Seleccione el grupo de dispositivos de destino al que desee que se aplique el trabajo. Puede ver el número de dispositivos a los que se aplicará la configuración del trabajo en la sección **Resumen**.

5. Elija **Propiedad de la nube**, **Propiedad** o **Comando** como el tipo de trabajo que se va a configurar. 

   Para definir una configuración de trabajo **Propiedad**, seleccione una propiedad y establezca su nuevo valor. Para establecer la configuración de un trabajo **Comando**, elija el comando que desea ejecutar. Un trabajo de propiedad puede establecer varias propiedades.

   ![Captura de pantalla que muestra las selecciones para crear un trabajo de propiedad denominado Set Light Threshold.](./media/howto-run-a-job/configure-job.png)

6. Seleccione **Ejecutar** o **Guardar**. El trabajo aparecerá ahora en la página principal **Trabajos**. En esta página, podrá ver su trabajo en ejecución actualmente y el historial de los trabajos ejecutados o guardados anteriormente. Puede volver a abrir el trabajo guardado en cualquier momento para continuar con la edición o ejecutarlo.

   ![Captura de pantalla que muestra el nombre, el estado y la descripción de un trabajo creado.](./media/howto-run-a-job/view-job.png)

   > [!NOTE]
   > Puede ver hasta 30 días del historial de trabajos ejecutados anteriormente.

7. Seleccione el trabajo guardado y ejecútelo con el botón **Ejecutar**. 

   Se abre el cuadro de diálogo **Run your job?** (¿Desea ejecutar su trabajo?). Para confirmar, seleccione el botón **Ejecutar trabajo**. 

   ![Captura de pantalla del cuadro de diálogo que confirma que desea ejecutar un trabajo.](./media/howto-run-a-job/run-job.png)

8. Un trabajo pasa por estas fases: pendiente, en ejecución y completado. Los detalles de ejecución del trabajo contienen métricas de resultados, detalles de duración y una cuadrícula de lista de dispositivos. 

   En esta descripción general también puede seleccionar **Registro de resultados** para descargar un archivo CSV de los detalles del trabajo, incluidos los dispositivos y sus valores de estado. Esta información puede ser útil para solucionar problemas.

   ![Captura de pantalla que muestra el estado del dispositivo.](./media/howto-run-a-job/download-details.png)

## <a name="manage-jobs"></a>Trabajos de administración

Para detener un trabajo en ejecución, ábralo y seleccione **Detener**. El estado del trabajo cambia para reflejar que el trabajo se ha detenido. En la sección **Resumen**, se muestran los dispositivos que se han completado, con errores o aún pendientes.

![Captura de pantalla que muestra un trabajo en ejecución y el botón para detener un trabajo.](./media/howto-run-a-job/manage-job.png)

Una vez que un trabajo se encuentra detenido, puede seleccionar **Continuar** para reanudar la ejecución del trabajo. El estado del trabajo cambia para reflejar que el trabajo está de nuevo en ejecución. La sección **Resumen** continúa actualizándose con el progreso más reciente.

![Captura de pantalla que muestra un trabajo detenido y el botón para continuar un trabajo.](./media/howto-run-a-job/stopped-job.png)

## <a name="copy-a-job"></a>Copia de un trabajo

Para copiar uno de los trabajos existentes, selecciónelo en la página **Trabajos** y seleccione **Detalles del trabajo**. Se abre la página **Detalles del trabajo**. 

![Captura de pantalla que muestra la página con los detalles del trabajo.](./media/howto-run-a-job/job-details.png)

Seleccione **Copiar**.

![Captura de pantalla que muestra el botón Copiar.](./media/howto-run-a-job/job-details-copy.png)

Se abrirá una copia de la configuración del trabajo para editarla y se anexará **Copia** al nombre del trabajo. Puede guardar o ejecutar el trabajo de nuevo.

![Captura de pantalla que muestra una copia de la configuración del trabajo.](./media/howto-run-a-job/copy-job.png)

## <a name="view-job-status"></a>Ver estado del trabajo

Una vez creado un trabajo, la columna **Estado** se actualiza con el mensaje de estado más reciente del trabajo. En la tabla siguiente se muestran los valores de estado posibles del trabajo:

| Mensaje de estado       | Significado de estado                                          |
| -------------------- | ------------------------------------------------------- |
| Completed            | Este trabajo se ejecutó en todos los dispositivos.              |
| Con error               | Se produjo un error en este trabajo y no se ejecutó por completo en los dispositivos.  |
| Pending              | Este trabajo aún no ha comenzado a ejecutarse en los dispositivos.         |
| En ejecución              | Este trabajo se está ejecutando actualmente en los dispositivos.             |
| Detenido              | Un usuario ha detenido manualmente este trabajo.           |

El mensaje de estado va seguido de una descripción general de los dispositivos en el trabajo. En la tabla siguiente se muestran los valores de estado del dispositivo posibles:

| Mensaje de estado       | Significado de estado                                                     |
| -------------------- | ------------------------------------------------------------------ |
| Correcto            | El número de dispositivos en los que se ejecutó correctamente el trabajo.       |
| Con error               | El número de dispositivos en los que no se ejecutó correctamente el trabajo.       |

Para ver el estado del trabajo y todos los dispositivos afectados, abra el trabajo. Verá uno de los mensajes de estado siguientes junto a cada nombre de dispositivo:

| Mensaje de estado       | Significado de estado                                                                |
| -------------------- | ----------------------------------------------------------------------------- |
| Completed            | El trabajo se ejecutó en este dispositivo.                                     |
| Con error               | No se pudo ejecutar el trabajo en este dispositivo. El mensaje de error muestra más información.  |
| Pending              | Aún no se ejecutó el trabajo en este dispositivo.                                   |

Seleccione **Descargar** para descargar un archivo CSV con los detalles del trabajo y la lista de dispositivos y sus valores de estado.

## <a name="filter-the-device-list"></a>Filtro de la lista de dispositivos

Puede filtrar la lista de dispositivos en la página **Detalles del trabajo** seleccionando el icono de filtro. Puede filtrar por los campos **Id. de dispositivo** o **Estado**.

:::image type="content" source="media/howto-run-a-job/filter.png" alt-text="Captura de pantalla que muestra las selecciones para filtrar una lista de dispositivos":::.

## <a name="customize-columns-in-the-device-list"></a>Personalización de las columnas en la lista de dispositivos

Puede elegir columnas adicionales para mostrarlas en la lista de dispositivos; para ello, seleccione el icono de opciones de columna.

:::image type="content" source="media/howto-run-a-job/column-options.png" alt-text="Captura de pantalla que muestra el icono de las opciones de columna":::.

Un cuadro de diálogo le permite elegir las columnas que se van a mostrar en la lista de dispositivos. Seleccione las columnas que desea mostrar, seleccione la flecha derecha y después **Aceptar**. Para seleccionar todas las columnas disponibles, elija **Seleccionar todos**.

:::image type="content" source="media/howto-run-a-job/column-picker-popup.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo para elegir las columnas que se van a mostrar":::.

Las columnas seleccionadas aparecen en la lista de dispositivos.

:::image type="content" source="media/howto-run-a-job/column-picker-column-selected.png" alt-text="Captura de pantalla que muestra las columnas seleccionadas en la lista de dispositivos":::.

Las columnas seleccionadas se conservan en una sesión de usuario o en las sesiones de usuario que tienen acceso a la aplicación.

## <a name="rerun-jobs"></a>Repetición de la ejecución de los trabajos

Puede volver a ejecutar un trabajo que tenga dispositivos con errores. Seleccione **Volver a ejecutar en dispositivos con error**.

:::image type="content" source="media/howto-run-a-job/rerun.png" alt-text="Captura de pantalla que muestra el botón para volver a ejecutar un trabajo en los dispositivos con errores":::.

Escriba el nombre y la descripción del trabajo y, a continuación, seleccione **Volver a ejecutar el trabajo**. Se envía un nuevo trabajo para volver a intentar la acción en los dispositivos con errores.

:::image type="content" source="media/howto-run-a-job/rerun-failed.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo para volver a ejecutar en dispositivos con errores":::.

> [!NOTE]
> No se pueden ejecutar más de cinco trabajos al mismo tiempo desde una aplicación de Azure IoT Central.
>
> Cuando se completa un trabajo y se elimina un dispositivo que se encuentra en la lista de dispositivos del trabajo, la entrada del dispositivo aparece como eliminada en el nombre del dispositivo. El vínculo de detalles del dispositivo no está disponible para el dispositivo eliminado.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a crear trabajos en la aplicación de Azure IoT Central, estos son algunos de los pasos siguientes:

- [Administración de dispositivos](howto-manage-devices.md)
- [Versión de la plantilla de dispositivo](howto-version-device-template.md)
