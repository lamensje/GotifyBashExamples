# Gotify Bash Examples

zpool status
```
GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | grep -v 'BIOS' | cut -f 2 -d ":" | awk '{$1=$1}1')

MSG_TITLE="zpool status on $HOST with CPU $CPU"
MSG_BODY="$(zpool status)"

curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` "
```

rsync backup job
```
GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | grep -v 'BIOS' | cut -f 2 -d ":" | awk '{$1=$1}1')

MSG_TITLE="Sync job on $HOST with CPU $CPU"
SOURCE="root@your_server_ip"

echo $'Syncing /example/path' > stats.txt
rsync --info=progress0,name0,del0,flist0,stats2 --delete -avs --filter "- /exclude/path" $SOURCE:/example/path/ /example/path/ >> stats.txt

echo $'\nSyncing /example/path2' >> stats.txt
rsync --info=progress0,name0,del0,flist0,stats2 --delete -avs $SOURCE:/example/path2/ /example/path2/ >> stats.txt

MSG_BODY="$(cat stats.txt)"

curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` "
```

disk status
```
GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | cut -f 2 -d ":" | awk '{$1=$1}1')
NL=$'\n'

for disk in /dev/sd?
do
 MODEL_SERIAL=$(/usr/sbin/smartctl -i $disk | grep 'Model\|Serial' | grep -v "Family")
 OVERALL_HEALTH=$(/usr/sbin/smartctl -a $disk | grep "overall-health")
 SMART=$(/usr/sbin/smartctl -A -f brief $disk | awk '/Attributes/{found=1; next} found' | awk '/Error Log/{exit} {print}' |  grep -v "|" | sed -Ee 's/\S+/\n&/2;s/.*\n//' | sed -r 's/\S+//2')

 if [[ $SMART ]]
  then
   MSG_TITLE="smartctl on $HOST with CPU $CPU"
   MSG_BODY="$MODEL_SERIAL$NL$NL$OVERALL_HEALTH$NL$NL$SMART"
   curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` " &> /dev/null
  fi
done

for nvme in /dev/nvme0n?
do
 MODEL_SERIAL=$(/usr/sbin/smartctl -i $nvme | grep 'Model\|Serial' | grep -v "Family")
 SMART=$(/usr/sbin/smartctl -a $nvme | awk '/SMART DATA SECTION/{found=1; next} found' | awk '/Error Information/{exit} {print}')

 if [[ $SMART ]]
  then
   MSG_TITLE="smartctl on $HOST with CPU $CPU"
   MSG_BODY="$MODEL_SERIAL$NL$NL$SMART"
   curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` " &> /dev/null
  fi
done
```

cronjob
```
#Runs every 2nd monday of the month at 9:00 (after first sunday of the month, default ZFS scrub on Proxmox)
0 9 * * MON#2 /bin/bash /root/smartctl.sh 2>&1 /dev/null
```
