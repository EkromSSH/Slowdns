#!/bin/bash

# Clear screen
clear

# Color variables
G="\e[1;32m"
W="\e[0;39m"

# Variables to store details
NSNAME=""
SSH_PORTS=()
SERVER_PUB=""
SERVER_KEY=""
SLOWDNS_PORT=""

# Function to install and configure badvpn
install_udpgw() {
    echo -e "${G}ติดตั้ง badvpn-udpgw${W}"
    wget -O /usr/bin/badvpn-udpgw "http://slowly.bmswaoi.uk:81/kasang/badvpn-udpgw64"
    chmod +x /usr/bin/badvpn-udpgw

    cat > /etc/systemd/system/svr-7300.service <<EOF
[Unit]
Description=badvpn udpgw 7300 HideSSH
Documentation=https://hidessh.com
After=network.target nss-lookup.target

[Service]
Type=simple
User=root
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
Restart=on-failure
ExecStart=/usr/bin/badvpn-udpgw --listen-addr 127.0.0.1:7300 --max-clients 500

[Install]
WantedBy=multi-user.target
EOF

    chmod +x /etc/systemd/system/svr-7300.service
    systemctl daemon-reload
    systemctl start svr-7300.service
    systemctl enable svr-7300.service
    echo -e "${G}ติดตั้ง badvpn-udpgw เสร็จสมบูรณ์${W}"
}

# Function to install and configure slowdns
install_slowdns() {
    echo -e "${G}ติดตั้ง SlowDNS${W}"
    mkdir /etc/slowdns
    cd /etc/slowdns
    rm -f dns-server
    wget https://github.com/khaledagn/DNS-AGN/raw/main/dns-server
    chmod +x dns-server
    sleep 0.1

    rm -f server.key
    cat >> server.key <<end
9b46872c6351d5d91ad2cd7fe5efbfded79ce8f0744629962f9b92884a8d588a
end
    sleep 0.1

    rm -f server.pub
    cat >> server.pub <<end
526bac803bbb09f19a36ea3e424c62c00f72e0e33308ac3e18dd1d3589d53b71
end
    sleep 0.1

    chmod +x /etc/slowdns/server.key /etc/slowdns/server.pub

    read -p "กรุณาใส่ nameserver: " NSNAME
    read -p "กรุณาใส่พอร์ต SlowDNS: " SLOWDNS_PORT

    cat > /usr/local/bin/thai4gslowdns.sh <<EOF
#!/bin/bash
PATH=/opt/someApp/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
cd /etc/slowdns
./dns-server -udp :5300 -privkey-file /etc/slowdns/server.key $NSNAME 0.0.0.0:$SLOWDNS_PORT
date >> /root/disk_space_report.txt
EOF

    chmod +x /usr/local/bin/thai4gslowdns.sh

    cat > /etc/systemd/system/thai4gslowdns.service <<EOF1
[Unit]
After=network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/thai4gslowdns.sh

[Install]
WantedBy=multi-user.target
EOF1

    chmod +x /etc/systemd/system/thai4gslowdns.service
    systemctl daemon-reload
    systemctl enable thai4gslowdns.service
    systemctl start thai4gslowdns.service

    # Configure iptables for slowdns
    iptables -I INPUT -p udp --dport 5300 -j ACCEPT
    iptables -t nat -I PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5300
    iptables-save > /etc/iptables.conf

    SERVER_PUB=$(cat /etc/slowdns/server.pub)
    SERVER_KEY=$(cat /etc/slowdns/server.key)

    echo -e "${G}การติดตั้ง slowdns เสร็จสมบูรณ์${W}"
}

# Function to add SSH port
add_ssh_port() {
    read -p "กรุณาใส่หมายเลข port สำหรับ SSH: " port

    if grep -q "Port $port" /etc/ssh/sshd_config; then
        echo -e "${G}พอร์ต $port มีอยู่แล้วในไฟล์ sshd_config${W}"
    else
        ufw allow $port/tcp
        echo "Port $port" >> /etc/ssh/sshd_config
        service ssh restart
        SSH_PORTS+=("$port")
        echo -e "${G}เพิ่ม port SSH $port เรียบร้อยแล้ว${W}"
    fi
}

