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
) \n\n\n$(echo "END:......")" > $(date | tr ' ' '_').txt 
cd

###
#########  POINT2 : ####################
crontab root -e
#ADD The Following
0 */1 * * * sleep 1;script4.sh
##ADD THE FOLLOWING TO script4.sh USING VIM ###
##ADDING FUNCTION TO CALCULATE FIELDS AVGS FOR DIFFERENT NUMBER OF FIELDS AND ENTRIES###
awkavgs() {
awk 'NR == 1 {
  for (i=1; i<=NF; i++) {
    FNames[i]=$i
  }
  next
}
{
  for (i=1; i<=NF; i++) {
    Sum[i]+=$i
    count[i]+=1
  }
}
END {
  for (i=1; i<=length(FNames); i++) {
    if (count[i] != 0 && Sum[i]/count[i] !=0 ) {
      print "Avg Of ",FNames[i],"Equals ",Sum[i]/count[i]
    }
  }
}'
}
#avg_calc file creation
count=0
cd TS_DATA
for file in *; do
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $count -eq 0 ]; then
    awk '/DISK/,/MEM/' "$file" | sed '$d' > avg_calc
  else
    awk '/DISK/,/MEM/' "$file" | sed '1d;2d;$d' >> avg_calc
  fi
  count=$((count + 1))
 fi
done
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
echo "MEM AVGs(MB) :......." >>  $(date | tr ' ' '_')_avg.txt
awk '/MEM/,/CPU/' avg_calc |sed '1d; $d' | tr -s ' ' | awkavgs >> $(date | tr ' ' '_')_avg.txt
echo "DISK AVGs(MB) :......" >>  $(date | tr ' ' '_')_avg.txt
awk '/DISK/,/MEM/' avg_calc |sed '1d; $d' | tr -s ' ' | sed '/./,/^$/s/^ //' | awkavgs >> $(date | tr ' ' '_')_avg.txt
echo "CPU AVGs :......." >>  $(date | tr ' ' '_')_avg.txt
grep avg-cpu avg_calc -A2 >> $(date | tr ' ' '_')_avg.txt
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
awk '/DISK/,/CPU/' /root/TS_DATA/stamp_avg.txt | sed '$d' >> /var/www/html/disk.html
echo "</pre></body></html>" >> /var/www/html/disk.html
echo "<html><body><pre>" > /var/www/html/cpu.html
awk '/CPU/ {flag=1} flag' /root/TS_DATA/stamp_avg.txt | sed '$d'>> /var/www/html/cpu.html
echo "</pre></body></html>" >> /var/www/html/cpu.html
#List to display data with timestamp:
cd TS_DATA
 echo "<html>" > /var/www/html/TS_LIST.html
 echo "<body>" >> /var/www/html/TS_LIST.html
 echo "<h1>List Of Data With Time Stamps:</h1>" > /var/www/html/TS_LIST.html
 echo "<ul>" >> /var/www/html/TS_LIST.html
count=0
for file in $(ls -t); do
 if [[ "$file" != "avg_calc" && "$file" != *avg* ]]; then
  if [ $count -lt 8 ]; then
    cp $file /var/www/html
    echo "<li> <a href=\"$file\"</a>$file</li>" >> /var/www/html/TS_LIST.html
   fi
     count=$((count + 1))
  fi
done
echo "</ul>" >> /var/www/html/TS_LIST.html
echo "</body>" >> /var/www/html/TS_LIST.html
echo "</html>" >> /var/www/html/TS_LIST.html
cd
systemctl restart httpd 
#############################
crontab -e 
###ADD:####
0 */1 * * * sleep 2;/root/script5.sh
#### TRY TO CONNECT FROM ANOTHER MACHINE, IP:192.168.1.5 ####
open the browser and type 192.168.1.5
```