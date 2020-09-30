---
title: Detección avanzada de ataques de varias fases en Azure Sentinel
description: Use la tecnología de fusión de Azure Sentinel para reducir el exceso de alertas y crear incidentes útiles basados en la detección avanzada de ataques de varias fases.
services: sentinel
documentationcenter: na
author: yelevin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2020
ms.author: yelevin
ms.openlocfilehash: ba872f221f3bde29f0bb48b04dc2259d3ab4938a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906268"
---
# <a name="advanced-multistage-attack-detection-in-azure-sentinel"></a>Detección avanzada de ataques de varias fases en Azure Sentinel


> [!IMPORTANT]
> Algunas características de Fusion de Azure Sentinel están actualmente en **versión preliminar pública**.
> Estas características se ofrecen sin Acuerdo de Nivel de Servicio y no es aconsejable usarlas en cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Mediante el uso de la tecnología Fusion, basada en el aprendizaje automático, Azure Sentinel puede detectar automáticamente ataques de varias fases identificando combinaciones de comportamientos anómalos y de actividades sospechosas que se observan en diversas etapas de la cadena de eliminación. A partir de estas detecciones, Azure Sentinel genera incidentes que, de otro modo, serían muy difíciles de detectar. Estos incidentes incluyen dos o más alertas o actividades. Por diseño, estos incidentes tienen poco volumen, alta fidelidad y alta gravedad.

Cuando se personaliza para adaptarla a su entorno, esta tecnología de detección no solo reduce las tasas de falsos positivos, sino que también puede detectar ataques con información limitada o que falta.

## <a name="configuration-for-advanced-multistage-attack-detection"></a>Configuración de la detección avanzada de ataques de varias fases

Esta detección está habilitada de forma predeterminada en Azure Sentinel. Para comprobar su estado, o bien para deshabilitar esta detección en caso de que use una solución alternativa para crear incidentes en función de varias alertas, siga estas instrucciones:

1. Si aún no lo ha hecho, inicie sesión en el [Azure Portal](https://portal.azure.com).

1. Vaya a **Azure Sentinel** > **Configuración** > **Análisis**.

1. Seleccione **Reglas activas** y busque **Detección avanzada de ataques de varias fases** en la columna **NOMBRE** filtrando la lista por el tipo de regla **Fusion**. Compruebe la columna **ESTADO** para confirmar si esta detección está habilitada o deshabilitada.

    :::image type="content" source="./media/fusion/selecting-fusion-rule-type.png" alt-text="{alt-text}":::

1. Para cambiar el estado, seleccione esta entrada y, en la hoja **Detección avanzada de ataques de varias fases**, seleccione **Editar**.

1. En la hoja **Asistente para creación de reglas**, se selecciona automáticamente el cambio de estado, así que seleccione **Siguiente: revisar** y, a continuación, **Guardar**. 

 Dado que el tipo de regla **Fusion** solo contiene una regla que no se puede modificar, las plantillas de reglas no son aplicables para este tipo de regla.

> [!NOTE]
> Actualmente, Azure Sentinel usa 30 días de datos del historial para entrenar los sistemas de aprendizaje automático. Estos datos siempre se cifran mediante las claves de Microsoft cuando pasan por la canalización de aprendizaje automático. Sin embargo, los datos de entrenamiento no se cifran mediante [claves administradas por el cliente (CMK)](customer-managed-keys.md) si CMK se ha habilitado en el área de trabajo de Azure Sentinel. Para deshabilitar Fusion, vaya a **Azure Sentinel** \> **Configuración** \> **Análisis \> Reglas activas \> Detección avanzada de ataques en varias fases** y, en la columna **Estado**, seleccione **Deshabilitar**.

## <a name="attack-detection-scenarios"></a>Escenarios de detección de ataques

En la siguiente sección se enumeran los tipos de escenarios de correlación, agrupados por clasificación de amenazas, que busca Azure Sentinel mediante la tecnología Fusion.

Como se ha mencionado anteriormente, Fusion correlaciona varias alertas de seguridad de diferentes productos para detectar ataques avanzados en varias fases, por lo que las detecciones de Fusion correctas se presentan como **incidentes de Fusion** en la página **Incidentes** de Azure Sentinel, y no como **alertas** en la tabla **Alertas de seguridad** en **Registros**.

Para habilitar estos escenarios de detección de ataques con tecnología Fusion, los orígenes de datos enumerados se deben ingerir mediante los conectores de datos de Azure Sentinel asociados.

> [!NOTE]
> Algunos de estos escenarios se encuentran en **versión preliminar pública**. Aparecerá indicado.

## <a name="compute-resource-abuse"></a>Uso abusivo de recursos de proceso

### <a name="multiple-vm-creation-activities-following-suspicious-azure-active-directory-sign-in"></a>Varias actividades de creación de máquinas virtuales después de un inicio de sesión sospechoso en Azure Active Directory
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial e impacto 

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y secuestro de recursos (T1496)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha creado un número anómalo de máquinas virtuales en una sola sesión después de un inicio de sesión sospechoso en una cuenta de Azure AD. Este tipo de alerta indica, con un alto grado de confianza, que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para crear nuevas máquinas virtuales con fines no autorizados, como la ejecución de operaciones de minería de datos de cifrado. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de varias actividades de creación de máquinas virtuales son:

- **Un viaje imposible a una ubicación inusual que conduce a varias actividades de creación de máquinas virtuales**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a varias actividades de creación de máquinas virtuales**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a varias actividades de creación de máquinas virtuales**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a varias actividades de creación de máquinas virtuales**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a varias actividades de creación de máquinas virtuales**

## <a name="data-exfiltration"></a>Filtración de datos

### <a name="office-365-mailbox-exfiltration-following-a-suspicious-azure-ad-sign-in"></a>Filtración de buzones de Office 365 después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial, filtración y recopilación

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078), recopilación de correos electrónicos (T1114) y filtración automatizada (T1020)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha establecido una regla de reenvío de bandeja de entrada sospechosa en la bandeja de entrada de un usuario después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta del usuario (indicada en la descripción del incidente de Fusion) se ha puesto en peligro y se ha usado para el filtrado de datos de la red de su organización mediante la habilitación de una regla de reenvío de buzón sin que el usuario real lo sepa. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de filtración de buzones de Office 365 son:

