## Configuracion de Servidor de correos utilizando postfix y dovecot

La configuracion funciona para un servidor de correos con un dominio, permite el envio y recepcion de correos, ademas de la configuracion de DKIM, SPF, DMARC y MTA-STS, y enciptacion de correos con TLS.

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