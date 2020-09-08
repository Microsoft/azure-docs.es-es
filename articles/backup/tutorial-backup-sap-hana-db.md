---
title: 'Tutorial: Copia de seguridad de bases de datos de SAP HANA en máquinas virtuales de Azure'
description: En este tutorial, aprenderá a hacer una copia de seguridad de una base de datos de SAP HANA que se ejecuta en una máquina virtual de Azure en un almacén de Azure Backup Recovery Services.
ms.topic: tutorial
ms.date: 02/24/2020
ms.openlocfilehash: b43fd5c432b06902de0a898fc4bb0f114143b3ba
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375285"
---
# <a name="tutorial-back-up-sap-hana-databases-in-an-azure-vm"></a>Tutorial: Copia de seguridad de bases de datos de SAP HANA en una máquina virtual de Azure

Este tutorial muestra cómo realizar una copia de seguridad de una base de datos de SAP HANA que se ejecuta en máquinas virtuales de Azure en un almacén de Azure Backup Recovery Services. En este artículo, aprenderá a:

> [!div class="checklist"]
>
> * Crear y configurar un almacén.
> * Detectar bases de datos.
> * Configuración de copias de seguridad

[Aquí](sap-hana-backup-support-matrix.md#scenario-support) están todos los escenarios que se admiten actualmente.

>[!NOTE]
>A partir del 1 de agosto de 2020, la copia de seguridad de SAP HANA para RHEL (7.4, 7.6, 7.7 y 8.1) está disponible con carácter general.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de seguir estos pasos antes de configurar copias de seguridad:

* Identifique o cree un [almacén de Recovery Services](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) en la misma región y suscripción que la máquina virtual que ejecuta SAP HANA.
* Permita la conectividad desde la máquina virtual a Internet para que pueda acceder a Azure, tal y como se describe en el siguiente procedimiento [Configuración de la conectividad de red](#set-up-network-connectivity).
* Asegúrese de que la longitud combinada del nombre de la máquina virtual del servidor de SAP HANA y el nombre del grupo de recursos no supere los 84 caracteres para Azure Resource Manager (máquinas virtuales de ARM) y 77 caracteres para máquinas virtuales clásicas. Esta limitación se debe a que algunos caracteres están reservados por el servicio.
* Debe existir una clave en **hdbuserstore** que cumpla los siguientes criterios:
  * Debe estar presente en el elemento **hdbuserstore** predeterminado. El valor predeterminado es la cuenta `<sid>adm` en la que SAP HANA está instalado.
  * En el caso de MDC, la clave debe apuntar al puerto SQL de **NAMESERVER**. En el caso de SDC, debe apuntar al puerto SQL de **INDEXSERVER**.
  * Debe tener credenciales para agregar y eliminar usuarios.
  * Tenga en cuenta que esta clave se puede eliminar después de que el script de registro previo correctamente se registre correctamente.
* Ejecute el script de configuración de la copia de seguridad de SAP HANA (script de registro previo) en la máquina virtual donde está instalado HANA como usuario raíz. [Este script](https://aka.ms/scriptforpermsonhana) prepara el sistema HANA para la copia de seguridad. Consulte la sección [Qué hace el script de registro previo](#what-the-pre-registration-script-does) para comprender mejor el script de registro previo.
* Si la configuración de HANA usa puntos de conexión privados, ejecute el [script de registro previo](https://aka.ms/scriptforpermsonhana) con el parámetro *-sn* o *--skip-network-checks*.

>[!NOTE]
>El script de registro previo instala **compat-unixODBC234** para las cargas de trabajo de SAP HANA que se ejecutan en RHEL (7.4, 7.6 y 7.7) y **unixODBC** para RHEL 8.1. [Este paquete se encuentra en el repositorio de Update Services de RHEL for SAP HANA (para RHEL 7 Server) para SAP Solutions (RPM)](https://access.redhat.com/solutions/5094721).  Para una imagen de RHEL de Azure Marketplace, el repositorio sería **rhui-rhel-sap-hana-for-rhel-7-server-rhui-e4s-rpms**.

## <a name="set-up-network-connectivity"></a>Configurar la conectividad de red

En todas las operaciones, una base de datos SAP HANA que se ejecuta en una máquina virtual de Azure necesita conectividad con el servicio Azure Backup, Azure Storage y Azure Active Directory. Esto puede lograrse mediante puntos de conexión privados o si se permite el acceso a las direcciones IP públicas o FQDN necesarios. No permitir la conectividad adecuada con los servicios de Azure necesarios puede provocar errores en operaciones como la detección de bases de datos, la configuración de copias de seguridad, la realización de copias de seguridad y la restauración de datos.

En la tabla siguiente se enumeran las distintas alternativas que se pueden usar para establecer la conectividad:

| **Opción**                        | **Ventajas**                                               | **Desventajas**                                            |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Puntos de conexión privados                 | Permite copias de seguridad a través de direcciones IP privadas dentro de la red virtual  <br><br>   Proporciona un control pormenorizado de la red y el almacén | Incurre en [costos](https://azure.microsoft.com/pricing/details/private-link/) estándar de punto de conexión privado |
| Etiquetas de servicio de NSG                  | Más fácil de administrar ya que los cambios de intervalo se combinan automáticamente   <br><br>   Sin costos adicionales. | Solo se puede usar con grupos de seguridad de red.  <br><br>    Proporciona acceso a todo el servicio. |
| Etiquetas de FQDN de Azure Firewall          | Más facilidad de administración, ya que los FQDN necesarios se administran automáticamente | Solo se puede usar con Azure Firewall.                         |
| Permite el acceso a los FQDN/IP del servicio | Sin costos adicionales.   <br><br>  Funciona con todos los firewalls y dispositivos de seguridad de red | Es posible que sea necesario acceder a un amplio conjunto de direcciones IP o FQDN   |
| Usar un servidor proxy HTTP                 | Un único punto de acceso a Internet para las máquinas virtuales.                       | Costos adicionales de ejecutar una máquina virtual con el software de proxy.         |

A continuación se proporcionan más detalles sobre el uso de estas opciones:

### <a name="private-endpoints"></a>Puntos de conexión privados

Los puntos de conexión privados permiten conectar de forma segura desde los servidores de una red virtual al almacén de Recovery Services. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual del almacén. El tráfico de red entre los recursos dentro de la red virtual y el almacén viaja a través de la red virtual y un vínculo privado en la red troncal de Microsoft. Esto elimina la exposición de la red pública de Internet. Obtenga más información sobre los puntos de conexión privados para Azure Backup [aquí](./private-endpoints.md).

### <a name="nsg-tags"></a>Etiquetas de NSG

Si emplea grupos de seguridad de red (NSG), use la etiqueta de servicio de *AzureBackup* para permitir el acceso de salida a Azure Backup. Además de la etiqueta de Azure Backup, también necesita permitir la conectividad para la autenticación y la transferencia de datos mediante la creación de [reglas de NSG](../virtual-network/security-overview.md#service-tags) similares para *Azure AD* y *Azure Storage*.  En los pasos siguientes se describe el proceso para crear una regla para la etiqueta de Azure Backup:

1. En **Todos los servicios**, vaya a **Grupos de seguridad de red** y seleccione el grupo de seguridad de red.

1. En **Configuración**, seleccione **Reglas de seguridad de salida**.

1. Seleccione **Agregar**. Escriba todos los detalles necesarios para crear una nueva regla, como se explica en [Configuración de reglas de seguridad](../virtual-network/manage-network-security-group.md#security-rule-settings). Asegúrese de que la opción **Destino** esté establecida en *Etiqueta de servicio* y de que **Etiqueta de servicio de destino** esté establecido en *AzureBackup*.

1. Seleccione **Agregar** para guardar la regla de seguridad de salida recién creada.

Puede crear reglas de seguridad de salida de NSG para Azure Storage y Azure AD de forma similar. Para más información sobre las etiquetas de servicio, consulte [este artículo](../virtual-network/service-tags-overview.md).

### <a name="azure-firewall-tags"></a>Etiquetas de Azure Firewall

Si usa Azure Firewall, cree una regla de aplicación mediante la [etiqueta de FQDN de Azure Firewall](../firewall/fqdn-tags.md) *AzureBackup*. Esto permite el acceso de salida total a Azure Backup.

### <a name="allow-access-to-service-ip-ranges"></a>Permitir el acceso a los intervalos de direcciones IP del servicio

Si decide permitir el acceso a las direcciones IP del servicio, vea los intervalos de direcciones IP en el archivo JSON disponible [aquí](https://www.microsoft.com/download/confirmation.aspx?id=56519). Tiene que permitir el acceso a las direcciones IP correspondientes a Azure Backup, Azure Storage y Azure Active Directory.

### <a name="allow-access-to-service-fqdns"></a>Permitir el acceso a los FQDN del servicio

También puede usar los siguientes FQDN para permitir el acceso a los servicios necesarios desde los servidores:

| Servicio    | Nombres de dominio a los que se va a acceder                             |
| -------------- | ------------------------------------------------------------ |
| Azure Backup  | `*.backup.windowsazure.com`                             |
| Azure Storage | `*.blob.core.windows.net` <br><br> `*.queue.core.windows.net` |
| Azure AD      | Permitir el acceso a los FQDN conforme a las secciones 56 y 59 según [este artículo](/office365/enterprise/urls-and-ip-address-ranges#microsoft-365-common-and-office-online) |

### <a name="use-an-http-proxy-server-to-route-traffic"></a>Empleo de un servidor proxy HTTP para enrutar el tráfico

Al hacer una copia de seguridad de una base de datos SAP HANA que se ejecuta en una máquina virtual de Azure, la extensión de copia de seguridad de la máquina virtual usa las API HTTPS para enviar comandos de administración a Azure Backup y datos a Azure Storage. La extensión de copia de seguridad también usa Azure AD para la autenticación. Enrute el tráfico de extensión de copia de seguridad de estos tres servicios a través del proxy HTTP. Use la lista de direcciones IP y FQDN mencionada anteriormente para permitir el acceso a los servicios necesarios. No se admiten servidores proxy autenticados.

## <a name="what-the-pre-registration-script-does"></a>Qué hace el script de registro previo

La ejecución del script de registro previo realiza las funciones siguientes:

* Según la distribución de Linux, el script instala o actualiza los paquetes necesarios que necesita el agente de Azure Backup.
* Realiza comprobaciones de conectividad de red saliente con los servidores de Azure Backup y servicios dependientes, como Azure Active Directory y Azure Storage.
* Inicia sesión en el sistema HANA con la clave de usuario que se enumera como parte de los [requisitos previos](#prerequisites). Esta clave se usa para crear un usuario de copia de seguridad (AZUREWLBACKUPHANAUSER) en el sistema HANA y **se puede eliminar después de que el script de registro previo se ejecute correctamente**.
* A AZUREWLBACKUPHANAUSER se le asignan estos roles y permisos necesarios:
  * ADMINISTRADOR DE BASE DE DATOS (en el caso de MDC) y ADMINISTRADOR DE COPIAS DE SEGURIDAD (en el caso de SDC): para crear nuevas bases de datos durante la restauración.
  * CATALOG READ: para leer el catálogo de copia de seguridad.
  * SAP_INTERNAL_HANA_SUPPORT: para acceder a algunas tablas privadas.
* El script agrega una clave a **hdbuserstore** para que el complemento de copias de seguridad de HANA, AZUREWLBACKUPHANAUSER, controle todas las operaciones (consultas de base de datos, operaciones de restauración, configuración y ejecución de copias de seguridad).

>[!NOTE]
> Puede pasar explícitamente la clave de usuario que se muestra como parte de los [requisitos previos](#prerequisites) como parámetro al script anterior al registro: `-sk SYSTEM_KEY_NAME, --system-key SYSTEM_KEY_NAME` <br><br>
>Para saber qué otros parámetros acepta el script, use el comando `bash msawb-plugin-config-com-sap-hana.sh --help`.

Para confirmar la creación de la clave, ejecute el comando HDBSQL en la máquina HANA con credenciales SIDADM:

```hdbsql
hdbuserstore list
```

La salida de comandos debe mostrar la tecla {SID}{DBNAME}, con el usuario como AZUREWLBACKUPHANAUSER.

>[!NOTE]
> Asegúrese de que tiene un conjunto único de archivos SSFS en `/usr/sap/{SID}/home/.hdb/`. Solo debería haber una carpeta en esta ruta de acceso.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un almacén de Recovery Services es una entidad que almacena las copias de seguridad y los puntos de recuperación creados a lo largo del tiempo. También contiene las directivas de copia de seguridad asociadas con las máquinas virtuales protegidas.

Para crear un almacén de Recovery Services:

1. Inicie sesión en su suscripción en [Azure Portal](https://portal.azure.com/).

2. En el menú izquierdo, seleccione **Todos los servicios**.

   ![Seleccionar Todos los servicios](./media/tutorial-backup-sap-hana-db/all-services.png)

3. En el cuadro de diálogo **Todos los servicios**, escriba **Recovery Services**. La lista de recursos se filtra en función de lo que especifique. En la lista de recursos, seleccione **Almacenes de Recovery Services**.

   ![Selección de almacenes de Recovery Services](./media/tutorial-backup-sap-hana-db/recovery-services-vaults.png)

4. En el panel **Almacenes de Recovery Services**, seleccione **Agregar**.

   ![Adición de un almacén de Recovery Services](./media/tutorial-backup-sap-hana-db/add-vault.png)

   Se abre el cuadro de diálogo **Almacén de Recovery Services**. Proporcione valores para **Nombre, Suscripción, Grupo de recursos** y **Ubicación**.

   ![Crear almacén de Recovery Services](./media/tutorial-backup-sap-hana-db/create-vault.png)

   * **Name**: el nombre se usa para identificar el almacén de Recovery Services y debe ser único en la suscripción de Azure. Especifique un nombre que tenga entre 2 y 50 caracteres. El nombre debe comenzar por una letra y consta solo de letras, números y guiones. En este tutorial, hemos usado el nombre **SAPHanaVault**.
   * **Suscripción**: elija la suscripción que desee usar. Si es miembro de una sola suscripción, verá solo ese nombre. Si no está seguro de qué suscripción va a utilizar, use la predeterminada (sugerida). Solo hay varias opciones si la cuenta profesional o educativa está asociada a más de una suscripción de Azure. Aquí hemos usado la suscripción **SAP HANA solution lab subscription**.
   * **Grupo de recursos**: Use un grupo de recursos existente o cree uno. Aquí hemos usado **SAPHANADemo**.<br>
   Para ver la lista de grupos de recursos disponibles en una suscripción, seleccione **Usar existente** y, después, seleccione un recurso en el cuadro de lista desplegable. Para crear un grupo de recursos, seleccione **Crear nuevo** y escriba el nombre. Para más información acerca de los grupos de recursos, consulte [Introducción a Azure Resource Manager](../azure-resource-manager/management/overview.md).
   * **Ubicación**: seleccione la región geográfica del almacén. El almacén debe estar en la misma región que las máquinas virtuales que ejecutan SAP HANA. Hemos usado **Este de EE. UU. 2**.

5. Seleccione **Revisar + crear**.

   ![Seleccionar Revisar + crear](./media/tutorial-backup-sap-hana-db/review-create.png)

Ahora se ha creado el almacén de Recovery Services.

## <a name="discover-the-databases"></a>Detección de bases de datos

1. En el almacén, en **Introducción**, seleccione **Copia de seguridad**. En **¿Dónde se ejecuta su carga de trabajo?** , seleccione **SAP HANA en máquina virtual de Azure**.
2. Seleccione **Iniciar detección**. Se iniciará la detección de máquinas virtuales de Linux no protegidas en la región del almacén. Verá la máquina virtual de Azure que quiere proteger.
3. En **Seleccionar máquinas virtuales**, seleccione el vínculo para descargar el script que proporciona permisos para que el servicio Azure Backup obtenga acceso a las máquinas virtuales de SAP HANA para la detección de bases de datos.
4. Ejecute el script en la máquina virtual que hospeda las bases de datos de SAP HANA de las que desea hacer una copia de seguridad.
5. Tras ejecutar el script en la máquina virtual, seleccione la máquina virtual en **Seleccionar máquinas virtuales**. A continuación, seleccione **Detectar bases de datos**.
6. Azure Backup detecta todas las bases de datos de SAP HANA en la máquina virtual. Durante la detección, Azure Backup registra la máquina virtual con el almacén e instala una extensión en la máquina virtual. No se instala ningún agente en la base de datos.

   ![Detección de bases de datos](./media/tutorial-backup-sap-hana-db/database-discovery.png)

## <a name="configure-backup"></a>Configuración de la copia de seguridad

Ahora que se han detectado las bases de datos de las que queremos realizar una copia de seguridad, vamos a habilitar la copia de seguridad.

1. Seleccione **Configurar copia de seguridad**.

   ![Configuración de la copia de seguridad](./media/tutorial-backup-sap-hana-db/configure-backup.png)

2. En **Seleccione los elementos de los que se va a realizar copia de seguridad**, seleccione una o varias bases de datos que quiera proteger y, a continuación, haga seleccione **Aceptar**.

   ![Seleccionar elementos de los que se va a hacer una copia de seguridad](./media/tutorial-backup-sap-hana-db/select-items-to-backup.png)

3. En **Directiva de copia de seguridad > Elegir directiva de copia de seguridad**, cree una nueva directiva de copia de seguridad para las bases de datos, según las instrucciones de la sección siguiente.

   ![Elegir directiva de copia de seguridad](./media/tutorial-backup-sap-hana-db/backup-policy.png)

4. Tras crear la directiva, en el **menú Copia de seguridad**, seleccione **Habilitar copia de seguridad**.

   ![Seleccionar Habilitar copia de seguridad](./media/tutorial-backup-sap-hana-db/enable-backup.png)

5. Realice el seguimiento del progreso de la configuración de copia de seguridad en el área **Notificaciones** del portal.

## <a name="creating-a-backup-policy"></a>Creación de una directiva de copia de seguridad

Una directiva de copia de seguridad define cuándo se realizan las copias de seguridad y cuánto tiempo se conservan.

* Una directiva se crea en el nivel de almacén.
* Varios almacenes pueden usar la misma directiva de copia de seguridad, pero debe aplicar la directiva de copia de seguridad a cada almacén.

Especifique la configuración de la directiva como se muestra a continuación:

1. En **Nombre de la directiva**, escriba un nombre para la nueva directiva. En este caso, escriba **SAPHANA**.

   ![Escribir el nombre de la nueva directiva](./media/tutorial-backup-sap-hana-db/new-policy.png)

2. En el menú **Full Backup policy** (Directiva de copia de seguridad completa), seleccione un valor para **Frecuencia de copia de seguridad**. Puede elegir **Diaria** o **Semanal**. En este tutorial, elegimos la copia de seguridad **Diaria**.

   ![Seleccionar una frecuencia de copia de seguridad](./media/tutorial-backup-sap-hana-db/backup-frequency.png)

3. En **Duración de retención**, configure las opciones de retención de la copia de seguridad completa.
   * De forma predeterminada, se seleccionan todas las opciones. Borre los límites del intervalo de retención que no desee usar y establezca aquellos que sí quiera utilizar.
   * El período de retención mínimo para cualquier tipo de copia de seguridad (completa, diferencial o de registro) es de siete días.
   * Los puntos de recuperación se etiquetan para la retención en función de su duración de retención. Por ejemplo, si selecciona la frecuencia diaria, solo se desencadena una copia de seguridad completa cada día.
   * La copia de seguridad de un día específico se etiqueta y se retiene según la duración de retención semanal y la configuración.
   * La duración de retención mensual y anual se comporta de forma similar.
4. En el menú de la **directiva de copia de seguridad completa**, seleccione **Aceptar** para aceptar la configuración.
5. A continuación, seleccione **Copia de seguridad diferencial** para agregar una directiva diferencial.
6. En **Differential Backup policy** (Directiva de copia de seguridad diferencial), seleccione **Enable** (Habilitar) para abrir los controles de retención y frecuencia. Hemos habilitado una copia de seguridad diferencial cada **domingo** a las **2:00 AM**, que se conserva durante **30 días**.

   ![Directiva de copia de seguridad diferencial](./media/tutorial-backup-sap-hana-db/differential-backup-policy.png)

   >[!NOTE]
   >Las copias de seguridad incrementales no se admiten en la actualidad.
   >

7. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.
8. Para agregar una directiva de copia de seguridad del registro de transacciones, seleccione **Copia de seguridad de registros**.
   * La **Copia de seguridad de registros** está establecida de forma predeterminada en **Habilitar**. No se puede deshabilitar, ya que SAP HANA administra todas las copias de seguridad de registros.
   * Hemos establecido **2 horas** como programación de copia de seguridad y **15 días** de período de retención.

    ![Directiva de copia de seguridad de registros](./media/tutorial-backup-sap-hana-db/log-backup-policy.png)

   >[!NOTE]
   > Las copias de seguridad de registros comienzan el flujo solo después de que se haya completado correctamente una copia de seguridad completa.
   >

9. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.
10. Al acabar de definir la directiva de copia de seguridad, seleccione **Aceptar**.

Ahora ha configurado correctamente las copias de seguridad de las bases de datos de SAP HANA.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [ejecutar copias de seguridad a petición en bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure](backup-azure-sap-hana-database.md#run-an-on-demand-backup).
* Obtenga información sobre cómo [restaurar bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure](sap-hana-db-restore.md).
* Obtenga información acerca de cómo [administrar bases de datos de SAP HANA de las que se realizan copias de seguridad mediante Azure Backup](sap-hana-db-manage.md).
* Obtenga información acerca de cómo [solucionar problemas comunes al realizar copias de seguridad de bases de datos de SAP HANA](backup-azure-sap-hana-database-troubleshoot.md).
