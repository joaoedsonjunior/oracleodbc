# oracleodbc
# Instructions for ODBC conection on Oracle

Tutorial desenvolvido para auxiliar na conexao do Oracle com Zabbix via ODBC.

# Fazer download do Oracle Instant Client e Instant Client ODBC
https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html

# Criar a pasta e descompactar o client
```sh
mkdir /opt/oracle/
```
# Copiar o client baixado pra pasta criada e descompactar
```sh
unzip instantclient-odbc instantclient
ln -s /opt/oracle/instantclient_12_2/libsqora.so.12.1 /lib/libsqora.so.12.1
ln -s /opt/oracle/instantclient_12_2/libsqora.so.12.1 /lib64/libsqora.so.12.1
ln -s /opt/oracle/instantclient_11_2/libocci.so.11.1 /lib/libocci.so.11.1
ln -s /opt/oracle/instantclient_11_2/libocci.so.11.1 /lib64/libocci.so.11.1
```

# Instalar ODBC Driver

Debian
```sh
apt install unixodbc odbcinst1debian2 libodbc1
```
Centos
```sh
dnf install libiodbc.x86_64 unixODBC.x86_64 libiodbc-devel.x86_64 unixODBC-devel.x86_64
```

# Validar se todas as libs estao presente
```sh
ldd /opt/oracle/instantclient_11_2/libsqora.so.12.1

# A saida deve ser parecida com o resultado abaixo
linux-vdso.so.1 (0x00007fff15589000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f9aaa3a1000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f9aaa09d000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f9aa9e80000)
        libnsl.so.1 => /lib/x86_64-linux-gnu/libnsl.so.1 (0x00007f9aa9c68000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f9aa9a60000)
        libaio.so.1 => /lib/x86_64-linux-gnu/libaio.so.1 (0x00007f9aa985e000)
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007f9aa9647000)
        libclntsh.so.12.1 => /usr/lib/oracle/12.2/client64/lib/libclntsh.so.12.1 (0x00007f9aa5ba2000)
        libclntshcore.so.12.1 => /usr/lib/oracle/12.2/client64/lib/libclntshcore.so.12.1 (0x00007f9aa55d4000)
        libodbcinst.so.2 => /usr/lib/x86_64-linux-gnu/libodbcinst.so.2 (0x00007f9aa53bf000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9aa5020000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f9aa4e09000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9aaa86a000)
        libmql1.so => /usr/lib/oracle/12.2/client64/lib/libmql1.so (0x00007f9aa4b92000)
        libipc1.so => /usr/lib/oracle/12.2/client64/lib/libipc1.so (0x00007f9aa475f000)
        libnnz12.so => /usr/lib/oracle/12.2/client64/lib/libnnz12.so (0x00007f9aa4016000)
        libons.so => /usr/lib/oracle/12.2/client64/lib/libons.so (0x00007f9aa3dc8000)
        libltdl.so.7 => /usr/lib/x86_64-linux-gnu/libltdl.so.7 (0x00007f9aa3bbe000)
 ```       

# Caso falte alguma lib da um locate na mesma e cria o link simbolico pra pasta lib(Em caso de 32bits) ou lib64(Em caso de 64bits)
```sh
Exemplo:
        libodbcinst.so.2 => not found
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9aa5020000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f9aa4e09000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9aaa86a000)
        libmql1.so => /usr/lib/oracle/12.2/client64/lib/libmql1.so (0x00007f9aa4b92000)
        libipc1.so => /usr/lib/oracle/12.2/client64/lib/libipc1.so (0x00007f9aa475f000)
        libnnz12.so => /usr/lib/oracle/12.2/client64/lib/libnnz12.so (0x00007f9aa4016000)
        libons.so => /usr/lib/oracle/12.2/client64/lib/libons.so (0x00007f9aa3dc8000)
        libltdl.so.7 => /usr/lib/x86_64-linux-gnu/libltdl.so.7 (0x00007f9aa3bbe000)
        
locate libodbcinst
/usr/lib64/libodbcinst.so
/usr/lib64/libodbcinst.so.2


# Crie o link conforme abaixo
ln -s /usr/lib64/libodbcinst.so.2 /usr/lib64/libodbcinst.so.1
```

