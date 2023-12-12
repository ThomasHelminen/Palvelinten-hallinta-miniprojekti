# Palvelinten-hallinta-miniprojekti 'peliluola'
Tervetuloa! Täältä löydät miniprojektini kokonaan uuden version Tero Karvisen vetämälle Palvelinten hallinta -kurssille, tervetuloa! Kyseessä on tehtävä 'h7', ja kurssi löytyy [täältä](https://terokarvinen.com/2023/configuration-management-2023-autumn/). 

"Vanha" versio tästä projektista kaikkine ongelmineen löytyy [täältä](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/blob/main/Miniprojektin_eka_versio.md).

Yritän käyttää tämän miniprojektin teossa kurssilta oppimiani asioita. Haluan oppia myös uutta, niin tulen käyttämään myös sellaisia komentoja, joita en ole käyttänyt aikaisemmin. Aiemmat kurssitehtävät löydät minun '[Palvelinten-hallinta -kurssi](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi)' repositorysta!

Käytän projektissa Vagrantia sekä herra-orja -arkkitehtuuria (herrakone ja kaksi orjakonetta). Luon tähän projektiin uudet virtuaalikoneet samalla tavalla kuin aiemmin kurssitehtävssä '[h2 karjaa, kohta d)](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi/blob/main/h2-karjaa.md)'.

Miniprojektin ideana on asentaa komentorivipelejä herralta orjakoneille käyttäen Saltia. Kun orjakoneilla pelataan peliä, niin tulostiedot eli, ns. leaderboardit siirtyvät herrakoneelle crontabin avulla tietyn väliajoin. Näin herrakoneelta pystyisi seuraamaaan kunkin orjakoneen tuloksia ja ennätyksiä "etänä". Katsotaan päästäänkö sinne asti!

Työkoneena toimii HP:n Elitebook 830 G5 (speksit: Windows 11 Pro versio 10.0.22631, Intel Core i5-8350U, 16GB RAM, 256GB SSD, Vagrant versio 2.4.0, VirtualBox versio 7.0.12). Kaikki kuvankaappaukset ovat minun ottamia, ellen toisin mainitse. Tervetuloa mukaan!

## Miniprojektin aloitus ja virtuaalikoneiden "alustus"

Aloitetaan tehtävä luomalla uusi hakemisto nimeltä 'h7-miniprojekti' isäntäkoneen C:\Users\Tomwh\ -hakemistoon. Siirryn komentorivillä tähän hakemistoon komennolla ``C:\Users\Tomwh\h7-miniprojekti``. Suoritan komennon ``vagrant init debian/bullseye64``, joka luo VagrantFilen tähän hakemistoon. Voidaan varmistaa tämä vielä listaamalla hakemiston sisältö komennolla ``ls``.

![vagrant-init](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/85d5eace-807d-4c7a-bab5-41b22afa3db4)

Nyt kun VagrantFile on olemassa, niin mennään muokkaamaan sitä GUI:n kautta. Käytän Karvisen [artikkelista](https://terokarvinen.com/2023/salt-vagrant/) löytyvää koodia tähän tiedostoon. Muutin koneiden nimet, muuten koodi on sama. Eli herrakone on nimeltä 'master' ja orjakoneet 'pelaaja1' ja 'pelaaja2'. VagrantFile näyttää nyt tältä:

![vagrantfile](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/550277fe-c6b0-4f09-b1cc-52f1ef9ecc04)

Nyt VagrantFilen muuttamisen jälkeen on aika käynnistää nämä virtuaalikoneet. Ajan komennon ``vagrant up``.

![vagrant-up](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/216203df-c51b-4b1a-9a96-e9a9cc837d9c)

Nyt alkoi koneiden määrittäminen. Katsotaan kuinka kauan tässä menee aikaa... Lopulta tähän kului noin kuusi minuuttia. Muodostetaan SSH-yhteys masteriin komennolla ``vagrant ssh master``. Onnistuneen yhteyden muodostamisen jälkeen ajan komennon ``$ sudo salt-key -A``, eli hyväksytän pelaajien avaimet. Tsekataan vielä, että herra-orja yhteys toimii komennolla ``$ sudo salt '*' test.ping``.

![ssh-master-salt-key-ping](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/956f794b-0c14-4213-8b4c-df15d853f472)

Toimii! Ajetaan vielä komennot ``$ sudo apt update`` ja ``$ sudo apt upgrade -y``.

![master-sudo-paketit](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/f40ec406-981e-4396-87a5-851e7b075421)

Nyt on paketit ajan tasalla. Haluan vielä varmistaa, että salt-master ja salt-minion ovat asentuneet virtuaalikoneiden määrityksen yhteydessä. Eli ajan masterilla komennon ``$ salt-master --version`` ja pelaaja1:lla (orjakoneella) ``$ salt-minion --version``.

![master-salt-version](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/9857df5d-b12e-41b9-95e1-a61dee86d434)

![pelaaja-salt-version](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/497d1d41-74d2-46e1-80f4-380489b16256)

Bueno. Ajattelen nyt vielä ajaa komennot ``$ sudo apt update`` ja ``$ sudo apt upgrade -y`` molemmilla pelaajilla. Paketit ovat nyt kunnossa.

## Suolaa

Nyt kun pohjatyöt ovat valmiina, siirrytään takaisin masterille ja luodaan uusi 'peliluola'-hakemisto komennolla ``sudo mkdir -p /srv/salt/peliluola``.

![mk-dir-peliluola](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/f0d29c2a-ac0b-450e-80d3-fad594279ab9)

Siirrytään tähän hakemistoon ja luodaan init.sls -tiedosto. Tämä tapahtuu komennolla ``sudo nano init.sls``. Ajetaan komento. Avautuu tyhjä nano-tekstieditori. Kirjoitan seuraavan koodin sinne:

![asennus-init-sls](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/926a191d-0cd1-4fec-8f2a-64099888f6e2)

Eli käytetään pkg.installed tilafunktiota, ja asennetaan kaksi peliä. Tallennetaan ja suljetaan tämä tiedosto. Liikutaan nyt yksi hakemisto taaksepäin hakemistoon /srv/salt/ ja luodaan tänne vielä top.sls -tiedosto. Lisään tiedostoon seuraavan koodin:

![top-sls](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/aa4852ed-071c-4c98-9a6c-493101179865)

Tallennetaan tiedosto ja suljetaan se. Nyt sormet ristiin. Tässä ei pitäisi mennä nyt mikään pieleen. Eilen aamulla yritin tehdä tosiaan tätä projektia hieman monimutkaisemmin, mutta taisin haukata liian ison palan osaamistasooni nähden. Joka tapauksessa, suoritetaan komento ``sudo salt '*' state.apply peliluola/``:

![state-apply](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/fbb72c08-2d4e-43e0-87d6-b13114805356)

Ei virheilmoituksia! Natsaa. Käydään varmistamassa vielä toisella pelaajalla, että pelit ovat todellakin asentuneet. Siirryn pelaaja1:lle ja suoritan komennon ``$ bastet``. Peli käynnistyy!

![bastet-pelaaja1](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/74f6a5a1-15f9-43be-b20a-8cd0005a0ba5)

Pelataan tämän kunniaksi peli tätä kaunista komentorivitetristä.

![bastet-highscore-pelaaja1](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/0964d8fd-202e-42a2-8eec-7f00ea3cd88a)

Käydään tarkistamassa vielä pelaaja2:lta, että moon-buggy -peli käynnistyy! Siirryn pelaaja2:lle ja ajan komennon ``$ moon-buggy``. Peli todella käynnistyy!

![moon-buggy-pelaaja2](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/103e89af-78ca-4f8e-9527-ffbf89383f1b)

Pelataan yksi peli tätäkin nyt, jotta saadaan jokin tulos...

![moon-buggy-pelaaja2-tulos](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/567e2501-c189-4ecb-b81b-0a30cae5e492)

Hyvä homma. Saimme käytettyä Saltia pelien asennukseen. Seuraavaksi yritetään saada tulokset siirrettyä säännölisin väliajoin masterille...

## cron, crontab ja salt.modules.cp.push, mitä ne ovat? 

Tosiaan tämä oli minulle myös hyvä kysymys. Etsin netistä tietoa, millä "työkalulla" voisi lähettää tietoa tai tiedostoja säännöllisesti. No vastaus löytyi. Vapaasti suomennettuna cron on Unix-, solaris- ja Linux-apuohjelma, jonka avulla cron-daemon voi suorittaa tehtäviä automaattisesti taustalla säännöllisin väliajoin. ([Admin's Choice](https://www.adminschoice.com/crontab-quick-reference)). Tässä artikkelissa kerrotaan myös miten cronia käytetään. Siitä lisää kohta.

Etsitään nyt pelien tulosten tiedostot. Etsin pelaaja1:n hakemistoja läpi. Löysin lopulta pelihakemiston. Se löytyi polusta /var/games. Tässä on suoraan 'bastet.scores2' -tiedosto. Avataan se, ja katsotaan löytyykö sieltä aikaisemmin pelattua kierrosta:

![bastet-scores](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/d5fd14a5-f722-4994-9575-5a399291f4b6)

Sieltä löytyi!

/var/games/ -hakemistossa on vielä moon-buggy -hakemisto. Siirrytään sinne.

![mbscore](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/39fd35f4-679c-45f9-9c63-9028023cd5b7)

Sieltä löyty moon-buggyn tulostiedosto. Nyt on molempien polut selvillä. Tämän projektin ensiyrityksessä tuli selväksi miksi en pystynyt käyttämään saltin salt.modules.cp.push tilaa crontabin kanssa. Tosiaan masterin konfigurointitiedostosta pitää muuttaa 'file_recv = True' ja käynnistää master uusiksi ([SaltProject](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html)), eli tehdään se nyt.

Siirrytään masterille ja liikutaan konfiguraatiotiedoston sisältävään hakemistoon komennolla ``$ /etc/salt/``. Tiedosto löytyi ja grepataan sieltä vielä tuo 'file_recv' ja tarkistetaan onko se False vai True. Eli ajetaan komento ``$ grep 'file_recv:' master``

![master-grep](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/9e6ce079-ed06-4e28-8ee1-3613cb3c3fb7)

Falsehan se on. Avataan tiedosto vim:llä. No eipä avatakkaan. vim-komentoa ei löydy. Ahaa. Asennetaan vim ensiksi! Ajan komennon ``$ sudo apt install vim -y``. 

![vim-install](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/38956230-dd8c-4883-8589-8e2ab12e9e4c)

Vim asentui ja nyt on aika yrittää äskeistä komentoa uusiksi. Eli ``sudo vim master``. Vim avautui. Painoin ``/``, jolla saa vimin haun auki. Etsin tästä pitkästä tiedostosta 'file_re' nimellä sen jälkeen. Löytyi! 

![file-recv-true](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/9fcf9390-b946-4ad2-a759-f11377b28e7c)


``i``:tä painalla pääsee insert-tilaan. Otan tuosta risuaidan pois (koska se on nyt kommenttina) ja muutan arvon Falsesta -> True. Tallennan ja poistun vimistä painalla ``esc`` ja ``:wq``.

![file-recv-true-vaihto](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/e3c91f4e-62c4-42db-a3ae-75597d4cd213)

Käynnistän Vagrantin nyt uusiksi. Eli ensiksi ``$ exit``, jolloin SSH-yhteys katkeaa, ja sitten ``vagrant halt``.

![vagrant-restart](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/0d87f5b9-872b-42fb-8cd6-18ed7a3c5f68)

Noniin, koneet ovat taas uudelleen käynnissä. Siirrytään taas /etc/salt/ -hakemistoon ja grepataan 'file_recv' sieltä. Eli taas sama komento kuin äsken ``$ grep 'file_recv:' master``.

![file-recv-true-ok](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/7a8aa6b8-7ea9-429a-ad93-2ba03646f273)

True! Nyt pelaajan (orjan) pitäisi pystyä puskemaan tiedostoja masterille. Vapaasti suomennettuna: Työnnä tiedosto kätyriltä isännälle, tiedosto tallennetaan salt-masterin masterin kätyritiedostojen välimuistiin (oletus on /var/cache/salt/master/minions/minion-id/files)" ([SaltProject](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html))

## crontabin kimppuun

Nyt kun tosiaan tiedämme pelien tulosten tiedostosijainnit, niin voimme alkaa käyttämään crontabia. Aloitan näyttämällä kaikki parametrivaihtoehdot crontabille:

![crontab-usage](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/8551cabf-af9a-4d70-9730-7bf4af5bb9fd)

Nyt on aika suorittaa crontabin muokkauskomento, eli ``$ crontab -e``. Ensimmäisellä kerralla komentorivi kysyy millä tekstieditorilla crontab avataan. Valitsin nanon, eli painoin numeroa 1.

![crontab-editor](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/62b81dde-38be-4ad0-97de-96d222c0a037)

Nyt on vuorossa laittaa koodi crontabiin. Seurasin Admin's Choicen [artikkelia](https://www.adminschoice.com/crontab-quick-reference), sekä SaltProjectin [artikkelia](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html) ja lisäsin seuraavan koodin joka ajaa cronia joka minuutti:

![crontab-edit](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/058f41e3-cfbb-4dbc-9cf0-424b28360eb6)

Eli kuvassa näkyy * * * * * ( tunnit, minuutit, kuukaudenpäivät, kuukaudet, päivä viikossa (0-6, 0 = sunnuntai (Admin's Choice)). Jätin kaikki tähdet paikoilleen, eli cron ajaa kyseisen saltin tilan määritellylle tiedostolle joka minuutti. Tässä tapauksessa ne olivat molemmat tiedostot, joista näkyvät pelien tulokset. Cron ajaa nämä molemmille pelaajille, eli tiedostot ovat pelaajakohtaiset. Tiedostojen pitäisi löytyä masterilta joko /var/cache/salt/master/minions/pelaaja1 TAI pelaaja2/files/var/games/ ja /moon-buggy hakemistoista.

Käydään pelaamassa muutama peli moon-buggya pelaaja2:lla. Pidän nyt vajaan kymmenen minuutin tauon. Pelaan vielä yhden pelin.

Noniin. Mennään takaisin masterille ja siirrytään pelaaja2:n "tiedostoihin". Eli siirrytään sinne komennolla ``$ cd /var/cache/salt/master/minions/pelaaja2/files/var/games/moon-buggy`` ja grepataan täällä 'pelaaja2' mbscore-tiedostosta! Nyt täällä pitäisi olla äskeisten pelien tulokset! Tätä hetkeä ollaan nyt rakennettu kolmisen tuntia kaiken kaikkiaan tällä vedolla...

![Näyttökuva 2023-12-12 034423](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/12ec8522-b094-4343-b002-70fb496481d5)

Sieltä löytyy! En ole koskenut siirtänyt mitään tiedostoja sen jälkeen kun lisäsin koodia crontabiin. Tätä hetkeä olen odottanut. Vihdoin onnistuminen. Testataan vielä pelaaja1:lla bastet-peli. Siirrytään pelaaja1:lle ja pelataan pari peliä.

Pelit pelattu. Käydään tarkastamssa miltä tilanne näyttää masterin hakemistossa:

![bastet-tulokset-pelaaja1](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/af969fc2-26c6-455a-894d-7f2ece780307)


Ja sieltä! Vamos! Miniprojektini haluttu tehtävä on vihdoin suoritettu! Nyt masteri pystyy katsotmaan kuinka hyviä pelaajia pelaaja1 ja pelaaja2 oikein ovat... :D. Nyt vielä joitain loppusanoja...

## Loppusanat

Nyt on aika laittaa ns. hokkarit naulakkoon tämän kurssin osalta. Tämä kurssi on vaatinut todella paljon aikaa, itselle välillä jopa vähän liikaakin. Tällä kurssilla on tullut opittua samalla eniten verrattuna muihin kursseihin. Olen tykännyt tehtävistä, ja siitä miten opetus on tapahtunut. Olisin toivonut lähiopetusta, koska luulen, että olisin saanut siitä vielä enemmän irti. Nyt kun on pyöritelty Vagrantia suhteellisen paljon, niin Linuxin terminaali on tullut myös erityisen tutuksi. Ja siitä tykkään. 

Voin suositella tätä kurssia niin lämpimästi vain kuin voin. Nyt jälkikäteen ne monet turhautumisen hetket tällä kurssi eivät tunnu enää lainkaan pahoilta. Nyt tajuaa, että tämän parin kuukauden aikana on oikeasti opittu. Kiitos.

-Thomas

## Lähdeluettelo
- Admin's Choice, s.a.. Crontab - Quick Reference. Luettavissa: https://www.adminschoice.com/crontab-quick-reference. Luettu 12.12.2023.
- Karvinen, T. 13.10.2023. Infra as Code 2023. Luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/. Luettu 11.12.2023.
- Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/. Luettu 12.12.2023.
- SaltProject, s.a.. salt.modules.cp. Luettavissa: https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html. Luettu 11.12.2023.
