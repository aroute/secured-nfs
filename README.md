# secured-nfs
NFS setup for OpenShift, based on RHEL recommendations

## NFS Setup
### NFS Setup On Master
```bash
$ lsblk
$ mkfs.ext4 /dev/sdb
$ mkdir /var/nfs-data/
$ blkid
<!--- Example UID: /dev/vdb: UUID="607b9d47-9280-433d-a233-0f40f060ec51" TYPE="ext4" -->
$ vi /etc/fstab
UUID=6448dee0-04eb-4f10-8e19-f84c8a81bfbf /var/nfs-data ext4 defaults 0 0
$ mount -a
$ mount
$ cat <<EOF >> /etc/exports
/var/nfs-data 172.16.1.0/8(rw,root_squash)
/var/nfs-data 10.0.0.0/8(rw,root_squash)
EOF
$ chown -R nfsnobody.nfsnobody /var/nfs-data/
$ chmod -R 0770 /var/nfs-data/
$ ls -al /var/nfs-data/
$ iptables -L -v -n | grep 2049
$ iptables -I INPUT -p tcp --dport 2049 -j ACCEPT
$ service iptables save
$ for i in rpcbind nfs-server nfs-lock nfs-idmap;do systemctl restart $i;done
$ for i in rpcbind nfs-server nfs-lock nfs-idmap;do systemctl enable $i;systemctl start $i;done
$ exportfs
```

### NFS Setup On Nodes
```bash
$ ssh infra
$ mkdir -p /var/nfs-data && mount master:/var/nfs-data /var/nfs-data
$ cat <<EOF >> /etc/fstab
master:/var/nfs-data        /var/nfs-data       nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800    0       0
EOF
$ shutdown -r now
```