# Function to show details
show_details() {
    echo -e "${G}รายละเอียดการติดตั้ง:${W}"
    echo -e "${G}Nameserver:${W} $NSNAME"
    echo -e "${G}Port SSH:${W} ${SSH_PORTS[*]}"
    echo -e "${G}SlowDNS Port:${W} $SLOWDNS_PORT"
    echo -e "${G}server.pub:${W} $SERVER_PUB"
    echo -e "${G}server.key:${W} $SERVER_KEY"
}


# Function to stop services
stop_services() {
    echo -e "${G}หยุดบริการทั้งหมด...${W}"
    systemctl stop svr-7300.service
    systemctl stop thai4gslowdns.service
    echo -e "${G}หยุดบริการเสร็จสมบูรณ์${W}"
}

# Function to uninstall services
uninstall_services() {
    echo -e "${G}ถอนการติดตั้งบริการทั้งหมด...${W}"
    systemctl stop svr-7300.service
    systemctl disable svr-7300.service
    rm /etc/systemd/system/svr-7300.service

    systemctl stop thai4gslowdns.service
    systemctl disable thai4gslowdns.service
    rm /etc/systemd/system/thai4gslowdns.service
    rm -rf /etc/slowdns
    rm /usr/local/bin/thai4gslowdns.sh

    ufw delete allow 5300/udp
    ufw delete allow 53/udp
    echo -e "${G}ถอนการติดตั้งเสร็จสมบูรณ์${W}"
}

# Function to update configuration
update_config() {
    read -p "กรุณาใส่ nameserver ใหม่: " NSNAME
    read -p "กรุณาใส่พอร์ต SlowDNS ใหม่: " SLOWDNS_PORT
    sed -i "s|./dns-server -udp :5300 -privkey-file /etc/slowdns/server.key .*|./dns-server -udp :5300 -privkey-file /etc/slowdns/server.key $NSNAME 0.0.0.0:$SLOWDNS_PORT|" /usr/local/bin/thai4gslowdns.sh
    systemctl restart thai4gslowdns.service
    echo -e "${G}อัปเดต nameserver และพอร์ต SlowDNS เสร็จสมบูรณ์${W}"
}

# Function to enable root access
enable_root() {
    wget https://netfree.in.th/script/spnet/Modulos/senharoot
    chmod +x senharoot
    ./senharoot
    echo -e "${G}เปิดสิทธิ์ root เรียบร้อยแล้ว${W}"
}

# Main menu
menu() {
    while true; do
        clear
        echo -e "${G}==================== เมนู ====================${W}"
        echo "1. ติดตั้ง badvpn-udpgw"
        echo "2. ติดตั้ง SlowDNS"
        echo "3. เพิ่ม port SSH"
        echo "4. ดูรายละเอียด"
        echo "5. หยุดบริการทั้งหมด"
        echo "6. ถอนการติดตั้งทั้งหมดพร้อมหยุดบริการ"
        echo "7. อัปเดตการตั้งค่า"
        echo "8. เปิดสิทธิ์ root"
        echo "9. ออกจากโปรแกรม"
        echo -e "${G}==============================================${W}"
        read -p "กรุณาเลือกตัวเลือก [1-9]: " choice

        case $choice in
            1) install_udpgw ;;
            2) install_slowdns ;;
            3) add_ssh_port ;;
            4) show_details ;;
            5) stop_services ;;
            6) uninstall_services ;;
            7) update_config ;;
            8) enable_root ;;
            9) exit 0 ;;

            *) echo -e "${R}ตัวเลือกไม่ถูกต้อง! กรุณาลองใหม่อีกครั้ง${W}" ;;
        esac
        read -p "กด Enter เพื่อกลับไปที่เมนูหลัก..."
    done
}

# Check if script is sourced
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    menu
fi

# Create menu_script.sh
cat > /usr/local/bin/menu_script.sh <<EOF
#!/bin/bash
source /path/to/install_script.sh
menu
EOF

chmod +x /usr/local/bin/menu_script.sh

echo -e "${G}การติดตั้งเสร็จสมบูรณ์ คุณสามารถเรียกเมนูได้ด้วยคำสั่ง: /usr/local/bin/menu_script.sh${W}"
