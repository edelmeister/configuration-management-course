# H3 Demoni

Tehtäväraportti tehtävälle [H3 Demoni](https://terokarvinen.com/palvelinten-hallinta/#h3-demoni) [[1]](#lähdeluettelo).

## Sisällysluettelo

[x) Lue ja tiivistä](#x-lue-ja-tiivistä)

[a) Apache easy mode](#a-apache-easy-mode)

[b) SSHouto](#b-sshouto)

[c) Oma moduli](#c-oma-moduli)

[d) VirtualHost](#d-virtualhost)

## x) Lue ja tiivistä

### Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port

Ohje [[2]](#lähdeluettelo) näyttää lyhyesti Salt-tilatiedoston ``sshd.sls``, joka asentaa minion:ille ``openssh-server``:in, luo sille konfiguraatiotiedoston ``sshd_config``, sekä huolehtii siitä, että SSH-demoni ``sshd`` on päällä. Tähän käytetään Salt:in tilafunktioita pkg, file ja service. Lisäksi ohje näyttää miltä ``sshd_config``:in sisällä on.

### Salt-dokumentaatio: pkg, file ja service

``pkg``-tilafunktiota käytetään asentamaan, päivittämään ja poistamaan paketteja. ``pkg.installed`` varmistaa, että määrätty paketti on asennettu. ``pkg.purged`` varmistaa, että paketti ja sen konfiguraatiotiedostot on poistettu. ``pkg.removed`` jättää konfiguraatiotiedostot rauhaan. Lisäämällä tilatiedostossa funktion alle parametrin ``pkgs`` voidaan määrittää useampi paketti, joita tilafunktio koskee. [[3]](#lähdeluettelo)

``file``-tilafunktiota käytetään manipuloimaan järjestelmän tiedostoja. ``file.managed`` on monikäyttöinen funktio, jolla manipuloidaan tiedostoa ja voidaan mm. varmistaa, että tietty tiedosto on minion:illa ja lataamaan se master:ilta, jos ei. Parametrilla ``content`` voidaan myös määrittää tiedoston sisältö. ``file.absent`` varmistaa, että tiedosto on poistettu. ``file.symlink``:illä varmistetaan, että tietyt symboliset linkit ovat olemassa. [[4]](#lähdeluettelo)

``service``-tilafunktiota käytetään käynnistämään, uudelleenkäynnistämään ja pysäyttämään järjestelmän demoneja, eli Linux:in järjestelmäpalveluja. ``service.running`` varmistaa, että demoni on päällä. Parametrilla ``watch`` voidaan määrittää, että demoni käynnistyy uudelleen, kun esim. konfiguraatiotiedostoa on muokattu. Parametrilla ``enable`` voidaan määrittää käynnistyykö demoni järjestelmän käynnistyessä. ``service.dead`` taas puolestaan varmistaa, että demoni on pois päältä. [[5]](#lähdeluettelo)

## a) Apache easy mode

Tämän kerran seikkailut alkoivat hauskalla yllärillä, kun ``vagrant up`` komennon seurauksena Windows 11 -koneeni heitti siniset veivit, viimeisinä sanoinaan: ``Stop code: DRIVER_IRQL_NOT_LESS_OR_EQUAL. What failed: VMMR0.r0``.

![BSOD](imgs/h3-01.jpg)

Veikkaukseni oli, että joko Vagrant tai VirtualBox on tähän syyllinen. Asensin ensin Vagrant:in uudelleen, mutta BSOD jatkui. Seuraavaksi latasin VirtualBox:in uusimman version ja asensin sen. Tämän jälkeen virtualisointi toimi kiltisti ja moitteetta. Myös "``VMMR0.r0``" viittaisi virtuaalikoneen olleen ongelma.

Käytän tässä tehtävässä kahta virtuaalikonetta, jotka loin Vagrant:illa hyödyntämällä H2-tehtävässä kirjoittamaani Vagrantfile-tiedostoa [[6]](https://github.com/edelmeister/configuration-management-course/blob/main/H2/H2-infra-as-code.md#d-master-minion-verkossa).

Ajoin kone001:llä komennon ``sudo salt-call --local state.single pkg.installed apache2``, joka asensi koneelle ``apache2``:n. Komennolla ``sudo salt-call --local state.single service.running apache2`` varmistin, että kyseinen demoni käynnistyi.

![Apache2 running.](imgs/h3-02.png)

Tarkistin lisäksi, että Salt ei valehtele komennolla ``systemctl status apache2``. Apache2 näyttäisi siis toimivan.

![Apache2 is really runnning.](imgs/h3-03.png)

Lisäksi testasin, että pystyn yhdistämään Apache2-palvelimelle menemällä host-koneeni selaimella osoitteeseen ``http://192.168.12.11``. Apache2:n oletussivu näkyi, joten yhteys toimi.

Seuraavaksi siirryin hakemistoon ``/var/www/html`` ja tein Apache2:n oletussivusta kopion. Sitten loin uuden ``index.html``-tiedoston, johon kirjoitin:

```HTML
<h1> MINUN VERKKOSIVU </h1>
<p> Tämä on Apache2-verkkosivupalvelin.</p>
```

Käynnistin palvelimen uudelleen komennolla ``sudo systemctl restart apache2``, virkistin selaimen ikkunan, ja ihailin uutta verkkosivuani.

![Uusi verkkosivu.](imgs/h3-04.png)

Sitten koitti aika automoida kyseisen verkkosivupalvelimen luonti kone002:lle Salt:illa.

Kirjoitin uuden Salt moduulin ``apache2`` tilatiedoston ``/srv/salt/apache2/init.sls``, joka asentaa Apache2:n, varmistaa, että se on päällä, sekä kopioi master:ilta (kone001) HTML-tiedoston. Tähän apuna käytin Karvisen tilatiedostoa ohjeessa [[2]](#lähdeluettelo). Loin lisäksi moduulin kansioon symbolisen linkin ``index.html``, joka osoittaa master:in tiedostoon ``/var/www/html/index.html``.

```YAML
apache2:
  pkg.installed
/var/www/html/index.html:
  file.managed:
    - source: salt://apache2/index.html
apache2:
  service.running
```

Yritin suorittaa tilatiedostoa komennolla ``sudo salt '*' state.apply apache2``, mutta Salt sylki virheilmoituksen:

![Virheilmoitus.](imgs/h3-05.png)

Tarkistin tiedoston kirjoitusvirheiltä ja pähkäilin asiaa tovin, mutten saanut sitä ratkaistua. Lopulta annoin ruutukaappauksen virhekoodista GPT-4o:lle ja kysyin mikä mahtaisi olla vikana. GPT-4o:n mukaan ongelma saattaa olla siinä, että ``init.sls``-tiedostossa on kaksi tilaa, joilla on sama ID, joka tässä tapauksessa on ``apache2:``. Muistin, että komennossa ``systemctl restart apache2`` ja ``...apache2.service`` ajavat samaa asiaa, joten nimesin tilatiedoston jälkimmäisen tilan ``apache2.service``.

```YAML
apache2:
  pkg.installed
/var/www/html/index.html:
  file.managed:
    - source: salt://apache2/index.html
apache2.service:
  service.running
```

Tämän jälkeen ajoin komennon ``sudo salt '*' state.apply apache2`` uudelleen, ja tällä kertaa komennot suorituivat onnistuneesti.

![Onnistunut tila.](imgs/h3-06.png)

Lisäksi pystyin host-koneella selaamaan sivulle ``http://192.168.12.12`` (kone002:n IP-osoite), jossa näkyi tekemäni HTML-sivu.

![Uusi HTML-sivu näkyy myös minion:illa.](imgs/h3-07.png)

## b) SSHouto

## c) Oma moduli

## d) VirtualHost

## Lähdeluettelo

[1]
T. Karvinen, “Palvelinten hallinta - configuration management systems course - 2024 autumn,” Terokarvinen.com, 2024. https://terokarvinen.com/palvelinten-hallinta/#h3-demoni (accessed Nov. 18, 2024).

[2]
T. Karvinen, “Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port | Tero Karvinen,” Terokarvinen.com, 2018. https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh (accessed Nov. 18, 2024).

[3]
VMware Inc., “salt.states.pkg,” Saltproject.io, Oct. 23, 2023. https://docs.saltproject.io/en/latest/ref/states/all/salt.states.pkg.html (accessed Nov. 18, 2024).

[4]
VMware Inc., “salt.states.file,” Saltproject.io, Oct. 23, 2023. https://docs.saltproject.io/en/latest/ref/states/all/salt.states.file.html (accessed Nov. 18, 2024).

[5]
VMware Inc., “salt.states.service,” Saltproject.io, Oct. 23, 2023. https://docs.saltproject.io/en/latest/ref/states/all/salt.states.service.html (accessed Nov. 18, 2024).

[6]
S. Edelmann, “Kotitehtäväraportti H2 Infra-as-Code - d) Master-Minion verkossa,” GitHub, Nov. 12, 2024. https://github.com/edelmeister/configuration-management-course/blob/main/H2/H2-infra-as-code.md#d-master-minion-verkossa (accessed Nov. 18, 2024).