---
title: Evaluación de máquinas virtuales de Hyper-V para la migración a Azure con Azure Migrate | Microsoft Docs
description: Se describe cómo evaluar máquinas virtuales de Hyper-V locales para su migración a Azure mediante Azure Migrate Server Assessment.
ms.topic: tutorial
ms.date: 06/03/2020
ms.custom: mvc
ms.openlocfilehash: eae7e2d371ed8156debe9ae24cf0744bd6273943
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950279"
---
# <a name="assess-hyper-v-vms-with-azure-migrate-server-assessment"></a>Evaluación de las máquinas virtuales de Hyper-V con Azure Migrate Server Assessment

En este artículo se muestra cómo evaluar máquinas virtuales de Hyper-V locales mediante la herramienta [Azure Migrate:Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool).


Este tutorial es el segundo de una serie que muestra cómo evaluar máquinas virtuales de Hyper-V y migrarlas a Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un proyecto de Azure Migrate.
> * Configurar y registrar un dispositivo de Azure Migrate.
> * Iniciar la detección continua de máquinas virtuales locales.
> * Agrupar las máquinas virtuales detectadas y evaluar el grupo.
> * Revisar la evaluación.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. Para obtener instrucciones detalladas, consulte los artículos acerca de los distintos procedimientos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prerequisites"></a>Requisitos previos

