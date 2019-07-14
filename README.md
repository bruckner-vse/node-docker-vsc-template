# Template pro projekt node.js, Docker, VSC

V tomto projektu jsou úvodní soubory vhodné pro vývoj
- v node.js,
- ve Visual Studio Code,
- pro debugging a provoz v kontejneru Docker,
- primárně pro platformu Windows 10 (vývojářův desktop - host), ale funkční i na ostatních platformách.

Předpoklady:
Na desktopu povolena virtualizace (v BIOS) a nainstalováno a zprovozněno:
- git,
- Docker Desktop for Windows (kontejnery typu linux, povolen přístup k disku s projekty),
- Visual Studio Code (s doplňkem Docker),
- node.js (není nezbytné, protože debugging probíhá v kontejneru, ale může se hodit),
- npm (není nezbytné, ale může se hodit, např. pro inicializaci projektu).

Nutno upravit package.json údaji o projektu. Závislost na hello-world-npm lze odstranit, je tam jen pro příklad.

Soubor docker-compose.yml je určen pro nasazení na produkční prostředí, pro debugging je zde navíc docker-compose.override.yml, který některé části přepíše: Zpřístupní port 9229 pro připojení debuggeru z hosta (pozor bezpečnostní mezera: při debuggingu je tento port na windows hsotu přístupný i z lokální sítě), připojí adresář projektu pro přímý přístup z kontejenru (při změně kódu na hostu není nutné budovat nový kontejner), a změní spouštěcí příkaz na npm debug skript (napsaný v package.json). Dále spustí kontejner s jednotkovými testy, který pak běží, dokud není ukončen pomocí "docker-compose down", a sleduje složku app a podsložku test pomocí npm test skriptu "mocha --watch". Výsledky testů lze zobrazit zobrazením v terminálu VSC připojením ke kontejneru testy (zde "docker attach node-docker-vsc-template_testy_1" - název kontejneru vytváří docker-compose z názvu adresáře projektu a názvu služby v docker-compose.override.yml, aktuální lze zjistit pomocí "docker ps")

Dockerfile vychází z jakéhokoli komunitního node kontejneru (zde node:10-alpine - úsporná lts verze). Nejprve nakopíruje package.json a pomocí npm-install nainstaluje závislosti do složky node-modules, která existuje pouze v kontejneru (nikoli na hostu, který kód ani při debuggingu nespouští), a je jako zvláštní volume (kvůli jednoduššímu budování kontejneru při změně kódu). Poté nakopíruje obsah složky projektu z hostu, s výjimkou souborů uvedených v .dockerignore. Toto kopírování má smysl pouze při budování produkčního kontejneru, při debuggingu je složka mapována na host.

Soubor .vscode/launch.json definuje způsob spouštění debuggeru. Jde o připojení pomocí protokolu inspector na port 9229 localhostu, na kterém je dostupný kontejner node.js s aplikací. Dále definuje spuštění testů. Kontejnery se vždy při spuštění debuggeru (F5) spustí pomocí docker-compose up (spustí se tedy i všechny další případné kontejnery, které by aplikace používala).

---

Poznámky: Pro produkci je vhodné zvážit spouštění aplikace jako uživatel node (nikoli jako root) a pomocí node app.js (nikoli pomoci npm start), kvůli lepšímu odchytávání linuxovýxh ukončovacích signálů.

Možné problémy při práci s Windows, Docker a VSC:
- Pokud docker odmítá připojit lokální host drive do kontejneru, lze to vyřešit v Docker Desktop / Settings / Shared Drives / Reset Credentials. Je pak nutné znovu zadat heslo k windows účtu pro připojení disku.
- Pokud se ve VSC při otevřeném app.js a stisknutí F5 nespustí debugger na poprvé, je možné, že se první build buduje příliš dlouho. Stačí chvilku počkat a znovu zmáčknout F5, pak už by to mělo být v pořádku.
- Další nespecifické problémy lze řešit zastavením a odstraněním kontejneru projektu a odstraněním image projektu v powershellu nebo GIT BASHi (apod.), nebo restartováním Dockeru.

