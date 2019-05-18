---
title: 'Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Workday.
services: active-directory
author: cmmdesai
documentationcenter: na
manager: daveba
ms.assetid: 1a2c375a-1bb1-4a61-8115-5a69972c6ad6
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/16/2019
ms.author: chmutali
ms.collection: M365-identity-device-management
ms.openlocfilehash: 31cf1f6da515aa9b453987383e78f466c5ba4fb9
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827289"
---
# <a name="tutorial-configure-workday-for-automatic-user-provisioning"></a>Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que hay que realizar para importar perfiles de trabajo desde Workday a Active Directory y Azure Active Directory, con reescritura opcional de la dirección de correo electrónico y nombre de usuario de Workday.

## <a name="overview"></a>Información general

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../manage-apps/user-provisioning.md) se integra con la [API de recursos humanos de Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) para poder aprovisionar cuentas de usuario. Azure AD usa esta conexión para habilitar los siguientes flujos de trabajo de aprovisionamiento de usuarios:

* **Aprovisionamiento de usuarios en Active Directory**: aprovisione conjuntos de usuarios seleccionados de Workday en uno o varios dominios de Active Directory.

* **Aprovisionamiento de usuarios solo de nube a Azure Active Directory**: en escenarios en los que no se utiliza Active Directory local, los usuarios pueden aprovisionarse directamente desde Workday a Azure Active Directory mediante el servicio de aprovisionamiento de usuarios de Azure AD.

* **Escribir nombre de usuario y la dirección de correo electrónico en Workday** -el servicio de aprovisionamiento de usuario de Azure AD puede reescribir las direcciones de correo electrónico y nombre de usuario de Azure AD con Workday.

### <a name="what-human-resources-scenarios-does-it-cover"></a>¿Qué escenarios de recursos humanos se tratan?

El flujo de trabajo de aprovisionamiento de usuarios de Workday compatible con el servicio de aprovisionamiento de usuarios de Azure AD permite la automatización de los siguientes escenarios de recursos humanos y de administración del ciclo de vida de identidades:

* **Contratación de nuevos empleados**: cuando se agrega un nuevo empleado a Workday, se crea automáticamente una cuenta de usuario en Active Directory, Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../manage-apps/user-provisioning.md), con reescritura de la dirección de correo electrónico en Workday.

* **Actualizaciones de atributos y de perfil de los empleados**: si se actualiza un registro de empleado en Workday (por ejemplo, el nombre, el cargo o el administrador), su cuenta de usuario se actualizará automáticamente en Active Directory, Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../manage-apps/user-provisioning.md).

* **Ceses de empleados**: cuando se prescinde de un empleado en Workday, su cuenta de usuario se deshabilita automáticamente de Active Directory, Azure Active Directory y, opcionalmente, de Office 365 y de [otras aplicaciones SaaS compatibles con Azure AD](../manage-apps/user-provisioning.md).

* **Recontrataciones de empleados**: cuando se vuelve a contratar a un empleado en Workday, se puede reactivar o volver a aprovisionar automáticamente su cuenta antigua (en función de sus preferencias) en Active Directory, Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../manage-apps/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

Esta solución de aprovisionamiento de usuario de Workday es ideal para:

* Organizaciones que desean una solución basada en la nube precompilada para el aprovisionamiento de usuarios de Workday

* Organizaciones que necesitan aprovisionamiento de usuarios directo de Workday a Active Directory, o a Azure Active Directory

* Organizaciones que necesitan aprovisionamiento de usuarios mediante los datos obtenidos del módulo HCM de Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* Organizaciones que necesitan combinar y mover usuarios y dejarles que se sincronicen con uno o varios bosques, dominios o unidades organizativas de Active Directory en función de únicamente la información de cambio detectada en el módulo HCM de Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* Organizaciones que usan Office 365 para el correo electrónico

## <a name="solution-architecture"></a>Arquitectura de la solución

En esta sección se describe la arquitectura de la solución de aprovisionamiento de usuarios de un extremo a otro para entornos híbridos comunes. Hay dos flujos relacionados:

* **Flujo de datos de RR.HH. relevante, desde Workday a Active Directory local:** en este flujo de trabajo (por ejemplo, nuevas contrataciones, transferencias o finalizaciones), los eventos se producen en primer lugar en el inquilino de la nube de RR.HH. de Workday y, a continuación, los datos de eventos fluyen hacia Active Directory local mediante Azure AD y el agente de aprovisionamiento. En función del evento, puede dar lugar a las operaciones de creación, actualización, habilitación o deshabilitación en AD.
* **Correo electrónico y la escritura diferida de nombre de usuario de flujo: de local de Active Directory a Workday:** Una vez completada la creación de cuentas en Active Directory, se sincronizan con Azure AD a través de Azure AD Connect y atributo de correo electrónico y nombre de usuario puede escribirse en Workday.

![Información general](./media/workday-inbound-tutorial/wd_overview.png)

### <a name="end-to-end-user-data-flow"></a>Flujo de datos de usuarios de un extremo a otro

1. El equipo de RR.HH. realiza las transacciones de trabajo (empleados que se incorporan a la empresa, se trasladan o la se van, o nuevas contrataciones, transferencias o terminaciones de contratos) en Workday HCM
2. El servicio de aprovisionamiento de Azure AD ejecuta las sincronizaciones programadas de las identidades de RR.HH. de Workday e identifica los cambios que deben procesarse para la sincronización con Active Directory local.
3. El servicio de aprovisionamiento de Azure AD invoca al agente de aprovisionamiento de AAD Connect local con una carga de solicitud que contiene las operaciones de creación, actualización, habilitación o deshabilitación de las cuentas de AD.
4. El agente de aprovisionamiento de Azure AD Connect usa una cuenta de servicio para agregar o actualizar los datos de la cuenta de AD.
5. El motor de Azure AD Connect / AD Sync ejecuta una sincronización diferencial para extraer las actualizaciones de AD.
6. Las actualizaciones de Active Directory se sincronizan con Azure Active Directory.
7. Si se configura el conector de reescritura de Workday, escribe nuevo atributo de correo electrónico y nombre de usuario en Workday, según el atributo coincidente que se usa.

## <a name="planning-your-deployment"></a>Planeamiento de la implementación

Antes de iniciar la integración de Workday, compruebe los requisitos previos siguientes y lea las siguientes instrucciones sobre cómo ajustar su arquitectura actual de Active Directory y los requisitos de aprovisionamiento de usuarios con las soluciones proporcionadas por Azure Active Directory. También se encuentra disponible un [plan de implementación](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-deployment-plans) completo con hojas de cálculo de planeamiento para facilitar la colaboración con las partes interesadas de recursos humanos y el asociado de integración de Workday.

En esta sección se tratan los siguientes aspectos de planeamiento:

