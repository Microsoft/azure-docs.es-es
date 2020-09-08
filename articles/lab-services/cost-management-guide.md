---
title: Guía de administración de costos de Azure Lab Services
description: Entienda las distintas formas de ver los costos de Lab Services.
author: rbest
ms.author: rbest
ms.date: 08/16/2020
ms.topic: article
ms.openlocfilehash: 98e04ba6bb1310935c4893a3616dfd68c2e99a55
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797639"
---
# <a name="cost-management-for-azure-lab-services"></a>Administración de costos de Azure Lab Services

Para Azure Lab Services, la administración de costos se puede dividir en dos áreas distintas: estimación de costos y análisis de costos. La estimación de costos se realiza cuando configura el laboratorio para asegurarse de que la estructura inicial de este se ajuste al presupuesto previsto. El análisis de costos normalmente se realiza al final del mes, a fin de determinar las acciones necesarias para el mes siguiente.

## <a name="estimate-the-lab-costs"></a>Estimación de los costos del laboratorio

Cada panel de laboratorio tiene una sección **Costos y facturación** que presenta una estimación aproximada de lo que va a costar el laboratorio ese mes. La estimación de costos resume el uso por horas con el número máximo de usuarios por el costo estimado por hora. Para obtener la estimación más precisa, configure el laboratorio, incluyendo la [programación](how-to-create-schedules.md). El panel reflejará el costo estimado. 

Esta estimación podría no mostrar todos los costos posibles. No se incluyen algunos recursos:

- El costo de preparación de la plantilla. La cantidad de tiempo necesaria para crear la plantilla puede variar considerablemente. El costo de ejecutar la plantilla es el mismo que el costo de laboratorio general por hora. 
- Cualquier costo de la [galería de imágenes compartidas](how-to-use-shared-image-gallery.md), ya que una galería puede compartirse entre varios laboratorios. 
- Horas utilizadas cuando el creador del laboratorio inicia una máquina virtual (VM).

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra el costo estimado en el panel.](./media/cost-management-guide/dashboard-cost-estimation.png)

## <a name="analyze-the-previous-months-usage"></a>Análisis del uso del mes anterior

El análisis de costos sirve para revisar el uso del mes anterior a fin de ayudarle a determinar cualquier ajuste necesario para el laboratorio. Puede encontrar el desglose de los costos anteriores en el [análisis del costo de la suscripción](https://docs.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis). En Azure Portal, escriba **Suscripciones** en el campo de búsqueda y seleccione la opción **Suscripciones**. 

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra el cuadro de búsqueda y la opción Suscripciones.](./media/cost-management-guide/subscription-search.png)

Seleccione la suscripción específica que quiere revisar.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra una suscripción seleccionada.](./media/cost-management-guide/subscription-select.png)

Seleccione **Análisis de costos** en el panel de la izquierda, en **Administración de costos**.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra un análisis del costo de la suscripción en un gráfico.](./media/cost-management-guide/subscription-cost-analysis.png)

Este panel permite un análisis de costos en profundidad, incluida la posibilidad de exportar a diferentes tipos de archivo siguiendo una programación. Para más información, consulte [Información general sobre Cost Management y facturación](https://docs.microsoft.com/azure/cost-management-billing/cost-management-billing-overview).

Puede filtrar por tipo de recurso. El uso de `microsoft.labservices/labaccounts` mostrará solo el costo asociado con Lab Services.

## <a name="understand-the-usage"></a>Descripción del uso

La siguiente captura de pantalla muestra un ejemplo de análisis de costos.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra un análisis de costos de ejemplo para una suscripción.](./media/cost-management-guide/cost-analysis.png)

Hay seis columnas de forma predeterminada: **Recurso**, **Tipo de recurso**, **Ubicación**, **Nombre del grupo de recursos**, **Etiquetas** y **Costo**. La columna **Recurso** contiene la información sobre la cuenta de laboratorio, el nombre del laboratorio y la máquina virtual. Las filas que muestran la cuenta de laboratorio, el nombre del laboratorio y el valor predeterminado (segunda y tercera filas) son el costo del laboratorio. Las máquinas virtuales usadas tienen un costo que puede ver para las filas que muestran la cuenta de laboratorio, el nombre del laboratorio, el valor predeterminado y el nombre de la máquina virtual. 

En este ejemplo, al sumar la primera fila y la segunda (ambas comienzan por **aaalab/dockerlab**), se obtiene el costo total del laboratorio "dockerlab" de la cuenta de laboratorio "aaalab".

