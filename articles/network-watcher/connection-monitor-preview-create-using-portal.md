---
title: 'Creación de una instancia de Connection Monitor (versión preliminar): Azure Portal'
titleSuffix: Azure Network Watcher
description: Aprenda a crear una instancia de Connection Monitor (versión preliminar) mediante Azure Portal.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/30/2020
ms.author: vinigam
ms.openlocfilehash: 4e7067e537ce8fb6faf82198f7863957f79ffb22
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87567833"
---
# <a name="create-a-connection-monitor-preview-using-the-azure-portal"></a>Creación de una instancia de Connection Monitor (versión preliminar) mediante Azure Portal

Aprenda a crear una instancia de Connection Monitor (versión preliminar) para supervisar la comunicación entre los recursos mediante Azure Portal. Admite implementaciones híbridas y en la nube de Azure.

## <a name="before-you-begin"></a>Antes de empezar 

En los monitores de conexión que crea en el Monitor de conexión (versión preliminar), puede agregar máquinas locales y máquinas virtuales de Azure como orígenes. Estos monitores de conexión también pueden supervisar la conectividad con los puntos de conexión. Los puntos de conexión pueden estar en Azure o en cualquier otra dirección URL o IP.

El Monitor de conexión (versión preliminar) incluye estas entidades:


* **Recurso de monitor de conexión**: un recurso de Azure específico de la región. Todas las entidades siguientes son propiedades de un recurso de monitor de conexión.
* **Punto de conexión**: un origen o destino que participa en las comprobaciones de conectividad. Algunos ejemplos de puntos de conexión son las máquinas virtuales de Azure, los agentes locales, las direcciones URL y las direcciones IP.
* **Configuración de prueba**: la configuración de una prueba específica para el protocolo. En función del protocolo elegido, puede definir el puerto, los umbrales, la frecuencia de las pruebas y otros parámetros.
* **Grupo de prueba**: el grupo que contiene puntos de conexión de origen, puntos de conexión de destino y configuraciones de prueba. Un monitor de conexión puede contener más de un grupo de prueba.
* **Prueba**: la combinación de un punto de conexión de origen, un punto de conexión de destino y una configuración de prueba. Una prueba es el nivel más granular en el que están disponibles los datos de supervisión. Los datos de supervisión incluyen el porcentaje de comprobaciones con errores y el tiempo de ida y vuelta (RTT).

    ![Diagrama que muestra un monitor de conexión y donde se define la relación entre los grupos de prueba y las pruebas](./media/connection-monitor-2-preview/cm-tg-2.png)

## <a name="steps-to-create"></a>Pasos de creación

Para crear una instancia de Connection Monitor (versión preliminar) mediante Azure Portal, siga estos pasos:

1. En la página principal de Azure Portal, vaya a **Network Watcher**.
1. A la izquierda, en la sección **Supervisión**, seleccione **Monitor de conexión (versión preliminar)** .
1. Verá todos los monitores de conexión que se crearon en el Monitor de conexión (versión preliminar). Para ver los monitores de conexión que se crearon en la experiencia clásica del Monitor de conexión, vaya a la pestaña **Monitor de conexión**.

    ![Captura de pantalla que muestra los monitores de conexión que se crearon en el Monitor de conexión (versión preliminar)](./media/connection-monitor-2-preview/cm-resource-view.png)   
    
1. En el panel **Monitor de conexión (versión preliminar)** , en la esquina superior izquierda, seleccione **Crear**.
1. En la pestaña **Aspectos básicos**, escriba la información correspondiente al monitor de conexión:
   * **Nombre del monitor de conexión**: agregue el nombre del monitor de conexión. Use las reglas de nomenclatura estándar para los recursos de Azure.
   * **Suscripción**: elija una suscripción para el monitor de conexión.
   * **Región**: elija una región para el monitor de conexión. Solo puede seleccionar las VM de origen que se crean en esta región.
   * **Configuración del área de trabajo**: el área de trabajo contiene los datos de supervisión. Puede usar un área de trabajo personalizada o el área de trabajo predeterminada. 
       * Para usar el área de trabajo predeterminada, active la casilla. 
       * Para elegir un área de trabajo personalizada, desactive la casilla. A continuación, elija la suscripción y la región del área de trabajo personalizada. 
1. En la parte inferior de la pestaña, seleccione **Siguiente: grupos de prueba**.

   ![Captura de pantalla que muestra la pestaña Aspectos básicos en el Monitor de conexión](./media/connection-monitor-2-preview/create-cm-basics.png)

