---
title: Introducción al control de acceso en Data Lake Storage Gen1 | Microsoft Docs
description: Obtenga información sobre los aspectos básicos del modelo de control de acceso de Azure Data Lake Storage Gen1, que deriva de HDFS.
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: d16f8c09-c954-40d3-afab-c86ffa8c353d
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: 11629338a808ae0f83ac513b6475dce7a53814da
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88190153"
---
# <a name="access-control-in-azure-data-lake-storage-gen1"></a>Control de acceso en Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 implementa un modelo de control de acceso que se deriva de HDFS y que, a su vez, se deriva del modelo de control de acceso POSIX. Este artículo resume los datos básicos del modelo de control de acceso de Data Lake Storage Gen1. 

## <a name="access-control-lists-on-files-and-folders"></a>Listas de control de acceso en archivos y carpetas

Hay dos tipos de listas de control de acceso (ACL): **ACL de acceso** y **ACL predeterminadas**.

* **ACL de acceso**: controlan el acceso a un objeto. Tanto los archivos como las carpetas tienen ACL de acceso.

* **ACL predeterminadas**: una "plantilla" de las ACL asociadas a una carpeta que determinan las ACL de acceso de todos los elementos secundarios que se crean en dicha carpeta. Los archivos no tienen ACL predeterminadas.


Tanto las ACL de acceso como las ACL predeterminadas tienen la misma estructura.



> [!NOTE]
> El cambio de la ACL predeterminada en un elemento primario no afecta a la ACL de acceso o a la ACL predeterminada de los elementos secundarios que ya existen.
>
>

## <a name="permissions"></a>Permisos

Los permisos de un objeto del sistema de archivos son de **lectura**, **escritura** y **ejecución**, y se pueden usar en archivos y carpetas, como se muestra en la tabla siguiente:

|            |    Archivo     |   Carpeta |
|------------|-------------|----------|
| **Lectura (R)** | Puede leer el contenido de un archivo | Requiere permisos de **lectura** y **ejecución** para enumerar el contenido de la carpeta.|
| **Escritura (W)** | Puede escribir o anexar a un archivo | Requiere permisos de **lectura** y **ejecución** para crear elementos secundarios en una carpeta. |
| **Ejecución (X)** | No significa nada en el contexto de Data Lake Storage Gen1 | Se requiere para atravesar los elementos secundarios de una carpeta. |

### <a name="short-forms-for-permissions"></a>Formas abreviadas de los permisos

**RWX** se usa para indicar **Lectura + Escritura + Ejecución**. Existe un formato numérico más condensado en el que **Lectura = 4**, **Escritura = 2** y **Ejecución = 1**, y su suma representa los permisos. A continuación se muestran algunos ejemplos.

| Formato numérico | Formato abreviado |      Qué significa     |
|--------------|------------|------------------------|
| 7            | `RWX`        | Lectura + Escritura + Ejecución |
| 5            | `R-X`        | Lectura + ejecución         |
| 4            | `R--`        | Lectura                   |
| 0            | `---`        | Sin permisos         |


### <a name="permissions-do-not-inherit"></a>Los permisos no se heredan

En el modelo de estilo de POSIX usado por Data Lake Storage Gen1, los permisos de un elemento se almacenan en el propio elemento. En otras palabras, los permisos de un elemento no se pueden heredar de los elementos primarios.

## <a name="common-scenarios-related-to-permissions"></a>Escenarios comunes relacionados con los permisos

A continuación, hay algunos escenarios comunes para ayudarle a entender qué permisos se necesitan para realizar ciertas operaciones en una cuenta de Data Lake Storage Gen1.

| Operación | Object              |    /      | Seattle/   | Portland/   | Data.txt       |
|-----------|---------------------|-----------|------------|-------------|----------------|
| Lectura      | Data.txt            |   `--X`   |   `--X`    |  `--X`      | `R--`          |
| Anexar a | Data.txt            |   `--X`   |   `--X`    |  `--X`      | `RW-`          |
| Eliminar    | Data.txt            |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Crear    | Data.txt            |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| List      | /                   |   `R-X`   |   `---`    |  `---`      | `---`          |
| List      | /Seattle/           |   `--X`   |   `R-X`    |  `---`      | `---`          |
| List      | /Seattle/Portland/  |   `--X`   |   `--X`    |  `R-X`      | `---`          |


