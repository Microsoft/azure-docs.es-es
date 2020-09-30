---
title: Supervisar la integridad de los archivos en Azure Security Center | Microsoft Docs
description: Aprenda a configurar la supervisión de la integridad de los archivos (FIM) en Azure Security Center mediante este tutorial.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/13/2019
ms.author: memildin
ms.openlocfilehash: 9e8bd56655adfa1f7cdb769ac6cd282193b1bcf2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90901379"
---
# <a name="file-integrity-monitoring-in-azure-security-center"></a>Supervisar la integridad de los archivos en Azure Security Center
Aprenda a configurar la supervisión de la integridad de los archivos (FIM) en Azure Security Center mediante este tutorial.


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).|
|Roles y permisos necesarios:|El **propietario del área de trabajo** puede habilitar o deshabilitar FIM (para obtener más información, consulte [Roles de Azure para Log Analytics](https://docs.microsoft.com/services-hub/health/azure-roles#azure-roles)).<br>El **lector** puede ver los resultados.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||





## <a name="what-is-fim-in-security-center"></a>¿Qué la supervisión de la integridad de los archivos (FIM) Security Center?
La supervisión de la integridad de los archivos (FIM), conocida también como supervisión de los cambios, examina los archivos y los registros del sistema operativo, el software de aplicación y demás para comprobar la existencia de cambios que podrían indicar un ataque. Para determinar si el estado actual del archivo es diferente del último examen del archivo, se usa un método de comparación. Puede aprovechar esta comparación para determinar si se han realizado modificaciones sospechosas o válidas en los archivos.

La supervisión de la integridad de los archivos de Security Center valida la integridad de los archivos de Windows, el Registro de Windows y los archivos de Linux. Seleccionará los archivos que quiere supervisar mediante la habilitación de FIM. Security Center supervisa los archivos con FIM habilitado en busca de actividades como:

- Eliminación y creación de archivos y del Registro
- Modificaciones de archivos (cambios en el tamaño de archivo, listas de control de acceso y hash del contenido)
- Modificaciones del Registro (cambios en el tamaño, listas de control de acceso, tipo y contenido)

Security Center recomienda entidades para supervisar, en las que se pueda habilitar fácilmente FIM. También puede definir sus propias directivas o entidades de FIM que quiere supervisar. En este tutorial se muestra cómo hacerlo.

> [!NOTE]
> La característica de supervisión de la integridad de los archivos (FIM) funciona con equipos y máquinas virtuales Windows y Linux, y está disponible solo si está habilitado **Azure Defender para servidores**. Consulte [Precios](security-center-pricing.md) para obtener más información. FIM carga datos en el área de trabajo de Log Analytics. Se aplican cargos de datos, según la cantidad de datos que cargue. Para más información, consulte [Precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).

FIM emplea la solución Azure Change Tracking para realizar el seguimiento de los cambios e identificarlos en su entorno. Cuando la supervisión de la integridad de los archivos está habilitada, tendrá un recurso **Change Tracking** de tipo **Solución**. Para obtener detalles sobre la frecuencia de recopilación de datos, vea [Detalles de la recopilación de datos de seguimiento de cambios](https://docs.microsoft.com/azure/automation/automation-change-tracking#change-tracking-data-collection-details) para Azure Change Tracking.

> [!NOTE]
> Si quita el recurso **Change Tracking**, deshabilitará también la característica de supervisión de la integridad de los archivos en Security Center.

## <a name="which-files-should-i-monitor"></a>¿Qué archivos se deben supervisar?
Al elegir los archivos que quiere supervisar, es conveniente que piense en los archivos que son críticos para su sistema y aplicaciones. Considere la posibilidad de elegir archivos que no espera que cambien sin haberlo planeado. Al elegir archivos que las aplicaciones o el sistema operativo cambian con frecuencia (por ejemplo, archivos de registro y archivos de texto) se genera mucho ruido que dificulta la identificación de un ataque.

Security Center proporciona la siguiente lista de elementos cuya supervisión se recomienda según los patrones de ataque conocidos. Aquí se incluyen archivos y claves del Registro de Windows. Todas las claves están en HKEY_LOCAL_MACHINE ("HKLM" en la tabla).

|**Archivos de Linux**|**Archivos de Windows**|**Claves del Registro de Windows**|
|:----|:----|:----|
|/bin/login|C:\autoexec.bat|HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}|
|/bin/passwd|C:\boot.ini|HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{603BCC1F-4B59-4E08-B724-D2C6297EF351}|
|/etc/*.conf|C:\config.sys|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\IniFileMapping\SYSTEM.ini\boot|
|/usr/bin|C:\Windows\system.ini|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows|
|/usr/sbin|C:\Windows\win.ini|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon|
|/bin|C:\Windows\regedit.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders|
|/sbin|C:\Windows\System32\userinit.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders|
|/boot|C:\Windows\explorer.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run|
|/usr/local/bin|C:\Archivos de programa\Microsoft Security Client\msseces.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce|
|/usr/local/sbin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx|
|/opt/bin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices|
|/opt/sbin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServicesOnce|
|/etc/crontab||HKLM\SOFTWARE\WOW6432Node\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}|
|/etc/init.d||HKLM\SOFTWARE\WOW6432Node\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{603BCC1F-4B59-4E08-B724-D2C6297EF351}|
|/etc/cron.hourly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\IniFileMapping\system.ini\boot|
|/etc/cron.daily||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Windows|
|/etc/cron.weekly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Winlogon|
|/etc/cron.monthly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnce|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunServices|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce|
|||HKLM\SYSTEM\CurrentControlSet\Control\hivelist|
|||HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\DomainProfile|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\PublicProfile|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\StandardProfile|


## <a name="using-file-integrity-monitoring"></a>Uso de la supervisión de la integridad de los archivos

1. Abra el panel de **Azure Defender**.

1. En el área **Protección avanzada**, seleccione **Supervisión de la integridad de los archivos**.

    :::image type="content" source="./media/security-center-file-integrity-monitoring/open-file-integrity-monitoring.png" alt-text="Inicio de FIM" lightbox="./media/security-center-file-integrity-monitoring/open-file-integrity-monitoring.png":::


    Se abre la hoja **Supervisión de la integridad de los archivos**.
    ![Panel de Security Center][2]

Se proporciona la siguiente información para cada área de trabajo:

- Número total de cambios que se produjeron durante la última semana (puede ver un guion "-" si FIM no está habilitado en el área de trabajo)
- Número total de equipos y las máquinas virtuales que informan al área de trabajo
- Ubicación geográfica del área de trabajo
- Suscripción de Azure en la que se encuentra el área de trabajo

También se pueden mostrar los siguientes botones en un área de trabajo:

- ![Icono Habilitar][3] Indica que FIM no está habilitada en el área de trabajo. Al seleccionar el área de trabajo puede habilitar FIM en todas las máquinas del área de trabajo.
- ![Icono de Actualizar plan][4] Indica que el área de trabajo o la suscripción no están protegidas por Azure Defender. Para usar la característica FIM, la suscripción debe estar protegida por Azure Defender.  La selección del área de trabajo permite actualizar.
- Un espacio en blanco (no hay ningún botón) significa que FIM ya está habilitado en el área de trabajo.

En **Supervisión de la integridad de los archivos**, puede seleccionar un área de trabajo en la que habilitar FIM, ver el panel de supervisión de la integridad de los archivos correspondiente a ese área de trabajo o [actualizar](security-center-pricing.md) el área de trabajo para usar Azure Defender.

## <a name="enable-fim"></a>Habilitación de FIM
Para habilitar FIM en un área de trabajo, siga estos pasos:

1. En **Supervisión de la integridad de los archivos**, seleccione un área de trabajo con el botón **Habilitar**.

1. Se abre la página **Habilitar supervisión de la integridad de los archivos** y muestra el número de máquinas Windows y Linux en el área de trabajo.

   ![Habilitación de la supervisión de la integridad de los archivos][5]

   También se muestra la configuración recomendada para Windows y Linux.  Expanda **Archivos de Windows**, **Registro** y **Archivos de Linux** para ver la lista completa de elementos recomendados.

1. Desactive todas las entidades recomendadas a las que no desea aplicar FIM.

1. Seleccione **Aplicar la supervisión de la integridad de los archivos** para habilitar FIM.

> [!NOTE]
> Puede cambiar la configuración en cualquier momento. Consulte a continuación [Edición de las entidades supervisadas](#edit-monitored-entities) para más información.


## <a name="view-the-fim-dashboard"></a>Visualización del panel de FIM
El panel **Supervisión de la integridad de los archivos** se muestra en las áreas de trabajo en las que está habilitado FIM. El panel de FIM se abre después de habilitar FIM en un área de trabajo o al seleccionar un área de trabajo en la ventana **Supervisión de la integridad de los archivos** que ya tiene FIM habilitado.

![Panel de Supervisión de la integridad de los archivos][6]

El panel de FIM de un área de trabajo muestra los detalles siguientes:

- Número total de máquinas conectadas al área de trabajo
- Número total de cambios que se produjeron durante el período de tiempo seleccionado
- Un desglose del tipo de cambio (archivos, registro)
- Un desglose de la categoría del cambio (modificado, agregado, quitado)

Al seleccionar Filtrar en la parte superior del panel, puede aplicar el período de tiempo del que desea ver los cambios.

![Filtro de período de tiempo][7]

La pestaña **Equipos** (que se muestra arriba) enumera todas las máquinas que informan a este área de trabajo. En el panel se muestra para cada máquina:

- Número total de cambios que se produjeron durante el período de tiempo seleccionado
- Un desglose de los cambios totales a medida que se producen cambios en los archivos o el registro

La hoja **Búsqueda de registros** se abre cuando escribe el nombre de una máquina en el campo de búsqueda o selecciona una máquina que aparecen en la pestaña Equipos. En Búsqueda de registros se muestran todos los cambios realizados durante el período de tiempo seleccionado para la máquina. Los cambios se pueden expandir para obtener más información.

![Búsqueda de registros][8]

La pestaña **Cambios** (se muestra a continuación) enumera todos los cambios del área de trabajo durante el período de tiempo seleccionado. En el panel se muestra para las entidades que se han cambiado:

- Equipo en el que se produjo el cambio
- Tipo de cambio (registro o archivo)
- Categoría del cambio (modificado, agregado, eliminado)
- Fecha y hora del cambio

![Cambios del área de trabajo][9]

Se abre la hoja **Cambiar detalles** al escribir un cambio en el campo de búsqueda o al seleccionar una entidad enumerada en la pestaña **Cambios**.

![Cambiar detalles][10]

## <a name="edit-monitored-entities"></a>Edición de las entidades supervisadas

1. Vuelva al panel **Supervisión de la integridad de los archivo** y seleccione **Configuración**.

   ![Configuración][11]

   Se abre la hoja **Configuración del área de trabajo** que muestra tres pestañas: **Registro de Windows**, **Archivos de Windows** y **Archivos de Linux**. En cada pestaña se muestran las entidades que se pueden editar de esa categoría. En cada entidad de la lista, Security Center identifica si FIM está habilitado (true) o deshabilitado (false).  Al editar la entidad, puede habilitar o deshabilitar FIM.

   ![Configuración del área de trabajo][12]

2. Seleccione una protección de identidades. En este ejemplo se selecciona un elemento en el Registro de Windows. Se abre la hoja **Edit for Change Tracking** (Editar para Change Tracking).

   ![Editar para Change Tracking][13]

En **Edit for Change Tracking** (Editar para Change Tracking), puede:

- Habilitar (true) o deshabilitar (false) la supervisión de la integridad de los archivos
- Proporcionar o cambiar el nombre de la entidad
- Proporcionar o cambiar el valor o la ruta de acceso
- Eliminar la entidad, descarta el cambio o guardar el cambio

## <a name="add-a-new-entity-to-monitor"></a>Agregar una nueva entidad para supervisar
1. Vuelva al panel **Supervisión de la integridad de los archivo** y seleccione **Configuración** en la parte superior. Se abre la hoja **Configuración del área de trabajo**.
2. En **Configuración del área de trabajo**, seleccione la pestaña correspondiente al tipo de entidad que quiere agregar: Registro de Windows, Archivos de Windows o Archivos de Linux. En este ejemplo, hemos seleccionado **Archivos de Linux**.

   ![Adición de un nuevo elemento para supervisar][14]

3. Seleccione **Agregar**. Se abre la hoja **Add for Change Tracking** (Agregar para Change Tracking).

   ![Escribir la información solicitada][15]

4. En la página **Agregar**, escriba la información solicitada y seleccione **Guardar**.

## <a name="disable-monitored-entities"></a>Deshabilitación de las entidades supervisadas
1. Vuelva a al panel **Supervisión de la integridad de los archivos**.
2. Seleccione un área de trabajo donde esté habilitado actualmente FIM. Un área de trabajo está habilitada para FIM si le falta el botón Habilitar o Actualizar plan.

   ![Seleccionar un área de trabajo donde esté habilitado FIM][16]

3. En Supervisión de la integridad de los archivos, seleccione **Configuración**.

   ![Seleccionar Configuración][17]

4. En **Configuración del área de trabajo**, seleccione un grupo donde **Habilitado** esté establecido en True.

   ![Configuración del área de trabajo][18]

5. En **Edit for Change Tracking** (Editar para Change Tracking), establezca **Habilitado** en False.

   ![Establecer Habilitado en False][19]

6. Seleccione **Guardar**.

## <a name="folder-and-path-monitoring-using-wildcards"></a>Supervisión de carpetas y rutas de acceso mediante caracteres comodín.

Utilice caracteres comodín para simplificar el seguimiento a través de directorios. Las siguientes reglas se aplican cuando configura la supervisión de carpetas mediante caracteres comodín:
-   Los caracteres comodín son necesarios para realizar el seguimiento de varios archivos.
-   Los caracteres comodín solo se pueden usar en el último segmento de una ruta de acceso, como C:\folder\file o /etc/*.conf.
-   Si una variable de entorno tiene una ruta de acceso no válida, la validación será correcta, pero se producirá un error en dicha ruta cuando se ejecute el inventario.
-   Al establecer la ruta de acceso, evite las rutas de acceso generales como c:\*.*, ya que daría lugar a que se recorrieran demasiadas carpetas.

## <a name="disable-fim"></a>Deshabilitación de FIM
Puede deshabilitar FIM. FIM emplea la solución Azure Change Tracking para realizar el seguimiento de los cambios e identificarlos en su entorno. Al deshabilitar FIM, quita la solución Change Tracking del área de trabajo seleccionada.

1. Para deshabilitar FIM, vuelva al panel **Supervisión de la integridad de los archivos**.
2. Seleccione un área de trabajo.
3. En **Supervisión de la integridad de los archivos**, seleccione **Deshabilitar**.

   ![Deshabilitación de FIM][20]

4. Seleccione **Quitar** para deshabilitarlo.

## <a name="next-steps"></a>Pasos siguientes
En este artículo ha obtenido información sobre cómo usar la Supervisión de la integridad de los archivos (FIM) de Security Center. Para obtener más información sobre Security Center, vea las páginas siguientes:

* [Establecimiento de directivas de seguridad](tutorial-security-policy.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
* [Administración de recomendaciones de seguridad](security-center-recommendations.md): conozca una serie de recomendaciones que le ayudarán a proteger los recursos de Azure.
* [Blog de seguridad de Azure](https://docs.microsoft.com/archive/blogs/azuresecurity/): obtenga las últimas noticias e información sobre la seguridad en Azure.

<!--Image references-->
[1]: ./media/security-center-file-integrity-monitoring/security-center-dashboard.png
[2]: ./media/security-center-file-integrity-monitoring/file-integrity-monitoring.png
[3]: ./media/security-center-file-integrity-monitoring/enable.png
[4]: ./media/security-center-file-integrity-monitoring/upgrade-plan.png
[5]: ./media/security-center-file-integrity-monitoring/enable-fim.png
[6]: ./media/security-center-file-integrity-monitoring/fim-dashboard.png
[7]: ./media/security-center-file-integrity-monitoring/filter.png
[8]: ./media/security-center-file-integrity-monitoring/log-search.png
[9]: ./media/security-center-file-integrity-monitoring/changes-tab.png
[10]: ./media/security-center-file-integrity-monitoring/change-details.png
[11]: ./media/security-center-file-integrity-monitoring/fim-dashboard-settings.png
[12]: ./media/security-center-file-integrity-monitoring/workspace-config.png
[13]: ./media/security-center-file-integrity-monitoring/edit.png
[14]: ./media/security-center-file-integrity-monitoring/add.png
[15]: ./media/security-center-file-integrity-monitoring/add-item.png
[16]: ./media/security-center-file-integrity-monitoring/fim-dashboard-disable.png
[17]: ./media/security-center-file-integrity-monitoring/fim-dashboard-settings-disabled.png
[18]: ./media/security-center-file-integrity-monitoring/workspace-config-disable.png
[19]: ./media/security-center-file-integrity-monitoring/edit-disable.png
[20]: ./media/security-center-file-integrity-monitoring/disable-fim.png