- **Un viaje imposible a una ubicación inusual que conduce a la filtración de los buzones de Office 365**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a la filtración de buzones de Office 365**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a la filtración de buzones de Office 365**

- **Evento de inicio de sesión desde una dirección IP anónima que conduce a la filtración de buzones de Office 365**

- **Evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a la filtración de buzones de Office 365**

### <a name="mass-file-download-following-suspicious-azure-ad-sign-in"></a>Descarga masiva de archivos después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial y filtración

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que un usuario ha descargado un número anómalo de archivos después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para el filtrado de datos de la red de su organización. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de descarga masiva de archivos son:  

- **Un viaje imposible a una ubicación inusual que conduce a una descarga de archivos masiva**

- **Evento de inicio de sesión desde una ubicación desconocida que conduce a una descarga de archivos masiva**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a una descarga de archivos masiva**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a una descarga de archivos masiva**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a una descarga de archivos masiva**

### <a name="mass-file-sharing-following-suspicious-azure-ad-sign-in"></a>Uso compartido de archivos masivo después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial y filtración

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y filtración a través de un servicio web (T1567)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha compartido con otros usuarios un número de archivos por encima de un umbral determinado después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta indicada en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para el filtrado de datos de la red de su organización mediante el uso compartido de archivos (como documentos, hojas de cálculo, etc.) con usuarios no autorizados con fines malintencionados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de uso compartido de archivos masivo son:  

- **Un viaje imposible a una ubicación inusual que conduce a un uso compartido de archivos masivo**

- **Evento de inicio de sesión desde una ubicación desconocida que conduce a un uso compartido de archivos masivo**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a un uso compartido de archivos masivo**

- **Evento de inicio de sesión desde una dirección IP anónima que conduce a un uso compartido de archivos masivo**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a un uso compartido de archivos masivo**