> [!NOTE]
> Para eliminar un archivo no es preciso tener permisos de escritura en él, siempre y cuando se cumplan las dos condiciones anteriores.
>
>


## <a name="users-and-identities"></a>Usuarios e identidades

Todos los archivos y carpetas tienen permisos distintos para estas identidades:

* El usuario propietario
* El grupo propietario
* Usuarios designados
* Grupos designados
* Los restantes usuarios

Las identidades de usuarios y grupos son las identidades de Azure Active Directory (Azure AD). Por tanto, a menos que se indique lo contrario, un "usuario", en el contexto de Data Lake Storage Gen1, puede referirse a un usuario o a un grupo de seguridad de Azure AD.

### <a name="the-super-user"></a>El superusuario

Un superusuario es el usuario con más derechos en la cuenta de Data Lake Storage Gen1. Un superusuario:

* Tiene permisos de lectura, escritura y ejecución (RWX) en **todos** los archivos y carpetas.
* Puede cambiar los permisos de cualquier archivo o carpeta.
* Puede cambiar el usuario propietario o el grupo propietario de cualquier archivo o carpeta.

Todos los usuarios que forman parte del rol **Propietarios** de una cuenta de Data Lake Storage Gen1 son automáticamente superusuarios.

### <a name="the-owning-user"></a>El usuario propietario

El usuario que creó el elemento es automáticamente el usuario propietario del elemento. Un usuario propietario puede:

* Cambiar los permisos de un archivo que le pertenece.
* Cambiar el grupo propietario de un archivo que le pertenece, siempre que el usuario propietario también sea miembro del grupo de destino.

> [!NOTE]
> El usuario propietario *no puede* cambiar el usuario propietario de un archivo o carpeta. Solo los superusuarios pueden cambiar el usuario propietario de un archivo o carpeta.
>
>

### <a name="the-owning-group"></a>El grupo propietario

**Información preliminar**

En las ACL de POSIX, todos los usuarios están asociados a un "grupo principal". Por ejemplo, el usuario "alice" puede pertenecer al grupo "finance". Alice puede pertenecer a varios grupos, pero uno de ellos siempre se designa como su grupo principal. En POSIX, cuando Alice crea un archivo, el grupo propietario de dicho archivo se establece como su grupo principal, que en este caso es "finance". De lo contrario, el grupo propietario se comporta de forma similar a los permisos asignados para otros usuarios o grupos.

Dado que no hay ningún "grupo principal" asociado a los usuarios de Data Lake Storage Gen1, el grupo propietario se asigna como sigue.

**Asignar el grupo propietario de un nuevo archivo o carpeta**

* **Caso 1**: la carpeta raíz "/". Esta carpeta se crea cuando se crea una cuenta de Data Lake Storage Gen1. En este caso, el grupo propietario se establece en un GUID con solo ceros.  Este valor no permite ningún acceso.  Es un marcador de posición hasta el momento en el que se asigna un grupo.
* **Caso 2** (cada dos casos): cuando se crea un elemento, se copia el grupo propietario de la carpeta primaria.

**Cambiar el grupo propietario**

El grupo propietario se puede cambiar por:
* Cualquier superusuario.
* El usuario propietario, si el usuario propietario también es miembro del grupo de destino.

> [!NOTE]
> El grupo propietario *no puede* cambiar las ACL de un archivo o carpeta.
>
> Para las cuentas creadas hasta septiembre de 2018, el grupo propietario se establecía en el usuario autor de la cuenta en el caso de la carpeta raíz del **Caso 1** anterior.  Una cuenta de usuario único no es válida para proporcionar permisos a través del grupo propietario; por lo tanto, esta configuración predeterminada no concede permisos. Puede asignar este permiso a un grupo de usuarios válidos.


## <a name="access-check-algorithm"></a>Algoritmo de comprobación de acceso

El siguiente seudocódigo representa el algoritmo de comprobación de acceso de las cuentas de Data Lake Storage Gen1.

