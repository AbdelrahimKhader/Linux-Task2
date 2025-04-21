# Linux Task 2

```bash
crontab root -e
#ADD The Following
0 */1 * * * script3.sh
touch script3.sh
touch script4.sh
chmod +x script3.sh
chmod +x script4.sh
#USING VIM for script3.sh ADD : 
cd TS_DATA
echo -e "$(echo "DISK(MB):......")\n$(df -h -B1M | grep -E 'File|sd') \n\n\n$(echo "MEM(MB):......")\n$(free -m|grep Mem -B1|tr -s " "|cut -d" " -f2,3,4|awk '{print $1 ,"\t" $2,"\t"  $3}'|sed '/./,/^$/s/^ //')\n\n\n$(echo "CPU:......")\n $(iostat | grep -2  avg
) \n\n\n$(echo "END:......")" | tee $(date | tr ' ' '_').txt
if ! test -f avg_calc ; then # to check non existance of file
        touch avg_calc
fi
cd

###
#########  POINT2 : ####################
crontab root -e
#ADD The Following
0 */1 * * * sleep 1;script4.sh
##ADD THE FOLLOWING TO script4.sh USING VIM ###
awkavgs() {
awk 'NR == 1 {
  for (i=1; i<=NF; i++) {
    FNames[i]=$i
  }
  next
}
{
  for (i=1; i<=NF; i++) {
    if ($i !~ /[a-zA-Z]/){
        Sum[i]+=$i
        count[i]+=1
    }
    else{
        Sum[i]=0
        count[i]=0
    }
  }
}
END {
  for (i=1; i<=length(FNames); i++) {
    if (count[i] != 0 && Sum[i] !=0 ) {
      print "Avg Of ",FNames[i],"Equals ",Sum[i]/count[i]
    }
  }
}'
}
#avg_calc file creation
count=0
cd TS_DATA
if [ "$(wc -l < avg_calc)" -ne 0 ]; then
for file in $(ls -t); do # Sort files by time of modification
 if [[ "$file" != "avg_calc" && "$file" != *avg*  && $count -lt 1 ]]; then #To find the newest file that have the timestamp form in its name
          echo "$(awk -v D="$(awk '/DISK/,/MEM/' "$file" | sed '1d;2d;$d')" \
               -v M="$(awk '/MEM/,/CPU/' "$file" | sed '1d;2d;$d')" \
               -v C="$(awk '/CPU/,/END/' "$file" | sed '1d;2d;3d;4d;$d')" \
               -v C="$(awk '/CPU/,/END/' "$file" | sed '1d;2d;3d;4d;$d')" \
               '{sub(/ADD_DISK_RECORD/, D "\nADD_DISK_RECORD")}1 {sub(/ADD_MEM_RECORD/, M "\nADD_MEM_RECORD")}1 {sub(/ADD_CPU_RECORD/, C "\nADD_CPU_RECORD")}1' avg_calc)" > avg_calc_temp
    cat avg_calc_temp > avg_calc
        count=$((count + 1))
 elif [[ $count -ge 1 ]]; then
        break #If the newest file that is not other than the timestamp file is found, break
 fi
done
else
for file in *; do
 # possible edits: 1) to avoid reading all files every new run, check if the current avg_calc file is empty, read all files, if not, only append
 # 2) to append, add a regix to the end of each field (such as : "ADD MEM" "ADD DISK" etc) to easily replace this regix with the new field record
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $count -eq 0 ]; then
    awk '/DISK/,/MEM/' "$file" | sed '$d' > avg_calc
  else
    awk '/DISK/,/MEM/' "$file" | sed '1d;2d;$d' >> avg_calc
  fi
  count=$((count + 1))
 fi
done
echo "ADD_DISK_RECORD" >> avg_calc
echo $count
count=0
# Second loop: MEM info
for file in *; do
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $count -eq 0 ]; then
    awk '/MEM/,/CPU/' "$file" | sed '$d' >> avg_calc
  else
    awk '/MEM/,/CPU/' "$file" | sed '1d;2d;$d' >> avg_calc
  fi
  count=$((count + 1))
 fi
done
echo "ADD_MEM_RECORD" >> avg_calc
count=0
# Third loop: CPU info
for file in *; do
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $count -eq 0 ]; then
    awk '/CPU/,/END/' "$file" | sed '$d' >> avg_calc
  else
    awk '/CPU/,/END/' "$file" | sed '1d;2d;3d;4d;$d' >> avg_calc
  fi
  count=$((count + 1))
 fi
done
echo "ADD_CPU_RECORD" >> avg_calc
fi
echo "MEM AVGs(MB) :......." >  stamp_avg.txt
awk '/MEM/,/ADD_MEM_RECORD/' avg_calc |sed '1d; $d' | tr -s ' ' | cut -d" " -f2  | awkavgs >> stamp_avg.txt
echo "CPU AVGs :......." >>  stamp_avg.txt
grep avg-cpu avg_calc -A2 >> stamp_avg.txt
cd

#######
### POINT3 APPACHE SERVER AND HTML FILE: #######
touch script5.sh 
chmod +x script5.sh 
yum install httpd
systemctl enable httpd
systemctl start httpd
cd /var/www/html
touch index.html 
touch cpu.html
touch memory.html
touch disk.html
touch TS_LIST.html
vim index .html 
### ADD ### :
<!DOCTYPE html>
<html>
<head>
  <title>Linux Task Web Page</title>
</head>
<body>
  <h1>Welcome To Linux Task Web Page</h1>
  <ul>
    <li><a href="cpu.html">CPU USAGE</a></li>
    <li><a href="memory.html">Memory USAGE</a></li>
    <li><a href="disk.html">Disk USAGE</a></li>
    <li><a href="TS_LIST.html">List Of Data With Timestamp</a></li>
  </ul>
</body>
</html>
##############
cd
vim script5.sh
#### ADD ################## 
#Display Avgs:
echo "<html><body><pre>" > /var/www/html/memory.html
awk '/MEM/,/DISK/' /root/TS_DATA/stamp_avg.txt | sed '$d' >> /var/www/html/memory.html
echo "</pre></body></html>" >> /var/www/html/memory.html
echo "<html><body><pre>" > /var/www/html/disk.html
echo "<html><body><pre>" > /var/www/html/cpu.html
awk '/CPU/ {flag=1} flag' /root/TS_DATA/stamp_avg.txt | sed '$d'>> /var/www/html/cpu.html
echo "</pre></body></html>" >> /var/www/html/cpu.html
#List to display data with timestamp:
cd TS_DATA

count=$(awk '/<!--FROM_HERE-->/,/<!--TO_HERE-->/' /var/www/html/TS_LIST.html|sed '1d;$d' | wc -l)
thr=8 #Maximum number of files in directory
for file in $(ls -t); do
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $(($thr - $count)) -eq 0 ]; then
   awk -v L="<li> <a href=\"${file}\">${file}</a></li>" \
    -v R="$(grep \<\!--FROM_HERE--\> -A1 /var/www/html/TS_LIST.html | sed '1d')" \
    '{sub(/<!--TO_HERE-->/, L "\n<!--TO_HERE-->") ;sub(R,"")}1 ' \
    /var/www/html/TS_LIST.html > /var/www/html/TS_LIST_temp.html #The Awk will Replace the first entry(Oldest) after the regex with the new entry
cp /var/www/html/TS_LIST_temp.html /var/www/html/TS_LIST.html
sed -i '/^$/d' /var/www/html/TS_LIST.html # TO Remove Empty Lines (Affected the calculation of current file entries)
cd /var/www/html
rm -f "$(ls -rt | grep txt | head -n 1)" #TO Remove The Oldest txt(timestamp) File From The Directory (The Html Directory)
cd - #return back to TS_DATA
mv $file /var/www/html
  elif [ $(($thr - $count)) -gt 0 ]; then
   mv $file /var/www/html
   awk -v L="<li> <a href=\"${file}\">${file}</a></li>" '{sub(/<!--FROM_HERE-->/,"<!--FROM_HERE-->\n" L)}1' /var/www/html/TS_LIST.html > /var/www/html/TS_LIST_temp.html
mv /var/www/html/TS_LIST_temp.html /var/www/html/TS_LIST.html
  else
    rm -f /var/www/html/$file
    rm -f /root/TS_DATA/$file
   fi
     count=$((count + 1))
  fi
done
cd
systemctl restart httpd 
#############################
crontab -e 
###ADD:####
0 */1 * * * sleep 2;/root/script5.sh
#### TRY TO CONNECT FROM ANOTHER MACHINE, IP:192.168.1.5 ####
open the browser and type 192.168.1.5
```
