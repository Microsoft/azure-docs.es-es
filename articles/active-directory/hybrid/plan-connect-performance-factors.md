---
title: Factores que influyen en el rendimiento de Azure AD Connect
description: En este documento se explica cómo afectan diferentes factores al motor de aprovisionamiento de Azure AD Connect. Estos factores ayudarán a las organizaciones a planear su implementación de Azure AD Connect para asegurarse de que cumple los requisitos de sincronización.
services: active-directory
author: billmath
manager: daveba
tags: azuread
ms.service: active-directory
ms.subservice: hybrid
ms.topic: conceptual
ms.workload: identity
ms.date: 10/06/2018
ms.reviewer: martincoetzer
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3e2c09bcd43b08778324a32cc052fad5b85714c4
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279591"
---
# <a name="factors-influencing-the-performance-of-azure-ad-connect"></a>Factores que influyen en el rendimiento de Azure AD Connect

Azure AD Connect sincroniza su instancia de Active Directory en Azure AD. Este servidor es un componente crítico para mover sus identidades de usuario a la nube. Los principales factores que afectan al rendimiento de una instancia de Azure AD Connect son:

| **Factor de diseño**| **Definición** |
|:-|-|
| Topología| La distribución de los puntos de conexión y de los componentes de Azure AD Connect debe administrarse en la red. |
| Escala| El número de objetos, como usuarios, grupos y unidades organizativas, que administrará Azure AD Connect. |
| Hardware| El hardware (físico o virtual) para Azure AD Connect y la capacidad de rendimiento dependiente de cada componente de hardware, incluida la configuración de la CPU, la memoria, la red y el disco duro. |
| Configuración| Cómo procesa Azure AD Connect los directorios y la información. |
| Cargar| Frecuencia de cambios de objeto. Las cargas pueden variar durante una hora, día o semana. Dependiendo del componente, es posible que tenga que diseñar para carga máxima o carga media. |

El propósito de este documento es describir los factores que afectan al rendimiento del motor de aprovisionamiento de Azure AD Connect. Las organizaciones grandes o complejas (organizaciones que aprovisionan más de 100 000 objetos) pueden usar las recomendaciones para optimizar su implementación de Azure AD Connect si experimentan alguno de los problemas de rendimiento que se describen aquí. Los otros componentes de Azure AD Connect, como los agentes y el [estado de Azure AD Connect](how-to-connect-health-agent-install.md) no se tratan aquí.

> [!IMPORTANT]
> Microsoft no admite la modificación ni el funcionamiento de Azure AD Connect con acciones distintas a estas que se documentan formalmente. Cualquiera de estas acciones pueden provocar un estado incoherente o incompatible de sincronización de Azure AD Connect. Como resultado, Microsoft no ofrece soporte técnico para estas implementaciones.

## <a name="azure-ad-connect-component-factors"></a>Factores de componente de Azure AD Connect

En el siguiente diagrama se muestra una arquitectura de alto nivel del motor de aprovisionamiento que se conecta a un único bosque, aunque también se admiten varios bosques. Esta arquitectura muestra cómo interactúan entre sí los distintos componentes.

![AzureADConnentInternal](media/plan-connect-performance-factors/AzureADConnentInternal.png)

El motor de aprovisionamiento se conecta a cada bosque de Active Directory y a Azure AD. El proceso de lectura de la información de cada directorio se llama Importar. Exportar hace referencia a la actualización de los directorios desde el motor de aprovisionamiento. La sincronización evalúa las reglas de cómo los objetos fluirán en el motor de aprovisionamiento. Para obtener más información, puede consultar [Azure AD Connect Sync: Descripción de la arquitectura](./concept-azure-ad-connect-sync-architecture.md).

Azure AD Connect usa los siguientes procesos, reglas y áreas de almacenamiento provisional para permitir la sincronización de Active Directory en Azure AD:

* **Espacio conector (CS)** : los objetos de cada directorio conectado (CD), los directorios reales, se preparan aquí primero antes de que el motor de aprovisionamiento los procese. Azure AD tiene su propio CS y cada bosque al que se conecta tiene su propio CS.
* **Metaverso (MV)** : los objetos que deben sincronizarse se crean aquí basándose en las reglas de sincronización. Los objetos deben existir en el MV antes de que puedan rellenar objetos y atributos en otros directorios conectados. Solo hay un MV.
* **Reglas de sincronización**: deciden qué objetos se crearán (proyectarán) o conectarán (unirán) a los objetos del MV. Las reglas de sincronización también deciden qué valores de atributo se copiarán o transformarán en y desde los directorios.
* **Perfiles de ejecución**: agrupa los pasos del proceso de copia de objetos y sus valores de atributo según las reglas de sincronización entre las áreas de almacenamiento provisional y los directorios conectados.