### <a name="suspicious-inbox-manipulation-rules-set-following-suspicious-azure-ad-sign-in"></a>Establecimiento de reglas de manipulación sospechosa de la bandeja de entrada después de un inicio de sesión sospechoso en Azure AD
Este escenario pertenece a dos clasificaciones de amenazas de esta lista: **filtración de datos** y **desplazamiento lateral**. Para mayor claridad, aparece en ambas secciones.

Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial, desplazamiento lateral y filtración

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y phishing interno de objetivo definido (T1534)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se han establecido reglas de bandeja de entrada anómalas en la bandeja de entrada de un usuario después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esto proporciona un alto grado de confianza en que la cuenta indicada en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para manipular las reglas de bandeja de entrada del correo electrónico del usuario con fines malintencionados. Esto podría ser un intento por parte de un atacante de filtrar datos de la red de la organización. Como alternativa, el atacante podría intentar generar correos electrónicos de suplantación de identidad (phishing) desde dentro de la organización (omitiendo los mecanismos de detección de suplantación de identidad en correos electrónicos de orígenes externos) con el fin de desplazarse lateralmente a través de la red obteniendo acceso a cuentas de usuario o a cuentas con privilegios elevados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta del establecimiento de reglas de manipulación sospechosa de la bandeja de entrada son:  

- **Un viaje imposible a una ubicación inusual que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

### <a name="multiple-power-bi-report-sharing-activities-following-suspicious-azure-ad-sign-in"></a>Varias actividades de uso compartido de informes de Power BI después de un inicio de sesión sospechoso en Azure AD 
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial y filtración 

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y filtración a través de un servicio web (T1567)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha compartido un número anómalo de informes de Power BI en una sola sesión después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para el filtrado de datos de la red de su organización mediante el uso compartido de informes de Power BI con usuarios no autorizados con fines malintencionados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de varias actividades de uso compartido de informes de Power BI son:  

- **Un viaje imposible a una ubicación inusual que conduce a varias actividades de uso compartido de informes de Power BI**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a varias actividades de uso compartido de informes de Power BI**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a varias actividades de uso compartido de informes de Power BI**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a varias actividades de uso compartido de informes de Power BI**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a varias actividades de uso compartido de informes de Power BI**

### <a name="suspicious-power-bi-report-sharing-following-suspicious-azure-ad-sign-in"></a>Uso compartido sospechoso de informes de Power BI después de un inicio de sesión sospechoso en Azure AD
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial y filtración 

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y filtración a través de un servicio web (T1567)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha producido una actividad sospechosa de uso compartido de informes de Power BI después de un inicio de sesión sospechoso en una cuenta de Azure AD. La actividad de uso compartido se ha identificado como sospechosa porque los informes de Power BI contenían información confidencial identificada mediante el procesamiento de lenguaje natural, y porque se han compartido con una dirección de correo electrónico externa, se han publicado en la Web o se han enviado como una instantánea a una dirección de correo electrónico suscrita externamente. Esta alerta indica con un alto grado de confianza que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para el filtrado de datos confidenciales de su organización mediante el uso compartido de informes de Power BI con usuarios no autorizados con fines malintencionados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de uso compartido de informes de Power BI son:  

- **Un viaje imposible a una ubicación inusual que conduce a un uso compartido sospechoso de informes de Power BI**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a un uso compartido sospechoso de informes de Power BI**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a un uso compartido sospechoso de informes de Power BI**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a un uso compartido sospechoso de informes de Power BI**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a un uso compartido sospechoso de informes de Power BI**

## <a name="data-destruction"></a>Destrucción de datos

### <a name="mass-file-deletion-following-suspicious-azure-ad-sign-in"></a>Eliminación masiva de archivos después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial e impacto

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y destrucción de datos (T1485)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha eliminado un número anómalo de archivos únicos después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esto proporciona una indicación de que la cuenta indicada en la descripción del incidente de Fusion puede haber estado en peligro y haberse usado para destruir datos con fines malintencionados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de eliminación de archivos masiva son:  

- **Un viaje imposible a una ubicación inusual que conduce a una eliminación de archivos masiva**

- **Evento de inicio de sesión desde una ubicación desconocida que conduce a una eliminación de archivos masiva**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a una eliminación de archivos masiva**

- **Evento de inicio de sesión desde una dirección IP anónima que conduce a una eliminación de archivos masiva**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a una eliminación de archivos masiva**

