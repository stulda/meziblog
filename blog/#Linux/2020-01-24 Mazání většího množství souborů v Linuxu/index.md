# Mazání většího množství souborů v Linuxu

Pokud potřebujete v Linuxu smazat větší množství souborů (řádově statisíce), klasický příkaz **rm** na to nestačí, protože není schopný přijmout tak velký seznam souborů jako argument a skončí s chybou **Argument list too long**

Běžně uváděným řešením je využití příkazu **find** na vyhledání souborů k mazání a následně předání názvů souborů hezky jednoho po druhém mazacímu příkazu **rm**.

```shell
find . -type f -exec rm -v {} \;
```
Ve finále ale můžeme na příkaz rm zcela zapomenout, find má parametr **-delete** a umí hezky mazat sám.

```shell
find . -type f -delete
```

Kromě toho, že příkaz na mazání je takto jednodušší a snadněji zapamatovatelný, je celkové mazání velkého množství souborů tímto způsobem i mnohem rychlejší.

Díky příkazu find pak můžeme také snadno smazat například jen soubory starší než 30 dnů:
```shell
find . -mtime +30 -delete
```

Nebo smazat jen soubory větší než 10MB:
```shell
find . -size +10M -delete
```

Chcete-li mazané soubory během mazání také vypsat, staří přidat parametr **-print**
