---
title: 'ML Studio (clásico): Códigos de error de API REST: Azure'
description: Estos códigos de error podrían ser devueltos por una operación en un servicio web Azure Machine Learning.
keywords: ''
services: machine-learning
author: likebupt
ms.author: keli19
editor: cgronlun
ms.assetid: 0923074b-3728-439d-a1b8-8a7245e39be4
ms.service: machine-learning
ms.subservice: studio
ms.topic: reference
ms.date: 11/16/2016
ms.openlocfilehash: 461249fc5482f8851c7184b45fe1146c21f2da83
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432077"
---
# <a name="azure-machine-learning-studio-classic-rest-api-error-codes"></a>Códigos de error de API REST de Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Una operación en un servicio web de Azure Machine Learning Studio (clásico) podría devolver los siguientes códigos de error.
 
## <a name="badargument-http-status-code-400"></a>BadArgument (código de estado HTTP 400)
 
Argumento no válido proporcionado.
 
Esta clase de errores significa que un argumento proporcionado en alguna parte no era válido. Podría tratarse de una credencial o ubicación de Azure Storage para algo pasado al servicio web. Examine el campo de código del error en la sección de detalles para diagnosticar qué argumento específico no era válido.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| BadParameterValue | El valor del parámetro proporcionado no satisface la regla de parámetro en el parámetro. |
| BadSubscriptionId | El identificador de suscripción que se usó para puntuar no es el que está presente en el recurso. |
| BadVersionCall | Se pasó un parámetro de versión no válida durante la llamada a la API: {0}. Consulte la página de ayuda de la API para pasar la versión correcta e inténtelo de nuevo. |
| BatchJobInputsNotSpecified | Las siguientes entradas necesarias no se especificaron con la solicitud: {0}. Asegúrese de que se especifican todos los datos de entrada e inténtelo de nuevo. |
| BatchJobInputsTooManySpecified | La solicitud especificó más entradas que las definidas en el servicio. Lista de entradas aceptadas: {0}. Asegúrese de que todos los datos de entrada se han especificado correctamente e inténtelo de nuevo. |
| BlobNameTooLong | La ruta de acceso de Azure Blob Storage proporcionada para la salida de diagnósticos es demasiado larga: {0}. Acorte la ruta de acceso e inténtelo de nuevo. |
| BlobNotFound | No se puede acceder al blob de Azure proporcionado: {0}.  Mensaje de error de Azure: {1}. |
| ContainerIsEmpty | No se proporcionó ningún nombre de contenedor de Azure Storage. Proporcione un nombre de contenedor válido e inténtelo de nuevo. |
| ContainerSegmentInvalid | Nombre de contenedor no válido. Proporcione un nombre de contenedor válido e inténtelo de nuevo. |
| ContainerValidationFailed | Error de validación del contenedor de blobs: {0}. |
| DataTypeNotSupported | Tipo de datos no admitido. Proporcione tipos de datos válidos e inténtelo de nuevo. |
| DuplicateInputInBatchCall | La solicitud por lotes no es válida. No se puede especificar una entrada única y múltiple al mismo tiempo. Quite uno de estos elementos de la solicitud e inténtelo de nuevo. |
| ExpiryTimeInThePast | La hora de expiración proporcionada pertenece al pasado: {0}. Proporcione una hora de expiración futura en UTC e inténtelo de nuevo. Para que no expire nunca, establezca la hora de expiración en NULL. |
| IncompleteSettings | Configuración de diagnósticos incompleta. |
| InputBlobRelativeLocationInvalid | No se proporcionó ningún nombre de Azure Storage Blob. Proporcione un nombre de blob válido e inténtelo de nuevo. |
| InvalidBlob | Especificación de blob no válido para blob: {0}. Compruebe que la cadena de conexión o ruta de acceso relativa, o la especificación de token de SAS sean correctas e inténtelo de nuevo. |
| InvalidBlobConnectionString | La cadena de conexión especificada para uno de los blobs de entrada y salida no es válida: {0}. Corrija este problema e inténtelo de nuevo. |
| InvalidBlobExtension | La referencia de blob {0} tiene una extensión de archivo no válida o no está presente. Las extensiones de archivo admitidas para este tipo de salida son: "{1}". |
| InvalidInputNames | Nombres de entrada de servicio no válidos especificados en la solicitud: {0}. Asigne los datos de entrada a las entradas de servicio correctas e inténtelo de nuevo. |
| InvalidOutputOverrideName | Nombre de invalidación de salida no válida: {0}. El servicio no tiene un nodo de salida con este nombre. Pase un nombre de nodo de salida correcto para invalidar (se aplica distinción entre mayúsculas y minúsculas). |
| InvalidQueryParameter | Parámetro de consulta no válido '{0}'. {1} |
| MissingInputBlobInformation | Falta información de Azure Storage Blob. Proporcione una cadena de conexión y una ruta de acceso relativa o URI válidos e inténtelo de nuevo. |
| MissingJobId | No se proporcionó un identificador de trabajo. Se devolvió un identificador de trabajo cuando un trabajo se envió por primera vez. Compruebe que el identificador de trabajo es correcto e inténtelo de nuevo. |
| MissingKeys | No hay claves proporcionadas o no se proporcionó una clave primaria o secundaria. |
| MissingModelPackage | No se proporcionó un identificador de paquete de modelo o un paquete de modelo. Proporcione un identificador de paquete de modelo o un paquete de modelo válido e inténtelo de nuevo. |
| MissingOutputOverrideSpecification | Falta la especificación de blob para la invalidación de salida {0} en la solicitud. Especifique una ubicación de blob válida con la solicitud o quite la especificación de salida si no se desea ninguna invalidación de ubicación. |
| MissingRequestInput | El servicio web espera una entrada, pero no se proporcionó ninguna entrada. Asegúrese de que se proporcionan entradas válidas según los puertos de entrada publicados en el modelo e inténtelo de nuevo. |
| MissingRequiredGlobalParameters | No se proporcionaron todos los parámetros de servicio web necesarios. Compruebe que los parámetros que se esperan para los módulos son correctos e inténtelo de nuevo. |
| MissingRequiredOutputOverrides | Cuando se llama a un punto de conexión de servicio cifrado es obligatorio pasar anulaciones de salida para todas las salidas del servicio. Faltan invalidaciones en este momento para estas salidas: {0} |
| MissingWebServiceGroupId | No se proporcionó un identificador de grupo de servicio web. Proporcione un identificador de grupo de servicio web válido e inténtelo de nuevo. |
| MissingWebServiceId | No se proporcionó un identificador de servicio web. Proporcione un identificador de servicio web válido e inténtelo de nuevo. |
| MissingWebServicePackage | No se proporcionó ningún paquete de servicio web. Proporcione un paquete de servicio web válido e inténtelo de nuevo. |
| MissingWorkspaceId | No se proporcionó ningún identificador de área de trabajo. Proporcione un identificador de área de trabajo válido e inténtelo de nuevo. |
| ModelConfigurationInvalid | Configuración de modelo no válido en el paquete de modelo. Asegúrese de que la configuración del modelo contiene la definición de puntos de conexión de salida, el punto de conexión de error estándar, el punto de conexión de salida estándar e inténtelo de nuevo. |
| ModelPackageIdInvalid | Identificador de paquete de modelo no válido. Compruebe que el identificador del paquete de modelo es correcto e inténtelo de nuevo. |
| RequestBodyInvalid | No hay cuerpo de solicitud proporcionado o error al deserializar el cuerpo de solicitud. |
| RequestIsEmpty | No se proporciona ninguna solicitud. Proporcione una solicitud válida e inténtelo de nuevo. |
| UnexpectedParameter | Parámetros inesperados proporcionados. Compruebe que todos los nombres de parámetro estén escritos correctamente, que solo e pasan los parámetros previstos e inténtelo de nuevo. |
| UnknownError | Error desconocido. |
| UserParameterInvalid | {0} |
| WebServiceConcurrentRequestRequirementInvalid | No se pueden cambiar los requisitos de solicitudes simultáneas para el servicio web {0}. |
| WebServiceIdInvalid | Se proporcionó un identificador de servicio web no válido. El identificador del servicio web debe ser un guid válido. |
| WebServiceTooManyConcurrentRequestRequirement | No se puede establecer el requisito de solicitudes simultáneas en más de {0}. |
| WebServiceTypeInvalid | Se proporcionó un tipo de servicio web no válido. Compruebe que el tipo de servicio web válido es correcto e inténtelo de nuevo. Tipos de servicio web válidos: {0}. |
 
