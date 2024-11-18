# Swap v souboru na Linuxu

Swapování je proces práce s pamětí RAM, kdy se část dat z paměti odleje na disk, aby se v paměti uvolnilo místo. Na Linuxu je swap zpravidla používán, jako vyhrazený diskový oddíl, ale jsou situace, kdy se hodí zapnout swapování do souboru. Klasický příklad, když si pořídíte laciný virtuální server s málo paměti a poskytovatel spustil virtuál bez swapu. Aby se předešlo případným pádům aplikace kvůli nedostatku RAM, aktivujete si swap v souboru na disku.

Nejprve si pomocí nástroje fallocate vytvoříme soubor, který bude použitý jako swap:

```shell
fallocate -l 1G /swapfile
```

Je možné použít i příkaz dd:

```shell
dd if=/dev/zero of=/swapfile bs=1024 count=1048576
```

Soubor swapfile o velikosti 1GB je připraven na disku a ještě je potřeba ho "naformátovat" jako swap:

```shell
mkswap /swapfile
```

Nastavíme mu správná oprávnění, aby se souborem mohl pracovat pouze root:

```shell
chmod 600 /swapfile
```

Nyní už samotná aktivace swapu:

```shell
swapon /swapfile
```

V tuto chvíli je již swap v systému používán. Pokud je potřeba ho zase vypnout, tak k tomu slouží obdobný příkaz `swapoff`.

Aby se swap aktivoval automaticky i po restartu počítače, stačí do souboru /etc/fstab přidat následující řádek:

```
/swapfile swap swap defaults 0 0
```

Na závěr ještě poznámka ke swapování obecně. Dlouho jsem si myslel, že pokud má počítač dostatečně velkou paměť, kde nehrozí, že paměť dojde, je použití swapu naprosto zbytečné. Nemusí to být vždy pravda. Paměť RAM kromě aplikací používá i operační systém jako diskovou cache a mohou nastat situace, kdy se vyplatí paměť vyplnit více diskovou cache, než v ní mít data z aplikací, která nejsou v daný moment používána.

