---
title: Implementación de una aplicación en Azure Stack | Microsoft Docs
description: Aprenda a implementar aplicaciones en Azure y Azure Stack con una canalización de CI/CD híbrida.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/19/2019
ms.author: mabrigg
ms.reviewer: anajod
ms.lastreviewed: 11/07/2018
ms.openlocfilehash: bfbafea13f49ada59dae88e0336f926a8ad89d36
ms.sourcegitcommit: 9aa9552c4ae8635e97bdec78fccbb989b1587548
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/20/2019
ms.locfileid: "56430257"
---
# <a name="tutorial-deploy-apps-to-azure-and-azure-stack"></a>Tutorial: Implementación de aplicaciones en Azure y Azure Stack

*Se aplica a: sistemas integrados de Azure Stack y Kit de desarrollo de Azure Stack*

Obtenga información sobre cómo implementar una aplicación en Azure y Azure Stack mediante una canalización híbrida de integración continua y entrega continua (CI/CD).

En este tutorial, creará un entorno de ejemplo para:

> [!div class="checklist"]
> * Iniciar una nueva compilación en función de las confirmaciones de código en el repositorio de Azure DevOps Services.
> * Implementar automáticamente la aplicación en Azure global para llevar a cabo las pruebas de aceptación del usuario.
> * Una vez que el código haya pasado las pruebas, implementar automáticamente la aplicación en Azure Stack.

## <a name="benefits-of-the-hybrid-delivery-build-pipe"></a>Ventajas de la canalización de compilación y entrega híbrida

La continuidad, la seguridad y la confiabilidad son elementos clave de la implementación de aplicaciones. Estos elementos son esenciales para la organización y fundamentales para el equipo de desarrollo. Una canalización de CI/CD híbrida le permite consolidar las canalizaciones de compilación en el entorno local y en la nube pública. Un modelo de entrega híbrido, a su vez, le permite cambiar las ubicaciones de implementación sin modificar la aplicación.

Otras ventajas derivadas del uso del enfoque híbrido son:

* Puede mantener un conjunto coherente de herramientas de desarrollo en el entorno de Azure Stack local y la nube pública de Azure.  Un conjunto de herramientas común facilita la implementación de patrones y prácticas de CI/CD.
* Las aplicaciones y los servicios implementados en Azure y Azure Stack son intercambiables y el mismo código se puede ejecutar en cualquier ubicación. Puede aprovechar las ventajas de las características y funcionalidades del entorno local y la nube pública.

Para más información sobre la integración continua y la implementación continua:

* [What is Continuous Integration?](https://www.visualstudio.com/learn/what-is-continuous-integration/) (¿Qué es la integración continua?)
* [What is Continuous Delivery?](https://www.visualstudio.com/learn/what-is-continuous-delivery/) (¿Qué es la entrega continua?)

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack es una extensión de Azure. Azure Stack aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.  
> 
> En las notas del producto [Consideraciones de diseño para aplicaciones híbridas](https://aka.ms/hybrid-cloud-applications-pillars) se revisan los pilares de la calidad de software (selección de ubicación, escalabilidad, disponibilidad, resistencia, capacidad de administración y seguridad) para diseñar, implementar y usar aplicaciones híbridas. Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.

## <a name="prerequisites"></a>Requisitos previos

Necesita contar con algunos componentes para crear una canalización de CI/CD híbrida. Los siguientes componentes tendrán un tiempo de preparación:

* Un asociado de OEM o hardware de Azure puede implementar una instancia de Azure Stack de producción. Todos los usuarios pueden implementar el Kit de desarrollo de Azure Stack (ASDK).
* Además, un operador de Azure Stack debe implementar la instancia de App Service, crear planes y ofertas, crear una suscripción de inquilino y agregar la imagen de Windows Server 2016.

>[!NOTE]
>Si ya tiene implementados algunos de estos componentes, asegúrese de que cumplen todos los requisitos antes de empezar este tutorial.

En este tutorial se da por supuesto que tiene algunos conocimientos básicos de Azure y Azure Stack. Para más información antes de iniciar el tutorial, lea los siguientes artículos:

* [Introducción a Azure](https://azure.microsoft.com/overview/what-is-azure/)
* [Conceptos clave de Azure Stack](https://docs.microsoft.com/azure/azure-stack/azure-stack-key-features)

### <a name="azure-requirements"></a>Requisitos de Azure

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Cree una [aplicación web](https://docs.microsoft.com/azure/app-service/overview) en Azure. Tome nota de la dirección URL de la aplicación web, ya que la necesitará en el tutorial.

### <a name="azure-stack-requirements"></a>Requisitos de Azure Stack

* Use un sistema integrado de Azure Stack o implemente el Kit de desarrollo de Azure Stack (ASDK). Para implementar ASDK:
    * El [Tutorial: Implementación de ASDK con el instalador](https://docs.microsoft.com/azure/azure-stack/asdk/asdk-deploy) ofrece instrucciones de implementación detalladas.
    * Use el script de PowerShell [ConfigASDK.ps1](https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1 ) para automatizar los pasos posteriores a la implementación de ASDK.

    > [!Note]
    > La instalación de ASDK tarda aproximadamente siete horas en completarse, así que realice los preparativos oportunos.

 * Implemente servicios PaaS de [App Service](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-deploy) para Azure Stack.
 * Cree [Planes y ofertas](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview) en Azure Stack.
 * Cree una [suscripción de inquilino](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm) en Azure Stack.
 * Cree una aplicación web en la suscripción de inquilino. Anote la nueva dirección URL de aplicación web, ya que la usará más adelante.
 * Implemente una máquina virtual Windows Server 2012 en la suscripción del inquilino. Este servidor se utilizará como servidor de compilación y para ejecutar Azure DevOps Services.
* Proporcione una imagen de Windows Server 2016 con .NET 3.5 para una máquina virtual (VM). Esta máquina virtual se crea en la instancia de Azure Stack como un agente de compilación privado.

### <a name="developer-tool-requirements"></a>Requisitos de la herramienta para desarrolladores

* Cree un [área de trabajo de Azure DevOps Services](https://docs.microsoft.com/azure/devops/repos/tfvc/create-work-workspaces). El proceso de registro crea un proyecto denominado **MyFirstProject**.
* [Instale Visual Studio 2017](https://docs.microsoft.com/visualstudio/install/install-visual-studio) e [inicie sesión en Azure DevOps Services](https://www.visualstudio.com/docs/setup-admin/team-services/connect-to-visual-studio-team-services).
* Conéctese al proyecto y [realice la clonación localmente](https://www.visualstudio.com/docs/git/gitquickstart).

 > [!Note]
 > El entorno de Azure Stack necesita las imágenes correctas sindicadas para ejecutar Windows Server y SQL Server. También debe tener implementado App Service.

## <a name="prepare-the-private-azure-pipelines-agent-for-azure-devops-services-integration"></a>Preparación del agente de Azure Pipelines privado para la integración de Azure DevOps Services

### <a name="prerequisites"></a>Requisitos previos

Azure DevOps Services se autentica en Azure Resource Manager mediante una entidad de servicio. Azure DevOps Services debe tener el rol de **Colaborador** para aprovisionar recursos en una suscripción de Azure Stack.

Los pasos siguientes describen lo que se requiere para configurar la autenticación:

1. Cree una entidad de servicio o utilice una existente.
2. Cree las claves de autenticación para la entidad de servicio.
3. Valide la suscripción de Azure Stack mediante el control de acceso basado en rol para permitir que el nombre de entidad de seguridad de servicio (SPN) forme parte del rol de colaborador.
4. Cree una nueva definición de servicio en Azure DevOps Services con los puntos de conexión de Azure Stack y la información de SPN.

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Consulte las instrucciones de [creación de entidades de servicio](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications) para crear una. Elija **Aplicación web o API** como tipo de aplicación o [use el script de PowerShell](https://github.com/Microsoft/vsts-rm-extensions/blob/master/TaskModules/powershell/Azure/SPNCreation.ps1#L5) como se explica en el artículo [Create an Azure Resource Manager service connection with an existing service principal](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal) (Creación de una conexión de servicio de Azure Resource Manager con una entidad de servicio existente).

 > [!Note]  
 > Si utiliza el script para crear un punto de conexión de Azure Resource Manager de Azure Stack, tiene que pasar los parámetros **-azureStackManagementURL** y **-environmentName**. Por ejemplo:   
> `-azureStackManagementURL https://management.local.azurestack.external -environmentName AzureStack`

### <a name="create-an-access-key"></a>Creación de una clave de acceso

Una entidad de servicio requiere una clave para la autenticación. Siga estos pasos para generar una clave.

1. En **Registros de aplicaciones**, en Azure Active Directory, seleccione su aplicación.

    ![Selección de la aplicación](media/azure-stack-solution-hybrid-pipeline/000_01.png)

2. Anote el valor del **identificador de aplicación**. Este valor se usará al configurar el punto de conexión de servicio en Azure DevOps Services.

    ![Identificador de aplicación](media/azure-stack-solution-hybrid-pipeline/000_02.png)

3. Para generar una clave de autenticación, seleccione **Configuración**.

    ![Edición de la configuración de la aplicación](media/azure-stack-solution-hybrid-pipeline/000_03.png)

4. A continuacion, seleccione **Claves**.

    ![Configuración de la clave](media/azure-stack-solution-hybrid-pipeline/000_04.png)

5. Proporcione una descripción de la clave y establezca la duración. Cuando haya terminado, seleccione **Guardar**.

    ![Descripción y duración de la clave](media/azure-stack-solution-hybrid-pipeline/000_05.png)

    Después de guardar la clave, se muestra la clave **VALUE**. Copie este valor, ya que no se puede obtener más adelante. Se proporciona el **valor de clave** junto con el identificador de la aplicación para iniciar sesión como la aplicación. Guarde el valor de clave donde la aplicación pueda recuperarlo.

    ![Valor de clave](media/azure-stack-solution-hybrid-pipeline/000_06.png)

### <a name="get-the-tenant-id"></a>Obtención del identificador de inquilino

Como parte de la configuración del punto de conexión de servicio, Azure DevOps Services requiere el **identificador de inquilino** que corresponde al directorio de AAD en el que se implementó la marca de Azure Stack. Utilice los pasos siguientes para obtener el identificador de inquilino.

1. Seleccione **Azure Active Directory**.

    ![Azure Active Directory del inquilino](media/azure-stack-solution-hybrid-pipeline/000_07.png)

2. Para obtener el identificador de inquilino, seleccione **Propiedades** en el inquilino de Azure AD.

    ![Visualización de las propiedades del inquilino](media/azure-stack-solution-hybrid-pipeline/000_08.png)

3. Copie el **id. de directorio**. Este valor es el id. de inquilino.

    ![Identificador de directorio](media/azure-stack-solution-hybrid-pipeline/000_09.png)

### <a name="grant-the-service-principal-rights-to-deploy-resources-in-the-azure-stack-subscription"></a>Concesión de derechos de entidad de servicio para implementar recursos en la suscripción de Azure Stack

Para acceder a los recursos de la suscripción, debe asignarle a la aplicación un rol. Decida qué rol representa los permisos más adecuados para la aplicación. Para más información sobre los roles disponibles, vea [RBAC: Roles integrados](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles).

Puede establecer el ámbito en el nivel de suscripción, grupo de recursos o recurso. Los permisos se heredan en los niveles inferiores del ámbito. Por ejemplo, el hecho de agregar una aplicación al rol Lector para un grupo de recursos significa que esta puede leer el grupo de recursos y cualquiera de sus recursos.

1. Desplácese hasta el nivel de ámbito al que desea asignar la aplicación. Por ejemplo, para asignar un rol en el ámbito de suscripción, seleccione **Suscripciones**.

    ![Selección de suscripciones](media/azure-stack-solution-hybrid-pipeline/000_10.png)

2. En **Suscripción**, seleccione Visual Studio Enterprise.

    ![Visual Studio Enterprise](media/azure-stack-solution-hybrid-pipeline/000_11.png)

3. En Visual Studio Enterprise, seleccione **Control de acceso (IAM)**.

4. Seleccione **Agregar asignación de roles**.

    ![Sumar](media/azure-stack-solution-hybrid-pipeline/000_13.png)

5. En **Agregar permisos**, seleccione el rol que desea asignar a la aplicación. En este ejemplo, el rol de **Propietario**.

    ![Rol de propietario](media/azure-stack-solution-hybrid-pipeline/000_14.png)

6. De manera predeterminada, las aplicaciones de Azure Active Directory no se muestran en las opciones disponibles. Para buscar la aplicación, debe proporcionar el nombre en el campo **Seleccionar** para buscarla. Seleccione la aplicación.

    ![Resultado de la búsqueda de la aplicación](media/azure-stack-solution-hybrid-pipeline/000_16.png)

7. Haga clic en **Guardar** para finalizar la asignación del rol. Verá la aplicación en la lista de usuarios asignados a un rol para ese ámbito.

### <a name="role-based-access-control"></a>Control de acceso basado en roles

El control de acceso basado en rol (RBAC) de Azure permite realizar una administración del acceso pormenorizada para Azure. Mediante el uso de RBAC, puede controlar el nivel de acceso que los usuarios necesitan para realizar sus trabajos. Para más información sobre el control de acceso basado en rol, consulte [Administración del acceso a los recursos de suscripción de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal?toc=%252fazure%252factive-directory%252ftoc.json).

### <a name="azure-devops-services-agent-pools"></a>Grupos de agentes de Azure DevOps Services

En lugar de administrar cada agente de forma individual, puede organizar los agentes en grupos de agentes. Un grupo de agentes define el límite de uso compartido de todos los agentes de ese grupo. En Azure DevOps Services, los grupos de agentes tienen el ámbito de la organización de Azure DevOps Services, lo que significa que puede compartir un grupo de agentes entre proyectos. Para más información acerca de los grupos de agentes, consulte [Creación de grupos de agentes y colas](https://docs.microsoft.com/azure/devops/pipelines/agents/pools-queues?view=vsts).

### <a name="add-a-personal-access-token-pat-for-azure-stack"></a>Adición de un token de acceso personal (PAT) a Azure Stack

Cree un token de acceso personal para acceder a Azure DevOps Services.

1. Inicie sesión en su organización de Azure DevOps Services y seleccione el nombre de perfil de la organización.

2. Seleccione **Administrar seguridad** para acceder a la página de creación de tokens de acceso.

    ![Inicio de sesión de usuario](media/azure-stack-solution-hybrid-pipeline/000_17.png)

    ![Selección de un proyecto](media/azure-stack-solution-hybrid-pipeline/000_18.png)

    ![Adición del token de acceso personal](media/azure-stack-solution-hybrid-pipeline/000_18a.png)

    ![Creación del token](media/azure-stack-solution-hybrid-pipeline/000_18b.png)

3. Copie el token.

    > [!Note]
    > Guarde la información del token. Esta información no se guarda y no se volverá a mostrar cuando abandone la página web.

    ![Token de acceso personal](media/azure-stack-solution-hybrid-pipeline/000_19.png)

### <a name="install-the-azure-devops-services-build-agent-on-the-azure-stack-hosted-build-server"></a>Instalación del agente de compilación de Azure DevOps Services en el servidor de compilación hospedado en Azure Stack

1. Conéctese a su servidor de compilación que implementó en el host de Azure Stack.
2. Descargue e instale el agente de compilación como un servicio mediante su token de acceso personal (PAT) y ejecútelo como la cuenta de administrador de la máquina virtual.

    ![Descarga del agente de compilación](media/azure-stack-solution-hybrid-pipeline/010_downloadagent.png)

3. Vaya a la carpeta del agente de compilación extraído. Ejecute el archivo **config.cmd** desde un símbolo del sistema con privilegios elevados.

    ![Agente de compilación extraído](media/azure-stack-solution-hybrid-pipeline/000_20.png)

    ![Registro del agente de compilación](media/azure-stack-solution-hybrid-pipeline/000_21.png)

4. Cuando finalice la ejecución de config.cmd, se actualiza la carpeta del agente de compilación con archivos adicionales. La carpeta con el contenido extraído tendrá este aspecto:

    ![Actualizar de la carpeta del agente de compilación](media/azure-stack-solution-hybrid-pipeline/009_token_file.png)

    Puede ver el agente en la carpeta de Azure DevOps Services.

## <a name="endpoint-creation-permissions"></a>Permisos de creación de puntos de conexión

Mediante la creación de puntos de conexión, una compilación de Visual Studio Online (VSTO) puede implementar aplicaciones de Azure Service en Azure Stack. Azure DevOps Services se conecta con el agente de compilación, que, a su vez, se conecta con Azure Stack.

![Aplicación de ejemplo NorthwindCloud en VSTO](media/azure-stack-solution-hybrid-pipeline/012_securityendpoints.png)

1. Inicie sesión en VSTO y vaya a la página de configuración de la aplicación.
2. En **Configuración**, seleccione **Seguridad**.
3. En **Grupos de Azure DevOps Services**, seleccione **Creadores de puntos de conexión**.

    ![Creadores de puntos de conexión de NorthwindCloud](media/azure-stack-solution-hybrid-pipeline/013_endpoint_creators.png)

4. En la pestaña **Miembros**, seleccione **Agregar**.

    ![Agregar un miembro](media/azure-stack-solution-hybrid-pipeline/014_members_tab.png)

5. En **Agregar usuarios y grupos**, escriba un nombre de usuario y seleccione ese usuario en la lista de usuarios.
6. Seleccione **Save changes** (Guardar los cambios).
7. En la lista **Grupos de Azure DevOps Services**, seleccione **Administradores de puntos de conexión**.

    ![Administradores de puntos de conexión de NorthwindCloud](media/azure-stack-solution-hybrid-pipeline/015_save_endpoint.png)

8. En la pestaña **Miembros**, seleccione **Agregar**.
9. En **Agregar usuarios y grupos**, escriba un nombre de usuario y seleccione ese usuario en la lista de usuarios.
10. Seleccione **Save changes** (Guardar los cambios).

Ahora que existe la información del punto de conexión, la conexión de Azure DevOps Services con Azure Stack está lista para su uso. El agente de compilación de Azure Stack obtiene instrucciones de Azure DevOps Services y, a continuación, el agente transmite la información del punto de conexión para la comunicación con Azure Stack.

## <a name="create-an-azure-stack-endpoint"></a>Creación de un punto de conexión de Azure Stack

### <a name="create-an-endpoint-for-azure-ad-deployments"></a>Creación de un punto de conexión para implementaciones de Azure AD

Puede seguir las instrucciones del artículo [Create an Azure Resource Manager service connection with an existing service principal](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal) (Creación de una conexión de servicio de Azure Resource Manager con una entidad de servicio existente) para crear una conexión de servicio con una entidad de servicio existente y usar la siguiente asignación:

Puede crear una conexión de servicio mediante la asignación siguiente:

| NOMBRE | Ejemplo | DESCRIPCIÓN |
| --- | --- | --- |
| Nombre de conexión | Azure AD de Azure Stack | El nombre de la conexión |
| Entorno | AzureStack | El nombre del entorno |
| Dirección URL del entorno | `https://management.local.azurestack.external` | El punto de conexión de administración |
| Nivel de ámbito | Subscription | El ámbito de la conexión |
| Id. de suscripción | 65710926-XXXX-4F2A-8FB2-64C63CD2FAE9 | Identificador de suscripción del usuario de Azure Stack |
| Nombre de la suscripción | name@contoso.com | El nombre de suscripción del usuario de Azure Stack |
| Id. de cliente de la entidad de servicio | FF74AACF-XXXX-4776-93FC-C63E6E021D59 | El identificador de la entidad de [esta](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-pipeline#create-a-service-principal) sección del artículo |
| Clave de entidad de servicio | THESCRETGOESHERE= | La clave del mismo artículo (o la contraseña si usa el script) |
| Id. de inquilino | D073C21E-XXXX-4AD0-B77E-8364FCA78A94 | El identificador del inquilino que se recupera con las instrucciones de [Obtención del identificador de inquilino](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-pipeline#get-the-tenant-id)  |
| Conexión: | Not verified (Sin comprobar) | Valide la configuración de conexión a la entidad de servicio. |

Ahora que se ha creado el punto de conexión, la conexión de DevOps con Azure Stack está lista para su uso. El agente de compilación de Azure Stack obtiene instrucciones de DevOps y, a continuación, el agente transmite la información del punto de conexión para la comunicación con Azure Stack.

![Creación de un agente de Azure AD](media/azure-stack-solution-hybrid-pipeline/016_save_changes.png)

### <a name="create-an-endpoint-for-ad-fs"></a>Creación de un punto de conexión para AD FS

La última actualización de Azure DevOps permite crear una conexión de servicio mediante una entidad de servicio con un certificado de autenticación. Esto es necesario cuando se implementa Azure Stack con AD FS como proveedor de identidades. 

![Creación de un agente de AD FS](media/azure-stack-solution-hybrid-pipeline/image06.png)

Puede crear una conexión de servicio mediante la asignación siguiente:

| NOMBRE | Ejemplo | DESCRIPCIÓN |
| --- | --- | --- |
| Nombre de conexión | Azure Stack ADFS | El nombre de la conexión |
| Entorno | AzureStack | El nombre del entorno |
| Dirección URL del entorno | `https://management.local.azurestack.external` | El punto de conexión de administración |
| Nivel de ámbito | Subscription | El ámbito de la conexión |
| Id. de suscripción | 65710926-XXXX-4F2A-8FB2-64C63CD2FAE9 | Identificador de suscripción del usuario de Azure Stack |
| Nombre de la suscripción | name@contoso.com | El nombre de suscripción del usuario de Azure Stack |
| Id. de cliente de la entidad de servicio | FF74AACF-XXXX-4776-93FC-C63E6E021D59 | El identificador de cliente de la entidad de servicio que se creó para AD FS. |
| Certificate | `<certificate>` |  Convierta el archivo de certificado de PFX a PEM Pegue el contenido del archivo de certificado PEM en este campo. <br> Conversión de PFX a PEM:<br>`openssl pkcs12 -in file.pfx -out file.pem -nodes -password pass:<password_here>` |
| Id. de inquilino | D073C21E-XXXX-4AD0-B77E-8364FCA78A94 | El identificador del inquilino que se recupera con las instrucciones de [Obtención del identificador de inquilino](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-pipeline#get-the-tenant-id) |
| Conexión: | Not verified (Sin comprobar) | Valide la configuración de conexión a la entidad de servicio. |

Ahora que se ha creado el punto de conexión, la conexión de Azure DevOps con Azure Stack está lista para su uso. El agente de compilación de Azure Stack obtiene instrucciones de Azure DevOps y, a continuación, el agente transmite la información del punto de conexión para la comunicación con Azure Stack.

> [!Note]
> Si el punto de conexión de ARM del usuario de Azure Stack no se expone a Internet, se producirá un error en la validación de conexión. Esto es normal y puede validar su conexión mediante la creación de una canalización de versión con una tarea simple. 

## <a name="develop-your-application-build"></a>Desarrollo de la aplicación

En esta parte del tutorial, vamos a:

* Agregar código a un proyecto de Azure DevOps Services.
* Crear una implementación de aplicaciones web independiente.
* Configuración del proceso de implementación continua

> [!Note]
 > El entorno de Azure Stack necesita las imágenes correctas sindicadas para ejecutar Windows Server y SQL Server. También debe tener implementado App Service. Revise la sección "Requisitos previos" de la documentación de App Service para conocer los requisitos del operador de Azure Stack.

La canalización de CI/CD híbrida se puede aplicar al código de aplicación y al código de infraestructura. Use [plantillas de Azure Resource Manager como código de aplicación web](https://azure.microsoft.com/resources/templates/) de Azure DevOps Services para implementar en ambas nubes.

### <a name="add-code-to-an-azure-devops-services-project"></a>Adición de código a un proyecto de Azure DevOps Services

1. Inicie sesión en Azure DevOps Services con una organización que tenga derechos de creación de proyectos en Azure Stack. La captura de pantalla siguiente muestra cómo conectarse al proyecto HybridCICD.

    ![Conexión a un proyecto](media/azure-stack-solution-hybrid-pipeline/017_connect_to_project.png)

2. **Clone el repositorio**; para ello, cree y abra la aplicación web predeterminada.

    ![Clonación del repositorio](media/azure-stack-solution-hybrid-pipeline/018_link_arm.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Creación de una implementación de aplicación web autocontenida para App Services en ambas nubes

1. Edite el archivo **WebApplication.csproj**: seleccione **Runtimeidentifier** y luego agregue `win10-x64.` Para más información, consulte la documentación [Implementaciones independientes](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd).

    ![Configuración de Runtimeidentifier](media/azure-stack-solution-hybrid-pipeline/019_runtimeidentifer.png)

2. Compruebe el código en Azure DevOps Services mediante Team Explorer.

3. Confirme que el código de la aplicación se ha insertado en Azure DevOps Services.

### <a name="create-the-build-pipeline"></a>Creación de la canalización de compilación

1. Inicie sesión en Azure DevOps Services con una organización que pueda crear una canalización de compilación.

2. Vaya a la página **Build Web Application** (Compilar aplicación web) del proyecto.

3. En **Argumentos**, agregue el código **-r win10-x64**. Esto es necesario para desencadenar una implementación independiente con .Net Core.

    ![Adición del argumento de canalización de compilación](media/azure-stack-solution-hybrid-pipeline/020_publish_additions.png)

4. Ejecute la compilación. El proceso de [compilación de implementación autocontenida](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd) publicará los artefactos que se pueden ejecutar en Azure y Azure Stack.

### <a name="use-an-azure-hosted-build-agent"></a>Uso de un agente de compilación hospedado en Azure

El uso de un agente de compilación hospedado en Azure DevOps Services es una opción adecuada para compilar e implementar aplicaciones web. Microsoft Azure realiza automáticamente el mantenimiento y las actualizaciones, lo que permite un ciclo de desarrollo ininterrumpido y continuo.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configuración del proceso de implementación continua (CD)

Azure DevOps Services y Team Foundation Server (TFS) proporcionan una canalización con una gran capacidad de configuración y administración para versiones de varios entornos, como desarrollo, ensayo, control de calidad (QA) y producción. Este proceso puede incluir la necesidad de aprobaciones en determinadas fases del ciclo de vida de la aplicación.

### <a name="create-release-pipeline"></a>Creación de la canalización de versión

La creación de una canalización de versión es el último paso en el proceso de compilación de la aplicación. Esta canalización de versión se usa para crear una versión e implementar una compilación.

1. Inicie sesión en Azure DevOps Services y vaya a **Azure Pipelines** para el proyecto.
2. En la pestaña **Versiones**, seleccione **\[ + ]** y, a continuación, elija **Crear definición de versión**.

   ![Creación de la canalización de versión](media/azure-stack-solution-hybrid-pipeline/021a_releasedef.png)

3. En **Seleccionar una plantilla**, elija **Implementación de Azure App Service** y, a continuación, seleccione **Aplicar**.

    ![Aplicar plantilla](media/azure-stack-solution-hybrid-pipeline/102.png)

4. En **Agregar artefacto**, en el menú desplegable **Origen (definición de compilación)**, seleccione la aplicación de compilación en la nube de Azure.

    ![Agregar artefacto](media/azure-stack-solution-hybrid-pipeline/103.png)

5. En la pestaña **Canalización**, seleccione el vínculo **1 fase**, **1 tarea** para **Ver tareas de entorno**.

    ![Ver tareas de la canalización](media/azure-stack-solution-hybrid-pipeline/104.png)

6. En la pestaña **Tareas**, escriba Azure como el **Nombre del entorno** y seleccione AzureCloud Traders-Web EP en la lista desplegable **Suscripción de Azure**.

    ![Establecimiento de variables de entorno](media/azure-stack-solution-hybrid-pipeline/105.png)

7. Escriba el **nombre de la instancia de Azure App Service**, que es "northwindtraders" en la siguiente captura de pantalla.

    ![Nombre de App Service](media/azure-stack-solution-hybrid-pipeline/106.png)

8. Para la fase de agente, seleccione **VS2017 hospedado** en la lista desplegable **Cola de agentes**.

    ![Agente hospedado](media/azure-stack-solution-hybrid-pipeline/107.png)

9. En **Implementar Azure App Service**, seleccione el valor de **Paquete o carpeta** válido para el entorno.

    ![Selección del paquete o la carpeta](media/azure-stack-solution-hybrid-pipeline/108.png)

10. En **Seleccionar archivo o carpeta**, seleccione **Aceptar** para **Ubicación**.

    ![Texto alternativo](media/azure-stack-solution-hybrid-pipeline/109.png)

11. Guarde todos los cambios y vuelva a **Canalización**.

    ![Texto alternativo](media/azure-stack-solution-hybrid-pipeline/110.png)

12. En la pestaña **Canalización**, seleccione **Agregar artefacto** y elija **NorthwindCloud Traders-Vessel** en la lista desplegable **Origen (definición de compilación)**.

    ![Adición de un nuevo artefacto](media/azure-stack-solution-hybrid-pipeline/111.png)

13. En **Seleccionar una plantilla**, agregue otro entorno. Elija **Implementación de Azure App Service** y, a continuación, seleccione **Aplicar**.

    ![Seleccionar plantilla](media/azure-stack-solution-hybrid-pipeline/112.png)

14. Escriba "Azure Stack" como el **Nombre del entorno**.

    ![Nombre del entorno](media/azure-stack-solution-hybrid-pipeline/113.png)

15. En la pestaña **Tareas**, busque y seleccione Azure Stack.

    ![Entorno de Azure Stack](media/azure-stack-solution-hybrid-pipeline/114.png)

16. En la lista desplegable **Suscripción de Azure**, seleccione "AzureStack Traders-Vessel EP" para el punto de conexión de Azure Stack.

    ![Texto alternativo](media/azure-stack-solution-hybrid-pipeline/115.png)

17. Escriba el nombre de la aplicación web de Azure Stack como el **Nombre de App Service**.

    ![Nombre de App Service](media/azure-stack-solution-hybrid-pipeline/116.png)

18. En **Selección de agente**, elija "AzureStack - bDouglas Fir" en la lista desplegable **Cola de agentes**.

    ![Elegir agente](media/azure-stack-solution-hybrid-pipeline/117.png)

19. En **Implementar Azure App Service**, seleccione el valor de **Paquete o carpeta** válido para el entorno. En **Seleccionar archivo o carpeta**, seleccione **Aceptar** para la **Ubicación** de la carpeta.

    ![Selección del paquete o carpeta](media/azure-stack-solution-hybrid-pipeline/118.png)

    ![Aprobar ubicación](media/azure-stack-solution-hybrid-pipeline/119.png)

20. En la pestaña **Variable**, busque la variable llamada **VSTS_ARM_REST_IGNORE_SSL_ERRORS**. Establezca el valor de la variable en **true** y establezca su ámbito en **Azure Stack**.

    ![Configuración de variable](media/azure-stack-solution-hybrid-pipeline/120.png)

21. En la pestaña **Canalización**, seleccione el icono del **Desencadenador de implementación continua** para el artefacto NorthwindCloud Traders-Web y establezca **Desencadenador de implementación continua** en **Habilitado**.  Lo mismo para el artefacto "NorthwindCloud Traders-Vessel".

    ![Establecimiento del desencadenador de implementación continua](media/azure-stack-solution-hybrid-pipeline/121.png)

22. En el entorno de Azure Stack, seleccione el icono de **Condiciones anteriores a la implementación** y establezca el desencadenador en **Después de la versión**.

    ![Establecimiento del desencadenador de las condiciones anteriores a la implementación](media/azure-stack-solution-hybrid-pipeline/122.png)

23. Guarde todos los cambios.

> [!Note]
> Algunos valores de configuración de las tareas de la versión se pueden haber definido automáticamente como [variables de entorno](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts#custom-variables) cuando se creó una canalización de versión desde una plantilla. No se puede modificar esta configuración en la configuración de la tarea. Sin embargo, puede editar estos valores en los elementos del entorno primario.

## <a name="create-a-release"></a>Creación de una versión

Ahora que ha completado las modificaciones a la canalización de versión, es hora de iniciar la implementación. Para ello, cree una versión a partir de la canalización de versión. Una versión se puede crear automáticamente; por ejemplo, el desencadenador de implementación continua se establece en la canalización de versión. Esto significa que al modificar el código fuente se iniciará una nueva compilación y, a partir de ella, una nueva versión. Sin embargo, en esta sección creará una nueva versión manualmente.

1. En la pestaña **Canalización**, abra la desplegable lista **Versión** y elija **Crear versión**.

    ![Creación de una versión](media/azure-stack-solution-hybrid-pipeline/200.png)

2. Escriba una descripción para la versión, compruebe que se seleccionan los artefactos correctos y, a continuación, elija **Crear**. Transcurridos unos instantes, aparece un mensaje emergente que indica que se ha creado la nueva versión y el nombre de la versión se muestra como un vínculo. Elija el vínculo para ver la página de resumen de la versión.

    ![Mensaje emergente de creación de la versión](media/azure-stack-solution-hybrid-pipeline/201.png)

3. La página de resumen de la versión muestra detalles acerca de la versión. En la siguiente captura de pantalla de "Release-2", la sección **Entornos** muestra el **Estado de implementación** para Azure como "En curso" y el estado para Azure Stack como "Correcto". Cuando el estado de implementación para el entorno de Azure cambie a "Correcto", aparece un mensaje emergente que indica que la versión está lista para su aprobación. Cuando una implementación está pendiente o se ha producido un error, se muestra un icono de información **(i)** azul. Desplácese sobre el icono para ver un elemento emergente que contiene la razón del retraso o los errores.

    ![Página de resumen de la versión](media/azure-stack-solution-hybrid-pipeline/202.png)

Otras vistas, como la lista de versiones, también muestran un icono que indica que la aprobación está pendiente. El elemento emergente para este icono muestra el nombre del entorno y más detalles relacionados con la implementación. Es fácil para un administrador consultar el progreso general de las versiones y ver qué versiones están pendientes de aprobación.

### <a name="monitor-and-track-deployments"></a>Supervisión y seguimiento de implementaciones

En esta sección se muestra cómo puede supervisar y realizar un seguimiento de todas las implementaciones. La versión para la implementación de dos sitios web de Azure App Service proporciona un buen ejemplo.

1. En la página de resumen de "Release-2", seleccione **Registros**. Durante una implementación, esta página muestra el registro en directo del agente. El panel izquierdo muestra el estado de cada operación de la implementación para cada entorno.

    Elija el icono de persona en la columna **Acción** correspondiente a una aprobación previa o posterior a la implementación para ver quién aprobó (o rechazó) la implementación y el mensaje que proporcionó.

2. Una vez finalizada la implementación, se muestra el archivo de registro completo en el panel derecho. Puede seleccionar cualquier **Paso** en el panel izquierdo para ver el archivo de registro de un único paso, como "Initialize job" (Inicializar trabajo). La posibilidad de ver los registros individuales facilita realizar el seguimiento y la depuración de partes individuales de la implementación general. También puede **Guardar** el archivo de registro de un paso o **Descargar todos los registros como zip**.

    ![Registros de versión](media/azure-stack-solution-hybrid-pipeline/203.png)

3. Abra la pestaña **Resumen** para ver información general sobre la versión. Esta vista muestra detalles de la compilación, los entornos en los que se implementó, el estado de implementación y otra información sobre la versión.

4. Seleccione un vínculo de entorno (**Azure** o **Azure Stack**) para ver información acerca de las implementaciones existentes y las pendientes en un entorno específico. Puede usar estas vistas como un modo rápido para comprobar que la misma compilación se implementó en ambos entornos.

5. Abra la **aplicación de producción implementada** en el explorador. Por ejemplo, para el sitio web de Azure App Services, abra la dirección URL `http://[your-app-name].azurewebsites.net`.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](https://docs.microsoft.com/azure/architecture/patterns).
