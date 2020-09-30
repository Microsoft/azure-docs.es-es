---
title: Conexión de las cuentas de AWS a Azure Security Center
description: Supervisión de los recursos de AWS desde Azure Security Center
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: ee1b8fe6ed97f3b71cda418cce9e432f7c045447
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932430"
---
#  <a name="connect-your-aws-accounts-to-azure-security-center"></a>Conexión de las cuentas de AWS a Azure Security Center

Las cargas de trabajo de nube abarcan normalmente varias plataformas de nube, por lo que los servicios de seguridad de la nube deben hacer lo mismo.

Azure Security Center protege las cargas de trabajo de Azure, Amazon Web Services (AWS) y Google Cloud Platform (GCP).

Al incorporar la cuenta de AWS a Security Center, se integran AWS Security Hub y Azure Security Center. Por lo tanto, Security Center ofrece visibilidad y protección en ambos entornos de nube para proporcionar:

- Aprovisionamiento automático de agentes (Security Center usa [Azure Arc](../azure-arc/servers/overview.md) para implementar el agente de Log Analytics en las instancias de AWS)
- Administración de directivas
- Administración de vulnerabilidades
- Detección y respuesta de puntos de conexión (EDR) integrados
- Detección de errores de configuración de seguridad
- Una sola vista que muestra recomendaciones de Security Center y resultados de AWS Security Hub
- Incorporación de los recursos de AWS a los cálculos de puntuación segura de Security Center
- Evaluaciones de cumplimiento normativo de los recursos de AWS

En la captura de pantalla siguiente puede ver que se muestran cuentas de AWS en el panel de información general de Security Center.

:::image type="content" source="./media/quickstart-onboard-aws/aws-account-in-overview.png" alt-text="Tres proyectos de GCP que se muestran en el panel de información general de Security Center" lightbox="./media/quickstart-onboard-gcp/gcp-account-in-overview.png":::

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Versión preliminar|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).|
|Roles y permisos necesarios:|**propietario** o **colaborador** en la suscripción de Azure en cuestión|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||



## <a name="connect-your-aws-account"></a>Conexión de una cuenta de AWS

### <a name="step-1-set-up-aws-security-hub"></a>Paso 1. Configuración de AWS Security Hub:

1. Para ver recomendaciones de seguridad para varias regiones, repita los pasos siguientes para cada región pertinente.

    > [!IMPORTANT]
    > Si utiliza una cuenta maestra de AWS, repita los tres pasos siguientes para configurar la cuenta maestra y todas las cuentas miembros conectadas de todas las regiones pertinentes.

    1. Habilite [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html) (Configuración de AWS).
    1. Habilite [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html).
    1. Compruebe que hay datos que fluyen a Security Hub.

        La primera vez que se habilita Security Hub, los datos pueden tardar varias horas en estar disponibles.

### <a name="step-2-set-up-authentication-for-security-center-in-aws"></a>Paso 2. Configuración de la autenticación para Security Center en AWS

Hay dos maneras de permitir que Security Center se autentique en AWS:

- **Crear un rol IAM para Security Center**: este es el método más seguro y es el que se recomienda.
- **Usuario de AWS para Security Center**: opción menos segura si no tiene IAM habilitado.

#### <a name="create-an-iam-role-for-security-center"></a>Creación de un rol IAM para Security Center
1. En la consola de Amazon Web Services, en **Security, Identity & Compliance** (Seguridad, identidad y cumplimiento), seleccione **IAM**.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-identity-and-compliance.png" alt-text="Servicios de AWS":::

1. Seleccione **Roles** (Roles) y **Create role** (Crear rol).
1. Seleccione **Another AWS account** (Otra cuenta de AWS).
1. Escriba la siguiente información:

    - **Account ID** (Id. de cuenta): escriba el identificador de la cuenta Microsoft (**158177204117**) que aparece en la página del conector de AWS en Security Center.
    - **Require External ID** (Requerir id. externo): esta opción debe estar seleccionada.
    - **External ID** (Id. externo): escriba el identificador de suscripción que se muestra en la página del conector de AWS en Security Center. 

