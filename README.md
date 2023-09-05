# ![Awakener's Orb](https://web.poecdn.com/image/Art/2DItems/Currency/TransferOrb.png) Awakened PoE Trade - PoE Ladder Edition

➡ [Download for Windows & Linux](https://snosme.github.io/awakened-poe-trade/download) ⬅

### Development

### 1. Set up the project

1) Clone project into VSCode
2) cd renderer
3) npm install
4) cd ..\main
5) npm install

### 2. How to build the project

Complete the steps in **1. Set up the project** prior to attempting this

1) Open the project in VSCode
2) cd renderer
3) npm run build
4) npm run make-index-files
5) cd ..\main
6) npm run build
7) npm run package

### 3. How to run in Dev mode

Complete the steps in **1. Set up the project** prior to attempting this

1) Open a cmd window
2) cd ..\awakened-poe-trade-poeladder\renderer
3) npm run dev
4) Leave that window running
5) Open the project in VSCode
6) cd main
7) npm run dev
8) Start PoE

#### 4. How to create the installer

Complete the steps in **2. How to build the project** prior to attempting this

1) Open the project in VSCode
2) cd main
3) npm run package

#### How to debug the installer:
The packaging process is dependent upon two separate builds: the "renderer" folder and the "main" folder. It then packages both of these up in to **%AppData%\Local\Programs\Awakened PoE Trade\resources\app.asar** based on the commands in **electron-builder.yml**.

You can manually edit the app.asar file by:
1) Installing 7-Zip (64-bit): https://www.7-zip.org/download.html
2) Installing the "Asar7z" plugin (64-bit): https://www.tc4shell.com/en/7zip/asar/

Extract the missing files/folders from a working version then add them to your **..\awakened-poe-trade-poeladder\main\dist** folder

### Acknowledgments

- [SnosMe (Alexander Drozdov)](https://github.com/SnosMe)
- [libuiohook](https://github.com/kwhat/libuiohook)
- [RePoE](https://github.com/brather1ng/RePoE)
- [poeprices.info](https://www.poeprices.info/)
- [poe.ninja](https://poe.ninja/)

![](https://i.imgur.com/MATqhv7.png)
