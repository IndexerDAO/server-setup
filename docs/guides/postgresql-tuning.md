# PostgreSQL Performance Tuning

```
#Get `pid` of PostgreSQL instance
head -n 1 $PGDATA/postmaster.pid

#Find VmPeak for the instance (in kB)
grep -i vmpeak /proc/{replace with PID}/status

#Confirm huge page size = 2048 kB
grep -i hugepagesize /proc/meminfo

#Calculate number of huge pages required 
#round up to nearest tens place, ie 193-> 200
#VM Peak / Huge page size 

# Edit systemctl.conf
nano /etc/sysctl.conf
> #Hugepage setting
> vm.nr_hugepages = {Replace with rounded number of huge pages required} 

#Stop postgres (may need to change to our specific version)
systemctl stop postgresql-13

#Reload kernel parameters
systemctl -p

#Validate changes went through
grep -i hugepages_total /proc/meminfo

#Update postgresql.conf
nano /var/lib/pgsql/13/data/postgresql.conf
> huge_pages = on
> work_mem = {Result of GREATEST(4 x CPU cores, 100)}
> shared_buffers = {Result of LEAST(RAM/2, 10GB)}

#Start postgresql
#May need to change version of postgres
systemctl start postgresql-13

#Validate changes went through
grep -i hugepages_ /proc/meminfo
```

## Sources
* https://dbsguru.com/steps-to-enable-hugepages-in-postgresql/
* https://www.enterprisedb.com/postgres-tutorials/introduction-postgresql-performance-tuning-and-optimization
