# Palvelinten-hallinta-miniprojekti
Täältä löydät miniprojektini Tero Karvisen vetämälle Palvelinten hallinta -kurssille, tervetuloa! Kyseessä on tehtävä 'h7', ja kurssi löytyy [täältä](https://terokarvinen.com/2023/configuration-management-2023-autumn/). Yritän käyttää tämän miniprojektin teossa pääsääntöisesti kurssilta oppimiani asioita. Aiemmat kurssitehtävät löydät minun '[Palvelinten-hallinta -kurssi](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi)' repositorysta!

Käytän projektissa Vagrantia sekä herra-orja -arkkitehtuuria (herra-kone ja kaksi orjakonetta), jonka loin aiemmin kurssitehtävssä '[h2 karjaa, kohta d)](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi/blob/main/h2-karjaa.md').

Miniprojektin ideana olisi asentaa komentorivipelejä herralta orjakoneille käyttäen Saltia. Kun orjakoneilla pelataan peliä, niin tulostiedot ns. leaderboardit siirtyvät herra-koneelle cronin avulla tietyn väliajoin. Näin herra-koneelta pystyisi seuraamaaan tuloksia ja ennätyksiä "etänä". Katsotaan päästäänkö sinne asti!

Työkoneena toimii HP:n Elitebook 830 G5 (speksit: Windows 11 Pro versio 10.0.22631, Intel Core i5-8350U, 16GB RAM, 256GB SSD, Vagrant versio 2.4.0, VirtualBox versio 7.0.12). Kaikki kuvankaappaukset ovat minun ottamia, ellen toisin mainitse. Tein kaikki pakolliset tehtävät sekä muutaman vapaaehtoisen tehtävän.  Tervetuloa mukaan!

## Miniprojekti

Lähdetään liikkeelle avaamalla isäntäkoneesta PowerShell järjestelmävalvojana. Siirrytään saldemo-kansioon, josta löytyy aiemmin luotu vagrantfile. Täällä ollessa ajetaan komento  ``vagrant up`` joka käynnistää virtuaalikoneet.

![vagrant-up](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/45c1cd31-f925-4a8b-aa03-b4467295751c)

Virtuaalikoneiden herättyä, otetaan SSH-yhteys tmasteriin (herrakoneseen) komennolla ``vagrant ssh tmaster``

![ssh-tmaster](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/d8ea24d4-f1fd-4ff6-8567-219505deacc8)

Siirrytään nyt herrakoneella /srv/salt/ -hakemistoon komennolla ``$ cd /srv/salt/`` ja luodaan tänne uusi hakemisto komennolla ``$ sudo mkdir pelit``. Siirrytään pelihakemistoon komennolla ``$ cd pelit/``. Lopuksi voimme vielä ajaa komennon ``$ pwd`` joka tulostaa nykyisen työhahekemiston polun.

## !EDIT! Neljän työtunnin jälkeen tilanne näyttää toivottomalta. Aika loppuu kesken ja omat työt alkavat. Tämä miniprojekti jää kesken ainakin täksi päiväksi.

Aloitin luomalla init.sls tiedoston tähän hakemistoon:

![pelit-sls-tiedosto](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/d955dccf-d2be-4a2c-8b2d-cba9a183b1ea)

Tämän pitäisi asentaa kyseiset pelit orjakoneille, tallentaa pelin tulokset herrakoneen /var/lib/ hakemistoon. Käytin tässä Jinja2-syntaksia, koska pelaajan id on dynaaminen tieto. Cron asennetaan varmuudeksi ja luodaan cronille tiedosto nimeltä pelitulokset. Sinne lisätään sisältö, eli se lähettää pelitulokset 30 minuutin välein. Ainakin teoriassa... require-osio varmentaa, että peli on asennettu, ennen kuin tätä tilaa aletaan suorittamaan. 

Loin myös top.sls -tiedoston /srv/salt/ -hakemistoon. se näyttää tältä:

![top-sls-tiedosto](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/2c3ca261-fa0f-4264-8b72-efc67819ace0)

Yritetään ajaa nyt tämä luotu init.sls tiedosto. Ja se onnistuukin helposti komennolla ``$ sudo salt '*' state.apply pelit``.

![pelit-jinja-error](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/3086fe84-ef93-4c78-902d-6cdcb0eaa4a8)

Virhettä pukkaa. Onneksi tämä oli helppo korjata! Rivillä 27 oli ] vaikka olisi pitänyt olla }}.

![pelit-sls-tiedosto-korjaus](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/81ab77af-7d86-482d-9659-2e2881d4dfaa)

Ajetaan komento uusiksi:

![uusi-ajo-pelit](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/a9373c26-54c0-40cb-ab3f-9e1c7f975c72)

Ei toimi vieläkään. Tämä liittynee tähän top.sls tiedostoon, mutta se näyttää mielestäni niin oikealta kun voi vain näyttää.

Tässä pähkäilin pidemmän aikaa ja ajattelin aloittaa alusta ja yksinkertaisemmin. Jospa vain yksi peli, ja se, että saisin pelin tulokset siirrettyä jotenkin edes master-koneelle.

Loin uuden hakemiston masterille nimeltä miniprojekti. Sinne loin uuden init.sls tiedoston, jonka sisältö näyttää nyt tältä:

