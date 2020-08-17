---
title: 'Sincronización de Azure AD Connect: Configuración de filtrado | Microsoft Docs'
description: Explica cómo configurar el filtrado en Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 880facf6-1192-40e9-8181-544c0759d506
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/26/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1879df40122549ddc4c57557017fa2c84c883368
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88061513"
---
# <a name="azure-ad-connect-sync-configure-filtering"></a>Sincronización de Azure AD Connect: Configuración del filtrado
Con el filtrado puede controlar qué objetos aparecen en Azure Active Directory (Azure AD) desde el directorio local. La configuración predeterminada aceptará todos los objetos en todos los dominios de los bosques configurados. Por lo general, esta configuración es la recomendada. Los usuarios con cargas de trabajo de Office 365, como Exchange Online y Skype Empresarial, se benefician de una lista global de direcciones completa para poder enviar correo electrónico y llamar a todos los integrantes. Con la configuración predeterminada, obtendrían la misma experiencia que con una implementación local de Exchange o Lync.

Sin embargo, en algunos casos se necesitan algunos cambios en la configuración predeterminada. Estos son algunos ejemplos:

* Quiere usar la [topología de varios directorios de Azure AD](plan-connect-topologies.md#each-object-only-once-in-an-azure-ad-tenant). Después, necesita aplicar un filtro para controlar qué objetos se sincronizan en un directorio determinado de Azure AD.
* Va a ejecutar un piloto para Azure u Office 365 y desea solo un subconjunto de usuarios en Azure AD. En el pequeño proyecto piloto, no es necesaria una lista global de direcciones completa para demostrar la funcionalidad.
* Tiene numerosas cuentas de servicio y otras de carácter no personal que no desea en Azure AD.
* Por motivos de cumplimiento, no elimina ninguna cuenta de usuario local. Solo las deshabilita. Pero solo quiere encontrar las cuentas activas en Azure AD.

En este artículo, se trata cómo configurar los distintos métodos de filtrado.

> [!IMPORTANT]
> Microsoft no admite la modificación ni el funcionamiento de la sincronización de Azure AD Connect con acciones distintas a estas que se documentan formalmente. Cualquiera de estas acciones pueden provocar un estado incoherente o incompatible de sincronización de Azure AD Connect. Como resultado, Microsoft no ofrece soporte técnico para estas implementaciones.

## <a name="basics-and-important-notes"></a>Conceptos básicos y notas importantes
En Azure AD Connect Sync, puede habilitar el filtrado en cualquier momento. Si comienza con una configuración predeterminada de la sincronización de directorios y después configura el filtrado, los objetos que se filtren ya no se sincronizan con Azure AD. Por este cambio, en Azure AD se eliminan los objetos de Azure AD que se sincronizaran previamente pero que se filtraran después.

Antes de empezar a realizar cambios en el filtrado, asegúrese de [deshabilitar la tarea programada](#disable-the-scheduled-task) para no exportar cambios cuya corrección aún no se ha comprobado por error.

Como el filtrado puede quitar varios objetos al mismo tiempo, querrá asegurarse de que los nuevos filtros sean correctos antes de exportar los cambios a Azure AD. Después de completar los pasos de configuración, se recomienda encarecidamente seguir los [pasos de comprobación](#apply-and-verify-changes) antes de exportar y realizar cambios en Azure AD.

Para evitar eliminar muchos objetos por error, la característica para [evitar eliminaciones por error](how-to-connect-sync-feature-prevent-accidental-deletes.md) está activada de forma predeterminada. Si elimina muchos objetos debido al filtrado (de forma predeterminada, 500), debe seguir los pasos descritos en este artículo para permitir que las eliminaciones se reflejen en Azure AD.

Si utiliza una compilación anterior a la de noviembre de 2015 ([1.0.9125](reference-connect-version-history.md)), realice un cambio en la configuración de un filtro y utilice la sincronización de hash de contraseñas; una vez terminada la configuración, deberá desencadenar una sincronización completa de todas las contraseñas. Para conocer los pasos para desencadenar una sincronización completa de las contraseñas, consulte [Desencadenamiento de una sincronización completa de todas las contraseñas](tshoot-connect-password-hash-synchronization.md#trigger-a-full-sync-of-all-passwords). Si usa la compilación 1.0.9125 o una posterior, la acción de **sincronización completa** normal también calcula si se deben sincronizar las contraseñas, por lo que no se requiere este paso adicional.

Si los objetos de **usuario** se eliminaron involuntariamente en Azure AD por un error de filtrado, puede volver a crearlos en Azure AD al quitar las configuraciones de filtrado. A continuación, podrá volver a sincronizar los directorios. Esta acción restaura los usuarios de la papelera de reciclaje en Azure AD. Sin embargo, no se pueden recuperar otros tipos de objeto. Por ejemplo, si elimina por error un grupo de seguridad que se usara para incluir un recurso en una ACL, no podrá recuperar ni el grupo ni sus ACL.

Azure AD Connect solo elimina objetos que alguna vez haya considerado dentro del ámbito. Si hay objetos en Azure AD que se crearon con otro motor de sincronización y que no estén dentro del ámbito, no se quitarán al agregar el filtrado. Por ejemplo, si comienza con un servidor DirSync que creó una copia completa de todo el directorio en Azure AD e instala un nuevo servidor de sincronización de Azure AD Connect en paralelo con el filtrado habilitado desde el principio, Azure AD Connect no quita los objetos adicionales creados por DirSync.

Se conservan las configuraciones de filtrado cuando instale o actualice a una versión más reciente de Azure AD Connect. Es siempre un procedimiento recomendado comprobar que la configuración no se haya cambiado por error después de una actualización a una nueva versión antes de ejecutar el primer ciclo de sincronización.

Si tiene más de un bosque, debe aplicar las configuraciones de filtrado que se describen en este tema a cada bosque (suponiendo que quiera la misma configuración para todos).

### <a name="disable-the-scheduled-task"></a>Deshabilitación de la tarea programada
Para deshabilitar el programador integrado que desencadena un ciclo de sincronización cada 30 horas, siga estos pasos:

1. Vaya a un símbolo del sistema de PowerShell.
2. Ejecute `Set-ADSyncScheduler -SyncCycleEnabled $False` para deshabilitar el programador.
3. Realice los cambios que se documentan en este artículo.
4. Ejecute `Set-ADSyncScheduler -SyncCycleEnabled $True` para volver a habilitar el programador.

**Si usa una compilación de Azure AD Connect anterior a 1.1.105.0**  
Para deshabilitar la tarea programada que desencadena un ciclo de sincronización cada tres horas, siga estos pasos:

1. Abra el **Programador de tareas** en el menú **Inicio**.
2. Directamente en **Biblioteca del Programador de tareas**, busque la tarea **Programador de sincronización de Azure AD**, haga clic con el botón derecho en ella y seleccione **Deshabilitar**.  
   ![Programador de tareas](./media/how-to-connect-sync-configure-filtering/taskscheduler.png)  
3. Ahora podrá realizar cambios de configuración y ejecutar el motor de sincronización manualmente desde la consola de **Synchronization Service Manager**.

Después de completar todos los cambios de filtrado, no olvide volver a seleccionar **Habilitar** para la tarea.

## <a name="filtering-options"></a>Opciones de filtrado
Puede aplicar los siguientes tipos de configuración de filtrado a la herramienta de sincronización de directorios:

* [**Por grupo**](#group-based-filtering): el filtrado por grupo único solo se puede configurar durante la instalación inicial con el Asistente para instalación.
* [**Por dominio**](#domain-based-filtering): esta opción permite seleccionar los dominios que se sincronizarán con Azure AD. También permite agregar y quitar dominios de la configuración del motor de sincronización al realizar cambios en la infraestructura local después de instalar la sincronización de Azure AD Connect.
* [**Por unidad organizativa**](#organizational-unitbased-filtering): esta opción permite seleccionar las unidades organizativas que se sincronizarán con Azure AD. Esta opción está en todos los tipos de objeto de las unidades organizativas seleccionadas.
* [**Por atributo**](#attribute-based-filtering): esta opción permite filtrar objetos en función de los valores de sus atributos. También puede tener filtros diferentes para distintos tipos de objetos.

Puede usar varias opciones de filtrado al mismo tiempo. Por ejemplo, puede usar el filtrado por unidad organizativa para incluir solo los objetos de una unidad organizativa. Al mismo tiempo, puede usar el filtrado por atributo para filtrar los objetos aún más. Cuando se usan varios métodos de filtrado, los filtros utilizan el operador lógico "AND" entre los filtros.

## <a name="domain-based-filtering"></a>Filtrado basado en dominios
En esta sección se proporcionan los pasos para configurar el filtro de dominios. Si agregó o quitó dominios del bosque después de instalar Azure AD Connect, también tendrá que actualizar la configuración del filtrado.

La mejor manera de cambiar el filtrado por dominio es ejecutar el Asistente para instalación y cambiar el [filtrado por dominio y unidad organizativa](how-to-connect-install-custom.md#domain-and-ou-filtering). El Asistente para instalación automatiza todas las tareas que se documentan en este tema.

Solo debe seguir estos pasos si por alguna razón no se puede ejecutar el Asistente para instalación.

La configuración del filtrado basado en dominios consta de estos pasos:

1. Seleccione los dominios que desee incluir en la sincronización.
2. En cada dominio agregado o quitado, ajuste de los perfiles de ejecución.
3. [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

### <a name="select-the-domains-to-be-synchronized"></a>Selección de los dominios que se van a sincronizar
Existen dos formas de seleccionar los dominios que se van a sincronizar:
    - Uso del servicio de sincronización
    - Uso del asistente de Azure AD Connect.


#### <a name="select-the-domains-to-be-synchronized-using-the-synchronization-service"></a>Selección de los dominios que se van a sincronizar mediante el servicio de sincronización
Para configurar el filtro de dominio, realice los pasos siguientes:

1. Inicie sesión en el servidor donde se ejecuta la sincronización de Azure AD Connect con una cuenta que pertenezca al grupo de seguridad **ADSyncAdmins** .
2. Inicie el **servicio de sincronización** desde el menú **Inicio**.
3. Seleccione **Conectores** y, en la lista **Conectores**, elija el tipo de conector **Active Directory Domain Services**. En **Acciones**, seleccione **Propiedades**.  
   ![Propiedades del conector](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. Haga clic en **Configurar particiones de directorio**.
5. En la lista **Select directory partitions** (Seleccionar particiones de directorio), seleccione o anule la selección de los dominios según sea necesario. Compruebe que solo estén seleccionadas las particiones que desee sincronizar.  
   ![Particiones](./media/how-to-connect-sync-configure-filtering/connectorpartitions.png)  
   Si ha cambiado la infraestructura local de Active Directory y ha agregado dominios al bosque o los ha quitado, haga clic en el botón **Actualizar** para actualizar la lista. Al actualizar, se le pedirán las credenciales. Proporcione las credenciales con acceso de lectura a Windows Server Active Directory. No tiene por qué ser el usuario que aparezca en el cuadro de diálogo.  
   ![Actualización necesaria](./media/how-to-connect-sync-configure-filtering/refreshneeded.png)  
6. Cuando termine, haga clic en el botón **Aceptar** para cerrar el cuadro de diálogo **Propiedades**. Si quitó dominios del bosque, aparecerá un mensaje para indicar que se eliminó un dominio y se borrará esa configuración.
7. Pase a ajustar el valor de los perfiles de ejecución.

#### <a name="select-the-domains-to-be-synchronized-using-the-azure-ad-connect-wizard"></a>Selección de los dominios que se van a sincronizar mediante el asistente de Azure AD Connect
Para configurar el filtro de dominio, realice los pasos siguientes:

1.  Inicie el asistente de Azure AD Connect.
2.  Haga clic en **Configurar**.
3.  Seleccione **Personalizar las opciones de sincronización** y, después, haga clic en **Siguiente**.
4.  Escriba sus credenciales de Azure AD
5.  En la pantalla **Directorios conectados**, haga clic en **Siguiente**.
6.  En la **página Filtrado de dominios y unidades organizativas**, haga clic en **Actualizar**.  Ahora aparecerán nuevos dominios y los dominios eliminados desaparecerán.
   ![Particiones](./media/how-to-connect-sync-configure-filtering/update2.png)  

### <a name="update-the-run-profiles"></a>Actualización de los perfiles de ejecución
Si ha actualizado el filtro de dominio, también debe actualizar los perfiles de ejecución.

1. En la lista **Conectores**, asegúrese de que está seleccionado el conector que cambió en el paso anterior. En **Acciones**, seleccione **Configure Run Profiles** (Configurar perfiles de ejecución).  
   ![Perfiles de ejecución de conector 1](./media/how-to-connect-sync-configure-filtering/connectorrunprofiles1.png)  
2. Busque e identifique los siguientes perfiles:
    * Importación completa
    * sincronización completa
    * Importación delta
    * Sincronización delta
    * Exportación
3. En cada perfil, ajuste los dominios **agregados** y **quitados**.
    1. Para cada uno de los cinco perfiles, siga estos pasos para cada dominio **agregado**:
        1. Seleccione el perfil de ejecución y haga clic en **Nuevo paso**.
        2. En la página **Configure Step** (Configurar paso), en la lista desplegable **Tipo**, seleccione el tipo de paso que comparta nombre con el perfil que esté configurando. A continuación, haga clic en **Siguiente**.  
        ![Perfiles de ejecución de conector 2](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep1.png)  
        3. En la página **Connector Configuration** (Configuración de conector), en la lista desplegable **Partición**, seleccione el nombre del dominio que ha agregado al filtro de dominio.  
        ![Perfiles de ejecución de conector 3](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep2.png)  
        4. Para cerrar el cuadro de diálogo **Configurar perfil de ejecución**, haga clic en **Finalizar**.
    2. Para cada uno de los cinco perfiles, siga estos pasos para cada dominio **quitado**:
        1. Seleccione el perfil de ejecución.
        2. Si el campo **Valor** del atributo **Partición**es un GUID, seleccione el paso de ejecución y haga clic en **Eliminar paso**.  
        ![Perfiles de ejecución de conector 4](./media/how-to-connect-sync-configure-filtering/runprofilesdeletestep.png)  
    3. Compruebe el cambio. Cada dominio que desee sincronizar debe aparecer como paso en cada perfil de ejecución.
4. Para cerrar el cuadro de diálogo **Configure Run Profiles** (Configurar perfiles de ejecución), haga clic en **Aceptar**.
5.  Para completar la configuración, debe ejecutar una **Importación completa** y una **Sincronización diferencial**. Continúe leyendo la sección [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

## <a name="organizational-unitbased-filtering"></a>Filtrado por unidad organizativa
La mejor manera de cambiar el filtrado por unidad organizativa es ejecutar el Asistente para instalación y cambiar el [filtrado por dominio y unidad organizativa](how-to-connect-install-custom.md#domain-and-ou-filtering). El Asistente para instalación automatiza todas las tareas que se documentan en este tema.

Solo debe seguir estos pasos si por alguna razón no se puede ejecutar el Asistente para instalación.

Para configurar el filtrado por unidad organizativa, realice los pasos siguientes:

1. Inicie sesión en el servidor donde se ejecuta la sincronización de Azure AD Connect con una cuenta que pertenezca al grupo de seguridad **ADSyncAdmins** .
2. Inicie el **servicio de sincronización** desde el menú **Inicio**.
3. Seleccione **Conectores** y, en la lista **Conectores**, elija el tipo de conector **Active Directory Domain Services**. En **Acciones**, seleccione **Propiedades**.  
   ![Propiedades del conector](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. Haga clic en **Configure Directory Partitions** (Configurar particiones de directorio), seleccione el dominio que desee configurar y haga clic en **Contenedores**.
5. Cuando se le solicite, proporcione las credenciales con acceso de lectura al entorno local de Active Directory. No tiene por qué ser el usuario que aparezca en el cuadro de diálogo.
6. En el cuadro de diálogo **Select Containers** (Seleccionar contenedores), borre las unidades organizativas que no desee sincronizar con el directorio en la nube y luego haga clic en **Aceptar**.  
   ![Unidades organizativas en el cuadro de diálogo Select Containers (Seleccionar contenedores)](./media/how-to-connect-sync-configure-filtering/ou.png)  
   * El contenedor **Computers** debe estar seleccionado para que los equipos con Windows 10 se sincronicen correctamente con Azure AD. Si los equipos unidos a un dominio se encuentran en otras unidades organizativas, asegúrese de que estén seleccionadas.
   * Se debe seleccionar el contenedor **ForeignSecurityPrincipals** si tiene varios bosques con confianzas. Este contenedor permitirá que se resuelva la pertenencia a grupos de seguridad de otros bosques.
   * La unidad organizativa **RegisteredDevices** debe estar seleccionada si ha habilitado la característica de escritura diferida de dispositivos. Si usa otra característica de reescritura, como la de grupos, asegúrese de que estas ubicaciones estén seleccionadas.
   * Seleccione cualquier otra unidad organizativa donde se encuentren usuarios, objetos InetOrgPerson, grupos, contactos y equipos. En la imagen anterior, se encuentran en la unidad organizativa ManagedObjects.
   * Si usa el filtrado basado en el grupo, se debe incluir la unidad organizativa donde se encuentra el grupo.
   * Tenga en cuenta que también puede configurar si las nuevas unidades organizativas agregadas tras la configuración de filtrado deben sincronizarse o no. Consulte la siguiente sección para obtener información detallada.
7. Cuando termine, haga clic en el botón **Aceptar** para cerrar el cuadro de diálogo **Propiedades**.
8. Para completar la configuración, debe ejecutar una **Importación completa** y una **Sincronización diferencial**. Continúe leyendo la sección [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

### <a name="synchronize-new-ous"></a>Sincronización de unidades organizativas nuevas
Las unidades organizativas nuevas creadas una vez que se ha configurado el filtrado se sincronizan de forma predeterminada. Esto se indica mediante una marca en la casilla. También se puede anular la selección de algunas subunidades organizativas. Para conseguirlo, haga clic en la casilla hasta que se ponga blanca con una marca azul (su estado predeterminado). Anule a continuación la selección de las subunidades que no desee sincronizar.

Si todas las subunidades están sincronizadas, la casilla se pondrá blanca con una marca azul.  
![Unidad organizativa con todas las casillas activadas](./media/how-to-connect-sync-configure-filtering/ousyncnewall.png)

Si ha anulado la selección de algunas subunidades, la casilla estará gris con una marca blanca.  
![Unidad organizativa con la selección de algunas subunidades organizativas anulada](./media/how-to-connect-sync-configure-filtering/ousyncnew.png)

Con esta configuración, se sincronizará la nueva unidad organizativa que se creó en ManagedObjects.

El Asistente para la instalación de Azure AD Connect siempre creará esta configuración.

### <a name="dont-synchronize-new-ous"></a>No sincronizar unidades organizativas nuevas
Puede configurar el motor de sincronización para que no sincronice las unidades organizativas nuevas una vez finalizada la configuración del filtrado. Este estado se indica en la interfaz de usuario con una casilla gris sin marcas. Para conseguirlo, haga clic en la casilla hasta que se ponga blanca sin marcas. A continuación, seleccione las subunidades que desee sincronizar.

![Unidad organizativa con raíz sin seleccionar](./media/how-to-connect-sync-configure-filtering/oudonotsyncnew.png)

Con esta configuración, no se sincronizará la nueva unidad organizativa que se creó en ManagedObjects.

## <a name="attribute-based-filtering"></a>Filtrado basado en atributos
Asegúrese de usar la compilación de noviembre de 2015 ([1.0.9125](reference-connect-version-history.md)) o una posterior para que estos pasos funcionen.

> [!IMPORTANT]
>Microsoft recomienda no modificar las reglas predeterminadas creadas por **Azure AD Connect**. Si desea modificar la regla, clónela y deshabilite la regla original. Realice cualquier cambio en la regla clonada. Tenga en cuenta que al hacerlo (mediante la deshabilitación de la regla original) se perderá cualquier corrección de errores o características habilitadas mediante esa regla.

El filtrado por atributo es la manera más flexible de filtrar objetos. Puede usar la capacidad del [aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning.md) para controlar prácticamente todos los aspectos de la sincronización de un objeto con Azure AD.

Se puede aplicar filtrado [entrante](#inbound-filtering), desde Active Directory hacia el metaverso, y [saliente](#outbound-filtering), desde el metaverso hacia Azure AD. Se recomienda aplicar filtrado entrante, ya que es el más fácil de mantener. El filtrado saliente solo debe usarse si es necesario para unir objetos de más de un bosque para realizar la evaluación.

### <a name="inbound-filtering"></a>Filtrado entrante
El filtrado entrante utiliza la configuración predeterminada en la que los objetos que se dirigen a Azure AD no tienen el atributo de metaverso cloudFiltered establecido en ningún valor que se vaya a sincronizar. Si el valor de este atributo está establecido en **True**, el objeto no se sincroniza. No debería estar establecido en **False**, por diseño. Para asegurarse de que otras reglas tengan la capacidad de aportar un valor, este atributo solo debería tener los valores **True** o **NULL** (ausente).

En el filtrado entrante se utilizará el **ámbito** para determinar qué objetos se sincronizan y cuáles no. Aquí realizará ajustes para satisfacer los requisitos de su organización. El módulo de ámbito cuenta con un **grupo** y una **cláusula** para determinar cuándo una regla de sincronización está dentro de él. Un grupo contiene una o varias cláusulas. Existe un operador lógico AND entre varias cláusulas y un operador lógico OR entre varios grupos.

Veamos un ejemplo:  
![Ámbito](./media/how-to-connect-sync-configure-filtering/scope.png)  
Se debe leer como **(departamento = TI) O BIEN (departamento = Ventas Y c = Estados Unidos)** .

En los ejemplos y los pasos siguientes, usaremos el objeto de usuario como ejemplo, pero esto sirve para todo tipo de objetos.

En los ejemplos siguientes, el valor de precedencia empieza por 50. Puede ser cualquier número no usado, pero debe ser menor que 100.

#### <a name="negative-filtering-do-not-sync-these"></a>Filtrado negativo: "no sincronizar estos"
En el siguiente ejemplo, filtraremos (no sincronizaremos) todos los usuarios en los que **extensionAttribute15** tenga el valor **NoSync**.

1. Inicie sesión en el servidor donde se ejecuta la sincronización de Azure AD Connect con una cuenta que pertenezca al grupo de seguridad **ADSyncAdmins** .
2. Inicie el **Synchronization Rules Editor** (Editor de reglas de sincronización) del menú **Inicio**.
3. Asegúrese de que **Entrante** esté seleccionado y haga clic en **Agregar nueva regla**.
4. Asigne a la regla un nombre descriptivo, como "*Entrante desde AD – Usuario FiltroNoSincronizar*". Seleccione el bosque correcto: **Usuario** para **CS object type** (Tipo de objeto de sistema conectado) y **Persona** para **MV object type** (Tipo de objeto de metaverso). En **Tipo de vínculo**, seleccione **Unir**. En **Precedencia**, escriba un valor que no use actualmente ninguna otra regla de sincronización (por ejemplo, 50) y haga clic en **Siguiente**.  
   ![Descripción de entrante 1](./media/how-to-connect-sync-configure-filtering/inbound1.png)  
5. En **Scoping filter** (Filtro de ámbito), haga clic en **Agregar grupo** y en **Agregar cláusula**. En **Atributo**, seleccione **ExtensionAttribute15**. Asegúrese de que el **Operador** esté configurado en **EQUAL** y escriba el valor **NoSync** en el cuadro **Valor**. Haga clic en **Next**.  
   ![Ámbito de entrante 2](./media/how-to-connect-sync-configure-filtering/inbound2.png)  
6. Deje las reglas de **unión** vacías y, después, haga clic en **Siguiente**.
7. Haga clic en **Add Transformation** (Agregar transformación), establezca **FlowType** en **Constante** y **Atributo de destino** en **cloudFiltered**. En el cuadro de texto **Origen**, escriba **True**. Haga clic en **Agregar** para guardar la regla.  
   ![Transformación de entrante 3](./media/how-to-connect-sync-configure-filtering/inbound3.png)
8. Para completar la configuración, debe ejecutar una **sincronización completa**. Continúe leyendo la sección [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

#### <a name="positive-filtering-only-sync-these"></a>Filtrado positivo: "solo sincronizar estos"
Expresar el filtrado positivo puede ser más complicado, ya que también se deben considerar también objetos que no sea obvio que se vayan a sincronizar, como las salas de conferencias. También va a invalidar el filtro predeterminado de la regla original **In from AD - User Join** (Dentro desde AD: unión de usuario). Cuando cree un filtro personalizado, asegúrese de no incluir objetos críticos del sistema, objetos de conflictos de replicación, buzones especiales y las cuentas de servicio de Azure AD Connect.

La opción de filtrado positivo requiere dos reglas de sincronización. Necesita una regla (o varias) con el ámbito correcto de los objetos para sincronizarlos. También necesita una segunda regla de sincronización general que filtre todos los objetos que aún no se hayan identificado como objeto para sincronizar.

En el ejemplo siguiente, solo sincronizaremos los objetos de usuario en los que el atributo departamento tenga el valor **Ventas**.

1. Inicie sesión en el servidor donde se ejecuta la sincronización de Azure AD Connect con una cuenta que pertenezca al grupo de seguridad **ADSyncAdmins** .
2. Inicie el **Synchronization Rules Editor** (Editor de reglas de sincronización) del menú **Inicio**.
3. Asegúrese de que **Entrante** esté seleccionado y haga clic en **Agregar nueva regla**.
4. Asigne a la regla un nombre descriptivo, como "*Entrante desde AD – Usuario Ventas sincronizar*". Seleccione el bosque correcto: **Usuario** para **CS object type** (Tipo de objeto de sistema conectado) y **Persona** para **MV object type** (Tipo de objeto de metaverso). En **Tipo de vínculo**, seleccione **Unir**. En **Precedencia**, escriba un valor que no use actualmente ninguna otra regla de sincronización (por ejemplo, 51) y haga clic en **Siguiente**.  
   ![Descripción de entrante 4](./media/how-to-connect-sync-configure-filtering/inbound4.png)  
5. En **Scoping filter** (Filtro de ámbito), haga clic en **Agregar grupo** y en **Agregar cláusula**. En **Atributo**, seleccione **department**. Asegúrese de que el Operador esté configurado en **EQUAL** y escriba el valor **Sales** en el cuadro **Valor**. Haga clic en **Next**.  
   ![Ámbito de entrante 5](./media/how-to-connect-sync-configure-filtering/inbound5.png)  
6. Deje las reglas de **unión** vacías y, después, haga clic en **Siguiente**.
7. Haga clic en **Add Transformation** (Agregar transformación), establezca **FlowType** en **Constante** y **Atributo de destino** en **cloudFiltered**. En el cuadro **Origen**, escriba **False**. Haga clic en **Agregar** para guardar la regla.  
   ![Transformación de entrante 6](./media/how-to-connect-sync-configure-filtering/inbound6.png)  
   Este es un caso especial en el que estableceremos cloudFiltered explícitamente en **False**.
8. Ahora tenemos que crear la regla de sincronización de comodín. Asigne a la regla un nombre descriptivo, como "*Entrante desde AD – Usuario Filtro de comodín*". Seleccione el bosque correcto: **Usuario** para **CS object type** (Tipo de objeto de sistema conectado) y **Persona** para **MV object type** (Tipo de objeto de metaverso). En **Tipo de vínculo**, seleccione **Unir**. En **Precedencia**, escriba un valor que no use actualmente ninguna otra regla de sincronización (por ejemplo, 99). Ha seleccionado un valor de precedencia mayor (menor prioridad) que la regla de sincronización anterior. Pero también ha dejado algo de espacio para poder agregar más reglas de filtrado de sincronización más tarde, cuando quiera empezar a sincronizar departamentos adicionales. Haga clic en **Next**.  
   ![Descripción de entrante 7](./media/how-to-connect-sync-configure-filtering/inbound7.png)  
9. Deje **Scoping filter** (Filtro de ámbito) vacío y haga clic en **Siguiente**. Un filtro vacío indica que la regla se aplica a todos los objetos.
10. Deje las reglas de **unión** vacías y, después, haga clic en **Siguiente**.
11. Haga clic en **Add Transformation** (Agregar transformación), establezca **FlowType** en **Constante** y **Atributo de destino** en **cloudFiltered**. En el cuadro **Origen**, escriba **True**. Haga clic en **Agregar** para guardar la regla.  
    ![Transformación de entrante 3](./media/how-to-connect-sync-configure-filtering/inbound3.png)  
12. Para completar la configuración, debe ejecutar una **sincronización completa**. Continúe leyendo la sección [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

Si es necesario, puede crear más reglas del primer tipo en las que se incluyan más objetos para la sincronización.

### <a name="outbound-filtering"></a>Filtrado saliente
En algunos casos es necesario realizar el filtrado después de que los objetos se hayan unido en metaverso. Por ejemplo, para determinar si debe sincronizarse un objeto, podría ser necesario observar el atributo mail del bosque de recursos y el atributo userPrincipalName del bosque de cuentas. En estos casos, se crea el filtrado en la regla de salida.

En este ejemplo, se cambia el filtrado de manera que se sincronicen solo los usuarios cuyos mail y userPrincipalName terminen en @contoso.com:

1. Inicie sesión en el servidor donde se ejecuta la sincronización de Azure AD Connect con una cuenta que pertenezca al grupo de seguridad **ADSyncAdmins** .
2. Inicie el **Synchronization Rules Editor** (Editor de reglas de sincronización) del menú **Inicio**.
3. En **Rules Type** (Tipo de reglas), haga clic en **Saliente**.
4. Según la versión de Connect que utilice, o bien busque la regla denominada **Out to AAD – User Join** (Salida a AAD - Unión de usuario) o la regla **Out a AAD - User Join SOAInAD** (Salida a AAD - SOAInAD de unión de usuario) y haga clic en **Editar**.
5. En el elemento emergente, responda **Sí** para crear una copia de la regla.
6. En la página **Descripción**, cambie la **Precedencia** a un valor sin usar, como 50.
7. Haga clic en **Scoping filter** (Filtro de ámbito) en el panel de navegación izquierdo y haga clic en **Agregar cláusula**. En **Atributo**, seleccione **mail**. En **Operador**, seleccione **ENDSWITH**. En **Valor**, escriba **\@contoso.com** y haga clic en **Agregar cláusula**. En **Atributo**, seleccione **userPrincipalName**. En **Operador**, seleccione **ENDSWITH**. En **Valor**, escriba **\@contoso.com**.
8. Haga clic en **Save**(Guardar).
9. Para completar la configuración, debe ejecutar una **sincronización completa**. Continúe leyendo la sección [Aplicación y comprobación de los cambios](#apply-and-verify-changes).

## <a name="apply-and-verify-changes"></a>Aplicación y comprobación de los cambios
Una vez realizados los cambios de configuración, debe aplicarlos a los objetos ya presentes en el sistema. También es posible que se deban procesar objetos que no estén en el motor de sincronización (y que el motor de sincronización necesite volver a leer el sistema de origen para comprobar el contenido).

Si cambió la configuración mediante el filtrado por **dominio** o **unidad organizativa**, deberá realizar una **Importación completa** seguida de una **Sincronización diferencial**.

Si cambió la configuración mediante el filtrado por **atributo**, deberá llevar a cabo una **Sincronización completa**.

Siga estos pasos:

1. Inicie el **servicio de sincronización** desde el menú **Inicio**.
2. Seleccione **Conectores**. En la lista **Conectores**, elija el conector en el que realizó un cambio de configuración antes. En **Acciones**, seleccione **Ejecutar**.  
   ![Ejecución de conector](./media/how-to-connect-sync-configure-filtering/connectorrun.png)  
3. En **Run profiles**(Perfiles de ejecución), seleccione la operación mencionada en la sección anterior. Si tiene que ejecutar dos acciones, ejecute la segunda una vez haya terminado la primera. (La columna **State** (Estado) es **Inactivo** para el conector seleccionado).

Después de la sincronización, todos los cambios se almacenan provisionalmente para exportarlos. Antes de realizar los cambios en Azure AD, se quiere comprobar que todos sean correctos.

1. Inicie un símbolo del sistema y vaya a `%ProgramFiles%\Microsoft Azure AD Sync\bin`.
2. Ejecute `csexport "Name of Connector" %temp%\export.xml /f:x`.  
   El nombre del conector puede encontrarse en el servicio de sincronización. Tendrá un nombre similar a contoso.com – AAD en Azure AD.
3. Ejecute `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv`.
4. Ahora tiene un archivo en %temp% denominado export.csv que se puede examinar en Microsoft Excel. Este archivo contiene todos los cambios que se van a exportar.
5. Realice los cambios necesarios en los datos o la configuración y ejecute estos pasos de nuevo (importación y sincronización y comprobación) hasta que los cambios que se vayan a exportar sean los previstos.

Cuando esté satisfecho, exporte los cambios a Azure AD.

1. Seleccione **Conectores**. En la lista **Conectores**, elija el conector de Azure AD. En **Acciones**, seleccione **Ejecutar**.
2. En **Run profiles** (Perfiles de ejecución), seleccione **Exportar**.
3. Si los cambios de configuración van a eliminar muchos objetos, verá un error al exportar si el número es mayor que el umbral configurado (de forma predeterminada, 500). Si aparece este error, deberá deshabilitar temporalmente la característica para [evitar eliminaciones por error](how-to-connect-sync-feature-prevent-accidental-deletes.md).

Ahora es el momento de volver a habilitar el programador.

1. Abra el **Programador de tareas** en el menú **Inicio**.
2. Directamente en **Biblioteca del Programador de tareas**, busque la tarea **Programador de sincronización de Azure AD**, haga clic con el botón derecho en ella y seleccione **Habilitar**.

## <a name="group-based-filtering"></a>Filtrado basado en grupo
Puede configurar el filtrado por grupo la primera vez que instale Azure AD Connect con la [instalación personalizada](how-to-connect-install-custom.md#sync-filtering-based-on-groups). Se ha diseñado para una implementación piloto donde se vaya a sincronizar solo un pequeño conjunto de objetos. Cuando deshabilite el filtrado por grupo, no lo podrá volver a habilitar. El filtrado por grupo *no es compatible* con la configuración personalizada. Solo se admite para configurar esta característica con el Asistente para instalación. Cuando haya completado la prueba piloto, utilice una de las opciones de filtrado de este tema. Si se usa el filtrado por UO junto con el filtrado basado en grupo, se deben incluir las unidades organizativas donde se encuentran el grupo y sus miembros.

Al sincronizar varios bosques de AD, puede configurar el filtrado basado en grupos mediante la especificación de un grupo diferente para cada conector de AD. Si desea sincronizar un usuario en un bosque de AD y el mismo usuario tiene uno o varios objetos en otros bosques de AD, debe asegurarse de que el objeto de usuario y todos sus objetos correspondientes se encuentran en el ámbito de filtrado basado en grupos. Por ejemplo:

* Tiene un usuario en un bosque que tiene un objeto de FSP (entidad de seguridad externa) correspondiente en otro bosque. Ambos objetos deben estar dentro del ámbito de filtrado basado en grupos. De lo contrario, el usuario no se sincronizará con Azure AD.

* Tiene un usuario en un bosque que, a su vez, tiene una cuenta de recurso correspondiente (por ejemplo, un buzón vinculado) en otro bosque. Además, ha configurado Azure AD Connect para vincular el usuario con la cuenta del recurso. Ambos objetos deben estar dentro del ámbito de filtrado basado en grupos. De lo contrario, el usuario no se sincronizará con Azure AD.

* Tiene un usuario en un bosque que tiene un contacto de correo correspondiente en otro bosque. Además, ha configurado Azure AD Connect para vincular el usuario con el contacto de correo. Ambos objetos deben estar dentro del ámbito de filtrado basado en grupos. De lo contrario, el usuario no se sincronizará con Azure AD.


## <a name="next-steps"></a>Pasos siguientes
- Más información sobre la configuración de la [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md).
- Más información sobre la [integración de las identidades locales con Azure AD](whatis-hybrid-identity.md).
