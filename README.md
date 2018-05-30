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
[Expert@Checkpoint]# cplic print > cplic.txt

[Expert@Checkpoint]# cat cplic | grep -o -P  '..?Jan.*?....|..?Feb.*?....|..?Mar.*?....|..?Apr.*?....|..?May.*?....|..?Jun.*?....|..?Jul.*?....|..?Aug.*?....|..?Sep.*?....|..?Oct.*?....|..?Nov.*?....|..?Dec.*?....'
```

### Print between double quotes

```bash
grep -o '".*"' | tr -d '"'
```

### Print between single quotes

```bash
grep -oP "(?<=').*?(?=')"
```

### Print between ()

```bash
grep -oP '(?<=\()[^\)]+'
grep -oP '\(\K[^\)]+'
```

### Print everything after match

```bash
grep -o "<MATCH>.*"
```

### Print IPv4 addresses

```bash
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}"
```

### Print lines containing the exact MATCH

```bash
grep -oh "\w*<MATCH>\w*"
```

### Print lines with numbers only

```bash
grep --only-matching '[[:digit:]]\+'
```

### Print lines with only one /

```bash
grep -e '^[^/]*/[^/]*$'
```

### Print match only

```bash
grep -E -o '<MATCH>\w+'
```

### Print word before match

```bash
grep -E "MATCH" | cut -d "," -f2 | awk '{print $1}'
```

### Print word containing string

```bash
grep -oh "\w*<STRING>\w*"
```

### Print between two patterns

```bash
grep -o -P '(?<=PATTERN1).*(?=PATTERN2)'
```

## awk Tricks

### Print between square brackets

```bash
awk 'NR>1{print $1}' RS=[ FS=]
```

### Print between ()

```bash
awk -F'[()]' '{print $2}'
```

### Print lines with numbers only

```bash
awk '{gsub("[^[:digit:]]+"," ")}1'
```

### Print the line immediately before a line that matches "/regex/" (but not the line that matches itself):

```bash
awk '/regex/ { print x }; { x=$0 }'
```

### Print the line immediately after a line that matches "/regex/" (but not the line that matches itself):

```bash
awk '/regex/ { print (x=="" ? "match on line 1" : x) }; { x=$0 }'
```

### Print column after match

```bash
awk -v srch="<PATTERN>" 'BEGIN{l=length(srch)}{t=match($0,srch);if(!t){next}$0=substr($0,t+l);print srch" "$2}' <filename> | awk '{print $1}'
```

### Print from column 3 to end

```bash
awk '{ print substr($0, index($0,$3)) }'
```

### Print lines that match any of "AAA" or "BBB", or "CCC"

```bash
awk '/AAA|BBB|CCC/'
```

### Replace column value

```bash
awk '{$<COL_NUMBER> = "<VALUE>"; print}'
```

### Turn list to table

```bash
cat FILENAME.txt | awk 'BEGIN { print "<table>" }
     { print "<tr><td>" $1 "</td><td>" $2 "</td><tr>" }
     END   { print "</table>" }'
	 
	 cat toTable | awk 'BEGIN { print "<tbody>" }
     { print "<tr><td><strong>" $1 "</strong></td>" }
	 { print "<td>" $2 "</td></tr>" }
     END   { print "</tbody>" }'
```

### Print between two patterns

```bash
awk -v FS="(PATTERN1|PATTERN2)" '{print $2}'
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

### insert a blank line above every line which matches "regex"
 ```bash
 sed '/regex/{x;p;x;}'
 ```
 
### insert a blank line below every line which matches "regex"
 ```bash
 sed '/regex/G'
 ```
 
 ### insert a blank line above and below every line which matches "regex"
 ```bash
 sed '/regex/{x;p;x;G;}'
 ```

