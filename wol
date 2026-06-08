#!/usr/bin/env bash
# Шалость WoL

FILE="wol_targets.txt"

scan_network() {
    echo "Сканирование сети и сбор MAC-адресов..."
    
    # Определяем локальную подсеть (fallback на 192.168.1.0/24)
    SUBNET=$(ip route 2>/dev/null | awk '/src/ {print $1}' | head -n 1)
    [ -z "$SUBNET" ] && SUBNET="192.168.1.0/24"

    # Тихое сканирование для обновления ARP-таблицы
    nmap -sn "$SUBNET" >/dev/null 2>&1

    # Извлекаем MAC-адреса, исключая широковещательный
    arp -a | grep -oE '([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}' | grep -v 'ff:ff:ff:ff:ff:ff' | sort -u > "$FILE"

    COUNT=$(wc -l < "$FILE")
    echo "Найдено устройств: $COUNT. Список сохранен в $FILE"
}

wake_devices() {
    if [ ! -f "$FILE" ]; then
        echo "Ошибка: файл $FILE не найден. Сначала выполните: $0 scan"
        exit 1
    fi

    echo "Массовое пробуждение устройств..."
    while IFS= read -r mac; do
        [[ -z "$mac" || "$mac" == \#* ]] && continue
        wakeonlan "$mac" >/dev/null 2>&1
        sleep 0.1
    done < "$FILE"
    
    echo "Пакеты отправлены."
}

case "$1" in
    scan) scan_network ;;
    wake) wake_devices ;;
    *)
        echo "Шалость WoL"
        echo "Использование: $0 {scan|wake}"
        echo "  scan  - собрать MAC-адреса устройств в сети"
        echo "  wake  - разбудить все устройства из списка"
        ;;
esac
