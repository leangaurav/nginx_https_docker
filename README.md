# HTTPS setup: Nginx Reverse Proxy+ Letsencrypt+ AWS Cloud + Docker

A detailed set of instructions can be found [here](https://leangaurav.medium.com/simplest-https-setup-nginx-reverse-proxy-letsencrypt-ssl-certificate-aws-cloud-docker-4b74569b3c61)  

Below is a basic setup guide:

**Step-1:** Open `config/nginx.conf` replace test.leangaurav.dev with the domain you wish to configure. It can be subdomain like mine or apex domain.

**Step-2:** You can change the nginx version in `docker/nginx.Dockerfile`

**Step-3:** Open `docker-compose-le.yaml` and replace `your.email@email.com` and `test.leangaurav.com`

**Step-4:** In a terminal (T1) run `docker-compose up --build nginx` monitor the logs for errors as we follow next steps

**Step-5:** In another terminal (T2) run `docker-compose -f docker-compose-le.yaml up --build`

**Step-6:** If things go well, the second terminal (T2) will show something like this
```Successfully received certificate.
letsencrypt_1  | Certificate is saved at: /etc/letsencrypt/live/yourdomain.com/fullchain.pem
letsencrypt_1  | Key is saved at:         /etc/letsencrypt/live/yourdomain.com/privkey.pem
letsencrypt_1  | This certificate expires on 2021-10-20.
letsencrypt_1  | These files will be updated when the certificate renews.
letsencrypt_1  |
letsencrypt_1  | NEXT STEPS:
letsencrypt_1  | - The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.
letsencrypt_1  |
letsencrypt_1  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
letsencrypt_1  | If you like Certbot, please consider supporting our work by:
letsencrypt_1  |  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
letsencrypt_1  |  * Donating to EFF:                    https://eff.org/donate-le
letsencrypt_1  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**Step-7:** Close the nginx container (press Ctrl+c or CMD+c) running in first terminal (T1) and replace with below config (after replacing the 4 occurence of `test.leangaurav.dev` with your domain name)
```
server {
    listen 80;
    listen [::]:80;
    server_name test.leangaurav.dev;
location / {
        return 301 https://$host$request_uri;
    }
location ~ /.well-known/acme-challenge {
        allow all;
        root /tmp/acme_challenge;
    }
}
server {
    listen 443 ssl;
    listen [::]:443 ssl http2;
    server_name test.leangaurav.dev;
ssl_certificate /etc/letsencrypt/live/test.leangaurav.dev/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.leangaurav.dev/privkey.pem;
}
```

**Step-8:** Run the nginx container again but with `-d` this time `docker-compose up --build -d nginx`
SSL certificates should be installed. Verify by visiting your domain. You will get an Nginx 404 but it will be served over https. Check the certificate details.

**Step-9:** Setup crontab for auto-renew by running command `crontab -e` and then pasting the below stuff there. Make sure you put the absolute path to the `docker-compose` file.
```0 0 * * 0 expr `date +\%W` \% 2 > /dev/null || docker-compose -f <absolute path to folder>/docker-compose-le.yaml up && docker exec -it nginx-service nginx -s reload ```