Para obtener el costo total de la galería de imágenes, cambie el tipo de recurso a `Microsoft.Compute/Galleries`. Es posible que una galería de imágenes compartidas no se muestren en los costos, dependiendo de dónde esté almacenada.

> [!NOTE]
> Una galería de imágenes compartidas está conectada a la cuenta de laboratorio. Eso significa que varios laboratorios pueden usar la misma imagen.

## <a name="separate-the-costs"></a>Separación de los costos

Algunas universidades han usado la cuenta de laboratorio y el grupo de recursos como formas de separar las clases. Cada clase tiene su propia cuenta de laboratorio y su propio grupo de recursos. 

En el panel de análisis de costos, agregue un filtro basado en el nombre del grupo de recursos con el nombre del grupo de recursos adecuado para la clase. Con ello, solo estarán visibles los costos de esa clase. Esto permite una demarcación más clara entre las clases al visualizar los costos. Puede usar la característica de [exportación programada](https://docs.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data) del análisis de costos para descargar los costos de cada clase en archivos independientes.

## <a name="manage-costs"></a>Administrar costos

En función del tipo de clase, hay formas de administrar los costos a fin de reducir el número de instancias de máquinas virtuales en ejecución que no estén siendo utilizadas por un alumno.

### <a name="automatic-shutdown-settings-for-cost-control"></a>Configuración de apagado automático para el control de costos

Las características de apagado automático le permiten evitar el malgasto de horas de uso en las máquinas virtuales de los laboratorios. La configuración siguiente detecta la mayoría de los casos en los que los usuarios dejan accidentalmente sus máquinas virtuales en ejecución:

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra los tres valores de apagado automático.](./media/cost-management-guide/auto-shutdown-disconnect.png)

Puede configurar estos valores tanto a nivel de cuenta de laboratorio como a nivel de laboratorio. Si los habilita a nivel de la cuenta de laboratorio, se aplican a todos los laboratorios que se encuentran en la cuenta de laboratorio. En todas las nuevas cuentas de laboratorio, esta configuración está activada de forma predeterminada. 

#### <a name="automatically-disconnect-users-from-virtual-machines-that-the-os-deems-idle"></a>Desconexión automática de los usuarios de las máquinas virtuales que el sistema operativo considere inactivas

> [!NOTE]
> Esta opción solo está disponible para las máquinas virtuales Windows.

Cuando la opción **Desconectar a los usuarios cuando las máquinas virtuales estén inactivas** está activada, se desconecta al usuario de las máquinas del laboratorio cuando el sistema operativo Windows considera que la sesión está inactiva (lo que incluye las máquinas virtuales de plantilla). La [definición de inactividad del sistema operativo Windows](https://docs.microsoft.com/windows/win32/taskschd/task-idle-conditions#detecting-the-idle-state) usa dos criterios: 

* Ausencia de usuario: no hay ninguna entrada del teclado ni del mouse.
* Ausencia de consumo de recursos: Todos los procesadores y todos los discos han estado inactivos durante un cierto porcentaje de tiempo.

Los usuarios verán un mensaje similar al siguiente en la máquina virtual antes de que se desconecten: 

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra un mensaje de advertencia de que una sesión ha pasado el tiempo límite de inactividad y se desconectará.](./media/cost-management-guide/idle-timer-expired.png)
 
La máquina virtual sigue en ejecución cuando se desconecta al usuario. Si el usuario inicia sesión para conectarse de nuevo a la máquina virtual, las ventanas o los archivos que se habían abierto o el trabajo sin guardar antes de la desconexión seguirán estando ahí. En este estado, dado que la máquina virtual está en ejecución, sigue contando como activa y acumula costos. 
 
Para apagar automáticamente las máquinas virtuales Windows inactivas que están desconectadas, use la combinación de los valores **Desconectar a los usuarios cuando las máquinas virtuales estén inactivas** y **Apagar las máquinas virtuales cuando los usuarios se desconecten**.

Por ejemplo, supongamos que configura las opciones de la siguiente manera:
 
* **Desconectar a los usuarios cuando las máquinas virtuales estén inactivas**: 15 minutos después de detectarse el estado de inactividad.
* **Apagar las máquinas virtuales cuando los usuarios se desconecten**: 5 minutos después de que el usuario se desconecte.
 
Las máquinas virtuales Windows se apagarán automáticamente 20 minutos después de que el usuario deje de usarlas. 
 
> [!div class="mx-imgBorder"]
> ![Diagrama que muestra la combinación de valores que resulta en el cierre automático de la máquina virtual.](./media/cost-management-guide/vm-idle-diagram.png)

