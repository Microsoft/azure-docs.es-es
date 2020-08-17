---
title: 'ML Studio (clásico): SQL Server local (Azure)'
description: Use datos de una base de datos de SQL Server para realizar análisis avanzados con Azure Machine Learning Studio (clásico).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 03/13/2017
ms.openlocfilehash: 133de2c6b4fd4f970595a0e46c24167ab10c0aea
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432118"
---
# <a name="perform-analytics-with-azure-machine-learning-studio-classic-using-a-sql-server-database"></a>Análisis mediante Azure Machine Learning Studio (clásico) con una base de datos de SQL Server

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Con frecuencia, a las compañías que trabajan con datos locales les gustaría aprovechar la escala y la agilidad de la nube para su cargas de trabajo de aprendizaje de automático. Sin embargo, no desean interrumpir los flujos de trabajo y los procesos de negocio actuales por mover sus datos locales a la nube. Ahora, Azure Machine Learning Studio (clásico) admite la lectura de los datos de una base de datos de SQL Server, seguida del entrenamiento y la puntuación de un modelo con estos datos. Ya no tendrá que copiar y sincronizar de forma manual los datos entre la nube y el servidor local. En su lugar, el módulo de **importación de datos** de Azure Machine Learning Studio (clásico) ahora puede leer directamente los trabajos de entrenamiento y puntuación en la base de datos de SQL Server.

En este artículo se proporciona información general sobre cómo introducir datos de SQL Server en Azure Machine Learning Studio (clásico). Se da por supuesto que conoce conceptos de Studio (clásico), como áreas de trabajo, módulos, conjuntos de datos, experimentos, *etc.*

> [!NOTE]
> Esta característica no está disponible para las áreas de trabajo gratuitas. Para más información acerca de los niveles y los precios de Machine Learning, consulte [Precios de Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning/).
>
>

<!-- -->



## <a name="install-the-data-factory-self-hosted-integration-runtime"></a>Instalación del entorno de ejecución de integración autohospedado de Data Factory
Para acceder a una base de datos de SQL Server en Azure Machine Learning Studio (clásico), debe descargar e instalar el entorno de ejecución de integración autohospedado de Data Factory, conocido anteriormente como Data Management Gateway. Al configurar la conexión en Machine Learning Studio (clásico), tiene la oportunidad de descargar e instalar el entorno de ejecución de integración (IR) con el cuadro de diálogo **Descarga y registro de la puerta de enlace de datos** que se describe a continuación.


También puede instalar el entorno con antelación descargando y ejecutando el paquete de instalación MSI desde [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=39717). El archivo MSI también puede usarse para actualizar una instancia del entorno existente a una versión más reciente, conservando toda la configuración.

El entorno de ejecución de integración autohospedado de Data Factory tiene los siguientes requisitos previos:

* Su integración requiere un sistema operativo de 64 bits con .NET Framework 4.6.1 o posterior.
* Las versiones de sistema operativo Windows compatibles son Windows 10, Windows Server 2012, Windows Server 2012 R2 y Windows Server 2016. 
* La configuración recomendada para la máquina del entorno de ejecución de integración es de al menos 2 GHz, CPU de 4 núcleos, 8 GB de RAM y un disco de 80 GB.
* Si la máquina host está en hibernación, el entorno de ejecución de integración no responderá a las solicitudes de datos. Por tanto, configure un plan de energía adecuado en el equipo antes de instalar el entorno de ejecución de integración. Si la máquina está configurada para hibernar, en la instalación del entorno se muestra un mensaje.
* Dado que la actividad de copia sucede con una frecuencia determinada, el uso de recursos (CPU, memoria) en el equipo también sigue el mismo patrón con horas pico y de inactividad. El uso de recursos también depende en gran medida de la cantidad de datos que se mueven. Cuando haya varios trabajos de copia en curso, observará que el uso de los recursos aumenta durante las horas pico. Aunque la configuración mínima de arriba es técnicamente suficiente, se recomienda una con más recursos en función de la carga específica para el movimiento de datos.

Tenga en cuenta lo siguiente al configurar y usar el entorno de ejecución de integración autohospedado de Data Factory:

