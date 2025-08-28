#!/bin/bash

# ========================================
# 👑 KING VIP IP - Ultra Fast Professional Scanner 👑
# ========================================

# Reňkler
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
RED='\033[0;31m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'

# ASCII art uly başlyk emoji bilen
if command -v figlet &> /dev/null
then
    echo -e "${YELLOW}"
    figlet -f slant "👑 KING VIP IP 👑"
    echo -e "${NC}"
else
    echo -e "${YELLOW}"
    echo "========================================"
    echo "        👑 KING VIP IP 👑             "
    echo "========================================"
    echo -e "${NC}"
fi

# IP girizmek
read -p "IP giriziň (mysal: 192.168.1.1): " ip

# CSV hasabat faýly
report="KING_VIP_IP_Report_$(date +%Y%m%d_%H%M%S).csv"
echo "IP,Status,Host,Open Ports,Traceroute,MAC" > $report

# Progress bar funksiýasy
progress_bar() {
    local duration=$1
    already_done() { for ((done=0; done<$1; done++)); do printf "█"; done }
    remaining() { for ((remain=$1; remain<50; remain++)); do printf " "; done }
    for ((i=0; i<=50; i++)); do
        printf "\rProgress : |"
        already_done $i
        remaining $i
        printf "| %d%%" $((i*2))
        sleep $duration
    done
    echo ""
}

# Interaktiw menu
while true; do
    echo -e "${CYAN}"
    echo "========================================"
    echo "           Funksiýa Menu                "
    echo "========================================"
    echo "1️⃣  Ping barla"
    echo "2️⃣  DNS lookup"
    echo "3️⃣  Port scan (1–65535)"
    echo "4️⃣  Traceroute"
    echo "5️⃣  MAC info"
    echo "6️⃣  Hemmesini barla (Full Scan)"
    echo "0️⃣  Çyk"
    echo -e "${NC}"

    read -p "Saýlawyňy giriziň: " choice

    case $choice in
        1)
            echo -e "${CYAN}🔍 Ping barlaýar...${NC}"
            ping -c 4 $ip
            ;;
        2)
            echo -e "${CYAN}🌐 DNS lookup...${NC}"
            nslookup $ip
            ;;
        3)
            echo -e "${CYAN}🔍 Port scan 1–65535 (paralel) ⏳${NC}"
            seq 1 65535 | parallel -j200 nc -z -w1 $ip {} &> /dev/null && echo "🔓 Port {} açyk"
            ;;
        4)
            echo -e "${CYAN}🛣️ Traceroute...${NC}"
            traceroute $ip
            ;;
        5)
            echo -e "${CYAN}💻 MAC info (lokal subnet)...${NC}"
            arp -n $ip
            ;;
        6)
            echo -e "${CYAN}🚀 Hemmesini barlaýar... ⏳${NC}"
            # Ping
            ping -c 1 -W 1 $ip &> /dev/null
            if [ $? -eq 0 ]; then
                status="${GREEN}✅ Açyk${NC}"
            else
                status="${RED}❌ Ýapyk${NC}"
            fi
            # DNS
            host_name=$(nslookup $ip 2>/dev/null | grep 'name =' | awk '{print $4}' | sed 's/\.$//')
            host_name=${host_name:-"❓ Unknown"}
            # Traceroute
            trace=$(traceroute -m 3 $ip 2>/dev/null | tail -n 1 | awk '{print $2}')
            trace=${trace:-"❓ Unknown"}
            # MAC
            mac=$(arp -n $ip 2>/dev/null | awk '/ether/ {print $3}')
            mac=${mac:-"❓ Unknown"}

            # Port scan 1–65535 parallel
            echo -e "${CYAN}🔍 Port scan 1–65535 (paralel) ⏳${NC}"
            open_ports=$(seq 1 65535 | parallel -j200 nc -z -w1 $ip {} && echo {} | tr '\n' ' ')

            echo -e "IP: $ip | Status: $status | Host: $host_name | Açyk portlar: $open_ports | Traceroute: $trace | MAC: $mac"
            # CSV hasabat ýazmak
            echo "$ip,$status,$host_name,\"$open_ports\",$trace,$mac" >> $report
            ;;
        0)
            echo -e "${BLUE}👋 Programdan çykdyň! Hasabat faýly: $report${NC}"
            exit 0
            ;;
        *)
            echo -e "${RED}❌ Nädogry saýlaw!${NC}"
            ;;
    esac
done
