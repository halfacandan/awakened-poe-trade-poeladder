# ![Awakener's Orb](https://web.poecdn.com/image/Art/2DItems/Currency/TransferOrb.png) Awakened PoE Trade - PoE Ladder Edition

➡ [Download for Windows & Linux](https://snosme.github.io/awakened-poe-trade/download) ⬅

### Development

#### How to create the installer:
1) Clone project into VSCode
2) cd main
3) npm install
4) npm run build
5) npm run package

#### How to debug the installer:
The package misses out crucial files when packaging the app. These are packages up in to **%AppData%\Local\Programs\Awakened PoE Trade\resources\app.asar**.

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
