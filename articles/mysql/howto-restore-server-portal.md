---
title: Copia de seguridad y restauración en Azure Database for MySQL mediante Azure Portal
description: En este artículo se describe cómo restaurar un servidor en Azure Database for MySQL mediante Azure Portal.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 6/30/2020
ms.openlocfilehash: 7c9e7cda862fe1112cce7ed8cff270843f0a8475
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902784"
---
# <a name="how-to-backup-and-restore-a-server-in-azure-database-for-mysql-using-the-azure-portal"></a>Copia de seguridad y restauración de un servidor en Azure Database for MySQL mediante Azure Portal

## <a name="backup-happens-automatically"></a>Las copias de seguridad se realizan automáticamente
Periódicamente, se realizan copias de seguridad de los servidores de Azure Database for MySQL para habilitar las características de restauración. Con esta característica, puede restaurar el servidor y todas sus bases de datos en un servidor nuevo a un momento dado anterior.

## <a name="prerequisites"></a>Requisitos previos
Para completar esta guía, necesita:
- Un [servidor de Azure Database for MySQL y una base de datos](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="set-backup-configuration"></a>Configuración de copia de seguridad

Elija la configuración del servidor para copias de seguridad con redundancia local o con redundancia geográfica en el momento de crear el servidor, en la ventana **Plan de tarifa**.

> [!NOTE]
> Después de crear un servidor, no se puede cambiar el tipo de redundancia elegido, redundancia geográfica o redundancia local.
>

Al crear un servidor mediante Azure Portal, en la ventana **Plan de tarifa** se selecciona **Redundancia local** o **Redundancia geográfica** para las copias de seguridad del servidor. En esta ventana se selecciona también el **Período de retención de copia de seguridad**, es decir, durante cuánto tiempo se almacenarán las copias de seguridad del servidor.

   :::image type="content" source="./media/howto-restore-server-portal/pricing-tier.png" alt-text="Plan de tarifa: elija la redundancia de las copias de seguridad":::

Para más información acerca de cómo establecer estos valores durante la creación, consulte la [guía de inicio rápido del servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md).

Para cambiar el período de retención de copia de seguridad en un servidor, siga estos pasos:
1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).
2. Seleccione un servidor de Azure Database for MySQL. Esta acción abre la página **Información general**.
3. Seleccione **Plan de tarifa** en el menú, en **Configuración**. Con el control deslizante puede cambiar el **Período de retención de copia de seguridad** entre 7 y 35 días.
En la captura de pantalla siguiente, se ha aumentado a 34 días.
:::image type="content" source="./media/howto-restore-server-portal/3-increase-backup-days.png" alt-text="Aumento del período de retención de copia de seguridad":::

4. Haga clic en **Aceptar** para confirmar el cambio.

El período de retención de copia de seguridad rige durante cuánto tiempo se puede realizar una restauración a un momento dado, porque se basa en las copias de seguridad disponibles. La restauración a un momento dado se describe con más detalle en la sección siguiente. 

## <a name="point-in-time-restore"></a>Restauración a un momento dado
Azure Database for MySQL permite restaurar el servidor a un momento dado y en una nueva copia del servidor. Puede usar este nuevo servidor para recuperar los datos o hacer que las aplicaciones cliente apunten a este nuevo servidor.

Por ejemplo, si una tabla se quitó accidentalmente a mediodía de hoy, podría restaurar al momento justo antes de mediodía y recuperar la tabla y los datos que faltan desde esa copia nueva del servidor. La restauración a un momento dado se realiza en el nivel del servidor, no en el nivel de base de datos.

Los siguientes pasos restauran el servidor de ejemplo a un momento dado:
1. En Azure Portal, seleccione el servidor de Azure Database for MySQL. 

2. En la barra de herramientas de la página **Información general** del servidor, seleccione **Restaurar**.

   :::image type="content" source="./media/howto-restore-server-portal/2-server.png" alt-text="Azure Database for MySQL - Información general - Botón Restaurar":::

3. Rellene el formulario Restaurar con la información necesaria:

   :::image type="content" source="./media/howto-restore-server-portal/3-restore.png" alt-text="Azure Database for MySQL - Información sobre restauración":::
   - **Punto de restauración**: seleccione el momento al que desea restaurar.
   - **Servidor de destino:** : proporcione un nombre para el nuevo servidor.
   - **Ubicación**: no se puede seleccionar la región. De manera predeterminada, es el mismo que el del servidor de origen.
   - **Plan de tarifa**: estos parámetros no se pueden cambiar al realizar una restauración a un momento dado. Es el mismo que el del servidor de origen. 

4. Haga clic en **Aceptar** para restaurar el servidor a un momento dado. 

5. Una vez finalizada la restauración, busque el nuevo servidor que se crea para comprobar que los datos se restauraron del modo esperado.