# Crie e edite o arquivo tnsnames.ora conforme abaixo.
```sh
vi /opt/oracle/tnsnames.ora

SERVIDOR =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = "IP")(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = "INSTANCE ORACLE")
    )
  )
```

# Edite o arquivo /etc/odbcinst.ini conforme abaixo
```sh
[oracle]
Description=ODBC for Oracle
Driver=/opt/oracle/instantclient_12_2/libsqora.so.12.1
FileUsage=1
Driver Logging=7
```
# Edite o arquivo /etc/odbc.ini
| Parametro            | Instrucoes |
|--------------------- | ---------------------------------------------------------|
| [Base DN] |
| Driver = oracle      | -- Configurado previamente no odbcinst [oracle] |
| DSN = Base DN	       | -- "INSTANCE ORACLE" conforme SERVICE_NAME do tnsnames |
| ServerName = 	       | -- Nome do Servidor configurado no TNSNAMES |
| UserID = 	       | -- ID usuário do monitoramento |
| Password =           | -- Senha do usuário do monitoramento |


# Feito isso exportar as variaveis de ambiente para teste
```sh
export LD_LIBRARY_PATH="/opt/oracle/instantclient_12_2/"
export ORACLE_HOME="/opt/oracle/instantclient_12_2/"
export TNS_ADMIN="/opt/oracle"
```

# Testar conexao com isql
```sh
isql -v "INSTANCE ORACLE"

# O resultado deve ser conforme abaixo
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
```

# Verificar onde o zabbix carrega as variaveis de ambiente
```sh
cat /usr/lib/systemd/system/zabbix-server.service (Em caso de Zabbix-Server)
cat /usr/lib/systemd/system/zabbix-proxy.service  (Em caso de proxy)
``` 
# Criar o arquivo conforme caminho encontrando na variavel EnvironmentFile
```sh
vi /etc/sysconfig/zabbix-server (Caso o path seja esse e caso seja zabbix server, adaptar de acordo com o ambiente)
```
# Adicionar as seguintes informacoes no arquivo:
```sh
LD_LIBRARY_PATH="/opt/oracle/instantclient_12_2/"
ORACLE_HOME="/opt/oracle/instantclient_12_2/"
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/oracle/instantclient_12_2/
TNS_ADMIN="/opt/oracle"

export ORACLE_HOME
export LD_LIBRARY_PATH
export TNS_ADMIN
export PATH
```

# Reiniciar o zabbix e verificar se carregou as variaveis de ambiente
```sh
systemctl restart zabbix-server
ps -aux |grep zabbix
cat /proc/PID_ZABBIX/environ

# O resultado tem que ser semelhante ao abaixo:
LANG=pt_BR.UTF-8LANGUAGE=pt_BR:pt:enPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/oracle/instantclient_12_2/INVOCATION_ID=d6c9931de97a4df6af7bb6db7f7d342aJOURNAL_STREAM=8:11598215CONFFILE=/etc/zabbix/zabbix_proxy.confLD_LIBRARY_PATH=/opt/oracle/instantclient_12_2/ORACLE_HOME=/opt/oracle/instantclient_12_2/TNS_ADMIN=/opt/oracle
```
# Feito isso va ate a interface web, adicione o Template Oracle by ODBC e cadastre as seguintes macros:
| MACRO              | Valor |
|------------------- | ----- |
| {$ORACLE.DSN}      | Instance ORACLE |
| {$ORACLE.PASSWORD} | Senha |
| {$ORACLE.PORT}     | 1521 (Porta padrao 1521 ou 1523, customizar de acordo com ambiente) |
| {$ORACLE.USER}     | Usuario banco |







        