Existen diferentes perfiles de ejecución para optimizar el rendimiento del motor de aprovisionamiento. La mayoría de las organizaciones usarán los programas y perfiles de ejecución predeterminados para las operaciones normales, pero es posible que algunas organizaciones tengan que [cambiar el programa](./how-to-connect-sync-feature-scheduler.md) o desencadenar otros perfiles de ejecución para hacer frente a situaciones poco comunes. Los siguientes perfiles de ejecución están disponibles:

### <a name="initial-sync-profile"></a>Perfil de sincronización inicial

El perfil de sincronización inicial es el proceso de lectura de los directorios conectados, como un bosque de Active Directory, por primera vez. A continuación, realiza un análisis en todas las entradas de la base de datos del motor de sincronización. El ciclo inicial creará nuevos objetos en Azure AD y tardará más tiempo en completarse si los bosques de Active Directory son grandes. La sincronización inicial incluye los pasos siguientes:

1. Importación completa en todos los conectores
2. Sincronización completa en todos los conectores
3. Exportación en todos los conectores

### <a name="delta-sync-profile"></a>Perfil de sincronización diferencial

Para optimizar el proceso de sincronización, este perfil de ejecución solo procesa los cambios (crea, elimina y actualiza) de objetos en los directorios conectados, desde el último proceso de sincronización. De forma predeterminada, el perfil de sincronización diferencial se ejecuta cada 30 minutos. Las organizaciones deben esforzarse por mantener el tiempo por debajo de los 30 minutos, para asegurarse de que Azure AD esté actualizado. Para supervisar el estado de Azure AD Connect, use el [agente de supervisión del estado](how-to-connect-health-sync.md) para ver cualquier problema con el proceso. El perfil de sincronización diferencial incluye los siguientes pasos:

1. Importación diferencial en todos los conectores
2. Sincronización diferencial en todos los conectores
3. Exportación en todos los conectores

Un escenario típico de sincronización diferencial de una organización empresarial es:

- ~1 % de los objetos se eliminan
- ~1 % de los objetos se crean
- ~5 % de los objetos se modifican

La frecuencia de cambio puede variar en función de la periodicidad con la que su organización actualiza a los usuarios de Active Directory. Por ejemplo, pueden producirse mayores frecuencias de cambios con el carácter estacional de la contratación y la reducción de la mano de obra.

### <a name="full-sync-profile"></a>Perfil de sincronización completo

Si ha realizado uno de los siguientes cambios de configuración, debe ejecutar un ciclo de sincronización completo:



- Incremento del alcance de los objetos o atributos que se van a importar desde los directorios conectados. Por ejemplo, al agregar un dominio o unidad organizativa en el ámbito de importación.
- Modificación de las reglas de sincronización. Por ejemplo, cuando crea una nueva regla para rellenar el cargo de un usuario en Azure AD desde extension_attribute3 en Active Directory. Esta actualización requiere que el motor de aprovisionamiento vuelva a examinar todos los usuarios existentes para actualizar sus cargos y aplicar el cambio en el futuro.

Las operaciones siguientes se incluyen en un ciclo de sincronización completa:

1. Importación completa en todos los conectores
2. Sincronización diferencial o completa en todos los conectores
3. Exportación en todos los conectores

> [!NOTE]
> Se requiere una planeación cuidadosa cuando se realizan actualizaciones en masa de muchos objetos en Active Directory o Azure AD. Las actualizaciones en masa harán que el proceso de sincronización diferencial tarde más tiempo al importar, ya que muchos objetos se cambiaron. Pueden producirse importaciones largas incluso si la actualización en masa no afecta al proceso de sincronización. Por ejemplo, asignar licencias a muchos usuarios en Azure AD dará lugar a un largo ciclo de importación desde Azure AD, pero no resultará en ningún cambio de atributo en Active Directory.

### <a name="synchronization"></a>Synchronization

El tiempo de ejecución del proceso de sincronización tiene las siguientes características de rendimiento:

* La sincronización tiene un único subproceso, lo que significa que el motor de aprovisionamiento no realiza ningún procesamiento paralelo de perfiles de ejecución de los objetos, atributos o directorios conectados.
* El tiempo de importación crece linealmente con el número de objetos que se sincronizan. Por ejemplo, si 10 000 objetos tardan 10 minutos en importarse, 20 000 objetos tardarán, aproximadamente, 20 minutos en el mismo servidor.
* La exportación también es lineal.
* La sincronización crece exponencialmente según el número de objetos con referencias a otros objetos. La pertenencia a grupos y los grupos anidados son lo que más afecta al rendimiento, ya que sus miembros se refieren a objetos de usuario o a otros grupos. Estas referencias deben encontrarse y referenciarse con objetos reales en el MV para completar el ciclo de sincronización.

