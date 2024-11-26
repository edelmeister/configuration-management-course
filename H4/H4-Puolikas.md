# H4 Puolikas

Tehtäväraportti tehtävälle [H4 Puolikas](https://terokarvinen.com/palvelinten-hallinta/#h4-puolikas) [[1]](#lähdeluettelo)

## Sisällysluettelo



## 1. Johdanto

Tehtävän tavoite on aloittaa oma Salt-moduuli luomalla ensimmäinen vedos. Moduulin rakennusta jatketaan myöhemmin palautteen ja vinkkien pohjalta.

Oma tavoitteeni on luoda Minecraft-palvelinmoduuli, joka automatisoi Minecraft-pelipalvelimen pystyttämisen kohteena olevalle minion-koneelle.

### 1.1. Alusta

Host-koneena toimii Windows 11 -pöytätietokone. Prosessori on i7-9700K ja RAM:ia on 32 gigatavua.

Tehtävässä käytetään kahta virtuaalikonetta, jotka luon Vagrantin ja VirtualBoxin avulla. Alla on kirjoittamani ``Vagranfile`` [[2]](#lähdeluettelo), joka luo virtuaalikoneet. Kone001 toimii Salt-masterina ja kone002 taas puolestaan minionina.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

$master = <<MASTER
echo "Downloading Salt bootstrap script." 
curl -o bootstrap-salt.sh -L https://github.com/saltstack/salt-bootstrap/releases/latest/download/bootstrap-salt.sh
echo "Making the script executable."
chmod +x bootstrap-salt.sh
echo "Launching bootstrap script to install salt-master." 
sudo ./bootstrap-salt.sh -M stable
echo "Restarting salt-master."
sudo service salt-master restart
echo "Setup done."
MASTER

$minion = <<MINION
echo "Downloading Salt bootstrap script."
curl -o bootstrap-salt.sh -L https://github.com/saltstack/salt-bootstrap/releases/latest/download/bootstrap-salt.sh
echo "Making the script executable."
chmod +x bootstrap-salt.sh
echo "Launching bootstrap script to install salt-minion."
sudo ./bootstrap-salt.sh stable
echo "Setting the address of salt-master."
echo "master: 192.168.12.11">/etc/salt/minion
echo "Restarting salt-master."
sudo service salt-minion restart
echo "Setup done."
MINION

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  config.vm.define "kone001", primary: true do |kone001|
    kone001.vm.provision :shell, inline: $master
    kone001.vm.network "private_network", ip: "192.168.12.11"
    kone001.vm.hostname = "kone001"
  end

  config.vm.define "kone002" do |kone002|
    kone002.vm.provision :shell, inline: $minion
    kone002.vm.network "private_network", ip: "192.168.12.12"
    kone002.vm.hostname = "kone002"
  end

end
```




## Lähdeluettelo

[1]
T. Karvinen, “Palvelinten Hallinta - Configuration Management Systems Course - 2024 Autumn,” Terokarvinen.com, 2024. https://terokarvinen.com/palvelinten-hallinta/#h4-puolikas (accessed Nov. 26, 2024).

[2]
S. Edelmann, “H2 Infra-as-code - Kotitehtäväraportti,” GitHub, Nov. 12, 2024. https://github.com/edelmeister/configuration-management-course/blob/main/H2/H2-infra-as-code.md#d-master-minion-verkossa (accessed Nov. 26, 2024).