## <a name="baduserargument-http-status-code-400"></a>BadUserArgument (código de estado HTTP 400)
 
Argumento de usuario no válido proporcionado.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| InputMismatchError | Los datos de entrada no coinciden con el esquema de puerto de entrada. |
| InputParseError | Error de análisis del vector de entrada.  Compruebe que el vector de entrada tiene el número correcto de columnas y tipos de datos.  Detalles adicionales: {0}. |
| MissingRequiredGlobalParameters | Faltan parámetros esperados por el servicio web. Compruebe que todos los parámetros necesarios esperados por el servicio web son correctos e inténtelo de nuevo. |
| UnexpectedParameter | Compruebe solamente que se pasan los parámetros necesarios esperados por el servicio web e inténtelo de nuevo. |
| UserParameterInvalid | {0} |
 
## <a name="invalidoperation-http-status-code-400"></a>InvalidOperation (código de estado HTTP 400)
 
La solicitud no es válida en el contexto actual.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| CannotStartJob | No se puede iniciar el trabajo porque está en el estado {0}. |
| IncompatibleModel | El modelo no es compatible con la versión de la solicitud. La versión de la solicitud solo admite modelos de salida de tabla de datos únicos. |
| MultipleInputsNotAllowed | El modelo no permite varias entradas. |
 