![miniprojekti-sls](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/c4d917ea-47c0-47b3-94db-01591dbbd7c4)

Lähden nyt jatkamaan projektia nyt paljon yksinkertaisemmin. Asennetaan vain yksi peli, luodaan tulostiedosto sekä konffaustiedosto cronille (johon kirjoitan säännön myöhemmin). Vaihdoin myös top.sls -tiedostoon 'pelit' sijaan 'miniprojekti', eli tämän hakemiston nimen.

Ajetaan seuraavaksi tämä komennolla ``$sudo salt '*' state.apply miniprojekti``

![miniprojekti-ajo](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/05681a3d-2d26-469e-ac35-f8a2b77e3c28)

Noniin nyt lähtee toimimaan! Uudet tiedostot luotiin, ja tuo moon-buggy peli oltiin jo asennettu, niin siksi sen tila ei vaihtunut.

Nyt on vuoro mennä orjakoneelle t001, ja käydä pelaamassa peliä, jotta saan tuloksia mitä sitten lähettää

![moon-buggy](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/8f16c8d1-47b7-41ad-80fe-ef00d105ba2c)

Tein tuloksen ja nimesin pelaajan "tester". Surffasin t001-koneella kansioon /var/games/moon-buggy josta löyty tiedosto 'mbscore'. Sieltä löytyikin nyt tulokseni!

![tulokset-tester](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/e771ba44-03df-4349-82d7-ebf48d456fae)

Seuraavaksi avasin crontabin. Eli ajoin komennon ``$ crontab -e`` ja lisäsin seuraavan rivin sinne: ``*/1 * * * * scp /var/games/moon-buggy/mbscore.txt vagrant@tmaster:/srv/salt/miniprojekti/kopiointitesti.txt``

Ajattelin käyttää tuttua ja turvallista scp:ta. Eli kirjoitin tuohon, että tämä tehtävä suoritetaan kahden minuutin välein. No mitään ei kuitenkaan tapahtunut. Yritin kopioida tiedostoa suoraan komennolla ``scp /var/games/moon-buggy/mbscore.txt tmaster@192.168.12.3:/srv/salt/miniprojekti/kopiointitesti.txt``. Tässä kohtaa yhteyttä yritettiin luoda, mutta kuitenkin lupa evättiin.

![scp-yritys](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/add5e048-5c39-4b8d-979e-f8304a2e43cf)

Yritän seuraavaksi vielä saltin komentoa ``sudo salt '*' cp.push /var/games/moon-buggy/mbscore``. Tila ajettiin, mutta vastaukseksi sain Falsea:

![orja-scp-ajo](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/d18d124e-a102-48fa-b82f-c9254f8c7aa5)

Ajoin vielä tämän saman vielä, mutta vaihdoin '*':n 'tmaster*:ksi

![scp-yritys](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/4c3969dd-fa04-46f8-974d-c991aa04124a)

Löysin Saltin [sivuilta](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html) seuraavan: "Since this feature allows a minion to push a file up to the master server it is disabled by default for security purposes. To enable, set file_recv to True in the master configuration file, and restart the master.".

Kävin vaihtamassa tuon file_recv:n Falsesta Trueksi.

![file-recv-true](https://github.com/ThomasHelminen/Palvelinten-hallinta-miniprojekti/assets/148875548/c80a5ae2-d986-41e8-a180-6a2b5fd42382)

Ajoin samat komennot kun äsken uusiksi, mutta ei tominut. Nyt alkaa aika loppumaan. Tällä kirjoitushetkellä kello on 13.22. ja deadline on tälle tehtävälle klo 14. Työni alkavat myös samalla minuutilla. Tulen jatkamaan tämän projektin tekemistä lähipäivinä. Minua harmittaa, että tämä ei mennyt ihan kuin Strömsössä. Tulen tarkentamaan raporttiani yksityiskohtaisemmaksi. Tällä hetkellä tilanne on tämä.

Edit. Nyt kuin luin tämän läpi, niin tämä näyttää siltä että on juosten pissaten tehty. Ja se on ikävää. Muutetaan tilanne lähitulevaisuudessa.

¤¤ Lopuksi

Tämä kurssi on vaatinut todella paljon aikaa, itselle välillä jopa vähän liikaakiin. Tällä kurssilla on tullut opittua samalla eniten. Olen tykännyt tehtävistä, ja siitä miten opetus on tapahtunut. Olisin toivonut lähiopetusta, koska luulen, että olisin saanut siitä vielä enemmän irti.

## PROJEKTI EI OLE VIELÄ LOPPUUN KÄSITELTY! TULEN YRITTÄMÄÄN SAAMAAN TÄMÄN ONNISTUNEEKSI VALMIIKSI, MUTTA NYT EI AIKA VAIN RIITTÄNYT.


## Lähdeluettelo
- Helminen T, 2023. Palvelinten-hallinta -kurssi, GitHub repository. Luettavissa: https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi. Luettu 11.12.2023.
- Karvinen, T. 13.10.2023. Infra as Code 2023. Luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/. Luettu 11.12.2023.
- Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/. Luettu 11.12.2023.
- SaltProject. salt.modules.cp. Luettavissa: https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html. Luettu 11.12.2023.
