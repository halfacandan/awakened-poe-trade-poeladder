# ![Awakener's Orb](https://web.poecdn.com/image/Art/2DItems/Currency/TransferOrb.png) Awakened PoE Trade - PoE Ladder Edition

➡ [Download for Windows & Linux](https://snosme.github.io/awakened-poe-trade/download) ⬅

### Development

#### How to create the installer

Only run steps 3 and 7 the first time you set up the project:
1) Clone project into VSCode
2) cd renderer
3) npm install
4) npm run build
5) npm run make-index-files
6) cd ..\main
7) npm install
8) npm run build
9) npm run package

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