### <a name="suspicious-email-deletion-activity-following-suspicious-azure-ad-sign-in"></a>Actividad de eliminación de correo electrónico sospechosa después de un inicio de sesión sospechoso en Azure AD
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial e impacto 

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y destrucción de datos (T1485)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha eliminado un número anómalo de correos electrónicos en una sola sesión después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esto proporciona una indicación de que la cuenta indicada en la descripción del incidente de Fusion puede haber estado en peligro y haberse usado para destruir datos con fines malintencionados, como perjudicar a la organización u ocultar actividades de correo electrónico relacionadas con el correo no deseado. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de actividades sospechosas de eliminación de correos electrónicos son:   

- **Un viaje imposible a una ubicación inusual que conduce a actividades sospechosas de eliminación de correos electrónicos**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a actividades sospechosas de eliminación de correos electrónicos**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a actividades sospechosas de eliminación de correos electrónicos**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a actividades sospechosas de eliminación de correos electrónicos**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a actividades sospechosas de eliminación de correos electrónicos**

## <a name="denial-of-service"></a>Denegación de servicio

### <a name="multiple-vm-delete-activities-following-suspicious-azure-ad-sign-in"></a>Varias actividades de eliminación de máquinas virtuales después de un inicio de sesión sospechoso en Azure AD
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial e impacto

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y denegación de servicio del punto de conexión (T1499)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha eliminado un número anómalo de máquinas virtuales en una sola sesión después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para intentar alterar o destruir el entorno de nube de la organización. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de varias actividades de eliminación de máquinas virtuales son:  

- **Un viaje imposible a una ubicación inusual que conduce a varias actividades de eliminación de máquinas virtuales**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a varias actividades de eliminación de máquinas virtuales**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a varias actividades de eliminación de máquinas virtuales**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a varias actividades de eliminación de máquinas virtuales**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a varias actividades de eliminación de máquinas virtuales**

## <a name="lateral-movement"></a>Desplazamiento lateral

### <a name="office-365-impersonation-following-suspicious-azure-ad-sign-in"></a>Suplantación de Office 365 después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial y desplazamiento lateral

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y phishing interno de objetivo definido (T1534)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha producido un número anómalo de acciones de suplantación después de un inicio de sesión sospechoso en una cuenta de Azure AD. Algunos programas de software cuentan con opciones que permiten a los usuarios suplantar a otros usuarios. Por ejemplo, los servicios de correo electrónico permiten que un usuario autorice a otros a enviar correos electrónicos en su nombre. Esta alerta indica con un alto grado de confianza que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para llevar a cabo actividades de suplantación con fines malintencionados, como el envío de correos electrónicos de suplantación de identidad (phishing) para distribuir malware o desplazarse lateralmente. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de suplantación de Office 365 son:  

- **Un viaje imposible a una ubicación inusual que conduce a la suplantación de Office 365**

- **Evento de inicio de sesión desde una ubicación desconocida que conduce a una suplantación de Office 365**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a una suplantación de Office 365**

- **Evento de inicio de sesión desde una dirección IP anónima que conduce a una suplantación de Office 365**

- **Evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a la suplantación de Office 365**
 
### <a name="suspicious-inbox-manipulation-rules-set-following-suspicious-azure-ad-sign-in"></a>Establecimiento de reglas de manipulación sospechosa de la bandeja de entrada después de un inicio de sesión sospechoso en Azure AD
Este escenario pertenece a dos clasificaciones de amenazas de esta lista: **desplazamiento lateral** y **filtración de datos**. Para mayor claridad, aparece en ambas secciones.

Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial, desplazamiento lateral y filtración

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078), phishing interno de objetivo definido (T1534) y filtración automatizada (T1020)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se han establecido reglas de bandeja de entrada anómalas en la bandeja de entrada de un usuario después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta indicada en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para manipular las reglas de bandeja de entrada del correo electrónico del usuario con fines malintencionados. Esto podría ser un intento por parte de un atacante de filtrar datos de la red de la organización. Como alternativa, el atacante podría intentar generar correos electrónicos de suplantación de identidad (phishing) desde dentro de la organización (omitiendo los mecanismos de detección de suplantación de identidad en correos electrónicos de orígenes externos) con el fin de desplazarse lateralmente a través de la red obteniendo acceso a cuentas de usuario o a cuentas con privilegios elevados. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta del establecimiento de reglas de manipulación sospechosa de la bandeja de entrada son:

- **Un viaje imposible a una ubicación inusual que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde una ubicación desconocida que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde un dispositivo infectado que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a una regla de manipulación sospechosa de la bandeja de entrada**

## <a name="malicious-administrative-activity"></a>Actividad administrativa malintencionada

### <a name="suspicious-cloud-app-administrative-activity-following-suspicious-azure-ad-sign-in"></a>Actividad de administración de aplicaciones en la nube sospechosa después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial, persistencia, evasión de defensas, desplazamiento lateral, recopilación, filtración e impacto

**Técnicas de MITRE ATT&CK:** N/D

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha realizado un número anómalo de actividades administrativas en una sola sesión después de un inicio de sesión sospechoso en Azure AD desde la misma cuenta. Esto indica que la cuenta que aparece en la descripción del incidente de Fusion puede haber estado en peligro y haberse usado para realizar cualquier cantidad de acciones administrativas no autorizadas con fines malintencionados. También indica que es posible que se haya puesto en peligro una cuenta con privilegios administrativos. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de actividades sospechosas de administración de aplicaciones en la nube son:  

- **Un viaje imposible a una ubicación inusual que conduce a una actividad sospechosa de administración de aplicaciones en la nube**

- **Evento de inicio de sesión desde una ubicación desconocida que conduce a una actividad de administración de aplicaciones en la nube sospechosa**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a una actividad de administración de aplicaciones en la nube sospechosa**

- **Evento de inicio de sesión desde una dirección IP anónima que conduce a una actividad de administración de aplicaciones en la nube sospechosa**

- **Evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a una actividad de administración de aplicaciones en la nube sospechosa**

## <a name="malicious-execution-with-legitimate-process"></a>Ejecución malintencionada con un proceso legítimo

### <a name="powershell-made-a-suspicious-network-connection-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>PowerShell estableció una conexión de red sospechosa, seguida por tráfico anómalo marcado por el firewall de Palo Alto Networks.
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Ejecución

**Técnicas de MITRE ATT&CK:** Intérprete de scripts y comandos (T1059)

**Orígenes de conexión de datos:** Microsoft Defender para punto de conexión (anteriormente conocido como Protección contra amenazas avanzada de Microsoft Defender o MDATP) y Palo Alto Networks 

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha realizado una solicitud de conexión de salida a través de un comando de PowerShell y, después, el firewall de Palo Alto Networks ha detectado una actividad de entrada anómala. Esto indica que es probable que un atacante haya accedido a la red y esté intentando realizar acciones malintencionadas. Los intentos de conexión de PowerShell que siguen este patrón podrían indicar actividades de control y comando de malware, solicitudes de descarga de malware adicional o que un atacante está estableciendo acceso interactivo remoto. Igual que sucede con todos los ataques que aprovechan recursos ya existentes en el entorno de destino, esta actividad podría ser un uso legítimo de PowerShell. Pero la ejecución de comandos de PowerShell seguida por una actividad de firewall entrante sospechosa aumenta la confianza en que PowerShell se está usando de manera malintencionada y debe investigarse más a fondo. En los registros de Palo Alto, Azure Sentinel se centra en [registros de amenazas](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs) y el tráfico se considera sospechoso cuando se permiten amenazas (datos sospechosos, archivos, inundaciones, paquetes, exámenes, spyware, direcciones URL, virus, puntos vulnerables, virus Wildfire y malware Wildfire). También puede consultar el registro de amenazas de Palo Alto que se corresponde con el [tipo de amenaza/contenido](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) enumerado en la descripción del incidente de Fusion para obtener más información sobre las alertas.

### <a name="suspicious-remote-wmi-execution-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Ejecución de WMI remota sospechosa seguida por tráfico anómalo marcado por el firewall de Palo Alto Networks
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Ejecución y detección

**Técnicas de MITRE ATT&CK:** Instrumental de administración de Windows (T1047)

