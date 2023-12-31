**Documentação para Configuração de um Cluster Hadoop utilizando Docker**

**PT-BR**

Este guia descreve como criar e configurar um cluster Hadoop em um ambiente Docker. Certifique-se de seguir os passos detalhadamente para garantir uma configuração adequada do seu cluster.

**Passo 1: Criar uma Rede Docker**
Primeiro, crie uma rede Docker para permitir a comunicação entre os containers do cluster.

```bash
docker network create --driver bridge --subnet 192.168.3.0/24 --gateway=192.168.3.1 hadoopnw
```

**Passo 2: Criar o Container Namenode**
Crie um container para o namenode.

```bash
docker run -it --network=hadoopnw --name=namenode --hostname=namenode -p 9870:9870 ubuntu
```

**Passo 3: Criar Containers Datanode**
Crie containers para os datanodes, um para cada nó necessário.

```bash
docker run -it --network=hadoopnw --name=datanode<numero> --hostname=datanode<numero> ubuntu
```
(Substitua `<numero>` pelo número do datanode, por exemplo: `--name=datanode1`)

**Passo 4: Configuração dos Containers**
Execute os seguintes passos em todos os containers do cluster.

**4.1: Atualizar o Container Ubuntu**
```bash
apt update -y
```

**4.2: Instalar Pacotes Necessários**
```bash
apt install -y ubuntu-server default-jdk default-jre nano openssh-server openssh-client sudo
```
Ao instalar os pacotes, selecione as opções: 2 -> 135 -> 77 -> 1 -> 27 -> 15

**4.3: Criar o Usuário Hadoop**
```bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
sudo su - hadoop
```

**4.4: Gerar Chave SSH**
```bash
sudo service ssh start
ssh-keygen -t rsa
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

**4.5: Compartilhar Chaves SSH**
No Namenode, compartilhe a chave SSH de cada datanode:
```bash
cat ~/.ssh/id_rsa.pub | ssh hadoop@datanode<numero> 'cat >> .ssh/authorized_keys'
```
Em cada Datanode, compartilhe a chave SSH do Namenode:
```bash
cat ~/.ssh/id_rsa.pub | ssh hadoop@namenode 'cat >> .ssh/authorized_keys'
```

**4.6: Conceder Permissões para authorized_keys**
```bash
chmod 640 ~/.ssh/authorized_keys
```

**4.7: Baixar e Configurar o Hadoop**
```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xvzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop
sudo mkdir /usr/local/hadoop/logs
sudo chown -R hadoop:hadoop /usr/local/hadoop
```

**4.8: Configurar Variáveis de Ambiente**
```bash
nano ~/.bashrc
```
Adicione as seguintes linhas ao final do arquivo:
```bash
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```
Depois, atualize as variáveis de ambiente:
```bash
source ~/.bashrc
```

**4.9: Configurar Arquivos de Configuração do Hadoop**
Edite os arquivos de configuração conforme abaixo:

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
Adicione:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_CLASSPATH+="$HADOOP_HOME/lib/*.jar"
```

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```
Adicione:
```xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://namenode:9000</value>
  <description>The default file system URI</description>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
Adicione:
```xml
<property>
  <name>dfs.replication</name>
  <value>1</value>
</property>
<property>
  <name>dfs.name.dir</name>
  <value>file:///home/hadoop/hdfs/namenode</value>
</property>
<property>
  <name>dfs.data.dir</name>
  <value>file:///home/hadoop/hdfs/datanode</value>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
Adicione:
```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
</property>
<property>
  <name>mapreduce.application.classpath</name>
  <value>
    $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*
  </value>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
Adicione:
```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
```

**Passo 5: Iniciar o Namenode**
No terminal do namenode, execute os seguintes comandos:
```bash
hdfs namenode -format
start-dfs.sh
start-yarn.sh
jps
```
Acesse "localhost:9870" no navegador para visualizar a interface gráfica do Hadoop.

**Passo 6: Iniciar os Datanodes**
Em

 cada terminal de datanode, execute:
```bash
hdfs --daemon start datanode
start-dfs.sh
```

