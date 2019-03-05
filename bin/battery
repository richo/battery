#!/bin/sh

HEART_FULL=♥
HEART_EMPTY=♡
[ -z "$NUM_HEARTS" ] &&
    NUM_HEARTS=5

cutinate()
{
    perc=$1
    inc=$(( 100 / $NUM_HEARTS))


    for i in `seq $NUM_HEARTS`; do
        if [ $perc -lt 100 ]; then
            echo $HEART_EMPTY
        else
            echo $HEART_FULL
        fi
        perc=$(( $perc + $inc ))
    done
}

linux_get_bat ()
{
    echo $(( $BAT_TOTAL / $BAT_COUNT ))
}


openbsd_get_bat ()
{
    bf=$(sysctl -n hw.sensors.acpibat0.amphour0 |  cut -d ' ' -f 1)
    bn=$(sysctl -n hw.sensors.acpibat0.amphour3 |  cut -d ' ' -f 1)
    echo "(($bn * 100) / $bf)" | bc -l | awk -F '.' '{ print $1 }';
}

freebsd_get_bat ()
{
    sysctl -n hw.acpi.battery.life
}

battery_status()
{
case $(uname -s) in
    "Linux")
        BATTERIES=$(ls /sys/class/power_supply | grep BAT)
        BAT_COUNT=$(ls /sys/class/power_supply | grep BAT | wc -l)
        for BATTERY in $BATTERIES; do
            BAT_PATH=/sys/class/power_supply/$BATTERY
            STATUS=$BAT_PATH/status
            [ "$1" = `cat $STATUS` ] || [ "$1" = "" ] || return 0
            if [ -f "$BAT_PATH/energy_full" ]; then
                naming="energy"
            elif [ -f "$BAT_PATH/charge_full" ]; then
                naming="charge"
            elif [ -f "$BAT_PATH/capacity" ]; then
                cat "$BAT_PATH/capacity"
                return 0
            fi
            BAT_PERCENT=$(( 100 * $(cat $BAT_PATH/${naming}_now) / $(cat $BAT_PATH/${naming}_full) ))
            BAT_TOTAL=$(( ${BAT_TOTAL-0} + $BAT_PERCENT ))
        done
        linux_get_bat
        ;;
    "FreeBSD")
        STATUS=`sysctl -n hw.acpi.battery.state`
        case $1 in
            "Discharging")
                if [ $STATUS -eq 1 ]; then
                    freebsd_get_bat
                fi
                ;;
            "Charging")
                if [ $STATUS -eq 2 ]; then
                    freebsd_get_bat
                fi
                ;;
            "")
                freebsd_get_bat
                ;;
        esac
        ;;
    "OpenBSD")
        openbsd_get_bat
        ;;
    "Darwin")
        case $1 in
            "Discharging")
                ext="No";;
            "Charging")
                ext="Yes";;
        esac

        ioreg -c AppleSmartBattery -w0 | \
        grep -o '"[^"]*" = [^ ]*' | \
        sed -e 's/= //g' -e 's/"//g' | \
        sort | \
        while read key value; do
            case $key in
                "MaxCapacity")
                    export maxcap=$value;;
                "CurrentCapacity")
                    export curcap=$value;;
                "ExternalConnected")
                    if [ -n "$ext" ] && [ "$ext" != "$value" ]; then
                        exit
                    fi
                ;;
                "FullyCharged")
                    if [ "$value" = "Yes" ]; then
                        exit
                    fi
                ;;
            esac
            if [[ -n "$maxcap" && -n $curcap ]]; then
                echo $(( 100 * $curcap / $maxcap ))
                break
            fi
        done
esac
}

BATTERY_STATUS=`battery_status $1`
[ -z "$BATTERY_STATUS" ] && exit

if [ -n "$CUTE_BATTERY_INDICATOR" ]; then
    cutinate $BATTERY_STATUS
else
    echo ${BATTERY_STATUS}%
fi
