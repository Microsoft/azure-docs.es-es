---
title: Realización de una copia de seguridad de una granja de SharePoint en Azure con DPM
description: En este artículo se incluye información general sobre la protección del Servidor de Azure Backup y el servidor de DPM de una granja de SharePoint en Azure.
ms.topic: conceptual
ms.date: 03/09/2020
ms.openlocfilehash: b326ce15cd605f4f768a03b95337bbc5bd6aaabc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032891"
---
# <a name="back-up-a-sharepoint-farm-to-azure-with-dpm"></a>Realización de una copia de seguridad de una granja de SharePoint en Azure con DPM

La copia de seguridad de una granja de SharePoint en Microsoft Azure se crea mediante System Center Data Protection Manager (DPM) casi de la misma manera que realiza la copia de seguridad de otros orígenes de datos. Azure Backup ofrece flexibilidad en la programación de copias de seguridad para crear puntos de copia de seguridad diarios, semanales, mensuales o anuales, y le ofrece diferentes opciones de directiva de retención para varios puntos de copia de seguridad. DPM ofrece la posibilidad de almacenar copias en discos locales para conseguir objetivos de tiempo de recuperación (RTO) más rápidos y de almacenar copias en Azure, para una retención económica más a largo plazo.

La copia de seguridad de SharePoint en Azure con DPM es un proceso muy similar al de copia de seguridad de SharePoint en DPM localmente. Las consideraciones específicas de Azure se indicarán en este artículo.

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>Las versiones compatibles de SharePoint y relacionadas con escenarios de protección

Para obtener una lista de las versiones de SharePoint compatibles y las versiones de DPM necesarias para realizar una copia de seguridad de estas, consulte [¿De qué elementos puede hacer una copia de seguridad DPM?](/system-center/dpm/dpm-protection-matrix?view=sc-dpm-2019#applications-backup)

## <a name="before-you-start"></a>Antes de comenzar

Antes de realizar una copia de seguridad de una granja de SharePoint en Azure, hay algunas cuantas cosas que debe confirmar.

### <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, asegúrese de que se cumplen todos los [requisitos previos para usar Microsoft Azure Backup](backup-azure-dpm-introduction.md#prerequisites-and-limitations) para proteger las cargas de trabajo. Algunas de las tareas que son requisito previo incluyen: crear un almacén de copia de seguridad, descargar las credenciales de almacén, instalar el agente de copia de seguridad de Azure y registrar Azure Backup Server y el servidor DPM con el almacén.

Puede encontrar más requisitos previos y limitaciones en el artículo [Hacer copias de seguridad de SharePoint con DPM](/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#prerequisites-and-limitations).

## <a name="configure-backup"></a>Configuración de la copia de seguridad

Para realizar una copia de seguridad de una granja de SharePoint, configure la protección de SharePoint mediante ConfigureSharePoint.exe y, a continuación, cree un grupo de protección en DPM. Para obtener instrucciones, consulte [Configurar la copia de seguridad](/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#configure-backup) en la documentación de DPM.

## <a name="monitoring"></a>Supervisión

Para supervisar el trabajo de copia de seguridad, siga las instrucciones de [Supervisión de la copia de seguridad de DPM](/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#monitoring).

## <a name="restore-sharepoint-data"></a>Restauración de datos de SharePoint

Para obtener información sobre cómo restaurar un elemento de SharePoint desde un disco con DPM, consulte [Restaurar datos de SharePoint](/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#restore-sharepoint-data).

## <a name="restore-a-sharepoint-database-from-azure-by-using-dpm"></a>Restauración de una base de datos de SharePoint de Azure con DPM

1. Para recuperar una base de datos de contenido de SharePoint, desplácese por los diversos puntos de recuperación (tal como se mostró anteriormente) y seleccione aquel que quiera recuperar.

    ![Protección de SharePoint con DPM8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)
2. Haga doble clic en el punto de recuperación de SharePoint para mostrar la información de catálogo de SharePoint disponible.

   > [!NOTE]
   > Como la granja de SharePoint está protegida para la retención a largo plazo en Azure, no hay información de catálogo (metadatos) disponible en el servidor de DPM. Como resultado, cada vez que deba recuperar una base de datos de contenido de SharePoint en un momento dado, deberá volver a catalogar la granja de SharePoint.
   >
   >
3. Haga clic en **Volver a catalogar**.

    ![Protección de SharePoint con DPM10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    Aparece la ventana de estado **Volver a catalogar la nube** .

    ![Protección de SharePoint con DPM11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    Cuando finaliza la catalogación, el estado cambia a *Correcto*. Haga clic en **Cerrar**.

    ![Protección de SharePoint con DPM12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)
4. Haga clic en el objeto de SharePoint que se muestra en la pestaña **Recuperación** de DPM para obtener la estructura de la base de datos de contenido. Haga clic con el botón derecho en el elemento y luego haga clic en **Recuperar**.

    ![Protección de SharePoint con DPM13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. En este momento, siga los pasos de recuperación que se han indicado en este artículo para recuperar una base de datos de contenido de SharePoint desde el disco.

## <a name="switching-the-front-end-web-server"></a>Cambio del servidor front-end web

Si tiene más de un servidor front-end web y desea cambiar el servidor que DPM usa para proteger la granja, siga las instrucciones de [Cambio del servidor front-end web](/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#switching-the-front-end-web-server).

## <a name="next-steps"></a>Pasos siguientes

* [Azure Backup Server y DPM: Preguntas frecuentes](backup-azure-dpm-azure-server-faq.md)
* [Solución de problemas de System Center Data Protection Manager](backup-azure-scdpm-troubleshooting.md)
