## Configuracion de Servidor de correos utilizando postfix y dovecot

La configuracion funciona para un servidor de correos multidominio utilizando como base de datos los archivos .db de postfix, permite el envio y recepcion de correos, ademas de la configuracion de DKIM, SPF, DMARC y MTA-STS, y enciptacion de correos con TLS de los diferentes dominios.


[postfix]: https://www.postfix.org/
[dovecot]: https://www.dovecot.org/
[mailutils]: https://mailutils.org/
### Lista de paquetes a instalar
- [postfix]
- [dovecot]
- [mailutils]
- opendkim
- clamav
- amavis

### Consideraciones previas
- Certificados SSL para el dominio
- Se debe tener un dominio registrado y configurado con un registro MX apuntando a la IP del servidor de correos.
 Se debe tener un registro TXT SPF configurado en el dominio. El nombre de este ser el nombre del dominio.
    - Ejemplo si el dominio es `example.com` el registro SPF debe ser `example.com`.
    - Su conteneido puede ser `v=spf1 +mx +a +ip4:86.86.87.87 +include:_spf.google.com -all`

- Se debe tener un registro TXT DKIM configurado en el dominio 
    - El nombre de este ser el nombre del dominio, ejemplo si el dominio es `example.com` el registro DKIM debe ser `default._domain_key.example.com`.
    - Su conteneido puede ser `v=DKIM1; k=rsa; p=clave publica de DKIM`

- Se debe tener un registro TXT DMARC configurado en el dominio.
    - El nombre de este ser el nombre del dominio, ejemplo si el dominio es `example.com` el registro DMARC debe ser `_dmarc.example.com`.
    - Su conteneido puede ser `v=DMARC1; p=none; rua=mailto:
    ; ruf=mailto:`

- Se debe tener un registro PTR configurado en la IP del servidor de correos.

- Se debe tener un registro A configurado en el dominio apuntando a la IP del servidor de correos.

- Opcional se puede tener un registro AAAA configurado en el dominio apuntando a la IP del servidor de correos, si se tiene IPv6, ademas debe de estar configurado en el servidor de correos para poder soportarlo.

- Se debe tener un registro TXT mta-sts configurado en el dominio. 
    - El nombre de este debe de ser si el dominio es `example.com` `_mta-sts.example.com`.
    - Su conteneido puede ser `v=STSv1; id=20210622085700Z;`

- Se debe tener un registro A para el sudominio `mta-sts` configurado en el dominio apuntando a la IP del servidor de correos.
    - El nombre del registro debe de ser si el dominio es `example.com` `mta-sts.example.com`.
    - Se debe de acceder a la URL `https://mta-sts.example.com/.well-known/mta-sts.txt` y verificar que se pueda acceder al archivo `mta-sts.txt`
    - El contenido del archivo `mta-sts.txt` debe de ser `version: STSv1 mode: enforce mx: example.com max_age: 86400`

- Si configura amavis asegurese de que el parametro smptpd_tls_security_level este en `may` si amavis no esta configurado con SSL o en `encrypt` si amavis esta configurado con SSL.

### Consideraciones de postfix para multiples dominios
- El parametro `mydestination` no debe de incluir los dominios que se encuentra en el parametro `virtual_mailbox_domains`.

- Se debe de tener un archivo `vmailbox` en `/etc/postfix/` con el siguiente formato:
    - `info@example.com  example.com/info/`
    - Cada vez que se modifique este archivo se debe de ejecutar el comando `postmap vmailbox` para que se actualice el archivo `vmailbox.db` que es el que utiliza postfix.

- Los homes de los usuarios deben de estar en `/var/mail/vhosts/` y deben de tener los permisos `vmail:vmail` y `750`.
    - Ejemplo: `/var/mail/vhosts/example.com/info/`

- Dentro de cada home de los usuarios deben existir dos archivos `passwd` y `shadow` con los datos de los usuarios.
    - Ejemplo: `/var/mail/vhosts/example.com/info/passwd`
    - Ejemplo: `/var/mail/vhosts/example.com/info/shadow`

- Registro MX en el dominio apuntando al hostname del servidor de correos.
    - Ejemplo: `example.com. 3600 IN MX 10 mail.example.com.`

- Se deben de cumplir las [Consideraciones previas](#consideraciones-previas) para cada dominio que se registre.