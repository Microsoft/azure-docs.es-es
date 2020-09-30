---
title: 'Azure AD Connect: Configuración de la ubicación de datos preferida para los recursos de Office 365'
description: Describe cómo poner los recursos de usuario de Office 365 cerca del usuario con la sincronización de Azure Active Directory Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/11/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ebd71eae77c75d490585c27d7bc04c069d006009
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398540"
---
# <a name="azure-active-directory-connect-sync-configure-preferred-data-location-for-office-365-resources"></a>Sincronización de Azure Active Directory Connect: Configuración de la ubicación de datos preferida para los recursos de Office 365
El objetivo de este tema es guiarle por la configuración del atributo de la ubicación de datos preferida en la sincronización de Azure Active Directory (Azure AD) Connect. Cuando alguien usa las funcionalidades multigeográficas de Office 365, este atributo se usa para designar la ubicación geográfica de los datos de Office 365 del usuario. (Los términos *región* y *geoárea* se usan indistintamente).

## <a name="enable-synchronization-of-preferred-data-location"></a>Habilitación de la sincronización de la ubicación de datos preferida
De forma predeterminada, los recursos de Office 365 para los usuarios se encuentran en la misma zona geográfica que el inquilino de Azure AD. For example, if your tenant is located in Norteamérica, then the users' Exchange mailboxes are also located in North America. Esto podría no ser adecuado para una organización multinacional.

Es posible definir la geoárea de un usuario si se establece el atributo **preferredDataLocation**. Puede tener los recursos de Office 365 del usuario, como el buzón de correo y OneDrive, en la misma geoárea que el usuario y, aún así, tener un inquilino para toda la organización.

> [!IMPORTANT]
> La replicación geográfica múltiple está disponible actualmente para los clientes con un Contrato Enterprise y un mínimo de 250 suscripciones a los servicios de Office 365. Póngase en contacto con su representante de Microsoft para más información.
>
>

