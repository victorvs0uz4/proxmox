## Configurando notificações de backup do Proxmox por e-mail.

1. ##### Instalando dependências de autenticação:

   ```bash
   apt-get install libsasl2-modules
   ```

2. ##### Vamos para página [App Passwords](https://security.google.com/settings/security/apppasswords) criar uma senha de aplicativo para nosso Proxmox.

3. ##### Criando arquivo de senhas:

   ```bash
   nano /etc/postfix/sasl_passwd
   ```

4. ##### Insira as informações do Login:

   ```bash
   smtp.gmail.com youremail@gmail.com:yourpassword
   ```

5. ##### Salve o arquivo de senhas.

6. ##### Criando banco de dados para o arquivo de senha:

   ```shell
   postmap hash:/etc/postfix/sasl_passwd
   ```

7. ##### Proteja o arquivo de senha:

   ```shell
   chmod 600 /etc/postfix/sasl_passwd
   ```

8. ##### Editando o arquivo de configuração do Postfix:

   ```shell
   nano /etc/postfix/main.cf
   ```

9. ##### Adicione/altere o seguinte: (os certificados podem ser encontrados em /etc/ssl/certs/):

   ```shell
   relayhost = smtp.gmail.com:587
   smtp_use_tls = yes
   smtp_sasl_auth_enable = yes
   smtp_sasl_security_options =
   smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
   smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
   smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
   smtp_tls_session_cache_timeout = 3600s
   ```

10. ##### Recarregando as configurações do Postfix:

    ```shell
    postfix reload
    ```

##### Testando:

```shell
echo "Essa é uma mensagem de teste." | mail -s "Teste" youremail@gmail.com
```
