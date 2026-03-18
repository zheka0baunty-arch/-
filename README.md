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
🖥️ Настройка через ЦУС (графика)
Рассмотрим настройку сетевых интерфейсов через ЦУС:
1. Запуск Центра управления системой (ЦУС) 
Шаг: Открыть меню → «Система» → «Центр управления системой» (или Alt+F2 → sus). 