1. Seleccione **Next** (Siguiente).
1. En la sección **Attach permission policies** (Asociar directivas de permisos), seleccione las siguientes directivas:

    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess

1. Tiene la opción de agregar etiquetas. La acción de agregar etiquetas al usuario no afecta a la conexión.
1. Seleccione **Next** (Siguiente).

1. En la lista de roles, seleccione el rol que ha creado.

1. Guarde el nombre del recurso de Amazon (ARN) para más adelante. 

#### <a name="create-an-aws-user-for-security-center"></a>Creación de un usuario de AWS para Security Center 
1. Abra la pestaña **Users** (Usuarios) y seleccione **Add user** (Agregar usuario).
1. En el paso **Details** (Detalles), escriba un nombre de usuario para Security Center y asegúrese de seleccionar **Programmatic access** (Acceso mediante programación) para el tipo de acceso de AWS. 
1. Seleccione **Next Permissions** (Siguientes permisos).
1. Seleccione **Attach existing policies directly** (Asociar directivas existentes directamente):
    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess
    
1. Seleccione **Siguiente: Etiquetas**. Tiene la opción de agregar etiquetas. La acción de agregar etiquetas al usuario no afecta a la conexión.
1. Seleccione **Review** (Revisar).
1. Guarde el archivo CSV de **Access key ID** (Id. de clave de acceso) y **Secret access key** (Clave de acceso secreta) generado automáticamente para más adelante.
1. Revise el resumen y luego haga clic en **Create user** (Crear usuario).


### <a name="step-3-configure-the-ssm-agent"></a>Paso 3. Configuración del agente SSM

Para automatizar las tareas entre los recursos de AWS, se requiere el administrador de sistemas de AWS. Si las instancias de EC2 no tienen el agente SSM, siga las instrucciones pertinentes desde Amazon:

- [Instalación y configuración del agente SSM en instancias de Windows](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-win.html)
- [Instalación y configuración del agente SSM en instancias de Linux de Amazon EC2](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html)


### <a name="step-4-create-a-service-principal-for-onboarding-at-scale"></a>Paso 4. Creación de una entidad de servicio para la incorporación a gran escala

Como **propietario** de la suscripción que quiere usar para la incorporación, cree una entidad de servicio para la incorporación de Azure Arc como se describe en [Creación de una entidad de servicio para la incorporación a escala](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).


### <a name="step-5-connect-aws-to-security-center"></a>Paso 5. Conexión de AWS a Security Center

1. En el menú de Security Center, seleccione **Conectores multinube**.
1. Seleccione **Add AWS account** (Agregar cuenta de AWS).
    :::image type="content" source="./media/quickstart-onboard-aws/add-aws-account.png" alt-text="Botón Add AWS account (Agregar cuenta de AWS) de la página Conectores multinube de Security Center":::
