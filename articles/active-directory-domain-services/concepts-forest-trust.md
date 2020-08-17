---
title: Funcionamiento de la confianza en Azure AD Domain Services | Microsoft Docs
description: Obtenga más información sobre el funcionamiento de la confianza de bosque con Azure AD Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 424a05d6a096538aa296bb11863702b816410fb9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87480652"
---
# <a name="how-trust-relationships-work-for-resource-forests-in-azure-active-directory-domain-services"></a>Funcionamiento de las relaciones de confianza para los bosques de recursos en Azure Active Directory Domain Services

Active Directory Domain Services (AD DS) proporciona seguridad a través de varios dominios o bosques mediante relaciones de confianza entre ellos. Antes de que se pueda realizar la autenticación a través de las confianzas, Windows debe comprobar si el dominio que se solicita, ya sea por parte de un usuario, un equipo o un servicio, tiene una relación de confianza con el dominio de la cuenta que realiza la solicitud.

Para comprobar si existe esta relación de confianza, el sistema de seguridad de Windows calcula una ruta de acceso de confianza entre el controlador de dominio del servidor que recibe la solicitud y un controlador de dominio del dominio de la cuenta que realiza la solicitud.

Los mecanismos de control de acceso que se proporcionan en AD DS y el modelo de seguridad distribuida de Windows ofrecen un entorno para el funcionamiento de las confianzas de dominio y bosque. Para que estas confianzas funcionen correctamente, todos los recursos o equipos deben tener una ruta de acceso de confianza directa a un controlador de dominio en el dominio donde se encuentran.

La ruta de acceso de confianza se implementa a través del servicio de Net Logon, mediante una conexión de llamada a procedimiento remoto (RPC) autenticada a la entidad del dominio de confianza. También se extiende un canal seguro a otros dominios de AD DS a través de relaciones de confianza entre dominios. Este canal seguro se utiliza para obtener y comprobar la información de seguridad, incluidos los identificadores de seguridad (SID) para usuarios y grupos.

Para obtener información general sobre cómo se aplican las confianzas a Azure AD DS, consulte [Conceptos y características del bosque de recursos][create-forest-trust].

Para empezar a usar confianzas en Azure AD DS, [cree un dominio administrado que use confianzas de bosque][tutorial-create-advanced].

## <a name="trust-relationship-flows"></a>Flujos de una relación de confianza

El flujo de las comunicaciones seguras a través de confianzas determina la elasticidad de una confianza. La forma de crear o configurar una confianza determina hasta qué punto se extiende la comunicación dentro de los bosques o entre ellos.

El flujo de la comunicación a través de las confianzas viene determinado por la dirección de la confianza. Las confianzas pueden ser unidireccionales o bidireccionales, y transitivas o no transitivas.

En el diagrama siguiente se muestra que todos los dominios del *Árbol 1* y del *Árbol 2* tienen relaciones de confianza transitivas de forma predeterminada. Como resultado, los usuarios del *Árbol 1* pueden acceder a los recursos de los dominios del *Árbol 2* y los usuarios del *Árbol 1* pueden acceder a los recursos del *Árbol 2*, si se asignan los permisos adecuados en el recurso.

![Diagrama de relaciones de confianza entre dos bosques](./media/concepts-forest-trust/trust-relationships.png)

### <a name="one-way-and-two-way-trusts"></a>Confianzas unidireccionales y bidireccionales

Las relaciones de confianza que permiten el acceso a los recursos pueden ser unidireccionales o bidireccionales.

Una confianza unidireccional es una ruta de acceso de autenticación creada entre dos dominios y que transcurre en una sola dirección. En una confianza unidireccional entre el *dominio A* y el *dominio B*, los usuarios del *dominio A* pueden acceder a los recursos del *dominio B*. Sin embargo, los usuarios del *dominio B* no pueden acceder a los recursos del *dominio A*.

Algunas confianzas unidireccionales pueden ser transitivas o no transitivas, dependiendo del tipo de confianza que se cree.

En una confianza bidireccional, el *dominio A* confía en el *dominio B* y el *dominio B* a su vez confía en el *dominio A*. Esta configuración implica que se pueden pasar solicitudes de autenticación entre los dos dominios en ambas direcciones. Algunas relaciones bidireccionales también pueden ser transitivas o no transitivas, dependiendo del tipo de confianza que se cree.

