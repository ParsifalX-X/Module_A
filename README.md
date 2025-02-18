## **🔰 Общий план работы**

### **1️⃣ Сетевое ядро (MikroTik CHR-1, Firewall, NAT, DHCP)**

- **Настроить IP-адреса на всех интерфейсах.**
- **Поднять DHCP-клиента** (если требуется получение адреса от провайдера).
- **Настроить DHCP-сервер** (с учетом диапазона раздачи адресов).
- **Сделать NAT (маскарадинг)** для выхода в интернет.
- **Настроить базовые firewall-правила** (разрешить ICMP, закрыть ненужные порты, SSH).
- **Создать пользователей и группы** с нужными правами.
- **Бэкап и снапшоты виртуалок.**

### **2️⃣ Разворачивание серверов и рабочих станций (RED OS, Astra Linux, Basalt SPO)**

- **Установка и первичная настройка** ОС.
- **Добавление в домен ALD/Kerberos** (если требуется).
- **Настройка пользователей и политик безопасности.**
- **Настройка SUDO и аудита.**
- **Разграничение доступа.**

### **3️⃣ Сетевые службы (DNS, DHCP, VPN, Kerberos, ALD)**

- **Настройка DHCP-серверов** для филиалов.
- **Развертывание Kerberos/ALD** (если предусмотрено).
- **Настройка защищенного соединения между филиалами** (VPN/IPSec).
- **Подключение клиентов к сети.**

### **4️⃣ Безопасность и защита (InfoTeKS, CyberProtect, IDS)**

- **Развернуть IDS (интеграция с TIAS, HS, MC, NS).**
- **Добавить базы сигнатур атак.**
- **Настроить мониторинг событий.**
- **Добавить политки доступа в InfoTeKS.**
- **Развернуть и проверить CyberProtect.**

### **5️⃣ Блокировка контента и контроль трафика (Firewalls, WebFilter)**

- **Создать SocialMediaBlockList.**
- **Добавить правила для блокировки соцсетей/ненужных сервисов.**
- **Настроить контроль доступа.**
- **Перенаправление трафика через цепочки правил.**

### **6️⃣ Тестирование и отчетность**

- Проверить **все критерии** из документа.
- **Пинг, трассировка, доступность сервисов.**
- **Сделать финальные бэкапы.**
- Оформить **отчет** по требованиям.

---

## **🔥 Подробная инструкция по шагам**

### **🛠 1. Настройка MikroTik (CHR-1)**

#### **1.1 Настроить IP-адреса**

Прописываем IP на всех интерфейсах:

```bash
/ip address add address=192.168.1.1/24 interface=ether1
/ip address add address=10.10.10.1/24 interface=ether2
```

Проверяем:

```bash
/ip address print
```

#### **1.2 Включаем DHCP-клиента (если нужен)**

```bash
/ip dhcp-client add interface=ether1 disabled=no
/ip dhcp-client print
```

#### **1.3 Настраиваем DHCP-сервер**

```bash
/ip pool add name=dhcp_pool ranges=192.168.1.100-192.168.1.200
/ip dhcp-server add name=dhcp1 interface=ether2 address-pool=dhcp_pool disabled=no
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1
```

Проверяем:

```bash
/ip dhcp-server print
```

#### **1.4 Настройка NAT**

```bash
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

#### **1.5 Настройка Firewall**

Закрываем все порты, кроме локалки и SSH:

```bash
/ip firewall filter add chain=input connection-state=established,related action=accept
/ip firewall filter add chain=input protocol=tcp dst-port=22 action=accept
/ip firewall filter add chain=input protocol=icmp action=accept
/ip firewall filter add chain=input action=drop
```

#### **1.6 Создаем пользователей**

```bash
/user add name=admin2 group=full password=StrongPass123!
/user group add name=restricted policy=local,read
```

---

### **🛠 2. Установка RED OS / Astra Linux / Basalt SPO**

1. Устанавливаем ОС.
2. Добавляем пользователей:

```bash
adduser admin
passwd admin
```

3. Включаем SSH:

```bash
systemctl enable sshd
systemctl start sshd
```

4. Добавляем в домен ALD (если есть).
5. Настраиваем sudo и права доступа.

---

### **🛠 3. VPN и безопасность (InfoTeKS, ALD, IDS, CyberProtect)**

#### **3.1 Развертывание VPN (InfoTeKS)**

1. Устанавливаем ПО InfoTeKS.
2. Настраиваем VPN-туннель между филиалами.
3. Проверяем соединение:

```bash
ping <IP удаленного филиала>
```

#### **3.2 Развертывание ALD и Kerberos**

```bash
apt install ald-init
ald-init setup --server
```

Создаем домен:

```bash
ald-init domain-create company.local
```

#### **3.3 IDS и мониторинг (TIAS, CyberProtect)**

1. Разворачиваем IDS (NS, HS, MC).
2. Настраиваем сенсоры для обнаружения атак.
3. Добавляем устройства в систему мониторинга.

---

### **🛠 4. Блокировка контента и контроль доступа**

#### **4.1 Блокируем соцсети**

Создаем список:

```bash
/ip firewall address-list add list=SocialMediaBlockList address=facebook.com
/ip firewall address-list add list=SocialMediaBlockList address=instagram.com
```

Запрещаем доступ:

```bash
/ip firewall filter add chain=forward src-address-list=SocialMediaBlockList action=drop
```

#### **4.2 Ограничение трафика**

```bash
/ip firewall filter add chain=forward protocol=tcp dst-port=139,445 action=accept
/ip firewall filter add chain=forward protocol=tcp action=drop
```

---

### **🛠 5. Проверка, отчеты и снапшоты**

1. **Тестируем работу сервисов:**
   - Пинг между филиалами.
   - Доступность DHCP, DNS, VPN.
   - Защита по Firewall.
2. **Сохраняем снапшоты ВМ:**

```bash
/qemu-img snapshot -c final_snapshot my_vm.qcow2
```

3. **Создаем резервную копию MikroTik:**

```bash
/export file=config_backup
```

4. **Оформляем отчет** по требованиям.

---
