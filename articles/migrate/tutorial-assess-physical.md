---
title: Evaluación de servidores físicos para la migración a Azure con Azure Migrate Server Assessment
description: Describe cómo evaluar los servidores físicos en el entorno local para la migración a Azure con Azure Migrate Server Assessment.
ms.topic: tutorial
ms.date: 04/15/2020
ms.openlocfilehash: 5b4d5241e4236d4c11f2e2a5a8feb7c73258cba0
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171380"
---
# <a name="assess-physical-servers-with-azure-migrateserver-assessment"></a>Evaluación de los servidores físicos con Azure Migrate:Server Assessment

En este artículo se muestra cómo evaluar servidores físicos locales mediante la herramienta Azure Migrate:Server Assessment.

[Azure Migrate](migrate-services-overview.md) proporciona un centro de herramientas que le ayuda a detecta las aplicaciones, la infraestructura y las cargas de trabajo, a evaluarlas y a migrarlas a Microsoft Azure. Este centro incluye herramientas de Azure Migrate y ofertas de fabricantes de software independientes (ISV) de terceros.

Este tutorial es el segundo de una serie que muestra cómo evaluar y migrar servidores físicos a Azure. En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear un proyecto de Azure Migrate.
> * Configurar un dispositivo con Azure Migrate que se ejecute de forma local para evaluar los servidores físicos.
> * Iniciar la detección continua de máquinas virtuales en el entorno local. El dispositivo envía a Azure los datos de configuración y rendimiento de los servidores detectados.
> * Agrupe los servidores detectados y evalúe el grupo de servidores.
> * Revisar la evaluación.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. Para obtener instrucciones detalladas, consulte los artículos acerca de los distintos procedimientos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prerequisites"></a>Requisitos previos