Todas las confianzas de dominio en un bosque de AD DS son confianzas transitivas bidireccionales. Cuando se crea un nuevo dominio secundario, se crea automáticamente una confianza transitiva bidireccional entre el nuevo dominio secundario y el dominio primario.

### <a name="transitive-and-non-transitive-trusts"></a>Confianzas transitivas y no transitivas

La transitividad determina si una confianza se puede extender fuera de los dos dominios con los que se formó.

* Una confianza transitiva se puede usar para extender las relaciones de confianza con otros dominios.
* Una confianza no transitiva se puede usar para denegar relaciones de confianza con otros dominios.

Cada vez que se crea un nuevo dominio en un bosque, se crea automáticamente una relación de confianza transitiva bidireccional entre el nuevo dominio y su dominio primario. Si se agregan dominios secundarios al nuevo dominio, la ruta de acceso de confianza fluye hacia arriba a través de la jerarquía de dominios, extendiendo la ruta de acceso de confianza inicial creada entre el nuevo dominio y su dominio primario. Las relaciones de confianza transitivas fluyen hacia arriba a través de un árbol de dominios a medida que este se forma, creando confianzas transitivas entre todos los dominios del árbol.

Las solicitudes de autenticación siguen estas rutas de acceso de confianza, por lo que cualquier dominio del bosque puede autenticar las cuentas de cualquier otro dominio del bosque. Con un único proceso de inicio de sesión, las cuentas que dispongan de los permisos adecuados pueden acceder a los recursos de cualquier dominio del bosque.

## <a name="forest-trusts"></a>Confianzas de bosque

Las confianzas de bosque ayudan a administrar infraestructuras de AD DS segmentadas y permiten el acceso a recursos y otros objetos entre varios bosques. Las confianzas de bosque son útiles para los proveedores de servicios, las empresas en proceso de fusión o adquisición, extranets empresariales de colaboración y compañías que buscan una solución para conseguir la autonomía administrativa.

Gracias a las confianzas de bosque, se pueden vincular dos bosques diferentes para formar una relación de confianza transitiva unidireccional o bidireccional. Una confianza de bosque permite a los administradores conectar dos bosques de AD DS con una única relación de confianza, para proporcionar una experiencia de autenticación y autorización directa a través de los bosques.

Una confianza de bosque solo se puede crear entre el dominio raíz de un bosque y el dominio raíz de otro bosque. Las confianzas de bosque solo se pueden crear entre dos bosques y no se pueden extender implícitamente a un tercer bosque. Este comportamiento significa que, si se crea una confianza de bosque entre el *Bosque 1* y el *Bosque 2*, y se crea otra confianza de bosque entre el *Bosque 2* y el *Bosque 3*, el *Bosque 1* no tiene una confianza implícita con el *Bosque 3*.

En el siguiente diagrama se muestran dos relaciones de confianza de bosque independientes entre tres bosques de AD DS de una única organización.

![Diagrama de relaciones de confianza de bosque dentro de una única organización](./media/concepts-forest-trust/forest-trusts-diagram.png)

En esta configuración de ejemplo se proporciona el siguiente acceso:

* Los usuarios del *Bosque 2* pueden acceder a los recursos de cualquier dominio del *Bosque 1* o del *Bosque 3*.
* Los usuarios del *Bosque 3* pueden acceder a los recursos de cualquier dominio del *Bosque 2*.
* Los usuarios del *Bosque 1* pueden acceder a los recursos de cualquier dominio del *Bosque 2*.

Esta configuración no permite que los usuarios del *Bosque 1* accedan a los recursos del *Bosque 3* y viceversa. Para que los usuarios del *Bosque 1* y del *Bosque 3* puedan compartir recursos, se debe crear una confianza transitiva bidireccional entre ambos.

Si se crea una confianza de bosque unidireccional entre dos bosques, los miembros del bosque de confianza pueden utilizar los recursos ubicados en el bosque que confía. Sin embargo, la confianza funciona en una sola dirección.

Por ejemplo, cuando se crea una confianza de bosque unidireccional entre el *Bosque 1* (el bosque de confianza) y el *Bosque 2* (el bosque que confía):

* Los miembros del *Bosque 1* pueden acceder a los recursos ubicados en el *Bosque 2*.
* Los miembros del *Bosque 2* no pueden acceder a los recursos ubicados en el *Bosque 1* mediante la misma confianza.