## <a name="libraryexecutionerror-http-status-code-400"></a>LibraryExecutionError (código de estado HTTP 400)
 
La ejecución del módulo encontró un error interno de biblioteca.
 
 
## <a name="moduleexecutionerror-http-status-code-400"></a>ModuleExecutionError (código de estado HTTP 400)
 
Error de ejecución del módulo.
 
 
## <a name="webservicepackageerror-http-status-code-400"></a>WebServicePackageError (código de estado HTTP 400)
 
Paquete de servicio web no válido. Compruebe que el paquete de servicio web proporcionado es correcto e inténtelo de nuevo.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| FormatError | El paquete del servicio web tiene un formato incorrecto. Detalles: {0} |
| RuntimesError | El gráfico del paquete del servicio web no es válido. Detalles: {0} |
| ValidationError | El gráfico del paquete del servicio web no es válido. Detalles: {0} |
 
## <a name="unauthorized-http-status-code-401"></a>No autorizado (código de estado HTTP 401)
 
La solicitud no está autorizada para acceder al recurso.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| AdminRequestUnauthorized | No autorizado |
| ManagementRequestUnauthorized | No autorizado |
| ScoreRequestUnauthorized | Credenciales no válidas proporcionadas. |
 
## <a name="notfound-http-status-code-404"></a>NotFound (código de estado HTTP 404)
 
Recurso no encontrado.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| ModelPackageNotFound | Paquete de modelo no encontrado. Compruebe que el identificador del paquete de modelo es correcto e inténtelo de nuevo. |
| WebServiceIdNotFoundInWorkspace | No se encuentra el servicio web en esta área de trabajo. Hay una discordancia entre webServiceId y workspaceId. Compruebe que el servicio web proporcionado forma parte del área de trabajo e inténtelo de nuevo. |
| WebServiceNotFound | Servicio web no encontrado. Compruebe que el identificador del servicio web es correcto e inténtelo de nuevo. |
| WorkspaceNotFound | Espacio de trabajo no encontrado. Compruebe que el identificador del área de trabajo es correcto e inténtelo de nuevo. |
 
## <a name="requesttimeout-http-status-code-408"></a>RequestTimeout (código de estado HTTP 408)
 
La operación no se pudo completar en el tiempo permitido.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| RequestCanceled | El cliente canceló la solicitud. |
| ScoreRequestTimeout | Tiempo de espera agotado para esta solicitud de ejecución. |
 
## <a name="conflict-http-status-code-409"></a>Conflict (código de estado HTTP 409)
 
El recurso ya existe.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| ModelOutputMetadataMismatch | Nombre del parámetro de salida no válido. Intente utilizar el módulo de editor de metadatos para cambiar el nombre de las columnas e inténtelo de nuevo. |
 
## <a name="memoryquotaviolation-http-status-code-413"></a>MemoryQuotaViolation (código de estado HTTP 413)
 
El modelo ha superado la cuota de memoria que tenía asignada.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| OutOfMemoryLimit | El modelo consumió más memoria de la que era apropiada para él. La memoria máxima permitida para el modelo es {0} MB. Compruebe el modelo por si tiene problemas. |
 
## <a name="internalerror-http-status-code-500"></a>InternalError (código de estado HTTP 500)
 