### number each line of file, but only print numbers if line is not blank
 ```bash
 sed '/./=' filename | sed '/./N; s/\n/ /'
 ```
 
 ### substitute "foo" with "bar" ONLY for lines which contain "baz"
  ```bash
  sed '/baz/s/foo/bar/g'
  ```
 
 ### substitute "foo" with "bar" EXCEPT for lines which contain "baz"
  ```bash
  sed '/baz/!s/foo/bar/g'
  ```
  
   ### reverse order of lines (similar to "tac")
   ```bash
   sed '1!G;h;$!d'               
   sed -n '1!G;h;$p'             
   ```
   
   ### if a line ends with a backslash, append the next line to it
   ```bash
    sed -e :a -e '/\\$/N; s/\\\n//; ta'
   ```
   
   # if a line begins with MATCH, append it to the previous line and replace MATCH with space
   ```bash
    sed -e :a -e '$!N;s/\nMATCH/ /;ta' -e 'P;D'
   ```
  
  ### reverse each character on the line (similar to "rev")
   ```bash
   sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//'
   ```
  
 ### change "witch" or "gem" or "puss" to "red"
 ```bash
 sed 's/witch/red/g;s/gem/red/g;s/puss/red/g'   # most seds
 gsed 's/witch\|gem\|puss/red/g'                # GNU sed only
 ```
 
 ### substitute (find and replace) "foo" with "bar" on each line
  ```bash
  sed 's/foo/bar/'             # replaces only 1st instance in a line
  sed 's/foo/bar/4'            # replaces only 4th instance in a line
  sed 's/foo/bar/g'            # replaces ALL instances in a line
  sed 's/\(.*\)foo\(.*foo\)/\1bar\2/' # replace the next-to-last case
  sed 's/\(.*\)foo/\1bar/'            # replace only the last case
  ```

### delete leading whitespace (spaces, tabs) from front of each line
 ```bash
 sed 's/^[ \t]*//'                    # see note on '\t' at end of file
 ```

### delete trailing whitespace (spaces, tabs) from end of each line
 ```bash
 sed 's/[ \t]*$//'                    # see note on '\t' at end of file
 ```

### print lines which match regexp
```bash
sed -n '/regexp/p'           # method 1
sed '/regexp/!d'             # method 2
```

### print line before regexp
```bash
sed -n '/regexp/{g;1!p;};h'
```

### print line after regexp
```bash
sed -n '/regexp/{n;p;}'
```

### Delete all empty lines
```bash
sed '/^\s*$/d' file
```

### Add line after a line with match
```bash
sed '/MATCH/a\ADD_THIS' file
```

### Add line before a line with match
```bash
sed '/MATCH/i\<INSERT>' file
```

### Add string to beginning of all lines
```bash
sed 's/^/<STRING>/'
```

### print lines with AAA, BBB and CCC (any order)
```bash
sed '/AAA/!d; /BBB/!d; /CCC/!d'
```

### Add string to beginning of all matching lines
```bash
sed '/<MATCH>/s/^/<STRING>/'
```

### print section of text if it contains MATCH (blank lines separate paragraphs)
```bash
sed -e '/./{H;$!d;}' -e 'x;/MATCH/!d;'
```

### print section of text if it contains AAA and BBB and CCC (in any order)
```bash
sed -e '/./{H;$!d;}' -e 'x;/AAA/!d;/BBB/!d;/CCC/!d'
```

### print section of text if it contains AAA or BBB or CCC
```bash
sed -e '/./{H;$!d;}' -e 'x;/AAA/b' -e '/BBB/b' -e '/CCC/b' -e d
```

### print from regexp to end of file 
```bash
sed -n '/regexp/,$p'
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

### Print name from escape characters

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

### Print after a line containing ABC

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
alias network='service network restart'
alias f='find / -name'
alias fhere='find . -name'
alias iptables='service iptables restart'
```

### Clear file descriptors of deleted files

```bash
lsof | grep "(deleted)$" | sed -re 's/^\S+\s+(\S+)\s+\S+\s+([0-9]+).*/\1\/fd\/\2/' | while read file; do bash -c ": > /proc/$file"; done
```

### Convert one line xml to normal

```bash
echo "<XML>" or cat file | xml_pp
```

### Delete everything after match

```bash
cut -d "<MATCH>" -f1
```

### URI encode string for URL requests (wget and curl)

```bash
perl -MURI::Escape -lne 'print uri_escape($_)'
alias hashpass='echo $PASS | awk -F : '"'"'{for (i=1;i<=NF;i++) {print $i}}'"'"
```

