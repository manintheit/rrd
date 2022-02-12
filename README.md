```bash
#!/bin/bash

sensor_data=$(sensors -j)
declare -A core_temps
# TODO: Enhance JQ to dynamically map CPU Core with its temperature.[Core0:30C, Core1:50]
core_temps['core0']=$(echo ${sensor_data} | jq -r '.[]."Core 0"."temp2_input"|select(.!=null)')
core_temps['core1']=$(echo ${sensor_data} | jq -r '.[]."Core 1"."temp3_input"|select(.!=null)')
core_temps['core2']=$(echo ${sensor_data} | jq -r '.[]."Core 2"."temp4_input"|select(.!=null)')
core_temps['core3']=$(echo ${sensor_data} | jq -r '.[]."Core 3"."temp5_input"|select(.!=null)')
core_temps['core4']=$(echo ${sensor_data} | jq -r '.[]."Core 4"."temp6_input"|select(.!=null)')
core_temps['core5']=$(echo ${sensor_data} | jq -r '.[]."Core 5"."temp7_input"|select(.!=null)')

rrdtool update cputemp.rrd -t Core0:Core1:Core2:Core3:Core4:Core5 N:${core_temps[core0]}:${core_temps[core1]}:${core_temps[core2]}:${core_temps[core3]}:${core_temps[core4]}:${core_temps[core5]}
```

## Create Database
```bash
rrdtool create cputemp.rrd      \
    --step 60 --start N         \
    DS:Core0:GAUGE:120:U:100    \
    DS:Core1:GAUGE:120:U:100    \
    DS:Core2:GAUGE:120:U:100    \
    DS:Core3:GAUGE:120:U:100    \
    DS:Core4:GAUGE:120:U:100    \
    DS:Core5:GAUGE:120:U:100    \
    RRA:AVERAGE:0.5:1:1200      \
    RRA:MIN:0.5:12:2400         \
    RRA:MAX:0.5:12:2400         \
    RRA:AVERAGE:0.5:12:2400
```

## Set Crontab

```bash
*/1 * * * * </path>/logcputemp.sh
```

## Check Data
```bash
rrdtool fetch cputemp.rrd AVERAGE -r 60 -s -10m
```

## Run Stress Test 
```bash
stress --cpu 6 --io 4 --vm 2 --vm-bytes 128M --timeout 120m
```


## Create Temp Graph

```bash
rrdtool graph cputemp.png \
--imgformat=PNG \
--start=-120m \
--title='CPU Temperature' \
--rigid \
--base=1000 \
--height=120 \
--upper-limit=100 \
--width=700 \
--alt-autoscale-max \
--lower-limit=0 \
--vertical-label "Temperature (°C)" \
--slope-mode \
--font TITLE:8: \
--font AXIS:8: \
--font LEGEND:10: \
--font UNIT:8: \
DEF:a="cputemp.rrd":Core0:AVERAGE \
DEF:b="cputemp.rrd":Core1:AVERAGE \
DEF:c="cputemp.rrd":Core2:AVERAGE \
DEF:d="cputemp.rrd":Core3:AVERAGE \
DEF:e="cputemp.rrd":Core4:AVERAGE \
DEF:f="cputemp.rrd":Core5:AVERAGE \
LINE1:a#DEAFAF:"Core0"  \
GPRINT:a:LAST:" Current\:%8.2lf %s"  \
GPRINT:a:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:a:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:a:MIN:"Minumum\:%8.2lf %s\n"  \
LINE2:b#0F7059:"Core1"  \
GPRINT:b:LAST:" Current\:%8.2lf %s"  \
GPRINT:b:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:b:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:b:MIN:"Minumum\:%8.2lf %s\n"  \
LINE3:c#03D3FC:"Core2"  \
GPRINT:c:LAST:" Current\:%8.2lf %s"  \
GPRINT:c:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:c:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:c:MIN:"Minumum\:%8.2lf %s\n"  \
LINE4:d#A903FC:"Core3"  \
GPRINT:d:LAST:" Current\:%8.2lf %s"  \
GPRINT:d:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:d:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:d:MIN:"Minumum\:%8.2lf %s\n"  \
LINE5:d#03FC03:"Core4"  \
GPRINT:e:LAST:" Current\:%8.2lf %s"  \
GPRINT:e:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:e:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:e:MIN:"Minumum\:%8.2lf %s\n"  \
LINE6:f#BF19A4:"Core5"  \
GPRINT:f:LAST:" Current\:%8.2lf %s"  \
GPRINT:f:AVERAGE:"Average\:%8.2lf %s"  \
GPRINT:f:MAX:"Maximum\:%8.2lf %s"  \
GPRINT:f:MIN:"Minumum\:%8.2lf %s\n"  \
```


![](img/cputemp.png)


## Network Monitoring

### :warning: Not Completed Yet

```bash
declare -A stats

tmp=$(ethtool -S wlo1 | grep -E '(tx|rx)_bytes' | tr -d ' ')
stats[rx_bytes]=$(echo "$tmp" | awk -F ":" '/rx_bytes/{print $2}')
stats[tx_bytes]=$(echo "$tmp" | awk -F ":" '/tx_bytes/{print $2}')
echo ${stats[rx_bytes]}
echo ${stats[tx_bytes]}
```