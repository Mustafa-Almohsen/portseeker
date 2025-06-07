#!/bin/bash

# Color variables
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No color

# Defaults
TOP_PORTS=false
THREADS=1
OUTPUT_FILE=""
TARGET_IP=""

function print_help {
    echo -e "\nUsage: $0 <target_ip> [options]"
    echo -e "\nOptions:"
    echo -e "  --top                Scan only top 1000 ports"
    echo -e "  --threads <num>      Run with <num> concurrent threads (default: 1)"
    echo -e "  --output <file>      Save output to a file"
    echo -e "  -h, --help           Show this help message\n"
    exit 0
}


TOP_1000_PORTS=(
     7 9 13 17 19 20 21 22 23 25 26 37 53 67 68 69 70 79 80 88 110 111 113 119 123 135 137 138 139 143
    161 162 179 199 389 427 443 445 465 512 513 514 515 520 548 554 587 593 623 631 636 873 902 989
    990 993 995 1025 1026 1027 1028 1029 1030 1080 1194 1214 1241 1352 1433 1434 1521 1701 1720 1723
    1755 1812 1813 1863 2049 2100 2144 2181 2222 2301 2483 2484 2967 3000 3128 3306 3389 3478 3689
    3690 3784 3868 3872 4011 4045 4190 4444 4445 4500 4555 4662 4899 5000 5001 5003 5009 5050 5060
    5061 5100 5120 5190 5222 5223 5432 5500 5555 5631 5666 5800 5900 5938 6000 6001 6002 6003 6004
    6005 6006 6007 6112 6660 6661 6662 6663 6664 6665 6666 6667 6668 6669 6679 6697 7000 7001 7002
    7070 7100 7200 7443 7777 8000 8001 8008 8010 8080 8081 8082 8083 8088 8090 8118 8181 8222 8333
    8443 8500 8600 8649 8880 8888 8899 9000 9001 9002 9003 9040 9050 9080 9090 9100 9119 9191 9200
    9415 9418 9443 9535 9800 9876 9898 9981 9987 9999 10000 10001 10010 10050 10051 10101 10110 10243
  
)

for p in $(seq 1 1024); do TOP_1000_PORTS+=($p); done

# Parse args
while [[ $# -gt 0 ]]; do
    case "$1" in
        --top)
            TOP_PORTS=true
            shift
            ;;
        --threads)
            THREADS="$2"
            shift 2
            ;;
        --output)
            OUTPUT_FILE="$2"
            shift 2
            ;;
        -h|--help)
            print_help
            ;;
        *)
            if [[ -z "$TARGET_IP" ]]; then
                TARGET_IP="$1"
                shift
            else
                echo -e "${RED}[-] Unknown argument: $1${NC}"
                exit 1
            fi
            ;;
    esac
done

if [[ -z "$TARGET_IP" ]]; then
    echo -e "${RED}[-] Error: Target IP is required.${NC}"
    print_help
fi


if [[ "$TOP_PORTS" = true ]]; then
    PORTS=("${TOP_1000_PORTS[@]}")
else
    PORTS=$(seq 1 65535)
fi

echo -e "[*] Starting scan on $TARGET_IP with ${THREADS} thread(s)..."
[[ "$TOP_PORTS" = true ]] && echo "[*] Mode: Top 1000 ports" || echo "[*] Mode: Full port scan"
[[ ! -z "$OUTPUT_FILE" ]] && echo "[*] Output will be saved to $OUTPUT_FILE"


write_output() {
    if [[ ! -z "$OUTPUT_FILE" ]]; then
        echo -e "$1" >> "$OUTPUT_FILE"
    fi
    echo -e "$1"
}


scan_port() {
    local PORT=$1
    local URL_HTTP="http://$TARGET_IP:$PORT"
    local URL_HTTPS="https://$TARGET_IP:$PORT"

    local HTTP_CODE=$(curl -sk -o /dev/null -w "%{http_code}" --max-time 2 "$URL_HTTP")

    if [[ "$HTTP_CODE" == "200" ]]; then
        write_output "${GREEN}[HTTP] $URL_HTTP --> 200 OK${NC}"
    elif [[ "$HTTP_CODE" =~ 30[1-8] ]]; then
        REDIRECT=$(curl -skI --max-time 2 "$URL_HTTP" | grep -i '^Location')
        write_output "${GREEN}[HTTP] $URL_HTTP --> Redirect ($HTTP_CODE) --> $REDIRECT${NC}"
    elif [[ "$HTTP_CODE" != "000" ]]; then
        write_output "${YELLOW}[HTTP] $URL_HTTP --> HTTP $HTTP_CODE${NC}"
    else
        local HTTPS_CODE=$(curl -sk -o /dev/null -w "%{http_code}" --max-time 2 "$URL_HTTPS")
        if [[ "$HTTPS_CODE" == "200" ]]; then
            write_output "${GREEN}[HTTPS] $URL_HTTPS --> 200 OK${NC}"
        elif [[ "$HTTPS_CODE" =~ 30[1-8] ]]; then
            REDIRECT=$(curl -skI --max-time 2 "$URL_HTTPS" | grep -i '^Location')
            write_output "${GREEN}[HTTPS] $URL_HTTPS --> Redirect ($HTTPS_CODE) --> $REDIRECT${NC}"
        elif [[ "$HTTPS_CODE" != "000" ]]; then
            write_output "${YELLOW}[HTTPS] $URL_HTTPS --> HTTP $HTTPS_CODE${NC}"
        else
            write_output "${RED}[-] $TARGET_IP:$PORT --> No HTTP/HTTPS response${NC}"
        fi
    fi
}


export -f scan_port write_output
export TARGET_IP OUTPUT_FILE RED GREEN YELLOW NC

if [[ "$TOP_PORTS" = true ]]; then
    printf "%s\n" "${PORTS[@]}" | xargs -P "$THREADS" -I {} bash -c 'scan_port "$@"' _ {}
else
    seq 1 65535 | xargs -P "$THREADS" -I {} bash -c 'scan_port "$@"' _ {}
fi
