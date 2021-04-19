# oracleodbc
### Instructions for ODBC conection on Oracle
### Tutorial desenvolvido para auxiliar na conexao do Oracle com Zabbix via ODBC.


## Primeiramente fazer download do Oracle Instant Client e Instant Client ODBC
Entre no site da **[Oracle](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)**.
```sh
oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
oracle-instantclient12.2-odbc-12.2.0.1.0-2.x86_64.rpm
```

## Etapas para instalação baseada em debian(Testado no ubuntu 18.04)

Instale o alien
```sh
sudo apt install alien -y
```
Instale os drivers ODBC
```sh
sudo apt libaio1 libaio-dev libodbc1 odbcinst1debian2 unixodbc unixodbc-dev -y
```
Agora instale o Instant Client e o ODBC
```sh
sudo alien -i oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
sudo alien -i oracle-instantclient12.2-odbc-12.2.0.1.0-2.x86_64.rpm
```
Editar os arquivos odbc.ini e odbcinst.ini
```sh
vi /etc/odbcinst.ini
```
# odbcinst.ini
```sh
[Oracle-12]
Description = Oracle 12
Driver = /usr/lib/oracle/12.2/client64/lib/libsqora.so.12.1
#Driver Logging = 7
FileUsage = 1
CPTimeout = 300
```

# odbc.ini
```sh
[DSN]
Driver = Oracle-12
Server = IP_DO_SERVIDOR
ServerName = //IP_DO_SERVIDOR/INSTANCE
UserID = user
Password = senha
```
Validar se todas as libs estao presente
```sh
ldd /usr/lib/oracle/12.2/client64/lib/libsqora.so.12.1

# A saida deve ser parecida com o resultado abaixo
linux-vdso.so.1 (0x00007ffedc5e1000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f711b7b5000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f711b417000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f711b1f8000)
        libnsl.so.1 => /lib/x86_64-linux-gnu/libnsl.so.1 (0x00007f711afde000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f711add6000)
        libaio.so.1 => /lib/x86_64-linux-gnu/libaio.so.1 (0x00007f711abd4000)
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007f711a9ba000)
        libclntsh.so.12.1 => /usr/lib/oracle/12.2/client64/lib/libclntsh.so.12.1 (0x00007f7116f15000)
        libclntshcore.so.12.1 => /usr/lib/oracle/12.2/client64/lib/libclntshcore.so.12.1 (0x00007f7116947000)
        libodbcinst.so.2 => /usr/lib/x86_64-linux-gnu/libodbcinst.so.2 (0x00007f7116732000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7116341000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f7116129000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f711bc7e000)
        libmql1.so => /usr/lib/oracle/12.2/client64/lib/libmql1.so (0x00007f7115eb2000)
        libipc1.so => /usr/lib/oracle/12.2/client64/lib/libipc1.so (0x00007f7115a7f000)
        libnnz12.so => /usr/lib/oracle/12.2/client64/lib/libnnz12.so (0x00007f7115336000)
        libons.so => /usr/lib/oracle/12.2/client64/lib/libons.so (0x00007f71150e8000)
        libltdl.so.7 => /usr/lib/x86_64-linux-gnu/libltdl.so.7 (0x00007f7114ede000)
 ```       
Feito isso exportar as variaveis de ambiente para teste
```sh
export LD_LIBRARY_PATH="/opt/oracle/instantclient_12_2/"
export ORACLE_HOME="/opt/oracle/instantclient_12_2/"
export TNS_ADMIN="/opt/oracle"
```

Testar conexao com isql
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

Verificar onde o zabbix carrega as variaveis de ambiente
```sh
cat /lib/systemd/system/zabbix-server.service (Em caso de Zabbix-Server)
cat /lib/systemd/system/zabbix-proxy.service  (Em caso de proxy)
``` 
Criar o arquivo conforme caminho encontrando na variavel EnvironmentFile
```sh
vi /etc/default/zabbix-server (Caso o path seja esse e caso seja zabbix server, adaptar de acordo com o ambiente)
```
Adicionar as seguintes informacoes no arquivo:
### zabbix-server ou zabbix-proxy
```sh
ORACLE_HOME=/usr/lib/oracle/12.2/client64/bin
LD_LIBRARY_PATH=/usr/lib/oracle/12.2/client64/lib
```
Reiniciar o zabbix e verificar se carregou as variaveis de ambiente
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




# Agradecimentos ao amigo Cabral Junior pelo empenho nos testes e homologação do tutorial e ao [Gabriel Gama](https://github.com/tr4kthebox) que auxiliou com markdown 

