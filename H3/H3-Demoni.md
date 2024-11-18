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