- [Complete](tutorial-prepare-physical.md) el primer tutorial de esta serie. Si no lo hace, las instrucciones de este tutorial no funcionarán.
- Esto es lo que debería haber hecho en el primer tutorial:
    - [Configurar los permisos de Azure](tutorial-prepare-physical.md) para Azure Migrate.
    - [Preparar los servidores físicos](tutorial-prepare-physical.md#prepare-for-physical-server-assessment) para la evaluación. Se deben comprobar los requisitos del dispositivo. También es preciso tener una cuenta configurada para la detección de servidores físicos. Los puertos necesarios deben estar disponibles y es preciso saber cuáles son las direcciones URL necesarias para acceder a Azure.




## <a name="set-up-an-azure-migrate-project"></a>Configuración de un proyecto de Azure Migrate

Para configurar un proyecto nuevo de Azure Migrate, siga los pasos que se indican a continuación.

1. En Azure Portal > **Todos los servicios**, busque **Azure Migrate**.
2. En **Servicios**, seleccione **Azure Migrate**.
3. En **Información general**, en **Detectar, evaluar y migrar servidores**, haga clic en **Evaluar y migrar servidores**.

    ![Detección y evaluación de servidores](./media/tutorial-assess-physical/assess-migrate.png)

4. En **Introducción**, haga clic en **Agregar herramientas**.
5. En **Migrar proyecto**, seleccione la suscripción a Azure y cree un grupo de recursos, en caso de que no lo tenga.  
6. En **Detalles del proyecto**, especifique el nombre del proyecto y la geografía en que desea crearlo. Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

    - La geografía del proyecto solo se utiliza para almacenar los metadatos que se recopilan de los servidores en el entorno local.
    - Al realizar una migración se puede seleccionar cualquier región de destino.

    ![Crear un proyecto de Azure Migrate](./media/tutorial-assess-physical/migrate-project.png)


7. Haga clic en **Next**.
8. En **Seleccione una herramienta de evaluación**, seleccione **Azure Migrate: Server Assessment** > **Siguiente**.

    ![Crear un proyecto de Azure Migrate](./media/tutorial-assess-physical/assessment-tool.png)

9. En **Seleccione una herramienta de migración**, seleccione **Omitir por ahora la adición de una herramienta de migración** > **Siguiente**.
10. En **Revisar y agregar herramientas**, revise la configuración y haga clic en **Agregar herramientas**.
11. Espere unos minutos para que se implemente el proyecto de Azure Migrate. Se le dirigirá a la página del proyecto. Si no ve el proyecto, puede acceder a él desde **Servidores** en el panel de Azure Migrate.


## <a name="set-up-the-appliance"></a>Configuración del dispositivo

Azure Migrate: Server Assessment ejecuta un dispositivo ligero.

- Este dispositivo realiza la detección del servidor físico y envía los metadatos y los datos de rendimiento a Azure Migrate Server Assessment.
- Para configurar el dispositivo:
    - Descargue un archivo comprimido con el script del instalador de Azure Migrate desde Azure Portal.
    - Extraiga el contenido del archivo comprimido. Inicie la consola de PowerShell con privilegios administrativos.
    - Ejecute el script de PowerShell para iniciar la aplicación web del dispositivo.
    - Configurar el dispositivo por primera vez y registrarlo en el proyecto de Azure Migrate.
- Se pueden configurar varias aplicaciones para un solo proyecto de Azure Migrate. En todos los dispositivos, puede detectar cualquier número de servidores físicos. Se puede detectar un máximo de 1000 servidores por dispositivo.

### <a name="download-the-installer-script"></a>Descarga del script del instalador

Descargue el archivo comprimido del dispositivo.

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , haga clic en **No virtualizado/Otro**.
3. Haga clic en **Descargar** para descargar el archivo comprimido.

    ![Descarga del instalador](./media/tutorial-assess-physical/download-appliance.png)


### <a name="verify-security"></a>Comprobación de la seguridad

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP:
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo de uso para la nube pública: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller.zip SHA256 ```
    - Ejemplo de uso para la nube gubernamental: ```  C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip SHA256 ```
3.  Compruebe las versiones más recientes del dispositivo y los valores hash:
    - Para la nube pública:

        **Escenario** | **Descargar*** | **Valor del código hash**
        --- | --- | ---
        Físico (63,1 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2105112) | 0a27adf13cc5755e4b23df0c05732c6ac08d1fe8850567cb57c9906fbc3b85a0

    - Para Azure Government:

        **Escenario** | **Descargar*** | **Valor del código hash**
        --- | --- | ---
        Físico (63,1 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2120100&clcid=0x409) | 93dfef131026e70acdfad2769cd208ff745ab96a96f013cdf3f9e1e61c9b37e1

### <a name="run-the-azure-migrate-installer-script"></a>Ejecución del script del instalador de Azure Migrate

El script del instalador hace lo siguiente:

- Instala los agentes y una aplicación web para la detección y evaluación de los servidores físicos.
- Instala los roles de Windows, incluido el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con detalles de configuración persistentes para Azure Migrate.
- Crea los siguientes archivos en la ruta de acceso:
    - **Archivos de configuración**:%Programdata%\Microsoft Azure\Config
    - **Archivos de registro**:%Programdata%\Microsoft Azure\Logs

Ejecute el script como se indica a continuación:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo.  No ejecute el script en una máquina en un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor anterior con privilegios administrativos (elevados).
3. Cambie el directorio de PowerShell a la carpeta en la que se ha extraído el contenido del archivo comprimido descargado.
4. Ejecute el script denominado **AzureMigrateInstaller.ps1** ejecutando el comando siguiente:

    - Para la nube pública: ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller> AzureMigrateInstaller.ps1 ```
    - Para Azure Government: ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>AzureMigrateInstaller.ps1 ```

    El script iniciará la aplicación web del dispositivo cuando finalice correctamente.

En caso de que surja algún problema, puede acceder a los registros de script en C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log para solucionarlo.

### <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).


### <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez.

1. Abra un explorador en cualquier máquina que pueda conectarse al dispositivo y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio, para lo que debe hacer clic en el acceso directo de la aplicación.
2. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
    - **License** (Licencia): Acepte los términos de licencia y lea la información de terceros.
    - **Connectivity** (Conectividad): la aplicación comprueba que el servidor tenga acceso a Internet. Si el servidor usa un proxy:
        - Haga clic en **Configuración de proxy** y especifique el puerto de escucha y la dirección del proxy con los formatos http://ProxyIPAddress o http://ProxyFQDN.
        - Especifique las credenciales si el proxy requiere autenticación.
        - Solo se admite un proxy HTTP.
    - **Time sync** (Sincronización de hora): Se comprueba la hora. Para que la detección del servidor funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
    - **Instalación de actualizaciones**: Azure Migrate Server Assessment comprueba que el dispositivo tiene instaladas las actualizaciones más recientes.

### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Haga clic en **Iniciar sesión**. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
2. En la pestaña nueva, inicie sesión con sus credenciales de Azure.
    - Inicie sesión con su nombre de usuario y contraseña.
    - No se admite el inicio de sesión con un PIN.
3. Después de iniciar sesión correctamente, vuelva a la aplicación web.
4. Seleccione la suscripción en la que se creó el proyecto de Azure Migrate. Seleccione el proyecto.
5. Escriba un nombre para el dispositivo. Este nombre debe ser alfanumérico y no puede tener más de 14 caracteres.
6. Haga clic en **Registrar**.


## <a name="start-continuous-discovery"></a>Inicio de detección continua

Ahora, conecte desde el dispositivo a los servidores físicos que se van a detectar e inicie la detección.

1. Haga clic en **Agregar credenciales** para especificar las credenciales de la cuenta que el dispositivo utilizará para detectar los servidores.  
2. Inicie sesión con el nombre de usuario y la contraseña. No se admite el inicio de sesión con una clave. Además, el usuario debe tener credenciales de inicio de sesión raíz o ser parte del grupo de administradores local.
3. Especifique el **sistema operativo**, el nombre descriptivo de las credenciales, el nombre de usuario y la contraseña. A continuación, haga clic en **Agregar**.
Puede agregar varias credenciales para los servidores Windows y Linux.
4. Haga clic en **Agregar servidor** y especifique los detalles del servidor: FQDN/dirección IP y nombre descriptivo de las credenciales (una entrada por fila) para conectarse al servidor.
5. Haga clic en **Validar**. Después de la validación, se muestra la lista de servidores que se pueden detectar.
    - Si se produce un error de validación para un servidor, mantenga el puntero del mouse sobre el icono en la columna **Estado** para revisar el error. Corrija los problemas y vuelva a validar.
    - Para quitar un servidor, seleccione > **Eliminar**.
6. Después de la validación, haga clic en **Guardar e iniciar la detección** para iniciar el proceso de detección.

De esta forma comienza la detección. Los metadatos de los servidores detectados tardan alrededor de 1.5 minutos por servidor en aparecer en Azure Portal.

### <a name="verify-servers-in-the-portal"></a>Comprobación de los servidores en el portal

Después de la detección, puede comprobar que los servidores aparecen en Azure Portal.

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Assessment**, haga clic en el icono que muestra el número de **servidores detectados**.

## <a name="set-up-an-assessment"></a>Configuración de una evaluación

Con Azure Migrate: Server Assessment se pueden crear dos Server Assessment.

**Valoración** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones basadas en los datos de rendimiento recopilados | **Tamaño de máquina virtual recomendado**: se basa en los datos de uso de la CPU y de la memoria.<br/><br/> **Tipo de disco recomendado (disco administrado estándar o Premium**): se basa en IOPS y en el rendimiento de los discos locales.
**Como local** | Evaluaciones que se basan en el tamaño local. | **Tamaño de máquina virtual recomendado**: se basa en el tamaño del servidor en el entorno local<br/><br> **Tipo de disco recomendado**: se basa en el valor del tipo de almacenamiento que se selecciona para la evaluación.


### <a name="run-an-assessment"></a>Ejecución de una evaluación

Las evaluaciones se realizan como se indica a continuación:

1. Consulte los [procedimientos recomendados](best-practices-assessment.md) para crear evaluaciones.
2. En la pestaña **Servidores**, en el icono **Azure Migrate: Server Assessment**, haga clic en **Evaluar**.

    ![Evaluar](./media/tutorial-assess-physical/assess.png)

2. En **Evaluar los servidores**, especifique el nombre de la evaluación.
3. Haga clic en **View all** (Ver todo) para revisar la configuración de la valoración.

    ![Propiedades de la evaluación](./media/tutorial-assess-physical/view-all.png)

3. En **Seleccionar o crear un grupo**, seleccione **Crear nuevo** y especifique un nombre de grupo. Un grupo recopila uno o varios servidores para la evaluación.
4. En **Agregar máquinas al grupo**, seleccione los servidores que se van a agregar al grupo.
5. Haga clic en **Crear evaluación** para crear el grupo y realizar la evaluación.

    ![Crear una evaluación](./media/tutorial-assess-physical/assessment-create.png)

6. Una vez creada la evaluación, se puede ver en **Servidores** > **Azure Migrate: Server Assessment** > **Evaluaciones**.
7. Haga clic en **Exportar la evaluación** para descargarla como un archivo de Excel.



## <a name="review-an-assessment"></a>Revisión de una evaluación

Una evaluación describe:

- **Preparación para Azure**: si los servidores son adecuados para la migración a Azure.
- **Estimación del costo mensual**: los costos mensuales estimados de proceso y almacenamiento por ejecutar los servidores en Azure.
- **Estimación del costo mensual de almacenamiento**: costos estimados del almacenamiento en disco después de la migración.

### <a name="view-an-assessment"></a>Visualización de una evaluación

1. En **Objetivos de migración** >  **Servidores**, haga clic en **Evaluaciones** en **Azure Migrate: Server Assessment**.
2. En **Evaluaciones**, haga clic en una evaluación para abrirla.

    ![Resumen de evaluaciones](./media/tutorial-assess-physical/assessment-summary.png)

### <a name="review-azure-readiness"></a>Revisión de la preparación para Azure

1. En **Preparación para Azure**, compruebe si los servidores están listos para la migración a Azure.
2. Revise el estado:
    - **Preparada para Azure**: Azure Migrate recomienda unas estimaciones de tamaño y costo de las máquinas virtuales en la evaluación.
    - **Preparado con condiciones**: muestra los problemas y las soluciones que se sugieren.
    - **No está preparado para Azure**: muestra los problemas y las soluciones que se sugieren.
    - **Preparación desconocida**: se usa cuando Azure Migrate no puede evaluar la preparación debido a problemas de disponibilidad de datos.

2. Haga clic en cualquiera de los estados de **Preparación para Azure**. Puede ver los detalles de la preparación del servidor y explorar en profundidad sus detalles, entre los que se incluye la configuración del proceso, el almacenamiento y la red.



### <a name="review-cost-details"></a>Revisión de los datos de costo

Esta vista muestra el costo estimado de almacenamiento y proceso que conlleva ejecutar máquinas virtuales en Azure.

1. Revise los costos mensuales de proceso y almacenamiento. Los costos se agregan para todos los servidores del grupo evaluado.

    - Las estimaciones de costo se basan en las recomendaciones de tamaño de una máquina, así como en sus discos y propiedades.
    - Se muestran los costos mensuales estimados de proceso y almacenamiento.
    - La estimación de costos es por la ejecución de los servidores en el entorno local como máquinas virtuales IaaS. Azure Migrate Server Assessment no tiene en cuenta los costos de PaaS o SaaS.

2. Puede revisar las estimaciones del costo de almacenamiento mensual. Esta vista muestra los costos de almacenamiento agregados del grupo evaluado, divididos por los diferentes tipos de discos de almacenamiento.
3. Puede explorar en profundidad para ver los detalles de servidores concretos.


### <a name="review-confidence-rating"></a>Examen de la clasificación de confianza

Cuando se realizan valoraciones basadas en el rendimiento, se asigna una clasificación de confianza a la evaluación.

![Clasificación de confianza](./media/tutorial-assess-physical/confidence-rating.png)

- Se concede una clasificación que oscila entre 1 estrella (la más baja) y 5 estrellas (la más alta).
- La clasificación de confianza sirve de ayuda para calcular la confiabilidad de las recomendaciones de tamaño que proporciona la evaluación.
- La clasificación de confianza se basa en la disponibilidad de los puntos de datos necesarios para calcular tal evaluación.

Estas son las posibles clasificaciones de confianza de una evaluación.

**Disponibilidad del punto de datos** | **Clasificación de confianza**
--- | ---
0 % - 20 % | 1 estrella
21 % - 40 % | 2 estrellas
41 % - 60 % | 3 estrellas
61 % - 80 % | 4 estrellas
81 % - 100 % | 5 estrellas

[Más información](best-practices-assessment.md#best-practices-for-confidence-ratings) acerca de los procedimientos recomendados para las clasificaciones de confianza.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configurar un dispositivo de Azure Migrate
> * Crear y revisar una evaluación

Vaya al tercer tutorial de la serie, donde aprenderá a migrar servidores físicos a Azure con Azure Migrate: Server Migration.

> [!div class="nextstepaction"]
> [Migración de servidores físicos](./tutorial-migrate-physical-virtual-machines.md)
