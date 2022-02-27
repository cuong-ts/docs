# Alert slack by hook

Send slack alert

```bash
#!/bin/bash

curl https://hooks.slack.com/services/xxx \
    -X POST -H "Content-type: application/json" \
    -d @- << EOF
{
    "channel": "$1",
    "username": "$2",
    "color": "danger",
    "icon_emoji": ":ghost:",
    "pretext": "$MONIT_DATE",
    "text": "$MONIT_SERVICE - $MONIT_DESCRIPTION"
}
EOF
```

Send slack alert with RAM usage

```bash
#!/bin/bash

LOG_FILE_PATH="/var/log/memory_$(date '+%Y-%m-%d-%H').log"

date | sudo tee -a $LOG_FILE_PATH
ps -eo rss,pid,user,command | sort -rn | head -20 | awk '{ hr[1024**2]="GB"; hr[1024]="MB"; for (x=1024**3; x>=1024; x/=1024) { if ($1>=x) { printf ("%-6.2f %s ", $1/x, hr[x]); break } } } { printf ("%-6s %-10s ", $2, $3) } { for ( x=4 ; x<=NF ; x++ ) { printf ("%s ",$x) } print ("\n") }' | sudo tee -a $LOG_FILE_PATH
memory_log=$(ps -eo rss,pid,user,command | sort -rn | head -10 | awk '{ hr[1024**2]="GB"; hr[1024]="MB"; for (x=1024**3; x>=1024; x/=1024) { if ($1>=x) { printf ("%-6.2f %s ", $1/x, hr[x]); break } } } { printf ("%-6s %-10s ", $2, $3) } { for ( x=4 ; x<=NF ; x++ ) { printf ("%s ",$x) } print ("\n") }')
curl https://hooks.slack.com/services/xxx \
    -X POST -H "Content-type: application/json" \
    -d @- << EOF
{
    "channel": "$1",
    "username": "$2",
    "color": "warning",
    "icon_emoji": ":ghost:",
    "pretext": "$MONIT_SERVICE - $MONIT_DESCRIPTION",
    "text": "\`\`\`$memory_log\`\`\`",
    "mrkdwn": true
}   
EOF
```