### Print between ()

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

### Print diff between two files using diff

```bash
diff -a --suppress-common-lines -y File1 File2
```

### Print names of all installed packages

```bash
rpm -qa --qf "%{NAME}\n"
```

## Regex Tricks

### Regex to match word of N characters

```bash
^\w{0,10}$ # allows words of up to 10 characters.
^\w{5,}$   # allows words of more than 4 characters.
^\w{5,10}$ # allows words of between 5 and 10 characters.
```

## BackBox Specific

### Send a mail from BackBox CLI

```bash
/backbox/backbox-3.0/bin/sendEmail -f alerts@backbox.co -t SENDER@backbox.co -s SMTP_ADDRESS -u MailTest -o message-file=
```

## Check Point Tricks

### Anti-spoofing check on all firewall interfaces

```bash
fw="xxx"; cpmiquerybin object "" network_objects "name='$fw'" |grep anti_spoof
```

### Print cluster names and IP(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "type='gateway_cluster'" -a __name__,ipaddr
```

### Print a list of names of all objects of type cluster member(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "type='cluster_member'" -a __name__
```

### Print a list of names of all valid cluster members from cluster object name(MDS/Provider-1)

```bash
cpmiquerybin object "" network_objects "" |grep -A 12 cluster_members |grep Name | awk -F "(" '{printf $2}' | sed -e 's/)/|/g'
cpmiquerybin attr "" network_objects "name='cluster_name'" -a cluster_members
```

### Print all members of a group(MDS/Provider-1)

```bash
cpmiquerybin object "" network_objects "name='group_name_goes_here'" | grep ":Name"
```

### Print CMA list of policy collections(MDS/Provider-1)

```bash
cpmiquerybin attr "" policies_collections "" -a __name__
```

### Print CMA policy names(MDS/Provider-1)

```bash
cpmiquerybin attr "" fw_policies "" -a __name__
```

### Print installable targets for a policy named standard(MDS/Provider-1)

```bash
cpmiquerybin attr "" policies_collections "name='Standar'" -a __name__,installable_targets
```

### Print IP for CLM name(MDS/Provider-1)

```bash
cpmiquerybin attr "mdsdb" network_objects "name='Cluster1'" -a __name__,ipaddr
```

### Print secondary CMA(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "(primary_management='false') & (management='true')" -a __name__
```

### List all MDSs(MDS/Provider-1)

```bash
cpmiquerybin attr "mdsdb" mdss "" -a __name__
```

### List CMAs(MDS/Provider-1)

```bash
cpmiquerybin attr "mdsdb" network_objects "management='true'" -a __name__,ipaddr
```

### List management/cma objects from cma env(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "management='true'" -a __name__,ipaddr 
```

### List primary MDS(MDS/Provider-1)

```bash
cpmiquerybin attr "mdsdb" mdss "primary='true'" -a __name__
```

### List services with 'Match for Any' ticked(MDS/Provider-1)

```bash
cpmiquerybin attr "" services "include_in_any='true'" -a __name__
```

### Query all objects for an ip address(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "ipaddr='<IP>'" -a __name__,ipaddr
```

### Query all Standalone Firewalls(MDS/Provider-1)

```bash
GATEWAYS=( `cpmiquerybin attr "" network_objects "(type='gateway') & (location='internal')" -a __name__ | tr '\n' ' '` )
CLUSTERS=( `cpmiquerybin attr "" network_objects "(type='gateway_cluster') & (location='internal')" -a __name__ | tr '\n' ' '` )
CLUSTER MEMBERS=( `cpmiquerybin attr "" network_objects "(type='cluster_member') | (type='gateway') & (location='internal')" -a __name__ | tr '\n'
```

### Query all Cluster Members(MDS/Provider-1)

```bash
cpmiquerybin attr "" network_objects "type='gateway'|type='cluster_member'|type='gateway_cluster'" -a __name__,ipaddr,svn_version_name,appliance_type
```

## Miscellaneous

### Date of yesterday
```bash
date -d@$(echo $(($(date +"%s")-86400))) +"%Y-%m-%d"
```

### Emoji

```
¯\_(ツ)_/¯
```
