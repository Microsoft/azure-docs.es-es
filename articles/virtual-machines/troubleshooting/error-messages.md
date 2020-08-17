---
title: Códigos de error habituales de máquina virtual en Azure | Microsoft Docs
description: Información sobre algunos de los códigos de error habituales que se producen al aprovisionar máquinas virtuales y administrarlas en Azure
services: virtual-machines
documentationcenter: ''
author: xujing-ms
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure
ms.date: 5/22/2017
ms.author: xujing
ms.openlocfilehash: 52508a6820ce0cbbbe3a0341a99894f8b92b1645
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831233"
---
# <a name="understand-common-error-messages-when-you-manage-virtual-machines-in-azure"></a>Información de mensajes de error habituales al administrar máquinas virtuales en Azure

En este artículo se describen algunos de los códigos y mensajes de error más habituales que pueden aparecer al crear máquinas virtuales (VM) y administrarlas en Azure.

>[!NOTE]
> Puede dejar comentarios en esta página para proporcionar información o a través del [foro de comentarios acerca de Azure](https://feedback.azure.com/forums/216843-virtual-machines) con la etiqueta #azerrormessage.

## <a name="error-response-format"></a>Formato de respuestas de error 
Las máquinas virtuales de Azure utilizan el siguiente formato JSON para las respuestas de error:

```json
{
  "status": "status code",
  "error": {
    "code":"Top level error code",
    "message":"Top level error message",
    "details":[
     {
      "code":"Inner level error code",
      "message":"Inner level error message"
     }
    ]
   }
}
```

Una respuesta de error siempre incluye un código de estado y un objeto de error. Cada objeto de error contiene siempre un código de error y un mensaje. Si la máquina virtual se crea mediante una plantilla, el objeto de error también tiene una sección de detalles que contiene un nivel interno de códigos de error y un mensaje. Normalmente, el nivel más interno del mensaje es el error raíz.


## <a name="common-virtual-machine-management-errors"></a>Errores habituales en la administración de máquinas virtuales

En esta sección se muestran los mensajes de error habituales que pueden producirse al administrar máquinas virtuales:

|  Código de error  |  Mensaje de error  |  
|  :------| :-------------|  
|  AcquireDiskLeaseFailed  |  No se pudo adquirir la concesión al crear el disco "{0}" mediante un blob con el identificador URI {1}. El blob ya está en uso.  |  
|  AllocationFailed  |  Error en la asignación. Pruebe a reducir el tamaño de la máquina virtual o el número de máquinas virtuales, vuelva a intentarlo más tarde o intente realizar la implementación en otro conjunto de disponibilidad o en otra ubicación de Azure.  |  
|  AllocationFailed  |  No se pudo realizar la asignación de la máquina virtual debido a un error interno. Vuelva a intentarlo más tarde o pruebe a realizar la implementación en otra ubicación.  |
|  ArtifactNotFound  |  No se encontró la extensión de máquina virtual con el publicador "{0}" y el tipo "{1}" en la ubicación "{2}".  |
|  ArtifactNotFound  |  La extensión de publicador "{0}", tipo "{1}" y versión de controlador de tipo "{2}" no se ha podido encontrar en el repositorio de extensiones.  |
|  ArtifactVersionNotFound  |  No se encontró ninguna versión en el repositorio de artefactos que se ajuste a la versión solicitada "{0}".  |
|  ArtifactVersionNotFound  |  No se encontró ninguna versión en el repositorio de artefactos que se ajuste a la versión solicitada "{0}" para la extensión de máquina virtual con el publicador "{1}" y el tipo "{2}".  |
|  AttachDiskWhileBeingDetached  |  El disco de datos "{0}" no se puede asociar a la máquina virtual "{1}" porque en la actualidad el disco está en proceso de desasociación. Espere hasta que el disco esté completamente desasociado para volver a intentarlo.  |
|  BadRequest  |  En esta región aún no se admiten conjuntos de disponibilidad alineados.  |
|  BadRequest  |  No se admiten la incorporación de una máquina virtual con discos administrados a un conjunto de disponibilidad no administrado ni la de una máquina virtual con discos basados en blobs a un conjunto de disponibilidad administrado. Cree un conjunto de disponibilidad con la propiedad "managed" establecida para agregarle una máquina virtual con discos administrados.  |
|  BadRequest  |  En esta región no se admiten discos administrados.  |
|  BadRequest  |  No se admiten varias instancias de VMExtension por controlador para un sistema operativo del tipo "{0}". VMExtension "{1}" con el controlador "{2}" ya se ha agregado o se ha especificado en la entrada.  |
|  BadRequest  |  La operación "{0}" no se admite en el recurso "{1}" con discos administrados.  |
|  CertificateImproperlyFormatted  |  La representación JSON del secreto recuperada de {0} tiene un campo de datos que es un archivo PFX con un formato incorrecto, o bien la contraseña proporcionada no decodifica el archivo PFX correctamente.  |
|  CertificateImproperlyFormatted  |  Los datos recuperados de {0} no se pueden deserializar en JSON.  |
|  Conflicto  |  Solo se permite el cambio de tamaño del disco al crear una VM o al desasignar la VM.  |
|  ConflictingUserInput  |  El disco "{0}" no se puede asociar, dado que ya pertenece a la máquina virtual "{1}".  |
|  ConflictingUserInput  |  Los grupos de recursos de origen y de destino son los mismos.  |
|  ConflictingUserInput  |  Las cuentas de almacenamiento de origen y de destino del disco {0} son distintas.  |
|  ContainerAlreadyOnLease  |  Ya hay una concesión en el contenedor de almacenamiento que contiene el blob con el identificador URI {0}.  |
|  CrossSubscriptionMoveWithKeyVaultResources  |  La solicitud de recursos de Move contiene recursos de KeyVault a los que hacen referencia uno o varios {0} en la solicitud. En la actualidad, esto no se admite en la instancia de Move entre suscripciones. Compruebe los detalles del error en los identificadores de recursos de KeyVault.  |
|  DiagnosticsOperationInternalError  |  Se produjo un error interno al procesar el perfil de diagnóstico de la máquina virtual {0}.  |
|  DiskBlobAlreadyInUseByAnotherDisk  |  El blob {0} ya lo usa otro disco que pertenece a la máquina virtual "{1}". Puede examinar los metadatos del blob para obtener información de referencia del disco.  |
|  DiskBlobNotFound  |  No se encuentra el blob VHD con el identificador URI {0} para el disco "{1}".  |
|  DiskBlobNotFound  |  No se encuentra el blob VHD con el identificador URI {0}.  |
|  DiskEncryptionKeySecretMissingTags  |  El secreto {0} no tiene las etiquetas {1}. Actualice la versión del secreto, agregue las etiquetas necesarias y vuelva a intentarlo.  |
|  DiskEncryptionKeySecretUnwrapFailed  |  Error al realizar el desajuste del valor {0} del secreto mediante la clave {1}.  |
|  DiskImageNotReady  |  La imagen de disco {0} se encuentra en estado {1}. Vuelva a intentarlo cuando la imagen esté lista.  |
|  DiskPreparationError  |  Se produjeron uno o varios errores al preparar los discos de la máquina virtual. Para más información, consulte la vista de la instancia del disco.  |
|  DiskProcessingError  |  El procesamiento del disco se ha detenido, ya que la máquina virtual tiene otros discos en los discos con errores.  |
|  ImageBlobNotFound  |  No se encuentra el blob VHD con el identificador URI {0} para el disco "{1}".  |
|  ImageBlobNotFound  |  No se encuentra el blob VHD con el identificador URI {0}.  |
|  IncorrectDiskBlobType  |  Los blobs de disco solo pueden ser del tipo blob en páginas. El blob {0} del disco "{1}" es de tipo blob en bloques.  |
|  IncorrectDiskBlobType  |  Los blobs de disco solo pueden ser del tipo blob en páginas. El blob {0} es de tipo "{1}".  |
|  IncorrectImageBlobType  |  Los blobs de disco solo pueden ser del tipo blob en páginas. El blob {0} del disco "{1}" es de tipo blob en bloques.  |
|  IncorrectImageBlobType  |  Los blobs de disco solo pueden ser del tipo blob en páginas. El blob {0} es de tipo "{1}".  |
|  InternalOperationError  |  No se pudo resolver la cuenta de almacenamiento {0}. Asegúrese de que se creó mediante el proveedor de recursos de almacenamiento en la misma ubicación que el recurso de proceso.  |
|  InternalOperationError  |  Error en las tareas de búsqueda del objetivo de {0}.  |
|  InternalOperationError  |  Error al validar el perfil de red de la máquina virtual "{0}".  |
|  InvalidAccountType  |  El valor de AccountType {0} no es válido.  |
|  InvalidParameter  |  El valor del parámetro {0} no es válido.  |
|  InvalidParameter  |  La contraseña de administrador especificada no está permitida.  |
|  InvalidParameter  |  "La contraseña proporcionada debe tener entre {0} y {1} caracteres y debe cumplir al menos {2} de los siguientes requisitos de complejidad de contraseñas: <ol><li> Contiene una letra mayúscula</li><li>Contiene una letra minúscula</li><li>Contiene un dígito numérico</li><li>Contiene un carácter especial</li></ol>  |
|  InvalidParameter  |  El nombre de usuario de administrador especificado no está permitido.  |
|  InvalidParameter  |  No se puede asociar un disco de SO existente si la máquina virtual se crea desde una plataforma o imagen de usuario.  |
|  InvalidParameter  |  El nombre del contenedor {0} no es válido. Los nombres de contenedor deben tener una longitud de entre 3 y 63 caracteres, y pueden contener solo caracteres alfanuméricos en minúscula y el signo de guion. Antes y después del guion debe ir un carácter alfanumérico.  |
|  InvalidParameter  |  El nombre del contenedor {0} en la dirección URL {1} no es válido. Los nombres de contenedor deben tener una longitud de entre 3 y 63 caracteres, y pueden contener solo caracteres alfanuméricos en minúscula y el signo de guion. Antes y después del guion debe ir un carácter alfanumérico.  |
|  InvalidParameter  |  El nombre del blob en la dirección URL {0} contiene una barra diagonal. Actualmente esto es algo que no se admite en discos.  |
|  InvalidParameter  |  El identificador URI {0} no parece ser un URI de blob correcto.  |
|  InvalidParameter  |  Un disco llamado "{0}" ya usa el mismo LUN: {1}.  |
|  InvalidParameter  |  Ya existe un disco llamado "{0}".  |
|  InvalidParameter  |  No se pueden especificar reemplazos de imagen de usuario para un disco ya definido en la referencia de imagen especificada.  |
|  InvalidParameter  |  Un disco llamado "{0}" ya usa la misma dirección URL de VHD {1}.  |
|  InvalidParameter  |  El número de dominios de error especificado {0} debe estar en el intervalo de {1} a {2}.  |
|  InvalidParameter  |  El tipo de licencia {0} no es válido. Tipos de licencia válidos son: Windows_Client o Windows_Server, distinguen mayúsculas de minúsculas.  |
|  InvalidParameter  |  La longitud del nombre de host de Linux no puede superar los {0} caracteres ni contener los siguientes caracteres: {1}.  |
|  InvalidParameter  |  La ruta de destino de las claves públicas SSH está actualmente limitada a su valor predeterminado, {0}, debido a un problema conocido del agente de aprovisionamiento de Linux.  |
|  InvalidParameter  |  Ya existe un disco en LUN {0}.  |
|  InvalidParameter  |  La suscripción {0} de la solicitud debe coincidir con la suscripción {1} que se encuentra en el identificador del disco administrado.  |
|  InvalidParameter  |  Los datos personalizados de OSProfile deben estar codificados en Base64 y su longitud máxima debe ser de {0} caracteres.  |
|  InvalidParameter  |  El nombre del blob en la dirección URL {0} debe terminar con la extensión "{1}".  |
|  InvalidParameter  |  "{0}" no es un prefijo de nombre de blob VHD capturado válido. Los prefijos válidos coincide con regex "{1}".  |
|  InvalidParameter  |  No se pueden agregar certificados a una máquina virtual si no se ha aprovisionado el agente de máquina virtual.  |
|  InvalidParameter  |  Ya existe un disco en LUN {0}.  |
|  InvalidParameter  |  La máquina virtual no se puede crear porque el tamaño solicitado, {0}, no está disponible en el clúster en el que el conjunto de disponibilidad está asignado actualmente. Los tamaños disponibles son: {1}. Se puede encontrar más información sobre la estrategia de cambio de tamaño de máquinas virtuales en https://aka.ms/azure-resizevm.  |
|  InvalidParameter  |  El tamaño de máquina virtual solicitado, {0}, no está disponible en la región actual. Los tamaños disponibles en la región actual son: {1}. Puede encontrar más información sobre los tamaños de máquina virtual disponibles en cada región en https://aka.ms/azure-regions.  |
|  InvalidParameter  |  El tamaño de máquina virtual solicitado, {0}, no está disponible en la región actual. Puede encontrar más información sobre los tamaños de máquina virtual disponibles en cada región en https://aka.ms/azure-regions.  |
|  InvalidParameter  |  El nombre de usuario administrador de Windows no puede tener una longitud de más de {0} caracteres, terminar en un punto (.) ni contener los caracteres siguientes: {1}.  |
|  InvalidParameter  |  El nombre del equipo Windows no puede tener una longitud de más de {0} caracteres, ser completamente numérico ni contener los caracteres siguientes: {1}.  |
|  MissingMoveDependentResources  |  La solicitud de movimiento de recursos no contiene todos los recursos dependientes. Consulte los detalles del error para conocer los identificadores de recursos que faltan.  |
|  MoveResourcesHaveInvalidState  |  La solicitud de movimiento de recursos contiene máquinas virtuales asociadas a cuentas de almacenamiento no válidas. Consulte la información relativa a estos identificadores de recursos y nombres de cuenta de almacenamiento de referencia.  |
|  MoveResourcesHavePendingOperations  |  La solicitud de movimiento de recursos contiene recursos en los que hay alguna operación pendiente. Consulte la información relativa a estos identificadores de recursos. Vuelva a intentar la operación cuando se hayan completado las operaciones pendientes.  |
|  MoveResourcesNotFound  |  La solicitud de movimiento de recursos contiene recursos que no se encuentran. Consulte la información relativa a estos identificadores de recursos.  |
|  NetworkingInternalOperationError  |  Error de asignación de red desconocido.  |
|  NetworkingInternalOperationError  |  Error de asignación de red desconocido  |
|  NetworkingInternalOperationError  |  Hubo un error interno al procesar el perfil de red de la máquina virtual.  |
|  NotFound  |  No se puede encontrar el conjunto de disponibilidad {0}.  |
|  NotFound  |  La máquina virtual de origen "{0}" especificada en la solicitud no existe en esta ubicación de Azure.  |
|  NotFound  |  No se encuentra el inquilino con el identificador {0}.  |
|  NotFound  |  No se puede encontrar la imagen {0}.  |
|  NotSupported  |  El tipo de licencia es {0}, pero el blob de imagen {1} no es local.  |
|  OperationNotAllowed  |  El conjunto de disponibilidad {0} no se puede eliminar. Antes de eliminar un conjunto de disponibilidad asegúrese de que no contiene ninguna máquina virtual.  |
|  OperationNotAllowed  |  No se permite cambiar la SKU del conjunto de disponibilidad de "Alineada" a "Clásica".  |
|  OperationNotAllowed  |  No se pueden modificar las extensiones de la máquina virtual si esta no se está ejecutando.  |
|  OperationNotAllowed  |  La acción de captura solo se admite en máquinas virtuales con discos basados en blobs. Use las API del recurso "Image" para crear una imagen de una máquina virtual administrada.  |
|  OperationNotAllowed  |  El recurso {0} no se puede crear desde la imagen {1} hasta que la imagen se haya creado correctamente.  |
|  OperationNotAllowed  |  No se permiten actualizaciones de encryptionSettings una vez que se ha asignado la máquina virtual. Vuelva a intentarlo cuando se desasigne.  |
|  OperationNotAllowed  |  No se admite la adición de un disco administrado a una máquina virtual con discos basados en blobs.  |
|  OperationNotAllowed  |  El número máximo de discos de datos que se pueden asociar a una máquina virtual de este tamaño es {0}.  |
|  OperationNotAllowed  |  No se admite la adición de un disco basado en blobs a una máquina virtual con discos administrados.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la imagen "{1}", ya que la imagen se ha marcado para eliminación. Solo puede volver a intentar realizar la operación de eliminación (o esperar hasta que se complete una en curso).  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que la máquina virtual es generalizada.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite porque la colección de puntos de restauración "{1}" está marcada para eliminación.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la extensión de máquina virtual "{1}", ya que está marcada para eliminación. Solo puede volver a intentar realizar la operación de eliminación (o esperar hasta que se complete una en curso).  |
|  OperationNotAllowed  |  La operación "{0}" no se permite porque las máquinas virtuales "{1}" se aprovisionan con la imagen "{2}".  |
|  OperationNotAllowed  |  La operación "{0}" no se permite porque el conjunto de escalado de máquinas virtuales "{1}" usa la imagen "{2}".  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que la máquina virtual está marcada para eliminación. Solo puede volver a intentar realizar la operación de eliminación (o esperar hasta que se complete una en curso).  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que la máquina virtual está desasignada o marcada para desasignarse.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que la máquina virtual está en ejecución. Desconéctela explícitamente por si apaga la máquina virtual desde el sistema operativo invitado.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que la máquina virtual no está desasignada.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que el estado de la extensión "{2}" es erróneo.  |
|  OperationNotAllowed  |  La operación "{0}" no se permite en la máquina virtual "{1}", ya que hay otra operación en curso.  |
|  OperationNotAllowed  |  La operación "{0}" necesita que la máquina virtual "{1}" sea generalizada.  |
|  OperationNotAllowed  |  La operación requiere que la máquina virtual se ejecute (o que esté establecida en el modo de ejecución).  |
|  OperationNotAllowed  |  El disco con tamaño de {0} GB, inferior al tamaño de {1} GB del disco correspondiente en la imagen, no se admite.  |
|  OperationNotAllowed  |  Las extensiones del conjunto de escalado de máquinas virtuales del controlador "{0}" solo se pueden agregar en el momento de la creación del conjunto de escalado de máquinas virtuales.  |
|  OperationNotAllowed  |  Las extensiones del conjunto de escalado de máquinas virtuales del controlador "{0}" solo se pueden eliminar en el momento de la eliminación del conjunto de escalado de máquinas virtuales.  |
|  OperationNotAllowed  |  La máquina virtual "{0}" ya usa discos administrados.  |
|  OperationNotAllowed  |  La máquina virtual "{0}" pertenece al conjunto de disponibilidad "Clásico" "{1}". Actualice el conjunto de disponibilidad para que use la SKU "Alineada" y, después, vuelva a intentar la conversión.  |
|  OperationNotAllowed  |  La máquina virtual creada a partir de la imagen no puede tener discos basados en blobs. Todos deben ser discos administrados.  |
|  OperationNotAllowed  |  La operación de captura no se puede completar porque la máquina virtual no es generalizada.  |
|  OperationNotAllowed  |  No se permiten las operaciones de administración en la máquina virtual "{0}", ya que los discos de la máquina virtual se convierten en discos administrados.  |
|  OperationNotAllowed  |  Una operación en curso cambia el estado de alimentación de la máquina virtual {0} a {1}. Realice la operación {2} transcurrido algún tiempo.  |
|  OperationNotAllowed  |  La máquina virtual no se puede agregar o actualizar. Es posible que el tamaño de máquina virtual solicitado {0} no esté disponibles en la unidad de asignación existente. Se puede encontrar más información sobre la estrategia de cambio de tamaño de máquinas virtuales en https://aka.ms/azure-resizevm.  |
|  OperationNotAllowed  |  No se puede ajustar el tamaño de la máquina virtual porque el tamaño solicitado, {0}, no está disponible en el clúster en el que el conjunto de disponibilidad está asignado actualmente. Los tamaños disponibles son: {1}. Se puede encontrar más información sobre la estrategia de cambio de tamaño de máquinas virtuales en https://aka.ms/azure-resizevm.  |
|  OperationNotAllowed  |  No se puede ajustar el tamaño de la máquina virtual porque el tamaño solicitado, {0}, no está disponible en el clúster en el que la máquina virtual está asignada actualmente. Para cambiar el tamaño de la máquina virtual a {1}, desasígnela (esta es la operación de detención de Azure Portal) y vuelva a intentar la operación de cambio de tamaño. Se puede encontrar más información sobre la estrategia de cambio de tamaño de máquinas virtuales en https://aka.ms/azure-resizevm.  |
|  OSProvisioningClientError  |  No se pudo aprovisionar el sistema operativo para la máquina virtual "{0}" porque el sistema operativo invitado se está aprovisionando actualmente.  |
|  OSProvisioningClientError  |  Error de aprovisionamiento del sistema operativo para la máquina virtual "{0}". Detalles del error: {1} asegúrese de que la imagen se haya preparado (generalizado) correctamente. <ul><li>Instrucciones para Windows: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/  </li></ul> |
|  OSProvisioningClientError  |  Error de generación de claves del host SSH. Detalles del error: {0}. Para resolver este problema, compruebe si el agente de Linux está configurado correctamente. <ul><li>Puede comprobar las instrucciones que aparecen en: https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux/ </li></ul> |
|  OSProvisioningClientError  |  El nombre de usuario especificado para la máquina virtual no es válido para esta distribución de Linux. Detalles del error: {0}.  |
|  OSProvisioningInternalError  |  Error de aprovisionamiento del sistema operativo para la máquina virtual "{0}" debido a un error interno.  |
|  OSProvisioningTimedOut  |  El aprovisionamiento del sistema operativo para la máquina virtual "{0}" no finalizó en el tiempo asignado. La máquina virtual aún puede finalizar el aprovisionamiento satisfactoriamente. Compruebe el estado de aprovisionamiento posteriormente.  |
|  OSProvisioningTimedOut  |  El aprovisionamiento del sistema operativo para la máquina virtual "{0}" no finalizó en el tiempo asignado. La máquina virtual aún puede finalizar el aprovisionamiento satisfactoriamente. Compruebe el estado de aprovisionamiento posteriormente. Además, asegúrese de que la imagen se ha preparado (generalizado) correctamente.   <ul><li>Instrucciones para [Windows]( ../windows/upload-generalized-managed.md).</li><li> Instrucciones para [Linux](../linux/capture-image.md).</li></ul>  |
|  OSProvisioningTimedOut  |  El aprovisionamiento del sistema operativo para la máquina virtual "{0}" no finalizó en el tiempo asignado. Sin embargo, se detectó la ejecución del agente invitado de la máquina virtual. Esto sugiere que el sistema operativo invitado no se ha preparado adecuadamente para usarse como una imagen de máquina virtual (con CreateOption = FromImage). Para resolver este problema, use el VHD tal cual con CreateOption = Attach o prepárelo correctamente para usarlo como imagen:   <ul><li>Instrucciones para Windows: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/ </li><li> Instrucciones para Linux: https://azure.microsoft.com/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OverConstrainedAllocationRequest  |  La ubicación seleccionada no dispone actualmente del tamaño de máquina virtual requerido.  |
|  ResourceUpdateBlockedOnPlatformUpdate  |  No se pueden actualizar los recursos en este momento porque hay una actualización de la plataforma en curso. Inténtelo de nuevo más tarde.  |
|  StorageAccountLimitation  |  La cuenta de almacenamiento "{0}" no es compatible con los blobs en páginas necesarios para crear discos.  |
|  StorageAccountLimitation  |  La cuenta de almacenamiento "{0}" superó su cuota asignada.  |
|  StorageAccountLocationMismatch  |  No se pudo resolver la cuenta de almacenamiento {0}. Asegúrese de que se creó mediante el proveedor de recursos de almacenamiento en la misma ubicación que el recurso de proceso.  |
|  StorageAccountNotFound  |  No se encontró la cuenta de almacenamiento {0}. Asegúrese de que no se eliminó y de que pertenece a la misma ubicación de Azure que la máquina virtual.  |
|  StorageAccountNotRecognized  |  Use una cuenta de almacenamiento administrada por el proveedor de recursos de almacenamiento. No se admite el uso de {0}.  |
|  StorageAccountOperationInternalError  |  Error interno al acceder a la cuenta de almacenamiento {0}.  |
|  StorageAccountSubscriptionMismatch  |  La cuenta de almacenamiento {0} no pertenece a la suscripción {1}.  |
|  StorageAccountTooBusy  |  La cuenta de almacenamiento "{0}" está demasiado ocupada actualmente. Considere la posibilidad de usar otra cuenta.  |
|  StorageAccountTypeNotSupported  |  El disco {0} usa {1}, que es una cuenta de almacenamiento de blobs. Reinténtelo con una cuenta de Storage con fines generales.  |
|  StorageAccountTypeNotSupported  |  La cuenta de almacenamiento {0} es de tipo {1}. Diagnósticos de arranque admite {2} tipos de cuenta de almacenamiento.  <ul><li>Este error se produce si usa la cuenta de almacenamiento premium para Diagnósticos de arranque. Para más información, consulte [Uso de Diagnósticos de arranque](boot-diagnostics.md). </li></ul> |
|  SubscriptionNotAuthorizedForImage  |  La suscripción no está autorizada.  |
|  TargetDiskBlobAlreadyExists  |  El blob {0} ya existe. Proporcione un identificador URI de blob distinto para crear un nuevo disco de datos en blanco "{1}".  |
|  TargetDiskBlobAlreadyExists  |  La operación de captura no puede continuar porque el blob de imagen de destino {0} ya existe y la marca para sobrescribir blobs VHD no está establecida. Elimine el blob o establezca la marca para sobrescribir blobs VHD y vuelva a intentarlo.  |
|  TargetDiskBlobAlreadyExists  |  La operación de captura no puede continuar porque el blob de imagen de destino {0} tiene una concesión activa.   |
|  TargetDiskBlobAlreadyExists  |  El blob {0} ya existe. Especifique un identificador URI de blob distinto como destino del disco "{1}".  |
|  TooManyVMRedeploymentRequests  |  Se recibieron demasiadas solicitudes de reimplementación para la máquina virtual "{0}" o para las máquinas virtuales del mismo conjunto de disponibilidad que esta máquina virtual. Inténtelo de nuevo más tarde.  |
|  VHDSizeInvalid  |  El valor de tamaño de disco especificado de {0} para el disco "{1}" con blob {2} no es válido. El tamaño de disco debe estar entre {3} y {4}.  |
|  VMAgentStatusCommunicationError  |  La máquina virtual "{0}" no ha indicado el estado del agente o de las extensiones de máquina virtual. Compruebe que la máquina virtual tiene un agente de máquina virtual en ejecución y que puede establecer conexiones salientes con el almacenamiento de Azure.  |
|  VMArtifactRepositoryInternalError  |  Error al comunicarse con el repositorio de artefactos para recuperar los detalles del artefacto de máquina virtual.  |
|  VMArtifactRepositoryInternalError  |  Se produjo un error interno al recuperar los datos del artefacto de la máquina virtual desde el repositorio de artefactos.  |
|  VMExtensionHandlerNonTransientError  |  El controlador "{0}" informó de un error en la extensión de máquina virtual "{1}" con el código de error terminal "{2}" y el mensaje de error: "{3}".  |
|  VMExtensionManagementInternalError  |  Error interno al procesar la extensión de máquina virtual "{0}".  |
|  VMExtensionManagementInternalError  |  Se produjeron varios errores durante la preparación de las extensiones de máquina virtual. Para más información, consulte la vista de instancia de extensión de máquina virtual.  |
|  VMExtensionProvisioningError  |  La máquina virtual indicó un error al procesar la extensión "{0}". Mensajes de error: "{1}".  |
|  VMExtensionProvisioningError  |  Varias extensiones de máquina virtual no se pudieron aprovisionar en la máquina virtual. Para más información, consulte la vista de instancia de la extensión de máquina virtual.  |
|  VMExtensionProvisioningTimeout  |  Se ha agotado el tiempo de espera del aprovisionamiento de la extensión de máquina virtual "{0}". Puede que la instalación de la extensión esté tardando demasiado o que no se haya podido obtener el estado de la extensión.  |
|  VMMarketplaceInvalidInput  |  La creación de una máquina virtual desde una imagen que no es de Marketplace no necesita información del plan; elimine esta información de la solicitud. El nombre del disco del sistema operativo es {0}.  |
|  VMMarketplaceInvalidInput  |  La información de compra no coincide. No se puede implementar desde la imagen de Marketplace. El nombre del disco del sistema operativo es {0}.  |
|  VMMarketplaceInvalidInput  |  La creación de una máquina virtual desde una imagen de Marketplace requiere que la solicitud incluya información del plan. El nombre del disco del sistema operativo es {0}.  |
|  VMNotFound  |  No se encuentra la máquina virtual "{0}".  |
|  VMRedeploymentFailed  |  La máquina virtual "{0}" no se pudo volver a implementar debido a un error interno. Inténtelo de nuevo más tarde.  |
|  VMRedeploymentTimedOut  |  La reimplementación de la máquina virtual "{0}" no finalizó en el tiempo asignado. Puede terminar correctamente en algún momento. De lo contrario, puede volver a intentar realizar la solicitud.  |
|  VMStartTimedOut  |  La máquina virtual "{0}" no se inició en el tiempo asignado. La máquina virtual puede iniciarse correctamente. Compruebe el estado de energía más adelante.  |


## <a name="next-steps"></a>Pasos siguientes
Si necesita más ayuda, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.