#### <a name="automatically-shut-down-virtual-machines-when-users-disconnect"></a>Apagado automático de las máquinas virtuales cuando los usuarios se desconecten
 
El valor **Apagar las máquinas virtuales cuando los usuarios se desconectan** admite máquinas virtuales Windows y Linux. Cuando esta opción está activada, el apagado automático se realizará cuando:
 
* En Windows, la conexión a Escritorio remoto (RDP) está desconectada.
* En Linux, la conexión SSH está desconectada.
 
> [!NOTE]
> Solo se admiten [distribuciones y versiones específicas de Linux](https://docs.microsoft.com/azure/virtual-machines/extensions/diagnostics-linux#supported-linux-distributions).
 
Puede especificar el tiempo que las máquinas virtuales deben esperar a que el usuario vuelva a conectarse antes de que se apaguen automáticamente. 

#### <a name="automatically-shut-down-virtual-machines-that-are-started-but-users-dont-connect"></a>Apagado automático de las máquinas virtuales que se han iniciado, pero a las que los usuarios no se conectan
 
En un laboratorio, un usuario puede iniciar una máquina virtual pero no llegar a conectarse a ella. Por ejemplo:
 
* Una programación en el laboratorio inicia todas las máquinas virtuales de la sesión de una clase, pero algunos alumnos no aparecen o no se conectan a sus máquinas. 
* Un usuario inicia una máquina virtual, pero se olvida de conectarse. 
 
El valor **Apagar las máquinas virtuales cuando los usuarios no se conecten** detectará estos casos y apagará automáticamente las máquinas virtuales. 
 
Para información sobre cómo configurar y habilitar el apagado automático de las máquinas virtuales al desconectarse, consulte estos artículos:

* [Configuración del apagado automático de las máquinas virtuales de una cuenta de laboratorio](how-to-configure-lab-accounts.md)
* [Configuración del apagado automático de las máquinas virtuales de un laboratorio](how-to-enable-shutdown-disconnect.md)

### <a name="scheduled-time-vs-quota-time"></a>Tiempo programado frente a tiempo de cuota

Entender el [tiempo programado](classroom-labs-concepts.md#schedules) frente al [tiempo de cuota](classroom-labs-concepts.md#quota) le permite configurar un laboratorio de la manera que mejor se ajuste a las necesidades del profesor y los alumnos. 

El tiempo programado es una hora establecida a la que todas las máquinas virtuales de los alumnos se han iniciado y están disponibles para conectarse. Normalmente, el tiempo programado se usa cuando todos los alumnos tienen su propia máquina virtual y están siguiendo las instrucciones del profesor a una hora establecida durante el día (como las horas de clase). La desventaja es que todas las máquinas virtuales de los alumnos se inician y están acumulando costos, aunque algún alumno no inicie sesión en la máquina virtual. 

El tiempo de cuota es el tiempo asignado a cada alumno, que este puede usar como decida, y que se suele usar para el estudio independiente. Las máquinas virtuales no se inician hasta que el alumno inicia la máquina virtual. 

Un laboratorio puede usar el tiempo de cuota, tiempo programado o una combinación de ambos. Si una clase no necesita tiempo programado, use solo el tiempo de cuota para un uso más eficaz de las máquinas virtuales.

### <a name="scheduled-event-stop-only"></a>Evento programado: solo detención

En la programación puede agregar un tipo de evento de solo detención, que detiene todas las máquinas a una hora determinada. Algunos propietarios de laboratorio establecen un evento de solo detención cada día a medianoche para reducir el costo y el uso de cuota si un alumno olvida apagar la máquina virtual que está usando. El inconveniente de este tipo de evento es que todas las máquinas virtuales se apagan, aunque un alumno esté usando una de ellas.

### <a name="other-costs-related-to-labs"></a>Otros costos relacionados con los laboratorios 

Hay costos que no se incluyen en Lab Services, pero que se pueden asociar a un servicio de laboratorio. Puede conectar una galería de imágenes compartidas a un laboratorio, pero no se mostrará en los costos de Lab Services, aunque tiene costos. Para ayudar a reducir los costos generales, debe quitar las imágenes no usadas de la galería, ya que las imágenes tienen un costo de almacenamiento heredado. 

Los laboratorios pueden tener conexiones a otros recursos de Azure a través de una red virtual. Cuando se quita un laboratorio, debe quitar la red virtual y los demás recursos.

## <a name="conclusion"></a>Conclusión

Esperamos que la información de este artículo le haya dado una mejor comprensión de las herramientas que pueden ayudarle a reducir los costos de uso.