- [Complete](tutorial-prepare-hyper-v.md) el primer tutorial de esta serie. Si no lo hace, las instrucciones de este tutorial no funcionarán.
- Esto es lo que debería haber hecho en el primer tutorial:
    - [Preparar Azure](tutorial-prepare-hyper-v.md#prepare-azure) para trabajar con Azure Migrate.
    - [Preparar la evaluación de los hosts y máquinas virtuales de Hyper-V](tutorial-prepare-hyper-v.md#prepare-for-assessment).
    - [Comprobar](tutorial-prepare-hyper-v.md#prepare-for-appliance-deployment) lo que se necesita para implementar la aplicación de Azure Migrate para la evaluación de Hyper-V.

## <a name="set-up-an-azure-migrate-project"></a>Configuración de un proyecto de Azure Migrate

1. En Azure Portal > **Todos los servicios**, busque **Azure Migrate**.
2. En los resultados de la búsqueda, seleccione **Azure Migrate**.
3. En **Información general**, en **Detectar, evaluar y migrar servidores**, haga clic en **Evaluar y migrar servidores**.

    ![Detección y evaluación de servidores](./media/tutorial-assess-hyper-v/assess-migrate.png)

4. En **Introducción**, haga clic en **Agregar herramientas**.
5. En la pestaña **Migrar proyecto**, seleccione la suscripción a Azure y cree un grupo de recursos si no lo tiene.
6. En **Detalles del proyecto**, especifique el nombre del proyecto y la región en la que desea crearlo. Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

    - La región del proyecto solo se utiliza para almacenar los metadatos que se recopilan de las máquinas virtuales locales.
    - Puede seleccionar una región de destino de Azure diferente cuando migre las máquinas virtuales. Se admiten todas las regiones de Azure como destino de la migración.

    ![Crear un proyecto de Azure Migrate](./media/tutorial-assess-hyper-v/migrate-project.png)

7. Haga clic en **Next**.
8. En **Seleccione una herramienta de evaluación**, seleccione **Azure Migrate: Server Assessment** > **Siguiente**.

    ![Crear un proyecto de Azure Migrate](./media/tutorial-assess-hyper-v/assessment-tool.png)

9. En **Seleccione una herramienta de migración**, seleccione **Omitir por ahora la adición de una herramienta de migración** > **Siguiente**.
10. En **Revisar y agregar herramientas**, revise la configuración y haga clic en **Agregar herramientas**.
11. Espere unos minutos para que se implemente el proyecto de Azure Migrate. Se le dirigirá a la página del proyecto. Si no ve el proyecto, puede acceder a él desde **Servidores** en el panel de Azure Migrate.

## <a name="set-up-the-azure-migrate-appliance"></a>Configuración del dispositivo de Azure Migrate


Azure Migrate:Server Assessment usa una aplicación de Azure Migrate ligera. La aplicación realiza la detección de la máquina virtual y envía tanto los metadatos como los datos de rendimiento de esta a Azure Migrate. El dispositivo se puede configurar de varias maneras.

- Configúrelo en una máquina virtual de Hyper-V mediante un disco duro virtual de Hyper-V descargado. Este es el método que se usa en este tutorial.
- Configúrelo en una máquina virtual de Hyper-V o en una máquina física con un script del instalador de PowerShell. Debe usarse [este método](deploy-appliance-script.md) si no se puede configurar una máquina virtual mediante un disco duro virtual, o si se encuentra en Azure Government.

Una vez creada la aplicación, compruebe que se puede conectar a Azure Migrate:Server Assessment, configúrela por primera vez y regístrela en el proyecto de Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Generación de la clave del proyecto de Azure Migrate

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, seleccione **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**.
3. En **1: Generar la clave del proyecto de Azure Migrate**, proporcione un nombre para el dispositivo de Azure Migrate que configurará para la detección de máquinas virtuales de Hyper-V. El nombre debe ser alfanumérico con 14 caracteres como máximo.
1. Haga clic en **Generar clave** para iniciar la creación de los recursos de Azure necesarios. No cierre la página Detectar máquinas durante la creación de recursos.
1. Después de la creación correcta de los recursos de Azure, se genera una **clave del proyecto Azure Migrate**.
1. Copie la clave, ya que la necesitará para completar el registro del dispositivo durante su configuración.

### <a name="download-the-vhd"></a>Descarga del disco duro virtual

En **2: Descargar dispositivo de Azure Migrate**, seleccione el archivo .VHD y haga clic en **Descargar**. 

   ![Selecciones para Detectar máquinas](./media/tutorial-assess-hyper-v/servers-discover.png)


   ![Selecciones para Generar clave](./media/tutorial-assess-hyper-v/generate-key-hyperv.png)


### <a name="verify-security"></a>Comprobación de la seguridad

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.

2. Ejecute el siguiente comando de PowerShell para generar el código hash para el archivo ZIP
    - ```C:\>Get-FileHash -Path <file_location> -Algorithm [Hashing Algorithm]```
    - Ejemplo de uso: ```C:\>Get-FileHash -Path ./AzureMigrateAppliance_v1.19.06.27.zip -Algorithm SHA256```

3.  Compruebe las versiones más recientes del dispositivo y los valores hash:

    - Para la nube pública de Azure:

        **Escenario** | **Descargar** | **SHA256**
        --- | --- | ---
        Hyper-V (10,4 GB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140422) |  79c151588de049cc102f61b910d6136e02324dc8d8a14f47772da351b46d9127

    - Para Azure Government:

        **Escenario*** | **Descargar** | **SHA256**
        --- | --- | ---
        Hyper-V (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140424) |  8025f315e41c01ebdb4ffb1de87982ae6cc4ea7c4cce612612c7e90a44e79b44


### <a name="create-the-appliance-vm"></a>Creación de la máquina virtual del dispositivo

Importe el archivo descargado y cree la VM.

1. Después de descargar el archivo VHD comprimido en el host de Hyper-V en el que se colocará la máquina virtual del dispositivo, extraiga su contenido.
    - En la ubicación de extracción, el archivo se descomprime en una carpeta denominada **AzureMigrateAppliance_númeroDeVersión**.
    - Esta carpeta contiene una subcarpeta, también denominada **AzureMigrateAppliance_númeroDeVersión**.
    - Esta subcarpeta contiene a su vez tres subcarpetas: **Snapshots**, **Virtual Hard Disks** y **Virtual Machines**.

2. Abra el administrador de Hyper-V. En **Acciones**, haga clic en **Importar máquina virtual**.

    ![Implementar disco duro virtual](./media/tutorial-assess-hyper-v/deploy-vhd.png)

2. En el Asistente para importar máquinas virtuales > **Antes de comenzar**, haga clic en **Siguiente**.
3. En **Buscar carpeta**, seleccione la carpeta **Virtual Machines**. A continuación, haga clic en **Siguiente**.
1. En **Seleccionar máquina virtual**, haga clic en **Siguiente**.
2. En **Elegir tipo de importación**, haga clic en **Copiar la máquina virtual (crear un identificador único nuevo)** . A continuación, haga clic en **Siguiente**.
3. En **Elegir destino**, deje la configuración predeterminada. Haga clic en **Next**.
4. En **Carpetas de almacenamiento**, deje la configuración predeterminada. Haga clic en **Next**.
5. En **Elegir red**, especifique el conmutador virtual que usará la VM. El conmutador necesita conectividad a Internet para enviar datos a Azure. [Aprenda a](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-switch-for-hyper-v-virtual-machines) crear un conmutador virtual.
6. En **Resumen**, revise los valores de configuración. Haga clic en **Finalizar**.
7. En Administrador de Hyper-V > **Máquinas virtuales**, inicie la VM.


## <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que la VM del dispositivo pueda conectarse a las direcciones URL de Azure para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).

### <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez.

> [!NOTE]
> Si configura la aplicación mediante un [script de PowerShell](deploy-appliance-script.md), en lugar del disco duro virtual descargado, los dos primeros pasos de este procedimiento no son pertinentes.

1. En Administrador de Hyper-V > **Máquinas virtuales**, haga clic con el botón derecho en la VM > **Conectar**.
2. Especifique el idioma, la zona horaria y la contraseña del dispositivo.
3. Abra un explorador en cualquier equipo que pueda conectarse a la máquina virtual y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio del dispositivo, para lo que debe hacer clic en el acceso directo de la aplicación.
1. Acepte los **términos de licencia** y lea la información de terceros.
1. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
    - **Connectivity** (Conectividad): la aplicación comprueba que la máquina virtual tiene acceso a Internet. Si la máquina virtual usa un proxy:
      - Haga clic en **Configurar el proxy** y especifique la dirección del proxy (con los formatos http://ProxyIPAddress o http://ProxyFQDN) ) y el puerto de escucha.
      - Especifique las credenciales si el proxy requiere autenticación.
      - Solo se admite un proxy HTTP.
      - Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, haga clic en **Guardar** para desencadenar la comprobación de conectividad.
    - **Time sync** (Sincronización de hora): Se comprueba la hora. Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
    - **Instalación de actualizaciones**: Azure Migrate Server Assessment comprueba que el dispositivo tiene instaladas las actualizaciones más recientes. Una vez finalizada la comprobación, puede hacer clic en **Ver servicios del dispositivo** para ver el estado y las versiones de los componentes que se ejecutan en el dispositivo.

### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la **clave del proyecto de Azure Migrate** copiada desde el portal. Si no tiene la clave, vaya a **Server Assessment > Detectar > Administrar los dispositivos existentes**, seleccione el nombre del dispositivo que proporcionó en el momento de la generación de la clave y copie la clave correspondiente.
1. Haga clic en **Iniciar sesión**. Se abrirá un mensaje de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
1. En la nueva pestaña, inicie sesión con su nombre de usuario y contraseña de Azure.
   
   No se admite el inicio de sesión con un PIN.
3. Después de iniciar sesión correctamente, vuelva a la aplicación web. 
4. Si la cuenta de usuario de Azure que se usa para el registro tiene los [permisos](tutorial-prepare-hyper-v.md#prepare-azure) adecuados en los recursos de Azure creados durante la generación de la clave, se iniciará el registro del dispositivo.
1. Una vez que el dispositivo se ha registrado correctamente, puede ver los detalles de registro haciendo clic en **Ver detalles**.



### <a name="delegate-credentials-for-smb-vhds"></a>Delegación de credenciales para discos duros virtuales de SMB

Si va a ejecutar discos duros virtuales en SMB, debe habilitar la delegación de credenciales desde el dispositivo a los hosts de Hyper-V. Para ello, habilite cada host para que actúe como delegado para el dispositivo. Si ha seguido los tutoriales en orden, esto debió hacerlo en el tutorial anterior, al preparar Hyper-V para la evaluación y la migración. Debería haber configurado CredSSP para los hosts [manualmente](tutorial-prepare-hyper-v.md#enable-credssp-to-delegate-credentials) o mediante la [ejecución de un script](tutorial-prepare-hyper-v.md#run-the-script).

Habilite en el dispositivo de la manera siguiente:

#### <a name="option-1"></a>Opción 1

En la VM del dispositivo, ejecute este comando. HyperVHost1 y HyperVHost2 son nombres de host de ejemplo.

```
Enable-WSManCredSSP -Role Client -DelegateComputer HyperVHost1.contoso.com, HyperVHost2.contoso.com, HyperVHost1, HyperVHost2 -Force
```

Ejemplo: ` Enable-WSManCredSSP -Role Client -DelegateComputer HyperVHost1.contoso.com HyperVHost2.contoso.com -Force `

#### <a name="option-2"></a>Opción 2

También puede hacerlo en el Editor de directivas de grupo local en el dispositivo:

1. En **Directiva de equipo local** > **Configuración del equipo**, haga clic en **Plantillas administrativas** > **Sistema** > **Delegación de credenciales**.
2. Haga doble clic en **Permitir delegación de credenciales nuevas** y seleccione **Habilitado**.
3. En **Opciones**, haga clic en **Mostrar** y agregue cada host de Hyper-V que desee detectar en la lista, con **wsman/** como prefijo.
4. En **Delegación de credenciales**, haga doble clic en **Permitir la delegación de credenciales nuevas con autenticación solo NTLM de servidor**. De nuevo, agregue cada host de Hyper-V que quiera detectar en la lista, con **wsman/** como prefijo.

## <a name="start-continuous-discovery"></a>Inicio de detección continua

Conéctese desde el dispositivo a los hosts o clústeres de Hyper-V e inicie la detección de VM.

1. En **Paso 1: Proporcionar las credenciales del host de Hyper-V**, haga clic en **Agregar credenciales** para especificar un nombre descriptivo para las credenciales, agregue un **nombre de usuario** y una **contraseña** para el host o el clúster de Hyper-V que utilizará el dispositivo para detectar máquinas virtuales. Haga clic en **Guardar**.
1. Si quiere agregar varias credenciales a la vez, haga clic en **Agregar más** para guardar y agregar más credenciales. Se admiten varias credenciales para la detección de máquinas virtuales de Hyper-V.
1. En el **Paso 2: Proporcionar los detalles del host o el clúster de Hyper-V**, haga clic en **Agregar origen de detección** para especificar la **dirección IP o el FQDN** del host o el clúster de Hyper-V y el nombre descriptivo de las credenciales para conectarse al host o al clúster.
1. Puede **Agregar un solo elemento** cada vez o **Agregar varios elementos** de una sola vez. También hay una opción para proporcionar los detalles del host o el clúster de Hyper-V a través de **Importar CSV**.

    ![Selecciones para Agregar origen de detección](./media/tutorial-assess-hyper-v/add-discovery-source-hyperv.png)

    - Si elige **Agregar un solo elemento**, debe especificar el nombre descriptivo de las credenciales y la **dirección IP o el FQDN** del host o el clúster de Hyper-V y hacer clic en **Guardar**.
    - Si elige **Agregar varios elementos** _(opción seleccionada de manera predeterminada)_ , puede agregar varios registros a la vez mediante la especificación de la **dirección IP o el FQDN** del host o el clúster de Hyper-V con el nombre descriptivo de las credenciales en el cuadro de texto. **Compruebe** los registros agregados y haga clic en **Guardar**.
    - Si elige **Importar CSV**, puede descargar un archivo de plantilla CSV, rellenar el archivo con la **dirección IP o el FQDN** del host o el clúster de Hyper-V y el nombre descriptivo de las credenciales. A continuación, importe el archivo en el dispositivo, **compruebe** los registros del archivo y haga clic en **Guardar**.

1. Al hacer clic en Guardar, el dispositivo intentará validar la conexión a los hosts o clústeres agregados de Hyper-V y mostrar el **estado de validación** en la tabla en cada host o clúster.
    - En el caso de los hosts o clústeres validados correctamente, puede ver más detalles si hace clic en su dirección IP o FQDN.
    - Si se produce un error de validación para un host, haga clic en **Error en la validación** en la columna Estado de la tabla para revisar el error. Corrija el problema y vuelva a validar.
    - Para quitar hosts o clústeres, seleccione **Eliminar**.
    - No se puede quitar un host específico de un clúster. Solo puede quitar todo el clúster.
    - Puede agregar un clúster, incluso si hay problemas con hosts específicos del clúster.
1. Puede **volver a validar** la conectividad a los hosts o clústeres en cualquier momento antes de iniciar la detección.
1. Haga clic en **Iniciar detección** para dar comienzo a la detección de máquinas virtuales de los hosts o clústeres validados correctamente. Una vez que la detección se ha iniciado correctamente, puede comprobar el estado de detección en cada host o clúster de la tabla.

De esta forma comienza la detección. Los metadatos de los servidores detectados tardan alrededor de 2 minutos por host en aparecer en Azure Portal.

### <a name="verify-vms-in-the-portal"></a>Comprobación de VM en el portal

Una vez finalizada la detección, puede verificar que las VM aparezcan en el portal.

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Assessment**, haga clic en el icono que muestra el número de **servidores detectados**.

## <a name="set-up-an-assessment"></a>Configuración de una evaluación

Se pueden ejecutar dos tipos de evaluaciones mediante Azure Migrate Server Assessment.

**Valoración** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones basadas en los datos de rendimiento recopilados | **Tamaño de máquina virtual recomendado**: se basa en los datos de uso de la CPU y de la memoria.<br/><br/> **Tipo de disco recomendado (disco administrado estándar o Premium**): se basa en IOPS y en el rendimiento de los discos locales.
**Como local** | Evaluaciones que se basan en el tamaño local. | **Tamaño de máquina virtual recomendado**: se basa en el tamaño de la máquina virtual local<br/><br> **Tipo de disco recomendado**: se basa en el valor del tipo de almacenamiento que se selecciona para la evaluación.



### <a name="run-an-assessment"></a>Ejecución de una evaluación

Las evaluaciones se realizan como se indica a continuación:

1. Consulte los [procedimientos recomendados](best-practices-assessment.md) para crear evaluaciones.
2. En **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Evaluar**.

    ![Evaluar](./media/tutorial-assess-hyper-v/assess.png)

3. En **Evaluar los servidores**, especifique el nombre de la evaluación.
4. Haga clic en **View all** (Ver todo) para revisar la configuración de la valoración.

    ![Propiedades de la evaluación](./media/tutorial-assess-hyper-v/assessment-properties.png)

3. En **Seleccionar o crear un grupo**, seleccione **Crear nuevo** y especifique un nombre de grupo. Un grupo recopila una o varias máquinas virtuales para su evaluación.
4. En **Agregar máquinas al grupo**, seleccione las máquinas virtuales que se van a agregar al grupo.
5. Haga clic en **Crear evaluación** para crear el grupo y realizar la evaluación.

    ![Crear una evaluación](./media/tutorial-assess-hyper-v/assessment-create.png)

6. Una vez creada la evaluación, se puede ver en **Servidores** > **Azure Migrate: Server Assessment**.
7. Haga clic en **Exportar la evaluación** para descargarla como un archivo de Excel.


## <a name="review-an-assessment"></a>Revisión de una evaluación

Una evaluación describe:

- **Preparación para Azure**: si las máquinas virtuales son adecuadas para la migración a Azure.
- **Estimación del costo mensual**: los costos mensuales estimados de proceso y almacenamiento por ejecutar las máquinas virtuales en Azure.
- **Estimación del costo mensual de almacenamiento**: costos estimados del almacenamiento en disco después de la migración.


### <a name="view-an-assessment"></a>Visualización de una evaluación

1. En **Objetivos de migración** >  **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Evaluar**.
2. En **Evaluaciones**, haga clic en una evaluación para abrirla.

    ![Resumen de evaluaciones](./media/tutorial-assess-hyper-v/assessment-summary.png)


### <a name="review-azure-readiness"></a>Revisión de la preparación para Azure

1. En **Preparación para Azure**, compruebe si las máquinas virtuales están listas para su migración a Azure.
2. Revise el estado de la máquina virtual:
    - **Preparada para Azure**: Azure Migrate recomienda unas estimaciones de tamaño y costo de las máquinas virtuales en la evaluación.
    - **Preparado con condiciones**: muestra los problemas y las soluciones que se sugieren.
    - **No está preparado para Azure**: muestra los problemas y las soluciones que se sugieren.
    - **Preparación desconocida**: se usa cuando Azure Migrate no puede evaluar la preparación debido a problemas de disponibilidad de datos.

2. Haga clic en cualquiera de los estados de **Preparación para Azure**. Puede ver los detalles de la preparación de la máquina virtual y explorar en profundidad los detalles de esta, entre los que se incluye la configuración de proceso, almacenamiento y red.

### <a name="review-cost-details"></a>Revisión de los datos de costo

Esta vista muestra el costo estimado de almacenamiento y proceso que conlleva ejecutar máquinas virtuales en Azure.

1. Revise los costos mensuales de proceso y almacenamiento. Los costos se agregan para todas las máquinas virtuales del grupo evaluado.

    - Las estimaciones de costo se basan en las recomendaciones de tamaño de una máquina, así como en sus discos y propiedades.
    - Se muestran los costos mensuales estimados de proceso y almacenamiento.
    - La estimación de costos es para ejecutar las máquinas virtuales locales como máquinas virtuales IaaS. Azure Migrate Server Assessment no tiene en cuenta los costos de PaaS o SaaS.

2. Puede revisar las estimaciones del costo de almacenamiento mensual. Esta vista muestra los costos de almacenamiento agregados del grupo evaluado, divididos por los diferentes tipos de discos de almacenamiento.
3. Puede explorar en profundidad para ver los detalles de cada una de las máquinas virtuales.


### <a name="review-confidence-rating"></a>Examen de la clasificación de confianza

Cuando se realizan valoraciones basadas en el rendimiento, se asigna una clasificación de confianza a la evaluación.

![Clasificación de confianza](./media/tutorial-assess-hyper-v/confidence-rating.png)

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

Vaya al tercer tutorial de la serie, donde aprenderá a migrar máquinas virtuales de Hyper-V a Azure con Azure Migrate Server Migration.

> [!div class="nextstepaction"]
> [Migración de máquinas virtuales de Hyper-V](./tutorial-migrate-hyper-v.md)
