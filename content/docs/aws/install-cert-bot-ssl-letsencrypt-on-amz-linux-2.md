# Install cert bot ssl let'sencrypt on amz linux 2

Install epel & certbot \(for nginx\)

```bash
sudo amazon-linux-extras install epel
sudo yum install certbot-nginx
```

Get a free ssl cert, make sure to set your server IP point to your domain before hand.

```bash
sudo certbot --nginx -d your.domain.com
```

Setup auto renew 

Some Certbot packages do something like this to ensure it runs at a more random time \(twice a day\):

```bash
0 */12 * * * perl -e 'sleep int(rand(3600))' && /usr/bin/certbot -q renew
```

