---
title: 'Apache HBase y Enterprise Security Package: Azure HDInsight'
description: 'Tutorial: Aprenda cómo se configura las directivas de Apache Ranger para HBase en Azure HDInsight con Enterprise Security Package.'
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: tutorial
ms.date: 09/04/2019
ms.openlocfilehash: 89e9faeb3c67d0fd0c57adea3a3f69ec5438e3a0
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "73044641"
---
# <a name="tutorial-configure-apache-hbase-policies-in-hdinsight-with-enterprise-security-package"></a>Tutorial: Configuración de directivas de Apache HBase en HDInsight con Enterprise Security Package

Aprenda cómo se configuran las directivas de Apache Ranger para los clústeres de Apache HBase de Enterprise Security Package (ESP). Los clústeres de ESP se conectan a un dominio, lo que permite a los usuarios autenticarse con credenciales de dominio. En este tutorial, creará dos directivas de Ranger para restringir el acceso a diferentes familias de columnas en una tabla de HBase.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear usuarios de dominio
> * Creación de directivas de Ranger
> * Creación de tablas en un clúster de HBase
> * Probar directivas de Ranger

## <a name="before-you-begin"></a>Antes de empezar

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).

* Inicie sesión en [Azure Portal](https://portal.azure.com/).

* Cree un [clúster de HBase de HDInsight con Enterprise Security Package](apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-apache-ranger-admin-ui"></a>Conexión a la interfaz de usuario administrador de Apache Ranger

1. En un explorador, conéctese a la interfaz de usuario administrador de Ranger desde la dirección URL `https://<ClusterName>.azurehdinsight.net/Ranger/`. No olvide cambiar `<ClusterName>` por el nombre del clúster de HBase.

    > [!NOTE]  
    > Las credenciales de Ranger no son las mismas que las credenciales del clúster de Hadoop. Para evitar que los exploradores usen credenciales almacenadas en caché de Hadoop, use una nueva ventana del explorador InPrivate para conectarse a la interfaz de usuario administrador de Ranger.

2. Inicie sesión con sus credenciales de administrador de Azure Active Directory (AD). Las credenciales de administrador de Azure AD no son las mismas que las credenciales del clúster de HDInsight ni las credenciales SSH del nodo de HDInsight Linux.

## <a name="create-domain-users"></a>Crear usuarios de dominio

Visite [Creación de un clúster de HDInsight con Enterprise Security Package](https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds) para aprender a crear los usuario del dominio **sales_user1** y **marketing_user1**. En un escenario de producción, los usuarios del dominio proceden de su inquilino de Active Directory.

## <a name="create-hbase-tables-and-import-sample-data"></a>Creación de tablas de HBase e importación de datos de ejemplo

Puede usar SSH para conectarse a los clústeres de HBase y, después, usar el [shell de Apache HBase](https://hbase.apache.org/0.94/book/shell.html) para crear tablas de HBase e insertar y consultar datos. Para más información, consulte [Uso SSH con HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="to-use-the-hbase-shell"></a>Para usar el shell de HBase

1. Desde SSH ejecute el siguiente comando de HBase:
   
    ```bash
    hbase shell
    ```

2. Cree una tabla HBase `Customers` con dos familias de columnas: `Name` y `Contact`.

    ```hbaseshell   
    create 'Customers', 'Name', 'Contact'
    list
    ```
3. Inserte algunos datos:
    
    ```hbaseshell   
    put 'Customers','1001','Name:First','Alice'
    put 'Customers','1001','Name:Last','Johnson'
    put 'Customers','1001','Contact:Phone','333-333-3333'
    put 'Customers','1001','Contact:Address','313 133rd Place'
    put 'Customers','1001','Contact:City','Redmond'
    put 'Customers','1001','Contact:State','WA'
    put 'Customers','1001','Contact:ZipCode','98052'
    put 'Customers','1002','Name:First','Robert'
    put 'Customers','1002','Name:Last','Stevens'
    put 'Customers','1002','Contact:Phone','777-777-7777'
    put 'Customers','1002','Contact:Address','717 177th Ave'
    put 'Customers','1002','Contact:City','Bellevue'
    put 'Customers','1002','Contact:State','WA'
    put 'Customers','1002','Contact:ZipCode','98008'
    ```
4. Vea el contenido de la tabla:
    
    ```hbaseshell
    scan 'Customers'
    ```

    ![Salida del shell de HBase de HDInsight Hadoop](./media/apache-domain-joined-run-hbase/hbase-shell-scan-table.png)

## <a name="create-ranger-policies"></a>Creación de directivas de Ranger

Cree una directiva de Ranger para **sales_user1** y **marketing_user1**.

1. Abra la **Interfaz de usuario administrador de Ranger**. Haga clic en **\<ClusterName>_hbase** en **HBase**.

   ![Interfaz de usuario de Apache Ranger de HDInsight](./media/apache-domain-joined-run-hbase/apache-ranger-admin-login.png)

2. La pantalla **List of Policies** (Lista de directivas) mostrará todas las directivas de Ranger creadas para este clúster. Puede aparecer una directiva configurada previamente. Haga clic en **Add New Policy** (Agregar nueva directiva).

    ![Lista de directivas de Apache Ranger HBase](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policies-list.png)

3. En la página **Create Policy** (Crear directiva), escriba los siguientes valores:

   |**Configuración**  |**Valor sugerido**  |
   |---------|---------|
   |Nombre de la directiva  |  sales_customers_name_contact   |
   |HBase Table   |  Clientes |
   |HBase Column-family   |  Name, Contact |
   |HBase Column   |  * |
   |Select Group  | |
   |Seleccionar usuario  | sales_user1 |
   |Permisos  | Lectura |

   Los siguientes caracteres comodín se pueden incluir en el nombre del tema:

   * `*` indica ninguna o más repeticiones de caracteres.
   * `?` indica cualquier carácter individual.

   ![Crear directiva de Apache Ranger, ventas](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policy-create-sales.png)

   >[!NOTE]
   >Espere unos instantes para que Ranger se sincronice con Azure AD si un usuario del dominio no se rellena automáticamente en **Seleccionar usuario**.

4. Haga clic en **Agregar** para guardar la directiva.

5. Haga clic en **Agregar nueva directiva** y escriba los siguientes valores:

   |**Configuración**  |**Valor sugerido**  |
   |---------|---------|
   |Nombre de la directiva  |  marketing_customers_contact   |
   |HBase Table   |  Clientes |
   |HBase Column-family   |  Contacto |
   |HBase Column   |  * |
   |Select Group  | |
   |Seleccionar usuario  | marketing_user1 |
   |Permisos  | Lectura |

   ![Crear directiva de Apache Ranger, marketing](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policy-create-marketing.png)  

6. Haga clic en **Agregar** para guardar la directiva.

## <a name="test-the-ranger-policies"></a>Prueba de las directivas de Ranger

Según las directivas de Ranger configuradas, **sales_user1** puede ver todos los datos para las columnas tanto en ambas familias de columnas `Name` y `Contact`. **marketing_user1** solo puede ver los datos en la familia de columnas `Contact`.

### <a name="access-data-as-sales_user1"></a>Acceso a los datos como sales_user1

1. Abra una nueva conexión SSH al clúster. Use el siguiente comandos para iniciar sesión en el clúster:

   ```bash
   ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
   ```

1. Use el comando kinit para cambiar al contexto de nuestro usuario deseado.

   ```bash
   kinit sales_user1
   ```

2. Abra el shell de HBase y examine la tabla `Customers`.

   ```hbaseshell
   hbase shell
   scan `Customers`
   ```

3. Tenga en cuenta que el usuario de ventas puede ver todas las columnas de la tabla `Customers`, incluidas las dos columnas en la familia de columnas `Name`, así como las cinco columnas de la familia de columnas `Contact`.

    ```hbaseshell
    ROW                                COLUMN+CELL
     1001                              column=Contact:Address, timestamp=1548894873820, value=313 133rd Place
     1001                              column=Contact:City, timestamp=1548895061523, value=Redmond
     1001                              column=Contact:Phone, timestamp=1548894871759, value=333-333-3333
     1001                              column=Contact:State, timestamp=1548895061613, value=WA
     1001                              column=Contact:ZipCode, timestamp=1548895063111, value=98052
     1001                              column=Name:First, timestamp=1548894871561, value=Alice
     1001                              column=Name:Last, timestamp=1548894871707, value=Johnson
     1002                              column=Contact:Address, timestamp=1548894899174, value=717 177th Ave
     1002                              column=Contact:City, timestamp=1548895103129, value=Bellevue
     1002                              column=Contact:Phone, timestamp=1548894897524, value=777-777-7777
     1002                              column=Contact:State, timestamp=1548895103231, value=WA
     1002                              column=Contact:ZipCode, timestamp=1548895104804, value=98008
     1002                              column=Name:First, timestamp=1548894897419, value=Robert
     1002                              column=Name:Last, timestamp=1548894897487, value=Stevens
    2 row(s) in 0.1000 seconds
    ```

### <a name="access-data-as-marketing_user1"></a>Acceso a los datos como marketing_user1

1. Abra una nueva conexión SSH al clúster. Use el siguiente comando para iniciar sesión como **marketing_user1**:

   ```bash
   ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
   ```

1. Use el comando kinit para cambiar al contexto de nuestro usuario deseado

   ```bash
   kinit marketing_user1
   ```

1. Abra el shell de HBase y examine la tabla `Customers`:

    ```hbaseshell
    hbase shell
    scan `Customers`
    ```

1. Tenga en cuenta que el usuario marketing solo puede ver las cinco columnas de la familia de columnas `Contact`.

    ```hbaseshell
    ROW                                COLUMN+CELL
     1001                              column=Contact:Address, timestamp=1548894873820, value=313 133rd Place
     1001                              column=Contact:City, timestamp=1548895061523, value=Redmond
     1001                              column=Contact:Phone, timestamp=1548894871759, value=333-333-3333
     1001                              column=Contact:State, timestamp=1548895061613, value=WA
     1001                              column=Contact:ZipCode, timestamp=1548895063111, value=98052
     1002                              column=Contact:Address, timestamp=1548894899174, value=717 177th Ave
     1002                              column=Contact:City, timestamp=1548895103129, value=Bellevue
     1002                              column=Contact:Phone, timestamp=1548894897524, value=777-777-7777
     1002                              column=Contact:State, timestamp=1548895103231, value=WA
     1002                              column=Contact:ZipCode, timestamp=1548895104804, value=98008
    2 row(s) in 0.0730 seconds
    ```

1. Vea los eventos de acceso de auditoría desde la interfaz de usuario de Ranger.

   ![Auditoría de directivas de la interfaz de usuario de Ranger de HDInsight](./media/apache-domain-joined-run-hbase/apache-ranger-admin-audit.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar el clúster HBase que creó mediante los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el cuadro **Búsqueda** en la parte superior, escriba **HDInsight**. 
1. Seleccione **Clústeres de HDInsight** en **Servicios**.
1. En la lista de clústeres de HDInsight que aparece, haga clic en el signo **...**  situado junto al clúster que ha creado para este tutorial. 
1. Haga clic en **Eliminar**. Haga clic en **Sí**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a un ejemplo de Apache HBase en HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md)