* Solo puede instalar una instancia del entorno en un solo equipo.
* Puede usar un solo entorno para varios orígenes de datos locales.
* Puede conectar varios entornos en diferentes equipos al mismo origen de datos local.
* Un entorno se configura para una única área de trabajo cada vez. Actualmente, los entornos de ejecución de integración no se pueden compartir entre áreas de trabajo.
* Puede configurar varios para una sola área de trabajo. Por ejemplo, podría usar uno que esté conectado a los orígenes de datos de prueba durante el desarrollo y otro de producción cuando esté preparado para ponerlo en funcionamiento.
* El entorno no tiene por qué estar en la misma máquina que el origen de datos. Sin embargo, si está más cerca del origen de datos, se reduce el tiempo que necesita la puerta de enlace para conectarse a este. Se recomienda que instale el entorno de ejecución de integración en una máquina diferente de la que hospeda el origen de datos local para que la puerta de enlace y el origen de datos no compitan por los recursos.
* Si ya tiene un entorno de integración instalado en el equipo que atiende los escenarios de Power BI o Azure Data Factory, instale uno distinto para Azure Machine Learning Studio (clásico) en otro equipo.

  > [!NOTE]
  > No se puede ejecutar en el mismo equipo el entorno de ejecución de integración autohospedado de Data Factory y Power BI Gateway.
  >
  >
* Debe usar dicho entorno para Azure Machine Learning Studio (clásico) incluso si utiliza Azure ExpressRoute para otros datos. Considere el origen de datos como uno de tipo local (que está detrás de un firewall), aunque utilice ExpressRoute. Use el entorno de ejecución de integración autohospedado de Data Factory para establecer la conectividad entre Machine Learning y el origen de datos.

Encontrará información detallada sobre los requisitos previos de instalación, los pasos de instalación y sugerencias para solucionar problemas en el artículo [Entorno de ejecución de integración en Data Factory](../../data-factory/concepts-integration-runtime.md).

## <a name="span-idusing-the-data-gateway-step-by-step-walk-classanchorspan-id_toc450838866-classanchorspanspaningress-data-from-your-sql-server-database-into-azure-machine-learning"></a><span id="using-the-data-gateway-step-by-step-walk" class="anchor"><span id="_Toc450838866" class="anchor"></span></span>Entrada de datos de la base de datos de SQL Server en Azure Machine Learning
En este tutorial, instalará una instancia de Azure Data Factory Integration Runtime en un área de trabajo de Azure Machine Learning, la configurará y después leerá datos de una base de datos de SQL Server.

