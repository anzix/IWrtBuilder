# Авто-сборщик [ImmortalWrt](https://github.com/immortalwrt/immortalwrt) (Китайский форк OpenWrt)

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

    2. В режиме tftp загружаем из репозитория `initramfs-recovery.itb` чтобы загрузится
       в ImmortalWrt (из ОЗУ)
    3. Внутри врменной ImmortalWrt загружаем `squashfs-sysupgrade.itb`

  - Если же вы на прошивке от OpenWrt/Kwrt и т.д, тогда прошивка происходит через
    меню LuCi с указанием файла `squashfs-sysupgrade.itb`

    Метод 1, из панели Luci

    - Переходим в `System` - `Backup / Flash Firmware`, в `Flash new firmware image`
      жмём `Flash image...` выбираем `squashfs-sysupgrade.itb`

      > [!WARNING]
      > Убираем галочку `Keep settings and retain the current configuration`, т.к
      > устройство должно загрузится начисто, без ваших конфигураций.

    Метод 2, консолью в ssh

    - Если же у вас не получается прошить графически, подключитесь к роутеру по
      ssh и используйте команду `sysupgrade` с аргументом `-n` (без сохранения
      данных)

      После прошиваем нажав `Continue`, подключаемся к панели только тогда когда
      индикатор на роутере будет гореть статично синим цветом

      ```sh
      sysupgrade -n immortalwrt-mediatek-filogic-netis_nx31-squashfs-sysupgrade.itb
      ```

## Преимущества

- Автоматический зафиксированный vermagic для использования официального kmod
  репозитория ImmortalWrt
- Встроенные пакеты для ускорения интернета 🚀
- Настроенный из коробки FullCone NAT, ZRam
- Дополнительный набор пакетов (современная тема, файловый менеджер, менеджер дисков)
- Настроенный https с самоподписанным сертификатом (openssl)
- Встроенный [скрипт для расширения встроенной памяти роутера](./files/usr/share/extpart.sh)

## Если вас не устраивают мои настройки?

Тогда [форкайте](https://github.com/anzix/IWrtBuilder/fork), используйте его как
шаблон и изменяйте под ваши нужды. Для понимания вот структура репозитория:

Каталоги и файлы в них:

- [.github/workflows/](./.github/workflows/)

  В этом каталоге хранятся файлы CI/CD для начального действия сборки а также повторяемых
  действий

  - [Build Filogic.yml](./.github/workflows/Build%20Filogic.yml) главный профиль
    сборки для платформы Filogic. Использует рецепты [`./.github/workflows/_openwrt-build-device.yml`](./.github/workflows/_openwrt-build-device.yml)
    для сборки каждого устройства и [`./.github/actions/openwrt-publish-release/action.yml`](./.github/actions/openwrt-publish-release/action.yml)
    для публикации

    > Можно настроить сборку по расписанию

  - [`_openwrt-build-device.yml`](./.github/workflows/_openwrt-build-device.yml)
    переиспользуемый workflow который использует [`Build Filogic.yml`](./.github/workflows/Build%20Filogic.yml).
    Содержит полную цепочку настройки виртуального окружения и сборки ImmortalWrt
    для каждого устройства. Является самым важным файлом

- [.github/workflows/](./.github/actions/)

  В этом каталоге содержатся файлы которые выполняют повторяющиеся действия после
  сборки (workflow) каждого устройства

  - [`openwrt-publish-release/action.yml`](./.github/actions/openwrt-publish-release/action.yml)
    объединяет артефакты для отдельных устройств, формирует описание release-body,
    публикует релиз на Github и удаляет старые экземпляры запусков

  - [`openwrt-release-firmware/action.yml`](./.github/actions/openwrt-release-firmware/action.yml)
    подготавливает целевой каталог для файлов прошивок, загружает его, создаёт
    release-tag и удалите старые артефакты.

- Полезные вспомогательные shell скрипты внутри [./files/](./files/): Накладывает
  на rootfs прошивку файлы в целевую файловую систему

- Профили устройства [./config/](./config/): Так называемые defconfig (содержит
  модули ядра, наличия каких-то оф. пакетов, feed репозитории и т.д):

  У каждого устройства собственный `.config`, как например [netis_nx31.config](./config/netis_nx31.config)

  > Также в данном каталоге могут быть файлы фукнкции, дополняющий текущий defconfig
  > устройства. Например [./config/docker.config](./config/docker.config) применяется
  > к текущему defconfig профилю устройства если в `Should Docker be compiled?`
  > выбран `true`

- Настройки при первом запуске: [zzz-default-settings](./default-settings-anzix/files/zzz-default-settings)
- Кастомные пакеты: [git-clone.sh](./sh/git-clone.sh), указываются в блоке `REPOS`
- Специфичные настройки (фиксирование хеша vermagic, применение собственных патчей
  и т.д): [specific-setup.sh](./sh/specific-setup.sh)

До начала сборки прошивки необходимо в вашем форк репозитории **зайти в
Settings - Actions, General** и в **`Workflow permissions`** поменять с **`Read
repository contents and packages permissions`** на **`Read and write permissions`**
и нажать **`Save`**

Это исправляет ошибку на этапе **`Generate unified publishing tags and content.`**

```txt
⚠️ GitHub release failed with status: 403
{"message":"Resource not accessible by integration","documentation_url":"https://docs.github.com/rest/releases/releases#create-a-release","status":"403"}
Skip retry — your GitHub token/PAT does not have the required permission to create a release
⚠️ Unexpected error fetching GitHub release for tag refs/heads/main: HttpError: Resource not accessible by integration - https://docs.github.com/rest/releases/releases#create-a-release
Error: Resource not accessible by integration - https://docs.github.com/rest/releases/releases#create-a-release
```

## Время сборки

В первый раз (или при большом перерыве) прошивка собирается где-то за 2-3 часа
(может чуть больше). После успешной сборки **и создания кэша** при повторной
сборки - 1 час (иногда чуть дольше, +15-40 минут). Если повезёт то вообще за
40~50 минут.

## Проблемы и способы их решения

1. В панели Luci во **Backup / Flash Firmware** при загрузке прошивки вылезает
   ошибка `XHR request timed out`

   Решение: Используйте `sysupgrade -n ...` из под консоли роутера, предварительно
   подключившись по ssh.

2. Текущие зеркала выдают ошибку `Bad gateway 502`

   Решение: Временно перейти на другое зеркало, например сами авторы ImmortalWrt
   в [тг канале](https://t.me/ctcgfw_project_openwrt/55) советуют использовать
   из [help.mirrorz.org](https://help.mirrorz.org/immortalwrt/)

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

3. При обновлении индексов пакета возникает множество ошибок связанных с
   `wgetSSL verify error: certificate is not yet valid`

   Это происходит из-за того, что при первом включении системное время указано
   неверно

   Переходим в `System` - `General Settings` и в `Local Time` жмём `Sync with browser`,
   после чего обновление индексов пакетов будет происходить нормально

## TODO

## Благодарность

- [qlxi/GL_AXT1800](https://github.com/qlxi/GL_AXT1800)
- [m0eak/Openwrt_Builder](https://github.com/m0eak/Openwrt_Builder)
- [P3TERX/Actions-OpenWrt](https://github.com/P3TERX/Actions-OpenWrt)
- [wukongdaily/AutoBuildImmortalWrt](https://github.com/wukongdaily/AutoBuildImmortalWrt)

## Лицензия

[MIT](https://github.com/anzix/IWrtBuilder/blob/main/LICENSE)
