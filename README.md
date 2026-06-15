# Авто-сборщик ImmortalWrt

## Информация по поддерживаемым устройствам

- Поддержка [Netis NX31](https://www.netisru.com/products/NX31.html)

  **Доступ по ssh и к панели**

  - IP: `192.168.1.1`
  - Порт: `22` (для ssh можно не указывать)
  - Пароль: нету, **как загрузитесь установите его сами**

    > Внимание: Пароля на частоты 2.4 и 5 герц нету, чтобы к вам не подключились
    > незнакомые люди рекомендуется либо:
    >
    > 1. Сразу поставить пароль в `Network` - `Wireless` - `Edit` (оба radio0 и
    >    radio1) и в `Interface Configuration` - `Wireless Securuty` выбираем
    >    в `Encryption` тип `WPA2-PSK` (или `WPA2-PSK/WPA3-SAE Mixed Mode`)
    >    и в `Key` указываем ваш пароль
    >
    > 2. На время отключить данные частоты, а когда они вам понадобятся включаем
    >    их

  **Прошивка**

  - Со стоковой прошивки netis происходит через 3 этапа

    1. Полную смену загрузчика uboot (FIP и опционально BL2) и перезагрузкой в
       режим tftp

       Файлы:

       - [immortalwrt-25.12.0-mediatek-filogic-netis_nx31-bl31-uboot.fip](https://downloads.immortalwrt.org/releases/25.12.0/targets/mediatek/filogic/immortalwrt-25.12.0-mediatek-filogic-netis_nx31-bl31-uboot.fip)
       - [immortalwrt-25.12.0-mediatek-filogic-netis_nx31-preloader.bin](https://downloads.immortalwrt.org/releases/25.12.0/targets/mediatek/filogic/immortalwrt-25.12.0-mediatek-filogic-netis_nx31-preloader.bin)

    2. В режиме tftp загружаем из репозитория `squashfs-factory` чтобы загрузится
       в ImmortalWrt (из ОЗУ)
    3. Внутри врменной ImmortalWrt загружаем `squashfs-sysupgrade.bin`

  - Если же вы на прошивке от OpenWrt/Kwrt и т.д, тогда прошивка происходит через
    меню LuCi с указанием файла `squashfs-sysupgrade.bin`

    Переходим в `System` - `Backup / Flash Firmware`, в `Flash new firmware image`
    жмём `Flash image...` выбираем `squashfs-sysupgrade.bin`

    > [!WARNING]
    > Убираем галочку `Keep settings and retain the current configuration`, т.к
    > устройство должно загрузится начисто, без ваших конфигураций.

    После прошиваем нажав `Continue`, подключаемся к панели только тогда когда
    индикатор на роутере будет гореть статично синим цветом

## Полезная информация

### TODO: Официальный репозиторий kmod и добавление feed'а

## Проблемы и способы их решения

1. Текущие зеркала выдают ошибку `Bad gateway 502`

   Решение: Временно перейти на другое зеркало, например сами авторы в [тг канале](https://t.me/ctcgfw_project_openwrt/55)
   советуют использовать из [help.mirrorz.org](https://help.mirrorz.org/immortalwrt/)

   Команда по смене зеркала

   ```sh
   # Создастся резервная копия текущего файла distfeeds.list с репозиториями из downloads.immortalwrt.org
   sed -e 's,https://downloads.immortalwrt.org,https://mirror.nju.edu.cn/immortalwrt,g' \
       -e 's,https://mirrors.vsean.net/openwrt,https://mirror.nju.edu.cn/immortalwrt,g' \
       -i.bak /etc/apk/repositories.d/distfeeds.list
   ```

   После чего обновление индексов будет происходить нормально

   ```sh
   opkg update
   ```

   Как только починят основное зеркало возвращаем обратно

   ```sh
   # Создастся резервная копия текущего файла distfeeds.list с репозиториями из mirror.nju.edu.cn
   sed -i.bak "s,https://mirror.nju.edu.cn,https://downloads.immortalwrt.org,g" "/etc/apk/repositories.d/distfeeds.list"
   ```

2. При обновлении индексов пакета возникает множество ошибок связанных с
   `wgetSSL verify error: certificate is not yet valid`

   Это происходит из-за того, что при первом включении системное время указано
   неверно

   Переходим в `System` - `General Settings` и в `Local Time` жмём `Sync with browser`,
   после чего обновление индексов пакетов будет происходить нормально

## TODO

## Благодарность

- [qlxi/GL_AXT1800](https://github.com/qlxi/GL_AXT1800)
- [m0eak/Openwrt_Builder](https://github.com/m0eak/Openwrt_Builder)
- [wukongdaily/AutoBuildImmortalWrt](https://github.com/wukongdaily/AutoBuildImmortalWrt)

И другие

- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [JiaY-shi/openwrt](https://github.com/JiaY-shi/openwrt.git)
- [tmate](https://github.com/tmate-io/tmate)
- [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [WeTransfer](https://wetransfer.com/)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © [**P3TERX**](https://p3terx.com)
