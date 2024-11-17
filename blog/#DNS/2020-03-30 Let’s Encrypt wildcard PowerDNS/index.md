# Let’s Encrypt wildcard certifikáty automaticky s PowerDNS

Wildcard certifikát je certifikát, který se vystaví pro celou doménu včetně všech subdomén. Možná si říkáte, jestli nejsou **wildcard certifikáty** zbytečné, když je možné si jednoduše jedním příkazem vystavit certifikát pro libovolný počet domén i subdomén (Let’s Encrypt nějaké limity na počet vystavených certifikátů v určeném čase pro doménu má, ale to teď nechme stranou, běžně tohoto limitu málokdo dosáhne). Hlavním důvodem proč používat wildcard certifikáty je soukromí a svým způsobem i bezpečnost.

Existuje totiž něco, čemu se říká **Certificate Transparency**, což je nařízení, které certifikační autority nutí zveřejňovat informace o všech vystavených certifikátech do veřejného registru. To znamená, že se každý může podívat pro jaké subdomény máte na své doméně vystaveny certifikáty, neboli jaké aplikace a weby vám na doméně běží. Určitě to má své opodstatnění, proč tyto údaje zveřejňovat, ale nemusí to být vždy žádoucí. Na subdoménách mohou být provozovány webové aplikace jen pro omezenou skupinu uživatelů a nikdo další nemusí vědět, co kde provozujete.

U Let’s Encrypt se wildcard certifikáty ověřují pomocí DNS záznamů, takže není vůbec potřeba webserver a můžete si tak snadno vystavit validní certifikáty i pro domény, které mají služby dostupné jen v interních sítích.

Na získání certifikátů používám nástroj [Certbot](https://certbot.eff.org/), ten při žádosti wildcard certifikát zobrazí _acme-challenge ověřovací řetězce, které je potřeba přenést do TXT záznamů domény, aby mohlo být provedeno jejich ověření. Díky podpoře pluginů v Certbotu, to lze celé snadno automatizovat.

Zaprvé, nainstalujeme PIP (package manager pro Python):

```shell
apt install python3-pip
```

Pokud nemáme, nainstalujeme Certbot nebo provedeme jeho upgrade na poslední verzi:

```shell
pip3 install --upgrade certbot
```

A nyní instalace Certbot pluginu pro práci s PowerDNS:

```shell
pip3 install certbot-dns-powerdns
```

Zda je plugin správně nainstalován můžeme ověřit výpisem používaných pluginů:

```shell
certbot plugins --text
```

Certbot je nyní připraven. PowerDNS server musí mít povolené API a nastaven klíč pro přístup k API. V powerdns.conf jsou to volby následující volby:

```
api=yes
api-key=<náhodný a dostatečně dlouhý klíč>
```

Pro použití API pomocí Certbot pluginu si připravíme konfigurační soubor, který si pojmenujeme třeba pdns-credentials.ini a vložíme ho do adresáře /etc/letsencrypt. Soubor bude obsahovat volby pro napojení na PowerDNS API:

```
certbot_dns_powerdns:dns_powerdns_api_url = http://127.0.0.1:5000
certbot_dns_powerdns:dns_powerdns_api_key = <náhodný a dostatečně dlouhý klíč>
```

Nakonec již samotné získání certifikátu:

```shell
certbot certonly -d domena.cz -d *.domena.cz --authenticator certbot-dns-powerdns:dns-powerdns --certbot-dns-powerdns:dns-powerdns-credentials /etc/letsencrypt/pdns-credentials.ini
```

Tento poměrně dlouhý příkaz Certbotu řekne, že má pouze vystavit certifikát a nedělat nic jiného pro domena.cz a *.domena.cz a že ověření má provést pomocí PowerDNS pluginu s informací, kde najde konfigurační soubor s PowerDNS API nastavením.

Pokud je vše správně nastaveno, Certbot nastaví TXT záznamy do DNS domény, počká minutu, než se záznamy propíší i do slave DNS serverů, a když záznamy úspěšně ověří, certifikát je vytvořen.