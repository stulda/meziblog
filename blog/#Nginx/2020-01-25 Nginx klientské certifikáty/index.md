# Nginx a zabezpečení aplikací klientskými certifikáty

Se zabezpečením webových aplikací je to složité, často bývají bez omezení přístupné veřejně z internetu a jediné co dělí potencionálního útočníka od přístupu do aplikace je přihlašovací formulář. Pokud provozujete aplikaci, která není určená pro veřejnost, ale pouze pro omezenou skupinu uživatelů, určitě stojí za zvážení implementace ověřování přístupu k aplikaci pomocí klientských certifikátů.

Klientské certifikáty přidávají další vrstvu zabezpečení ještě před samotnou webovou aplikaci, přímo do webového serveru. Klientský certifikát si uživatel uloží do svého operačního systému nebo webového prohlížeče a při přístupu na doménu s aplikací webový server certifikát ověří. Pokud je certifikát v pořádku, aplikace se v prohlížeči normálně zobrazí. V opačném případě se uživatel nebo potencionální útočník k aplikaci vůbec nedostane, jako by neexistovala.

## Příprava certifikátů

Jako první si vytvoříme privátní klíč pro naší certifikační autoritu (CA).

```shell
openssl genrsa -des3 -out ca.key 4096
```

Budete požádáni o nastavení hesla. Určitě si ho někam poznamenejte, budete o něj požádání pokaždé když se bude tvořit nový certifikát nebo podepisovat nový klientský certifikát.

Jako další si vytvoříme CA certifikát:
```shell
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```

Budete požádáni o několik údajů, vyplnit můžete dle libosti, jen Common Name nechat prázdné.

Založení klíče uživatele:
```shell
openssl genrsa -des3 -out user.key 4096
```

Pomocí klíče vytvoříme Certificate Signing Request (CSR):
```shell
openssl req -new -key user.key -out user.csr
```
Požadované údaje vyplňte dle libosti, A challenge password je možno nechat prázdné.

Nyní naší žádost o klientský certifikát podepíšeme pomocí naší certifikační autority:
```shell
openssl x509 -req -days 3650 -in user.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out user.crt
```

Nakonec si jen certifikáty vyexportujeme do formátu, který je možné nainstalovat do webových prohlížečů:
```shell
openssl pkcs12 -export -out user.pfx -inkey user.key -in user.crt -certfile ca.crt
```

Exportovaný PFX soubor už jen stačí bezpečně dopravit ke klientovi a nainstalovat do systému/prohlížeče.

## Nastavení Nginx

V konfiguraci u konkrétní domény, která má být zabezpečena klientskými certifikáty, stačí přidat pouze dva řádky.

```nginx
server {
    ...
    ssl_client_certificate /etc/nginx/client-certs/ca.crt;
    ssl_verify_client on;
    ...
}
```

Volba `ssl_client_certificate` nastavuje cestu k certifikátu naší certifikační autority a volba `ssl_verify_client on;` zapíná samotné ověřování klientských certifikátů. Pokud nemá uživatel nainstalovaný správný certifikát, k aplikaci se nedostane a webový server zobrazí chybu _400 Bad Request - No required SSL certificate_ was sent.

Je ještě možnost volbu `ssl_verify_client` nastavit na hodnotu optional a upravit si chování web serveru pokud na server přistupuje někdo bez validního certifikátu. Například takto:

```nginx
ssl_client_certificate /etc/nginx/client-certs/ca.crt;
ssl_verify_client optional;

location / {

    if ($ssl_client_verify != SUCCESS) {
        return 404;
    }
    ...
}
```

V tomto případě klient bez certifikátu nedostane zpět chybu s informací o tom, že nebyl odeslán klientský certifikát, ale zobrazí se chyba 404 Not Found, jako by na doméně nebylo vůbec nic.
