# UAV drooni lendamise ülesanne

Ülesande mõte on saada UAV lendamise simuleerimine tööle Linux'il ning seejärel optimeerida lendamistrajektoori MRS süsteemis (Multi-robot Systems Group UAV system), kasutades Apptainer'i keskkonda. Ülesanne põhineb "MRS Summer School 2024" ning on tehtud eesti keelseks Robotitehnika aine raames. See juhend pole 1-1 ning on proovitud teha võimalikult lihtsalt ja jälgitavaks (lisades kirjeldusi erinevatele lisaprotsessidele), kuid ülesannet selgitava juhendi saab leida MRS summer school [github](https://github.com/ctu-mrs/summer-school-2024)'i lehelt.

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
Seejärel saab installeerida vajalikud eelnõuded Apptainer'i jaoks kasutades käsklusi (võib natuke aega võtta):
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
Go on keel milles Apptainer on kirjutatud, selle saamiseks sisesta iga käsklus eraldi (wget käsu jaoks olev versioon on muutuv ning selle lingi leiab [siit](https://go.dev/dl/). Kopeeri kõige uuema versiooni link wget käskluse taha).
```
cd /tmp
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
Seejärel võid kasutada käsklust "go version" ning tulemusena peaks andma vastama versiooni näiteks: "go version go1.23.4 linux/amd64"

![image](https://github.com/user-attachments/assets/b89ece01-e7bc-4a0a-9bc4-c6259cd8afcb)

### 2.2. golangci-lint installeerimine:
See ei ole kõik, Apptainer nõuab palju. Järgmisena on vaja ühte töövahendit, mis tagab Apptainer'is koodi järjekindlust.
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
Palju õnne, kogu eelmise tegemisega sai eeltöö tehtud, nüüd saab reaalse ülesande poolega alustada. Muidugi enne lahendamist on mõningaid asju veel vaja alla laadida. Esiteks tee koopia [MRS school](https://github.com/ctu-mrs/summer-school-2024) repository'st. Selleks vajuta github leheküljel paremal üleval olevat pluss märki, seejärel import repository (sellega läheb aega). Kui on olemas saad vajaliku alla laadida käsklustega:
```
mkdir -p ${HOME}/git
cd ${HOME}/git && git clone <sinu koopia url>
```
See laeb alla kõik vajaliku. Kui soov teha eraldi või on midagi puudu saab vaadata juhendi lõpust manuaalset verisooni.
```
cd ${HOME}/git/summer-school-2024 && ./install.sh
```
Tulemus võiks olla midagi sellist:
![image](https://github.com/user-attachments/assets/a0ef6cc5-5a38-4218-89ea-81ec9ddaa0fb)


Selleks, et esiteks simulatsiooniga alustada on olemas paar võimalust. Simulatsioonide alustamiseks ning muu käskluste kasutamiseks peaksid paiknema vastavas kaustas, näide: ```kasutaja@arvuti:~/git/koopianimi$```. Linux-is navigeerimine toimub ```cd``` käsklusega.

**1. Offline, mitte simuleeriv versioon (soovitatav variant)**

Käivitamiseks on käsklus
```
./simulation/run_offline.sh
```
![image](https://github.com/user-attachments/assets/39c67ac3-235b-449e-b698-4b6102dedfef)
Nagu näha, esialgne trajektoor ei ole piisav ning efektiivsusest kaugel.

**2. Online, süsteemi simuleerimine lokaalselt**

Käivitamiseks on käsklus
```
./simulation/run_simulation.sh
```

Lõpetamiseks on käsklus
```
./simulation/kill_simulation.sh
```

Ülesande lahendamiseks/mõistmiseks kasuta https://github.com/ctu-mrs/summer-school-2024, peamiselt nõuab python failide muutmist, et optimeerida drooni lendamist. Failid, mida muuta, leiad ```kaustanimi/mrim_task/mrim_planner``` kaustast. Soovitatav on kasutada interneti võimalusi erinevate funktsioonide ja meetodite uurimiseks, implementeerimiseks. Eelmainitud lingis on ka ülesande tulemused kirjas, milleni peaks jõudma (vastavad kiirused ning liikumisega seotud parameetrid).

- ```scripts/```
    - ```planner.py```: siit leiab näiteid ning ideid, kuidas lahendada.
    - ```trajectory.py```: sisaldab funktsioone, mis on seotud trajektooriga. Siia saab näiteks lisada interpoleerimist.
    - ```solvers/```
        - ```tsp_solvers.py```: VP-de (viewpoint) määramine TSP (travelling salesman problem) jaoks, marsruudi planeerimine ja TSP lahendamine. Siin saab VP-si määrata UAV-dele ning uurida trajektooriplaneerijate mõju TSP-le.
        - ```utils.py```: erinevate kasutuses olevate funktsioonide allikas, võib oma funktsioone siia lisada.
    - ```path_planners/grid_based```
        - ```aster.py```: A* pathfinding implementatsioon. On võimalik algoritmi tööd parendada.
    - ```path_planners/sampling_based```
        - ```rrt.py```: RRT (rapidly exploring random trees) meetodi implementatsioon. Saab samuti ise lisada meetodeid, et algoritm toimiks paremini.
    - ```config/```
        - ```virtual.yaml``` ja ```real_world.yaml```: erinevad ülesandega seotud parameetrid.

Koodi muutmiseks saab süsteemi sees kasutada Python keskkonda kasutades käsklust ```pycharm.sh``` ning hiljem ```03_compile.sh```, sarnaselt simulatsiooni stardile.



## LISA* MRS UAV System installeerimine (manuaalselt)
Allikas [siit](https://github.com/ctu-mrs/mrs_uav_system). Süsteemi tööle saamiseks on vaja ROS'i (robot operating system, siin ka mainin et täpsemalt Noetic). Kui esialgu sai korrektselt alla laaditud 20.4 versioon Ubuntu'st ning mitte kõige uuem versioon, mida wsl --install käsklus annab, peaks see minema sujuvalt. 22.04 laseb ainult ROS2 alla laadida, mis ei tööta MRS-iga.

### LISA.1 ROS (Noetic)
Selle eraldi allalaadimisega läheb väga kaua aega kuna faili suurus on ka suurem.
```
curl https://ctu-mrs.github.io/ppa-stable/add_ros_ppa.sh | bash
sudo apt install ros-noetic-desktop-full
```

### LISA.2 rosbuild
Kui ROS keskkonda konfigureerida on valik catkin ja rosbuild-i vahel (olenevalt ROS-i väljaandest). Konfigureerimise juhendi leiab [siit](http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment)
```
sudo apt install python3-rosinstall
```
rosws on vajalik ROS workspace loomiseks.
```
rosws init ~/noetic_workspace /opt/ros/noetic
```
Siin ära unusta vahetada kasutajanimi endal kasutuses oleva vastu.
```
source /home/<kasutajanimi>/noetic_workspace/setup.bash
```
```
mkdir ~/noetic_workspace/sandbox
rosws set ~/noetic_workspace/sandbox
```

### LISA.3 MRS
Lisa stabiilne versioon enda repository-sse käsklusega:
```
curl https://ctu-mrs.github.io/ppa-stable/add_ppa.sh | bash
```

Installeeri MRS UAV süsteem, selle käsklusega läheb natuke aega.
```
sudo apt install ros-noetic-mrs-uav-system-full
```
Alusta näidis Gazebo simulatsioon.
```
roscd mrs_uav_gazebo_simulation/tmux/one_drone
./start.sh
```
./start.sh käsklusega peaks avanema väike simulatsioon (kolm erinevat akent)
![image](https://github.com/user-attachments/assets/07034f64-8d90-4a7d-8ab0-b8ee4a649422)
![Screenshot 2025-01-15 135637](https://github.com/user-attachments/assets/dbb886c8-4937-4263-aa56-75cb55227e7f)
![Screenshot 2025-01-15 135627](https://github.com/user-attachments/assets/a65575e4-e24f-4a53-aba6-fe4099455654)

### Probleemide korral
Kui tekib probleeme, mis on tõenäoline ROSiga (näiteks laed vale ROS versiooni nagu mina), kasuta järgmisi käsklusi (eraldi, koos, kombinatsioonina vms), et eelnevad teod kustutada ning alusta otsast peale. Gedit on mõeldud .bashrc faili muutmiseks, juhul kui sinna vajadus manuaalselt lisada/eemaldada käsklusi. Viimase kahe käsklusega saab ROS-iga seotud kaustad, failid eemaldada. Juhul kui neist pole piisav on ka variant Ubuntu deinstalleerida ja alustada otsast peale.
```
gedit .bashrc
rm -rf ~/ros
sudo rm /etc/apt/sources.list.d/ros-latest.list
```
Samuti võib vaadata MRS cheatsheet-i programmis navigeerimise jaoks või lähtuda originaaljuhendist
https://github.com/ctu-mrs/mrs_cheatsheet

