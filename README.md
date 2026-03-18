# Практика: Настройка сетевых параметров в Alt Linux

**Оригинальная лекция:** [Лекция №09 Настройка сетевых параметров](https://github.com/aleti000/fgos2023/blob/main/mdk02.01/%D0%BC%D0%B0%D1%82%D0%B5%D1%80%D0%B8%D0%B0%D0%BB/%D0%9B%D0%B5%D0%BA%D1%86%D0%B8%D0%B8/%D0%9B%D0%B5%D0%BA%D1%86%D0%B8%D1%8F%20%E2%84%9609%20%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0%20%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D1%8B%D1%85%20%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%BE%D0%B2.md#лекция-09-настройка-сетевых-параметров)

## 📋 Основные инструменты диагностики

| Команда | Описание |
|---------|----------|
| `ip a` | Состояние интерфейсов и IP |
| `ip route` | Таблица маршрутизации |
| `ping 8.8.8.8` | Проверка доступности |
| `nmcli` | NetworkManager CLI |
| `ss -tulpn` | Активные соединения |

## 🔧 Настройка через etcnet (/etc/net/ifaces/eth0/)

### 1. Основные команды
```bash
ifup eth0      # Поднять интерфейс
ifdown eth0    # Опустить интерфейс  
ifup -a        # Все интерфейсы
systemctl restart network  # Перезапуск сети
```
2. Структура файлов конфигурации
```bash
options (статический IP):
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
CONFIG_IPV4=yes
ONBOOT=yes

ipv4address:
192.168.0.2/24

ipv4route:
default via 192.168.0.1
10.0.0.0/24 via 192.168.0.254

resolv.conf:
search mydom1.local domain2.ru
nameserver 8.8.8.8
nameserver 192.168.0.5
```
3. Минимальная статическая настройка
```bash
mkdir -p /etc/net/ifaces/eth0
# Заполнить options, ipv4address, ipv4route
ifdown eth0 && ifup eth0
```
# 🖥️ Настройка через ЦУС (графика)
Рассмотрим настройку сетевых интерфейсов через ЦУС:
## 1. Запуск Центра управления системой (ЦУС) 
Шаг: Открыть меню → «Система» → «Центр управления системой» (или Alt+F2 → sus).
<img width="518" height="418" alt="image" src="https://github.com/user-attachments/assets/73cee591-24f1-4c5f-b050-93328add590a" />

## 2. Переход в раздел «Сеть»
Шаг: В главном окне ЦУС найти и кликнуть по иконке/разделу «Сеть» или «Сетевые подключения».
<img width="606" height="88" alt="image" src="https://github.com/user-attachments/assets/500e4eaa-ceb1-438c-a064-f70f8270a02d" />

## 3. Выбор интерфейса для настройки
Шаг: В списке сетевых подключений найти проводной интерфейс (eth0, enp0s3 или по имени вашей NIC). Кликнуть правой кнопкой → «Изменить подключение» или «Настроить».
<img width="594" height="404" alt="image" src="https://github.com/user-attachments/assets/9128d139-c782-4adb-bbc6-54fe4b147a70" />

## 4. Переключение на ручную настройку IPv4
Шаг: Во вкладке «IPv4» изменить метод подключения с «Автоматически (DHCP)» на «Ручная».
<img width="605" height="409" alt="image" src="https://github.com/user-attachments/assets/657de5f6-ac84-41d7-8585-28bbaebff5ad" />
Шаг: Выбрать «Ручная» → появляются поля для ввода адреса, маски, шлюза.

## 5. Ввод параметров IP-адресации
Заполнить поля:
    Адрес: 192.168.0.19
    Маска подсети: 255.255.255.0 (или /24)
    Шлюз: 192.168.0.1 

## 6. Настройка DNS-серверов
Шаг: В той же вкладке IPv4 прокрутить вниз до раздела DNS → добавить:
    8.8.8.8
<img width="607" height="413" alt="image" src="https://github.com/user-attachments/assets/dcd28758-487c-4993-bd62-10826e01eac1" />

## 7. Включение автозапуска
Шаг: Поставить галочку «Автоматически подключаться к этой сети» (или аналогичный пункт).
<img width="606" height="410" alt="image" src="https://github.com/user-attachments/assets/47e82942-1654-49df-88a8-7129062d5715" />

## 8. Проверка результата (терминал)
ip a show dev enp0s3 
ip route show 
ping -c3 192.168.0.1
<img width="666" height="329" alt="image" src="https://github.com/user-attachments/assets/c2aba1dd-55f6-4a18-baac-966c6655f24d" />

## ⚡ Настройка через nmcli

### 1. Диагностика состояния
```bash
nmcli connection show     # Все подключения NetworkManager
nmcli device status       # Состояние интерфейсов
```

### 2. Статический IP (полная команда)
nmcli connection add type ethernet ifname eth0 con-name eth0-static \
ipv4.method manual \
ipv4.addresses 192.168.0.2/24 \
ipv4.gateway 192.168.0.1 \
ipv4.dns "8.8.8.8 192.168.0.2" \
connection.autoconnect yes

### 3. Управление подключением
nmcli connection show eth0-static    # Проверка профиля
nmcli connection up eth0-static      # Активация
nmcli connection down eth0-static    # Деактивация
nmcli connection delete eth0-static  # Удаление

### 4. DHCP-подключение
nmcli connection add type ethernet ifname eth0 con-name eth0-dhcp \
ipv4.method auto connection.autoconnect yes
nmcli connection up eth0-dhcp

### 5. Проверка результата
ip a show dev eth0                    # IP-адрес интерфейса
ip route                              # Таблица маршрутизации
ping -c3 192.168.0.1 && ping -c3 8.8.8.8 && ping -c3 ya.ru

### 6. Быстрое переключение профилей
nmcli connection down eth0-static
nmcli connection up eth0-dhcp

### 7. Основные команды управления
| Действие        | Команда                                                              |
| --------------- | -------------------------------------------------------------------- |
| Список активных | nmcli connection show --active                                       |
| Изменить IP     | nmcli connection modify eth0-static ipv4.addresses "192.168.1.10/24" |
| Перезагрузить   | nmcli connection up eth0-static                                      |

🚀 Рабочий цикл администратора

1. nmcli device status     ← что подключено?
2. nmcli connection show   ← какие профили есть?
3. nmcli connection add    ← создать/изменить
4. nmcli connection up     ← активировать
5. ip a + ping             ← проверить
6. nmcli connection down   ← переключиться

Преимущество nmcli: атомарные команды, без редактирования файлов, легко в скрипты и Ansible.