### <a name="filtering"></a>Filtros

El tamaño de la topología de Active Directory que quiere importar es el número uno de los factores que afectan al rendimiento y al tiempo total que tardarán los componentes internos del motor de aprovisionamiento.

[Filtrado](./how-to-connect-sync-configure-filtering.md): debe usarse para reducir los objetos que se van a sincronizar. Impedirá que se procesen y exporten objetos innecesarios a Azure AD. En orden de preferencia, las siguientes técnicas de filtrado están disponibles:



- **Filtrado basado en dominio**: use esta opción para seleccionar los dominios específicos que quiere sincronizar con Azure AD. Debe agregar y quitar dominios de la configuración del motor de sincronización al realizar cambios en la infraestructura local después de instalar la sincronización de Azure AD Connect.
- **Filtrado de la unidad organizativa (UO)** : usa las UO para dirigirse a objetos específicos en dominios de Active Directory para el aprovisionamiento en Azure AD. El filtrado por unidad organizativa es el segundo mecanismo de filtrado que se recomienda, porque utiliza consultas de ámbito LDAP sencillas para importar un subconjunto más pequeño de objetos de Active Directory.
- **Filtrado de atributos por objeto**: usa los valores de atributo de los objetos para decidir si aprovisionar un determinado objeto de Active Directory en Azure AD. El filtrado de atributos es excelente para ajustar los filtros cuando el filtrado por dominio y por unidad organizativa no cumple los requisitos específicos de filtrado. El filtrado de atributos no reduce el tiempo de importación, pero puede reducir el tiempo de sincronización y exportación.
- **Filtrado basado en grupos**: usa la pertenencia a grupos para decidir qué objetos deben aprovisionarse en Azure AD. El filtrado basado en grupos solo es adecuado para situaciones de prueba y no se recomienda para entornos de producción, debido a la sobrecarga adicional necesaria para comprobar la pertenencia a grupos durante el ciclo de sincronización.

