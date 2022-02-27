# Check SSL certificate expiration date\(s\)

prepare checkssl.sh file with content bellow:

{% code title="checkssl.sh" %}
```bash
#!/bin/bash
#  By B Shea  Dec2018 & Mar2020
#  https://www.holylinux.net

# Test for OpenSSL - if not installed stop here.
if ! [[ -x $(which openssl) ]]; then
  printf "\nOpenSSL not found or not executable.\nPlease install OpenSSL before proceeding.\n\n"
  exit 1
fi

### user adjustable variables ###
#openssl query timeout:
openssl_timeout="timeout 10"
# 30 days is default on warnings - overidden on command line with '-d':
days_to_warn=30
# default name for file lists
sitelist=./websites.txt

### Clear/list/set defaults for variables ###
epoch_day=86400
epoch_warning=$((days_to_warn*epoch_day))
regex_numbers='^[0-9]+$'
expire="0"
website=""
port=""
tls="0"
sTLS=""
show_tls=""
certfilename=""
location=""
filename=""
displaysite=""
#COLORS
color="0"
RED=$(tput setaf 1)    #expired!!
GREEN=$(tput setaf 2)  #within bounds
YELLOW=$(tput setaf 3) #warning/date close!
NC=$(tput sgr0)        #reset to normal
#
usage="
$(basename "$0") [-h] [-c] [-d DAYS] [-f FILENAME] | [-w WEBSITE] | [-s SITELIST]

Retrieve the expiration date(s) on SSL certificate(s) using OpenSSL.

Usage:
    -h  Help

    -c  Color output

    -d  Amount of days to show warnings (default is 30 days)
        Example: -d 15

    -f  SSL date from FILENAME
        Example: -f /home/user/example.pem

    -w  SSL date from SITE(:PORT) (Port defaults to 443)
        Example: -w www.example.com

    -s  SSL date(s) from SITELIST
        Example:      -s ./websites.txt
        List format:  sub.domain.tld:993 (one per line - port optional)

Example:
    $ $(basename "$0") -c -d 14 -s ./websites.txt

    WARNS (in color) if within 14 days of expiring on each entry in the file list.

"

#FUNCTIONS

is_integer() {
    if ! [[ "$1" =~ $regex_numbers ]]; then
      printf "\nError.\nNot a number. You used a parameter that requires a whole number.\n$usage"
      exit 1
    fi
}
menu_input() {
  echo
  echo "1: Enter file location of certificate"
  echo "2: Enter an Internet site in form of subdomain.domain.tld(:port)"
  echo
  read -p "Enter 1 or 2 (anything else quits): " -n 1 -r
  echo
}
get_lookup_input() {
    location=""
    echo
    read -p "Please enter the $lookuptype location: " location
}
set_format() {
    set_formatting="%-40s%-25s\n"
    set_formatting_green=$set_formatting
    set_formatting_yellow=$set_formatting
    set_formatting_red=$set_formatting
    printf "\nWarning is $days_to_warn days.\n"
    printf "Color is "
    if [[ $color == "1" ]]; then
      set_formatting_green="$GREEN%-40s$NC%-25s\n"
      set_formatting_yellow="$YELLOW%-40s$NC%-25s\n"
      set_formatting_red="$RED%-40s$NC%-25s\n"
      printf "enabled.\n\n"
    else
      printf "disabled.\n\n"
    fi
    printf "$set_formatting" "LOCATION" "EXPIRATION DATE"
    printf "$set_formatting" "--------" "---------------"
}
parse_port() {
    port=443
    tls="0"
    show_tls=""
    parseurl=$(echo $website | awk '$1 ~ /^.*:/' | cut -d':' -f1)
    parseport=$(echo $website | awk '$1 ~ /^.*:/' | cut -d':' -f2)
    if [[ $parseport =~ $regex_numbers ]]; then # -> port was found
      website=$parseurl
      port=$parseport
      if [[ $port == "587" ]]; then # Use TLS lookup and notify
        show_tls=" (TLS)"
        tls="1"
      fi
    fi
}
check_expiry() {
    expire="0"
    # use epoch times for calcs/compares
    today_epoch="$(gdate +%s)"
    sTLS=""

    if [[ $tls == "1" ]]; then
      sTLS=" -starttls smtp"
    fi

    if [ "$lookuptype" == "FILENAME" ]; then
      expire_date=$(openssl x509 -in $certfilename$sTLS -noout -dates 2>/dev/null | \
                  awk -F= '/^notAfter/ { print $2; exit }')
    else
      expire_date=$($openssl_timeout openssl s_client -servername $website -connect $website:$port$sTLS </dev/null 2>/dev/null | \
                  openssl x509 -noout -dates 2>/dev/null | \
                  awk -F= '/^notAfter/ { print $2; exit }')
    fi
    if ! [[ -z $expire_date ]]; then # -> found date-process it:
      expire_epoch=$(gdate +%s -d "$expire_date")
      timeleft=`expr $expire_epoch - $today_epoch`
      if [[ $timeleft -le $epoch_warning ]]; then #WARN
        expire="1"
      fi
      if [[ $today_epoch -ge $expire_epoch ]]; then #EXPIRE
        expire="2"
      fi
    else
      expire="3"
      expire_date="N/A                     "
    fi
}
output_site() {
    parse_port
    check_expiry
    if [ "$lookuptype" != "FILENAME" ]; then
      display_site="$website:$port$show_tls"
    else
      display_site="$filename$show_tls"
    fi
    if   [[ $expire == "1" ]]; then
      printf "$set_formatting_yellow"  "$display_site" "$expire_date !"  # YELLOW OUTPUT - warning
    elif [[ $expire == "2" ]]; then
      printf "$set_formatting_red"     "$display_site" "$expire_date !!" # RED OUTPUT - expired
    elif [[ $expire == "3" ]]; then
      printf "$set_formatting"         "$display_site" "$expire_date !!!" # NO COLOR - NOT FOUND
    else
      printf "$set_formatting_green"   "$display_site" "$expire_date"    # GREEN OUTPUT
    fi
}

#

client_lookup() {
    lookuptype="WEBSITE"
    if [[ -z $website ]]; then #loop lookup - ask for input
      get_lookup_input
      website=$location
    fi
    set_format
    output_site
    lookuptype=""
    website=""
    echo
}
file_lookup() {
    lookuptype="FILENAME"
    if [[ -z $certfilename ]]; then #loop lookup - ask for input
      get_lookup_input
      certfilename=$location
    fi
    filename=$(basename -- "$certfilename")
    set_format
    output_site
    lookuptype=""
    filename=""
    echo
}
list_lookup() {
    lookuptype="FILELIST"
    file_contents=$(<$sitelist)
    set_format
    while IFS= read -r website; do
      if ! [[ -z $website ]]; then
        output_site
      fi
    done <<<"$file_contents"
    lookuptype=""
    echo
}

#HANDLE ARGUMENTS

while getopts ':hcd:f:s:w:' option; do
  case "$option" in
    h) printf "$usage"
       exit 0
       ;;
    c) color="1"
       ;;
    d) is_integer "$OPTARG"
       if [ "$OPTARG" -ge 1 -a "$OPTARG" -le 365 ]; then
         days_to_warn="$OPTARG"
         epoch_warning=$((days_to_warn*epoch_day))
       else
         printf "\nDays must be between 1 and 365\n$usage"
         exit 1
       fi
       ;;
    f) certfilename=$OPTARG
       [[ -r $certfilename ]] && file_lookup || printf "\nFile not found/not readable. Permissions?\n\n"; exit 1;
       exit 0
       ;;
    s) sitelist=$OPTARG
       [[ -r $sitelist ]] && list_lookup || printf "\nFile not found/not readable. Permissions?\n\n"; exit 1;
       exit 0
       ;;
    w) website=$OPTARG
       client_lookup
       exit 0
       ;;
    :) printf "\nYou specified a flag that needs an argument.\n$usage" 1>&2
       exit 1
       ;;
    *) printf "\nI do not understand '"$1" "$2"'.\n$usage" 1>&2
       exit 1
       ;;
  esac
done
shift $((OPTIND - 1))

#LOOP RUN (default if no flags)

if [ $# -eq 0 ]; then # no command line arguments/flags found
printf "\nNo flags used or available. Interactive mode.\n"
  while :
  do
    menu_input
    if [[ $REPLY == "1" ]]
    then
      file_lookup
    elif [[ $REPLY == "2" ]]
    then
      client_lookup
    else # exit
      [[ "$0" = "$BASH_SOURCE" ]] && exit 1 || return 1
    fi
  echo
  done
fi%
```
{% endcode %}

prepare website list to check with example format:

{% code title="websites.txt" %}
```text
www.abc.com
zxc.com
```
{% endcode %}

check your ssl expired date:

```text
./checkssl.sh -c -d 45 -s websites.txt
```

{% hint style="info" %}
some advande usages:

```text
checkssl [-h] [-c] [-d DAYS] [-f FILENAME] | [-w WEBSITE] | [-s SITELIST]

Retrieve the expiration date(s) on SSL certificate(s) using OpenSSL.

Usage:
    -h  Help

    -c  Color output

    -d  Amount of days to show warnings (default is 30 days)
        Example: -d 15

    -f  SSL date from FILENAME
        Example: -f /home/user/example.pem

    -w  SSL date from SITE(:PORT) (Port defaults to 443)
        Example: -w www.example.com

    -s  SSL date(s) from SITELIST
        Example:      -s ./websites.txt
        List format:  sub.domain.tld:993 (one per line - port optional)

Example:
    $ checkssl -c -d 14 -s ./websites.txt

    WARNS (in color) if within 14 days of expiring on each entry in the file list.
```
{% endhint %}

**Eg output:**

![](../.gitbook/assets/image.png)

