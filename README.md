# UAV drooni lendamise ülesanne

Ülesande mõte on saada UAV lendamise simuleerimine tööle Linux'il ning seejärel optimeerida lendamistrajektoori MRS keskkonnas (Multi-robot Systems Group UAV system), kasutades Apptainer'it. Ülesanne põhineb "MRS Summer School 2024" ning on tehtud eesti keelseks Robotitehnika aine raames. See juhend pole 1-1 ning on proovitud teha võimalikult lihtsalt ja jälgitavaks, kuid originaaljuhendi saab leida MRS summer school [github](https://github.com/ctu-mrs/summer-school-2024)'i lehelt.

## 1. WSL-i installeerimine:
Ava command prompt või PowerShell ning kirjuta järgnev käsk. Lisainfo saab [siit](https://learn.microsoft.com/en-us/windows/wsl/setup/environment#get-started).
```
wsl --install
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
Viimane nõue enne Apptainer'i installeerimist on libsubid toetus:
```
sudo apt-get install -y libsubid-dev
```
### 2.1. Go installeerimine:
Go on keel milles Apptainer on kirjutatud, mistõttu on ka vaja see saada.