**Orígenes de conexión de datos:** Microsoft Defender para punto de conexión (anteriormente MDATP) y Palo Alto Networks 

**Descripción:** Los incidentes de Fusion de este tipo indican que los comandos de la interfaz de administración de Windows (WMI) se han ejecutado de forma remota en un sistema y, después, el firewall de Palo Alto Networks ha detectado una actividad de entrada sospechosa. Esto indica que un atacante puede haber conseguido acceder a la red y está intentando desplazarse lateralmente, escalar privilegios o ejecutar cargas malintencionadas. Igual que sucede con todos los ataques que aprovechan recursos ya existentes en el entorno de destino, esta actividad podría ser un uso legítimo de WMI. Pero la ejecución remota de comandos de WMI seguida por una actividad de firewall entrante sospechosa aumenta la confianza en que WMI se está usando de manera malintencionada y debe investigarse más a fondo. En los registros de Palo Alto, Azure Sentinel se centra en [registros de amenazas](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs) y el tráfico se considera sospechoso cuando se permiten amenazas (datos sospechosos, archivos, inundaciones, paquetes, exámenes, spyware, direcciones URL, virus, puntos vulnerables, virus Wildfire y malware Wildfire). También puede consultar el registro de amenazas de Palo Alto que se corresponde con el [tipo de amenaza/contenido](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) enumerado en la descripción del incidente de Fusion para obtener más información sobre las alertas.

## <a name="malware-c2-or-download"></a>Comando y control (C2) o descarga de malware

### <a name="network-request-to-tor-anonymization-service-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Solicitud de red a un servicio de anonimización de TOR, seguida por un tráfico anómalo marcado por el firewall de Palo Alto Networks.
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Comando y control

**Técnicas de MITRE ATT&CK:** Canal cifrado (T1573) y Proxy (T1090)

**Orígenes de conexión de datos:** Microsoft Defender para punto de conexión (anteriormente MDATP) y Palo Alto Networks 

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha realizado una solicitud de conexión de salida al servicio de anonimización de TOR y, después, el firewall de Palo Alto Networks ha detectado una actividad de entrada anómala. Esto indica que es probable que un atacante haya accedido a la red y esté intentando ocultar sus acciones e intenciones. Las conexiones a la red TOR que siguen este patrón podrían indicar actividades de control y comando de malware, solicitudes de descarga de malware adicional o que un atacante está estableciendo acceso interactivo remoto. En los registros de Palo Alto, Azure Sentinel se centra en [registros de amenazas](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs) y el tráfico se considera sospechoso cuando se permiten amenazas (datos sospechosos, archivos, inundaciones, paquetes, exámenes, spyware, direcciones URL, virus, puntos vulnerables, virus Wildfire y malware Wildfire). También puede consultar el registro de amenazas de Palo Alto que se corresponde con el [tipo de amenaza/contenido](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) enumerado en la descripción del incidente de Fusion para obtener más información sobre las alertas.

### <a name="outbound-connection-to-ip-with-a-history-of-unauthorized-access-attempts-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Una conexión saliente a IP con un historial de intentos de acceso no autorizados, seguida por tráfico anómalo marcado por el firewall de Palo Alto Networks
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Comando y control

**Técnicas de MITRE ATT&CK:** No aplicable

**Orígenes de conexión de datos:** Microsoft Defender para punto de conexión (anteriormente MDATP) y Palo Alto Networks 

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha establecido una conexión de salida a una dirección IP con un historial de intentos de acceso no autorizado y, después, el firewall de Palo Alto Networks ha detectado una actividad de entrada anómala. Esto indica que es probable que un atacante haya accedido a la red. Los intentos de conexión que siguen este patrón podrían indicar actividades de control y comando de malware, solicitudes de descarga de malware adicional o que un atacante está estableciendo acceso interactivo remoto. En los registros de Palo Alto, Azure Sentinel se centra en [registros de amenazas](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs) y el tráfico se considera sospechoso cuando se permiten amenazas (datos sospechosos, archivos, inundaciones, paquetes, exámenes, spyware, direcciones URL, virus, puntos vulnerables, virus Wildfire y malware Wildfire). También puede consultar el registro de amenazas de Palo Alto que se corresponde con el [tipo de amenaza/contenido](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) enumerado en la descripción del incidente de Fusion para obtener más información sobre las alertas.

