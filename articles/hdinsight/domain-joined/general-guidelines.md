---
title: Directrices generales de seguridad de la empresa en Azure HDInsight
description: Presentamos algunos procedimientos recomendados que facilitan la implementación y administración de Enterprise Security Package.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 8d97886232eecc369746e33df484cbfb9d40da72
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530272"
---
# <a name="enterprise-security-general-information-and-guidelines-in-azure-hdinsight"></a>Directrices generales e información de seguridad de la empresa en Azure HDInsight

Al implementar un clúster de HDInsight seguro, hay algunos procedimientos recomendados que facilitan la implementación y administración de clústeres. A continuación, se tratan algunas directrices e información generales.

## <a name="use-of-secure-cluster"></a>Uso de un clúster seguro

### <a name="recommended"></a>Recomendado

* Varios usuarios usarán el clúster al mismo tiempo.
* Los usuarios tienen distintos niveles de acceso a los mismos datos.

### <a name="not-necessary"></a>No es necesario

* Si va a ejecutar solo trabajos automatizados (como una cuenta de usuario único), un clúster estándar es suficiente.
* Puede realizar la importación de datos mediante un clúster estándar y usar la misma cuenta de almacenamiento en otro clúster seguro en el que los usuarios puedan ejecutar trabajos de análisis.

## <a name="use-of-local-account"></a>Uso de una cuenta local

* Si utiliza una cuenta de usuario compartida o una cuenta local, será difícil identificar quién usó la cuenta para cambiar la configuración o el servicio.
* El uso de cuentas locales es problemático cuando los usuarios ya no forman parte de la organización.

## <a name="ranger"></a>Ranger

### <a name="policies"></a>Directivas

* De manera predeterminada, Ranger usa **Denegar** como directiva.

* Cuando el acceso a los datos se realiza a través de un servicio en el que se ha habilitado la autorización:
  * Se invoca un complemento de autorización de Ranger y se le aplica el contexto de la solicitud.
  * Ranger aplica las directivas configuradas para el servicio. Si las directivas de Ranger fallan, la comprobación de acceso se aplaza en el sistema de archivos. Algunos servicios, como MapReduce, solo comprueban si el archivo o la carpeta son propiedad del mismo usuario que envía la solicitud. Servicios como Hive comprueban si la propiedad coincide o se adecúa a los permisos del sistema de archivos (`rwx`).

* En el caso de Hive, además de tener los permisos para crear, actualizar o eliminar permisos, el usuario debe tener permisos `rwx` en el directorio del almacenamiento y en todos los subdirectorios.

* Las directivas se pueden aplicar a grupos (preferiblemente) en lugar de individuos.

* El autorizador de Ranger evaluará todas las directivas de Ranger de dicho servicio para cada solicitud. Esta evaluación podría afectar al tiempo que se tarda en aceptar el trabajo o la consulta.

### <a name="storage-access"></a>Acceso al almacenamiento

* Si el tipo de almacenamiento es WASB, no implica ningún token de OAuth.
* Si Ranger ha llevado a cabo la autorización, el acceso al almacenamiento se produce mediante la identidad administrada.
* Si Ranger no ha llevado a cabo ninguna autorización, el acceso al almacenamiento se produce mediante el token de OAuth del usuario.

### <a name="hierarchical-name-space"></a>Espacio de nombres jerárquico

Cuando el espacio de nombres jerárquico no se ha habilitado:

* No hay permisos heredados.
* El único permiso del sistema de archivos que funciona es el rol de Azure **Storage Data XXXX**, que se va a asignar al usuario directamente en Azure Portal.

### <a name="default-hdfs-permissions"></a>Permisos de HDFS predeterminados