1. Configure las opciones en la pestaña **AWS authentication** (Autenticación de AWS):
    1. Escriba un **nombre para mostrar** para el conector.
    1. Confirme que la suscripción es correcta. Se trata de la suscripción que incluirá las recomendaciones del conector y de AWS Security Hub.
    1. En función de la opción de autenticación elegida en el [Paso 2. Configuración de la autenticación para Security Center en AWS](#step-2-set-up-authentication-for-security-center-in-aws):
        - Seleccione **Assume Role** (Asumir rol) y pegue el ARN desde [Creación de un rol IAM para Security Center](#create-an-iam-role-for-security-center) :::image type="content" source="./media/quickstart-onboard-aws/paste-arn-in-portal.png" alt-text="Pegado del archivo ARN en el campo correspondiente del asistente de conexión de AWS en Azure Portal":::

            O BIEN

        - Seleccione **Credentials** (Credenciales) y pegue la **clave de acceso** y la **clave secreta** del archivo .csv que guardó en [Creación de un usuario de AWS para Security Center](#create-an-aws-user-for-security-center).
1. Seleccione **Next** (Siguiente).
1. Configure las opciones de la pestaña **Configuración de Azure Arc**:

    Security Center detecta las instancias de EC2 en la cuenta de AWS conectada y usa SSM para incorporarlas a Azure Arc. 

    > [!TIP]
    > La lista de sistemas operativos compatibles se encuentra en las siguientes preguntas frecuentes.

    1. Seleccione el **grupo de recursos** y la **región de Azure** a los que se incorporarán las instancias de AWS EC2 detectadas en la suscripción seleccionada.
    1. Escriba el **identificador de la entidad de servicio** y el **secreto de cliente de la entidad de servicio** de Azure Arc, como se describe en [Creación de una entidad de servicio para la incorporación a escala](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale)
    1. Si la máquina se conecta a Internet mediante un servidor proxy, especifique la dirección IP del servidor proxy o el nombre y el número de puerto que usará la máquina para comunicarse con él. Escriba el valor con el formato ```http://<proxyURL>:<proxyport>```.
    1. Seleccione **Revisar + crear**.

        Revisión de la información de resumen

        En las secciones de etiquetas se enumeran todas las etiquetas de Azure que se crearán automáticamente para cada instancia de EC2 incorporada con sus propios detalles pertinentes para reconocerla fácilmente en Azure. 

        Más información sobre las etiquetas de Azure en [Uso de etiquetas para organizar los recursos de Azure y la jerarquía de administración](../azure-resource-manager/management/tag-resources.md).

### <a name="step-7-confirmation"></a>Paso 7. Confirmación

Cuando el conector se ha creado correctamente y AWS Security Hub se ha configurado adecuadamente:

- Security Center examina el entorno en busca de las instancias de AWS EC2 y las incorpora a Azure Arc, lo que permite instalar el agente de Log Analytics y proporcionar recomendaciones de seguridad y protección contra amenazas. 
- El servicio ASC busca nuevas instancias de AWS EC2 cada 6 horas y las incorpora de acuerdo con la configuración.
- El estándar CIS de AWS se mostrará en el panel de cumplimiento normativo de Security Center.
- Si está habilitada la directiva de Security Hub, las recomendaciones aparecerán en el portal de Security Center y en el panel de cumplimiento normativo entre 5 y 10 minutos después de que finalice la incorporación.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png" alt-text="Recursos y recomendaciones de AWS en la página de recomendaciones de Security Center":::



## <a name="monitoring-your-aws-resources"></a>Supervisión de los recursos de AWS

Como se mostró anteriormente, en la página de recomendaciones de seguridad de Azure Security Center aparecen los recursos de AWS junto con los recursos de Azure y GCP, lo que ofrece una verdadera vista de varias nubes.

Para ver todas las recomendaciones activas de los recursos por tipo de recurso, use la página de inventario de recursos de Security Center y filtre por el tipo de recurso de AWS que le interesa:

:::image type="content" source="./media/quickstart-onboard-aws/aws-resource-types-in-inventory.png" alt-text="Filtro de tipo de recurso de la página de inventario de recursos que muestra las opciones de AWS"::: 


## <a name="aws-in-security-center-faq"></a>Preguntas frecuentes sobre AWS en Security Center

### <a name="what-operating-systems-for-my-ec2-instances-are-supported"></a>¿Qué sistemas operativos son compatibles con las instancias de EC2?

Sistema operativo compatible con la incorporación automática a Azure Arc para máquinas AWS

- Ubuntu 16.04: el agente SSM está preinstalado de forma predeterminada
- Ubuntu 18.04: el agente SSM está preinstalado de forma predeterminada
- Windows Server: el agente-SSM está preinstalado de forma predeterminada
- CentOS Linux 7: se debe instalar SSM manualmente o incorporarse por separado
- SUSE Linux Enterprise Server (SLES) 15 (x64): se debe instalar SSM manualmente o incorporarse por separado
- Red Hat Enterprise Linux (RHEL) 7 (x64): se debe instalar SSM manualmente o incorporarse por separado


## <a name="next-steps"></a>Pasos siguientes

La conexión de la cuenta de AWS forma parte de la experiencia de varias nubes disponible en Azure Security Center. Para información relacionada, consulte la página siguiente:

- [Conexión de las cuentas de GCP a Azure Security Center](quickstart-onboard-gcp.md)
