# Blokování reklam a malwaru pomocí DNS

Reklamy na internetu jsou zlo. Ne že bych chtěl provozovatelům a redaktorům webů odepřít jejich příjmy, rozhodně si odměnu za svou práci zaslouží, ale reklam je na těch webech tolik, až je to neúnosné. Načítání stránek je kvůli reklamám pomalejší, už se párkrát stalo, že skrze reklamní systémy byl šířen malware, a reklamní systémy sledují chování návštěvníků napříč různými weby atd.

Existuje několik řešení jak reklamy a malware omezit, velmi oblíbené jsou rozšíření pro prohlížeče Adblock nebo uBlock a tak podobně. Jenže, aby mohlo rozšíření na webu blokovat obsah, musí mít oprávnění číst obsah všech webů, které navštívíte, a to také není úplně ideální. Lepší je zařídit se po svém.

Na internetu jsou k dispozici veřejné seznamy domén reklamních systémů a adres šířících malware, které jsou pravidelně aktualizovány. Já používám seznamy z tohoto repositáře na GitHubu: [https://github.com/StevenBlack/hosts](https://github.com/StevenBlack/hosts)

Kromě základního seznamu domén s adwarem a malwarem jsou k dispozici seznamy s porno weby, fakenews, gamblerskými weby, sociálními sítěmi a jsou zde i různě nakombinovány.

Pokud chcete blokovat reklamy jen na svém počítači, stačí stáhnout jeden ze seznamů a nahrát ho jako soubor _/etc/hosts_ na Linuxu nebo _C:\Windows\System32\Drivers\etc\hosts_ na Windows (Na Windows to nemám otestované).

Já ale nechci blokovat adresy jen na jednom počítači, chci chránit všechna zařízení, včetně těch mobilních, která jsou připojená do mé lokální sítě nebo do VPN a k tomu mi pomůže můj linuxový router a DNS resolver Unbound.

Nejprve si připravíme nastavení DNS resolveru Unbound, do konfiguračního souboru /etc/unbound/unbound.conf v sekci server: přidáme následující řádek:

```
include: /etc/unbound/adblock.conf
```

Soubor adblock.conf bude obsahovat seznam blokovaných domén a na naplnění souboru doménami jsem si připravil následující jednoduchý skriptík:

```shell
#!/bin/bash

wget https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts -O /tmp/adblock-hosts

cat /tmp/adblock-hosts | grep '^0\.0\.0\.0' | awk '{print "local-zone: \""$2"\" redirect\nlocal-data: \""$2" A 0.0.0.0\""}' > /etc/unbound/adblock.conf

rm /tmp/adblock-hosts

systemctl restart unbound
```

Nejprve se stáhne seznam adres, převede se do formátu, aby tomu Unbound rozuměl a provede se restart služby.

Teď už jen stačí, aby DHCP server na routeru nastavoval klientům adresu našeho DNS resolveru a už mají všechny počítače i mobilní zařízení v síti nastaveno blokování reklam a malwaru. Já to takto používám i na své VPN, ke které je můj mobilní telefon i notebook neustále připojen, takže jsem bez reklam kdykoliv, kdekoliv a bez ohledu na to, jaký webový prohlížeč zrovna použiji.

Nic ale není stoprocentní. Provozovatelé reklamních systémů si jsou vědomi toho, že lidé reklamy čím dál více blokují a snaží se vymýšlet různé obezličky, jak k lidem tu reklamu i tak docpat. Už se objevují reklamy z reklamních systémů, které běží na stejné doméně, jako je doména webu, který právě navštěvujete, a tam blokování pomocí DNS nepomůže. Nevím přesně, jak to funguje, ale obdivuji odvahu provozovatelů webů, kteří si něco takového spustí na svém serveru.
