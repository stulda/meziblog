# Jetbrains vývojové nástroje a kontrola českého pravopisu

Ačkoliv firma **JetBrains** má své kořeny v Praze, není český jazyk v jejich produktech ve výchozím stavu podporován. Zejména kontrola pravopisu v komentářích a textových řetězcích je pro mnohé programátory velmi užitečná. Naštěstí je možné si český slovník pro kontrolu pravopisu do Jetbrains vývojových nástrojů snadno přidat.

Do Jetbrains IDE lze nahrát slovník pro kontrolu pravopisu ve formátu `.dic`. Což je jednoduchý textový soubor, kde každé slovo slovníku je na samostatném řádku.

Na internetu jsou ke stažení různé slovníky, ale mě se nejvíce osvědčilo vygenerovat si slovník z balíčku `aspell`, který je dostupný v repositářích většiny linuxových distribucí a díky Homebrew i na macOS.

Instalace balíčku `aspell` v Debianu a jeho derivátech:
```shell
sudo apt install aspell aspell-cs
```

Instalace balíčku `aspell` na macOS pomocí Homebrew:
```shell
brew install aspell
```

Vygenerování slovníku do souboru `cs.dic`:
```shell
aspell --lang cs dump master | aspell --lang cs expand | tr ' ' '\n' > cs.dic
```

Soubor `cs.dic` pak stačí vybrat v Jetbrains IDE v nastaveních pod položkou `Editor` -> `Spelling` -> `Dictionaries`