Puede encontrar una lista de todas las geoáreas de Office 365 en [¿Dónde se encuentran tus datos?](https://aka.ms/datamaps)

Las zonas geográficas de Office 365 disponibles para la funcionalidad multigeográfica son:

| Geoárea | Valor de preferredDataLocation |
| --- | --- |
| Asia Pacífico | APC |
| Australia | AUS |
| Canadá | CAN |
| Unión Europea | EUR |
| Francia | FRA |
| India | IND |
| Japón | JPN |
| Corea | KOR |
| Sudáfrica | ZAF |
| Suiza | CHE |
| Emiratos Árabes Unidos | ARE |
| Reino Unido | GBR |
| Estados Unidos | NAM |

* Si una geoárea no aparece en esta tabla, por ejemplo, Sudamérica, no se puede usar con la funcionalidad multigeográfica.

* No todas las cargas de trabajo de Office 365 admiten la configuración de la geoárea de un usuario.

### <a name="azure-ad-connect-support-for-synchronization"></a>Compatibilidad de Azure AD Connect con la sincronización

Azure AD Connect es compatible con la sincronización del atributo **preferredDataLocation** para los objetos **User** en la versión 1.1.524.0 y posteriores. Concretamente:

* El esquema del tipo de objeto **User** en el conector de Azure AD se extiende para incluir el atributo **preferredDataLocation**. El atributo es del tipo "cadena de un solo valor".
* El esquema del tipo de objeto **Person** en el metaverso se extiende para incluir el atributo **preferredDataLocation**. El atributo es del tipo "cadena de un solo valor".

De manera predeterminada, el atributo **preferredDataLocation** no está habilitado para la sincronización. Esta característica está diseñada para organizaciones grandes. El esquema de Active Directory en Windows Server 2019 tiene un atributo **msDS-preferredDataLocation** que debe usar con este fin. Si no ha actualizado el esquema de Active Directory y no puede hacerlo, debe identificar un atributo para que contenga la geoárea de Office 365 para los usuarios. Esto será diferente para cada organización.

> [!IMPORTANT]
> Azure AD permite que el atributo **preferredDataLocation** de los **objetos User de la nube** se configure directamente con Azure AD PowerShell. Para configurar este atributo en los **objetos User sincronizado**, debe usar Azure AD Connect.

Antes de habilitar la sincronización:

* Si no ha actualizado el esquema de Active Directory a 2019, decida qué atributo de Active Directory local se usará como atributo de origen. Debería ser del tipo **cadena de un solo valor**.
* Si configuró previamente el atributo **preferredDataLocation** en **objetos User sincronizados** existentes de Azure AD con Azure AD PowerShell, deberá restituir los valores de atributo a los objetos **User** correspondientes de la instancia local de Active Directory.

    > [!IMPORTANT]
    > Si no restituye estos valores, Azure AD Connect quita los valores de atributo existentes en Azure AD cuando la sincronización del atributo **preferredDataLocation** está habilitada.

* Ahora configure el atributo de origen en al menos un par de objetos User de la instancia de Active Directory local. Puede usar esto para una posterior comprobación.

En las secciones siguientes se detallan los pasos para habilitar la sincronización del atributo **preferredDataLocation**.

> [!NOTE]
> Los pasos se describen en el contexto de una implementación de Azure AD con topología de bosque único y sin reglas de sincronización personalizadas. Si tiene una topología de varios bosques, reglas de sincronización personalizadas configuradas o un servidor de ensayo, debe ajustar los pasos según corresponda.

## <a name="step-1-disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>Paso 1: Deshabilitar el programador de sincronización y comprobar que no hay ninguna sincronización en curso
Para evitar que cualquier cambio no deseado se exporte a Azure AD, asegúrese de que no se realice ninguna sincronización mientras está realizando la actualización de las reglas de sincronización. Para deshabilitar el programador de sincronización integrado:

1. Inicie una sesión de PowerShell en el servidor de Azure AD Connect.
2. Deshabilite la sincronización programada mediante la ejecución de este cmdlet: `Set-ADSyncScheduler -SyncCycleEnabled $false`.
3. Inicie **Synchronization Service Manager**. Para ello, vaya a **INICIO** > **Servicio de sincronización**.
4. Seleccione la pestaña **Operaciones** y confirme que no hay ninguna operación cuyo estado sea *en curso*.

![Captura de pantalla de Synchronization Service Manager](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step1.png)

## <a name="step-2-refresh-the-schema-for-active-directory"></a>Paso 2: Actualizar el esquema para Active Directory
Si ha actualizado el esquema de Active Directory a 2019 y Connect se instaló antes de la extensión del esquema, la caché del esquema de Connect no cuenta con el esquema actualizado. Por tanto, debe actualizar el esquema desde el asistente para que aparezca en la interfaz de usuario.

1. Inicie el Asistente de Azure AD Connect desde el escritorio.
2. Seleccione la opción **Actualizar esquema de directorios** y haga clic en **Siguiente**.
3. Escriba las credenciales de Azure AD y haga clic en **Siguiente**.
4. En la página **Actualizar esquema de directorios**, asegúrese de que todos los bosques están seleccionados; después, haga clic en **Siguiente**.
5. Cuando termine, cierre el asistente.

![Captura de pantalla de Actualizar esquema de directorios en el asistente de Connect](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-refreshschema.png)

## <a name="step-3-add-the-source-attribute-to-the-on-premises-active-directory-connector-schema"></a>Paso 3: Agregar el atributo de origen al esquema del conector de Active Directory local
**Este paso solo es necesario si ejecuta la versión 1.3.21 o una versión anterior de Connect. Si utiliza la versión 1.4.18 o una versión más reciente, vaya al paso 5.**  
No todos los atributos de Azure AD se importan en el espacio del conector de Active Directory local. Si seleccionó usar un atributo no sincronizado de manera predeterminada, deberá importarlo. Para agregar el atributo de origen a la lista de atributos importados:

1. Seleccione la pestaña **Conectores** de Synchronization Service Manager.
2. Haga clic con el botón derecho en la instancia del conector de Active Directory local y seleccione **Propiedades**.
3. En el cuadro de diálogo emergente, vaya a la pestaña **Seleccionar atributos**.
4. Asegúrese de que el atributo de origen que seleccionó está activado en la lista de atributos. Si no ve el atributo, active la casilla de verificación **Mostrar todo**.
5. Seleccione **Aceptar** para guardar.

![Captura de pantalla de Synchronization Service Manager y el cuadro de diálogo Propiedades](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step2.png)

## <a name="step-4-add-preferreddatalocation-to-the-azure-ad-connector-schema"></a>Paso 4: Agregar **preferredDataLocation** al esquema del conector de Azure AD
**Este paso solo es necesario si ejecuta la versión 1.3.21 o una versión anterior de Connect. Si utiliza la versión 1.4.18 o una versión más reciente, vaya al paso 5.**  
De manera predeterminada, el atributo **preferredDataLocation** no se importa en el espacio del conector de Azure AD. Para agregarlo a la lista de los atributos importados:

1. Seleccione la pestaña **Conectores** de Synchronization Service Manager.
2. Haga clic con el botón derecho en el conector de Azure AD y seleccione **Propiedades**.
3. En el cuadro de diálogo emergente, vaya a la pestaña **Seleccionar atributos**.
4. Seleccione el atributo **preferredDataLocation** en la lista.
5. Seleccione **Aceptar** para guardar.

![Captura de pantalla de Synchronization Service Manager y el cuadro de diálogo Propiedades](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step3.png)

## <a name="step-5-create-an-inbound-synchronization-rule"></a>Paso 5: Crear una regla de sincronización de entrada
La regla de sincronización de entrada permite enviar el valor del atributo desde el atributo de origen de la instancia local de Active Directory al metaverso.

1. Inicie el **Editor de reglas de sincronización**. Para ello, vaya a **INICIO** > **Editor de reglas de sincronización**.
2. Establezca el filtro de búsqueda **Dirección** como **Entrada**.
3. Para crear una regla de entrada, seleccione **Agregar nueva regla**.
4. En la pestaña **Descripción**, proporcione la configuración siguiente:

    | Atributo | Value | Detalles |
    | --- | --- | --- |
    | Nombre | *Proporcione un nombre*. | Por ejemplo, "In from AD – User preferredDataLocation" |
    | Descripción | *Proporcione una descripción personalizada* |  |
    | Sistema conectado | *Elija el conector de Active Directory local* |  |
    | Tipo de objeto de sistema conectado | **User** |  |
    | Propiedades de objeto de metaverso | **Person** |  |
    | Tipo de vínculo | **Join** |  |
    | Prioridad | *Elija un número entre 1 y 99* | El intervalo de 1 a 99 se reserva para las reglas de sincronización personalizadas. No elija ningún valor que use otra regla de sincronización. |

5. Mantenga el **Filtro de ámbito** vacío para incluir todos los objetos. Puede que necesite retocar el filtro de ámbito en función de su implementación de Azure AD Connect.
6. Vaya a la pestaña **Transformación** e implemente la siguiente regla de transformación:

    | Tipo de flujo | Atributo de destino | Source | Aplicar una vez | Tipo de combinación |
    | --- | --- | --- | --- | --- |
    |Directo | preferredDataLocation | Elegir el atributo de origen | No activado | Actualizar |

7. Para crear la regla de entrada, seleccione **Agregar**.

![Captura de pantalla de Crear regla de sincronización de entrada](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step4.png)

## <a name="step-6-create-an-outbound-synchronization-rule"></a>Paso 6: Crear una regla de sincronización de salida
La regla de sincronización de salida permite enviar el valor del atributo desde el metaverso al atributo **preferredDataLocation** en Azure AD:

1. Vaya al **Editor de Reglas de sincronización**.
2. Establezca el filtro de búsqueda **Dirección** como **Salida**.
3. Seleccione **Agregar nueva regla**.
4. En la pestaña **Descripción**, proporcione la configuración siguiente:

    | Atributo | Value | Detalles |
    | ----- | ------ | --- |
    | Nombre | *Proporcione un nombre*. | Por ejemplo, "Out to Azure AD – User preferredDataLocation" |
    | Descripción | *Proporcione una descripción* ||
    | Sistema conectado | *Seleccione Azure AD Connector* ||
    | Tipo de objeto de sistema conectado | **User** ||
    | Propiedades de objeto de metaverso | **Person** ||
    | Tipo de vínculo | **Join** ||
    | Prioridad | *Elija un número entre 1 y 99* | El intervalo de 1 a 99 se reserva para las reglas de sincronización personalizadas. No elija ningún valor que use otra regla de sincronización. |

5. Vaya a la pestaña **Filtro de ámbito** y agregue un solo grupo de filtro de ámbito con dos cláusulas:

    | Atributo | Operator | Value |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | Usuario |
    | cloudMastered | NOTEQUAL | True |

    El filtro de ámbito determina a qué objetos de Azure AD se aplica esta regla de sincronización de salida. En este ejemplo, se usa el mismo filtro de ámbito de la regla de sincronización integrada "Out to Azure AD – User Identity". Impide que la regla de sincronización se aplique a los objetos **User** que no se sincronizan desde una instancia local de Active Directory. Puede que necesite retocar el filtro de ámbito en función de su implementación de Azure AD Connect.

6. Vaya a la pestaña **Transformación** e implemente la siguiente regla de transformación:

    | Tipo de flujo | Atributo de destino | Source | Aplicar una vez | Tipo de combinación |
    | --- | --- | --- | --- | --- |
    | Directo | preferredDataLocation | preferredDataLocation | No activado | Actualizar |

7. Haga clic en **Agregar** para crear la regla de salida.

![Captura de pantalla de Crear regla de sincronización de salida](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step5.png)

## <a name="step-7-run-full-synchronization-cycle"></a>Paso 7: Ejecutar el ciclo de sincronización completo
En general, se requiere un ciclo de sincronización completo. Esto se debe a que agregó atributos nuevos a los esquemas de los conectores de Azure AD y de Active Directory e introdujo reglas de sincronización personalizadas. Compruebe los cambios antes de exportarlos a Azure AD. Puede usar los pasos siguientes para comprobar los cambios mientras sigue los pasos que forman un ciclo de sincronización completo manualmente.

1. Ejecute **Importación completa** en el conector de Active Directory local:

   1. Vaya a la pestaña **Operations** (Operaciones) de Synchronization Service Manager.
   2. Haga clic con el botón derecho en el **conector de Active Directory local** y seleccione **Ejecutar**.
   3. En el cuadro de diálogo, seleccione **Importación completa** y seleccione **Aceptar**.
   4. Espere a que se complete la operación.

      > [!NOTE]
      > Puede omitir la importación completa en el conector de Active Directory local si el atributo de origen ya está incluido en la lista de atributos importados. En otras palabras, no debería haber hecho ningún cambio en el paso 2 de este artículo.

2. Ejecute una **importación completa** en el conector de Azure AD:

   1. Haga clic con el botón derecho en **Conector de Azure AD** y seleccione **Ejecutar**.
   2. En el cuadro de diálogo, seleccione **Importación completa** y seleccione **Aceptar**.
   3. Espere a que se complete la operación.

3. Compruebe los cambios de la regla de sincronización en un objeto **User** existente.

   El atributo de origen de la instancia local de Active Directory y **preferredDataLocation** de Azure AD se han importado en cada espacio del conector correspondiente. Antes de continuar con el paso de sincronización completa, obtenga una vista previa de un objeto **User** existente en el espacio del conector de Active Directory local. El objeto que ha seleccionado debe tener el atributo de origen rellenado. Una vista previa correcta con el valor **preferredDataLocation** rellenado en el metaverso es un buen indicador de que configuró correctamente las reglas de sincronización. Para información sobre cómo obtener una vista previa, consulte [Comprobación del cambio](how-to-connect-sync-change-the-configuration.md#verify-the-change).

4. Ejecute una **sincronización completa** en el conector de Active Directory local:

   1. Haga clic con el botón derecho en el **conector de Active Directory local** y seleccione **Ejecutar**.
   2. En el cuadro de diálogo, seleccione **Sincronización completa** y **Aceptar**.
   3. Espere a que se complete la operación.

5. Compruebe las **exportaciones pendientes** en Azure AD:

   1. Haga clic con el botón derecho en **Conector de Azure AD** y seleccione **Espacio del conector de búsqueda**.
   2. En el cuadro de diálogo **Search Connector Space** (Espacio del conector de búsqueda):

        a. Establezca el **ámbito** en **Pending Export** (Exportación pendiente).<br>
        b. Active las tres casillas, incluidas **Agregar, Modificar y Eliminar**.<br>
        c. Para ver la lista de objetos con cambios que se deben exportar, seleccione **Buscar**. Para examinar los cambios de un determinado objeto, haga doble clic en el objeto.<br>
        d. Compruebe que los cambios sean los previstos.

6. Ejecute **Exportar** en el **conector de Azure AD**

   1. Haga clic con el botón derecho en **Conector de Azure AD** y seleccione **Ejecutar**.
   2. En el cuadro de diálogo **Ejecutar conector**, seleccione **Exportar** y **Aceptar**.
   3. Espere a que se complete la operación.

> [!NOTE]
> Es posible que observe que los pasos no incluyen el paso de sincronización completa en el conector de Azure AD ni el paso de exportación en el conector de Active Directory. Los pasos no son necesarios, ya que los valores de atributo solo se envían desde la instancia local de Active Directory a Azure AD.

## <a name="step-8-re-enable-sync-scheduler"></a>Paso 8: Volver a habilitar el programador de sincronización
Vuelva a habilitar el programador de sincronización integrado:

1. Inicie una sesión de PowerShell.
2. Vuelva a habilitar la sincronización programada mediante la ejecución de este cmdlet: `Set-ADSyncScheduler -SyncCycleEnabled $true`

## <a name="step-9-verify-the-result"></a>Paso 9: Comprobar el resultado
Ahora es el momento de comprobar la configuración y habilitarla para los usuarios.

1. Agregue la zona geográfica al atributo seleccionado en un usuario. La lista de geoáreas disponibles se puede encontrar en esta tabla.  
![Captura de pantalla del atributo de AD agregado a un usuario](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-adattribute.png)
2. Espere a que el atributo se sincronice con Azure AD.
3. Con Exchange Online PowerShell, compruebe que la región del buzón de correo se ha configurado correctamente.  
![Captura de pantalla de Exchange Online PowerShell](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-mailboxregion.png)  
Suponiendo que el inquilino se haya marcado para que pueda usar esta característica, el buzón de correo se mueve a la zona geográfica correcta. Esto se puede comprobar examinando el nombre del servidor donde se encuentra el buzón.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las funcionalidades multigeográficas de Office 365:

* [Sesiones sobre funcionalidades multigeográficas en Ignite](https://aka.ms/MultiGeoIgnite)
* [Funcionalidades multigeográficas en OneDrive](https://aka.ms/OneDriveMultiGeo)
* [Funcionalidades multigeográficas en SharePoint Online](https://aka.ms/SharePointMultiGeo)

Más información sobre el modelo de configuración del motor de sincronización:

* Obtenga más información sobre el modelo de configuración en el artículo de información sobre el [aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning.md).
* Consulte más detalles sobre el lenguaje de expresiones en el artículo [Descripción de las expresiones de aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md).

Temas de introducción:

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)
