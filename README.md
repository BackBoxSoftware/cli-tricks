# CLI tricks

Often in our daily work we encounter the need to run stuff in CLI - and too often this proves to be trickier than one would expect. In the spirit of saving time for others, we've decided to compile a list of the ones we've found to be useful and not-so-obvious.

- [grep](#grep-tricks)
- [awk](#awk-tricks)
- [sed](#sed-tricks)
- [VI](#vi-tricks)
- [bash](#bash-tricks)
- [regex](#regex-tricks)
- [BackBox](#backbox-specific)
- [Check Point](#check-point-tricks)
- [Misc](#miscellaneous)
  - [Emoji](#emoji)

## grep Tricks

### Check Point license expiration

```bash
grep -o -P  '..?Jan.*?_|..?Feb.*?_|..?Mar.*?_|..?Apr.*(?=.zip)|..?May.*?_|..?Jun.*?_|..?Jul.*?_|..?Aug.*?_|..?Sep.*?_|..?Oct.*?_|..?Nov.*?_|..?Dec.*?_'

cat cplic | grep -o -P  '..?Jan.*?....|..?Feb.*?....|..?Mar.*?....|..?Apr.*?....|..?May.*?....|..?Jun.*?....|..?Jul.*?....|..?Aug.*?....|..?Sep.*?....|..?Oct.*?....|..?Nov.*?....|..?Dec.*?....'
```

### Get between double quotes

```bash
grep -o '".*"' | tr -d '"'
```

### Get between single quotes

```bash
grep -oP "(?<=').*?(?=')"
```

### Get between ()

```bash
grep -oP '(?<=\()[^\)]+'
grep -oP '\(\K[^\)]+'
```

### Get everything after match

```bash
grep -o "<MATCH>.*"
```

### Get IPv4 address

```bash
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}"
```

### Get lines with exactly the word

```bash
grep -oh "\w*<MATCH>\w*"
```

### Get lines with numbers only

```bash
grep --only-matching '[[:digit:]]\+'
```

### Get lines with only one /

```bash
grep -e '^[^/]*/[^/]*$'
```

### Get match only

```bash
grep -E -o '<MATCH>\w+'
```

### Print word before match

```bash
grep -E "MATCH" | cut -d "," -f2 | awk {print $1}'
```

### ~~add description~~

```bash
grep -oh "\w*#\w*"
```

### ~~add description~~

```bash
grep -o -P '(?<=FROM HERE).*(?=TO HERE)'
```

## awk Tricks

### Get between square brackets

```bash
awk 'NR>1{print $1}' RS=[ FS=]
```

### Get between ()

```bash
awk -F'[()]' '{print $2}'
```

### Get lines with numbers only

```bash
awk '{gsub("[^[:digit:]]+"," ")}1'
```

### Print after

```bash
awk '/regex/ { print (x=="" ? "match on line 1" : x) }; { x=$0 }'
```

### Print column after match

```bash
awk -v srch="<PATTERN>" 'BEGIN{l=length(srch)}{t=match($0,srch);if(!t){next}$0=substr($0,t+l);print srch" "$2}' <filename> | awk '{print $1}'
```

### Print from column to end

```bash
awk '{ print substr($0, index($0,$3)) }'
```

### Print lines that match any of "AAA" or "BBB", or "CCC"

```bash
awk '/AAA|BBB|CCC/'
```

### Print the line immediately before a line that matches "regex" (but not the line that matches itself)

```bash
awk '/regex/ { print x }; { x=$0 }'
```

### Replace column value

```bash
awk '{$<COL_NUMBER> = "<VALUE>"; print}'
```

### Turn list to table

```bash
 | awk 'BEGIN { print "<table>" }
     { print "<tr><td>" $1 "</td><td>" $2 "</td><tr>" }
     END   { print "</table>" }'
	 
	 cat toTable | awk 'BEGIN { print "<tbody>" }
     { print "<tr><td><strong>" $1 "</strong></td>" }
	 { print "<td>" $2 "</td></tr>" }
     END   { print "</tbody>" }'
```

### ~~add description~~

```bash
awk -v FS="(FROM HERE|TO HERE)" '{print $2}'
```

## sed Tricks

### SED BREAKDOWN

```
s/          <-- this means it should perform a substitution
.*          <-- this means match zero or more characters
\[          <-- this means match a literal [ character
\(          <-- this starts saving the pattern for later use
[^]]*       <-- this means match any character that is not a [ character
                the outer [ and ] signify that this is a character class
                having the ^ character as the first character in the class means "not"
\)          <-- this closes the saving of the pattern match for later use
\]          <-- this means match a literal ] character
/\1         <-- this means replace everything matched with the first saved pattern
               (the match between "\(" and "\)" )
/g          <-- this means the substitution is global (all occurrences on the line)
\< EXACT MATCH \>
```

### Add line after

```bash
sed '/pam_unix.so/a\
auth        required      pam_faillock.so preauth silent deny=6 unlock_time=1800 fail_interval=900' file
```

### Add line before

```bash
sed '/pam_unix.so/i\
<INSERT>' file
```

### Add string to beginning of all lines

```bash
sed 's/^/<STRING>/'
```

### Add string to beginning of all matching lines

```bash
sed '/<MATCH>/s/^/<STRING>/'
```

### Add string to end of all lines

```bash
sed '/s/$/<string>/'
```

### Add string to end of all matching lines

```bash
sed '/<MATCH>/s/$/ myalias/'
```

### Add to end of matching line

```bash
sed '/match/ s/$/ anotherthing/' file
```

### Append a line to the next if it ends with a backslash \

```bash
sed -e :a -e '/\\$/N; s/\\\n//; ta'
```

### Delete both leading and trailing whitespace from each line

```bash
sed 's/^[ \t]*//;s/[ \t]*$//'
```

### Delete trailing whitespace (tabs and spaces) from each line

```bash
sed 's/[ \t]*$//'
```

### Delete line with match X if next one has match Y

```bash
sed '/X/{$!N;/\n.*Y/!P;D}'
```

### Delete from line with match until line with match

```bash
sed 's/<MATCH>/,/<MATCH>//g'
```

### Delete from line with match until line with match in loop

```bash
sed '/<MATCH>/,/<MATCH>/!d;/;/p'
```

### Double-space a file

```bash
sed G
```

### Double-space a file which already has blank lines in it - do it so that the output contains no more than one blank line between two lines of text

```bash
sed '/^$/d;G'
```

### Find line with <MATCH> and replace <STR> with <REPLACE>

```bash
sed '/<MATCH>/s/<STR>/<REPLACE>/g'
```

### Format minutes and seconds

```bash
echo '345,0m0.047s' | sed -n -r 's/^(.*),.*[^0-9]([0-9]*)\.(.*)s$/\1,\2\3/p'
345,0047
```

### Get name from escape characters

```bash
sed 's/[0-9][0-9];[0-9][0-9]H//g' | egrep -o '[^][]+'
```

### Insert a blank line above and below every line that matches "regex"

```bash
sed '/regex/{x;p;x;G;}'
```

### Insert a blank line above every line that matches "regex"

```bash
sed '/regex/{x;p;x;}'
```

### Insert a blank line below every line that matches "regex"

```bash
sed '/regex/G'
```

### Print after

```bash
sed -n '/ABC/,+1p' infile
```

### Remove between two strings

```bash
sed 's/<STRING1>.*<STRING2>//'
```

### Remove characters

```bash
sed -e 's|[<THIS>\<THIS>]||g'
```

### Remove lines with / only

```bash
sed 's|/|:|g'
```

### Remove special characters: !@#\$%^&*<>"()

```bash
sed 's/[!@#\$%^&*<>"()]//g'
```

### Replace matching line

```bash
sed -i "/aaa=/c\aaa=xxx" your_file_here
```

### Triple-space a file

```bash
sed 'G;G'
```

### Undo double-spacing

```bash
sed 'n;d'
```

## VI Tricks

### Find and replace - case insensitive

```bash
:%s/unix/Linux/gi
```

### Find and replace - case sensitive

```bash
:%s/UNIX/bar/gI
```

### Find and replace - whole word only

```bash
:%s/\<UNIX\>/Linux/gc
```

### Find and replace - with confirmation

```bash
:%s/UNIX/Linux/gc
```

## Bash Tricks

### Aliases

```bash
alias ..='cd ..'
alias sql='mysql -uroot -pbackbox6 backboxV3'
alias c='clear'
alias cls='clear;ls'
# Grabs the disk usage in the current directory
alias usage='du -ch | grep total'
alias ksh='du -ksh *'
# Gives you what is using the most space. Both directories and files. Varies on
# current directory
alias most='du -hsx * | sort -rh | head -10'
# ls aliases
alias lf='ls -alF --color=auto'
alias la='ls -al --color=auto'
alias ll='ls -l  --color=auto'
alias l='ls -l  --color=auto'
alias lh='ls -lh  --color=auto'
# create directory
alias md='mkdir -p'
alias t='tail -f '
alias bbx='service backbox restart'
alias network='service network restart'
alias f='find / -name'
alias fhere='find . -name'
alias iptables='service iptables restart'
```

### Clear file descriptors of deleted files

```bash
lsof | grep "(deleted)$" | sed -re 's/^\S+\s+(\S+)\s+\S+\s+([0-9]+).*/\1\/fd\/\2/' | while read file; do bash -c ": > /proc/$file"; done
```

### Convert xml to normal

```bash
echo "<XML>" or cat file | xml_pp
```

### Delete everything after match

```bash
cut -d "<MATCH>" -f1
```

### Encode password to URL

```bash
perl -MURI::Escape -lne 'print uri_escape($_)'
alias hashpass='echo $PASS | awk -F : '"'"'{for (i=1;i<=NF;i++) {print $i}}'"'"
```

### Get between ()

```bash
perl -lne '/\(\K[^\)]+/ and print $&'
```

### Print difference of two files

```bash
comm -13 <(sort file1) <(sort file2)
```

### Replace string with new line

```bash
perl -pe 's/(?<!^)(?=<STRING>)/\n/g' <filename>
```

### ~~add description~~

```bash
diff -a --suppress-common-lines -y ACL ACL_baseline
```

### ~~add description~~

```bash
rpm -qa --qf "%{NAME}\n"
```

## Regex Tricks

### ~~add description~~

```bash
^\w{0,10}$ # allows words of up to 10 characters.
^\w{5,}$   # allows words of more than 4 characters.
^\w{5,10}$ # allows words of between 5 and 10 characters.
```

## BackBox Specific

### ~~add description~~

```bash
/backbox/backbox-3.0/bin/sendEmail -f alerts@backbox.co -t SENDER@backbox.co -s 172.31.255.1 -u MailTest -o message-file=

sed -i 's\SSL_RSA_WITH_RC4_128_MD5, SSL_RSA_WITH_RC4_128_SHA ECDHE-RSA-AES256-GCM-SHA384\TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA\g' /backbox/backbox-3.0/app-server/apache-tomcat-7.0.37/conf/server.xml

service backbox restart
```

```sql
SELECT COMMAND FROM SESSIONS JOIN SESSIONCOMMANDS WHERE SESSION_ID=SESSIONS.ID AND SESSIONS.OPTION_ID = 101;

SELECT FROM PRODUCTS SET PRODUCT_TYPE='simple' WHERE PRODUCT_NAME='name';

UPDATE DEVICE_MONITOR_TEST SET ENABLE=0;

SELECT * FROM SESSIONS WHERE NAME='<name>';

UPDATE DEVICE_MONITOR_TEST SET ENABLE=1 WHERE SESSION_ID=<ID>;

4.5 - SELECT SOLUTION_ID, DEVICE_BACKUP_HISTORY.ID, DEVICE_MONITOR_ID, SESSION_ID FROM DEVICE_BACKUP_HISTORY JOIN DEVICE_MONITOR_TEST JOIN SESSIONS WHERE DEVICE_BACKUP_HISTORY.DEVICE_MONITOR_ID = DEVICE_MONITOR_TEST.ID AND DEVICE_MONITOR_TEST.SESSION_ID = SESSIONS.ID AND DEVICE_BACKUP_HISTORY.ID=%%HID%%;
5 - SELECT SOLUTION_ID, DEVICE_BACKUP_HISTORY.ID, DEVICE_MONITOR_ID, SESSION_ID FROM DEVICE_BACKUP_HISTORY JOIN DEVICE_MONITOR JOIN SESSIONS WHERE DEVICE_BACKUP_HISTORY.DEVICE_MONITOR_ID = DEVICE_MONITOR.ID AND DEVICE_MONITOR.SESSION_ID = SESSIONS.ID AND DEVICE_BACKUP_HISTORY.ID=%%HID%%;
```

## Check Point Tricks

### Anti-spoofing check on all firewall interfaces

```bash
fw="xxx"; cpmiquerybin object "" network_objects "name='$fw'" |grep anti_spoof
```

### Get cluster names and IP

```bash
cpmiquerybin attr "" network_objects "type='gateway_cluster'" -a __name__,ipaddr
```

### Get a list of names of all objects of type cluster member

```bash
cpmiquerybin attr "" network_objects "type='cluster_member'" -a __name__
```

### Get a list of names of all valid cluster members from cluster object name

```bash
cpmiquerybin object "" network_objects "" |grep -A 12 cluster_members |grep Name | awk -F "(" '{printf $2}' | sed -e 's/)/|/g'
cpmiquerybin attr "" network_objects "name='cluster_name'" -a cluster_members
```

### Get all members of a group

```bash
cpmiquerybin object "" network_objects "name='group_name_goes_here'" | grep ":Name"
```

### Get CMA list of policy collections

```bash
cpmiquerybin attr "" policies_collections "" -a __name__
```

### Get CMA policy names

```bash
cpmiquerybin attr "" fw_policies "" -a __name__
```

### Get installable targets for a policy named standard

```bash
cpmiquerybin attr "" policies_collections "name='Standar'" -a __name__,installable_targets
```

### Get IP for CLM name

```bash
cpmiquerybin attr "mdsdb" network_objects "name='Cluster1'" -a __name__,ipaddr
```

### Get secondary CMA

```bash
cpmiquerybin attr "" network_objects "(primary_management='false') & (management='true')" -a __name__
```

### List all MDSs

```bash
cpmiquerybin attr "mdsdb" mdss "" -a __name__
```

### List CMAs

```bash
cpmiquerybin attr "mdsdb" network_objects "management='true'" -a __name__,ipaddr
```

### List management/cma objects from cma env

```bash
cpmiquerybin attr "" network_objects "management='true'" -a __name__,ipaddr 
```

### List primary MDS

```bash
cpmiquerybin attr "mdsdb" mdss "primary='true'" -a __name__
```

### List services with 'Match for Any' ticked

```bash
cpmiquerybin attr "" services "include_in_any='true'" -a __name__
```

### Query all objects for an ip address

```bash
cpmiquerybin attr "" network_objects "ipaddr='<IP>'" -a __name__,ipaddr
```

### Standalone Firewalls

```bash
GATEWAYS=( `cpmiquerybin attr "" network_objects "(type='gateway') & (location='internal')" -a __name__ | tr '\n' ' '` )
CLUSTERS=( `cpmiquerybin attr "" network_objects "(type='gateway_cluster') & (location='internal')" -a __name__ | tr '\n' ' '` )
CLUSTER MEMBERS=( `cpmiquerybin attr "" network_objects "(type='cluster_member') | (type='gateway') & (location='internal')" -a __name__ | tr '\n
```

### ~~add description~~

```bash
cpmiquerybin attr "" network_objects "type='gateway'|type='cluster_member'|type='gateway_cluster'" -a __name__,ipaddr,svn_version_name,appliance_type
```

## Miscellaneous

### Emoji

```
¯\_(ツ)_/¯
```