
# Практическая часть: Настройка VPN-сервера на базе OpenVPN в VirtualBox

## Цель
Цель практической части — развернуть VPN-сервер на базе OpenVPN в виртуальной машине с Linux (Ubuntu), чтобы продемонстрировать, как использование VPN может защищать трафик при передаче данных, в том числе при работе с банковскими картами в интернете.

## Обоснование
При оплате товаров и услуг банковской картой через интернет пользователь передаёт конфиденциальную информацию (номер карты, CVV, одноразовые коды и т. д.). Без использования защищённого канала данные могут быть перехвачены злоумышленниками. VPN-соединение позволяет зашифровать трафик и снизить риски при передаче данных.

## Оборудование и программное обеспечение
- Хостовая ОС: Windows 10  
- VirtualBox  
- Гостевая ОС: Ubuntu Server 22.04  
- OpenVPN  
- Easy-RSA (для генерации сертификатов)  

## Этап 1: Установка Ubuntu в VirtualBox
1. Создана виртуальная машина в VirtualBox с параметрами:  
   - Тип: Linux  
   - ОЗУ: 2048 МБ  
   - Сеть: "Сетевой мост" или NAT  
2. Установлена Ubuntu Server 22.04 из ISO-образа.  
3. Настроен доступ по SSH (опционально).  

## Этап 2: Установка и настройка OpenVPN
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openvpn easy-rsa -y

make-cadir ~/openvpn-ca
cd ~/openvpn-ca

./easyrsa init-pki
./easyrsa build-ca

./easyrsa gen-req server nopass
./easyrsa sign-req server server

./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1

./easyrsa gen-dh
openvpn --genkey --secret ta.key
```

## Этап 3: Настройка конфигурации сервера
```bash
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf

echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Настроены пути к ключам и сертификатам в `server.conf`. Настроены iptables.

## Этап 4: Запуск сервера и проверка
```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
sudo systemctl status openvpn@server
```

## Этап 5: Создание клиентского .ovpn-файла и подключение
Создан `.ovpn` файл с:
- Адресом сервера  
- Клиентскими ключами  
- CA-сертификатом  
- TLS-ключом  

Настроено подключение через OpenVPN клиент на Windows.

## Заключение
VPN-сервер на базе OpenVPN успешно развёрнут в виртуальной машине. Такое решение позволяет зашифровать трафик и защищает передаваемые данные, включая банковскую информацию. Использование VPN особенно актуально при работе в общественных сетях или недоверенных Wi-Fi-точках.
