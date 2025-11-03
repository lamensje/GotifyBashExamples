# GotifyBashExamples

GOTIFY_URL="http://<your_gotify_server_ip>:9080"
GOTIFY_TOKEN="your_token"

HOST=$(hostname)
CPU=$(lscpu | grep 'Model name' | cut -f 2 -d ":" | awk '{$1=$1}1')

MSG_TITLE="You command on $HOST with CPU $CPU"
MSG_BODY="$(your bash command)"

curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=\`\`\`$MSG_BODY\`\`\` "
