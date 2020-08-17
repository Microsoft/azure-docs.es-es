---
title: Copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure
description: En este artículo, aprenderá a realizar copias de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure con Azure Backup.
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: 4cfd8233b9a696b5b4b1981eefa81aa9723f6431
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86538981"
---
# <a name="back-up-sql-server-databases-in-azure-vms"></a>Copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure

Las bases de datos SQL Server son cargas de trabajo críticas que requieren un bajo objetivo de punto de recuperación (RPO) y retención a largo plazo. Puede hacer una copia de seguridad de las bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure mediante [Azure Backup](backup-overview.md).

En este artículo se muestra cómo hacer una copia de seguridad de una base de datos de SQL Server que se ejecuta en una máquina virtual de Azure en un almacén de Azure Backup Recovery Services.

En este artículo, aprenderá a:

> [!div class="checklist"]
>
> * Crear y configurar un almacén.
> * Detectar bases de datos y configurar copias de seguridad.
> * Configurar la protección automática de las bases de datos.

>[!NOTE]
>**La eliminación temporal de SQL Server en máquinas virtuales de Azure y la eliminación temporal de SAP HANA en cargas de trabajo de máquinas virtuales de Azure** están ahora disponible en versión preliminar.<br>
>Para suscribirse a la versión preliminar, escriba a AskAzureBackupTeam@microsoft.com.

## <a name="prerequisites"></a>Requisitos previos

Para poder realizar copias de seguridad de la base de datos de SQL Server, primero debe comprobar si reúne los siguientes criterios:

1. Identifique o cree un [almacén de Recovery Services](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) en la misma región y suscripción que la máquina virtual que hospeda la instancia de SQL Server.
1. Compruebe que la máquina virtual tenga [conectividad de red](backup-sql-server-database-azure-vms.md#establish-network-connectivity).
1. Asegúrese de que las bases de datos de SQL Server siguen las [directrices de nomenclatura de bases de datos para Azure Backup](#database-naming-guidelines-for-azure-backup).
1. Asegúrese de que la longitud combinada del nombre de la máquina virtual de SQL Server y el nombre del grupo de recursos no supere los 84 caracteres en el caso de Azure Resource Manager (máquinas virtuales ARM_) y los 77 caracteres en el caso de las máquinas virtuales clásicas. Esta limitación se debe a que algunos caracteres están reservados por el servicio.
1. Compruebe que no dispone de otras soluciones de copia de seguridad habilitadas para la base de datos. Deshabilite todas las demás copias de seguridad de SQL Server antes de hacer una copia de seguridad de la base de datos.

> [!NOTE]
> Puede habilitar Azure Backup para una máquina virtual de Azure y también para una base de datos de SQL Server que se ejecute en la máquina virtual sin que se produzcan conflictos.

### <a name="establish-network-connectivity"></a>Establecimiento de conectividad de red

En todas las operaciones, una VM con SQL Server necesita conectividad con el servicio Azure Backup, Azure Storage y Azure Active Directory. Esto puede lograrse mediante puntos de conexión privados o si se permite el acceso a las direcciones IP públicas o FQDN necesarios. No permitir la conectividad adecuada con los servicios de Azure necesarios puede provocar errores en operaciones como la detección de bases de datos, la configuración de copias de seguridad, la realización de copias de seguridad y la restauración de datos.

En la tabla siguiente se enumeran las distintas alternativas que se pueden usar para establecer la conectividad:

| **Opción**                        | **Ventajas**                                               | **Desventajas**                                            |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Puntos de conexión privados                 | Permite copias de seguridad a través de direcciones IP privadas dentro de la red virtual  <br><br>   Proporciona un control pormenorizado de la red y el almacén | Incurre en [costos](https://azure.microsoft.com/pricing/details/private-link/) estándar de punto de conexión privado |
| Etiquetas de servicio de NSG                  | Más fácil de administrar ya que los cambios de intervalo se combinan automáticamente   <br><br>   Sin costos adicionales. | Solo se puede usar con grupos de seguridad de red.  <br><br>    Proporciona acceso a todo el servicio. |
| Etiquetas de FQDN de Azure Firewall          | Más facilidad de administración, ya que los FQDN necesarios se administran automáticamente | Solo se puede usar con Azure Firewall.                         |
| Permite el acceso a los FQDN/IP del servicio | Sin costos adicionales.   <br><br>  Funciona con todos los firewalls y dispositivos de seguridad de red | Es posible que sea necesario acceder a un amplio conjunto de direcciones IP o FQDN   |
| Usar un servidor proxy HTTP                 | Un único punto de acceso a Internet para las máquinas virtuales.                       | Costos adicionales de ejecutar una máquina virtual con el software de proxy.         |

A continuación se proporcionan más detalles sobre el uso de estas opciones:

#### <a name="private-endpoints"></a>Puntos de conexión privados

Los puntos de conexión privados permiten conectar de forma segura desde los servidores de una red virtual al almacén de Recovery Services. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual del almacén. El tráfico de red entre los recursos dentro de la red virtual y el almacén viaja a través de la red virtual y un vínculo privado en la red troncal de Microsoft. Esto elimina la exposición de la red pública de Internet. Obtenga más información sobre los puntos de conexión privados para Azure Backup [aquí](./private-endpoints.md).

#### <a name="nsg-tags"></a>Etiquetas de NSG

Si emplea grupos de seguridad de red (NSG), use la etiqueta de servicio de *AzureBackup* para permitir el acceso de salida a Azure Backup. Además de la etiqueta de Azure Backup, también necesita permitir la conectividad para la autenticación y la transferencia de datos mediante la creación de [reglas de NSG](../virtual-network/security-overview.md#service-tags) similares para *Azure AD* y *Azure Storage*.  En los pasos siguientes se describe el proceso para crear una regla para la etiqueta de Azure Backup:

1. En **Todos los servicios**, vaya a **Grupos de seguridad de red** y seleccione el grupo de seguridad de red.

1. En **Configuración**, seleccione **Reglas de seguridad de salida**.

1. Seleccione **Agregar**. Escriba todos los detalles necesarios para crear una nueva regla, como se explica en [Configuración de reglas de seguridad](../virtual-network/manage-network-security-group.md#security-rule-settings). Asegúrese de que la opción **Destino** esté establecida en *Etiqueta de servicio* y de que **Etiqueta de servicio de destino** esté establecido en *AzureBackup*.

1. Haga clic en **Agregar** para guardar la regla de seguridad de salida recién creada.

Puede crear reglas de seguridad de salida de NSG para Azure Storage y Azure AD de forma similar.

#### <a name="azure-firewall-tags"></a>Etiquetas de Azure Firewall

Si usa Azure Firewall, cree una regla de aplicación mediante la [etiqueta de FQDN de Azure Firewall](../firewall/fqdn-tags.md) *AzureBackup*. Esto permite el acceso de salida total a Azure Backup.

#### <a name="allow-access-to-service-ip-ranges"></a>Permitir el acceso a los intervalos de direcciones IP del servicio

Si decide permitir el acceso a las direcciones IP del servicio, vea los intervalos de direcciones IP en el archivo JSON disponible [aquí](https://www.microsoft.com/download/confirmation.aspx?id=56519). Tiene que permitir el acceso a las direcciones IP correspondientes a Azure Backup, Azure Storage y Azure Active Directory.

#### <a name="allow-access-to-service-fqdns"></a>Permitir el acceso a los FQDN del servicio

También puede usar los siguientes FQDN para permitir el acceso a los servicios necesarios desde los servidores:

| Servicio    | Nombres de dominio a los que se va a acceder                             |
| -------------- | ------------------------------------------------------------ |
| Azure Backup  | `*.backup.windowsazure.com`                             |
| Azure Storage | `*.blob.core.windows.net` <br><br> `*.queue.core.windows.net` |
| Azure AD      | Permitir el acceso a los FQDN conforme a las secciones 56 y 59 según [este artículo](/office365/enterprise/urls-and-ip-address-ranges#microsoft-365-common-and-office-online) |

#### <a name="use-an-http-proxy-server-to-route-traffic"></a>Empleo de un servidor proxy HTTP para enrutar el tráfico

Cuando hace copia de seguridad de una base de datos de SQL Server en una máquina virtual de Azure, la extensión de copia de seguridad en la máquina virtual usa las API HTTPS para enviar comandos de administración a Azure Backup y datos a Azure Storage. La extensión de copia de seguridad también usa Azure AD para la autenticación. Enrute el tráfico de extensión de copia de seguridad de estos tres servicios a través del proxy HTTP. Use la lista de direcciones IP y FQDN mencionada anteriormente para permitir el acceso a los servicios necesarios. No se admiten servidores proxy autenticados.

### <a name="database-naming-guidelines-for-azure-backup"></a>Instrucciones de nomenclatura de la base de datos para Azure Backup

Evite el uso de los elementos siguientes en los nombres de las bases de datos:

* Espacios iniciales o finales
* Signos de exclamación (!)
* Corchetes de cierre (])
* Punto y coma ';'
* Barra diagonal '/'

El establecimiento de alias está permitido para los caracteres no admitidos, pero se recomienda evitarlos. Para obtener más información, consulte [Descripción del modelo de datos del servicio Tabla](/rest/api/storageservices/understanding-the-table-service-data-model).

>[!NOTE]
>La operación **Configurar protección** no se admite en las bases de datos con caracteres especiales como "+" o "&" en su nombre. Puede cambiar el nombre de la base de datos o habilitar la opción **Auto Protection** (Protección automática), que le permitirá proteger correctamente estas bases de datos.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>Detección de bases de datos SQL Server

Detección de las bases de datos que se ejecutan en la máquina virtual:

1. En [Azure Portal](https://portal.azure.com), abra el almacén de Recovery Services que usa para la copia de seguridad de la base de datos.

2. En el panel **Almacén de Recovery Services**, seleccione **Copia de seguridad**.

   ![Seleccionar Copia de seguridad para abrir el menú Objetivo de Backup](./media/backup-azure-sql-database/open-backup-menu.png)

3. En **Objetivo de Backup**, establezca **¿Dónde se ejecuta su carga de trabajo?** en **Azure**.

4. En **What do you want to backup** (¿De qué quiere realizar una copia de seguridad?), seleccione **SQL Server in Azure VM** (SQL Server en la máquina virtual de Azure).

    ![Seleccionar SQL Server en una máquina virtual de Azure para la copia de seguridad](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

5. En **Backup Goal** > **Discover DBs in VMs** (Objetivo de Backup > Detectar bases de datos en máquinas virtuales), seleccione **Start Discovery** (Iniciar detección) para buscar las máquinas virtuales sin protección en la suscripción. Puede que la búsqueda tarde un rato, según el número de máquinas virtuales sin protección en la suscripción.

   * Tras la detección, las máquinas virtuales no protegidas deben aparecer en la lista por nombre y grupo de recursos.
   * Si una máquina virtual no aparece en la lista como cabría esperar, compruebe que no se haya copiado ya en un almacén.
   * Varias máquinas virtuales pueden tener el mismo nombre pero pertenecer a distintos grupos de recursos.

     ![La copia de seguridad queda pendiente durante la búsqueda de bases de datos en máquinas virtuales](./media/backup-azure-sql-database/discovering-sql-databases.png)

6. En la lista de máquinas virtuales, seleccione la que ejecuta la base de datos de SQL Server > **Discover DBs** (Detectar bases de datos).

7. Realice el seguimiento de la detección de las bases de datos en **Notificaciones**. El tiempo necesario para esta acción depende del número de bases de datos de máquina virtual. Cuando se detectan las bases de datos seleccionadas, aparece un mensaje de operación correcta.

    ![Mensaje de implementación correcta](./media/backup-azure-sql-database/notifications-db-discovered.png)

8. Azure Backup detecta todas las bases de datos de SQL Server en la máquina virtual. Durante la detección, los siguientes elementos tienen lugar en segundo plano:

    * Azure Backup registra la máquina virtual en el almacén para la copia de seguridad de la carga de trabajo. Todas las bases de datos de la máquina virtual registrada solo se pueden copiar en este almacén.
    * Azure Backup instala la extensión AzureBackupWindowsWorkload en la máquina virtual. No se instala ningún agente en la base de datos SQL.
    * Azure Backup crea la cuenta de servicio NT Service\AzureWLBackupPluginSvc en la máquina virtual.
      * Todas las operaciones de copia de seguridad y restauración utilizan la cuenta de servicio.
      * NT Service\AzureWLBackupPluginSvc necesita permisos de administrador del sistema de SQL. Todas las máquinas virtuales de SQL Server creadas en Marketplace llevan instalado SqlIaaSExtension. La extensión AzureBackupWindowsWorkload usa SQLIaaSExtension para obtener automáticamente los permisos necesarios.
    * Si no ha creado la máquina virtual desde Marketplace o si tiene SQL 2008 y 2008 R2, es posible que la máquina virtual no tenga instalado SqlIaaSExtension y que la operación de detección produzca el mensaje de error UserErrorSQLNoSysAdminMembership. Para solucionar este problema, siga las instrucciones de [Establecer permisos de máquina virtual](backup-azure-sql-database.md#set-vm-permissions).

        ![Seleccionar la máquina virtual y la base de datos](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup"></a>Configuración de la copia de seguridad  

1. En **Objetivo de Backup** > **Paso 2: Configurar copia de seguridad**, seleccione **Configurar copia de seguridad**.

   ![Seleccionar Configurar copia de seguridad](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

1. Haga clic en **Agregar recursos** para ver todos los grupos de disponibilidad registrados y las instancias de SQL Server independientes.

    ![Seleccionar Agregar recursos](./media/backup-azure-sql-database/add-resources.png)

1. En la pantalla **Seleccionar los elementos de los que se debe realizar una copia de seguridad**, seleccione la flecha situada a la izquierda de una fila para expandir la lista de todas las bases de datos no protegidas en esa instancia o el grupo de disponibilidad Always On.

    ![Seleccionar los elementos de los que se debe realizar una copia de seguridad](./media/backup-azure-sql-database/select-items-to-backup.png)

1. Elija todas las bases de datos que quiere proteger y, después, seleccione **Aceptar**.

   ![Protección de la base de datos](./media/backup-azure-sql-database/select-database-to-protect.png)

   Para optimizar las cargas de copia de seguridad, el número máximo de bases de datos en un trabajo de copia de seguridad en Azure Backup se establece en 50.

     * Para proteger más de 50 bases de datos, configure varias copias de seguridad.
     * Para [habilitar](#enable-auto-protection) toda la instancia o el grupo de disponibilidad Always On, en la lista desplegable **PROTECCIÓN AUTOMÁTICA**, seleccione **ACTIVADA** y luego **ACEPTAR**.

         > [!NOTE]
         > La característica de [protección automática](#enable-auto-protection) no solo habilita la protección en todas las bases de datos existentes a la vez, sino que también protege automáticamente las nuevas bases de datos que se agregan a esa instancia o al grupo de disponibilidad.  

1. Defina la **directiva de copia de seguridad**. Puede realizar una de las operaciones siguientes:

   * Seleccione la directiva predeterminada como *HourlyLogBackup*.
   * Elegir una directiva de copia de seguridad existente creada previamente para SQL.
   * Defina una nueva directiva basada en el objetivo de punto de recuperación (RPO) y en la duración de retención.

     ![Seleccionar directiva de copia de seguridad](./media/backup-azure-sql-database/select-backup-policy.png)

1. Haga clic en **Habilitar copia de seguridad** para enviar la operación **Configurar protección** y realizar un seguimiento del progreso de la configuración en el área **Notificaciones** del portal.

   ![Seguimiento del progreso de la configuración](./media/backup-azure-sql-database/track-configuration-progress.png)

### <a name="create-a-backup-policy"></a>Crear una directiva de copia de seguridad

Una directiva de copia de seguridad define cuándo se realizan las copias de seguridad y cuánto tiempo se conservan.

* Una directiva se crea en el nivel de almacén.
* Varios almacenes pueden usar la misma directiva de copia de seguridad, pero debe aplicar la directiva de copia de seguridad a cada almacén.
* Al crear una directiva de copia de seguridad, la copia de seguridad completa diaria es la predeterminada.
* Puede agregar una copia de seguridad diferencial, pero solo si configura las copias de seguridad completas para que se realicen semanalmente.
* Más información sobre los [diferentes tipos de directivas de copia de seguridad](backup-architecture.md#sql-server-backup-types).

Para crear una directiva de copia de seguridad:

1. En el almacén, seleccione **Directivas de copia de seguridad** > **Agregar**.
1. En **Agregar**, seleccione **SQL Server en una máquina virtual de Azure** para definir el tipo de directiva.

   ![Elección de un tipo de directiva para la nueva directiva de copia de seguridad](./media/backup-azure-sql-database/policy-type-details.png)

1. En **Nombre de la directiva**, escriba un nombre para la nueva directiva.

    ![Escribir nombre de la directiva](./media/backup-azure-sql-database/policy-name.png)

1. Haga clic en el vínculo **Editar** correspondiente a **Copia de seguridad completa** para modificar la configuración predeterminada.

   * Seleccione una **frecuencia de copia de seguridad**. Elija **Diario** o **Semanal**.
   * En **Diariamente**, seleccione la hora y zona horaria en la que comienza el trabajo de copia de seguridad. No puede crear copias de seguridad diferenciales para copias de seguridad completas diarias.

     ![Nuevos campos de directiva de copia de seguridad](./media/backup-azure-sql-database/full-backup-policy.png)  

1. De forma predeterminada, todas las opciones de **DURACIÓN DE RETENCIÓN** están seleccionadas. Borre cualquier límite de la duración de retención que no desee y, a continuación, establezca los intervalos que se van a usar.

    * El período de retención mínimo para cualquier tipo de copia de seguridad (completa, diferencial o de registro) es de siete días.
    * Los puntos de recuperación se etiquetan para la retención en función de su duración de retención. Por ejemplo, si selecciona la frecuencia diaria, solo se desencadena una copia de seguridad completa cada día.
    * La copia de seguridad de un día específico se etiqueta y se retiene según la duración de la retención semanal y la configuración de esta.
    * La duración de retención mensual y anual se comporta de forma similar.

       ![Configuración del intervalo de la duración de retención](./media/backup-azure-sql-database/retention-range-interval.png)

1. Seleccione **Aceptar** para aceptar la configuración de las copias de seguridad completas.
1. Haga clic en el vínculo **Editar** correspondiente a **Copia de seguridad diferencial** para modificar la configuración predeterminada.

    * En **Differential Backup policy** (Directiva de copia de seguridad diferencial), seleccione **Enable** (Habilitar) para abrir los controles de retención y frecuencia.
    * Puede desencadenar una copia de seguridad diferencial al día. No se puede desencadenar una copia de seguridad diferencial y una copia de seguridad completa en el mismo día.
    * Como máximo, las copias de seguridad diferenciales se pueden retener durante 180 días.
    * No se admite la copia de seguridad diferencial para la base de datos maestra.

      ![Directiva de copia de seguridad diferencial](./media/backup-azure-sql-database/differential-backup-policy.png)

1. Haga clic en el vínculo **Editar** correspondiente a **Copia de seguridad de registros** para modificar la configuración predeterminada.

    * En el menú **Log Backup** (Copia de seguridad de registros), seleccione **Enable** (Habilitar) y, luego, establezca los controles de retención y frecuencia.
    * Las copia de seguridad de registros pueden realizarse cada 15 minutos y se pueden retener durante un período máximo de 35 días.
    * Si la base de datos se encuentra en el [modelo de recuperación simple](/sql/relational-databases/backup-restore/recovery-models-sql-server?view=sql-server-ver15), se pausará la programación de copias de seguridad de registros de esa base de datos y, por tanto, no se desencadenará ninguna copia de seguridad de registros.
    * Si el modelo de recuperación de la base de datos cambia de **Completa** a **Simple**, las copias de seguridad de registros se pausarán en un plazo de 24 horas después del cambio en el modelo de recuperación. Del mismo modo, si el modelo de recuperación cambia desde **Simple**, lo que implica que puedan admitirse copias de seguridad de registros en la base de datos, las programaciones de copias de seguridad de registros se habilitarán en un plazo de 24 horas después del cambio en el modelo de recuperación.

      ![Directiva de copia de seguridad de registros](./media/backup-azure-sql-database/log-backup-policy.png)

1. En el menú **Directiva de copia de seguridad**, elija si quiere habilitar la **compresión de copia de seguridad de SQL** o no. Esta opción está deshabilitada de manera predeterminada. Si está habilitada, SQL Server enviará una secuencia de copia de seguridad comprimida a VDI. Azure Backup invalida los valores predeterminados de nivel de instancia con la cláusula COMPRESSION/NO_COMPRESSION según el valor de este control.

1. Después de completar las modificaciones en la directiva de copia de seguridad, seleccione **Aceptar**.

> [!NOTE]
> Cada copia de seguridad de registros está encadenada a la copia de seguridad completa anterior para formar una cadena de recuperación. Esta copia de seguridad completa se conservará hasta que expire la retención de la última copia de seguridad de registros. Esto puede significar que la copia de seguridad completa se conservará durante un período adicional para asegurarse de que se pueden recuperar todos los registros. Supongamos que el usuario tiene una copia de seguridad completa semanal, una copia diferencial diaria y registros de 2 horas. Todos ellos se conservan 30 días. Sin embargo, la copia completa semanal puede limpiarse o eliminarse en realidad solo después de que esté disponible la siguiente copia de seguridad completa, es decir, después de 30+7 días. Por ejemplo, una copia de seguridad completa semanal se produce el 16 de noviembre. Según la directiva de retención, debe conservarse hasta el 16 de diciembre. La última copia de seguridad de registros de esta copia completa se produce antes de la siguiente copia completa programada, el 22 de noviembre. Hasta que este registro esté disponible el 22 de diciembre, no se puede eliminar la copia completa del 16 de noviembre. Por lo tanto, la copia completa del 16 de noviembre se conserva hasta el 22 de diciembre.

## <a name="enable-auto-protection"></a>Habilitación de la protección automática  

Puede habilitar la protección automática para hacer una copia de seguridad automática de todas las bases de datos existentes y futuras en una instancia de SQL Server independiente o en un grupo de disponibilidad de AlwaysOn.

* No hay límite en cuanto al número de bases de datos que se pueden seleccionar para la protección automática en una misma operación. La detección se suele ejecutar cada ocho horas, pero puede detectar y proteger nuevas bases de datos de inmediato si ejecuta manualmente una detección seleccionando la opción **Volver a detectar bases de datos**.
* No puede proteger o excluir selectivamente las bases de datos de la protección en una instancia en el momento en que habilita la protección automática.
* Si la instancia ya incluye algunas bases de datos protegidos, permanecerán protegidos en sus respectivas directivas incluso después de activar la protección automática. Todas las bases de datos no protegidas que se agreguen posteriormente tendrán una única directiva que se define en el momento de habilitar la protección automática, indicada en **Configurar copia de seguridad**. Sin embargo, puede cambiar la directiva asociada a una base de datos protegida automáticamente más adelante.  

Para habilitar la protección automática:

  1. En **Items to backup** (Elementos para copia de seguridad), seleccione la instancia para la que quiere habilitar la protección automática.
  2. Seleccione la lista desplegable en **AUTOPROTECT** (Protección automática), elija **ACTIVAR** y, a continuación, seleccione **Aceptar**.

      ![Habilitación de la protección automática en el grupo de disponibilidad](./media/backup-azure-sql-database/enable-auto-protection.png)

  3. La copia de seguridad se configura para todas las bases de datos juntas y se puede realizar un seguimiento de ella en **Backup Jobs** (Trabajos de copia de seguridad).

Si tiene que deshabilitar la protección automática, haga clic en el nombre de la instancia en **Configurar copia de seguridad** y seleccione **Deshabilitar la protección automática**. Se seguirán haciendo copias de seguridad de todas las bases de datos, pero las bases de datos futuras no estarán protegidas automáticamente.

![Deshabilitación de la protección automática en dicha instancia](./media/backup-azure-sql-database/disable-auto-protection.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo:

* [Restauración de bases de datos de SQL Server con copia de seguridad](restore-sql-database-azure-vm.md)
* [Administración de bases de datos de SQL Server con copia de seguridad](manage-monitor-sql-database-backup.md)