1. En la pestaña **Grupos de prueba**, seleccione **+ Grupo de prueba**. Para configurar los grupos de prueba, consulte [Creación de grupos de prueba en el Monitor de conexión](#create-test-groups-in-a-connection-monitor). 
1. En la parte inferior de la pestaña, seleccione **Siguiente: Revisar y crear** para revisar el monitor de conexión.

   ![Captura de pantalla que muestra la pestaña Grupos de prueba y el panel donde se agregan los detalles del grupo de prueba](./media/connection-monitor-2-preview/create-tg.png)

1. En la pestaña **Revisar y crear**, revise la información básica y los grupos de prueba antes de crear el monitor de conexión. Si necesita editar el monitor de conexión:
   * Para editar los detalles básicos, seleccione el icono de lápiz.
   * Para editar un grupo de prueba, selecciónelo.

   > [!NOTE] 
   > La pestaña **Revisar y crear** muestra el costo mensual durante la fase de versión preliminar del Monitor de conexión. Actualmente, la columna **CURRENT COST** no muestra ningún cargo. Cuando el Monitor de conexión esté disponible con carácter general, esta columna mostrará un cargo mensual. 
   > 
   > Incluso en la fase de versión preliminar del Monitor de conexión, se aplican los cargos de ingesta de Log Analytics.

1. Cuando esté listo para crear el monitor de conexión, en la parte inferior de la pestaña **Revisar y crear**, seleccione **Crear**.

   ![Captura de pantalla del Monitor de conexión que muestra la pestaña Revisar y crear](./media/connection-monitor-2-preview/review-create-cm.png)

El Monitor de conexión (versión preliminar) creará el recurso de monitor de conexión en segundo plano.

## <a name="create-test-groups-in-a-connection-monitor"></a>Creación de grupos de prueba en un monitor de conexión

Cada grupo de prueba de un monitor de conexión incluye orígenes y destinos que se prueban en parámetros de red. Se prueban para ver el porcentaje de comprobaciones con errores y el valor de RTT en las configuraciones de prueba.

En Azure Portal, para crear un grupo de prueba en un monitor de conexión, especifique los valores para los campos siguientes:

* **Disable Test Group** (Deshabilitar grupo de prueba): puede seleccionar este campo para deshabilitar la supervisión de todos los orígenes y destinos que el grupo de prueba especifica. Esta selección está desactivada de forma predeterminada.
* **Nombre**: nombre del grupo de prueba.
* **Orígenes**: puede especificar máquinas virtuales de Azure y máquinas locales como orígenes si los agentes están instalados en ellas. Para instalar un agente para el origen, consulte [Instalación de los agentes de supervisión](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#install-monitoring-agents).
   * Para elegir los agentes de Azure, seleccione la pestaña **Agentes de Azure**. Aquí solo se verán las máquinas virtuales que están enlazadas a la región que especificó al crear el monitor de conexión. De manera predeterminada, las máquinas virtuales se agrupan en la suscripción a la que pertenecen. Estos grupos están contraídos. 
   
       Puede explorar en profundidad desde el nivel de suscripción a otros niveles de la jerarquía:

      **Suscripción** > **Grupos de recursos** > **Redes virtuales** > **Subredes** > **Máquinas virtuales con agentes**

      También puede cambiar el valor del campo **Agrupar por** para iniciar el árbol desde cualquier otro nivel. Por ejemplo, si se agrupa por red virtual, verá las máquinas virtuales que tienen agentes en la jerarquía **Redes virtuales** > **Subredes** > **Máquinas virtuales con agentes**.

      ![Captura de pantalla del Monitor de conexión, que muestra el panel Agregar orígenes y la pestaña Agentes de Azure](./media/connection-monitor-2-preview/add-azure-sources.png)

   * Para elegir los agentes locales, seleccione la pestaña **Agentes que no son de Azure**. De forma predeterminada, los agentes se agrupan en áreas de trabajo por región. Todas estas áreas de trabajo tienen configurada la solución Network Performance Monitor. 
   
       Si necesita agregar Network Performance Monitor al área de trabajo, puede obtenerlo en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.NetworkMonitoringOMS?tab=Overview). Para información sobre cómo agregar Network Performance Monitor, consulte [Soluciones de supervisión en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/insights/solutions). 
   
       En la vista **Crear un monitor de conexión**, en la pestaña **Aspectos básicos**, aparece seleccionada la región predeterminada. Si cambia la región, puede elegir agentes de áreas de trabajo de la región nueva. También puede cambiar el valor del campo **Agrupar por** para agrupar por subredes.

      ![Captura de pantalla del Monitor de conexión, que muestra el panel Agregar orígenes y la pestaña Agentes que no son de Azure](./media/connection-monitor-2-preview/add-non-azure-sources.png)


   * Para revisar los agentes de Azure y los que no son de Azure que ha seleccionado, vaya a la pestaña **Revisar**.

      ![Captura de pantalla del Monitor de conexión, que muestra el panel Agregar orígenes y la pestaña Revisar](./media/connection-monitor-2-preview/review-sources.png)

   * Cuando termine de configurar los orígenes, en la parte inferior del panel **Agregar orígenes**, seleccione **Listo**.

* **Destinos**: puede supervisar la conectividad a las máquinas virtuales de Azure o a cualquier punto de conexión (una IP pública, una dirección URL o un FQDN) si se especifican como destinos. En un solo grupo de prueba, puede agregar máquinas virtuales de Azure, direcciones URL de Office 365, direcciones URL de Dynamics 365 y puntos de conexión personalizados.

    * Para elegir las máquinas virtuales de Azure como destinos, seleccione la pestaña **Máquinas virtuales de Azure**. De forma predeterminada, las máquinas virtuales de Azure se agrupan en una jerarquía de suscripción que se encuentra en la misma región que seleccionó en la vista **Crear un monitor de conexión**, en la pestaña **Aspectos básicos**. Puede cambiar la región y elegir VM de Azure de la región recién seleccionada. Luego puede explorar en profundidad desde el nivel Suscripción a otros niveles de la jerarquía, como el nivel Agentes de Azure.

       ![Captura de pantalla del panel Agregar destinos, donde se muestra la pestaña Máquinas virtuales de Azure](./media/connection-monitor-2-preview/add-azure-dests1.png)

       ![Captura de pantalla del panel Agregar destinos en el que se muestra el nivel Suscripción](./media/connection-monitor-2-preview/add-azure-dests2.png)

    * Para elegir puntos de conexión como destinos, seleccione la pestaña **Puntos de conexión**. La lista de puntos de conexión incluye las direcciones URL de prueba de Office 365 y las direcciones URL de prueba de Dynamics 365, agrupadas por nombre. Además de estos puntos de conexión, puede elegir un punto de conexión creado en otros grupos de prueba en el mismo monitor de conexión. 
    
        Para agregar un punto de conexión nuevo, en la esquina superior derecha, seleccione **+ Puntos de conexión**. A continuación, proporcione un nombre de punto de conexión y una dirección URL, IP o FQDN.

       ![Captura de pantalla que muestra dónde agregar puntos de conexión como destinos en el Monitor de conexión](./media/connection-monitor-2-preview/add-endpoints.png)

    * Para revisar las máquinas virtuales de Azure y los puntos de conexión que eligió, seleccione la pestaña **Revisar**.
    * Cuando termine de elegir destinos, seleccione **Listo**.

* **Configuraciones de prueba**: puede asociar configuraciones de prueba a un grupo de prueba. Azure Portal solo permite una configuración de prueba por cada grupo de prueba, pero puede usar ARMClient para agregar más.

    * **Nombre**: asigne un nombre a la configuración de prueba.
    * **Protocolo**: elija TCP, ICMP o HTTP. Para cambiar HTTP a HTTPS, seleccione **HTTP** como el protocolo y **443** como el puerto.
        * **Crear configuración de prueba de red**: esta casilla solo aparece si selecciona **HTTP** en el campo **Protocolo**. Active esta casilla para crear otra configuración de prueba que use los mismos orígenes y destinos que especificó en cualquier otra parte de la configuración. La configuración de prueba recién creada se denomina `<the name of your test configuration>_networkTestConfig`.
        * **Deshabilitar traceroute**: este campo se aplica a los grupos de prueba cuyo protocolo es TCP o ICMP. Active esta casilla evitar que los orígenes detecten la topología y RTT de salto a salto.
    * **Puerto de destino**: puede personalizar este campo con un puerto de destino de su elección.
    * **Frecuencia de prueba**: use este campo para elegir la frecuencia con la que los orígenes harán ping a los destinos en el protocolo y el puerto que especificó. Puede elegir 30 segundos, 1 minuto, 5 minutos, 15 minutos o 30 minutos. Los orígenes probarán la conectividad a los destinos según el valor que elija.  Por ejemplo, si selecciona 30 segundos, los orígenes comprobarán la conectividad con el destino al menos una vez en un período de 30 segundos.
    * **Umbral correcto**: puede establecer umbrales en estos parámetros de red:
       * **Comprobaciones con error**: establezca el porcentaje de comprobaciones que pueden presentar errores cuando los orígenes comprueban la conectividad con los destinos mediante los criterios especificados. En el caso del protocolo TCP o ICMP, el porcentaje de comprobaciones con error puede ser igual al porcentaje de pérdida de paquetes. En el caso del protocolo HTTP, este campo representa el porcentaje de solicitudes HTTP que no recibieron respuesta.
       * **Tiempo de ida y vuelta**: establezca el RTT en milisegundos para indicar cuánto tiempo pueden tardar los orígenes en conectarse al destino a través de la configuración de prueba.
    
       ![Captura de pantalla que muestra dónde establecer una configuración de prueba en el Monitor de conexión](./media/connection-monitor-2-preview/add-test-config.png)


## <a name="scale-limits"></a>Límites de escalado

Los monitores de conexión tienen los límites de escala siguientes:

* Máximo de monitores de conexión por suscripción por región: 100
* Máximo de grupos de prueba por monitor de conexión: 20
* Máximo de orígenes y destinos por monitor de conexión: 100
* Máximo de configuraciones de prueba por monitor de conexión: 2 a través de Azure Portal

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [analizar los datos de supervisión y a definir alertas](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#analyze-monitoring-data-and-set-alerts).
* Aprenda a [diagnosticar los problemas de la red](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#diagnose-issues-in-your-network).
