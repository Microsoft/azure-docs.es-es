---
title: Escalado de detección y evaluación mediante Azure Migrate | Microsoft Docs
description: Se describe cómo evaluar un número elevado de máquinas locales mediante el servicio Azure Migrate.
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: raynew
ms.openlocfilehash: fe86c758dbf05f91d53cb918b7794c12ab3f39bc
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518752"
---
# <a name="discover-and-assess-a-large-vmware-environment"></a>Detección y evaluación de un entorno grande de VMware

Azure Migrate tiene un límite de 1500 máquinas por proyecto. En este artículo se describe cómo evaluar un número elevado de máquinas virtuales (VM) locales mediante [Azure Migrate](migrate-overview.md).

> [!NOTE]
> Tenemos una versión preliminar disponible que permite la detección de hasta 10 000 máquinas virtuales de VMware en un solo proyecto con un solo dispositivo. Si está interesado en probarlo, regístrese [aquí.](https://aka.ms/migratefuture)

## <a name="prerequisites"></a>Requisitos previos

- **VMware**: Las VM que planea migrar deben administrarse mediante vCenter Server, versión 5.5, 6.0, 6.5 o 6.7. Además, necesita un host de ESXi que ejecute la versión 5.5 o posterior para implementar la máquina virtual del recopilador.
- **Cuenta de vCenter**: necesita una cuenta de solo lectura para acceder a vCenter Server. Azure Migrate usa esta cuenta para detectar las máquinas virtuales locales.
- **Permisos**: en vCenter Server necesitará permisos para crear una máquina virtual mediante la importación de un archivo en formato .OVA.
- **Configuración de las estadísticas**: este requisito solo es aplicable al [modelo de detección de una sola vez](https://docs.microsoft.com/azure/migrate/concepts-collector), que actualmente está en desuso. Para el modelo de detección de un solo uso, la configuración de las estadísticas de vCenter Server se debe establecer en el nivel 3 antes de empezar la implementación. El nivel estadístico se establecerá en 3 para cada uno de los intervalos de colección de día, semana y mes. Si el nivel es inferior a 3 en cualquiera de los tres intervalos de colección, la valoración funcionará, pero los datos de rendimiento de almacenamiento y red no se recopilarán. Las recomendaciones de tamaño se basarán entonces en los datos de rendimiento de CPU y memoria y en los datos de configuración de discos y adaptadores de red.

> [!NOTE]
> El dispositivo de detección de una sola vez está en desuso, ya que este método dependía de la configuración de las estadísticas de vCenter Server para la disponibilidad de punto de datos de rendimiento y la media recopilada de los contadores de rendimiento, lo que daba lugar a un cálculo de tamaño insuficiente de las máquinas virtuales para la migración a Azure.

### <a name="set-up-permissions"></a>Configuración de permisos

Azure Migrate necesita acceso a los servidores de VMware para detectar automáticamente las máquinas virtuales para su evaluación. La cuenta de VMware necesita los siguientes permisos:

- Tipo de usuario: Al menos un usuario de solo lectura
- Permisos: Objeto de centro de datos  –> Propagar al objeto secundario, rol = solo lectura
- Detalles: El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.
- Para restringir el acceso, asigne el rol Sin acceso con Propagar a objetos secundarios a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).

Si va a implementar en un entorno de varios inquilinos y le gustaría ámbito por carpeta de máquinas virtuales de un único inquilino, no se puede seleccionar la carpeta de la máquina virtual directamente al definir el ámbito de colección de Azure Migrate. Estos son instrucciones para la detección de ámbito por carpeta de máquinas virtuales:

1. Cree un usuario por inquilino y asignar permisos de solo lectura a todas las máquinas virtuales que pertenecen a un inquilino determinado. 
2. Conceda este acceso de usuario de solo lectura a todos los objetos de elemento primario donde se hospedan las máquinas virtuales. Todos los objetos primarios - host, la carpeta de hosts, el clúster, la carpeta de clústeres: en la jerarquía hasta el centro de datos son que se incluya. No es necesario propagar los permisos para todos los objetos secundarios.
3. Use las credenciales para la detección de seleccionar el centro de datos como *ámbito de la colección*. El RBAC configurar garantiza que el usuario correspondiente de vCenter tendrán acceso a máquinas virtuales solo específico del inquilino.

## <a name="plan-your-migration-projects-and-discoveries"></a>Planificación de los proyectos de migración y las detecciones

En función del número de máquinas virtuales que se vayan a detectar, puede crear varios proyectos e implementar varias aplicaciones en el entorno. Las aplicaciones pueden conectarse a una sola instancia de vCenter Server y a un único proyecto (a menos que se detenga la detección y se reinicie de cero).

En el caso de una detección de una sola vez (actualmente en desuso), funciona en un modelo de disparo y olvido; una vez realizada la detección, puede usar el mismo recopilador para recopilar datos de otra instancia de vCenter Server o enviarlos a otro proyecto de migración.

> [!NOTE]
> El dispositivo de detección de una sola vez está en desuso, ya que este método dependía de la configuración de las estadísticas de vCenter Server para la disponibilidad de punto de datos de rendimiento y la media recopilada de los contadores de rendimiento, lo que daba lugar a un cálculo de tamaño insuficiente de las máquinas virtuales para la migración a Azure. Se recomienda mover a la aplicación de detección continua.

Planee las detecciones y evaluaciones en función de los límites siguientes:

| **Entidad** | **Límite de máquinas** |
| ---------- | ----------------- |
| Proyecto    | 1500             |
| Detección  | 1500             |
| Evaluación | 1500             |

Tenga en cuenta estas consideraciones de planeación:

- Al realizar una detección mediante Azure Migrate Collector, puede establecer el ámbito de detección en una carpeta, un centro de datos, un clúster o un host de vCenter Server.
- Para hacer más de una detección desde la misma instancia de vCenter Server, verifique en vCenter Server si las máquinas virtuales que desea detectar están en carpetas, centros de datos, clústeres o hosts que admitan la limitación de 1500 máquinas.
- Se recomienda que, para los fines de evaluación, mantenga las máquinas con interdependencias dentro del mismo proyecto y de la misma evaluación. En vCenter Server, asegúrese de que las máquinas dependientes están en la misma carpeta, centro de datos o clúster para la evaluación.

Dependiendo del escenario, puede dividir las detecciones como se indica a continuación:

### <a name="multiple-vcenter-servers-with-less-than-1500-vms"></a>Varias instancias de vCenter Server con menos de 1500 máquinas virtuales
Si tiene varios servidores de vCenter en el entorno y el número total de máquinas virtuales es inferior a 1500, puede usar el enfoque siguiente en función del escenario:

**Detección continua:** en el caso de la detección continua, una aplicación solo puede conectarse a un único proyecto. Por tanto, tendrá que implementar un dispositivo para cada uno de los servidores vCenter y, después, crear un proyecto para cada dispositivo y desencadenar las detecciones en consecuencia.

**Detección de una sola vez (actualmente en desuso):** puede usar un único recopilador y un único proyecto de migración para detectar todas las máquinas virtuales en las instancias de vCenter Server. Como el recopilador de detección de una sola vez detecta los servidores vCenter de uno en uno, puede ejecutar el mismo recopilador en todos los servidores vCenter, uno detrás de otro, y apuntar el recopilador al mismo proyecto de migración. Después de completar todas las detecciones, puede crear las evaluaciones de las máquinas.


### <a name="multiple-vcenter-servers-with-more-than-1500-vms"></a>Varias instancias de vCenter Server con más de 1500 máquinas virtuales

Si tiene varias instancias de vCenter Server con menos de 1500 máquinas virtuales por vCenter Server, pero más de 1500 máquinas virtuales entre todas las instancias de vCenter Server, debe crear varios proyectos de migración (un proyecto de migración solamente puede contener 1500 máquinas virtuales). Puede conseguirlo creando un proyecto de migración por instancia de vCenter Server y dividir las detecciones.

**Detección continua:** tendrá que crear varias aplicaciones recopiladoras (una para cada instancia de vCenter Server) y conectar cada aplicación a un proyecto y desencadenar la detección según proceda.

**Detección de una sola vez (actualmente en desuso):** Puede usar un único recopilador para detectar cada instancia de vCenter Server (una detrás de otra). Si desea que las detecciones se inicien al mismo tiempo, también puede implementar varios dispositivos y ejecutar las detecciones en paralelo.

### <a name="more-than-1500-machines-in-a-single-vcenter-server"></a>Más de 1500 máquinas en una sola instancia de vCenter Server

Si tiene más de 1500 máquinas virtuales en una sola instancia de vCenter Server, debe dividir la detección en varios proyectos de migración. Para dividir las detecciones, puede aprovechar el campo de ámbito en el dispositivo y especifique el host, clúster, carpeta de hosts, la carpeta de clústeres o el centro de datos que desea detectar. Por ejemplo, si tiene dos carpetas en vCenter Server, una con 1000 máquinas virtuales (Carpeta1) y otra con 800 (Carpeta2), puede usar el campo de ámbito para dividir las detecciones entre esas carpetas.

**Detección continua:** en este caso, tendrá que crear dos aplicaciones recopiladoras; para la primera, especifique el ámbito como Carpeta1 y conéctela al primer proyecto de migración. Puede iniciar en paralelo la detección de Carpeta2 mediante el segundo dispositivo de recopilador y conectarlo al segundo proyecto de migración.

**Detección de una sola vez (actualmente en desuso):** puede usar el mismo recopilador para desencadenar ambas detecciones. En la primera detección, puede especificar Carpeta1 como el ámbito y apuntarla al primer proyecto de migración; una vez completada la primera detección, puede usar el mismo recopilador, cambiar su ámbito a Carpeta2 y los detalles del proyecto de migración al segundo proyecto de migración y realizar la segunda detección.

### <a name="multi-tenant-environment"></a>Entorno multiinquilino

Si tiene un entorno que se comparte entre los inquilinos y no desea detectar las máquinas virtuales de un inquilino en la suscripción de otro inquilino, puede usar el campo Ámbito en el dispositivo recopilador para definir el ámbito de la detección. Si los inquilinos comparten hosts, cree una credencial que tenga acceso de solo lectura solamente a las máquinas virtuales que pertenecen al inquilino específico y, luego, use esta credencial en el dispositivo recopilador y especifique el ámbito como el host para realizar la detección.

## <a name="discover-on-premises-environment"></a>Detectar el entorno local

Cuando esté listo con el plan, puede iniciar la detección de las máquinas virtuales locales:

### <a name="create-a-project"></a>Crear un proyecto

Cree un proyecto de Azure Migrate según sus necesidades:

1. En Azure Portal, haga clic en **Crear un recurso**.
2. Busque **Azure Migrate** y seleccione el servicio **Azure Migrate** en los resultados de búsqueda. Seleccione **Crear**.
3. Especifique un nombre de proyecto y la suscripción de Azure para el proyecto.
4. Cree un nuevo grupo de recursos.
5. Especifique la ubicación en la que desea crear el proyecto y seleccione **Crear**. Tenga en cuenta que todavía puede evaluar las máquinas virtuales para una ubicación de destino diferente. La ubicación especificada para el proyecto se utiliza para almacenar los metadatos que se recopilan a partir de máquinas virtuales locales.

### <a name="set-up-the-collector-appliance"></a>Configuración del dispositivo de recopilador

Azure Migrate crea una VM local conocida como el dispositivo de recopilador. Esta máquina virtual detecta máquinas virtuales de VMware locales y envía metadatos sobre ellas al servicio Azure Migrate. Para configurar la aplicación del recopilador, descargue un archivo OVA e impórtelo a la instancia de vCenter Server local.

#### <a name="download-the-collector-appliance"></a>Descarga del dispositivo de recopilador

Si tiene varios proyectos, tiene que descargar la aplicación del recopilador solo una vez en vCenter Server. Después de descargar y configurar la aplicación, ejecútela en cada proyecto y especifique el identificador de proyecto exclusivo y la clave.

1. En el proyecto de Azure Migrate, haga clic en **Introducción** > **Detectar y evaluar** > **Detectar máquinas**.
2. En **Detectar máquinas**, haga clic en **Descargar**, para descargar la aplicación.

    El dispositivo de Azure Migrate se comunica con vCenter Server Detección y genera perfiles continuamente del entorno local con el fin de recopilar datos de uso en tiempo real para cada máquina virtual. Recopila contadores de número máximo de cada métrica (uso de la CPU, uso de memoria etc.). Este modelo no depende de la configuración de las estadísticas de vCenter Server para la recopilación de datos de rendimiento. Puede detener la generación de perfiles continua en cualquier momento desde el dispositivo.

    > [!NOTE]
    > El dispositivo de detección de una sola vez está en desuso, ya que este método dependía de la configuración de las estadísticas de vCenter Server para la disponibilidad de punto de datos de rendimiento y la media recopilada de los contadores de rendimiento, lo que daba lugar a un cálculo de tamaño insuficiente de las máquinas virtuales para la migración a Azure.

    **Gratificación instantánea:** con el dispositivo de detección continua, una vez que la detección se ha completado (tarda un par de horas según el número de máquinas virtuales), podrá crear valoraciones de forma instantánea. Puesto que la recopilación de datos de rendimiento se inicia al iniciar la detección, si desea obtener gratificación instantánea, para el criterio de tamaño en la valoración debe seleccionar *como local*. Para obtener valoraciones basadas en el rendimiento, se recomienda esperar al menos un día después de iniciar la detección para obtener recomendaciones de tamaño que sean confiables.

    Tenga en cuenta que el dispositivo solo recopila datos de rendimiento de forma continua, no detecta ningún cambio de configuración en el entorno local (por ejemplo, adición de máquina virtual, eliminación o adición de disco, entre otros). Si se produce un cambio de configuración en el entorno local, puede hacer lo siguiente para reflejar los cambios en el portal:

    - Adición de elementos (máquinas virtuales, discos, núcleos, etc.): para reflejar estos cambios en Azure Portal, puede detener la detección desde la aplicación y después iniciarla de nuevo. Así se asegurará de que los cambios se actualizan en el proyecto de Azure Migrate.

    - Eliminación de máquinas virtuales: debido a la forma en que está diseñada la aplicación, la eliminación de las máquinas virtuales no se refleja aunque detenga e inicie la detección. Esto se debe a que los datos de las detecciones posteriores se agregan a las detecciones más antiguas y no se reemplazan. En este caso, puede simplemente omitir la máquina virtual en el portal quitándola del grupo y recalculando la valoración.

3. En **Copiar las credenciales del proyecto**, copie la clave y el identificador del proyecto. Los necesitará cuando configure el recopilador.


#### <a name="verify-the-collector-appliance"></a>Comprobación del dispositivo de recopilador

Compruebe que el archivo OVA es seguro, antes de implementarlo:

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.

2. Ejecute el siguiente comando para generar el código hash para el archivo OVA:

   ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```

   Ejemplo de uso: ```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.ova SHA256```

3. Asegúrese de que el código hash generado coincida con la configuración siguiente.

#### <a name="continuous-discovery"></a>Detección continua

Para la versión 1.0.10.4 de OVA

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | 2ca5b1b93ee0675ca794dd3fd216e13d
SHA1 | 8c46a52b18d36e91daeae62f412f5cb2a8198ee5
SHA256 | 3b3dec0f995b3dd3c6ba218d436be003a687710abab9fcd17d4bdc90a11276be

#### <a name="one-time-discovery-deprecated-now"></a>Detección de una sola vez (en desuso)

Para la versión 1.0.9.15 de OVA (publicada el 23/10/2018)

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | e9ef16b0c837638c506b5fc0ef75ebfa
SHA1 | 37b4b1e92b3c6ac2782ff5258450df6686c89864
SHA256 | 8a86fc17f69b69968eb20a5c4c288c194cdcffb4ee6568d85ae5ba96835559ba

Para la versión 1.0.9.14 de OVA (publicada el 24/8/2018)

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | 6d8446c0eeba3de3ecc9bc3713f9c8bd
SHA1 | e9f5bdfdd1a746c11910ed917511b5d91b9f939f
SHA256 | 7f7636d0959379502dfbda19b8e3f47f3a4744ee9453fc9ce548e6682a66f13c

Para la versión 1.0.9.12 de OVA

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | d0363e5d1b377a8eb08843cf034ac28a
SHA1 | df4a0ada64bfa59c37acf521d15dcabe7f3f716b
SHA256 | f677b6c255e3d4d529315a31b5947edfe46f45e4eb4dbc8019d68d1d1b337c2e

Para la versión 1.0.9.8 de OVA

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | b5d9f0caf15ca357ac0563468c2e6251
SHA1 | d6179b5bfe84e123fabd37f8a1e4930839eeb0e5
SHA256 | 09c68b168719cb93bd439ea6a5fe21a3b01beec0e15b84204857061ca5b116ff

Para la versión 1.0.9.7 de OVA

**Algoritmo** | **Valor del código hash**
--- | ---
MD5 | d5b6a03701203ff556fa78694d6d7c35
SHA1 | f039feaa10dccd811c3d22d9a59fb83d0b01151e
SHA256 | e5e997c003e29036f62bf3fdce96acd4a271799211a84b34b35dfd290e9bea9c

### <a name="create-the-collector-vm"></a>Creación de la VM de recopilador

Importe el archivo descargado a vCenter Server:

1. En la consola de cliente de vSphere, seleccione **File** (Archivo) > **Deploy OVF Template** (Implementar plantilla de OVF).

    ![Implementación de OVF](./media/how-to-scale-assessment/vcenter-wizard.png)

2. En el Deploy OVF Template Wizard (Asistente para implementar la plantilla de OVF) > **Source** (Origen), especifique la ubicación del archivo OVA.
3. En **Name** (Nombre) y **Location** (Ubicación), especifique un nombre descriptivo para la VM de recopilador y el objeto de inventario en el que se hospedará la VM.
4. En **Host/Cluster** (Host o clúster), especifique el host o clúster donde se ejecutará el recopilador de máquina virtual.
5. En el almacenamiento, especifique el destino de almacenamiento para la VM de recopilador.
6. En **Disk Format** (Formato de disco), especifique el tamaño y el tipo de disco.
7. En **Network Mapping** (Asignación de red), especifique la red a la que se conectará la VM de recopilador. La red necesita conectividad a Internet, para enviar metadatos a Azure.
8. Revise y confirme la configuración y, a continuación, seleccione **Finish** (Finalizar).

### <a name="identify-the-id-and-key-for-each-project"></a>Identificación del identificador y de la clave de cada proyecto

Si tiene varios proyectos, asegúrese de identificar el identificador y la clave de cada uno. Necesita la clave al ejecutar el recopilador para detectar las máquinas virtuales.

1. En el proyecto, seleccione **Introducción** > **Detectar y evaluar** > **Detectar máquinas**.
2. En **Copiar las credenciales del proyecto**, copie la clave y el identificador del proyecto.
    ![Copia de las credenciales del proyecto](./media/how-to-scale-assessment/copy-project-credentials.png)

### <a name="run-the-collector-to-discover-vms"></a>Ejecución del recopilador para detectar VM

Para cada detección que necesite realizar, ejecute el recopilador para detectar máquinas virtuales en el ámbito requerido. Ejecute las detecciones una detrás de otra. No se admiten las detecciones simultáneas, y cada detección debe tener un ámbito distinto.

1.  En la consola del cliente de vSphere, haga clic con el botón derecho en VM > **Open Console** (Abrir consola).
2.  Proporcione el idioma, la zona horaria y las preferencias de contraseña para el dispositivo.
3.  En el escritorio, seleccione el acceso directo **Run collector** (Ejecutar recopilador).
4.  En Azure Migrate Collector, abra **Set Up Prerequisites** (Configurar requisitos previos) y luego:

     a. Acepte los términos de licencia y lea la información de terceros.

    El recopilador comprueba que la VM tenga acceso a Internet.

    b. Si la máquina virtual tiene acceso a Internet a través de un proxy, seleccione **Configuración de proxy** y especifique el puerto de escucha y la dirección del proxy. Especifique las credenciales si el proxy requiere autenticación.

    El recopilador comprueba que el servicio del recopilador se está ejecutando. El servicio se instala de forma predeterminada en la VM de recopilador.

    c. Descargue e instale VMware PowerCLI.

5.  En **Specify vCenter Server details** (Especificar detalles de vCenter Server), haga lo siguiente:
    - Especifique el nombre (FQDN) o la dirección IP de vCenter Server.
    - En **Nombre de usuario** y **Contraseña**, especifique las credenciales de cuenta de solo lectura que utilizará el recopilador para detectar las máquinas virtuales en vCenter Server.
    - En **Seleccionar ámbito**, especifique un ámbito para la detección de máquinas virtuales. El recopilador solo puede detectar máquinas virtuales dentro del ámbito especificado. El ámbito se puede establecer en una carpeta, centro de datos o clúster específico. No debe contener más de 1000 máquinas virtuales.

6.  En **Specify migration project** (Especificar proyecto de migración), especifique el identificador y la clave del proyecto. Si no los copió, abra Azure Portal desde la máquina virtual de recopilador. En la página **Introducción** del proyecto, seleccione **Detectar máquinas** y copie los valores.  
7.  En **View collection progress** (Ver progreso de recopilación), supervise el proceso de detección y compruebe que los metadatos recopilados de las máquinas virtuales se encuentran dentro del ámbito. El recopilador proporciona un tiempo de detección aproximado.

#### <a name="verify-vms-in-the-portal"></a>Comprobación de VM en el portal

El recopilador generará perfiles del entorno local de forma continuada y seguirá enviando los datos de rendimiento en intervalos de una hora. Puede revisar las máquinas en el portal una hora después de que se inicie la detección. Se recomienda encarecidamente esperar al menos un día antes de crear evaluaciones basadas en el rendimiento para las máquinas virtuales.

1. En el proyecto de migración, haga clic en **Administrar** > **Máquinas**.
2. Compruebe que las VM que desea detectar aparecen en el portal.

### <a name="data-collected-from-on-premises-environment"></a>Datos recopilados del entorno local

La aplicación recopiladora detecta los siguientes datos de configuración sobre las máquinas virtuales seleccionadas.

1. Nombre para mostrar de la máquina virtual (en vCenter)
2. Ruta de acceso de inventario de la máquina virtual (host o carpeta en vCenter)
3. Dirección IP
4. Dirección MAC
5. Sistema operativo
5. Número de núcleos, discos, NIC
6. Tamaño de memoria, tamaños de disco
7. Y los contadores de rendimiento de la máquina virtual, el disco y la red se muestran en la tabla siguiente.

La aplicación recopiladora recopila los siguientes contadores de rendimiento para cada máquina virtual desde el host ESXi en intervalos de 20 segundos. Estos son los contadores de vCenter y, aunque la terminología indica el promedio, los ejemplos de 20 segundos son contadores en tiempo real. Después, el dispositivo acumula los ejemplos de 20 segundos para crear un único punto de datos para cada 15 minutos mediante la selección del valor máximo de los ejemplos de 20 segundos y lo envía a Azure. Los datos de rendimiento para las máquinas virtuales comienzan a estar disponibles en el portal dos horas después de haberse iniciado la detección. Se recomienda encarecidamente esperar al menos un día antes de crear evaluaciones basadas en el rendimiento para obtener recomendaciones de tamaño precisas. Si busca una gratificación instantánea, puede crear evaluaciones con criterio de tamaño, por ejemplo *como local*, que no tendrá en cuenta los datos de rendimiento para el ajuste del tamaño.

**Contador** |  **Impacto en la evaluación**
--- | ---
cpu.usage.average | Costo y tamaño de VM recomendados  
mem.usage.average | Costo y tamaño de VM recomendados  
virtualDisk.read.average | Calcula el tamaño del disco, el costo de almacenamiento y el tamaño de máquina virtual
virtualDisk.write.average | Calcula el tamaño del disco, el costo de almacenamiento y el tamaño de máquina virtual
virtualDisk.numberReadAveraged.average | Calcula el tamaño del disco, el costo de almacenamiento y el tamaño de máquina virtual
virtualDisk.numberWriteAveraged.average | Calcula el tamaño del disco, el costo de almacenamiento y el tamaño de máquina virtual
net.received.average | Calcula el tamaño de la máquina virtual                          
net.transmitted.average | Calcula el tamaño de la máquina virtual     

> [!WARNING]
> El método de detección de una sola vez basado en la configuración de estadísticas de vCenter Server para la recopilación de datos de rendimiento se encuentra ya en desuso.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [crear un grupo](how-to-create-a-group.md) para evaluación.
- [Obtenga más información](concepts-assessment-calculation.md) sobre cómo se calculan las evaluaciones.