* De manera predeterminada, los usuarios no tienen acceso a la carpeta **/** en HDFS (deben estar en el rol de propietario de blob de almacenamiento para poder obtener acceso).
* En el caso del directorio de almacenamiento provisional para MapReduce y otros, se crea un directorio específico del usuario y se proporcionan permisos `sticky _wx`. Los usuarios pueden crear archivos y carpetas debajo, pero no pueden ver otros elementos.

### <a name="url-auth"></a>Autenticación de URL

Si la autenticación de URL está habilitada:

* La configuración incluirá los prefijos que se cubren en la autenticación de URL (como `adl://`).
* Si el acceso es para este URL, Ranger comprobará si el usuario está en la lista de permitidos.
* Ranger no comprobará ninguna de las directivas detalladas.

## <a name="resource-groups"></a>Grupos de recursos

Use un nuevo grupo de recursos para cada clúster para poder distinguir entre los recursos de clúster.

## <a name="nsgs-firewalls-and-internal-gateway"></a>NSG, firewalls y puerta de enlace interna

* Use grupos de seguridad de red (NSG) para bloquear redes virtuales.
* Use firewall para controlar las directivas de acceso de salida.
* Use la puerta de enlace interna que no esté abierta para Internet pública.

## <a name="azure-active-directory"></a>Azure Active Directory

[Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) es un servicio de administración de acceso y de identidades basado en la nube de Microsoft.

### <a name="policies"></a>Directivas

* Deshabilite la directiva de acceso condicional mediante la directiva basada en direcciones IP. Esto requiere la habilitación de puntos de conexión de servicio en las redes virtuales en las que se implementan los clústeres. Si usa un servicio externo para MFA (diferente de AAD), la directiva basada en direcciones IP no funcionará.

* Se requiere la directiva `AllowCloudPasswordValidation` para los usuarios federados. Dado que HDInsight usa el nombre de usuario o la contraseña directamente para obtener tokens de Azure AD, esta directiva debe estar habilitada para todos los usuarios federados.

* Habilite los puntos de conexión de servicio si se requiere la omisión del acceso condicional mediante direcciones IP de confianza.

### <a name="groups"></a>Grupos

* Implemente siempre los clústeres con un grupo.
* Use Azure AD para administrar las pertenencias a grupos (más fácil que intentar administrar los servicios individuales en el clúster).

### <a name="user-accounts"></a>Cuentas de usuario

* Use una cuenta de usuario única para cada escenario. Por ejemplo, use una cuenta para importar, otra para realizar consultas u otra para procesar trabajos.
* Use directivas de Ranger basadas en grupos en lugar de directivas individuales.
* Disponga de un plan sobre cómo administrar los usuarios que ya no deben tener acceso a los clústeres.

## <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services

[Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md) (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, protocolo ligero de acceso a directorios (LDAP) y autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory.

Azure AD DS es necesario para que los clústeres seguros se unan a un dominio.
HDInsight no puede depender de controladores de dominio locales ni personalizados, ya que presenta demasiados puntos de error, uso compartido de credenciales, permisos de DNS, etc. Para más información, consulte [Preguntas más frecuentes de Azure AD DS](../../active-directory-domain-services/faqs.md).

### <a name="azure-ad-ds-instance"></a>Instancia de Azure AD DS

* Cree la instancia con `.onmicrosoft.com domain`. De esta forma, no habrá varios servidores DNS sirviendo al dominio.
* Cree un certificado autofirmado para los LDAP y cárguelo en Azure AD DS.
* Use una red virtual emparejada para implementar clústeres (será útil cuando tenga una gran cantidad de equipos implementando clústeres de HDInsight ESP). Así se garantiza que no sea necesario abrir puertos (NSG) en la red virtual con el controlador de dominio.
* Configure el DNS de la red virtual correctamente (el nombre de dominio de Azure AD DS debe resolverse sin entradas de archivo de hosts).
* Si va a restringir el tráfico saliente, asegúrese de que ha leído la información del [soporte técnico para firewall en HDInsight](../hdinsight-restrict-outbound-traffic.md).

### <a name="properties-synced-from-azure-ad-to-azure-ad-ds"></a>Propiedades sincronizadas de Azure AD a Azure AD DS

* Azure AD conecta las sincronizaciones del entorno local con Azure AD.
* Azure AD DS sincroniza desde Azure AD.

Azure AD DS sincroniza objetos desde Azure AD periódicamente. La hoja de Azure AD DS en Azure Portal muestra el estado de sincronización. Durante cada fase de la sincronización, es posible que algunas propiedades únicas entren en conflicto y cambien de nombre. Preste atención a la asignación de propiedad de Azure AD a Azure AD DS.

Para obtener más información, consulte [Rellenado de UserPrincipalName de Azure AD](../../active-directory/hybrid/plan-connect-userprincipalname.md) y [Cómo funciona la sincronización de Azure AD DS](../../active-directory-domain-services/synchronization.md).

### <a name="password-hash-sync"></a>Sincronización de hash de contraseñas

* Las contraseñas se sincronizan de forma diferente desde otros tipos de objetos. En Azure AD y Azure AD DS solo se sincronizan valores hash de contraseña no reversible.
* La sincronización del entorno local a Azure AD debe habilitarse a través de AD Connect.
* La sincronización de Azure AD a Azure AD DS es automática (las latencias son inferiores a 20 minutos).
* Los valores hash de contraseña solo se sincronizan cuando se cambia una contraseña. Al habilitar la sincronización de hash de contraseña, las contraseñas existentes no se sincronizan, ya que se almacenan de forma irreversible. Al cambiar la contraseña, se sincronizan los valores hash de contraseña.

### <a name="computer-objects-location"></a>Ubicación de los objetos de equipo

Cada clúster virtual está asociado a una única unidad organizativa. Un usuario interno se aprovisiona en la unidad organizativa. Todos los nodos están unidos a un dominio en la misma unidad organizativa.

### <a name="active-directory-administrative-tools"></a>Herramientas administrativas de Azure Active Directory

Para ver los pasos sobre cómo instalar las herramientas administrativas de Azure Active Directory en una VM de Windows Server, consulte [Instalación de herramientas administrativas](../../active-directory-domain-services/tutorial-create-management-vm.md).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="cluster-creation-fails-repeatedly"></a>La creación del clúster falla repetidamente

Motivos más comunes:

* La configuración de DNS no es correcta y se produce un error en la unión a un dominio de nodos de clúster.
* Los NSG son demasiado restrictivos, lo cual evita la unión a un dominio.
* La identidad administrada no tiene permisos suficientes.
* El nombre del clúster no es único en los seis primeros caracteres (debido a un clúster activo o uno eliminado).

## <a name="authentication-setup-and-configuration"></a>Instalación y configuración de la autenticación

### <a name="user-principal-name-upn"></a>Nombre principal de usuario (UPN)

* Use minúsculas en todos los servicios: los UPN no distinguen mayúsculas de minúsculas en los clústeres ESP.
* El prefijo UPN debe coincidir con SAMAccountName en Azure AD-DS. No es necesario buscar coincidencias con el campo de correo.

### <a name="ldap-properties-in-ambari-configuration"></a>Propiedades de LDAP en la configuración de Ambari

Para obtener una lista completa de las propiedades de Ambari que afectan a la configuración del clúster de HDInsight, consulte [Configuración de la autenticación LDAP de Ambari](https://ambari.apache.org/1.2.1/installing-hadoop-using-ambari/content/ambari-chap2-4.html).

## <a name="next-steps"></a>Pasos siguientes

* [Configuraciones de Enterprise Security Package con Azure Active Directory Domain Services en Azure HDInsight](./apache-domain-joined-configure-using-azure-adds.md)

* [Sincronización de los usuarios de Azure Active Directory en un clúster de HDInsight](../hdinsight-sync-aad-users-to-cluster.md).
