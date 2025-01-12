# UAV drooni lendamise ülesanne

Ülesande mõte on saada UAV lendamise simuleerimine tööle Linux'il ning seejärel optimeerida lendamistrajektoori MRS keskkonnas (Multi-robot Systems Group UAV system), kasutades Apptainer'it. Ülesanne põhineb "MRS Summer School 2024" ning on tehtud eesti keelseks Robotitehnika aine raames. See juhend pole 1-1 ning on proovitud teha võimalikult lihtsalt ja jälgitavaks (lisades kirjeldusi erinevatele lisaprotsessidele), kuid ülesannet selgitava juhendi saab leida MRS summer school [github](https://github.com/ctu-mrs/summer-school-2024)'i lehelt.

## 1. WSL-i installeerimine:
Ava command prompt või PowerShell ning kirjuta järgnev käsk. Lisainfo saab [siit](https://learn.microsoft.com/en-us/windows/wsl/setup/environment#get-started). (NB! Lihtsalt wsl --install annab 22.04 versiooni, see ei lase ROS Noetic hiljem installeerida)
```
wsl --install -d Ubuntu-20.04
```
Seejärel tee arvutile restart. Taaskäivitusel peaks süsteem automaatselt küsima Linux'i jaoks kasutajanime ja parooli (soovitav on panna midagi, mis meelest ei lähe).

## 2. Apptainer'i installeerimine:
Selleks, et ülesandega seonduvaid skripte ja programme kasutada läheb vaja Apptainer rakendust. Eraldi juhendi leiab [siit](https://github.com/apptainer/apptainer/blob/main/INSTALL.md). Esiteks tuleks veenduda, et kõik Linux'iga seotud oleks uuendatud kõige värskemate versioonide peale. Selle tegemiseks saab kasutada käsklust (Ubuntu puhul, mis on WSL-iga default):
```
sudo apt-get update
```
Seejärel saab installeerida vajalikud eelnõuded Apptainer'i jaoks kasutades käsklusi (võib natuke aega võtta, soovitatav on võtta kõrvale tass teed/kohvi):
```
sudo apt-get install -y \
    build-essential \
    libseccomp-dev \
    pkg-config \
    uidmap \
    squashfs-tools \
    fakeroot \
    cryptsetup \
    tzdata \
    dh-apparmor \
    curl wget git
```

### 2.1. Go installeerimine:
Go on keel milles Apptainer on kirjutatud, selle saamiseks sisesta iga käsklus eraldi (wget käsu jaoks olev versioon on muutuv ning selle lingi leiab [siit](https://go.dev/dl/). Kopeeri kõige uuema versiooni link).
```
cd /tmp
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
Seejärel võid kasutada käsklust "go version" ning tulemusena peaks andma vastama versiooni näiteks: "go version go1.23.4 linux/amd64"
![image](https://github.com/user-attachments/assets/b89ece01-e7bc-4a0a-9bc4-c6259cd8afcb)

### 2.2. golangci-lint installeerimine:
See ei ole kahjuks kõik, Apptainer nõuab palju. Järgmisena on vaja ühte töövahendit, mis tagab Apptainer'is koodi järjekindlust.
```
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.59.1
```
Seejärel:
```
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

### 2.2. Repository kloonimine ja kompileerimine:
Siin võid asukoha valida ise (kindel kaust vms, kus eelistad hoida) kuna Go installeerimine enne eemaldas kindla asukohaga seotud vajaduse. Muidugi võid lihtsalt n-ö otse lasta ja mitte midagi muuta.
```
git clone https://github.com/apptainer/apptainer.git
cd apptainer
```
Kompileerimiseks tuleb kasutada järgmisi käsklusi (see võib võtta päris kaua):
```
./mconfig
cd $(/bin/pwd)/builddir
make
sudo make install
```
Peale pikka kompileerimist saab kontrollida, juhul kui installatsiooni käigus juba probleeme ei olnud, et kõik on ikka korras, Apptainer'i versiooni kasutades käsklust:
```
apptainer --version
```
![image](https://github.com/user-attachments/assets/15c220c4-12f0-452f-ad8f-33b9280a41e5)

Sellega peaks eeltööga olema kõik.

# Ülesanne
Palju õnne, kogu eelmise tegemisega sai EELTÖÖ tehtud, nüüd saab reaalse ülesande poolega alustada. Muidugi enne lahendamist on mõningaid asju veel vaja alla laadida. Esiteks tee koopia MRS school repository'st. Selleks vajuta github leheküljel paremal üleval olevat pluss märki, seejärel import repository (sellega läheb aega). Kui on olemas saad vajaliku alla laadida käsklustega:
```
mkdir -p ${HOME}/git
cd ${HOME}/git && git clone <your new repository's link>
```
## 1. MRS UAV System installeerimine
Allikas [siit](https://github.com/ctu-mrs/mrs_uav_system). Süsteemi tööle saamiseks on vaja ROS'i (robot operating system, siin ka ütleb et peaks Noetic). Probleem sellega on, et WSL laeb uuema Ubuntu versiooni alla, mis tähendab, et ei ole võimalik ROS Noetic kasutada (kuna see nõuab Ubuntu 20.4). Seega peab proovima ROS2-ga, täpsemalt Jazzy versioon. https://docs.ros.org/en/jazzy/Installation/Alternatives/Ubuntu-Development-Setup.html

### 1.1 ROS2 installeerimine
```
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```
```
sudo apt install software-properties-common
sudo add-apt-repository universe
```
```
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```
```
sudo apt update && sudo apt install -y \
  python3-flake8-blind-except \
  python3-flake8-class-newline \
  python3-flake8-deprecated \
  python3-mypy \
  python3-pip \
  python3-pytest \
  python3-pytest-cov \
  python3-pytest-mock \
  python3-pytest-repeat \
  python3-pytest-rerunfailures \
  python3-pytest-runner \
  python3-pytest-timeout \
  ros-dev-tools
```
vaja vcstool et jargmist asja teha
```
sudo apt install vcstool
```
```
mkdir -p ~/ros2_jazzy/src
cd ~/ros2_jazzy
vcs import --input https://raw.githubusercontent.com/ros2/ros2/jazzy/ros2.repos src
```
update alati vaja
```
sudo apt upgrade
```
enne seda vaja dependency (kui seda ei tee siis jargmine kasklus annab errori, ja ytleb et installeeri python3-rosdep2 -> rosdep2 ei ole ainult ilma numbrita versioon)
```
sudo apt install python3-rosdep
```
seejarel (annab koik sh noetic?)
```
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
```
veel yks dependency et ros2 kasutada
```
sudo apt install colcon
```
compile ros2
```
cd ~/ros2_jazzy/
colcon build --symlink-install
```
### Probleemide korral
Kui tekib probleeme, mis on tõenäoline ROSiga (näiteks laed vale ROS versiooni nagu mina), kasuta järgmisi käsklusi, et eelnevad teod kustutada ning alusta otsast peale.
```
rm -rf ~/ros2_jazzy
rm -rf ~/ros
sudo rm /etc/apt/sources.list.d/ros-latest.list
```