> [!TIP]
> Antes de comenzar, deshabilite el bloqueador de elementos emergentes del explorador para `studio.azureml.net`. Si usa el explorador Google Chrome, descargue e instale uno de los complementos disponibles en Google Chrome WebStore de entre las [extensiones de la aplicación ClickOnce](https://chrome.google.com/webstore/search/clickonce?_category=extensions).
>
> [!NOTE]
> El entorno de ejecución de integración autohospedado de Azure Data Factory se conocía anteriormente como Data Management Gateway. El tutorial paso a paso seguirá para hacer referencia a ella como una puerta de enlace.  

### <a name="step-1-create-a-gateway"></a>Paso 1: Crear una puerta de enlace
El primer paso consiste en crear y configurar la puerta de enlace para acceder a la base de datos SQL.

1. Inicie sesión en [Azure Machine Learning Studio (clásico)](https://studio.azureml.net/Home/) y seleccione el área de trabajo en la que desee trabajar.
2. Haga clic en la hoja **SETTINGS** (Configuración) de la izquierda y en la pestaña **DATA GATEWAYS** (Puertas de enlace de datos) de la parte superior.
3. Haga clic en **NEW DATA GATEWAY** (Nueva puerta de enlace de datos) de la parte inferior de la pantalla.

    ![Nueva puerta de enlace de datos](./media/use-data-from-an-on-premises-sql-server/new-data-gateway-button.png)
4. En el cuadro de diálogo **New data gateway** (Nueva puerta de enlace de datos), escriba un valor en **Gateway Name** (Nombre de puerta de enlace) y, opcionalmente, en **Description** (Descripción). Haga clic en la flecha situada en la esquina inferior derecha para ir al siguiente paso de la configuración.

    ![Especificación de la descripción y el nombre de la puerta de enlace](./media/use-data-from-an-on-premises-sql-server/new-data-gateway-dialog-enter-name.png)
5. En el diálogo Download and register data gateway (Descargar y registrar puerta de enlace), copie el valor de GATEWAY REGISTRATION KEY (Clave de registro de la puerta de enlace) en el Portapapeles.

    ![Descarga y registro de la puerta de enlace de datos](./media/use-data-from-an-on-premises-sql-server/download-and-register-data-gateway.png)
6. <span id="note-1" class="anchor"></span>Si aún no ha descargado ni instalado Microsoft Data Management Gateway, haga clic en **Download data management gateway**(Descargar Data Management Gateway). Esto lo lleva al Centro de descarga de Microsoft, donde puede seleccionar la versión de puerta de enlace que necesita, descargarla e instalarla. Encontrará información detallada sobre los requisitos previos de instalación, los pasos de instalación y sugerencias para solucionar problemas en las secciones del principio del artículo [Movimiento de datos entre orígenes locales y la nube con Data Management Gateway](../../data-factory/tutorial-hybrid-copy-portal.md).
7. Una vez instalada la puerta de enlace, se abre el Administrador de configuración de Data Management Gateway y se muestra el cuadro de diálogo **Registrar puerta de enlace** . Pegue la **clave de registro de puerta de enlace** que copió en el portapapeles y haga clic en **Registrar**.
8. Si ya tiene una puerta de enlace instalada, ejecute el Administrador de configuración de Data Management Gateway. Haga clic en **Cambiar clave**, pegue la **clave de registro de puerta de enlace** que ha copiado en el Portapapeles en el paso anterior y haga clic en **Aceptar**.
9. Cuando la instalación haya finalizado, se muestra el cuadro de diálogo **Registrar puerta de enlace** para el Administrador de configuración de Data Management Gateway. Pegue la CLAVE DE REGISTRO DE PUERTA DE ENLACE que copió en el Portapapeles en el paso anterior y haga clic en **Registrar**.

    ![Registro de la puerta de enlace](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-register-gateway.png)
10. La configuración de puerta de enlace está completa cuando se establecen los siguientes valores en la pestaña **Inicio** del Administrador de configuración de Microsoft Data Management Gateway:

    * En **Nombre de puerta de enlace** y **Nombre de instancia** aparece el nombre de la puerta de enlace.
    * **Registro** está establecido en **Registrado**.
    * El **Estado** está establecido en **Iniciado**.
    * La barra de estado de la parte inferior muestra **Conectado al servicio en la nube Data Management Gateway** junto con una marca de verificación verde.

      ![Administrador de Data Management Gateway](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-registered.png)

      También se actualiza Azure Machine Learning Studio (clásico) cuando el registro se ha completado.

    ![Registro de puerta de enlace correcto](./media/use-data-from-an-on-premises-sql-server/gateway-registered.png)
11. En el cuadro de diálogo **Download and register data gateway** (Descargar y registrar puerta de enlace), haga clic en la marca de verificación para completar la instalación. En la página **Settings** (Configuración), se muestra el estado de la puerta de enlace como "Online" (En línea). En el panel derecho, encontrará el estado y otra información útil.

    ![Configuración de puerta de enlace](./media/use-data-from-an-on-premises-sql-server/gateway-status.png)
12. En el Administrador de configuración de Microsoft Data Management Gateway, cambie a la pestaña **Certificado** . El certificado especificado en esta pestaña se usa para cifrar y descifrar las credenciales para el almacén de datos local que especifique en el portal. Este es el certificado predeterminado. Microsoft recomienda que lo cambie al suyo propio que esté en una copia de seguridad en el sistema de administración de certificados. Haga clic en **Cambiar** para usar su propio certificado.

    ![Cambio del certificado de puerta de enlace](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-certificate.png)
13. (opcional) Si desea habilitar el registro detallado para solucionar problemas con la puerta de enlace, en el Administrador de configuración de Microsoft Data Management Gateway, cambie a la pestaña **Diagnóstico** y active la casilla **Habilitar el registro detallado para la solución de problemas**. La información de registro se encuentra en el Visor de eventos de Windows, en el nodo **Registros de aplicaciones y servicios** -&gt;**Data Management Gateway**. También puede usar la pestaña **Diagnóstico** para probar la conexión a un origen de datos local mediante la puerta de enlace.

    ![Habilitación del registro detallado](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-verbose-logging.png)

Así finaliza el proceso de configuración de la puerta de enlace en Azure Machine Learning Studio (clásico).
Ya está listo para usar los datos locales.

Puede crear y configurar varias puertas de enlace en Estudio (clásico) para cada área de trabajo. Por ejemplo, puede conectar una puerta de enlace a los orígenes de datos de prueba durante el desarrollo, y una puerta de enlace diferente para los orígenes de datos de producción. Azure Machine Learning Studio (clásico) ofrece la flexibilidad de configurar varias puertas de enlace en función del entorno corporativo. Actualmente no se puede compartir una puerta de enlace entre áreas de trabajo y solamente se puede instalar una única puerta de enlace en un equipo. Para más información, consulte [Movimiento de datos entre orígenes locales y la nube con Data Management Gateway](../../data-factory/tutorial-hybrid-copy-portal.md).

### <a name="step-2-use-the-gateway-to-read-data-from-an-on-premises-data-source"></a>Paso 2: Uso de la puerta de enlace para leer datos de un origen de datos local
Después de configurar la puerta de enlace, puede agregar un módulo **Importar datos** a un experimento que introduce los datos de la base de datos de SQL Server.

1. En Machine Learning Studio (clásico), seleccione la pestaña **EXPERIMENTOS**, haga clic en la pestaña **+ NUEVO** de la esquina inferior izquierda y seleccione **Blank Experiment** (Experimento en blanco). También puede seleccionar uno de los varios experimentos de ejemplo disponibles.
2. Busque y arrastre el módulo **Importar datos** al lienzo del experimento.
3. Haga clic en **Save as** (Guardar como) bajo el lienzo. Escriba "Tutorial de SQL Server local en Azure Machine Learning Studio (clásico)" como nombre del experimento, seleccione el área de trabajo y haga clic en la marca de verificación de **Aceptar**.

   ![Guardado de un experimento con un nuevo nombre](./media/use-data-from-an-on-premises-sql-server/experiment-save-as.png)
4. Haga clic en el módulo **Importar datos** para seleccionarlo y, en el panel **Properties** (Propiedades) de la derecha del lienzo, seleccione "SQL Database local" en la lista desplegable **Data source** (Origen de datos).
5. Seleccione en **Data gateway** (Puerta de enlace de datos) la puerta de enlace que instaló y registró. Puede configurar otra si selecciona "(add new Data Gateway…)" (Agregar nueva puerta de enlace de datos).

   ![Selección de la puerta de enlace de datos para el módulo de importación de datos](./media/use-data-from-an-on-premises-sql-server/import-data-select-on-premises-data-source.png)
6. Escriba el servidor SQL en **Database server name** (Nombre del servidor de base de datos) y un valor para **Database name** (Nombre de base de datos), junto con el texto para **Database query** (Consulta de base de datos) para la consulta de SQL Database que quiera ejecutar.
7. Haga clic en **Enter values** (Escribir valores) de **User name and password** (Nombre de usuario y contraseña) y escriba sus credenciales de la base de datos. Puede usar la autenticación integrada de Windows o la autenticación de SQL Server según la configuración de SQL Server.

   ![Especificación de las credenciales de la base de datos](./media/use-data-from-an-on-premises-sql-server/database-credentials.png)

   El mensaje "values required" (valores necesarios) cambiará a "values set" (valores establecidos) con una marca de verificación verde. Basta con que escriba las credenciales una vez, a menos que la información de la base de datos o la contraseña cambien. Azure Machine Learning Studio (clásico) usa el certificado que proporcionó al instalar la puerta de enlace para cifrar las credenciales en la nube. Azure nunca almacena credenciales locales sin cifrado.

   ![Propiedades del módulo de importación de datos](./media/use-data-from-an-on-premises-sql-server/import-data-properties-entered.png)
8. Haga clic en **RUN** (Ejecutar) para ejecutar el experimento.

Una vez finalizado el experimento, puede visualizar los datos importados desde la base de datos si hace clic en el puerto de salida del módulo **Importar datos** y selecciona **Visualizar**.

Una vez que haya terminado de desarrollar el experimento, puede implementar el modelo y ponerlo en operación. Mediante el servicio de ejecución por lotes, se leerán los datos de la base de datos de SQL Server configurada en el módulo **Importar datos** y se usarán para la puntuación. Aunque puede usar el servicio de solicitud-respuesta para puntuar datos locales, Microsoft recomienda usar el [complemento de Excel](excel-add-in-for-web-services.md) en su lugar. Actualmente, no se admite la escritura en una base de datos de SQL Server mediante **Exportar datos**, ni en experimentos ni en servicios web publicados.