> [!IMPORTANT]
> El bosque de recursos de Azure AD Domain Services solo admite una confianza de bosque unidireccional hacia un entorno de Active Directory local.

### <a name="forest-trust-requirements"></a>Requisitos de la confianza de bosque

Para poder crear una confianza de bosque, debe comprobar que cuenta con la infraestructura del sistema de nombres de dominio (DNS) correcta. Solo se pueden crear confianzas de bosque cuando está disponible una de las siguientes configuraciones de DNS:

* Un solo servidor DNS raíz es el servidor DNS raíz para ambos espacios de nombres DNS de bosque: la zona raíz contiene delegaciones para cada uno de los espacios de nombres DNS y las sugerencias de raíz de todos los servidores DNS incluyen el servidor DNS raíz.
* No hay ningún servidor DNS raíz compartido y los servidores DNS raíz de cada espacio de nombres DNS de bosque usan reenviadores condicionales DNS para cada espacio de nombres DNS, con el fin de enrutar consultas sobre nombres en el otro espacio de nombres.

    > [!IMPORTANT]
    > El bosque de recursos de Azure AD Domain Services debe usar esta configuración de DNS. El hospedaje de un espacio de nombres DNS que no sea el espacio de nombres DNS del bosque de recursos no es una característica de Azure AD Domain Services. La configuración correcta precisa de reenviadores condicionales.

* No hay ningún servidor DNS raíz compartido y los servidores DNS raíz de cada espacio de nombres DNS de bosque usan zonas secundarias DNS configuradas en cada espacio de nombres DNS para enrutar las consultas sobre nombres en el otro espacio de nombres.

Para crear una confianza de bosque, debe ser miembro del grupo de administradores de dominio (en el dominio raíz del bosque) o del grupo de administradores de empresa en Active Directory. A cada confianza se le asigna una contraseña que deben conocer los administradores de ambos bosques. Los miembros del grupo de administradores de empresa de ambos bosques pueden crear las confianzas en ambos bosques a la vez; en este escenario, se genera y escribe automáticamente una contraseña criptográficamente aleatoria para ambos bosques.

La confianza de bosque de salida para Azure AD Domain Services se crea en Azure Portal. La relación de confianza con el propio dominio administrado no se crea manualmente. La confianza de bosque de entrada debe ser configurada por un usuario con los privilegios anteriormente indicados en el entorno de Active Directory local.

## <a name="trust-processes-and-interactions"></a>Procesos e interacciones de confianza

Muchas transacciones entre dominios y entre bosques dependen de las confianzas de dominio o bosque para realizar diversas tareas. En esta sección se describen los procesos y las interacciones que se producen cuando se accede a los recursos a través de confianzas y se evalúan las referencias de autenticación.

### <a name="overview-of-authentication-referral-processing"></a>Información general sobre el procesamiento de referencia de autenticación

Cuando se refiere una solicitud de autenticación a un dominio, el controlador de dominio de este debe determinar si existe una relación de confianza con el dominio del que procede la solicitud. También se debe determinar la dirección de la confianza y si esta es transitiva o no transitiva antes de autenticar al usuario para acceder a los recursos del dominio. El proceso de autenticación que se produce entre los dominios de confianza varía según el protocolo de autenticación que se use. Los protocolos Kerberos V5 y NTLM procesan las referencias para la autenticación en un dominio de manera diferente.

### <a name="kerberos-v5-referral-processing"></a>Procesamiento de referencia en Kerberos V5

El protocolo de autenticación Kerberos V5 depende del servicio de Net Logon en los controladores de dominio para la autenticación de cliente y la información de autorización. El protocolo Kerberos se conecta a un Centro de distribución de claves (KDC) en línea y al almacén de cuentas de Active Directory para obtener vales de sesión.

El protocolo Kerberos también usa confianzas para los servicios de concesión de vales (TGS) entre dominios y para validar los certificados de atributos de privilegios (PAC) a través de un canal seguro. Este protocolo realiza la autenticación entre dominios solo con dominios kerberos de sistemas operativos que no son de Windows, como un dominio kerberos MIT, y no necesita interactuar con el servicio de Net Logon.

Si el cliente usa Kerberos V5 para la autenticación, solicita un vale al servidor en el dominio de destino de un controlador de dominio en su dominio de cuenta. El Centro de distribución de claves de Kerberos actúa como intermediario de confianza entre el cliente y el servidor, y proporciona una clave de sesión que permite a las dos partes autenticarse entre sí. Si el dominio de destino es diferente del dominio actual, el Centro de distribución de claves sigue un proceso lógico para determinar si se puede referir una solicitud de autenticación:

1. ¿El dominio del servidor que se solicita confía directamente en el dominio actual?
    * En caso afirmativo, envía al cliente una referencia al dominio solicitado.
    * En caso negativo, continúa en el paso siguiente.

2. ¿Existe una relación de confianza transitiva entre el dominio actual y el siguiente dominio en la ruta de acceso de confianza?
    * En caso afirmativo, envía al cliente una referencia al siguiente dominio en la ruta de acceso de confianza.
    * En caso negativo, envía al cliente un mensaje de inicio de sesión denegado.

### <a name="ntlm-referral-processing"></a>Procesamiento de referencia en NTLM

El protocolo de autenticación NTLM depende del servicio de Net Logon de los controladores de dominio para la autenticación de cliente y la información de autorización. Este protocolo autentica a los clientes que no usan la autenticación Kerberos. NTLM usa confianzas para pasar solicitudes de autenticación entre dominios.

Si el cliente usa NTLM para la autenticación, la solicitud inicial de autenticación va directamente desde el cliente al servidor de recursos del dominio de destino. Este servidor crea un desafío al que el cliente responde. A continuación, el servidor envía la respuesta del usuario a un controlador de dominio en su dominio de cuenta de equipo. Este controlador de dominio comprueba la cuenta de usuario en su base de datos de cuentas de seguridad.

Si la cuenta no existe en la base de datos, el controlador de dominio utiliza la siguiente lógica para determinar si se realiza la autenticación de paso a través, se reenvía la solicitud o se deniega la solicitud:

1. ¿El dominio actual tiene una relación de confianza directa con el dominio del usuario?
    * En caso afirmativo, el controlador de dominio envía las credenciales del cliente a un controlador de dominio en el dominio del usuario para la autenticación de paso a través.
    * En caso negativo, continúa en el paso siguiente.

2. ¿El dominio actual tiene una relación de confianza transitiva con el dominio del usuario?
    * En caso afirmativo, pasa la solicitud de autenticación al siguiente dominio en la ruta de acceso de confianza. Este controlador de dominio repite el proceso comprobando las credenciales del usuario en su propia base de datos de cuentas de seguridad.
    * En caso negativo, envía al cliente un mensaje de inicio de sesión denegado.

### <a name="kerberos-based-processing-of-authentication-requests-over-forest-trusts"></a>Procesamiento basado en Kerberos de solicitudes de autenticación a través de confianzas de bosque

Cuando dos bosques están conectados mediante una confianza de bosque, las solicitudes de autenticación realizadas con los protocolos Kerberos V5 o NTLM se pueden enrutar entre los bosques para proporcionar acceso a los recursos de ambos.