## <a name="ransomware"></a>Ransomware

### <a name="ransomware-execution-following-suspicious-azure-ad-sign-in"></a>Ejecución de ransomware después de un inicio de sesión sospechoso en Azure AD

**Tácticas de MITRE ATT&CK:** Acceso inicial e impacto

**Técnicas de MITRE ATT&CK:** Cuenta válida (T1078) y datos cifrados para causar impacto (T1486)

**Orígenes de conexión de datos:** Microsoft Cloud App Security y Azure Active Directory Identity Protection

**Descripción:** Los incidentes de Fusion de este tipo indican que se ha detectado un comportamiento anómalo de un usuario que apunta a un ataque de ransomware después de un inicio de sesión sospechoso en una cuenta de Azure AD. Esta indicación proporciona un alto grado de confianza en que la cuenta que aparece en la descripción del incidente de Fusion se ha puesto en peligro y se ha usado para cifrar datos con el fin de chantajear al propietario de los datos o de denegarle el acceso a ellos. Las permutaciones de alertas de inicios de sesión sospechosos en Azure AD con la alerta de ejecución de ransomware son:  

- **Un viaje imposible a una ubicación inusual que conduce a ransomware en las aplicaciones en la nube**

- **Evento de inicio de sesión desde una ubicación desconocida que conducen a ransomware en las aplicaciones en la nube**

- **Evento de inicio de sesión desde un dispositivo infectado que conduce a ransomware en las aplicaciones en la nube**

- **Un evento de inicio de sesión desde una dirección IP anónima que conduce a ransomware en las aplicaciones en la nube**

- **Un evento de inicio de sesión desde un usuario con credenciales filtradas que conduce a ransomware en las aplicaciones en la nube**

## <a name="remote-exploitation"></a>Explotación remota

### <a name="suspected-use-of-attack-framework-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Sospecha de uso de un marco de ataque seguida por tráfico anómalo marcado por el firewall de Palo Alto Networks
Este escenario está actualmente en **versión preliminar pública**.

**Tácticas de MITRE ATT&CK:** Acceso inicial, ejecución, desplazamiento lateral y elevación de privilegios

**Técnicas de MITRE ATT&CK:** Explotación de una aplicación de acceso público (T1190), explotación de la ejecución del cliente (T1203), explotación de servicios remotos (T1210) y explotación para la elevación de privilegios (T1068)

**Orígenes de conexión de datos:** Microsoft Defender para punto de conexión (anteriormente MDATP) y Palo Alto Networks 

**Descripción:** Los incidentes de Fusion de este tipo indican que se han detectado usos no estándar de protocolos que se asemejan al uso de marcos de ataque (como Metasploit) y, después, el firewall de Palo Alto Networks ha detectado una actividad de entrada sospechosa. Puede tratarse de una indicación inicial de que un atacante ha aprovechado un servicio para acceder a los recursos de la red, o bien que ya ha accedido a ellos y está intentando explotar aún más los sistemas y servicios disponibles para desplazarse lateralmente o escalar privilegios. En los registros de Palo Alto, Azure Sentinel se centra en [registros de amenazas](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs) y el tráfico se considera sospechoso cuando se permiten amenazas (datos sospechosos, archivos, inundaciones, paquetes, exámenes, spyware, direcciones URL, virus, puntos vulnerables, virus Wildfire y malware Wildfire). También puede consultar el registro de amenazas de Palo Alto que se corresponde con el [tipo de amenaza/contenido](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) enumerado en la descripción del incidente de Fusion para obtener más información sobre las alertas.

## <a name="next-steps"></a>Pasos siguientes

Ahora que conoce más detalles sobre la detección avanzada de ataques de varias fases, puede que le interese el siguiente inicio rápido para aprender a obtener visibilidad sobre sus datos y a detectar posibles amenazas: [Introducción a Azure Sentinel](quickstart-get-visibility.md).

Si está preparado para investigar los incidentes que se han creado, consulte el siguiente tutorial: [Investigación de incidentes con Azure Sentinel](tutorial-investigate-cases.md).

