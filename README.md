# MacOSMojaveOnHyper-V
Установка MacOS Mojave или Catalina на Hyper-V

# Цель
Установка MacOS Mojave или Catalina гипервизор Hyper-V и запуск в полностью автоматическом режиме с доступностью хоста из внутренней сети. 
Так как основная цель сборка приложений, то мы не учитываем работу с графической подсистемой. Работает сносно, но для постоянной работы лучше купить железо.


## Сокращения
* HP - машина на которой установлен Hyper-V
* Ubuntu18 - Виртуальная машина на которой будет разворачиваться KVM
* MacHost - Виртуальная машина MacOS Mojave или Catalina 

## Ограничения
Данная инструкция подразумевает что ваш процессор поддерживает инструкции **VT-x** и **EPT** поэтому будет работать только на процессорах **Intel** .


# Установка

## Подготовка

Создайте виртуальную машину со следующими параметрами
1. Количество процессоров 4
2. Поколение 2
3. Количество памяти 8192 Мб
4. Размер жесткого диска 80 - 127 Гб

Все остальные параметры выберите по своему усмотрению

На гипервизоре выполните команду в powershell

```powershell
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
VMName - в моем случае Ubuntu18
```powershell
Set-VMProcessor -VMName Ubuntu18 -ExposeVirtualizationExtensions $true
```


Установка переменных, добавьте следующую строку в файл **/etc/modprobe.d/kvm.conf**
```sh
# /etc/modprobe.d/kvm.conf
echo 1 > /sys/module/kvm/parameters/ignore_msrs
```

Установка переменных, добавьте следующую строку в файл **/etc/sysctl.conf**
```sh
# /etc/sysctl.conf
net.ipv4.ip_forward=1
```

Установка пакетов 

```sh
apt install -y qemu qemu-kvm libvirt0 virt-manager bridge-utils libvirt-daemon-system libvirt-clients uml-utilities libguestfs-tools
```

Клонирование репозитория
```sh
cd ~
git clone https://github.com/kholia/OSX-KVM.git
cd OSX-KVM
```

Получение образа
```sh
./fetch-macOS.py
```
Выбираете подходящий, я использовал Mojave

Конвертируете образ
```sh
dmg2img BaseSystem.dmg BaseSystem.img
```

Создаете диск для виртуальной машины
```<language>
qemu-img create -f qcow2 mac_hdd_ng.img 128G
```

Создаете сеть для установки, позже мы настроим сеть более вдумчиво

```sh
sudo ip tuntap add dev tap0 mode tap
sudo ip link set tap0 up promisc on
sudo ip link set dev virbr0 up
sudo ip link set dev tap0 master virbr0
```

Запускаете сеть для установки
```sh
virsh net-start default

virsh net-autostart default
```


Итак, все готово для установки. 

Отредактируйте скрипт **boot-macOS-NG.sh** задав значения для памяти сменив **-m 3072** **-m 8192** в противном случае вы будете ждать очень долго!
Запускайте консоль

```sh
./boot-macOS-NG.sh
```
После этого процесс установки пройдет в штатном режиме. 

# Шаги после инсталляции

## Установка Clover на  загрузочный диск
См. https://artem.services/?p=722
## Автостарт виртуальной машины
 В процессе
## Настройка сети
 В процессе

# Файлы для контроля


# Ссылки
1. https://github.com/kholia/OSX-KVM
2. https://artem.services/?p=722
3. https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/nested-virtualization


