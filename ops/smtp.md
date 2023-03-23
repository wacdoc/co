# Custruite u vostru propiu servitore di mandatu di mail SMTP

## preambulu

SMTP pò cumprà direttamente servizii da i venditori di nuvola, cum'è:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali cloud push email](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Pudete ancu custruisce u vostru servitore di mail - mandatu illimitatu, costu generale bassu.

Quì sottu, dimustremu passu per passu cumu custruisce u nostru servitore di mail.

## Selezzione di u servitore

U servitore SMTP self-hosted richiede un IP publicu cù porti 25, 456 è 587 aperti.

I nuvuli publichi cumunimenti utilizati anu bluccatu questi porti per difettu, è pò esse pussibule apre l'emessu un ordine di travagliu, ma hè assai prublema dopu à tuttu.

Aghju ricumandemu di cumprà da un òspite chì hà questi porti aperti è sustene a creazione di nomi di domini inversi.

Quì, vi cunsigliu [Contabo](https://contabo.com) .

Contabo hè un fornitore di hosting basatu in Munich, Germania, fundatu in 2003 cù prezzi assai competitivi.

Se sceglite l'euro cum'è a valuta di compra, u prezzu serà più prezzu (un servitore cù memoria 8GB è CPU 4 custa circa 529 yuan à l'annu, è a tarifa di installazione iniziale hè gratuita per un annu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Quandu fate un ordine, rimarcate `prefer AMD` , è u servitore cù CPU AMD avarà un rendimentu megliu.

In u seguitu, pigliaraghju u VPS di Contabo cum'è un esempiu per dimustrà cumu custruisce u vostru servitore di mail.

## Cunfigurazione di u sistema Ubuntu

U sistema operatore quì hè Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Se u servitore in ssh mostra `Welcome to TinyCore 13!` (cum'è mostra in a figura sottu), significa chì u sistema ùn hè micca stallatu ancu. Per piacè disconnect ssh è aspettate qualchì minutu per accede di novu.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Quandu `Welcome to Ubuntu 22.04.1 LTS` appare, l'inizializazione hè cumpleta, è pudete cuntinuà cù i seguenti passi.

### [Opcional] Inizializza l'ambiente di sviluppu

Stu passu hè facultativu.

Per comodità, aghju messu l'installazione è a cunfigurazione di u sistema di u software ubuntu in [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Eseguite u cumandimu seguitu per installà cun un clic.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

L'utilizatori cinesi, per piacè utilizate u cumandimu seguitu invece, è a lingua, u fusu orariu, etc. serà automaticamente stabilitu.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo permette l'IPV6

Habilita IPV6 per chì SMTP pò ancu mandà email cù indirizzi IPV6.

edità `/etc/sysctl.conf`

Mudificà o aghjunghje e seguenti linee

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Segui cun [u tutoriale contabo: Aghjunghjendu a cunnessione IPv6 à u vostru servitore](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Edite `/etc/netplan/01-netcfg.yaml` , aghjunghje uni pochi di linii cum'è mostra in a figura sottu (U schedariu di cunfigurazione predeterminatu Contabo VPS hà digià sti linii, basta uncomment).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Allora `netplan apply` per fà chì a cunfigurazione mudificata hà effettu.

Dopu chì a cunfigurazione hè successu, pudete aduprà `curl 6.ipw.cn` per vede l'indirizzu ipv6 di a vostra reta esterna.

## Clona l'operazioni di repository di cunfigurazione

```
git clone https://github.com/wactax/ops.soft.git
```

## Generate un certificatu SSL gratuitu per u vostru nome di duminiu

L'invio di mail richiede un certificatu SSL per a criptografia è a firma.

Utilizemu [acme.sh](https://github.com/acmesh-official/acme.sh) per generà certificati.

acme.sh hè un strumentu di signatura di certificati automatizatu open source,

Entre in u magazzinu di cunfigurazione ops.soft, eseguite `./ssl.sh` , è un cartulare `conf` serà creatu in **u cartulare superiore** .

Truvate u vostru fornitore DNS da [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , editate `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Allora eseguite `./ssl.sh 123.com` per generà certificati `123.com` è `*.123.com` per u vostru nome di duminiu.

A prima corsa installerà automaticamente [acme.sh](https://github.com/acmesh-official/acme.sh) è aghjunghje un compitu programatu per rinnuvamentu automaticu. Pudete vede `crontab -l` , ci hè una linea cusì cusì.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

U percorsu per u certificatu generatu hè qualcosa cum'è `/mnt/www/.acme.sh/123.com_ecc。`

U rinnuvamentu di u certificatu chjamarà u script `conf/reload/123.com.sh` , edità stu script, pudete aghjunghje cumandamenti cum'è `nginx -s reload` per rinfriscà a cache di certificatu di l'applicazioni rilativi.

## Custruite un servitore SMTP cù chasquid

[chasquid](https://github.com/albertito/chasquid) hè un servitore SMTP open source scrittu in lingua Go.

Cum'è un sustitutu di l'antichi prugrammi di u servitore di mail cum'è Postfix è Sendmail, chasquid hè più simplice è faciule d'utilizà, è hè ancu più faciule per u sviluppu secundariu.

Eseguite `./chasquid/init.sh 123.com` serà installatu automaticamente cun un clic (sustituisci 123.com cù u vostru nome di duminiu di mandatu).

## Configurate e-mail Signature DKIM

DKIM hè utilizatu per mandà e-mail signatures per impediscenu e lettere da esse trattate cum'è spam.

Dopu à u cumandimu corre cù successu, vi sarà dumandatu à stabilisce u record DKIM (cum'è mostra sottu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Basta aghjunghje un record TXT à u vostru DNS (cum'è mostra quì sottu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Vede u statutu di u serviziu è i logs

 `systemctl status chasquid` Vede u statu di serviziu.

U statu di funziunamentu normale hè cum'è mostra in a figura sottu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` o `journalctl -xeu chasquid` pò vede u logu di errore.

## Cunfigurazione inversa di u nome di duminiu

U nome di duminiu inversu hè di permette l'indirizzu IP per esse risoltu à u nome di duminiu currispundenti.

Stabbilimentu di un nome di dominiu inversu pò impedisce l'e-mail da esse identificatu cum'è spam.

Quandu u mail hè ricivutu, u servitore di ricivutu realicerà l'analisi di u nome di dominiu inversu nantu à l'indirizzu IP di u servitore di mandatu per cunfirmà s'ellu u servitore di u mandamentu hà un nome di duminiu inversu validu.

Se u servitore di mandatu ùn hà micca un nome di dominiu inversu o se u nome di dominiu inversu ùn currisponde à l'indirizzu IP di u servitore di mandatu, u servore chì riceve pò ricunnosce l'email cum'è spam o rifiutà.

Visita [https://my.contabo.com/rdns](https://my.contabo.com/rdns) è cunfigurà cum'è mostra quì sottu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Dopu avè stabilitu u nome di dominiu inversu, ricordate di cunfigurà a risuluzione in avanti di u nome di duminiu ipv4 è ipv6 à u servitore.

## Edite u nome di host di chasquid.conf

Mudificà `conf/chasquid/chasquid.conf` à u valore di u nome di duminiu inversu.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Allora eseguite `systemctl restart chasquid` per riavvia u serviziu.

## Backup conf à u repository git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Per esempiu, aghju una copia di salvezza di u cartulare conf à u mo propiu prucessu github cum'è seguita

Crea prima un magazzinu privatu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Entre in u cartulare conf è sottumette à u magazzinu

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Aghjunghjite u mittente

corre

```
chasquid-util user-add i@wac.tax
```

Pò aghjunghje un mittente

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Verificate chì a password hè stallata currettamente

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Dopu aghjunghje l'utilizatore, `chasquid/domains/wac.tax/users` serà aghjurnatu, ricordate di mandà à u magazzinu.

## DNS aghjunghje un record SPF

SPF (Sender Policy Framework) hè una tecnulugia di verificazione di e-mail aduprata per prevene u fraudulente email.

Verifica l'identità di un mittente di mail cuntrollanu chì l'indirizzu IP di u mittente currisponde à i registri DNS di u nome di dominiu chì pretende esse, impediscendu à i fraudsters di mandà email falsi.

L'aghjustà di i registri SPF pò impedisce chì e-mail sò identificate cum'è spam quant'è pussibule.

Se u vostru servitore di nome di duminiu ùn sustene micca u tipu SPF, aghjunghje solu un registru di tipu TXT.

Per esempiu, u SPF di `wac.tax` hè a siguenti

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF per `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Nota chì aghju `include:_spf.google.com` quì, questu hè perchè cunfigurà `i@wac.tax` cum'è l'indirizzu di mandatu in a casella di posta di Google più tardi.

## Cunfigurazione DNS DMARC

DMARC hè l'abbreviazione di (Autentificazione, Reporting & Conformance di Missaghju basatu in Dominiu).

Hè utilizatu per catturà i rimbalzi SPF (forsi causati da errori di cunfigurazione, o qualcunu hè fintendu di esse voi per mandà spam).

Aggiungi record TXT `_dmarc` ,

U cuntenutu hè a siguenti

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

U significatu di ogni paràmetru hè a siguenti

### p (Politica)

Indica cumu gestisce e-mail chì fallenu a verificazione SPF (Sender Policy Framework) o DKIM (DomainKeys Identified Mail). U paràmetru p pò esse stabilitu à unu di trè valori:

* nimu: Nisuna azione hè fatta, solu u risultatu di verificazione hè rimbursatu à u mittente per mezu di u mecanismu di rapportu email.
* Quarantine: Mettite u mail chì ùn hà micca passatu a verificazione in u cartulare di puzzicheghju, ma ùn rifiutarà micca u mail direttamente.
* rifiutà: rifiutà direttamente e-mail chì fallenu a verificazione.

### fo (Opzioni di fallimentu)

Specifica a quantità di infurmazione restituita da u mecanismu di rapportu. Pò esse stabilitu à unu di i valori seguenti:

* 0: Segnala i risultati di validazione per tutti i missaghji
* 1: Solu signalà i missaghji chì fallenu a verificazione
* d: Sulu rapportu fallimenti verification nomu duminiu
* s: signalate solu i fallimenti di verificazione SPF
* l: Segnala solu i fallimenti di verificazione DKIM

### rua & ruf

* rua (URI di rapportu per i rapporti aggregati): Indirizzu email per riceve rapporti aggregati
* ruf (Reporting URI for Forensic reports): indirizzu email per riceve rapporti detallati

## Aghjunghjite i registri MX per rinvià e-mail à Google Mail

Perchè ùn aghju micca pussutu truvà una cassetta postale corporativa gratuita chì sustene l'indirizzi universali (Catch-All, pò riceve ogni email mandatu à stu nome di duminiu, senza restrizioni à i prefissi), aghju utilizatu chasquid per rinvià tutti i mail à a mo mailbox di Gmail.

**Sì avete a vostra propria casella postale di cummerciale pagata, per piacè ùn mudificà micca u MX è saltate stu passu.**

Edite `conf/chasquid/domains/wac.tax/aliases` , stabilisce a cassetta postale di spedizione

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` indica tutti l'e-mail, `i` hè u prefissu di l'indirizzu email di l'utilizatore chì hà inviatu creatu sopra. Per rinvià u mail, ogni utilizatore hà bisognu di aghjunghje una linea.

Allora aghjunghje u record MX (puntu direttamente à l'indirizzu di u nome di duminiu inversu quì, cum'è mostra in a prima linea in a figura sottu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Dopu chì a cunfigurazione hè cumpleta, pudete aduprà altre indirizzi email per mandà email à `i@wac.tax` è `any123@wac.tax` per vede s'ellu pudete riceve email in Gmail.

Se no, verificate u log chasquid ( `grep chasquid /var/log/syslog` ).

## Mandate un email à i@wac.tax cù Google Mail

Dopu chì Google Mail hà ricevutu u mail, naturalmente sperava di risponde cù `i@wac.tax` invece di i.wac.tax@gmail.com.

Visita [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) è cliccate "Aggiungi un altru indirizzu email".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Dopu, entre u codice di verificazione ricevutu da l'email chì hè statu trasmessu.

Infine, pò esse stabilitu cum'è l'indirizzu di u mittente predeterminatu (inseme cù l'opzione di risponde cù u stessu indirizzu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

In questu modu, avemu cumpletu u stabilimentu di u servitore di mail SMTP è à u stessu tempu aduprà Google Mail per mandà è riceve email.

## Mandate un email di prova per verificà se a cunfigurazione hè successu

`ops/chasquid`

Esegui `direnv allow` di installà dipendenze (direnv hè statu stallatu in u prucessu di inizializazione di una chjave precedente è un ganciu hè statu aghjuntu à a cunchiglia)

poi corre

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

U significatu di i paràmetri hè a siguenti

* user: nome d'utilizatore SMTP
* pass: password SMTP
* à : destinatariu

Pudete mandà un email di prova.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Hè cunsigliatu di utilizà Gmail per riceve e-mail di prova per verificà se e cunfigurazioni sò successu.

### Cifratura standard TLS

Cum'è mostra in a figura sottu, ci hè stu picculu serratura, chì significa chì u certificatu SSL hè statu attivatu bè.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Dopu cliccate "Mostra e-mail originale"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Comu mostra in a figura sottu, a pagina di mail originale di Gmail mostra DKIM, chì significa chì a cunfigurazione DKIM hè successu.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Verificate u Ricevutu in l'intestazione di l'email originale, è pudete vede chì l'indirizzu di u mittente hè IPV6, chì significa chì IPV6 hè ancu cunfiguratu bè.
