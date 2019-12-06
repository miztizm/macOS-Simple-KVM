# checkra1n на Linux через QEMU IOMMU Pass Through USB Controller

## Требования и клон скрипта (Ubuntu 18.04)

`wget https://raw.githubusercontent.com/miztizm/macOS-Simple-KVM/master/install.sh -v -O install.sh; chmod +x install.sh; ./install.sh`

## Проверьте, если виртуализация процессора включена

`dmesg | grep -E "DMAR|IOMMU"`

Если пустой вывод, то включите виртуализацию (virtualization) в BIOS 

## Вывести USB ID's

`lspci -nn | grep -i USB`

## Запишите первые цифры (XX:XX.XX) для QEMU, вторые для GRUB (XXXX:XXXX).

`00:14.0 USB controller [0c03]: Intel Corporation Cannon Lake PCH USB 3.1 xHCI Host Controller [8086:a36d] (rev 10)`

Числа могут отличаться и у всех разные.

## Добавьте вторые числа в grub. (XXXX:XXXX)

`sudo gedit /etc/default/grub`

Intel

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash iommu=pt intel_iommu=on vfio-pci.ids=8086:a36d"` 

AMD

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash iommu=pt amd_iommu=on vfio-pci.ids=8086:a36d"` 

## Обновитe GRUB

`sudo update-grub`

## Перезагрузка

`sudo reboot`

## Посмотрите если vfio включен

`dmesg | grep -i vfio`

##  Изменить файлы конфигурации (XX:XX.XX)

Отредактируйте нижнию строку в `macOS.sh` и замените число на `00:14.0`

`-device vfio-pci,host=00:14.0,bus=port.1,multifunction=on`

Отредактируйте `USBmacOS.sh` и замените число на `00:14.0` (Три нуля, нужно оставить)

`sudo ./driverctl/driverctl --nosave set-override 0000:00:14.0 vfio-pci`

## Убедитесь, что у вас есть macOS.qcow2 в папке.

Образ виртуального диска macOS.7z (С предустановленным checkra1n beta 0.9.6) 

Загрузить: https://drive.google.com/open?id=1h_Yrqkasg9oln0BrnZfZKSraAtPTC_Jv

Заметка: Виртуальная машина не имеет пароля. Если вам нyжно изменить паролья, для ввода комманд.

`sudo passwd checkra1n`

### Извлеките macOS.7z в вашу папку macOS-Simple-KVM

`sudo apt-get install dtrx`
`dtrx macOS.7z`

## Запустите виртуальную машину и привязка USB-контроллера

`sudo ./USBmacOS.sh`

Заметка: если у вас ошибка `qemu-system-x86_64: -device vfio-pci,host=00:14.0,bus=port.1,multifunction=on: vfio error: 0000:00:14.0: group 4 is not viable
Please ensure all devices within the iommu_group are bound to their vfio bus driver.`

### Добовляем связанные usb девайсы (XX:XX.XX)

Вам нужно заново перепривязать из группы вашего контролера, все остальные от него связанные подключения.
Смотрим список:

`lspci -nn`

Найдите 00:14.0 После него должны быть и другие ( 00:14.2, 00:14.3, 00:14.5 )

Отредактируйте `USBmacOS.sh` добавьте после первой строчки другие девайсы:

`sudo ./driverctl/driverctl --nosave set-override 0000:00:14.2 vfio-pci
sudo ./driverctl/driverctl --nosave set-override 0000:00:14.3 vfio-pci
sudo ./driverctl/driverctl --nosave set-override 0000:00:14.5 vfio-pci`


# Подключите ваш iPhone/iPad/iPod и запустите checkra1in и следуйте инструкциям! Ня, Apple!

## Если вы хотите скачать официальный BaseSystem.dmg для Mac OS, вы можете следовать оригиналу [macOS-Simple-KVM](https://github.com/foxlet/macOS-Simple-KVM/blob/master/README.md) 

## [Более четкие инструкции на эту тему у vmra1n](https://www.reddit.com/r/jailbreak/comments/dxdmua/tutorial_detailed_guide_on_how_to_run_checkra1n/)