Muchos [objetos de desconector](concept-azure-ad-connect-sync-architecture.md#relationships-between-staging-objects-and-metaverse-objects) persistentes en el CS de Active Directory pueden provocar tiempos de sincronización más largos, porque el motor de aprovisionamiento debe reevaluar cada objeto de desconector para una posible conexión en el ciclo de sincronización. Para solucionar este problema, considere una de las siguientes recomendaciones:



- Coloque los objetos de desconector fuera del ámbito de importación mediante el filtrado de unidad organizativa o dominio.
- Proyecte o combine los objetos al MV y establezca el atributo [cloudFiltered](how-to-connect-sync-configure-filtering.md#negative-filtering-do-not-sync-these) en True, para evitar el aprovisionamiento de estos objetos en Azure AD CS.

> [!NOTE]
> Los usuarios pueden confundirse o pueden surgir problemas con los permisos de las aplicaciones cuando se filtran demasiados objetos. Por ejemplo, en una implementación en línea híbrida de Exchange, los usuarios con buzones locales verán más usuarios en su lista global de direcciones que aquellos con buzones de Exchange en línea. En otros casos, puede que un usuario quiera conceder acceso en una aplicación en la nube a otro usuario que no forma parte del ámbito del conjunto filtrado de objetos.

### <a name="attribute-flows"></a>Flujos de atributo

Flujos de atributo es el proceso para copiar o transformar los valores de atributo de objetos de un directorio conectado a otro directorio conectado. Se definen como parte de las reglas de sincronización. Por ejemplo, cuando se cambia el número de teléfono de un usuario en Active Directory, se actualizará el número de teléfono en Azure AD. Las organizaciones pueden [modificar los flujos de atributo](./how-to-connect-sync-change-the-configuration.md) para adaptarse a distintos requisitos. Se recomienda copiar los flujos de atributo existentes antes de modificarlos.

Los redireccionamientos simples, como el flujo de un valor de atributo a un atributo diferente, no tienen un impacto en el rendimiento material. Un ejemplo de una redirección es el flujo de un número de móvil en Active Directory al número de teléfono de la oficina en Azure AD.

La transformación de los valores de atributo puede tener un impacto en el rendimiento del proceso de sincronización. La transformación de los valores de atributo incluyen modificar, volver a formatear, concatenar o restar valores de atributos.

Las organizaciones pueden impedir que ciertos atributos fluyan a Azure AD, pero no influirán en el rendimiento del motor de aprovisionamiento.

> [!NOTE]
> No elimine los flujos de atributos no deseados de las reglas de sincronización. Se recomienda deshabilitarlos, porque las reglas eliminadas vuelven a crearse durante las actualizaciones de Azure AD Connect.

## <a name="azure-ad-connect-dependency-factors"></a>Factores de dependencia de Azure AD Connect

El rendimiento de Azure AD Connect depende del rendimiento de los directorios conectados a los que exporta y desde los que importa. Por ejemplo, el tamaño de Active Directory que es necesario importar o la latencia de red al servicio de Azure AD. La base de datos SQL que usa el motor de aprovisionamiento también afecta al rendimiento general del ciclo de sincronización.

### <a name="active-directory-factors"></a>Factores de Active Directory

Como se mencionó anteriormente, el número de objetos que se van a importar afecta significativamente al rendimiento. El [hardware y los requisitos previos para Azure AD Connect](how-to-connect-install-prerequisites.md) describen los niveles de hardware específicos en función del tamaño de la implementación. Azure AD Connect solo admite topologías específicas tal como se describe en [Topologías de Azure AD Connect](plan-connect-topologies.md). No hay ninguna recomendación ni optimización de rendimiento para topologías no admitidas.

Asegúrese de que el servidor de Azure AD Connect cumple los requisitos de hardware según el tamaño de Active Directory que quiere importar. Una conectividad de red incorrecta o lenta entre el servidor de Azure AD Connect y los controladores de dominio de Active Directory puede ralentizar la importación.

### <a name="azure-ad-factors"></a>Factores de Azure AD

Azure AD usa la limitación para proteger el servicio en la nube frente a ataques por denegación de servicio (DoS). Actualmente, Azure AD tiene un límite de 7000 escrituras por cada 5 minutos (84 000 por hora). Por ejemplo, se pueden limitar las operaciones siguientes:



- Exportación de Azure AD Connect a Azure AD.
- Las aplicaciones o scripts de PowerShell que actualizan directamente Azure AD, incluso en segundo plano, como las pertenencias a grupos dinámicos.
- Usuarios que actualizan sus propios registros de identidad, como registrarse para MFA o SSPR (restablecimiento de la contraseña de autoservicio).
- Operaciones dentro de la interfaz gráfica de usuario.

Plan para las tareas de implementación y mantenimiento, para asegurarse de que el ciclo de sincronización de Azure AD Connect no se ve afectado por la limitación. Por ejemplo, si realiza muchas contrataciones a la vez y crea miles de identidades de usuario, esto puede dar lugar a actualizaciones en pertenencias dinámicas a grupos, asignaciones de licencias y registros de restablecimiento de contraseñas de autoservicio. Es mejor distribuir estas escrituras a lo largo de varias horas o días.

### <a name="sql-database-factors"></a>Factores de la base de datos SQL

El tamaño de la topología de Active Directory de origen influirá en el rendimiento de la base de datos SQL. Siga los [requisitos de hardware](how-to-connect-install-prerequisites.md) de la base de datos de SQL Server y tenga en cuenta las siguientes recomendaciones:



- Las organizaciones con más de 100 000 usuarios pueden reducir las latencias de red mediante la implantación de la base de datos SQL y el motor de aprovisionamiento en el mismo servidor.
- Debido a los altos requisitos de entrada y salida (E/S) del disco del proceso de sincronización, use las unidades de estado sólido (SSD) para la base de datos SQL del motor de aprovisionamiento para obtener mejores resultados y, si no es posible, considere las configuraciones RAID 1 o RAID 0.
- No realice ninguna sincronización completa de manera preventiva; ya que genera una renovación innecesaria y tiempos de respuesta más lentos.

## <a name="conclusion"></a>Conclusión

Para optimizar el rendimiento de la implementación de Azure AD Connect, tenga en cuenta las siguientes recomendaciones:



- Use la [configuración de hardware recomendada](how-to-connect-install-prerequisites.md) según el tamaño de la implementación para el servidor de Azure AD Connect.
- Al actualizar Azure AD Connect en implementaciones a gran escala, considere el uso de [método de migración oscilante](./how-to-upgrade-previous-version.md#swing-migration), para asegurarse de que tiene el menor tiempo de inactividad y la mejor fiabilidad. 
- Use SSD para la base de datos SQL para obtener el mejor rendimiento de escritura.
- Filtre el ámbito de Active Directory para incluir solo los objetos que se deben aprovisionar en Azure AD, mediante el filtrado de atributos, de unidades organizativas o dominios.
- Si necesita cambiar las reglas de flujo de atributo predeterminadas, en primer lugar copie la regla y, después, cambie la copia y deshabilite la regla original. No olvide volver a ejecutar una sincronización completa.
- Planee el tiempo adecuado para el perfil de ejecución de sincronización completa inicial.
- Esfuércese en completar el ciclo de sincronización diferencial en 30 minutos. Si el perfil de sincronización diferencial no se completa en 30 minutos, modifique la frecuencia de sincronización predeterminada para incluir un ciclo de sincronización diferencial completo.
- Supervise el [estado de sincronización de Azure AD Connect](how-to-connect-health-agent-install.md) en Azure AD.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).