**Passo 7: Executar um Jar para Testar o Cluster**
No namenode, execute:
```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar pi 2 10000000
```

Isso conclui a configuração do cluster Hadoop utilizando Docker. Certifique-se de revisar os passos e ajustar conforme necessário para o seu ambiente.


**EN**

**Documentation for Setting Up a Hadoop Cluster using Docker**

This guide outlines the steps to create and configure a Hadoop cluster in a Docker environment. Make sure to follow the steps carefully to ensure a proper setup of your cluster.

**Step 1: Create a Docker Network**
First, create a Docker network to allow communication between the cluster containers.

```bash
docker network create --driver bridge --subnet 192.168.3.0/24 --gateway=192.168.3.1 hadoopnw
```

**Step 2: Create the Namenode Container**
Create a container for the namenode.

```bash
docker run -it --network=hadoopnw --name=namenode --hostname=namenode -p 9870:9870 ubuntu
```

**Step 3: Create Datanode Containers**
Create containers for datanodes, one for each required node.

```bash
docker run -it --network=hadoopnw --name=datanode<number> --hostname=datanode<number> ubuntu
```
(Replace `<number>` with the datanode number, e.g., `--name=datanode1`)

**Step 4: Container Configuration**
Execute the following steps on all cluster containers.

**4.1: Update the Ubuntu Container**
```bash
apt update -y
```

**4.2: Install Required Packages**
```bash
apt install -y ubuntu-server default-jdk default-jre nano openssh-server openssh-client sudo
```
When installing packages, select the options: 2 -> 135 -> 77 -> 1 -> 27 -> 15

**4.3: Create the Hadoop User**
```bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
sudo su - hadoop
```

**4.4: Generate SSH Key**
```bash
sudo service ssh start
ssh-keygen -t rsa
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

**4.5: Share SSH Keys**
On the Namenode, share each datanode's SSH key:
```bash
cat ~/.ssh/id_rsa.pub | ssh hadoop@datanode<number> 'cat >> .ssh/authorized_keys'
```
On each Datanode, share the Namenode's SSH key:
```bash
cat ~/.ssh/id_rsa.pub | ssh hadoop@namenode 'cat >> .ssh/authorized_keys'
```

**4.6: Grant Permissions for authorized_keys**
```bash
chmod 640 ~/.ssh/authorized_keys
```

**4.7: Download and Configure Hadoop**
```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xvzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop
sudo mkdir /usr/local/hadoop/logs
sudo chown -R hadoop:hadoop /usr/local/hadoop
```

**4.8: Set Environment Variables**
```bash
nano ~/.bashrc
```
Add the following lines at the end of the file:
```bash
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```
Then, update the environment variables:
```bash
source ~/.bashrc
```

**4.9: Configure Hadoop Configuration Files**
Edit the configuration files as shown below:

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
Add:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_CLASSPATH+="$HADOOP_HOME/lib/*.jar"
```

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```
Add:
```xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://namenode:9000</value>
  <description>The default file system URI</description>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
Add:
```xml
<property>
  <name>dfs.replication</name>
  <value>1</value>
</property>
<property>
  <name>dfs.name.dir</name>
  <value>file:///home/hadoop/hdfs/namenode</value>
</property>
<property>
  <name>dfs.data.dir</name>
  <value>file:///home/hadoop/hdfs/datanode</value>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
Add:
```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
</property>
<property>
  <name>mapreduce.application.classpath</name>
  <value>
    $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*
  </value>
</property>
```

```bash
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
Add:
```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
```

**Step 5: Start the Namenode**
In the namenode terminal, execute:
```bash
hdfs namenode -format
start-dfs.sh
start-yarn.sh
jps
```
Access "localhost:9870" in the browser to view the Hadoop graphical interface.

**Step 6: Start the Datanodes**
In each datanode terminal, execute:
```bash
hdfs --daemon start datanode
start-dfs.sh
```

**Step 7: Run a Jar to Test the Cluster**
In the namenode terminal, execute:
```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar pi 2 10000000
```

This completes the Hadoop cluster setup using Docker. Be sure to review the steps and make adjustments as needed for your environment.