Error interno en la ejecución.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| AdminAuthenticationFailed |  |
| BackendArgumentError |  |
| BackendBadRequest |  |
| ClusterConfigBlobMisconfigured |  |
| ContainerProcessTerminatedWithSystemError | El proceso del contenedor se bloqueó con error del sistema. |
| ContainerProcessTerminatedWithUnknownError | El proceso del contenedor se bloqueó con error desconocido. |
| ContainerValidationFailed | Error de validación del contenedor de blobs: {0}. |
| DeleteWebServiceResourceFailed |  |
| ExceptionDeserializationError |  |
| FailedGettingApiDocument |  |
| FailedStoringWebService |  |
| InvalidMemoryConfiguration | InvalidMemoryConfiguration, ConfigValue: {0} |
| InvalidResourceCacheConfiguration |  |
| InvalidResourceDownloadConfiguration |  |
| InvalidWebServiceResources |  |
| MissingTaskInstance | No se proporcionaron argumentos. Compruebe que se pasan argumentos válidos e inténtelo de nuevo. |
| ModelPackageInvalid |  |
| ModuleExecutionFailed |  |
| ModuleLoadFailed |  |
| ModuleObjectCloneFailed |  |
| OutputConversionFailed |  |
| PortDataTypeNotSupported | El identificador de puerto {0} tiene un tipo de datos no admitido: {1}. |
| ResourceDownload |  |
| ResourceLoadFailed |  |
| ServiceUrisNotFound |  |
| SwaggerGeneration | Error en la generación de Swagger. Detalles: {0} |
| UnexpectedScoreStatus |  |
| UnknownBackendErrorResponse |  |
| UnknownError |  |
| UnknownJobStatusCode | Código de estado de trabajo desconocido {0}. |
| UnknownModuleError |  |
| UpdateWebServiceResourceFailed |  |
| WebServiceGroupNotFound |  |
| WebServicePackageInvalid | InvalidWebServicePackage. Detalles: {0} |
| WorkerAuthorizationFailed |  |
| WorkerUnreachable |  |
 
## <a name="internalerrorsystemlowonmemory-http-status-code-500"></a>InternalErrorSystemLowOnMemory (código de estado HTTP 500)
 
Error interno en la ejecución. Sistema con poca memoria. Inténtelo de nuevo.
 
 
## <a name="modelpackageformaterror-http-status-code-500"></a>ModelPackageFormatError (código de estado HTTP 500)
 
Paquete de modelo no válido. Compruebe que el paquete de modelo proporcionado es correcto e inténtelo de nuevo.
 
 
## <a name="webservicepackageinternalerror-http-status-code-500"></a>WebServicePackageInternalError (código de estado HTTP 500)
 
Paquete de servicio web no válido. Compruebe que el paquete web de modelo proporcionado es correcto e inténtelo de nuevo.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| ModuleError | El gráfico del paquete del servicio web no es válido. Detalles: {0} |
 
## <a name="initializingcontainers-http-status-code-503"></a>InitializingContainers (código de estado HTTP 503)
 
La solicitud no se puede ejecutar cuando los contenedores se están inicializando.
 
 
## <a name="serviceunavailable-http-status-code-503"></a>ServiceUnavailable (código de estado HTTP 503)
 
El servicio no está disponible temporalmente.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| NoMoreResources | No hay recursos disponibles para la solicitud. |
| RequestThrottled | Solicitud limitada para el punto de conexión {0}. La simultaneidad máxima para el punto de conexión es {1}. |
| TooManyConcurrentRequests | Demasiadas solicitudes simultáneas enviadas. |
| TooManyHostsBeingInitialized | Se están inicializando demasiados hosts al mismo tiempo. Considere la posibilidad de limitarlo o intentarlo de nuevo. |
| TooManyHostsBeingInitializedPerModel | Se están inicializando demasiados hosts al mismo tiempo. Considere la posibilidad de limitarlo o intentarlo de nuevo. |
 
## <a name="gatewaytimeout-http-status-code-504"></a>GatewayTimeout (código de estado HTTP 504)
 
La operación no se pudo completar en el tiempo permitido.
 
| Código de error | Mensaje de usuario |
| ---------- |--------------|
| BackendInitializationTimeout | No se puede completar la inicialización del servicio web en el tiempo permitido. |
| BackendScoreTimeout | No se puede completar la ejecución de la solicitud de servicio en el tiempo permitido. |
 
