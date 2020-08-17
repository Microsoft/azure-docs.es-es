---
title: Configuración de un laboratorio para administración y desarrollo con Azure SQL Database | Azure Lab Services
description: Aprenda a configurar un laboratorio para administración y desarrollo con Azure SQL Database.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 9fc0a965869207ba8d1b4eb6f45e878ae4b93c3a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079029"
---
# <a name="set-up-a-lab-to-manage-and-develop-with-sql-server"></a>Configuración de un laboratorio para administración y desarrollo con SQL Server

En este artículo se describe cómo configurar un laboratorio para una clase de administración y desarrollo de bases de datos básica de SQL Server en Azure Lab Services.  Los conceptos de base de datos son uno de los cursos introductorios en la mayoría de los departamentos de informática de las universidades. El lenguaje de consulta estructurado (SQL) es un estándar internacional.  SQL es el lenguaje estándar para la administración de bases de datos de relaciones, como la adición, el acceso y la administración de contenido en una base de datos.  Se destaca por su procesamiento rápido, su confiabilidad comprobada, su facilidad y flexibilidad de uso.

En este artículo, le mostraremos cómo configurar una plantilla de máquina virtual en un laboratorio con [Visual Studio 2019](https://visualstudio.microsoft.com/vs/), [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) y [Azure Data Studio](https://github.com/microsoft/azuredatastudio).  Para este laboratorio, usaremos una [base de datos de SQL Server](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview) compartida para todo el laboratorio. [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview) es una oferta de motor de base de datos de plataforma como servicio (PaaS) de Azure.

## <a name="lab-configuration"></a>Configuración del laboratorio

Para configurar este laboratorio, para empezar, necesita una suscripción a Azure y una cuenta de laboratorio. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Una vez que obtenga una suscripción a Azure, puede crear una nueva cuenta de laboratorio en Azure Lab Services. Para obtener más información sobre la creación de una nueva cuenta de laboratorio, consulte el [tutorial para configurar una cuenta de laboratorio](https://docs.microsoft.com/azure/lab-services/classroom-labs/tutorial-setup-lab-account). También puede usar una cuenta de laboratorio existente.

### <a name="lab-account-settings"></a>Configuración de la cuenta de laboratorio

Habilite la configuración que se describe en la tabla siguiente para la cuenta de laboratorio. Para obtener más información sobre cómo habilitar imágenes de Marketplace, consulte [Especificación de las imágenes de Marketplace disponibles para los creadores de laboratorios](specify-marketplace-images.md).

| Configuración de la cuenta de laboratorio | Instructions |
| ------------------- | ------------ |
| Imagen de Marketplace | Habilite la imagen "Visual Studio 2019 Community (versión más reciente) en Windows 10 Enterprise N (x64)" para usarla en su cuenta de laboratorio. |

### <a name="shared-resource-configuration"></a>Configuración de recursos compartidos

Para usar un recurso compartido en Lab Services, primero tiene que crear la red virtual y los propios recursos.  Para crear la red virtual y conectarla al laboratorio, siga las instrucciones de [Creación de un laboratorio con un recurso compartido en Azure Lab Services](how-to-create-a-lab-with-shared-resource.md).  Recuerde que todos los recursos externos a Lab Services se facturarán por separado y no se incluirán en las estimaciones de costos del laboratorio.

>[!WARNING]
>Los recursos compartidos de un laboratorio deben configurarse antes de crear el laboratorio.  Si la red virtual no está [emparejada con la cuenta de laboratorio](how-to-connect-peer-virtual-network.md) *antes* de que se cree el laboratorio, este no tendrá acceso al recurso compartido.

Ahora que el lado de la red está controlado, vamos a crear una base de datos de SQL Server.  Crearemos una [base de datos única](https://docs.microsoft.com/azure/sql-database/sql-database-single-database-get-started?tabs=azure-portal), ya que es la opción de implementación más rápida para Azure SQL Database.  Para otras opciones de implementación, cree un [grupo elástico](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-pool#creating-a-new-sql-database-elastic-pool-using-the-azure-portal), una [instancia administrada](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started) o una [máquina virtual SQL](https://docs.microsoft.com/azure/virtual-machines/windows/sql/quickstart-sql-vm-create-portal).

1. En el menú de Azure Portal, elija **Crear recurso**.
2. Seleccione **SQL Database** y haga clic en el botón **Crear**.
3. En la pestaña **Básica** del formulario **Crear una base de datos SQL**, seleccione el grupo de recursos para la base de datos.  Usaremos *sqldb-rg*.
4. En **Nombre de la base de datos de destino**, escriba *classlabdb*.
5. En el valor **Servidor**, haga clic en **Crear nuevo** para crear un nuevo servidor que contenga la base de datos.
6. En el control flotante **Nuevo servidor**, escriba el nombre del servidor.  Usaremos *classlabdbserver*.  Este nombre tiene ser único a nivel global.
7. Escriba *azureuser* en **Inicio de sesión de administrador del servidor**.
8. Escriba una contraseña que sea sencilla de recordar.  La contraseña tiene que tener por lo menos ocho caracteres y contener caracteres especiales.
9. Elija región para la **ubicación**.  Si es posible, escriba la misma ubicación de la cuenta de laboratorio y la red virtual emparejada, para minimizar la latencia.
10. Haga clic en **Aceptar** para volver al formulario **Crear base de datos SQL**.
11. Haga clic en el vínculo **Configurar la base de datos** en la opción de configuración **Proceso y almacenamiento**.
12. Modifique los valores de configuración de la base de datos según sea necesario para la clase.  Puede elegir entre las opciones aprovisionadas y sin servidor.  En este ejemplo, usaremos la opción sin servidor de escalabilidad automática, con un máximo de 4 núcleos virtuales y un mínimo de 1 núcleo virtual. Conservaremos el valor de pausa automática en un mínimo de 1 hora. Haga clic en **Aplicar**.
13. Haga clic en **Siguiente: Redes**.
14. En la pestaña Redes, elija como **Método de conectividad** el valor Punto de conexión privado.
15. En la sección **Puntos de conexión privados**, haga clic en **Agregar punto de conexión privado**.
16. En el control flotante **Crear un punto de conexión privado**, elija el mismo grupo de recursos que la red virtual emparejada con la cuenta de laboratorio.
17. Para **Ubicación**, seleccione la misma ubicación que la red virtual.
18. En **Nombre**, escriba *labsql-endpt*.
19. Deje el conjunto de subrecurso de destino establecido en SqlServer.
20. En **Red virtual**, elija la misma red virtual que está emparejada con la cuenta de laboratorio.
21. En **Subred**, seleccione la subred en la que desea que se hospede el punto de conexión.  La dirección IP asignada al punto de conexión estará dentro del intervalo asignado a esa subred.
22. Establezca **Integrar con la zona DNS privada** en **No**. Para simplificar, usaremos Azure DNS a través de nuestra zona DNS privada o de nuestros propios servidores DNS.
23. Haga clic en **OK**.
24. Haga clic en **Siguiente: Configuración adicional**.
25. En la opción **Usar datos existentes**, elija **Muestra**.  Al crear la base de datos se utilizarán los datos de AdventureWorksLT.
26. Haga clic en **Revisar + crear**.
27. Haga clic en **Crear**.

Una vez que se complete correctamente la implementación de SQL Database, podemos crear el laboratorio e instalar el software en la máquina de plantilla del laboratorio.

### <a name="lab-settings"></a>Configuración del laboratorio

Use las opciones de la tabla siguiente al configurar un laboratorio educativo. Para obtener más información sobre cómo crear un laboratorio de clase, consulte el [tutorial Configuración de un laboratorio educativo](tutorial-setup-classroom-lab.md).

| Configuración del laboratorio | Valor/instrucciones |
| ------------ | ------------------ |
| Tamaño de la máquina virtual | Mediana. Este tamaño es el más adecuado para bases de datos relacionales, análisis y almacenamiento en caché en memoria. |
| Imagen de máquina virtual | Visual Studio 2019 Community (versión más reciente) en Windows 10 Enterprise N (x64) |

Ahora que ya hemos creado el laboratorio, vamos a modificar la máquina de la plantilla con el software que necesitamos.

## <a name="visual-studio"></a>Visual Studio

La imagen elegida anteriormente incluye [Visual Studio 2019 Community](https://visualstudio.microsoft.com/vs/community/).  Todas las cargas de trabajo y los conjuntos de herramientas ya están instalados en la imagen.  Utilice el Instalador de Visual Studio para [instalar las herramientas opcionales](https://docs.microsoft.com/visualstudio/install/modify-visual-studio?view=vs-2019) que desee.  [Inicie sesión en Visual Studio](https://docs.microsoft.com/visualstudio/ide/signing-in-to-visual-studio?view=vs-2019#how-to-sign-in-to-visual-studio) para desbloquear la edición Community.

Visual Studio contiene el conjunto de herramientas **Almacenamiento y procesamiento de datos**, que incluye SQL Server Data Tools (SSDT).  Para más información sobre las funcionalidades de SSDT, consulte [Información general sobre SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/sql-server-data-tools?view=sql-server-ver15).  Para comprobar que la conexión de la clase a la instancia de SQL Server compartida se realizará sin problemas, consulte [cómo conectar con una base de datos y examinar objetos existentes](https://docs.microsoft.com/sql/ssdt/how-to-connect-to-a-database-and-browse-existing-objects?view=sql-server-ver15). Si se le solicita, agregue la dirección IP de la máquina de la plantilla a la [lista de equipos permitidos](https://docs.microsoft.com/azure/azure-sql/database/firewall-configure) que pueden conectarse a la instancia de SQL Server.

Visual Studio admite varias cargas de trabajo, entre ellas **Web y nube** y **Dispositivos móviles y de escritorio**.  Estas dos cargas de trabajo son compatibles con SQL Server como origen de datos. Para más información sobre el uso de ASP.NET Core para SQL Server, consulte el tutorial [Compilación de una aplicación ASP.NET Core y SQL Database en Azure App Service](https://docs.microsoft.com/azure/app-service/tutorial-dotnetcore-sqldb-app).  Use la biblioteca [System.Data.SqlClient](https://docs.microsoft.com/dotnet/api/system.data.sqlclient) para conectarse a una base de datos de SQL Database desde una aplicación de [Xamarin](https://docs.microsoft.com/xamarin).

## <a name="install-azure-data-studio"></a>Instalación de Azure Data Studio

[Azure Data Studio](https://github.com/microsoft/azuredatastudio) es un nuevo entorno de escritorio multiplataforma para varias bases de datos para profesionales de datos que usan la familia de plataformas de datos locales y en la nube en Windows, macOS y Linux.

1. Descargue el [instalador del *sistema* de Azure Data Studio para Windows](https://go.microsoft.com/fwlink/?linkid=2127432). Para buscar instaladores para otros sistemas operativos compatibles, vaya a la página de descarga de [Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/download).
2. En la página **License Agreement** (Contrato de licencia), seleccione **I accept the agreement** (Acepto el contrato). Haga clic en **Next**.
3. En la página **Select Destination Location** (Seleccionar ubicación de destino), haga clic en **Next** (Siguiente).
4. En la página **Select Start Menu Folder** (Seleccionar carpeta del menú Inicio), haga clic en **Next** (Siguiente).
5. En la página **Select Additional Tasks** (Seleccionar tareas adicionales), active la casilla **Create a desktop icon** (Crear un icono en el escritorio) si desea tener un icono en el escritorio.  Haga clic en **Next**.
6. En **Ready to Install** (Preparado para instalar), haga clic en **Next** (Siguiente).
7. Espere a que se ejecute el instalador.  Haga clic en **Finalizar**

Ahora que hemos instalado Azure Data Studio, vamos a configurar la conexión a Azure SQL Database.

1. En la página de **presentación** de Azure Data Studio, haga clic en el vínculo **New Connection** (Nueva conexión).
2. En el cuadro **Connection Details** (Detalles de conexión), rellene la información necesaria.
    - Establezca **Server** (Servidor) en *classlabdbserver.database.windows.net*.
    - Establezca **User** (Usuario) en *azureuser*.
    - Establezca **Password** (Contraseña) en la contraseña que se usó para crear la base de datos.
    - Active **Remember Password** (Recordar contraseña).
    - En **Database** (Base de datos), seleccione *classlabdb*.
3. Haga clic en **Conectar**.

## <a name="install-sql-server-management-studio"></a>Instalación de SQL Server Management Studio

[SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) es un entorno integrado para administrar cualquier infraestructura de SQL.  SSMS es una herramienta que los administradores de bases de datos utilizan para implementar, supervisar y actualizar la infraestructura de datos.

1. [Descargue SQL Server Management Studio](https://aka.ms/ssmsfullsetup). Una vez descargado, inicie el instalador.
2. En la página de **presentación**, haga clic en **Install** (Instalar).
3. En la página **Setup Completed** (Instalación completada), haga clic en **Close** (Cerrar).
4. Inicie SQL Server Management Studio.  
5. En la página **Dependency Configuration process** (Proceso de configuración de dependencias), haga clic en **Close** (Cerrar).

Ahora que está instalado SSMS, puede [conectarse a una instancia de SQL Server y realizar consultas](https://docs.microsoft.com/sql/ssms/tutorials/connect-query-sql-server). Al configurar la conexión, use los valores siguientes:

- Tipo de servidor: Motor de base de datos
- Nombre del servidor: *classlabdbserver.database.windows.net*
- Autenticación: Autenticación de SQL Server
- Inicio de sesión: *azureuser*
- Contraseña: la utilizada para crear la base de datos.

## <a name="cost-estimate"></a>Estimación del costo

Vamos a cubrir una posible estimación de costos para esta clase. Esta estimación no incluye el costo de ejecutar SQL Server.  Consulte [Precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database) para obtener información detallada y actualizada sobre los precios de las bases de datos.

Usaremos una clase de 25 alumnos. Hay 20 horas de tiempo de clase programado. Además, cada alumno obtiene una cuota de 10 horas para deberes o tareas fuera del tiempo de clase programado. El tamaño de la máquina virtual que elegimos es el medio, que tiene 42 unidades de laboratorio.

A continuación se incluye un ejemplo de una posible estimación del costo para esta clase:

25 alumnos \* (20 horas programadas \+ 10 horas de cuota) *0,42 USD por hora = 315,00 USD

>[!IMPORTANT]
>La estimación de costos solo se utiliza con fines de ejemplo. Para conocer los detalles actuales sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="next-steps"></a>Pasos siguientes

Los pasos siguientes son comunes a la configuración de cualquier laboratorio.

- [Creación, administración y publicación de una plantilla](how-to-create-manage-template.md)
- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users)
