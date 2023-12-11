# Palvelinten-hallinta-miniprojekti
Täältä löydät miniprojektini Tero Karvisen vetämälle Palvelinten hallinta -kurssille, tervetuloa! Kyseessä on tehtävä 'h7', ja kurssi löytyy [täältä](https://terokarvinen.com/2023/configuration-management-2023-autumn/). Yritän käyttää tämän miniprojektin teossa pääsääntöisesti kurssilta oppimiani asioita. Aiemmat kurssitehtävät löydät minun '[Palvelinten-hallinta -kurssi](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi)' repositorysta!

Käytän projektissa Vagrantia sekä herra-orja -arkkitehtuuria (herra-kone ja kaksi orjakonetta), jonka loin aiemmin kurssitehtävssä '[h2 karjaa, kohta d)](https://github.com/ThomasHelminen/Palvelinten-hallinta--kurssi/blob/main/h2-karjaa.md').

Miniprojektin ideana olisi asentaa komentorivipelejä herralta orjakoneille käyttäen Saltia. Kun orjakoneilla pelataan peliä, niin tulostiedot ns. leaderboardit siirtyvät herra-koneelle tietyn väliajoin. Näin herra-koneelta pystyisi seuraamaaan tuloksia ja ennätyksiä "etänä". Katsotaan päästäänkö sinne asti!

Työkoneena toimii HP:n Elitebook 830 G5 (speksit: Windows 11 Pro versio 10.0.22631, Intel Core i5-8350U, 16GB RAM, 256GB SSD, Vagrant versio 2.4.0, VirtualBox versio 7.0.12). Kaikki kuvankaappaukset ovat minun ottamia, ellen toisin mainitse. Tein kaikki pakolliset tehtävät sekä muutaman vapaaehtoisen tehtävän.  Tervetuloa mukaan!

## Miniprojekti

Lähdetään liikkeelle avaamalla isäntäkoneesta PowerShell järjestelmävalvojana. Siirrytään saldemo-kansioon, josta löytyy aiemmin luotu vagrantfile. Täällä ollessa ajetaan komento  ``vagrant up`` joka käynnistää virtuaalikoneet.


## Lähdeluettelo
- Karvinen, T. 13.10.2023. Infra as Code 2023. Luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/. Luettu 11.12.2023.
- Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/. Luettu 11.12.2023.
