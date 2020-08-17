---
title: Copia de seguridad de un servidor de Exchange con System Center DPM
description: Obtenga información acerca de cómo realizar una copia de seguridad de un servidor Exchange en Azure Backup con System Center 2012 R2 DPM
ms.reviewer: kasinh
ms.topic: conceptual
ms.date: 01/31/2019
ms.openlocfilehash: 2d547b1d86b95a4f90d3faaa2f676c7cc37255d3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091137"
---
# <a name="back-up-an-exchange-server-to-azure-backup-with-system-center-2012-r2-dpm"></a>Copia de seguridad de un servidor Exchange en Azure Backup con System Center 2012 R2 DPM

En este artículo se describe cómo configurar un servidor de System Center 2012 R2 Data Protection Manager (DPM) para realizar una copia de seguridad de un servidor Microsoft Exchange en Azure Backup.  

## <a name="updates"></a>Actualizaciones

Para registrar correctamente el servidor DPM con Copia de seguridad de Azure, debe instalar el paquete acumulativo de actualizaciones más reciente de System Center 2012 R2 DPM y la versión más reciente de Azure Backup Agent. Obtenga el último paquete acumulativo de actualizaciones en el [Catálogo de Microsoft Update](https://catalog.update.microsoft.com/v7/site/Search.aspx?q=System%20Center%202012%20R2%20Data%20protection%20manager).

> [!NOTE]
> En los ejemplos de este artículo, se instala la versión 2.0.8719.0 de Azure Backup Agent y el paquete acumulativo de actualizaciones 6 en System Center 2012 R2 DPM.
>
>

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, asegúrese de que se cumplen todos los [requisitos previos](backup-azure-dpm-introduction.md#prerequisites-and-limitations) para usar Microsoft Azure Backup a fin de proteger las cargas de trabajo. Entre estos requisitos previos se incluyen los siguientes:

* Se ha creado un almacén de Copia de seguridad en el sitio de Azure.
* En el servidor DPM se han descargado las credenciales del almacén y del agente.
* El agente está instalado en el servidor DPM.
* Las credenciales de almacén se utilizaban para registrar el servidor DPM.
* Si va a proteger Exchange 2016, actualice al paquete acumulativo de actualizaciones 9 de DPM 2012 R2 o posterior.

## <a name="dpm-protection-agent"></a>Agente de protección DPM

Para instalar al agente de protección DPM en el servidor Exchange, siga estos pasos:

1. Asegúrese de que los firewalls estén configurados correctamente. Consulte [Configuración de excepciones de firewall para el agente](/system-center/dpm/configure-firewall-settings-for-dpm?view=sc-dpm-2019).
2. Instale el agente en el servidor Exchange; para ello, haga clic en **Administración > Agentes > Instalar** en la Consola de administrador DPM. Consulte [Instalación del agente de protección DPM](/system-center/dpm/deploy-dpm-protection-agent?view=sc-dpm-2019) para ver pasos detallados.

## <a name="create-a-protection-group-for-the-exchange-server"></a>Creación de un grupo de protección para el servidor Exchange

1. En la Consola de administrador DPM, haga clic en **Protección** y luego en **Nuevo** en la cinta de herramientas para abrir el asistente **Crear nuevo grupo de protección**.
2. En la pantalla **Bienvenido** del asistente, haga clic en **Siguiente**.
3. En la pantalla **Seleccionar tipo de grupo de protección**, seleccione **Servidores** y haga clic en **Siguiente**.
4. Seleccione la base de datos del servidor Exchange que desea proteger y haga clic en **Siguiente**.

   > [!NOTE]
   > Si va a proteger Exchange 2013, compruebe en [Requisitos previos de Exchange 2013](/system-center/dpm/back-up-exchange).
   >
   >

    En el ejemplo siguiente, está seleccionada la base de datos de Exchange 2010.

    ![Seleccionar a miembros del grupo](./media/backup-azure-backup-exchange-server/select-group-members.png)
5. Seleccione el método de protección de datos.

    Asigne un nombre al grupo de protección y, a continuación, seleccione las dos opciones siguientes:

   * Deseo protección a corto plazo mediante disco.
   * Deseo protección en línea.
6. Haga clic en **Next**.
7. Seleccione la opción **Ejecutar Eseutil para comprobar la integridad de los datos** si desea comprobar la integridad de las bases de datos de Exchange Server.

    Después de seleccionar esta opción, se ejecutará una comprobación de coherencia de copia de seguridad en el servidor DPM para evitar el tráfico de E/S que se genera al ejecutar el comando **eseutil** en el servidor Exchange.

   > [!NOTE]
   > Para usar esta opción, debe copiar los archivos Ese.dll y Eseutil.exe en el directorio C:\Archivos de programa\Microsoft System Center 2012 R2\DPM\DPM\bin del servidor DPM. De lo contrario, se desencadena el siguiente error:  
   > ![Error de eseutil](./media/backup-azure-backup-exchange-server/eseutil-error.png)
   >
   >
8. Haga clic en **Next**.
9. Seleccione la base de datos para **Copia de seguridad de copia**, y luego haga clic en **Siguiente**.

   > [!NOTE]
   > Si no selecciona "Copia de seguridad completa" al menos para una copia de DAG de una base de datos, no se truncarán los registros.
   >
   >
10. Configure los objetivos de **Copia de seguridad a corto plazo** y haga clic en **Siguiente**.
11. Revise el espacio en disco disponible y haga clic en **Siguiente**.
12. Seleccione la hora a la que el servidor DPM creará la replicación inicial y haga clic en **Siguiente**.
13. Seleccione las opciones de comprobación de coherencia y haga clic en **Siguiente**.
14. Elija la base de datos de la que desea realizar una copia de seguridad en Azure y haga clic en **Siguiente**. Por ejemplo:

    ![Especificar datos de protección en línea](./media/backup-azure-backup-exchange-server/specify-online-protection-data.png)
15. Defina la programación de **Azure Backup** y luego haga clic en **Siguiente**. Por ejemplo:

    ![Especificar programación de copia de seguridad en línea](./media/backup-azure-backup-exchange-server/specify-online-backup-schedule.png)

    > [!NOTE]
    > Los puntos de recuperación de notas en línea están basados en los puntos de recuperación completos rápidos. Por lo tanto, debe programar el punto de recuperación en línea después de la hora especificada para el punto de recuperación completo rápido.
    >
    >
16. Configure la directiva de retención para **Azure Backup** y luego haga clic en **Siguiente**.
17. Elija una opción de replicación en línea y haga clic en **Siguiente**.

    Si tiene una base de datos grande, se puede tardar mucho tiempo en crear la copia de seguridad inicial a través de la red. Para evitar este problema, puede crear una copia de seguridad sin conexión.  

    ![Especificar directiva de retención en línea](./media/backup-azure-backup-exchange-server/specify-online-retention-policy.png)
18. Confirme la configuración y haga clic en **Crear grupo**.
19. Haga clic en **Cerrar**.

## <a name="recover-the-exchange-database"></a>Recuperación de la base de datos de Exchange

1. Para recuperar una base de datos de Exchange, haga clic en **Recuperación** en la Consola de administrador DPM.
2. Busque la base de datos de Exchange que desea recuperar.
3. Seleccione un punto de recuperación en línea en la lista desplegable *Hora de recuperación* .
4. Haga clic en **Recuperar** para iniciar el **Asistente para recuperación**.

Para los puntos de recuperación en línea, existen cinco tipos de recuperación:

* **Recuperación en la ubicación original de Exchange Server**: los datos se recuperarán en la ubicación original de Exchange Server.
* **Recuperación en otra base de datos de Exchange Server**: los datos se recuperarán en otra base de datos en otra ubicación de Exchange Server.
* **Recuperación en una base de datos de recuperación**: los datos se recuperarán en una base de datos de recuperación de Exchange (RDB).
* **Copia en una carpeta de red**: los datos se recuperarán en una carpeta de red.
* **Copia en cinta**: si tiene una biblioteca de cintas o una unidad de cinta independiente conectada y configurada en el servidor DPM, el punto de recuperación se copiará en una cinta libre.

    ![Elegir replicación en línea](./media/backup-azure-backup-exchange-server/choose-online-replication.png)

## <a name="next-steps"></a>Pasos siguientes

* [Preguntas más frecuentes de Azure Backup](backup-azure-backup-faq.md)
