# GotifyBashExamples

Zpool status
```
GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | cut -f 2 -d ":" | awk '{$1=$1}1')

MSG_TITLE="zpool status on $HOST with CPU $CPU"
MSG_BODY="$(zpool status)"

curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` "
```

rsync backup job
```
GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | cut -f 2 -d ":" | awk '{$1=$1}1')

MSG_TITLE="Sync job on $HOST with CPU $CPU"
SOURCE="root@your_server_ip"

echo $'Syncing /example/path' > stats.txt
rsync --info=progress0,name0,del0,flist0,stats2 --delete -avs --filter "- /exclude/path" $SOURCE:/example/path/ /example/path/ >> stats.txt

echo $'\nSyncing /example/path2' >> stats.txt
rsync --info=progress0,name0,del0,flist0,stats2 --delete -avs $SOURCE:/example/path2/ /example/path2/ >> stats.txt

MSG_BODY="$(cat stats.txt)"

curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` "
```
