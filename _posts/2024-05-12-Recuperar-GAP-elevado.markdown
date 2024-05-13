---
title:  "Oracle - Recuperar un GAP elevado 'STANDBY'"
author: ekko
categories: 
  - Oracle
  - Rman
show_excerpts: true
tags: 
  - Oracle
  - rman
date: 2024-05-12 19:00:00 +0000
---
### Recuperar GAP elevado
En primer lugar, vamos a comprobar que el GAP es realmente alto como para tomar esta alternativa:
`STANDBY`
```sql
select thread#, max(sequence#) "Last Standby Seq Received" from v$archived_log val, v$database vdb where val.resetlogs_change# = vdb.resetlogs_change# group by thread# order by 1;
select thread#, max(sequence#) "Last Standby Seq Applied" from v$archived_log val, v$database vdb where val.resetlogs_change# = vdb.resetlogs_change# and val.applied in ('YES','IN-MEMORY') group by thread# order by 1;
select PROCESS, STATUS, THREAD#, SEQUENCE#, BLOCK#, BLOCKS from V$MANAGED_STANDBY where SEQUENCE# >0 order by SEQUENCE#;
```
Em los resultados mostrá que el último aplicado con el último recibido es muy alto, por lo que tenemos un GAP como demuestra el proceso MRP0 **"MRP0 WAIT_FOR_GAP 1 XXXX  0 0"**. Por tanto vamos a resolver la situación haciendo un backup desde la secuencia en la que se encuentra ahora la _STANDBY_ en la _PRIMARIA_ para restablecer la situación.
> The system change number (SCN) is Oracle's clock - every time we commit, the clock increments. The SCN just marks a consistent point in time in the database.

Se comprueba el SCN que se ha quedado en la standby
`STANDBY`
```sql
select to_char(current_scn) from v$database;
-- Por ejemplo 123456789
```

Para el ejercicio voy a crear un directorio específico con suficiente espacio (que tenga los permisos adecuado), por ejemplo /backup/GAP_BACKUP.
Ahora en la primaria, hacemos un backup desde el SCN que hemos obtenido con **RMAN**.
`PRIMARY`
```bash
RMAN> run {
2> allocate channel c1 type disk format '/backup/GAP_BACKUP/%U.bkp';
3> backup incremental from scn 123456789 database;
4> }
```
También tenemos que hacer una copia del controlfile para la stadby en la primaria.
`PRIMARY`
```sql
alter database create standby controlfile as '/backup/GAP_BACKUP/ctl_standby.ctl';
```
Copiamos los ficheros al servidor de destino, previamente he creado una carpeta en el servidor de destino llamada /backup/GAP_RECOVER. Por tanto copiará el/los archivos *.bkp* y el *ctl_standby.ctl*
```bash
scp /backup/GAP_BACKUP/* oracle@serverSTANDBY:/backup/GAP_RECOVER/
```
Ahora vamos realizamos las operaciones de restore en la standby.
`STANDBY`
Paramos la BBDD
```sql
-- Localizamos el controlfile actual (o lo normal es que tengas varios)
SHOW PARAMETER control_files;
-- Por ejemplo lo tenemos ubicado en /u01/oradata/control01.ctl y /u02/oradata/control02.ctl
-- Para la BBDD
shutdown immediate;
-- Levantar sin montar
startup nomount;
````
Ahora movemos los controlfile y copiamos con el que hemos traído desde el servidor de la _PRIMARIA_

```bash
mv /u01/oradata/control01.ctl /u01/oradata/control01.ctl.$(date +"%Y-%m-%d")
mv /u02/oradata/control02.ctl /u02/oradata/control02.ctl.$(date +"%Y-%m-%d")
cp /backup/GAP_RECOVER/ctl_standby.ctl /u01/oradata/control01.ctl
cp backup/GAP_RECOVER/ctl_standby.ctl /u02/oradata/control02.ctl
```
Montar nuevamente 
```sql
alter database mount standby database;
```
El próximo paso es recuperar desde **RMAN**
```bash
# Indicamos la ubicación
rman target /
RMAN> catalog start with '/backup/GAP_RECOVER';
# Iniciar el restore
RMAN> recover database;
...
Se ha finalizado recover a las...
```
En este punto hemos resuelto el GAP, ahora solo tenemos que volver a indicar que inicie el proceso
```sql
alter database recover managed standby database disconnect from session;
```
Podemos comprobar si está aplicando logs desde la consulta que hicimos al principio *"Last Standby Seq Received" "Last Standby Seq Applied"*.

También podemos probar a rotar en la _PRIMARIA_ para comprobar que la secuencia continúa recibiendo/aplicando
`PRIMARY`
```sql
alter system switch logfile;
```
Y de nuevo comprobar la secuencia en _STANDBY_

¡Nos vemos!
