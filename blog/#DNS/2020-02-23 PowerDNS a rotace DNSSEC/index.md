# PowerDNS a rotace DNSSEC klíčů

Pokud je doména jednou pomocí **DNSSEC** zabezpečena, tak to bohužel neznamená, že už se o to není třeba dál starat. DNSSEC klíče je dobré jednou za čas vyměnit (doporučuje se 1x za rok), snižuje se tím šance na jejich kompromitování. SSL certifikáty pro weby jsou také vystavovány jen na omezenou dobu. U webů stačí vystavit nový certifikát, vyměnit ho na webserveru a je hotovo. S DNS je to trochu složitější, není možné otisk klíče z nadřazené zóny jednoduše smazat a nahradit ho jiným, protože DNS se různě cachuje, záznamy mají nastavenu dobu platnosti (TTL) a změna se neprojeví všude hned.

Z tohoto důvodu se provádí rotace klíčů (v angličtině Key Rollover), kdy se vystaví nový klíč, ale původní stále zůstává v platnosti, takže nějakou dobu jsou v nadřazené zóně uvedeny dva otisky klíčů a až když je jistota, že všechny resolvery ověřují vůči novému klíči, je možné starý smazat.

Jako první tedy v PowerDNS přidáme nový (zatím neaktivní) klíč do zóny:

```shell
pdnsutil add-zone-key meziblog.cz ksk inactive
```

Otisk tohoto klíče přidáme do KEYSETu domény u našeho registrátora nebo pokud se používá automatická správa klíče pomocí CDNSKEY záznamu, tak stačí počkat až správce domény zaregistruje, že je v zóně nový klíč a přidá ho do KEYSETu automaticky.

Když už je otisk nového klíče publikován v nadřazené zóně, je potřeba počkat pár dní nebo klidně i týden, abychom měli jistotu, že bude nový otisk klíče dostupný všude.

Nyní provedeme samotnou rotaci klíčů, nový klíč označíme jako aktivní a ten starý jako neaktivní.

```shell
pdnsutil activate-zone-key meziblog.cz ID-noveho-klice
pdnsutil deactivate-zone-key meziblog.cz ID-stareho-klice
```

ID klíčů zjistíme pomocí příkazu show-zone:

```shell
pdnsutil show-zone meziblog.cz
```

Po provedené rotaci klíčů je nutné opět několik dní počkat a až poté můžeme smazat starý klíč:

```shell
pdnsutil remove-zone-key meziblog.cz ID-stareho-klice
```

Následně můžeme smazat i otisk starého klíče z nadřazené zóny. Opět v případě použití CDNSKEY záznamu, smazání provede správce domény automaticky.

Celý tento proces a jeho správné provedení je pro fungování DNSSEC dost zásadní a pár registrátorům/hostingům se již stalo, že chybou při rotaci klíčů odstavili na nějakou dobu velké množství domén. Já doufám, že se mi nic takového nikdy nestane a budu se snažit rotaci klíčů maximálně automatizovat.
