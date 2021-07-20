---
title:  "SQL*Plus - Historial en Unix-Linux"
author: yoko
categories: 
  - Linux
  - Oracle
show_excerpts: true
tags: 
  - Red Hat
  - CentOS
  - Oracle
  - Sqlplus
date: 2021-07-19 19:00:00 +0000
---
![center-aligned-image](/images/sqlplus.webp){: .align-center}
# Introducción
Has instalado recientemente Oracle y al usar SQL\*Plus no funciona el historial?  
El prompt de SQL*Plus en Unix y Linux por defecto no tiene historial ni es posible usar las flechas de dirección para moverte hacia delante y atrás además de no poder navegar por el historial como indicábamos.


## SQL\*Plus - Habilitar las fechas de dirección
Vamos a instalar `rlwrap` para solucionar nuestro problema.

### Requisitos
Necesitamos instalar gcc,wget y GNU readline:
```bash
yum update -y
yum install gcc readline-devel -y
```
Descarga e instalación
```bash
cd /tmp/
wget https://github.com/hanslub42/rlwrap/archive/refs/heads/master.zip
unzip master.zip
cd rlwrap-master/
autoreconf --install
./configure
make
make test
make install
```

## Añadir un alias
Vamos a añadir el alias al usuario oracle para que cada vez que usemos SQL*Plus sea junto a rlwrap.

```bash
cat << EOF >> /home/oracle/.bashrc
alias sqlplus='rlwrap sqlplus'
alias rman='rlwrap rman'
EOF
```

## ¡Prueba ahora!
```bash
su - oracle
sqlplus sys / as sysdba
```


¡Nos vemos!
