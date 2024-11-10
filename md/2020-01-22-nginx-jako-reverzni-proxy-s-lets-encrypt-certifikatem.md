# Nginx jako reverzní proxy s Let’s Encrypt certifikátem

Reverzní proxy umožňuje schovat několik webových aplikací běžících na různých číselných portech (například v Dockeru) pod jeden webový server. Každou aplikaci je tak možno provozovat na vlastní doméně a rovnou přidat SSL šifrování. Možností, jak toho docílit je několik. Já jsem si oblíbil webový server **Nginx**, který má přehlednou konfiguraci, je velmi výkonný a postupně z trůnu sesazuje dlouholetého krále webových serverů Apache.

Jako certifikační autoritu na vystavení důvěryhodného certifikátu používám **Let’s Encrypt**, což se už stává standardem. Získání certifikátů je zdarma a jejich vystavení i obnovu lze zcela automatizovat.

Příprava je jednoduchá, stačí nainstalovat balíky Nginx a Certbot. Já na serverech většinou používám Debian, kde instalace vypadá takto:

```bash
apt install nginx certbot
```

## Vystavení Let’s Encrypt certifikátu

Jako první si připravíme Nginx konfiguraci pro naší doménu zatím jen na portu 80, tedy pro http protokol.

```nginx
server {

    server_name meziblog.cz www.meziblog.cz;

    listen 80;
    listen [::]:80;

    location ^~ /.well-known/ {
        alias /srv/web/.well-known/;
    }

    return 301 https://meziblog.cz$request_uri;

}
```

Důležité je nastavení location `^~ /.well-known/`, které definuje umístění adresáře `.well-known`. V mém případě používám na serveru adresář `/srv/web/`, kde se nachází defaultní web, když není nastavena žádná doména. V tomo adresáři by pak měl být podadresář .well-known, do kterého certbot zapisuje data, pomocí kterých se ověří vlastnictví domény a následně vystaví certifikát.

Poslední řádek `return 301 https://meziblog.cz$request_uri;` nastavuje přesměrování, aby všechny požadavky na server byly přesměrovány z http na https (až bude ready certifikát).

Provedeme reload Nginxu:

```bash
systemctl reload nginx
```

A můžeme se pustit do vystavení certifikátu:

```bash
certbot certonly --webroot -w /srv/web/ -d meziblog.cz -d www.meziblog.cz
```

První parametr `certonly` říká, aby certbot vystavil pouze certifikát a nedělal nic jiného. Pro certbot totiž existují moduly, které automaticky upravují nastavení webových serverů hned po vystavení certifikátu. To já ale nemám rád, akorát se tím nadělá nepořádek v konfiguračních souborech.

Dále se definuje `--webroot` adresář nastavený v konfiguraci web serveru, do kterého certbot zapíše data pro ověření (pravděpodobně to budou nějaké textové soubory, nikdy jsem neměl potřebu to zkoumat) a už jen následuje seznam domén, pro které se bude certifikát vystavovat.

Pokud byl certifikát úspěšně vystaven je nutné zajistit jeho pravidelnou obnovu, protože certifikáty Let’s Encrypt mají platnost pouze 3 měsíce. Stačí níže uvedený příkaz nastavit do cronu, aby se prováděl každý den někdy v noci a je o vše postaráno.

```bash
certbot renew && systemctl reload nginx
```

## Konfigurace SSL a reverzní proxy

Nyní už máme vše připraveno pro zapnutí HTTPS protokolu. Nejdůležitější je nastavit správně cesty k certifikátu a klíči. Certbot po úspěšném vystavení ukáže cesty, kam soubory uložil, typicky je to do */etc/letsencrypt/live/doména*.

Takto vypadá aktuální Nginx konfigurace pro tento web:

```nginx
server {

    server_name meziblog.cz www.meziblog.cz;

    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /etc/letsencrypt/live/meziblog.cz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/meziblog.cz/privkey.pem;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout  10m;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    if ($host = www.meziblog.cz) {
        return 301 https://meziblog.cz$request_uri;
    }

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header HOST $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://127.0.0.1:3001;
    }

}
```

Volby, které se postarají o načtení certifikátu a privátního klíče jsou `ssl_certificate` a `ssl_certificate_key`.

Následuje několik dalších voleb, které upravují nastavení SSL. Sám přesně nevím co přesně všechny dělají, ale existuje velmi užitečný web [cipherli.st](https://cipherli.st), který obsahuje konfigurace pro několik webových serverů, aby nastavení SSL bylo co nejbezpečnější. Stačí jen zkopírovat a máte jistotu, že vaše nastavení bude pro tuto chvíli bezpečné. Já jsem ale udělal jednu malou úpravu a u `ssl_protocols` jsem zapnul i protokol TLSv1.2, protože je stále ještě považovaný za bezpečný a TLSv1.3 ještě nemusí mít podporu všude.

V další části se pomocí `add_header` nastavují základní bezpečnostní hlavičky, zejména pak **HSTS** neboli **Strict-Transport-Security**. Tato hlavička říká prohlížeči, že web poběží vždy na HTTPS protokolu. Prohlížeč si to pamatuje a už nikdy nedovolí přístup k webu přes HTTP a ani nedovolí přístup na web pokud nemá validní certifikát, pozor na to.

Pod hlavičkami je přidáno přesměrování, aby web byl dostupný pouze na doméně bez www na začátku. Někdo má zase raději, když web běží na doméně jen s www. Podle mě je to dávno přežitek a www je zbytečné. Každopádně je dobré, aby web běžel jen na jedné doméně a z druhé bylo nastaveno přesměrování.

Nakonec následuje nastavení samotné **reverzní proxy**. Volba `proxy_pass` určuje na jakou IP adresu a port budou požadavky směrovány. Předtím se ještě nastavují nějaké hlavičky pomocí `proxy_set_header`, aby cílová aplikace měla informace o tom, že běží schovaná za reverzní proxy, jakou skutečnou IP adresu má klient, který si zobrazuje web, na jaké doméně aplikace běží a že komunikuje na HTTPS protokolu.

Vše s tímto nastavením bezvadně funguje a bezpečnost nastavení SSL lze snadno kdykoliv otestovat díky webu [SSL Labs](https://www.ssllabs.com/ssltest/).

#nginx #ssl #letsencrypt