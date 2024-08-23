# Instalación y Configuración de Hadoop, Hive y Spark en Oracle Linux 8

Esta guía completa te ayudará a instalar y configurar un entorno de Big Data en Oracle Linux 8 utilizando **Hadoop**, **Hive** y **Spark**.

## Requisitos Previos

- **Java 8** o superior.
- Usuario con privilegios de sudo.
- Conexión a internet para descargar los paquetes necesarios.

## Instalación y Configuración

### Paso 1: Instalación de Java y Configuración de Hadoop

1. Instala Java 8:

    ```bash
    sudo dnf install java-1.8.0-openjdk-devel
    ```

2. Configurar Java a la versión por defecto:
   
   ```bash
   sudo update-alternatives --config java
   ```

3. Crea el usuario `hadoop` y configura SSH sin contraseña:

    ```bash
    sudo adduser hadoop
    sudo passwd hadoop
    sudo usermod -aG wheel hadoop
    su - hadoop
    ssh-keygen -t rsa -P ""
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys
    ```

3. Descarga e instala Hadoop:

    ```bash
    wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
    tar -zxvf hadoop-3.3.6.tar.gz
    sudo mv hadoop-3.3.6 /usr/local/hadoop
    ```


4. Configura las variables de entorno para Hadoop:

    ```bash
    vim ~/.bashrc
    # Añadir las siguientes líneas al final del archivo
    # Hadoop Variables
   export HADOOP_HOME=/usr/local/hadoop
   export HADOOP_INSTALL=$HADOOP_HOME
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
   export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
  
   source ~/.bashrc
   ```

5. Configura Java en Hadoop:

    ```bash
    vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
    ```

### Paso 2: Configuración de Archivos de Configuración de Hadoop

1. Edita `$HADOOP_HOME/etc/hadoop/core-site.xml` Añade las siguientes líneas entre las etiquetas *configuration*:

    ```xml
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    ```

2. Edita `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` Añade las siguientes líneas entre las etiquetas *configuration*:

    ```xml
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/datanode</value>
    </property>
    ```

3. Edita `$HADOOP_HOME/etc/hadoop/mapred-site.xml` Añade las siguientes líneas entre las etiquetas *configuration*:

    ```xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    ```

4. Edita `$HADOOP_HOME/etc/hadoop/yarn-site.xml`:

    ```xml
    <configuration>
       <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
       </property>
       <property>
          <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
          <value>org.apache.hadoop.mapred.ShuffleHandler</value>
       </property>
       <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>localhost</value>
       </property>
       <property>
          <name>yarn.nodemanager.local-dirs</name>
          <value>/home/hadoop/hadoopdata/yarn/local</value>
       </property>
       <property>
          <name>yarn.nodemanager.log-dirs</name>
          <value>/home/hadoop/hadoopdata/yarn/logs</value>
       </property>
       <property>
          <name>yarn.nodemanager.resource.memory-mb</name>
          <value>1024</value>
       </property>
       <property>
          <name>yarn.nodemanager.resource.cpu-vcores</name>
          <value>1</value>
       </property>
    </configuration>
    ```

5. Formatea el NameNode y arranca los servicios de Hadoop:

    ```bash
    # Finalmente, formatea el NameNode para preparar el HDFS:
    hdfs namenode -format
    start-dfs.sh
    start-yarn.sh
    
   # verificamos
   jps

   # Si se requiere detener el servicio de hadoop
   stop-all.sh
   ```

### Paso 3: Pruebas de Hadoop

1. Crear un directorio en HDFS

   ```bash
   # Crear un directorio en HDFS 
   hadoop fs -mkdir /input
   # subimos el archvio
   hadoop fs -put $HADOOP_HOME/etc/hadoop/hadoop-env.sh /input
   # verificamos
   hadoop fs -ls /input
   # la salida 
   # -rw-r--r--   1 hadoop supergroup      16812 2024-08-17 12:50 /input/hadoop-env.sh
   ```

- Validar Hadoop Web Interface http://localhost:9870
- Hadoop NameNode web interface http://localhost:8088


### Paso 4: Instalación y Configuración de Hive

1. Descarga e instala Hive:

    ```bash
    wget https://downloads.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz
    tar -zxvf apache-hive-3.1.3-bin.tar.gz
    sudo mv apache-hive-3.1.3-bin /usr/local/hive
    ```

2. Configura las variables de entorno para Hive:

    ```bash
    vim ~/.bashrc
    # Añadir las siguientes líneas al final del archivo
    export HIVE_HOME=/usr/local/hive
    export PATH=$PATH:$HIVE_HOME/bin

    source ~/.bashrc
    ```

3. Configura Hive:

    ```bash
    cd $HIVE_HOME/conf/
    cp hive-log4j2.properties.template hive-log4j2.properties
    cp hive-default.xml.template hive-site.xml
    cp hive-env.sh.template hive-env.sh
    vim hive-env.sh
    export HADOOP_HOME=/usr/local/hadoop/
    export HIVE_CONF_DIR=/usr/local/hive/conf
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
    chmod +x hive-env.sh
    ```