* [Requisitos previos](#prerequisites)
* [Selección de aplicaciones de conectores de aprovisionamiento que implementar](#selecting-provisioning-connector-apps-to-deploy)
* [Planeamiento de la implementación del agente de aprovisionamiento de Azure AD Connect](#planning-deployment-of-azure-ad-connect-provisioning-agent)
* [Integración con varios dominios de Active Directory](#integrating-with-multiple-active-directory-domains)
* [Planeamiento de Workday para las transformaciones y la asignación de atributos de usuario de Active Directory](#planning-workday-to-active-directory-user-attribute-mapping-and-transformations)

### <a name="prerequisites"></a>Requisitos previos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Una suscripción válida a Azure AD Premium P1 o posterior con acceso de administrador global
* Un inquilino de implementación de Workday para fines de pruebas y de integración
* Permisos de administrador en Workday para crear un usuario de integración del sistema y efectuar cambios para probar los datos de los empleados con fines de pruebas
* Para el aprovisionamiento de usuarios en Active Directory, se necesita un servidor que ejecute Windows Server 2012 o una versión posterior con el entorno de ejecución .NET 4.7.1+ a fin de hospedar el [agente de aprovisionamiento local](https://go.microsoft.com/fwlink/?linkid=847801)
* [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) para la sincronización de usuarios entre Active Directory y Azure AD.

### <a name="selecting-provisioning-connector-apps-to-deploy"></a>Selección de aplicaciones de conectores de aprovisionamiento que implementar

Para facilitar el aprovisionamiento de flujos de trabajo entre Workday y Active Directory, Azure AD proporciona varias aplicaciones de conectores de aprovisionamiento que puede agregar desde la galería de aplicaciones de Azure AD:

![Galería de aplicaciones de AAD](./media/workday-inbound-tutorial/wd_gallery.png)

* **Aprovisionamiento de usuarios de Workday en Active Directory**: esta aplicación facilita el aprovisionamiento de cuentas de usuario desde Workday a un dominio de Active Directory. Si tiene varios dominios, puede agregar una instancia de esta aplicación desde la galería de aplicaciones de Azure AD para cada dominio de Active Directory en el que deba efectuar un aprovisionamiento.

* **Aprovisionamiento de usuarios de Workday en Azure AD**: mientras AAD Connect es la herramienta que se debe usar para sincronizar los usuarios de Active Directory con Azure Active Directory, esta aplicación se puede usar para facilitar el aprovisionamiento de usuarios que solo están en la nube desde Workday a un inquilino de Azure Active Directory.

* **Escritura diferida de Workday**: esta aplicación facilita la escritura diferida de direcciones de correo electrónico del usuario de Azure Active Directory a Workday.

> [!TIP]
> La aplicación "Workday" normal se usa para configurar el inicio de sesión único entre Workday y Azure Active Directory.

Use el siguiente diagrama de decisiones para identificar qué aplicaciones de aprovisionamiento de Workday son pertinentes para su escenario.
    ![Diagrama de flujo de decisiones](./media/workday-inbound-tutorial/wday_app_flowchart.png "Diagrama de flujo de decisiones")

Utilice la tabla de contenido para ir a la sección correspondiente de este tutorial.

### <a name="planning-deployment-of-azure-ad-connect-provisioning-agent"></a>Planeamiento de la implementación del agente de aprovisionamiento de Azure AD Connect

> [!NOTE]
> Esta sección solo es pertinente si tiene previsto implementar Workday en la aplicación de aprovisionamiento de usuarios de Active Directory. Puede omitir esto si va a implementar la escritura diferida de Workday o Workday en la aplicación de aprovisionamiento de usuarios de Azure AD.

La solución de aprovisionamiento de usuarios para AD de Workday requiere la implementación de uno o varios agentes de aprovisionamiento en servidores que ejecutan Windows 2012 R2 o posterior con, al menos, 4 GB de RAM y .NET 4.7.1+ en tiempo de ejecución. Las siguientes consideraciones deben tenerse en cuenta antes de instalar el agente de aprovisionamiento:

* Asegúrese de que el servidor host que ejecuta el agente de aprovisionamiento tiene acceso de red para el dominio de AD de destino.
* El Asistente para configuración del agente de aprovisionamiento registra el agente con el inquilino de Azure AD y el proceso de registro requiere acceso a *.msappproxy.net a través del puerto SSL 443. Asegúrese de que las reglas de firewall de salida que permiten esta comunicación están en vigor. El agente admite la [configuración de proxy HTTPS saliente](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication).
* El agente de aprovisionamiento usa una cuenta de servicio para comunicarse con los dominios de AD de la red local. Antes de la instalación del agente, se recomienda que cree una cuenta de servicio con permisos de administrador de dominio y una contraseña que no expire.  
* Durante la configuración del agente de aprovisionamiento, puede seleccionar los controladores de dominio que deben controlar las solicitudes de aprovisionamiento. Si tiene varios controladores de dominio distribuidos geográficamente, instale al agente de aprovisionamiento en el mismo sitio que los controladores de dominio preferidos para mejorar la confiabilidad y el rendimiento de la solución de un extremo a otro.
* Para lograr una alta disponibilidad, puede implementar más de un agente de aprovisionamiento y registrarlos para controlar el mismo conjunto de dominios de Active Directory locales.

> [!IMPORTANT]
> En entornos de producción, Microsoft recomienda que tenga un mínimo de tres agentes de aprovisionamiento configurados con el inquilino de Azure AD para lograr una alta disponibilidad.

### <a name="integrating-with-multiple-active-directory-domains"></a>Integración con varios dominios de Active Directory

> [!NOTE]
> Esta sección solo es pertinente si tiene previsto implementar Workday en la aplicación de aprovisionamiento de usuarios de Active Directory. Puede omitir esto si va a implementar la escritura diferida de Workday o Workday en la aplicación de aprovisionamiento de usuarios de Azure AD.

Según la topología de Active Directory, deberá decidir el número de aplicaciones de conector de aprovisionamiento de usuarios y el número de agentes de aprovisionamiento para configurarlos. A continuación, se presentan algunos de los patrones comunes de implementación que puede consultar mientras planea la implementación.

#### <a name="deployment-scenario-1--single-workday-tenant---single-ad-domain"></a>Escenario de implementación 1: Un solo inquilino de Workday -> Dominio de AD único

En este escenario, tiene un inquilino de Workday y le gustaría aprovisionar usuarios en un único dominio de AD de destino. Esta es la configuración de producción recomendada para esta implementación.

|   |   |
| - | - |
| No. de agentes de aprovisionamiento para implementar de forma local | 3 (para alta disponibilidad y conmutación por error) |
| No. de aplicaciones de aprovisionamiento de usuarios de Workday en AD para configurar en Azure Portal | 1 |

  ![Escenario 1.](./media/workday-inbound-tutorial/dep_scenario1.png)

#### <a name="deployment-scenario-2--single-workday-tenant---multiple-child-ad-domains"></a>Escenario de implementación 2: Un solo inquilino de Workday -> Varios dominios secundarios de AD

Este escenario implica el aprovisionamiento de usuarios desde Workday en varios dominios secundarios de AD de destino en un bosque. Esta es la configuración de producción recomendada para esta implementación.

|   |   |
| - | - |
| No. de agentes de aprovisionamiento para implementar de forma local | 3 (para alta disponibilidad y conmutación por error) |
| No. de aplicaciones de aprovisionamiento de usuarios de Workday en AD para configurar en Azure Portal | una aplicación por dominio secundario |

  ![Escenario 2.](./media/workday-inbound-tutorial/dep_scenario2.png)

#### <a name="deployment-scenario-3--single-workday-tenant---disjoint-ad-forests"></a>Escenario de implementación 3: Un solo inquilino de Workday -> Bosques de AD no contiguos

Este escenario implica el aprovisionamiento de usuarios desde Workday en dominios de bosques de AD no contiguos. Esta es la configuración de producción recomendada para esta implementación.

|   |   |
| - | - |
| No. de agentes de aprovisionamiento para implementar de forma local | 3 por cada bosque de AD no contiguo |
| No. de aplicaciones de aprovisionamiento de usuarios de Workday en AD para configurar en Azure Portal | una aplicación por dominio secundario |

  ![Escenario 3.](./media/workday-inbound-tutorial/dep_scenario3.png)

### <a name="planning-workday-to-active-directory-user-attribute-mapping-and-transformations"></a>Planeamiento de Workday para las transformaciones y la asignación de atributos de usuario de Active Directory

> [!NOTE]
> Esta sección solo es pertinente si tiene previsto implementar Workday en la aplicación de aprovisionamiento de usuarios de Active Directory. Puede omitir esto si va a implementar la escritura diferida de Workday o Workday en la aplicación de aprovisionamiento de usuarios de Azure AD.

Antes de configurar el aprovisionamiento de usuarios en un dominio de Active Directory, tenga en cuenta las siguientes preguntas. Las respuestas a estas preguntas determinarán cómo deben establecerse los filtros de ámbito y las asignaciones de atributos.

* **¿Qué usuarios de Workday se deben aprovisionar en este bosque de Active Directory?**

  * *Ejemplo: usuarios donde el atributo "Company" de Workday contenga el valor "Contoso" y el atributo "Worker_Type" contenga "Regular"*

* **¿Cómo se enrutan los usuarios a diferentes unidades organizativas (OU)?**

  * *Ejemplo: los usuarios se enrutan a las unidades organizativas que corresponden a una ubicación de oficina, como se define en los atributos "Municipality" y "Country_Region_Reference" de Workday*

* **¿Cómo se deben rellenar los atributos siguientes en Active Directory?**

  * Nombre común (cn)
    * *Ejemplo: use el valor User_ID de Workday establecido por Recursos Humanos*

  * Identificador de empleado (employeeId)
    * *Ejemplo: use el valor Worker_ID de Workday*

  * Nombre de cuenta SAM (sAMAccountName)
    * *Ejemplo: use el valor User_ID de Workday, que se filtra mediante una expresión de aprovisionamiento de Azure AD para quitar caracteres incorrectos*

  * Nombre principal de usuario (userPrincipalName)
    * *Ejemplo: use el valor User_ID de Workday con una expresión de aprovisionamiento de Azure AD para anexar un nombre de dominio*

* **¿Cómo se hacen coincidir los usuarios de Workday y Active Directory?**

  * *Ejemplo: los usuarios con un valor "Worker_ID" específico de Workday se hacen coincidir con los usuarios de Active Directory donde "employeeID" tenga el mismo valor. Si el valor Worker_ID no se encuentra en Active Directory, cree un nuevo usuario.*
  
* **¿El bosque de Active Directory ya contiene los identificadores de usuario necesarios para que funcione la lógica de coincidencia?**

  * *Ejemplo: Si esta configuración es una nueva implementación de Workday, se recomienda rellenar previamente Active Directory con los valores Worker_ID correctos de Workday (o un valor de identificación único de su elección) para mantener la lógica de coincidencia lo más simple posible.*

El modo de instalación y configuración de estas aplicaciones de conectores de aprovisionamiento especiales es el tema de las secciones restantes de este tutorial. Las aplicaciones que elija para la configuración dependerán de los sistemas que deba aprovisionar y de la cantidad de dominios de Active Directory y de inquilinos de Azure AD que haya en su entorno.

## <a name="configure-integration-system-user-in-workday"></a>Configuración de un usuario de integración en Workday

Un requisito común de todos los conectores de aprovisionamiento de Workday es que requieren credenciales de un usuario del sistema de integración de Workday para poder conectarse a la API de recursos humanos de Workday. En esta sección se describe cómo crear un usuario del sistema de integración de Workday y tiene las siguientes secciones:

* [Creación de un usuario del sistema de integración](#creating-an-integration-system-user)
* [Creación de un grupo de seguridad de integración](#creating-an-integration-security-group)
* [Configuración de permisos de directiva de seguridad de dominio](#configuring-domain-security-policy-permissions)
* [Configuración de los permisos de directiva de seguridad del proceso de negocio](#configuring-business-process-security-policy-permissions)
* [Activación de cambios en directiva de seguridad](#activating-security-policy-changes)

> [!NOTE]
> Se puede omitir este procedimiento y usar una cuenta de administrador global de Workday como cuenta de integración de sistemas. Puede funcionar correctamente en las demostraciones, pero no se recomienda en las implementaciones de producción.

### <a name="creating-an-integration-system-user"></a>Creación de un usuario del sistema de integración

**Para crear un usuario del sistema de integración:**

1. Inicie sesión en su inquilino de Workday usando una cuenta de administrador. En **Workday Aplication** (Aplicación de Workday), escriba create user (crear usuario) en el cuadro de búsqueda y haga clic en **Create Integration System User** (Crear usuario del sistema de integración).

    ![Creación de usuarios](./media/workday-inbound-tutorial/wd_isu_01.png "Creación de usuarios")
2. Para completar la tarea **Create Integration System User** (Crear usuario del sistema de integración), especifique el nombre de usuario y la contraseña del nuevo usuario del sistema de integración.  
  
* Deje la opción **Require New Password at Next Sign In** (Solicitar una nueva contraseña en el siguiente inicio de sesión) sin activar, ya que este usuario iniciará sesión mediante programación.
* Deje **Session Timeout Minutes** (Minutos de tiempo de espera de la sesión) en 0 (su valor predeterminado), con el fin de evitar que las sesiones del usuario agoten el tiempo de espera de manera prematura.
* Seleccione la opción **Do Not Allow UI Sessions** (No permitir sesiones de la interfaz de usuario) ya que proporciona una capa adicional de seguridad que impide que un usuario con la contraseña del sistema de integración inicie sesión en Workday.

    ![Crear usuario de sistema de integración](./media/workday-inbound-tutorial/wd_isu_02.png "Crear usuario de sistema de integración")

### <a name="creating-an-integration-security-group"></a>Creación de un grupo de seguridad de integración

En este paso, creará un grupo de seguridad del sistema de integración sin o con restricciones en Workday y asignará a este grupo el usuario del sistema de integración creado en el paso anterior.

**Para crear un grupo de seguridad:**

1. Escriba create security group (crear grupo de seguridad) en el cuadro de búsqueda y luego haga clic en **Create Security Group**(Crear grupo de seguridad).

    ![Crear grupo de seguridad](./media/workday-inbound-tutorial/wd_isu_03.png "Crear grupo de seguridad")
2. Complete la tarea **Create Security Group** (Crear grupo de seguridad). 

   * Hay dos tipos de grupos de seguridad en Workday:
     * **Sin restricciones**: todos los miembros del grupo de seguridad pueden tener acceso a todas las instancias de datos protegidas por el grupo de seguridad.
     * **Con restricciones**: todos los miembros del grupo de seguridad tienen acceso contextual a un subconjunto de las instancias de datos (filas) al que el grupo de seguridad puede acceder.
   * Consulte con su asociado de integración de Workday cuál es el tipo de grupo de seguridad apropiado que debe seleccionar para la integración.
   * Cuando sepa el tipo de grupo, seleccione **Integration System Security Group (Unconstrained)** (Grupo de seguridad del sistema de integración, sin restricciones) o **Integration System Security Group (Constrained)** (Grupo de seguridad del sistema de integración, con restricciones) en el menú desplegable **Type of Tenanted Security Group** (Tipo de grupo de seguridad con inquilinos).

     ![Crear grupo de seguridad](./media/workday-inbound-tutorial/wd_isu_04.png "Crear grupo de seguridad")

3. Una vez que se realice correctamente la creación del grupo de seguridad, verá una página donde puede asignar miembros al grupo de seguridad. Agregue a este grupo de seguridad el usuario del sistema de integración creado en el paso anterior. Si usa el grupo de seguridad *con restricciones*, también deberá seleccionar el ámbito apropiado de la organización.

    ![Editar grupo de seguridad](./media/workday-inbound-tutorial/wd_isu_05.png "Editar grupo de seguridad")

### <a name="configuring-domain-security-policy-permissions"></a>Configuración de permisos de directiva de seguridad de dominio

En ese paso, concederá al grupo de seguridad los permisos de directiva "seguridad de dominio" para los datos de los empleados.

**Para configurar permisos de directiva de seguridad de dominio:**

1. Escriba **Domain Security Configuration** (Configuración de seguridad de dominio) en el cuadro de búsqueda y, a continuación, haga clic en el vínculo **Domain Security Configuration Report** (Informe de configuración de seguridad de dominio).  

    ![Directivas de seguridad de dominio](./media/workday-inbound-tutorial/wd_isu_06.png "Directivas de seguridad de dominio")  
2. En el cuadro de texto **Dominio**, busque los siguientes dominios y agréguelos al filtro de uno en uno.  
   * *External Account Provisioning* (Aprovisionamiento de cuentas externas)
   * *Worker Data: Public Worker Reports* (Datos de empleado: informes de trabajadores públicos)
   * *Person Data: Work Contact Information* (Datos personales: información de contacto de trabajo)
   * *Worker Data: All Positions* (Datos de empleado: todos los cargos)
   * *Worker Data: Current Staffing Information* (Datos de empleado: información de plantilla actual)
   * *Worker Data: Business Title on Worker Profile* (Datos de empleado: cargo empresarial en el perfil del trabajador)
   * *Cuentas de WORKDAY*
   
     ![Directivas de seguridad de dominio](./media/workday-inbound-tutorial/wd_isu_07.png "Directivas de seguridad de dominio")  

     ![Directivas de seguridad de dominio](./media/workday-inbound-tutorial/wd_isu_08.png "Directivas de seguridad de dominio") 

     Haga clic en **OK**.

3. En el informe que se muestra, seleccione el botón de puntos suspensivos (...) que aparece junto a **External Account Provisioning** (Aprovisionamiento de cuentas externas) y haga clic en la opción de menú **Domain -> Edit Security Policy Permissions**  (Dominio -> Editar permisos de directiva de seguridad)

    ![Directivas de seguridad de dominio](./media/workday-inbound-tutorial/wd_isu_09.png "Directivas de seguridad de dominio")  

4. En la página **Edit Domain Security Policy Permissions** (Editar permisos de directiva de seguridad de dominio), desplácese hacia abajo hasta la sección **Integration Permissions** (Permisos de integración). Haga clic en el signo "+" para agregar el grupo de sistema de integración a la lista de grupos de seguridad con los permisos de integración **Get** (Obtención) y **Put** (Colocación).

    ![Editar permiso](./media/workday-inbound-tutorial/wd_isu_10.png "Editar permiso")  

5. Haga clic en el signo "+" para agregar el grupo de sistema de integración a la lista de grupos de seguridad con los permisos de integración **Get** (Obtención) y **Put** (Colocación).

    ![Editar permiso](./media/workday-inbound-tutorial/wd_isu_11.png "Editar permiso")  

6. Repita los pasos del 3 al 5 anteriores con cada una de las directivas de seguridad restantes:

   | Operación | Directiva de seguridad de dominio |
   | ---------- | ---------- |
   | Obtener y poner | Worker Data: Public Worker Reports (Datos de empleado: informes de trabajadores públicos) |
   | Obtener y poner | Person Data: Work Contact Information (Datos personales: información de contacto de trabajo) |
   | Obtener | Worker Data: All Positions (Datos de empleado: todos los cargos) |
   | Obtener | Worker Data: Current Staffing Information (Datos de empleado: información de plantilla actual) |
   | Obtener | Worker Data: Business Title on Worker Profile (Datos de empleado: cargo empresarial en el perfil del trabajador) |
   | Obtener y poner | Cuentas de WORKDAY |

### <a name="configuring-business-process-security-policy-permissions"></a>Configuración de los permisos de directiva de seguridad del proceso de negocio

En ese paso, concederá permisos de directiva "seguridad de proceso de negocio" al grupo de seguridad para los datos de trabajo. Este paso es necesario para configurar el conector de aplicaciones de escritura diferida de Workday.

**Para configurar los permisos de directiva de seguridad del proceso de negocio:**

1. Escriba **Business Process Policy** (Directiva de proceso de negocio) en el cuadro de búsqueda y, a continuación, haga clic en la tarea **Edit Business Process Security Policy** (Editar directiva de seguridad del proceso de negocio) del vínculo.  

    ![Directivas de seguridad del proceso de negocio](./media/workday-inbound-tutorial/wd_isu_12.png "Business Process Security Policies")  

2. En el cuadro de texto **Business Process Type** (Tipo de proceso de negocio), busque *póngase en contacto con* y seleccione el proceso de negocio **Contact Change** (Cambio de contacto) y haga clic en **OK** (Aceptar).

    ![Directivas de seguridad del proceso de negocio](./media/workday-inbound-tutorial/wd_isu_13.png "Business Process Security Policies")  

3. En la página **Edit Business Process Security Policy** (Editar directiva de seguridad del proceso de negocio), desplácese a la sección **Maintain Contact Information (Web Service)** [Mantener información de contacto (servicio web)].

    ![Directivas de seguridad del proceso de negocio](./media/workday-inbound-tutorial/wd_isu_14.png "Business Process Security Policies")  

4. Seleccione y agregue el nuevo grupo de seguridad del sistema de integración a la lista de grupos de seguridad que pueden iniciar la solicitud de servicios web. Haga clic en **Done** (Acabado). 

    ![Directivas de seguridad del proceso de negocio](./media/workday-inbound-tutorial/wd_isu_15.png "Business Process Security Policies")  

### <a name="activating-security-policy-changes"></a>Activación de cambios en directiva de seguridad

**Para activar cambios en directiva de seguridad:**

1. Escriba activate en el cuadro de búsqueda y, a continuación, haga clic en el vínculo **Activate Pending Security Policy Changes** (Activar cambios en la directiva de seguridad pendientes).

    ![Activar](./media/workday-inbound-tutorial/wd_isu_16.png "Activar")

1. Inicie la tarea Activate Pending Security Policy Changes (Activar cambios en la directiva de seguridad pendientes), para ello tiene que escribir un comentario para fines de auditoría y luego hacer clic en el botón **Aceptar**.
1. Complete la tarea que aparece en la pantalla siguiente; para ello, active la casilla etiquetada como **Confirm** (Confirmar) y luego haga clic en **OK** (Aceptar).

    ![Activar seguridad pendiente](./media/workday-inbound-tutorial/wd_isu_18.png "Activar seguridad pendiente")  

## <a name="configuring-user-provisioning-from-workday-to-active-directory"></a>Configuración del aprovisionamiento de usuarios de Workday a Active Directory

En esta sección se describen los pasos para el aprovisionamiento de cuentas de usuario desde Workday en cada dominio de Active Directory dentro del ámbito de su integración.

* [Instalación y configuración de agentes de aprovisionamiento en el entorno local](#part-1-install-and-configure-on-premises-provisioning-agents)
* [Adición de la aplicación del conector de aprovisionamiento y creación de la conexión con Workday](#part-2-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configuración de las asignaciones de atributos](#part-3-configure-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-install-and-configure-on-premises-provisioning-agents"></a>Parte 1: Instalación y configuración de agentes de aprovisionamiento en el entorno local

Para aprovisionar en Active Directory local, debe haber un agente instalado en un servidor que tenga .NET 4.7.1+ Framework y acceso de red al dominio o dominios de Active Directory deseados.

> [!TIP]
> Puede comprobar la versión de la plataforma de .NET en el servidor mediante las instrucciones proporcionadas [aquí](https://docs.microsoft.com/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed).
> Si el servidor no tiene instalado .NET 4.7.1 o posterior, puede descargarlo [aquí](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows).  

Una vez que ha implementado .NET 4.7.1+, puede descargar el **[agente de aprovisionamiento local aquí](https://go.microsoft.com/fwlink/?linkid=847801)** y seguir los pasos indicados a continuación para llevar a cabo la configuración del agente.

1. Inicie sesión en Windows Server donde desea instalar al nuevo agente.

1. Inicie el instalador del agente de aprovisionamiento, acepta los términos y haga clic en el **instalar** botón.

   ![Pantalla Instalar](./media/workday-inbound-tutorial/pa_install_screen_1.png "Install Screen")
   
1. Una vez completada la instalación, se iniciará el asistente y verá la pantalla **Conectar a Azure AD**. Haga clic en el botón **Autenticar** para conectarse a su instancia de Azure AD.

   ![Conectar a Azure AD](./media/workday-inbound-tutorial/pa_install_screen_2.png "Connect Azure AD")
   
1. Autentíquese en su instancia de Azure AD con las credenciales de administrador global.

   ![Autenticación de administrador](./media/workday-inbound-tutorial/pa_install_screen_3.png "Admin Auth")

   > [!NOTE]
   > Las credenciales de administrador de Azure AD se usan solo para conectarse al inquilino de Azure AD. El agente no almacena las credenciales localmente en el servidor.

1. Tras una autenticación correcta con Azure AD, verá la pantalla **Conectar a Active Directory**. En este paso, escriba su nombre de dominio de AD y haga clic en el botón **Agregar directorio**.

   ![Agregar directorio](./media/workday-inbound-tutorial/pa_install_screen_4.png "Add Directory")
  
1. Ahora se pedirá que escriba las credenciales necesarias para conectarse al dominio de AD. En la misma pantalla, puede usar la opción **Select domain controller priority** (Seleccionar prioridad del controlador de dominio) para especificar los controladores de dominio que el agente debe utilizar para enviar las solicitudes de aprovisionamiento.

   ![Credenciales del dominio](./media/workday-inbound-tutorial/pa_install_screen_5.png)
   
1. Después de configurar el dominio, el instalador muestra una lista de los dominios configurados. En esta pantalla, puede repetir los pasos 5 y 6 para agregar más dominios, o hacer clic en **Siguiente** para continuar con el registro del agente.

   ![Dominios configurados](./media/workday-inbound-tutorial/pa_install_screen_6.png "Configured Domains")

   > [!NOTE]
   > Si tiene varios dominios de AD (por ejemplo, na.contoso.com o emea.contoso.com), agregue cada dominio individualmente a la lista.
   > No basta solo con agregar el dominio primario (por ejemplo, contoso.com). Debe registrar cada dominio secundario en el agente.
   
1. Revise los detalles de configuración y haga clic en **Confirmar** para registrar el agente.
  
   ![Pantalla de confirmación](./media/workday-inbound-tutorial/pa_install_screen_7.png "Confirm Screen")
   
1. El Asistente para configuración muestra el progreso del registro del agente.
  
   ![Registro del agente](./media/workday-inbound-tutorial/pa_install_screen_8.png "Agent Registration")
   
1. Una vez que se realiza el registro del agente, puede hacer clic en **Salir** para salir del asistente.
  
   ![Pantalla Salir](./media/workday-inbound-tutorial/pa_install_screen_9.png "Exit Screen")
   
1. Compruebe la instalación del agente y asegúrese de que se está ejecutando; para ello, abra el complemento "Servicios" y busque el servicio denominado "Microsoft Azure AD Connect Provisioning Agent".
  
   ![Servicios](./media/workday-inbound-tutorial/services.png)

### <a name="part-2-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Parte 2: agregación de la aplicación del conector de aprovisionamiento y creación de la conexión con Workday

**Para configurar Workday en el aprovisionamiento de Active Directory:**

1. Vaya a <https://portal.azure.com>.

2. En la barra de navegación de la izquierda, seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Add an application** (Agregar una aplicación) y seleccione la categoría **All** (Todos).

5. Busque **Workday Provisioning to Active Directory** (Aprovisionamiento de Workday en Active Directory) y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **modo** **de aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Debe ser similar: **username\@nombre_inquilino**

   * **Contraseña de administrador**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de inquilino**: escriba la dirección URL al punto de conexión de servicios web de Workday de su inquilino. Debería ser similar a https://wd3-impl-services1.workday.com/ccx/service/contoso4, donde *contoso4* se reemplaza por el nombre correcto del inquilino y *wd3-impl* se reemplaza por la cadena de entorno correcta.

   * **Bosque de Active Directory**: el nombre de su dominio de Active Directory, según se ha registrado con el agente. Use la lista desplegable para seleccionar el dominio de destino para el aprovisionamiento. Este valor suele ser una cadena como: *contoso.com*.

   * **Contenedor de Active Directory:** escriba el DN del contenedor donde el agente debe crear cuentas de usuario de forma predeterminada.
        Ejemplo: *OU=Standard Users,OU=Users,DC=contoso,DC=test*
        
     > [!NOTE]
     > Esta configuración solo entra en juego para creaciones de cuentas de usuario si el atributo *parentDistinguishedName* no está configurado en las asignaciones de atributos. Esta configuración no se usa para la búsqueda de usuarios o las operaciones de actualización. El subárbol de todo el dominio se encuentra en el ámbito de la operación de búsqueda.

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Enviar una notificación por correo electrónico cuando se produzca un error".

     > [!NOTE]
     > El servicio de aprovisionamiento de Azure AD envía la notificación por correo electrónico si el trabajo de aprovisionamiento entra en un estado de[cuarentena](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning#quarantine).

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que las credenciales de Workday y las credenciales de AD configuradas en la instalación del agente sean válidas.

     ![Azure Portal](./media/workday-inbound-tutorial/wd_1.png)

   * Una vez que las credenciales se guardan correctamente, en la sección **Asignaciones** se mostrará la asignación predeterminada **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local).

### <a name="part-3-configure-attribute-mappings"></a>Parte 3: configuración de las asignaciones de atributos

En esta sección configurará cómo fluyen los datos de los usuarios de Workday a Active Directory.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local).

1. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de Workday que deben estar en el ámbito para el aprovisionamiento en AD; para ello, debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "Todos los usuarios de Workday". Filtros de ejemplo:

   * Ejemplo: Ámbito a los usuarios con el Id. de trabajador entre 1.000.000 y 2.000.000 (excepto 2000000)

      * Atributo: WorkerID

      * Operador: REGEX Match

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Ejemplo: solo los empleados, no los trabajadores temporales

      * Atributo: EmployeeID

      * Operador: NO ES NULO

   > [!TIP]
   > Al configurar la aplicación de aprovisionamiento por primera vez, deberá probar y verificar las expresiones y asignaciones de atributos para asegurarse de que ofrece el resultado deseado. Microsoft recomienda usar los filtros de ámbito en **Ámbito de objeto de origen** para probar las asignaciones con algunos usuarios de prueba de Workday. Una vez haya verificado que las asignaciones funcionan, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En el campo **Acciones del objeto de destino**, puede filtrar de forma global qué acciones se realizan en Active Directory. **Crear** y **Actualizar** son las más habituales.

1. En la sección **Asignaciones de atributos** puede definir cómo se asignan los distintos atributos de Workday a los atributos de Active Directory.

1. Haga clic en una asignación de atributos existente para actualizarla o haga clic en **Agregar nueva asignación** en la parte inferior de la pantalla para agregar asignaciones nuevas. Las asignaciones de atributos admiten estas propiedades:

      * **Tipo de asignación**

         * **Directo**: escribe el valor del atributo de Workday en el atributo de AD sin cambios.

         * **Constante**: escribe un valor de cadena estático y constante en el atributo de AD.

         * **Expresión**: le permite escribir un valor personalizado en el atributo de AD, en función de uno o varios atributos de Workday. [Para obtener más información, consulte este artículo sobre las expresiones](../manage-apps/functions-for-customizing-application-data.md).

      * **Atributo de origen**: el atributo de usuario de Workday. Si el atributo que está buscando no está presente, consulte [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

      * **Valor predeterminado**: opcional. Si el atributo de origen tiene un valor vacío, la asignación escribirá este valor.
            La configuración más habitual consiste en dejarlo en blanco.

      * **Atributo de destino**: atributo de usuario de Active Directory.

      * **Hacer coincidir objetos con este atributo**: especifica si se debe usar o no esta asignación para identificar de forma unívoca a los usuarios entre Workday y Active Directory. Este valor se suele establecer en el campo Id. del trabajador de Workday, que se suele asignar a uno de los atributos de id. de empleado de Active Directory.

      * **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Si hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

      * **Aplicar esta asignación**

         * **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios

         * **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios

1. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.

   ![Azure Portal](./media/workday-inbound-tutorial/wd_2.png)

#### <a name="below-are-some-example-attribute-mappings-between-workday-and-active-directory-with-some-common-expressions"></a>A continuación se muestran algunos ejemplos de asignaciones de atributos entre Workday y Active Directory, con algunas expresiones comunes.

* La expresión que se asigna al atributo *parentDistinguishedName* se usa para aprovisionar a los usuarios en distintas unidades organizativas en función de uno o varios atributos de origen de Workday. En este ejemplo se coloca a los usuarios en distintas unidades organizativas en función de la ciudad en la que se encuentran.

* El atributo *userPrincipalName* de Active Directory se genera mediante la función de desduplicación [SelectUniqueValue](../manage-apps/functions-for-customizing-application-data.md#selectuniquevalue) que comprueba la existencia de un valor generado en el dominio de AD de destino y solo se establece siempre que sea único.  

* [Aquí encontrará documentación sobre cómo escribir expresiones](../manage-apps/functions-for-customizing-application-data.md). Esta sección incluye ejemplos sobre cómo quitar caracteres especiales.

| ATRIBUTO DE WORKDAY | ATRIBUTO DE ACTIVE DIRECTORY |  ¿IDENTIFICADOR COINCIDENTE? | CREAR / ACTUALIZAR |
| ---------- | ---------- | ---------- | ---------- |
| **WorkerID**  |  EmployeeID | **Sí** | Escrito únicamente en Crear |
| **PreferredNameData**    |  cn    |   |   Escrito únicamente en Crear |
| **SelectUniqueValue (unir ("\@", únase a (".", \[FirstName\], \[LastName\]), "contoso.com"), unir ("\@", únase a (".", Mid (\[FirstName\], 1, 1 (), \[LastName\]), "contoso.com"), únase a ("\@", únase a (".", Mid (\[FirstName\], 1, 2), \[LastName\]), "contoso.com"))**   | userPrincipalName     |     | Escrito únicamente en Crear 
| **Replace(Mid(Replace(\[UserID\], , "(\[\\\\/\\\\\\\\\\\\\[\\\\\]\\\\:\\\\;\\\\\|\\\\=\\\\,\\\\+\\\\\*\\\\?\\\\&lt;\\\\&gt;\])", , "", , ), 1, 20), , "([\\\\.)\*\$](file:///\\.)*$)", , "", , )**      |    sAMAccountName            |     |         Escrito únicamente en Crear |
| **Switch(\[Active\], , "0", "True", "1", "False")** |  accountDisabled      |     | Crear y Actualizar |
| **FirstName**   | givenName       |     |    Crear y Actualizar |
| **Apellidos**   |   sn   |     |  Crear y Actualizar |
| **PreferredNameData**  |  displayName |     |   Crear y Actualizar |
| **Company**         | company   |     |  Crear y Actualizar |
| **SupervisoryOrganization**  | department  |     |  Crear y Actualizar |
| **ManagerReference**   | administrador  |     |  Crear y Actualizar |
| **BusinessTitle**   |  título     |     |  Crear y Actualizar | 
| **AddressLineData**    |  streetAddress  |     |   Crear y Actualizar |
| **Municipality**   |   l   |     | Crear y Actualizar |
| **CountryReferenceTwoLetter**      |   co |     |   Crear y Actualizar |
| **CountryReferenceTwoLetter**    |  c  |     |         Crear y Actualizar |
| **CountryRegionReference** |  st     |     | Crear y Actualizar |
| **WorkSpaceReference** | physicalDeliveryOfficeName    |     |  Crear y Actualizar |
| **PostalCode**  |   postalCode  |     | Crear y Actualizar |
| **PrimaryWorkTelephone**  |  telephoneNumber   |     | Crear y Actualizar |
| **Fax**      | facsimileTelephoneNumber     |     |    Crear y Actualizar |
| **Mobile**  |    mobile       |     |       Crear y Actualizar |
| **LocalReference** |  preferredLanguage  |     |  Crear y Actualizar |                                               
| **Switch(\[Municipality\], "OU=Standard Users,OU=Users,OU=Default,OU=Locations,DC=contoso,DC=com", "Dallas", "OU=Standard Users,OU=Users,OU=Dallas,OU=Locations,DC=contoso,DC=com", "Austin", "OU=Standard Users,OU=Users,OU=Austin,OU=Locations,DC=contoso,DC=com", "Seattle", "OU=Standard Users,OU=Users,OU=Seattle,OU=Locations,DC=contoso,DC=com", “London", "OU=Standard Users,OU=Users,OU=London,OU=Locations,DC=contoso,DC=com")**  | parentDistinguishedName     |     |  Crear y Actualizar |

Una vez completada la configuración de la asignación de atributos, ahora puede [habilitar e iniciar el servicio de aprovisionamiento de usuarios](#enable-and-launch-user-provisioning).

## <a name="configuring-user-provisioning-to-azure-ad"></a>Configuración del aprovisionamiento de usuarios en Azure AD

En las secciones siguientes se describen los pasos para configurar el aprovisionamiento de usuarios desde Workday en Azure AD para implementaciones exclusivas en la nube.

* [Adición de la aplicación del conector de aprovisionamiento de Azure AD y creación de la conexión con Workday](#part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configuración de asignaciones de atributos de Workday y Azure AD](#part-2-configure-workday-and-azure-ad-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

> [!IMPORTANT]
> Siga el siguiente procedimiento únicamente si tiene usuarios que solo están en la nube que se deban aprovisionar en Azure AD, y no en Active Directory local.

### <a name="part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Agregación de la aplicación del conector de aprovisionamiento de Azure AD y creación de la conexión con Workday

**Para configurar el aprovisionamiento de Workday en Azure Active Directory para los usuarios que solo están en la nube:**

1. Vaya a <https://portal.azure.com>.

2. En la barra de navegación de la izquierda, seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Agregar una aplicación** y, luego, seleccione la categoría **Todos**.

5. Busque **Workday to Azure AD provisioning** (Aprovisionamiento de Workday en Azure AD) y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **modo** **de aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Debe tener un aspecto similar al siguiente: username@contoso4

   * **Contraseña de administrador**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de inquilino**: escriba la dirección URL al punto de conexión de servicios web de Workday de su inquilino. Debería ser similar a https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources, donde *contoso4* se reemplaza por el nombre correcto del inquilino y *wd3-impl* se reemplaza por la cadena de entorno correcta. Si no se conoce esta dirección URL, trabaje con su asociado de integración o representante de soporte técnico de Workday para determinar la dirección URL correcta que se debe usar.

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Enviar una notificación por correo electrónico cuando se produzca un error".

   * Haga clic en el botón **Probar conexión**.

   * Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que la dirección URL y las credenciales de Workday sean válidas en Workday.

### <a name="part-2-configure-workday-and-azure-ad-attribute-mappings"></a>Parte 2: Configuración de asignaciones de atributos de Workday y Azure AD

En esta sección configurará cómo fluyen los datos de los usuarios de Workday a Azure Active Directory para los usuarios que solo están en la nube.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize Workers to Azure AD** (Sincronizar trabajadores con Azure AD).

2. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de Workday que deben estar en el ámbito para el aprovisionamiento en Azure AD; para ello debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "Todos los usuarios de Workday". Filtros de ejemplo:

   * Ejemplo: ámbito de los usuarios que tengan los id. de trabajador comprendidos entre 1000000 y 2000000

      * Atributo: WorkerID

      * Operador: REGEX Match

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Ejemplo: solo los trabajadores temporales, no los empleados habituales

      * Atributo: ContingentID

      * Operador: NO ES NULO

3. En el campo **Acciones del objeto de destino**, puede filtrar de forma global qué acciones se realizan en Azure AD. **Crear** y **Actualizar** son las más habituales.

4. En la sección **Asignaciones de atributos** puede definir cómo se asignan los distintos atributos de Workday a los atributos de Active Directory.

5. Haga clic en una asignación de atributos existente para actualizarla o haga clic en **Agregar nueva asignación** en la parte inferior de la pantalla para agregar asignaciones nuevas. Las asignaciones de atributos admiten estas propiedades:

   * **Tipo de asignación**

      * **Directo**: escribe el valor del atributo de Workday en el atributo de AD sin cambios.

      * **Constante**: escribe un valor de cadena estático y constante en el atributo de AD.

      * **Expresión**: le permite escribir un valor personalizado en el atributo de AD, en función de uno o varios atributos de Workday. [Para obtener más información, consulte este artículo sobre las expresiones](../manage-apps/functions-for-customizing-application-data.md).

   * **Atributo de origen**: el atributo de usuario de Workday. Si el atributo que está buscando no está presente, consulte [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

   * **Valor predeterminado**: opcional. Si el atributo de origen tiene un valor vacío, la asignación escribirá este valor.
            La configuración más habitual consiste en dejarlo en blanco.

   * **Atributo de destino**: atributo de usuario de Azure AD.

   * **Hacer coincidir objetos con este atributo**: especifica si se debe usar o no este atributo para identificar de forma unívoca a los usuarios entre Workday y Azure AD. Este valor se suele establecer en el campo Id. del trabajador de Workday, que se suele asignar al atributo de id. de empleado (nuevo) o a un atributo de extensión de Azure AD.

   * **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Si hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

   * **Aplicar esta asignación**

     * **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios

     * **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios

6. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.

Una vez completada la configuración de la asignación de atributos, ahora puede [habilitar e iniciar el servicio de aprovisionamiento de usuarios](#enable-and-launch-user-provisioning).

## <a name="configuring-azure-ad-attribute-writeback-to-workday"></a>Configurar la escritura diferida de atributos de Azure AD con Workday

Siga estas instrucciones para configurar la reescritura de direcciones de correo electrónico del usuario y el nombre de usuario de Azure Active Directory a Workday.

* [Adición de la aplicación del conector de escritura diferida y creación de la conexión con Workday](#part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday)
* [Configuración de las asignaciones de atributos de escritura diferida](#part-2-configure-writeback-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Adición de la aplicación del conector de escritura diferida y creación de la conexión con Workday

**Para configurar el conector de escritura diferida de Workday:**

1. Vaya a <https://portal.azure.com>.

2. En la barra de navegación de la izquierda, seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Agregar una aplicación** y, luego, seleccione la categoría **Todos**.

5. Busque **Workday Writeback** (Reescritura de Workday) y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **modo** **de aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Debe ser similar: *username\@contoso4*

   * **Contraseña de administrador**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de inquilino**: escriba la dirección URL al punto de conexión de servicios web de Workday de su inquilino. Este valor debería ser similar a https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources, donde *contoso4* se reemplaza por el nombre correcto del inquilino y *wd3-impl* se reemplaza por la cadena de entorno correcta (si es necesario).

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Enviar una notificación por correo electrónico cuando se produzca un error".

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que la dirección URL y las credenciales de Workday sean válidas en Workday.

### <a name="part-2-configure-writeback-attribute-mappings"></a>Parte 2: Configuración de las asignaciones de atributos de escritura diferida

En esta sección, va a configurar cómo los atributos de escritura diferida fluyen desde Azure AD a Workday. En este momento, el conector solo admite la reescritura de dirección de correo electrónico y nombre de usuario de Workday.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize Azure Active Directory Users to Workday** (Sincronizar usuarios de Azure Active Directory con Workday).

2. En el campo **Ámbito de objeto de origen** tiene la opción de filtrar los conjuntos de usuarios de Azure Active Directory que deben reescribir sus direcciones de correo electrónico en Workday. El ámbito predeterminado es "Todos los usuarios de Azure AD".

3. En la sección **Asignaciones de atributos**, actualice el identificador coincidente para indicar el atributo en Azure Active Directory donde se almacena el identificador de trabajo o el identificador de empleado de Workday. Un método de búsqueda de coincidencias conocido consiste en sincronizar el Id. de trabajador de Workday o el Id. de empleado con el atributo extensionAttribute1-15 en Azure AD y, luego, usar dicho atributo en Azure AD para hacer coincidir los usuarios en Workday.

4. Asigna normalmente Azure AD *userPrincipalName* atributo WORKDAY *UserID* atributo y el de Azure AD se asignan *correo* atributo para el día laborable  *Dirección de correo electrónico* atributo. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.

Una vez completada la configuración de la asignación de atributos, ahora puede [habilitar e iniciar el servicio de aprovisionamiento de usuarios](#enable-and-launch-user-provisioning).

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Una vez completadas las configuraciones de la aplicación de aprovisionamiento de Workday, puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos de Workday, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En la pestaña **Aprovisionamiento**, establezca **Estado de aprovisionamiento** en **Activado**.

2. Haga clic en **Save**(Guardar).

3. Esta operación iniciará la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el inquilino de Workday. 

4. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. Los registros de auditoría muestran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, por ejemplo, los usuarios que se leen fuera de Workday y que luego se agregan o actualizan en Active Directory. Consulte la sección Solución de problemas para obtener instrucciones sobre cómo revisar los registros de auditoría y corregir los errores de aprovisionamiento.

5. Una vez completada la sincronización inicial, se escribe un informe resumido de auditoría en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.

   ![Azure Portal](./media/workday-inbound-tutorial/wd_3.png)

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

* **Preguntas sobre la funcionalidad de la solución**
  * [Al procesar una nueva contratación en Workday, ¿cómo configura la solución la contraseña para la nueva cuenta de usuario en Active Directory?](#when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory)
  * [¿La solución admite el envío de notificaciones de correo electrónico una vez finalizadas las operaciones de aprovisionamiento?](#does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete)
  * [¿Cómo se puede administrar la entrega de contraseñas para los nuevos contratados y proporcionar un mecanismo para restablecer la contraseña de forma segura?](#how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password)
  * [¿La solución almacena en memoria caché los perfiles de usuario de Workday en la nube de Azure AD o en la capa del agente de aprovisionamiento?](#does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer)
  * [¿La solución permite asignar grupos de AD locales al usuario?](#does-the-solution-support-assigning-on-premises-ad-groups-to-the-user)
  * [¿Qué API de Workday usa la solución para consultar y actualizar los perfiles de trabajo de Workday?](#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles)
  * [¿Puede configurar mi inquilino de HCM en Workday con dos inquilinos de Azure AD?](#can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants)
  * [¿Por qué no es compatible la aplicación de aprovisionamiento de usuarios de "Workday en Azure AD" si hemos implementado Azure AD Connect?](#why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect)
  * [¿Cómo puedo sugerir mejoras o solicitar nuevas características relacionadas con la integración de Workday y Azure AD?](#how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration)

* **Preguntas sobre el agente de aprovisionamiento**
  * [¿Cuál es la versión de disponibilidad general del agente de aprovisionamiento?](#what-is-the-ga-version-of-the-provisioning-agent)
  * [¿Cómo puedo saber la versión de mi agente de aprovisionamiento?](#how-do-i-know-the-version-of-my-provisioning-agent)
  * [¿Microsoft inserta automáticamente las actualizaciones del agente de aprovisionamiento?](#does-microsoft-automatically-push-provisioning-agent-updates)
  * [¿Puedo instalar el agente de aprovisionamiento en el mismo servidor que ejecuta AAD Connect?](#can-i-install-the-provisioning-agent-on-the-same-server-running-aad-connect)
  * [¿Cómo se puede configurar el agente de aprovisionamiento para usar un servidor proxy para la comunicación HTTP saliente?](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication)
  * [¿Cómo se puede garantizar que el agente de aprovisionamiento sea capaz de comunicarse con el inquilino de Azure AD y que ningún firewall bloquee los puertos requeridos por el agente?](#how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent)
  * [¿Cómo puedo anular el registro del dominio asociado con mi agente de aprovisionamiento?](#how-do-i-de-register-the-domain-associated-with-my-provisioning-agent)
  * [¿Cómo puedo desinstalar el agente de aprovisionamiento?](#how-do-i-uninstall-the-provisioning-agent)
  
* **Preguntas sobre la asignación y configuración de atributos de Workday en AD**
  * [¿Cómo puedo realizar una copia de seguridad o exportar una copia de trabajo del esquema y la asignación de atributos de aprovisionamiento de Workday?](#how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema)
  * [Tengo atributos personalizados en Workday y Active Directory. ¿Cómo se puede configurar la solución para que funcione con los atributos personalizados?](#i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes)
  * [¿Puedo aprovisionar la foto de un usuario desde Workday en Active Directory?](#can-i-provision-users-photo-from-workday-to-active-directory)
  * [¿Cómo se pueden sincronizar los números de móvil desde Workday según el consentimiento del usuario para su uso público?](#how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage)
  * [¿Cómo puedo dar formato a los nombres para mostrar en AD según los atributos de departamento/país/ciudad del usuario y controlar las variaciones regionales?](#how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances)
  * [¿Cómo se puede usar SelectUniqueValue para generar valores únicos para el atributo samAccountName?](#how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute)
  * [¿Cómo se pueden quitar caracteres con signos diacríticos y convertirlos en caracteres habituales del alfabeto inglés?](#how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets)

### <a name="solution-capability-questions"></a>Preguntas sobre la funcionalidad de la solución

#### <a name="when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory"></a>Al procesar una nueva contratación en Workday, ¿cómo configura la solución la contraseña para la nueva cuenta de usuario en Active Directory?

Cuando el agente de aprovisionamiento local obtiene una solicitud para crear una cuenta de AD, genera automáticamente una contraseña aleatoria compleja diseñada para satisfacer los requisitos de complejidad de contraseña definidos por el servidor de AD y la define en el objeto de usuario. Esta contraseña no se registra en ningún lugar.

#### <a name="does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete"></a>¿La solución admite el envío de notificaciones de correo electrónico una vez finalizadas las operaciones de aprovisionamiento?

No, el envío de notificaciones por correo electrónico después de completar las operaciones de aprovisionamiento no se admite en la versión actual.

#### <a name="how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password"></a>¿Cómo se puede administrar la entrega de contraseñas para los nuevos contratados y proporcionar un mecanismo para restablecer la contraseña de forma segura?

Uno de los últimos pasos implicados en el nuevo aprovisionamiento de cuentas de AD es la entrega de la contraseña temporal asignada a la cuenta de AD del usuario. Muchas empresas todavía usan el enfoque tradicional de entrega de la contraseña temporal al administrador del usuario, quien, a continuación, la cede al nuevo trabajador contratado/eventual. Este proceso tiene un error de seguridad inherente y existe una opción para implementar un enfoque mejorado mediante funcionalidades de Azure AD.

Como parte del proceso de contratación, los equipos de recursos humanos normalmente ejecutan una comprobación en segundo plano y revisan el número de móvil del nuevo contratado. Con la integración del aprovisionamiento de usuarios de Workday en AD, puede basarse en este hecho y lanzar una funcionalidad de autoservicio de restablecimiento de contraseña para el usuario el primer día. Esto se realiza mediante la propagación del atributo "Número de móvil" del nuevo contratado de Workday a AD y, después, de AD a Azure AD con AAD Connect. Una vez que ya existe “Número de móvil” en Azure AD, puede habilitar [Autoservicio de restablecimiento de contraseña (SSPR)](../authentication/howto-sspr-authenticationdata.md) para la cuenta del usuario, para que, desde el primer día, un nuevo contratado pueda usar el número de móvil registrado y verificado para su autenticación.

#### <a name="does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer"></a>¿La solución almacena en memoria caché los perfiles de usuario de Workday en la nube de Azure AD o en la capa del agente de aprovisionamiento?

No, la solución no mantiene una caché de perfiles de usuario. El servicio de aprovisionamiento de Azure AD solo actúa como procesador de datos, de tal forma que lee los datos en Workday y los escribe en la instancia de destino de Active Directory o Azure AD. Consulte la sección [Administración de datos personales](#managing-personal-data) para obtener información relacionada con la retención de datos y la privacidad del usuario.

#### <a name="does-the-solution-support-assigning-on-premises-ad-groups-to-the-user"></a>¿La solución permite asignar grupos de AD locales al usuario?

Actualmente no se admite esta funcionalidad. La solución alternativa recomendada es implementar un script de PowerShell que realiza consultas en el punto de conexión de Azure AD Graph API para los datos de registro de auditoría y utilizar eso para desencadenar escenarios como la asignación de grupos. Este script de PowerShell puede anexarse a un programador de tareas e implementarse en el mismo equipo que ejecuta el agente de aprovisionamiento.  

#### <a name="which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles"></a>¿Qué API de Workday usa la solución para consultar y actualizar los perfiles de trabajo de Workday?

Actualmente, la solución utiliza las siguientes API de Workday:

* Get_Workers (v21.1) para capturar información de trabajo.
* Maintain_Contact_Information (v26.1) para la característica de escritura diferida de correo electrónico de trabajo.
* Update_Workday_Account (v31.2) para la función Username Writeback

#### <a name="can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants"></a>¿Puede configurar mi inquilino de HCM en Workday con dos inquilinos de Azure AD?

Sí, esta configuración es compatible. Estos son los pasos de alto nivel para configurar este escenario:

* Implemente el agente de aprovisionamiento 1 y regístrelo en el inquilino de Azure AD 1.
* Implemente el agente de aprovisionamiento 2 y regístrelo en el inquilino de Azure AD 2.
* En función de los "dominios secundarios" que cada agente de aprovisionamiento administrará, configure cada agente con los dominios. Un agente puede controlar varios dominios.
* En Azure Portal, configure la aplicación de aprovisionamiento de usuarios de Workday en AD en cada inquilino y configúrela con los dominios correspondientes.

#### <a name="why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect"></a>¿Por qué no es compatible la aplicación de aprovisionamiento de usuarios de "Workday en Azure AD" si hemos implementado Azure AD Connect?

Cuando se usa Azure AD en modo híbrido (donde contiene una combinación de usuarios en la nube + locales), es importante tener una definición clara del "origen de autoridad". Los escenarios híbridos típicos requieren la implementación de Azure AD Connect. Cuando se implementa Azure AD Connect, el entorno de AD local es el origen de autoridad. La introducción del conector de Workday a Azure AD en la combinación puede provocar una situación donde los valores de atributo de Workday podrían sobrescribir los valores establecidos por Azure AD Connect. Por tanto, el uso de la aplicación de aprovisionamiento "Workday en Azure AD" no es compatible si Azure AD Connect está habilitado. En tales situaciones, se recomienda usar la aplicación de aprovisionamiento de "usuarios de Workday en AD" para incorporar a los usuarios en AD local y luego sincronizarlos en Azure AD mediante Azure AD Connect.

#### <a name="how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration"></a>¿Cómo puedo sugerir mejoras o solicitar nuevas características relacionadas con la integración de Workday y Azure AD?

Sus comentarios son muy valiosos porque nos ayudan a definir la dirección de las futuras versiones y mejoras. Agradecemos todos los comentarios y le instamos a que envíe sus ideas o sugerencias de mejora en el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory). Para encontrar comentarios específicos relacionados con la integración de Workday, seleccione la categoría *Aplicaciones SaaS* y busque con la palabra clave *Workday* para encontrar comentarios existentes relativos a Workday.

![Aplicaciones SaaS para UserVoice](media/workday-inbound-tutorial/uservoice_saas_apps.png)

![Workday para UserVoice](media/workday-inbound-tutorial/uservoice_workday_feedback.png)

Al sugerir una nueva idea, compruebe si alguien ha sugerido ya una característica similar. En su caso, puede votar a favor de la solicitud de característica o mejora. También puede dejar un comentario sobre su caso de uso específico para mostrar su apoyo a la idea y demostrar lo importante que también será para usted la característica.

### <a name="provisioning-agent-questions"></a>Preguntas sobre el agente de aprovisionamiento

#### <a name="what-is-the-ga-version-of-the-provisioning-agent"></a>¿Cuál es la versión de disponibilidad general del agente de aprovisionamiento?

* La versión de disponibilidad general del agente de aprovisionamiento es la 1.1.30 y posterior.
* Si la versión del agente es inferior a la 1.1.30, ejecuta la versión preliminar pública y esta se actualizará automáticamente a la versión de disponibilidad general si el servidor que hospeda el agente tiene .NET 4.7.1 en tiempo de ejecución.
  * También puede [comprobar la versión de .NET](https://docs.microsoft.com/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) instalada en el servidor. Si el servidor no ejecuta .NET 4.7.1, puede [descargar e instalar .NET 4.7.1](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows). El agente de aprovisionamiento se actualizará automáticamente a la versión de disponibilidad general después de instalar .NET 4.7.1.

#### <a name="how-do-i-know-the-version-of-my-provisioning-agent"></a>¿Cómo puedo saber la versión de mi agente de aprovisionamiento?

* Inicie sesión en el servidor de Windows está instalado el agente de aprovisionamiento.
* Vaya al menú **Panel de control** -> **Desinstalar o cambiar un programa**.
* Busque la versión correspondiente a la entrada **Microsoft Azure AD Connect Provisioning Agent**.

  ![Azure Portal](./media/workday-inbound-tutorial/pa_version.png)

#### <a name="does-microsoft-automatically-push-provisioning-agent-updates"></a>¿Microsoft inserta automáticamente las actualizaciones del agente de aprovisionamiento?

Sí, Microsoft actualiza automáticamente el agente de aprovisionamiento. Puede deshabilitar las actualizaciones automáticas al detener el servicio de Windows **Agent Updater de Microsoft Azure AD Connect**.

#### <a name="can-i-install-the-provisioning-agent-on-the-same-server-running-aad-connect"></a>¿Puedo instalar el agente de aprovisionamiento en el mismo servidor que ejecuta AAD Connect?

Sí, puede instalar el agente de aprovisionamiento en el mismo servidor que ejecuta AAD Connect.

#### <a name="at-the-time-of-configuration-the-provisioning-agent-prompts-for-azure-ad-admin-credentials-does-the-agent-store-the-credentials-locally-on-the-server"></a>En el momento de la configuración, el agente de aprovisionamiento solicita las credenciales de administrador de Azure AD. ¿El agente almacena las credenciales localmente en el servidor?

Durante la configuración, el agente de aprovisionamiento solicita las credenciales de administrador de Azure AD solo para conectarse al inquilino de Azure AD. No almacena las credenciales localmente en el servidor. Sin embargo, conserva las credenciales usadas para conectarse al *dominio de Active Directory local* en un almacén de contraseñas local de Windows.

#### <a name="how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication"></a>¿Cómo se puede configurar el agente de aprovisionamiento para usar un servidor proxy para la comunicación HTTP saliente?

El agente de aprovisionamiento admite el uso de un proxy de salida. Puede configurarlo al editar el archivo de configuración del agente **C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config**. Agregue al archivo las siguientes líneas, hacia el final del archivo, justo antes de la etiqueta de cierre `</configuration>`.
Reemplace las variables [proxy-server] y [proxy-port] con los valores del puerto y el nombre del servidor proxy.

```xml
    <system.net>
          <defaultProxy enabled="true" useDefaultCredentials="true">
             <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
             />
         </defaultProxy>
    </system.net>
```

#### <a name="how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent"></a>¿Cómo se puede garantizar que el agente de aprovisionamiento sea capaz de comunicarse con el inquilino de Azure AD y que ningún firewall bloquee los puertos requeridos por el agente?

También puede comprobar si tiene todos los puertos necesarios abiertos abriendo la [herramienta de prueba de puertos de conector](https://aadap-portcheck.connectorporttest.msappproxy.net/) desde la red local. Cuanto mayor sea el número de marcas de verificación verdes, mayor será la resistencia.

Para asegurarse de que la herramienta ofrezca los resultados correctos, debe:

* Abrir la herramienta en un explorador desde el servidor donde esté instalado el agente de aprovisionamiento.
* Asegurarse de que los servidores proxy o firewalls aplicables al agente de aprovisionamiento también estén aplicados a esta página. Esto puede hacerse en Internet Explorer, vaya a **configuración -> Opciones de Internet -> conexiones -> configuración de LAN**. En esta página, verá el campo "Usar un servidor proxy para la LAN". Active esta casilla y escriba la dirección del proxy en el campo "Dirección".

#### <a name="can-one-provisioning-agent-be-configured-to-provision-multiple-ad-domains"></a>¿Un agente de aprovisionamiento se puede configurar para aprovisionar varios dominios de AD?

Sí, un agente de aprovisionamiento puede configurarse para controlar varios dominios de AD, siempre que el agente tenga una línea de visión de los controladores de dominio correspondientes. Microsoft recomienda configurar un grupo de tres agentes de aprovisionamiento que abastezca al mismo conjunto de dominios de AD para garantizar una alta disponibilidad y proporcionar soporte técnico para la conmutación por error.

#### <a name="how-do-i-de-register-the-domain-associated-with-my-provisioning-agent"></a>¿Cómo puedo anular el registro del dominio asociado con mi agente de aprovisionamiento?

* En Azure Portal, obtenga el *Id. de inquilino* del inquilino de Azure AD.
* Inicie sesión en el servidor de Windows que ejecuta al agente de aprovisionamiento.
* Abra PowerShell como administrador de Windows.
* Cambie al directorio que contiene los scripts de registro y ejecute los siguientes comandos reemplazando el parámetro \[Id. de inquilino\] con el valor del identificador del inquilino.

  ```powershell
  cd “C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder”
  Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder\AppProxyPSModule.psd1"
  Get-PublishedResources -TenantId "[tenant ID]"
  ```

* En la lista de agentes que aparece, copie el valor del campo "Id." de dicho recurso cuyo parámetro *resourceName* es el mismo que el nombre de dominio de AD.
* Pegue el valor de identificador en este comando y ejecute el comando en PowerShell.

  ```powershell
  Remove-PublishedResource -ResourceId "[resource ID]" -TenantId "[tenant ID]"
  ```

* Vuelva a ejecutar el asistente de configuración del agente.
* Será necesario volver a configurar todos los demás agentes asignados anteriormente a este dominio.

#### <a name="how-do-i-uninstall-the-provisioning-agent"></a>¿Cómo puedo desinstalar el agente de aprovisionamiento?

* Inicie sesión en el servidor de Windows está instalado el agente de aprovisionamiento.
* Vaya al menú **Panel de control** -> **Desinstalar o cambiar un programa**.
* Desinstale los programas siguientes:
  * Microsoft Azure AD Connect Provisioning Agent
  * Agent Updater de Microsoft Azure AD Connect
  * Microsoft Azure AD Connect Provisioning Agent Package

### <a name="workday-to-ad-attribute-mapping-and-configuration-questions"></a>Preguntas sobre la asignación y configuración de atributos de Workday en AD

#### <a name="how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema"></a>¿Cómo puedo realizar una copia de seguridad o exportar una copia de trabajo del esquema y la asignación de atributos de aprovisionamiento de Workday?

Puede usar Microsoft Graph API para exportar la configuración de aprovisionamiento de usuarios de Workday. Consulte los pasos descritos en la sección [Exportación e importación de la configuración de la asignación de atributos de aprovisionamiento de usuarios de Workday](#exporting-and-importing-your-configuration) para obtener más información.

#### <a name="i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes"></a>Tengo atributos personalizados en Workday y Active Directory. ¿Cómo se puede configurar la solución para que funcione con los atributos personalizados?

La solución es compatible con los atributos personalizados de Workday y Active Directory. Para agregar los atributos personalizados al esquema de asignación, abra la hoja **Asignación de atributos** y desplácese hacia abajo para expandir la sección **Mostrar opciones avanzadas**. 

![Editar lista de atributos](./media/workday-inbound-tutorial/wd_edit_attr_list.png)

Para agregar los atributos personalizados de Workday, seleccione la opción *Editar lista de atributos para Workday* y, para agregar los atributos personalizados de AD, seleccione la opción *Editar lista de atributos para Active Directory local*.

Consulte también:

* [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes)

#### <a name="how-do-i-configure-the-solution-to-only-update-attributes-in-ad-based-on-workday-changes-and-not-create-any-new-ad-accounts"></a>¿Cómo se puede configurar la solución para actualizar solo los atributos de AD según los cambios de Workday sin crear cuentas de AD?

Esta configuración puede realizarse en **Acciones del objeto de destino** en la hoja **Asignaciones de atributos** como se muestra a continuación:

![Actualizar acción](./media/workday-inbound-tutorial/wd_target_update_only.png)

Active la casilla "Actualizar" solo para que las operaciones de actualización fluyan de Workday a AD. 

#### <a name="can-i-provision-users-photo-from-workday-to-active-directory"></a>¿Puedo aprovisionar la foto de un usuario desde Workday en Active Directory?

La solución actualmente no permite establecer los atributos binarios como *thumbnailPhoto* y *jpegPhoto* en Active Directory.

#### <a name="how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage"></a>¿Cómo se pueden sincronizar los números de móvil desde Workday según el consentimiento del usuario para su uso público?

* Vaya a la hoja "Aprovisionamiento" de la aplicación de aprovisionamiento de Workday.
* Haga clic en las asignaciones de atributos. 
* En **Asignaciones**, seleccione **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workday Workers to Azure AD** (Sincronizar trabajadores de Workday con Azure AD).
* En la página Asignaciones de atributos, desplácese hacia abajo y active la casilla "Mostrar opciones avanzadas".  Haga clic en **Edit attribute list for Workday** (Editar lista de atributos para Workday).
* En la hoja que se abre, busque el atributo "Móvil" y haga clic en la fila para poder editar la **Expresión de API** ![RGPD móvil](./media/workday-inbound-tutorial/mobile_gdpr.png).

* Reemplace la **Expresión de API** con la siguiente expresión nueva, que recupera el número de teléfono móvil profesional solo si la "Marca de uso público" está establecida en "True" en Workday.

    ```
     wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Contact_Data/wd:Phone_Data[translate(string(wd:Phone_Device_Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='MOBILE' and translate(string(wd:Usage_Data/wd:Type_Data/wd:Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='WORK' and string(wd:Usage_Data/@wd:Public)='1']/@wd:Formatted_Phone
    ```

* Guarde la lista de atributos.
* Guarde la asignación de atributos.
* Borre el estado actual y reinicie la sincronización completa.

#### <a name="how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances"></a>¿Cómo puedo dar formato a los nombres para mostrar en AD según los atributos de departamento/país/ciudad del usuario y controlar las variaciones regionales?

Es un requisito común para configurar el *displayName* atributo de AD para que también proporciona información acerca de departamento y el país o región del usuario. Por ejemplo, si John Smith trabaja en el departamento de marketing en Estados Unidos, puede que desee que su atributo *displayName* aparezca como *Smith, John (Marketing-EE. UU.)*.

Aquí es cómo puede controlar estos requisitos para construir *CN* o *displayName* para incluir los atributos como empresa, unidad de negocio, ciudad o país o región.

* Cada atributo de Workday se recupera mediante una expresión de API XPATH subyacente, que se puede configurar en **Asignación de atributos -> Sección avanzada -> Editar lista de atributos para Workday**. Esta es la expresión de API XPATH predeterminada para los atributos de Workday *PreferredFirstName*, *PreferredLastName*, *Company* y *SupervisoryOrganization*.

     | Atributo de Workday | Expresión de API XPATH |
     | ----------------- | -------------------- |
     | PreferredFirstName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:First_Name/text() |
     | PreferredLastName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:Last_Name/text() |
     | Empresa | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Company']/wd:Organization_Reference/@wd:Descriptor |
     | SupervisoryOrganization | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Supervisory']/wd:Organization_Name/text() |
  
   Confirme con su equipo de Workday que la expresión de API anterior es válida para la configuración de su inquilino de Workday. Si es necesario, puede modificar los atributos como se describe en la sección [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

* Del mismo modo, la información de país existente en Workday se recupera con la siguiente expresión XPATH: *wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference*.

     Hay cinco atributos relacionados con el país que se encuentran disponibles en la sección de la lista de atributos de Workday.

     | Atributo de Workday | Expresión de API XPATH |
     | ----------------- | -------------------- |
     | CountryReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-3_Code']/text() |
     | CountryReferenceFriendly | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/@wd:Descriptor |
     | CountryReferenceNumeric | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Numeric-3_Code']/text() |
     | CountryReferenceTwoLetter | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-2_Code']/text() |
     | CountryRegionReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Region_Reference/@wd:Descriptor |

  Confirme con su equipo de Workday que las expresiones de API anteriores son válidas para la configuración de su inquilino de Workday. Si es necesario, puede modificar los atributos como se describe en la sección [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

* Para compilar la expresión de asignación de atributo adecuado, identifique qué atributo de Workday "de forma autoritativa" representa el usuario nombre, nombre, país o región y la última departamento. Supongamos que los atributos son *PreferredFirstName*, *PreferredLastName*, *CountryReferenceTwoLetter* y *SupervisoryOrganization*, respectivamente. Puede usar esto para compilar una expresión para el atributo *displayName* de AD como se indica a continuación para obtener un nombre para mostrar como *Smith, John (Marketing-EE. UU.)*.

    ```
     Append(Join(", ",[PreferredLastName],[PreferredFirstName]), Join(""," (",[SupervisoryOrganization],"-",[CountryReferenceTwoLetter],")"))
    ```
    Una vez obtenida la expresión apropiada, edite la tabla Asignaciones de atributos y modifique la asignación de atributos *displayName* como se muestra a continuación:   ![Asignación de DisplayName](./media/workday-inbound-tutorial/wd_displayname_map.png)

* Para ampliar el ejemplo anterior, supongamos que desea convertir los nombres de ciudad de Workday en valores abreviados y usarlos para crear nombres para mostrar como *Smith, John (CHI)* o *Doe, Jane (NYC)*; este resultado se puede conseguir con una expresión Switch mediante el atributo *Municipalidad* de Workday como la variable determinante.

     ```
    Switch
    (
      [Municipality],
      Join(", ", [PreferredLastName], [PreferredFirstName]),  
           "Chicago", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(CHI)"),
           "New York", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(NYC)"),
           "Phoenix", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(PHX)")
    )
     ```
    Consulte también:
  * [Sintaxis de la función Switch](../manage-apps/functions-for-customizing-application-data.md#switch)
  * [Sintaxis de la función Join](../manage-apps/functions-for-customizing-application-data.md#join)
  * [Sintaxis de la función Append](../manage-apps/functions-for-customizing-application-data.md#append)

#### <a name="how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute"></a>¿Cómo se puede usar SelectUniqueValue para generar valores únicos para el atributo samAccountName?

Supongamos que desea generar valores únicos para el atributo *samAccountName* mediante una combinación de los atributos *FirstName* y *LastName* de Workday. A continuación se presenta una expresión con la que puede empezar:

```
SelectUniqueValue(
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,1), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,2), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,3), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , )
)
```

Cómo funciona la expresión anterior: si el usuario es John Smith, primero intenta generar JSmith; si ya existe JSmith, entonces genera JoSmith, pero si también existe, genera JohSmith. La expresión también garantiza que el valor generado cumple las restricciones de longitud y la restricción de caracteres especiales asociadas con *samAccountName*.

Consulte también:

* [Sintaxis de la función Mid](../manage-apps/functions-for-customizing-application-data.md#mid)
* [Sintaxis de la función Replace](../manage-apps/functions-for-customizing-application-data.md#replace)
* [Sintaxis de la función SelectUniqueValue](../manage-apps/functions-for-customizing-application-data.md#selectuniquevalue)

#### <a name="how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets"></a>¿Cómo se pueden quitar caracteres con signos diacríticos y convertirlos en caracteres habituales del alfabeto inglés?

Use la función [NormalizeDiacritics](../manage-apps/functions-for-customizing-application-data.md#normalizediacritics) para quitar caracteres especiales en el nombre y apellido del usuario, mientras crea la dirección de correo electrónico o el valor CN para el usuario.

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

En esta sección se proporcionan instrucciones específicas sobre cómo solucionar problemas de aprovisionamiento con la integración de Workday mediante los registros de auditoría de Azure AD y los registros del Visor de eventos de Windows Server. Se basa en los conceptos y pasos de solución de problemas genéricos de [Tutorial: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../manage-apps/check-status-user-account-provisioning.md).

En esta sección se tratan los siguientes aspectos sobre la solución de problemas:

* [Configuración del Visor de eventos de Windows para solucionar problemas del agente](#setting-up-windows-event-viewer-for-agent-troubleshooting)
* [Configuración de registros de auditoría de Azure Portal para solucionar problemas con el servicio](#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
* [Información sobre registros para las operaciones de creación de cuentas de usuario de AD](#understanding-logs-for-ad-user-account-create-operations)
* [Información sobre registros para las operaciones de actualización del administrador](#understanding-logs-for-manager-update-operations)
* [Resolución de errores comunes](#resolving-commonly-encountered-errors)

### <a name="setting-up-windows-event-viewer-for-agent-troubleshooting"></a>Configuración del Visor de eventos de Windows para solucionar problemas del agente

* Inicie sesión en la máquina de Windows Server donde se implementa el agente de aprovisionamiento
* Abra la aplicación de escritorio **Visor de eventos de Windows Server**.
* Seleccione **Registros de Windows > Aplicación**.
* Use la opción **Filtrar registro actual...** para ver todos los eventos registrados en el origen **AAD.Connect.ProvisioningAgent** y excluir eventos con el identificador "5", especificando el filtro "-5", tal como se muestra a continuación.

  ![Visor de eventos de Windows](media/workday-inbound-tutorial/wd_event_viewer_01.png))

* Haga clic en **Aceptar** y ordene la vista resultados según la columna **Fecha y hora**.

### <a name="setting-up-azure-portal-audit-logs-for-service-troubleshooting"></a>Configuración de registros de auditoría de Azure Portal para solucionar problemas con el servicio

* Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección **Registros de auditoría** de la aplicación de aprovisionamiento de Workday.
* Use el botón **Columnas** de la página Registros de auditoría para mostrar solo las siguientes columnas en la vista (Fecha, Actividad, Estado y Motivo del estado). Esta configuración garantiza que pueda centrarse solo en los datos que sean pertinentes para solucionar problemas.

  ![Columnas del registro de auditoría](media/workday-inbound-tutorial/wd_audit_logs_00.png)

* Use los parámetros de consulta **Destino** e **Intervalo de fechas** para filtrar la vista. 
  * Establezca el parámetro de consulta **Destino** con el "Id. del trabajador" o el "Id. de empleado" del objeto de trabajador de Workday.
  * Establezca el parámetro **Intervalo de fechas** en un período de tiempo apropiado durante el cual desea investigar los problemas o errores de aprovisionamiento.

  ![Filtros del registro de auditoría](media/workday-inbound-tutorial/wd_audit_logs_01.png)

### <a name="understanding-logs-for-ad-user-account-create-operations"></a>Información sobre registros para las operaciones de creación de cuentas de usuario de AD

Cuando se detecta un nuevo empleado en Workday (supongamos que con el identificador de empleado *21023*), el servicio de aprovisionamiento de Azure AD intenta crear una cuenta de usuario de AD para el empleado y, en el proceso, crea cuatro registros de auditoría como se describe a continuación:

  [![Registro de auditoría de creación de ops](media/workday-inbound-tutorial/wd_audit_logs_02.png)](media/workday-inbound-tutorial/wd_audit_logs_02.png#lightbox)

Al hacer clic en cualquiera de los registros de auditoría, se abre la página **Detalles de la actividad**. Esto es lo que muestra la página **Detalles de la actividad** para cada tipo de registro de auditoría.

* Registro **Workday Import**: este registro muestra la información del trabajador capturada de Workday. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la captura de datos de Workday. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None  // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportAdd // For full sync, value is "EntryImportAdd" and for delta sync, value is "EntryImport"
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID (usually the Worker ID or Employee ID field)
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the record
  ```

* Registro **AD Import**: este registro muestra información sobre la cuenta capturada de AD. Como durante la creación del usuario inicial no hay ninguna cuenta de AD, en *Activity Status Reason* (Motivo del estado de actividad) se indicará que no se encontró ninguna cuenta con el atributo de identificador de coincidencia en Active Directory. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la captura de datos de Workday. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportObjectNotFound // Implies that object was not found in AD
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  ```

  Para buscar registros del agente de aprovisionamiento correspondientes a esta operación de importación de AD, abra los registros del Visor de eventos de Windows y use la opción de menú **Buscar…** para encontrar las entradas de registro que contengan el valor de atributo Id. de coincidencia/Propiedad de combinación (en este caso, *21023*).

  ![Buscar](media/workday-inbound-tutorial/wd_event_viewer_02.png)

  Busque la entrada con *Id. de evento = 9*, que proporcionará el filtro de búsqueda de LDAP utilizado por el agente para recuperar la cuenta de AD. Puede comprobar si este es el filtro de búsqueda adecuado para recuperar entradas de usuario únicas.

  ![Búsqueda LDAP](media/workday-inbound-tutorial/wd_event_viewer_03.png)

  El registro que le sigue inmediatamente con *Id. de evento = 2* captura el resultado de la operación de búsqueda y si devuelve algún resultado.

  ![Resultados LDAP](media/workday-inbound-tutorial/wd_event_viewer_04.png)

* Registro **Synchronization rule action**: este registro muestra los resultados de las reglas de asignación de atributos y los filtros de ámbito configurados junto con la acción de aprovisionamiento que se usará para procesar el evento de Workday de entrada. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la acción de sincronización. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot sync issues
  EventName : EntrySynchronizationAdd // Implies that the object will be added
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  ```

  Si hay problemas con las expresiones de asignación de atributos o los datos de Workday de entrada presentan errores (por ejemplo, valores vacíos o NULL para los atributos obligatorios), aparecerá un error en esta fase con el texto ErrorCode en el que se proporcionarán los detalles del error.

* Registro **AD Export**: este registro muestra el resultado de la operación de creación de la cuenta de AD junto con los valores de atributo definidos en el proceso. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a operación de creación de la cuenta. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo. En la sección "Detalles adicionales", "EventName" se establece en "EntryExportAdd", "JoiningProperty" se establece en el valor del atributo Id. de coincidencia, "SourceAnchor" se establece en el valor WorkdayID (WID) asociado con el registro y "TargetAnchor" se establece en el valor del atributo "ObjectGuid" de AD del usuario recién creado. 

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportAdd // Implies that object will be created
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  TargetAnchor : 83f0156c-3222-407e-939c-56677831d525 // set to the value of the AD "objectGuid" attribute of the new user
  ```

  Para buscar registros del agente de aprovisionamiento correspondientes a esta operación de exportación de AD, abra los registros del Visor de eventos de Windows y use la opción de menú **Encontrar…** para encontrar las entradas de registro que contengan el valor de atributo Id. de coincidencia/Propiedad de combinación (en este caso, *21023*).  

  Busque un registro POST HTTP correspondiente a la marca de tiempo de la operación de exportación con el *Id. de evento = 2*. Este registro contendrá los valores de atributo enviados por el servicio de aprovisionamiento al agente de aprovisionamiento.

  [![Adición de SCIM](media/workday-inbound-tutorial/wd_event_viewer_05.png)](media/workday-inbound-tutorial/wd_event_viewer_05.png#lightbox)

  Inmediatamente después del evento anterior, debe haber otro evento que captura la respuesta de la operación de creación de la cuenta de AD. Este evento devuelve el nuevo valor objectGuid creado en AD y se establece como el atributo TargetAnchor en el servicio de aprovisionamiento.

  [![Adición de SCIM](media/workday-inbound-tutorial/wd_event_viewer_06.png)](media/workday-inbound-tutorial/wd_event_viewer_06.png#lightbox)

### <a name="understanding-logs-for-manager-update-operations"></a>Información sobre registros para las operaciones de actualización del administrador

El atributo manager es un atributo de referencia en AD. El servicio de aprovisionamiento no establece el atributo manager como parte de la operación de creación de un usuario. En su lugar, el atributo manager se establece como parte de una operación de *actualización* después de que se haya creado la cuenta de AD para el usuario. Como ampliación del ejemplo anterior, supongamos que se activa un nuevo trabajador con el identificador de empleado "21451" en Workday y que el administrador del nuevo trabajador (*21023*) ya tiene una cuenta de AD. En este escenario, al buscar los registros de auditoría para el usuario 21451, se muestran cinco entradas.

  [![Actualización del administrador](media/workday-inbound-tutorial/wd_audit_logs_03.png)](media/workday-inbound-tutorial/wd_audit_logs_03.png#lightbox)

Los cuatro primeros registros son similares a los que hemos explorado como parte de la operación de creación del usuario. El quinto registro se corresponde con la exportación asociada con la actualización del atributo manager. El registro muestra el resultado de la operación de actualización del administrador de la cuenta de AD, que se realiza con el atributo *objectGuid* del administrador.

  ```JSON
  // Modified Properties
  Name : manager
  New Value : "83f0156c-3222-407e-939c-56677831d525" // objectGuid of the user 21023

  // Additional Details
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportUpdate // Implies that object will be created
  JoiningProperty : 21451 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : 9603bf594b9901693f307815bf21870a // WorkdayID of the user
  TargetAnchor : 43b668e7-1d73-401c-a00a-fed14d31a1a8 // objectGuid of the user 21451

  ```

### <a name="resolving-commonly-encountered-errors"></a>Resolución de errores comunes

Esta sección trata los errores habituales con el aprovisionamiento de usuarios de Workday y cómo resolverlos. Los errores se agrupan como sigue:

* [Errores del agente de aprovisionamiento](#provisioning-agent-errors)
* [Errores de conectividad](#connectivity-errors)
* [Errores de creación de cuentas de usuario de AD](#ad-user-account-creation-errors)
* [Errores de actualización de cuentas de usuario de AD](#ad-user-account-update-errors)

#### <a name="provisioning-agent-errors"></a>Errores del agente de aprovisionamiento

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Error al instalar al agente de aprovisionamiento con el mensaje de error:  *No se pudo iniciar el servicio "Microsoft Azure AD Connect Provisioning Agent" (AADConnectProvisioningAgent). Compruebe que tiene suficientes privilegios para iniciar el sistema*. | Este error normalmente aparece si intenta instalar al agente de aprovisionamiento en un controlador de dominio y una directiva de grupo impide que el servicio se inicie.  También aparece si tiene una versión anterior del agente en ejecución y no la ha desinstalado antes de iniciar una nueva instalación.| Instale el agente de aprovisionamiento en un servidor que no sea controlador de dominio. Asegúrese de que las versiones anteriores del agente están desinstaladas antes de instalar el nuevo agente.|
|2.| El servicio de Windows "Microsoft Azure AD Connect Provisioning Agent" está en estado de *inicio* y no cambia al estado *En ejecución*. | Como parte de la instalación, el asistente del agente crea una cuenta local (**NT Service\\AADConnectProvisioningAgent**) en el servidor, que es la cuenta de **inicio de sesión** utilizada para iniciar el servicio. Si una directiva de seguridad del servidor de Windows impide que las cuentas locales ejecuten los servicios, se producirá este error. | Abra la *Consola de Servicios*. Haga clic con el botón derecho en el servicio de Windows "Microsoft Azure AD Connect Provisioning Agent" y, en la pestaña Inicio de sesión, especifique la cuenta de un administrador de dominio para ejecutar el servicio. Reinicie el servicio. |
|3.| Al configurar el agente de aprovisionamiento con el dominio de AD en el paso *Conectar a Active Directory*, el asistente tarda mucho tiempo intentando cargar el esquema de AD y finalmente se agota el tiempo de espera. | Este error normalmente se muestra si el asistente no puede ponerse en contacto con el servidor de controlador de dominio de AD debido a problemas de firewall. | En la pantalla del asistente *Conectar a Active Directory*, al proporcionar las credenciales del dominio de AD, hay una opción denominada *Select domain controller priority* (Seleccionar prioridad del controlador de dominio). Use esta opción para seleccionar un controlador de dominio que esté en el mismo sitio que el servidor del agente y asegúrese de que no haya ninguna regla de firewall que bloquee la comunicación. |

#### <a name="connectivity-errors"></a>Errores de conectividad

Si el servicio de aprovisionamiento no puede conectarse a Active Directory o Workday, podría provocar que el aprovisionamiento entre en estado de cuarentena. Use la tabla siguiente para solucionar los problemas de conectividad.

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Al hacer clic en **Probar conexión**, obtendrá el mensaje de error: *Error al conectarse a Active Directory. Asegúrese de que está ejecutando el agente de aprovisionamiento local y de que está configurado con el dominio de Active Directory correcto*. | Este error suele aparecer si el agente de aprovisionamiento no está en ejecución o si hay un firewall que bloquea la comunicación entre Azure AD y el agente de aprovisionamiento. Este error también puede aparecer si el dominio no está configurado en el asistente del agente. | Abra la consola *Servicios* en el servidor de Windows para confirmar que el agente está en ejecución. Abra el asistente del agente de aprovisionamiento y confirme que el dominio correcto está registrado en el agente.  |
|2.| El trabajo de aprovisionamiento entra en estado de cuarentena los fines de semana (viernes y sábado) y recibimos una notificación por correo electrónico de que existe un error con la sincronización. | Una de las causas comunes de este error es el tiempo de inactividad planeado de Workday. Si usa un inquilino de implementación de Workday, tenga en cuenta que Workday programó un tiempo de inactividad para sus inquilinos de implementación durante los fines de semana (normalmente desde el viernes por la noche hasta el sábado por la mañana) y, durante dicho período, las aplicaciones de aprovisionamiento de Workday pueden entrar en estado de cuarentena porque no pueden conectarse a Workday. Vuelve a su estado habitual una vez que el inquilino de implementación de Workday puede volver a conectarse. En raras ocasiones, puede que este error también aparezca si la contraseña del usuario del sistema de integración cambió debido a la actualización del inquilino o si la cuenta tiene un estado bloqueado o expirado. | Póngase en contacto con el asociado de integración o el administrador de Workday para saber cuándo Workday tiene programados tiempos de inactividad, para ignorar los mensajes de alerta durante dicho período y confirmar la disponibilidad una vez que la instancia de Workday vuelva a recuperar la conexión.  |


#### <a name="ad-user-account-creation-errors"></a>Errores de creación de cuentas de usuario de AD

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Errores en la operación de exportación en el registro de auditoría con el mensaje *Error: OperationsError-SvcErr: Error en la operación. No se configuró ninguna referencia superior para el servicio de directorio. Por tanto, el servicio de directorio no puede emitir referencias a los objetos fuera de este bosque.* | Este error suele aparecer si la unidad organizativa del *Contenedor de Active Directory* no está configurada correctamente o si hay problemas con la asignación de expresiones utilizada para *parentDistinguishedName*. | Compruebe si hay algún error ortográfico en el parámetro de la unidad organizativa del *Contenedor de Active Directory*. Si usa *parentDistinguishedName* en la asignación de atributos, asegúrese de que siempre se evalúa en un contenedor conocido dentro del dominio de AD. Consulte el evento *Exportar* en los registros de auditoría para ver el valor generado. |
|2.| Errores en la operación de exportación en el registro de auditoría con el código de error: *SystemForCrossDomainIdentityManagementBadResponse* y el mensaje *Error: ConstraintViolation-AtrErr: Un valor de la solicitud no es válido. Un valor del atributo no estaba en el rango de valores aceptables. \nDetalles del error: CONSTRAINT_ATT_TYPE - company*. | Aunque este error es específico del atributo *company*, puede que también vea este error para otros atributos como *CN*. Este error aparece debido a una restricción de esquema aplicada en AD. De forma predeterminada, los atributos como *company* y *CN* de AD tienen un límite máximo de 64 caracteres. Si el valor procedente de Workday tiene más de 64 caracteres, verá este mensaje de error. | Consulte el evento *Exportar* en los registros de auditoría para ver el valor del atributo indicado en el mensaje de error. Considere la posibilidad de truncar el valor procedente de Workday mediante la función [Mid](../manage-apps/functions-for-customizing-application-data.md#mid) o con el cambio de las asignaciones por un atributo de AD que no tenga restricciones de longitud similares.  |

#### <a name="ad-user-account-update-errors"></a>Errores de actualización de cuentas de usuario de AD

Durante el proceso de actualización de cuentas de usuario de AD, el servicio de aprovisionamiento lee la información de Workday y AD, ejecuta las reglas de asignación de atributos y determina si es necesario aplicar algún cambio. En consecuencia, se desencadena un evento de actualización. Si se presenta algún error con cualquiera de estos pasos, este se refleja en los registros de auditoría. Use la tabla siguiente para solucionar errores de actualización comunes.

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Errores de la acción de las reglas de sincronización en el registro de auditoría con el mensaje *EventName = EntrySynchronizationError and ErrorCode = EndpointUnavailable*. | Este error aparece si el servicio de aprovisionamiento no puede recuperar datos del perfil de usuario de Active Directory debido a un error de procesamiento detectado por el agente de aprovisionamiento local. | Consulte los registros del Visor de eventos del agente de aprovisionamiento para ver si existen eventos de error que indiquen problemas con la operación de lectura (Filtrar por id. de evento n.º 2). |
|2.| El atributo manager de AD no se actualiza para determinados usuarios de AD. | La causa más probable de este error es que usa reglas de ámbito y el administrador del usuario no forma parte del ámbito. Este problema también puede darse si el atributo de identificador de coincidencia del administrador (por ejemplo, EmployeeID) no se encuentra en el dominio de AD de destino o no está establecido en el valor correcto. | Revise el filtro de ámbito y agregue el usuario administrador al ámbito. Compruebe el perfil de administrador de AD para asegurarse de que haya un valor para el atributo de identificador de coincidencia. |

## <a name="managing-your-configuration"></a>Administración de la configuración

En esta sección se describe cómo puede ampliar, personalizar y administrar la configuración de aprovisionamiento de usuarios controlada por Workday. Se tratan los temas siguientes:

* [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes)  
* [Exportación e importación de la configuración](#exporting-and-importing-your-configuration)

### <a name="customizing-the-list-of-workday-user-attributes"></a>Personalización de la lista de atributos de usuario de Workday

Las aplicaciones de aprovisionamiento de Workday para Active Directory y Azure AD incluyen ambas una lista predeterminada de atributos de usuario de Workday que puede seleccionar. Sin embargo, estas listas no son exhaustivas. Workday admite muchos cientos de posibles atributos de usuario, que pueden ser estándar o exclusivos de su inquilino de Workday.

El servicio de aprovisionamiento de Azure AD admite la posibilidad de personalizar la lista o los atributos de Workday para incluir cualquiera de los atributos expuestos en la operación [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) de la API de Recursos Humanos.

Para realizar este cambio, debe usar [Workday Studio](https://community.workday.com/studio-download) para extraer las expresiones XPath que representan los atributos que quiere usar y, luego, agregarlas a la configuración de aprovisionamiento mediante el editor de atributos avanzados de Azure Portal.

**Para recuperar una expresión XPath para un atributo de usuario de Workday, siga estos pasos:**

1. Descargue e instale [Workday Studio](https://community.workday.com/studio-download). Necesitará una cuenta de la comunidad de Workday para acceder al instalador.

2. Descargue el archivo WSDL Human_Resources de Workday desde esta dirección URL: https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Human_Resources.wsdl

3. Inicie Workday Studio.

4. En la barra de comandos, seleccione la opción **Workday > Test Web Service in Tester** (Probar servicio web en Evaluador).

5. Seleccione **External** (Externo) y seleccione el archivo WSDL Human_Resources que descargó en el paso 2.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio1.png)

6. Establezca el campo **Location** (Ubicación) en `https://IMPL-CC.workday.com/ccx/service/TENANT/Human_Resources`, pero reemplace "IMPL-CC" por el tipo de instancia real y "TENANT" por el nombre real del inquilino.

7. Establezca **Operation** (Operación) en **Get_Workers**.

8.  Haga clic en el vínculo pequeño de **configuración** bajo los paneles de solicitud/respuesta para establecer las credenciales de Workday. Marque **Authentication** (Autenticación) y escriba el nombre de usuario y la contraseña de la cuenta del sistema de integración de Workday. Asegúrese de dar formato al nombre de usuario como nombre\@de inquilinos y dejar el **WS-Security UsernameToken** opción seleccionada.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio2.png)

9. Seleccione **Aceptar**.

10. En el panel **Request** (Solicitud), pegue el archivo XML debajo y establezca **Employee_ID** en el identificador de empleado de un usuario real en el inquilino de Workday. Seleccione un usuario que tenga rellenado el atributo que desea extraer.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="https://www.w3.org/2001/XMLSchema">
      <env:Body>
        <wd:Get_Workers_Request xmlns:wd="urn:com.workday/bsvc" wd:version="v21.1">
          <wd:Request_References wd:Skip_Non_Existing_Instances="true">
            <wd:Worker_Reference>
              <wd:ID wd:type="Employee_ID">21008</wd:ID>
            </wd:Worker_Reference>
          </wd:Request_References>
          <wd:Response_Group>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Personal_Information>true</wd:Include_Personal_Information>
            <wd:Include_Employment_Information>true</wd:Include_Employment_Information>
            <wd:Include_Management_Chain_Data>true</wd:Include_Management_Chain_Data>
            <wd:Include_Organizations>true</wd:Include_Organizations>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Transaction_Log_Data>true</wd:Include_Transaction_Log_Data>
            <wd:Include_Photo>true</wd:Include_Photo>
            <wd:Include_User_Account>true</wd:Include_User_Account>
          <wd:Include_Roles>true</wd:Include_Roles>
          </wd:Response_Group>
        </wd:Get_Workers_Request>
      </env:Body>
    </env:Envelope>
    ```

11. Haga clic en **Send Request** (Enviar solicitud) (flecha verde) para ejecutar el comando. Si se ejecuta correctamente, la respuesta debe aparecer en el panel **Response** (Respuesta). Compruebe la respuesta para asegurarse de que tiene los datos del identificador de usuario especificado y no un error.

12. Si es correcta, copie el código XML del panel **Response** (Respuesta) y guárdelo como un archivo XML.

13. En la barra de comandos de Workday Studio, seleccione **File > Open File...** (Archivo > Abrir archivo...) y abra el archivo XML que ha guardado. Esta acción abrirá el archivo en el editor XML de Workday Studio.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio3.png)

14. En el árbol de archivos, desplácese a través de **/env: Envelope > env: Body > wd:Get_Workers_Response > wd:Response_Data > wd: Worker** para encontrar los datos del usuario.

15. En **wd: Worker**, busque el atributo que quiere agregar y selecciónelo.

16. Copie la expresión XPath del atributo seleccionado del campo **Document Path** (Ruta de acceso a documento).

17. Quite el prefijo **/env:Envelope/env:Body/wd:Get_Workers_Response/wd:Response_Data/** de la expresión copiada.

18. Si el último elemento de la expresión copiada es un nodo (ejemplo: "/ wd: Birth_Date"), anexe **/text()** al final de la expresión. Esto no es necesario si el último elemento es un atributo (ejemplo: "/@wd: tipo").

19. El resultado debe ser similar a `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`. Este valor es el que copiará en Azure Portal.

**Para agregar el atributo de usuario personalizado de Workday a la configuración de aprovisionamiento, siga estos pasos:**

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección Aprovisionamiento de la aplicación de aprovisionamiento de Workday, como se ha descrito anteriormente en este tutorial.

2. Establezca **Estado de aprovisionamiento** en **Desactivado** y seleccione **Guardar**. Este paso le ayudará a garantizar que los cambios se apliquen solo cuando esté listo.

3. En **Asignaciones**, seleccione **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workday Workers to Azure AD** (Sincronizar trabajadores de Workday con Azure AD).

4. Desplácese a la parte inferior de la pantalla siguiente y seleccione **Mostrar opciones avanzadas**.

5. Seleccione **Edit attribute list for Workday** (Editar lista de atributos para Workday).

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad1.png)

6. Desplácese a la parte inferior de la lista de atributos, al lugar donde están los campos de entrada.

7. En **Name** (Name), escriba un nombre para mostrar para el atributo.

8. En **Type** (Tipo), seleccione el tipo que corresponde adecuadamente a su atributo (el más común es **String** [Cadena]).

9. En **API Expression** (Expresión de API), escriba la expresión XPath que copió de Workday Studio. Ejemplo: `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`

10. Seleccione **Add Attribute** (Agregar atributo).

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad2.png)

11. Seleccione **Save** (Guardar) arriba y, luego, **Yes** (Sí) en el cuadro de diálogo. Cierre la pantalla de asignación de atributos si aún está abierta.

12. De nuevo en la pestaña principal **Aprovisionamiento**, seleccione otra vez **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workers to Azure AD** (Sincronizar trabajadores con Azure AD).

13. Seleccione **Add new mapping** (Agregar nueva asignación).

14. El nuevo atributo debe aparecer ahora en la lista **Atributo de origen**.

15. Agregue una asignación para el nuevo atributo según sea necesario.

16. Cuando termine, recuerde establecer **Estado de aprovisionamiento** de nuevo en **Activado** y haga clic en Guardar.

### <a name="exporting-and-importing-your-configuration"></a>Exportación e importación de la configuración

En esta sección se describe cómo usar Microsoft Graph API y el Probador de Graph para exportar el esquema y las asignaciones de atributos de aprovisionamiento de Workday a un archivo JSON e importarlos de nuevo en Azure AD.

#### <a name="step-1-retrieve-your-workday-provisioning-app-service-principal-id-object-id"></a>Paso 1: Recuperar el identificador de la entidad de servicio de la aplicación de aprovisionamiento de Workday (Id. de objeto)

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección Propiedades de la aplicación de aprovisionamiento de Workday.
1. En la sección Propiedades de la aplicación de aprovisionamiento, copie el valor GUID asociado con el campo *Id. de objeto*. Este valor también se conoce como el elemento **ServicePrincipalId** de la aplicación y se usará en las operaciones del Probador de Graph.

   ![Identificador de la entidad de servicio de la aplicación de Workday](./media/workday-inbound-tutorial/wd_export_01.png)

#### <a name="step-2-sign-into-microsoft-graph-explorer"></a>Paso 2: Iniciar sesión en el Probador de Graph de Microsoft

1. Iniciar el [Probador de Graph de Microsoft](https://developer.microsoft.com/graph/graph-explorer)
1. Haga clic en el botón "Iniciar sesión con Microsoft" e inicie sesión con las credenciales de administrador de la aplicación o de administrador global de Azure AD.

    ![Inicio de sesión en Graph](./media/workday-inbound-tutorial/wd_export_02.png)

1. Una vez haya iniciado sesión correctamente, verá los detalles de la cuenta de usuario en el panel izquierdo.

#### <a name="step-3-retrieve-the-provisioning-job-id-of-the-workday-provisioning-app"></a>Paso 3: Recuperar el identificador del trabajo de aprovisionamiento de la aplicación de aprovisionamiento de Workday

En el Probador de Graph de Microsoft, ejecute la siguiente consulta GET; para ello, reemplace [servicePrincipalId] con la propiedad **ServicePrincipalId** extraído del [paso 1](#step-1-retrieve-your-workday-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs
```

Obtendrá una respuesta como la que se muestra a continuación. Copie el "atributo de identificador" que aparece en la respuesta. Este valor se corresponde con la propiedad **ProvisioningJobId** y se utilizará para recuperar los metadatos del esquema subyacente.

   [![Id. de trabajo de aprovisionamiento](./media/workday-inbound-tutorial/wd_export_03.png)](./media/workday-inbound-tutorial/wd_export_03.png#lightbox)

#### <a name="step-4-download-the-provisioning-schema"></a>Paso 4: Descargar el esquema de aprovisionamiento

En el Probador de Graph de Microsoft, ejecute la siguiente consulta GET, reemplace [servicePrincipalId] y [ProvisioningJobId] con las propiedades ServicePrincipalId y ProvisioningJobId recuperadas en los pasos anteriores.

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Copie el objeto JSON de la respuesta y guárdelo en un archivo para crear una copia de seguridad del esquema.

#### <a name="step-5-import-the-provisioning-schema"></a>Paso 5: Importar el esquema de aprovisionamiento

> [!CAUTION]
> Realice este paso solo si necesita modificar el esquema de configuración y no puede hacerlo en Azure Portal o si necesita restaurar la configuración desde un archivo de copia de seguridad anterior con un esquema válido que funcione.

En el Probador de Graph de Microsoft, configure la siguiente consulta PUT, reemplace [servicePrincipalId] y [ProvisioningJobId] con las propiedades ServicePrincipalId y ProvisioningJobId recuperadas en los pasos anteriores.

```http
    PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

En la pestaña "Cuerpo de la solicitud", copie el contenido del archivo del esquema JSON.

   [![Cuerpo de la solicitud](./media/workday-inbound-tutorial/wd_export_04.png)](./media/workday-inbound-tutorial/wd_export_04.png#lightbox)

En la pestaña "Encabezados de solicitud", agregue el atributo de encabezado Content-Type con el valor “application/json”.

   [![Encabezados de solicitud](./media/workday-inbound-tutorial/wd_export_05.png)](./media/workday-inbound-tutorial/wd_export_05.png#lightbox)

Haga clic en el botón "Ejecutar consulta" para importar el nuevo esquema.

## <a name="managing-personal-data"></a>Administración de datos personales

La solución de aprovisionamiento Workday para Active Directory requiere que se instale un agente de aprovisionamiento en un servidor de Windows local, y este agente crea registros en el registro de eventos de Windows, que puede contener datos personales en función de las asignaciones de atributos de Workday para AD. Para cumplir las obligaciones de privacidad del usuario, puede asegurarse de que no se conserva ningún dato en los registros de eventos durante más de 48 horas; para ello, configure una tarea programada de Windows para borrar el registro de eventos.

El servicio de aprovisionamiento de Azure AD se engloba dentro de la categoría de **procesador de datos** de la clasificación del RGPD. Como canalización del procesador de datos, el servicio proporciona servicios de procesamiento de datos a asociados clave y consumidores finales. El servicio de aprovisionamiento de Azure AD no genera datos de usuario y no tiene control independiente sobre qué datos personales se recopilan y cómo se usan. La recuperación de datos, la agregación, el análisis y los informes en el servicio de aprovisionamiento de Azure AD se basan en los datos empresariales existentes.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

Con respecto a la retención de datos, el servicio de aprovisionamiento de Azure AD no genera informes, realiza análisis ni proporciona información más allá de 30 días. Por tanto, el servicio de aprovisionamiento de Azure AD no almacena, procesa ni retiene ningún dato más allá de 30 días. Este diseño es compatible con el reglamento del GDPR, las regulaciones de cumplimiento de la privacidad de Microsoft y las directivas de retención de datos de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday](workday-tutorial.md)
* [Aprenda a integrar otras aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [Obtenga información sobre cómo usar Microsoft Graph API para administrar configuraciones de aprovisionamiento](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)