Cuando se establece una confianza de bosque por primera vez, cada bosque recopila todos los espacios de nombres de confianza del bosque asociado y almacena la información en un [objeto de dominio de confianza](#trusted-domain-object) (TDO). Los espacios de nombres de confianza incluyen los nombres de los árboles de dominio, los sufijos de nombre principal de usuario (UPN), los sufijos de nombre de entidad de seguridad de servicio (SPN) y los espacios de nombres de identificador de seguridad (SID) usados en el otro bosque. Los objetos TDO se replican en el catálogo global.

Para que los protocolos de autenticación puedan seguir la ruta de acceso de confianza del bosque, el nombre de entidad de seguridad de servicio (SPN) del equipo de recursos debe resolverse en una ubicación del otro bosque. Este SPN puede ser uno de los siguientes nombres:

* El nombre DNS de un host.
* El nombre DNS de un dominio.
* El nombre distintivo de un objeto de punto de conexión de servicio.

Cuando una estación de trabajo de un bosque intenta acceder a los datos de un equipo de recursos de otro bosque, el proceso de autenticación de Kerberos se pone en contacto con el controlador de dominio para obtener un vale de servicio al SPN del equipo de recursos. Una vez que el controlador de dominio consulta el catálogo global y determina que el SPN no está en el mismo bosque que el controlador de dominio, este envía una referencia para su dominio primario de vuelta a la estación de trabajo. En ese punto, la estación de trabajo consulta el vale de servicio en el dominio primario y sigue la cadena de referencia hasta que llega al dominio donde se encuentra el recurso.

En el diagrama y los pasos siguientes se proporciona una descripción detallada del proceso de autenticación de Kerberos que se utiliza cuando los equipos que ejecutan Windows intentan acceder a recursos de un equipo ubicado en otro bosque.

![Diagrama del proceso de Kerberos a través de una confianza de bosque](media/concepts-forest-trust/kerberos-over-forest-trust-process-diagram.png)

1. El *Usuario 1* inicia sesión en la *Estación de trabajo 1* con las credenciales del dominio *europe.tailspintoys.com*. A continuación, el usuario intenta acceder a un recurso compartido en el *Servidor de archivos 1* ubicado en el bosque *usa.wingtiptoys.com*.

2. La *Estación de trabajo 1* se pone en contacto con el Centro de distribución de claves de Kerberos en un controlador de dominio de su dominio, *CD secundario 1*, y solicita un vale de servicio para el SPN del *Servidor de archivos 1*.

3. El *CD secundario 1* no encuentra el SPN en la base de datos de su dominio y consulta el catálogo global para ver si está en algún dominio del bosque *tailspintoys.com*. Dado que un catálogo global está limitado a su propio bosque, no encuentra el SPN.

    A continuación, el catálogo global comprueba su base de datos para obtener información acerca de las confianzas de bosque establecidas con su bosque. Si encuentra alguna, compara los sufijos de nombre incluidos en el objeto de dominio de confianza (TDO) de la confianza de bosque con el sufijo del SPN de destino para encontrar una coincidencia. Una vez que encuentra una coincidencia, el catálogo global proporciona una sugerencia de enrutamiento de vuelta al *CD secundario 1*.

    Las sugerencias de enrutamiento ayudan a dirigir las solicitudes de autenticación hacia el bosque de destino. Las sugerencias solo se usan cuando ningún canal de autenticación tradicional, como el controlador de dominio local y después el catálogo global, puede localizar un SPN.

4. El *CD secundario 1* envía una referencia para su dominio primario de vuelta a la *Estación de trabajo 1*.

5. La *Estación de trabajo 1* se pone en contacto con un controlador de dominio del *CD raíz de bosque 1* (su dominio primario) para obtener una referencia a un controlador de dominio (*CD raíz de bosque 2*) en el dominio raíz del bosque *wingtiptoys.com*.

6. La *Estación de trabajo 1* se pone en contacto con el *CD raíz de bosque 2* en el bosque *wingtiptoys.com* para obtener un vale de servicio al servicio solicitado.

7. El *CD raíz de bosque 2* se pone en contacto con su catálogo global para encontrar el SPN; el catálogo global encuentra una coincidencia y la envía de vuelta al *CD raíz de bosque 2*.

8. A continuación, el *CD raíz de bosque 2* envía la referencia a *usa.wingtiptoys.com* de vuelta a la *Estación de trabajo 1*.

9. La *Estación de trabajo 1* se pone en contacto con el Centro de distribución de claves en el *CD secundario 2* y negocia el vale para que el *Usuario 1* consiga acceso al *Servidor de archivos 1*.

10. Una vez que la *Estación de trabajo 1* tiene un vale de servicio, lo envía al *Servidor de archivos 1*, que lee las credenciales de seguridad del *Usuario 1* y crea un token de acceso en consecuencia.

## <a name="trusted-domain-object"></a>Objeto de dominio de confianza

Cada confianza de dominio o bosque dentro de una organización se representa mediante un objeto de dominio de confianza (TDO) almacenado en el contenedor del *sistema* dentro de su dominio.

### <a name="tdo-contents"></a>Contenido del objeto de dominio de confianza

La información contenida en un objeto de dominio de confianza varía en función de si este objeto fue creado por una confianza de dominio o por una confianza de bosque.

Cuando se crea una confianza de dominio, en el objeto de dominio de confianza se representan atributos como el nombre de dominio DNS, el identificador de seguridad del dominio, el tipo de confianza, la transitividad de la confianza y el nombre de dominio recíproco. Los objetos de dominio de confianza en las relaciones de confianza de bosque almacenan atributos adicionales, para identificar todos los espacios de nombres de confianza del bosque asociado. Estos atributos incluyen los nombres de los árboles de dominio, los sufijos de nombre principal de usuario (UPN), los sufijos de nombre de entidad de seguridad de servicio (SPN) y los espacios de nombres de identificador de seguridad (SID).

Dado que las confianzas se almacenan en Active Directory como objetos de dominio de confianza, todos los dominios de un bosque tienen conocimiento de las relaciones de confianza vigentes en todo el bosque. Del mismo modo, cuando dos o más bosques se unen mediante confianzas de bosque, los dominios raíz de cada bosque tienen conocimiento de las relaciones de confianza que están en vigor en todos los dominios de los bosques de confianza.

### <a name="tdo-password-changes"></a>Cambios de contraseña de los objetos de dominio de confianza

Los dos dominios de una relación de confianza comparten una contraseña, que se almacena en el objeto de dominio de confianza en Active Directory. Como parte del proceso de mantenimiento de cuentas, el controlador de dominio que confía cambia cada 30 días la contraseña almacenada en el objeto de dominio de confianza. Dado que todas las confianzas bidireccionales son, en realidad, dos confianzas unidireccionales que van en direcciones opuestas, en ellas el proceso se produce dos veces.

Una confianza tiene un lado que confía y un lado de confianza. En el lado de confianza, se puede usar cualquier controlador de dominio grabable para el proceso. En el lado que confía, es el emulador de PDC el que realiza el cambio de contraseña.

Para cambiar una contraseña, los controladores de dominio llevan a cabo el proceso siguiente:

1. El emulador del controlador de dominio principal (PDC) del dominio que confía crea una nueva contraseña. Un controlador de dominio en el dominio de confianza nunca inicia el cambio de contraseña. Siempre lo inicia el emulador de PDC del dominio que confía.

2. El emulador de PDC del dominio que confía establece el campo *OldPassword* del objeto de dominio de confianza en el campo *NewPassword* actual.

3. El emulador de PDC del dominio que confía establece el campo *NewPassword* del objeto de dominio de confianza en la nueva contraseña. El hecho de mantener una copia de la contraseña anterior permite revertir a ella en caso de que el controlador de dominio del dominio de confianza no pueda recibir el cambio, o en caso de que el cambio no se replique antes de que se realice una solicitud que utilice la nueva contraseña de confianza.

4. El emulador de PDC del dominio que confía realiza una llamada remota a un controlador de dominio en el dominio de confianza pidiéndole que establezca la contraseña de la cuenta de confianza en la nueva contraseña.

5. El controlador de dominio del dominio de confianza cambia la contraseña de confianza a la nueva contraseña.

6. En cada lado de la confianza, las actualizaciones se replican en los demás controladores de dominio del dominio. En el dominio que confía, el cambio desencadena una replicación urgente del objeto de dominio de confianza.

Ahora la contraseña se ha cambiado en ambos controladores de dominio. La replicación normal distribuye los objetos de dominio de confianza a los demás controladores de dominio del dominio. Sin embargo, es posible que el controlador de dominio del dominio que confía cambie la contraseña sin actualizar correctamente un controlador de dominio del dominio de confianza. Este escenario puede producirse si no se pudo establecer un canal seguro, que es necesario para procesar el cambio de contraseña. También es posible que el controlador de dominio del dominio de confianza no esté disponible en algún momento durante el proceso y que no reciba la contraseña actualizada.

Para solucionar las situaciones en las que el cambio de contraseña no se comunica correctamente, el controlador de dominio del dominio que confía nunca cambia la nueva contraseña a menos que se haya autenticado correctamente (configurado un canal seguro) con la nueva contraseña. Este comportamiento explica por qué se conservan las contraseñas antigua y nueva en el objeto de dominio de confianza del dominio que confía.

Un cambio de contraseña no finaliza hasta que la autenticación que usa la contraseña se realiza correctamente. La antigua contraseña almacenada se puede utilizar en el canal seguro hasta que el controlador de dominio del dominio de confianza recibe la nueva contraseña, lo que permite efectuar un servicio ininterrumpido.

Si se produce un error en la autenticación con la nueva contraseña porque esta no es válida, el controlador del dominio que confía intenta autenticarse con la contraseña anterior. Si se autentica correctamente con la contraseña anterior, reanuda el proceso de cambio de contraseña en un plazo de 15 minutos.

Las actualizaciones de la contraseña de confianza deben replicarse en los controladores de dominio de ambos lados de la confianza en un plazo de 30 días. Si se cambia la contraseña de confianza después de 30 días y un controlador de dominio solo tiene la contraseña N-2, no podrá usar la confianza del lado que confía y no podrá crear un canal seguro en el lado de confianza.

## <a name="network-ports-used-by-trusts"></a>Puertos de red usados por las confianzas

Dado que las confianzas deben implementarse a través de varios límites de red, es posible que tengan que abarcar uno o varios firewalls. En este caso, se puede canalizar el tráfico de confianza a través de un firewall o se pueden abrir puertos específicos en el firewall para permitir el paso del tráfico.

> [!IMPORTANT]
> Active Directory Domain Services no admite la restricción del tráfico RPC de Active Directory a puertos específicos.

Lea la sección **Windows Server 2008 y versiones posteriores** del artículo de soporte técnico de Microsoft [Configuración de un firewall para dominios y confianzas de Active Directory](https://support.microsoft.com/help/179442/how-to-configure-a-firewall-for-domains-and-trusts) para obtener información acerca de los puertos necesarios para una confianza de bosque.

## <a name="supporting-services-and-tools"></a>Servicios y herramientas compatibles

Para admitir las confianzas y la autenticación, se usan algunas características y herramientas de administración adicionales.

### <a name="net-logon"></a>Net Logon

El servicio de Net Logon mantiene un canal seguro desde un equipo basado en Windows a un controlador de dominio. También se usa en los siguientes procesos relacionados con la confianza:

* Configuración y administración de confianza: Net Logon ayuda a mantener contraseñas de confianza, recopila información de confianza y comprueba las confianzas mediante la interacción con el proceso de LSA y el objeto de dominio de confianza.

    En el caso de las confianzas de bosque, la información de confianza incluye el registro de información de confianza de bosque (*FTInfo*), que contiene el conjunto de espacios de nombres que un bosque de confianza notifica que administra, anotado con un campo que indica si cada notificación es de confianza para el bosque que confía.

* Autenticación: proporciona credenciales de usuario a través de un canal seguro a un controlador de dominio y devuelve los SID de dominio y los derechos de usuario para el usuario.

* Ubicación del controlador de dominio: ayuda a encontrar o localizar controladores de dominio en un dominio o entre dominios.

* Validación de paso a través: Net Logon procesa las credenciales de los usuarios de otros dominios. Cuando un dominio que confía debe comprobar la identidad de un usuario, pasa las credenciales del usuario al dominio de confianza a través de Net Logon para su comprobación.

* Comprobación del certificado de atributos de privilegios (PAC): cuando un servidor que usa el protocolo Kerberos para la autenticación debe comprobar el PAC de un vale de servicio, envía este PAC a través del canal seguro a su controlador de dominio para la comprobación.

### <a name="local-security-authority"></a>Autoridad de seguridad local

La autoridad de seguridad local (LSA) es un subsistema protegido que mantiene información sobre todos los aspectos de la seguridad local de un sistema. La LSA, que se conoce colectivamente como una directiva de seguridad local, proporciona diversos servicios de traducción entre los nombres y los identificadores.

El subsistema de seguridad LSA proporciona servicios, tanto en modo kernel como en modo de usuario, para validar el acceso a los objetos, comprobar los privilegios de usuario y generar mensajes de auditoría. El subsistema LSA es responsable de comprobar la validez de todos los vales de sesión presentados por los servicios de los dominios de confianza o que no son de confianza.

### <a name="management-tools"></a>Herramientas de administración

Los administradores pueden usar *Dominios y confianzas de Active Directory*, *Netdom* y *Nltest* para exponer, crear, quitar o modificar confianzas.

* *Dominios y confianzas de Active Directory* es el complemento de Microsoft Management Console (MMC) que se usa para administrar confianzas de dominios, niveles funcionales de dominios y bosques, y sufijos de nombre principal de usuario.
* Las herramientas de línea de comandos *Netdom* y *Nltest* se pueden usar para buscar, mostrar, crear y administrar confianzas. Estas herramientas se comunican directamente con la autoridad LSA en un controlador de dominio.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los bosques de recursos, consulte [Funcionamiento de la confianza de bosque en Azure AD DS][concepts-trust].

Para empezar a crear un dominio administrado con un bosque de recursos, consulte [Creación y configuración de un dominio administrado de Azure AD DS][tutorial-create-advanced]. Después puede continuar en [Creación de una confianza de bosque de salida a un dominio local][create-forest-trust].

<!-- LINKS - INTERNAL -->
[concepts-trust]: concepts-forest-trust.md
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-forest-trust]: tutorial-create-forest-trust.md