4. Edita `$HIVE_HOME/conf/hive-site.xml` Agregamos despues del parametro configuracion en las primeras lineas:

    ```xml
      <property>
         <name>system:java.io.tmpdir</name>
         <value>/tmp/hive/java</value>
      </property>
      <property>
         <name>system:user.name</name>
         <value>${user.name}</value>
      </property>
    ```

5. Crea un directorio en HDFS para almacenar los datos de Hive:

    ```bash
    hdfs dfs -mkdir -p /user/hive/warehouse
    hdfs dfs -chmod g+w /user/hive/warehouse
    ```

6. Crea y configura la base de datos para Hive:

    ```bash
    cd $HIVE_HOME
    mkdir ddbb
    cd ddbb
    schematool -dbType derby -initSchema
    # Comprobamos que la base de datos ha sido generada con éxito:
    ls -l
    # la salida debe mostrar los archivos
    # derby.log 
    # metastore_db
    ```

7. Levantamos el servicio de hive
   
   ```bash
    hive
    # si se configuró correctamente
    show databases;
   ```
8. Si aparece la advertencia de SLF4J, resuélvelo moviendo el archivo conflictivo:

    ```bash
    mv /usr/local/hive/lib/log4j-slf4j-impl-2.17.1.jar /tmp
    ```

### Paso 5: Instalación y Configuración de Apache Spark

1. Descarga e instala Spark:

    ```bash
    wget https://downloads.apache.org/spark/spark-3.5.2/spark-3.5.2-bin-hadoop3.tgz
    tar -xvzf spark-3.5.2-bin-hadoop3.tgz
    sudo mv spark-3.5.2-bin-hadoop3 /usr/local/spark
    ```


2. Configura las variables de entorno para Spark:

    ```bash
    vim ~/.bashrc
    # Añadir las siguientes líneas al final del archivo
    export SPARK_HOME=/usr/local/spark
    export PATH=$PATH:$SPARK_HOME/bin
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin
    export SPARK_DIST_CLASSPATH=$(hadoop classpath)
    
    source ~/.bashrc
    ```

3. Configura Spark para usar Hadoop:

    ```bash
     cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
     vim $SPARK_HOME/conf/spark-env.sh
     # Añadir las siguientes líneas
     export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
     export SPARK_DIST_CLASSPATH=$(hadoop classpath)
    ```

4. Verificar la Instalación

    ```bash
     $SPARK_HOME/bin/spark-shell

     Spark session available as 'spark'.
      Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.5.2
      /_/
    ```
     ```bash
     val rdd = sc.textFile("hdfs:///input/hadoop-env.sh")

     #rdd: org.apache.spark.rdd.RDD[String] = hdfs:///input/hadoop-env.sh MapPartitionsRDD[1] at textFile at <console>:23

     rdd.take(10).foreach(println)
    ```

### Paso 6: Instalación de Python 3.11 y PySpark

1. Instala Python 3.11:

    ```bash
    sudo dnf install make gcc zlib-devel openssl-devel libffi-devel
    sudo dnf install python3.11 python3-devel
    sudo dnf install sqlite-devel
    ```

2. Si se desea instalar desde el repositorio:

    ```bash
    sudo wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
    sudo tar xzf Python-3.11.0.tgz
    mv Python-3.11.0 /usr/src/python-3.11
    cd /usr/src/python-3.11
    sudo ./configure --enable-optimizations
    sudo make altinstall
    ```

3. Instala PySpark:

    ```bash
    pip3.11 install pyspark
    pip3.11 install ipython
    ```

4. Configura las variables de entorno para PySpark:

    ```bash
    vim ~/.bashrc
    # Añadir las siguientes líneas al final del archivo
    export PYSPARK_PYTHON=/usr/local/bin/python3.11
    export PYSPARK_DRIVER_PYTHON=/usr/local/bin/python3.11
    
    source ~/.bashrc
    ```

5. Finalmente  tu archivo .bashrc deberia estar asi.

    ```bash
    # Hadoop Variables
    export HADOOP_HOME=/usr/local/hadoop
    export HADOOP_INSTALL=$HADOOP_HOME
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export YARN_HOME=$HADOOP_HOME
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
    export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib/*:.

    # Hive Variables
    export HIVE_HOME=/usr/local/hive
    export PATH=$PATH:$HIVE_HOME/bin:$HIVE_HOME/conf
    # export HIVE_CONF_DIR=$HIVE_HOME/conf

    # Hive Spark
    export SPARK_HOME=/usr/local/spark
    export PATH=$PATH:$SPARK_HOME/bin
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin
    export SPARK_DIST_CLASSPATH=$(hadoop classpath)

    # evitar mensajes de advertencia es para python
    export PATH=$PATH:/usr/local/bin

    # anadimos los entornos de variables para que reconosca pyspark la version de python
    export PYSPARK_PYTHON=/usr/local/bin/python3.11
    export PYSPARK_DRIVER_PYTHON=/usr/local/bin/python3.11
    ```