```
def access_check( user, desired_perms, path ) : 
  # access_check returns true if user has the desired permissions on the path, false otherwise
  # user is the identity that wants to perform an operation on path
  # desired_perms is a simple integer with values from 0 to 7 ( R=4, W=2, X=1). User desires these permissions
  # path is the file or folder
  # Note: the "sticky bit" is not illustrated in this algorithm
  
# Handle super users.
  if (is_superuser(user)) :
    return True

  # Handle the owning user. Note that mask IS NOT used.
  entry = get_acl_entry( path, OWNER )
  if (user == entry.identity)
      return ( (desired_perms & entry.permissions) == desired_perms )

  # Handle the named users. Note that mask IS used.
  entries = get_acl_entries( path, NAMED_USER )
  for entry in entries:
      if (user == entry.identity ) :
          mask = get_mask( path )
          return ( (desired_perms & entry.permmissions & mask) == desired_perms)

  # Handle named groups and owning group
  member_count = 0
  perms = 0
  entries = get_acl_entries( path, NAMED_GROUP | OWNING_GROUP )
  for entry in entries:
    if (user_is_member_of_group(user, entry.identity)) :
      member_count += 1
      perms | =  entry.permissions
  if (member_count>0) :
    return ((desired_perms & perms & mask ) == desired_perms)
 
  # Handle other
  perms = get_perms_for_other(path)
  mask = get_mask( path )
  return ( (desired_perms & perms & mask ) == desired_perms)
```

### <a name="the-mask"></a>La máscara

Como se muestra en el algoritmo de comprobación de acceso, la máscara limita el acceso a **usuarios con nombre**, el **grupo propietario** y **grupos con nombre**.  

> [!NOTE]
> En una nueva cuenta de Data Lake Storage Gen1, la máscara de la ACL de acceso de la carpeta raíz ("/") adopta como predeterminado el valor RWX.
>
>

### <a name="the-sticky-bit"></a>El bit persistente

El bit persistente es una característica más avanzada de un sistema de archivos POSIX. En el contexto de Data Lake Storage Gen1, es improbable que se necesite el bit persistente. En resumen, si el bit persistente está habilitado en una carpeta, solo el usuario propietario del elemento secundario puede eliminarlo o cambiarle el nombre.

El bit persistente no se muestra en Azure Portal.

## <a name="default-permissions-on-new-files-and-folders"></a>Permisos predeterminados en archivos y carpetas nuevos

Cuando se crea un nuevo archivo o carpeta en una carpeta existente, la ACL predeterminada de la carpeta primaria determina:

- La ACL predeterminada y la ACL de acceso de una carpeta secundaria.
- La ACL de acceso de un archivo secundario (los archivos no tienen ninguna ACL predeterminada).

### <a name="umask"></a>umask

Al crear un archivo o carpeta, se usa umask para modificar cómo se establecen las ACL predeterminadas en el elemento secundario. umask es un valor de 9 bits en las carpetas principales que contiene un valor RWX para **usuario propietario**, **grupo propietario** y **otro**.

El valor de umask para Azure Data Lake Storage Gen1 es una constante establecida en 007. Este valor se convierte en

| componente umask     | Formato numérico | Formato abreviado | Significado |
|---------------------|--------------|------------|---------|
| umask.owning_user   |    0         |   `---`      | En el caso del usuario propietario, copie la ACL predeterminada del elemento principal en la ACL de acceso del elemento secundario | 
| umask.owning_group  |    0         |   `---`      | En el caso del grupo propietario, copie la ACL predeterminada del elemento principal en la ACL de acceso del elemento secundario | 
| umask.other         |    7         |   `RWX`      | En el caso de otro, quite todos los permisos en la ACL de acceso del elemento secundario |

El valor de umask usado por Azure Data Lake Storage Gen1 significa realmente que el valor de otro nunca se transmite de forma predeterminada en los nuevos objetos secundarios, independientemente de lo que indique la ACL predeterminada. 

El siguiente pseudocódigo muestra cómo se aplica umask al crear las ACL de un elemento secundario.

```
def set_default_acls_for_new_child(parent, child):
    child.acls = []
    for entry in parent.acls :
        new_entry = None
        if (entry.type == OWNING_USER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_user))
        elif (entry.type == OWNING_GROUP) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_group))
        elif (entry.type == OTHER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.other))
        else :
            new_entry = entry.clone(perms = entry.perms )
        child_acls.add( new_entry )
```

## <a name="common-questions-about-acls-in-data-lake-storage-gen1"></a>Preguntas frecuentes acerca de las ACL en Data Lake Storage Gen1

