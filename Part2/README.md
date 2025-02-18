# **🔥 Полное руководство по настройке корпоративной IT-инфраструктуры**

### **⏳ Временной лимит: 6 часов**

### **🛠️ Что будем делать?**

- Разворачивать виртуальные машины с Astra Linux, MikroTik и pfSense
- Настраивать сеть, DHCP, NAT, VPN, Firewall
- Включать контроль доступа и мониторинг безопасности
- Использовать отечественные решения (Astra, ALD, Eltex, CyberProtect)

---

# **📌 Часть 1: Разворачивание виртуальной инфраструктуры (1 час)**

## **🖥️ 1.1. Подготовка оборудования**

Перед настройкой нужно убедиться, что у тебя есть:  
✅ Компьютер / сервер с достаточной мощностью (8 CPU, 16GB RAM)  
✅ Установленный **KVM/QEMU**, **VMware ESXi** или **VirtualBox**  
✅ ISO-образы для установки операционных систем

### **📂 1.2. Скачивание образов**

**MikroTik CHR (Cloud Hosted Router)**  
🔗 [Скачать с официального сайта](https://mikrotik.com/download)  
Выбираем: **CHR RAW Disk Image**

**Astra Linux (FW1, C1, C2)**  
🔗 [Скачать Astra Linux](https://astralinux.ru/download/)  
Выбираем: **Astra Linux Common Edition / Special Edition**

**pfSense (FW2, IDS/IPS)**  
🔗 [Скачать pfSense](https://www.pfsense.org/download/)  
Выбираем: **ISO Installer**

---

## **🛠️ 1.3. Создание виртуальных машин**

### **KVM (QEMU)**

1. Создаем диск для MikroTik CHR:

```bash
qemu-img create -f qcow2 mikrotik-chr.qcow2 512M
```

2. Запускаем установку MikroTik:

```bash
qemu-system-x86_64 -hda mikrotik-chr.qcow2 -m 256 -enable-kvm -net nic -net user
```

### **VMware / VirtualBox**

1. Создаем ВМ
2. Указываем **CPU: 2, RAM: 2GB, HDD: 10GB**
3. Подключаем скачанный ISO
4. Запускаем установку

---

# **📌 Часть 2: Настройка сети (1.5 часа)**

## **🌐 2.1. Настройка MikroTik (Маршрутизация и DHCP)**

### **🔹 2.1.1. Настройка базового IP**

Заходим в MikroTik:

```bash
/ip address add address=192.168.1.1/24 interface=ether1
```

**Проверка:**

```bash
/ip address print
```

Если видим `192.168.1.1/24` — всё ок.

---

### **🔹 2.1.2. Настройка DHCP**

1. Создаем пул адресов:

```bash
/ip pool add name=dhcp-pool ranges=192.168.1.10-192.168.1.200
```

2. Настраиваем DHCP-сервер:

```bash
/ip dhcp-server add name=dhcp1 interface=ether1 address-pool=dhcp-pool disabled=no
```

3. Указываем настройки сети:

```bash
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=8.8.8.8
```

**Проверка:**

```bash
/ip dhcp-server lease print
```

---

### **🔹 2.1.3. Настройка NAT**

```bash
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

**Проверка доступа в интернет:**

```bash
ping 8.8.8.8
```

Если пингуется – отлично, NAT работает.

---

## **🛡️ 2.2. Настройка Firewall (pfSense)**

1. Заходим в Web GUI **(https://192.168.1.2:443)**
2. Переходим в **Firewall → Rules → WAN**
3. Нажимаем **Add Rule**
4. **Protocol:** TCP
5. **Destination Port:** 22 (SSH) / 443 (Web)
6. **Action:** Pass
7. **Apply Changes**

---

# **📌 Часть 3: Настройка VPN (1.5 часа)**

## **🔑 3.1. OpenVPN на MikroTik**

1. Включаем сервер:

```bash
/interface ovpn-server server set enabled=yes certificate=server-cert require-client-certificate=yes
```

2. Создаем пользователя:

```bash
/ppp secret add name=vpn-user password=StrongPass service=ovpn
```

3. Клиентская настройка (на рабочей станции):

```bash
/interface ovpn-client add connect-to=192.168.1.1 user=vpn-user password=StrongPass disabled=no
```

4. Проверяем подключение:

```bash
/interface ovpn-server print
```

---

# **📌 Часть 4: Мониторинг и безопасность (1.5 часа)**

## **🚨 4.1. IDS (Intrusion Detection System)**

### **🛠️ 4.1.1. Установка Suricata (на Astra Linux)**

```bash
apt install suricata
```

### **📜 4.1.2. Добавляем правило обнаружения атак**

```bash
echo "alert icmp any any -> any any (msg:'ICMP Packet Detected'; sid:1000001;)" >> /etc/suricata/rules/local.rules
```

### **🔄 4.1.3. Перезапускаем IDS**

```bash
systemctl restart suricata
```

---

# **📌 Часть 5: Финальная настройка и сдача (1 час)**

## **📑 5.1. Создание документации**

**Обязательно включить в отчет:**
✅ Схему сети  
✅ IP-адреса и настройки сервисов  
✅ Конфигурацию DHCP, NAT, VPN  
✅ Скриншоты интерфейсов

---

## **📂 5.2. Создание резервных копий**

**MikroTik:**

```bash
/export file=backup
```

**KVM/QEMU:**

```bash
virsh snapshot-create-as FW1 "Before Config" --description "Initial state"
```

**VMware:**

1. VM → Snapshot → Take Snapshot

---

## **🚀 5.3. Финальная проверка перед сдачей**

✅ Интернет работает  
✅ Раздаются IP через DHCP  
✅ VPN подключается  
✅ IDS обнаруживает атаки  
✅ Отчет составлен

---