El nuevo servidor creado mediante restauración a un momento dado tiene el mismo nombre de inicio de sesión y contraseña de administrador del servidor que el servidor existente tenía en ese momento dado. Puede cambiar la contraseña en la página **Información general** del nuevo servidor.

Adicionalmente, después de que finalice la operación de restauración, hay dos parámetros de servidor que se restablecen a los valores predeterminados (y no se copian del servidor principal) después de la operación de restauración.
*   time_zone: este valor se establece en DEFAULT value **SYSTEM**
*   event_scheduler: este parámetro se establece en **OFF** en el servidor restaurado

Tendrá que copiar el valor del servidor principal y establecerlo en el servidor restaurado, para lo que tendrá que volver a configurar el [parámetro del servidor](howto-server-parameters.md)

El servidor creado durante una restauración no tiene los puntos de conexión de servicio de red virtual que existían en el servidor original. Estas reglas deben configurarse por separado para este nuevo servidor. Se restauran las reglas de firewall del servidor original.

## <a name="geo-restore"></a>Restauración geográfica
Si ha configurado el servidor para copias de seguridad con redundancia geográfica, se puede crear un nuevo servidor a partir de la copia de seguridad de ese servidor existente. Este nuevo servidor puede crearse en cualquier región en la que Azure Database for MySQL esté disponible.  

1. Seleccione el botón **Crear un recurso** (+) de la esquina superior izquierda del portal. Seleccione **Bases de datos** > **Azure Database for MySQL**.

   :::image type="content" source="./media/howto-restore-server-portal/1_navigate-to-mysql.png" alt-text="Desplazamiento a Azure Database for MySQL.":::
 
2. Proporcione la suscripción, el grupo de recursos y el nombre del nuevo servidor. 

3. Seleccione **Copia de seguridad** como el **Origen de datos**. Esta acción carga un menú desplegable en el que se proporciona una lista de servidores que tienen habilitadas copias de seguridad con redundancia geográfica.
   
   :::image type="content" source="./media/howto-restore-server-portal/3-geo-restore.png" alt-text="Seleccione el origen de datos.":::
    
   > [!NOTE]
   > Al crear por primera vez un servidor, puede que no esté disponible para la restauración geográfica inmediatamente. Los metadatos pueden tardar unas horas en rellenarse.
   >

4. Seleccione la lista desplegable **Copia de seguridad**.
   
   :::image type="content" source="./media/howto-restore-server-portal/4-geo-restore-backup.png" alt-text="Selección de la lista desplegable Copia de seguridad.":::

5. Seleccione el servidor de origen desde el que se va a restaurar.
   
   :::image type="content" source="./media/howto-restore-server-portal/5-select-backup.png" alt-text="Selección de la copia de seguridad.":::

6. El servidor usará de forma predeterminada los valores para el número de **Núcleos virtuales**, **Período de retención de copia de seguridad**, la opción **Redundancia de copia de seguridad**, la **versión del motor** y las **credenciales de administrador**. Seleccione **Continuar**. 
   
   :::image type="content" source="./media/howto-restore-server-portal/6-accept-backup.png" alt-text="Continuación con la copia de seguridad.":::

7. Rellene el resto del formulario con sus preferencias. Puede seleccionar cualquier valor en **Ubicación**.

    Después de seleccionar la ubicación, puede seleccionar **Configurar servidor** para actualizar **Generación de procesos** (si está disponible en la región que ha elegido), el número de **Núcleos virtuales**, **Período de retención de copia de seguridad** y la opción  **Redundancia de copia de seguridad**. No se permite cambiar el **Plan de tarifa** (Básico, Uso general o Memoria optimizada) ni el tamaño de **Almacenamiento** durante la restauración.

   :::image type="content" source="./media/howto-restore-server-portal/7-create.png" alt-text="Cumplimentación del formulario."::: 

8. Seleccione **Review + create** (Revisar y crear) para revisar las selecciones. 

9. Seleccione **Crear** para realizar el aprovisionamiento del servidor. Esta operación puede tardar algunos minutos.

El nuevo servidor creado mediante restauración geográfica tiene el mismo nombre de inicio de sesión y contraseña de administrador del servidor que el servidor existente tenía cuando se inició la restauración. La contraseña se puede cambiar en la página **Información general** del nuevo servidor.

El servidor creado durante una restauración no tiene los puntos de conexión de servicio de red virtual que existían en el servidor original. Estas reglas deben configurarse por separado para este nuevo servidor. Se restauran las reglas de firewall del servidor original.

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [copias de seguridad](concepts-backup.md) del servicio.
- Más información sobre las [réplicas](concepts-read-replicas.md).
- Más información sobre las opciones de [continuidad del negocio](concepts-business-continuity.md).