### <a name="do-i-have-to-enable-support-for-acls"></a>¿Es preciso habilitar la compatibilidad con las ACL?

No. El control de acceso mediante las ACL siempre está activado en las cuentas de Data Lake Storage Gen1.

### <a name="which-permissions-are-required-to-recursively-delete-a-folder-and-its-contents"></a>¿Qué permisos son necesarios para eliminar de forma recursiva una carpeta y su contenido?

* La carpeta primaria debe tener permisos de **escritura y ejecución**.
* Tanto la carpeta que se va a eliminar como todas las carpetas dentro de ella, requieren permisos de **lectura, escritura y ejecución**.

> [!NOTE]
> No necesita permisos de escritura para eliminar archivos en carpetas. Además, la carpeta raíz "/" **nunca** se puede eliminar.
>
>

### <a name="who-is-the-owner-of-a-file-or-folder"></a>¿Quién es el propietario de un archivo o carpeta?

El creador de un archivo o carpeta se convierte en el propietario.

### <a name="which-group-is-set-as-the-owning-group-of-a-file-or-folder-at-creation"></a>¿Qué grupo se establece como grupo propietario de un archivo o carpeta al crearlo?

El grupo propietario se copia de la carpeta primaria en la que se crea el archivo o carpeta.

### <a name="i-am-the-owning-user-of-a-file-but-i-dont-have-the-rwx-permissions-i-need-what-do-i-do"></a>Soy el usuario propietario de un archivo, pero no tengo los permisos de RWX que necesito. ¿Qué puedo hacer?

El usuario propietario puede cambiar los permisos del archivo y concederse los permisos de RWX que necesite.

### <a name="when-i-look-at-acls-in-the-azure-portal-i-see-user-names-but-through-apis-i-see-guids-why-is-that"></a>Cuando examino las ACL en Azure Portal veo los nombres de usuario, pero cuando lo hago mediante API veo GUID, ¿por qué?

Las entradas de las ACL se almacenan como GUID que corresponden a los usuarios en Azure AD. Las API devuelven los GUID tal cual. Azure Portal intenta que las ACL sean más fáciles de usar convirtiendo los GUID en nombres descriptivos siempre que sea posible.

### <a name="why-do-i-sometimes-see-guids-in-the-acls-when-im-using-the-azure-portal"></a>¿Por qué a veces veo GUID en las ACL al usar Azure Portal?

Cuando el usuario ya no existe en Azure AD, se mostrará un GUID. Normalmente esto sucede cuando el usuario ha dejado la empresa o si se ha eliminado su cuenta de Azure AD.

### <a name="does-data-lake-storage-gen1-support-inheritance-of-acls"></a>¿Admite Data Lake Storage Gen1 la herencia de ACL?

No, pero las ACL predeterminada pueden usarse para establecer las ACL para archivos y carpetas secundarios recién creados en la carpeta principal.  

### <a name="where-can-i-learn-more-about-posix-access-control-model"></a>¿Dónde puedo obtener más información sobre el modelo de control de acceso POSIX?

* [POSIX Access Control Lists on Linux](https://www.linux.com/news/posix-acls-linux) (Listas de control de acceso de POSIX en Linux)
* [HDFS Permission Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html) (Guía de permisos de HDFS)
* [POSIX FAQ (PREGUNTAS MÁS FRECUENTES SOBRE POSIX)](https://www.opengroup.org/austin/papers/posix_faq.html)
* [POSIX 1003.1 2008](https://standards.ieee.org/findstds/standard/1003.1-2008.html)
* [POSIX 1003.1 2013](https://pubs.opengroup.org/onlinepubs/9699919799.2013edition/)
* [POSIX 1003.1 2016](https://pubs.opengroup.org/onlinepubs/9699919799.2016edition/)
* [ACL de POSIX en Ubuntu](https://help.ubuntu.com/community/FilePermissionsACLs)
* [ACL: Using Access Control Lists on Linux](https://bencane.com/2012/05/27/acl-using-access-control-lists-on-linux/) (ACL: uso de listas de control de acceso en Linux)

## <a name="see-also"></a>Consulte también

* [Introducción a Azure Data Lake Storage Gen1](data-lake-store-overview.md)
