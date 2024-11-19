# Postfix a mailová fronta

Když na Linuxu provozujete mailový server s největší pravděpodobností jako MTA ([Mail Transfer Agent](https://cs.wikipedia.org/wiki/Mail_Transfer_Agent)) budete používat Postfix. Pokud se mail z nějakého důvodu nedaří doručit, uloží se do mailové fronty (mail queue) a nezáleží na tom, jestli se mail nedaří doručit na vzdálený mail server nebo do lokální schránky. Pokud se ve frontě začíná hromadit větší množství mailů, je to jasná známka toho, že je něco špatně. Může to být problém s konektivitou serveru, problém s doručením do lokálních schránek, nebo některý z mailů byl hacknut, rozesílá spam a zprávy nejsou doručovány protože neprojdou antispamem na druhé straně nebo se odesílají na neexistující adresy.

V každém případě, na mail serveru je důležité frontu monitorovat. Frontu si zobrazíte následujícími příkazy:

```shell
mailq
```
nebo
```shell
postqueue -p
```

Pokud se nahromadí maily ve frontě, Postfix se je pokouší opakovaně doručit po určitých intervalech, když se mu to vůbec nepodaří, pošle informaci o nedoručení odesílateli. Když nastane problém s doručením, který se podaří vyřešit na straně serveru, je dobré Postfixu říct, ať se maily z fronty pokusí doručit hned. Na to existuje následující příkaz:

```shell
postqueue -f
```

Parametr -f znamená _Flush the queue_, což ve mě prvně evokovalo, že se maily z fronty prostě zahodí a bál jsem se ho použít. Ve skutečnosti příkaz provede přesně to co potřebujeme, znovu se pokusí doručit maily z fronty a pokud se je opět nepodaří doručit, ve frontě zůstanou.

Smazání zprávy z fronty se provádí pomocí příkazu postsuper s parametrem -d a identifikátorem zprávy:

```shell
postsuper -d mail_queue_id
```

Chcete-li smazat všechny zprávy z fronty:

```shell
postsuper -d ALL
```
