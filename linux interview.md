https://kodekloud.com/blog/linux-interview-questions/


| Old knowledge            | Modern RHEL                        |
| ------------------------ | ---------------------------------- |
| `service`                | `systemctl`                        |
| `chkconfig`              | `systemctl enable/disable`         |
| `yum`                    | `dnf` / `yum` compatibility        |
| `ntpd`                   | `chronyd`                          |
| iptables                 | firewalld/nftables                 |
| ifconfig                 | `ip`                               |
| netstat                  | `ss`                               |
| `route`                  | `ip route`                         |
| `rhui`/traditional repos | subscription/repository management |
| Python 2                 | Python 3                           |
| legacy network scripts   | NetworkManager/nmcli               |
| cgroups v1               | cgroups v2                         |
| older boot tooling       | GRUB2/systemd                      |
| SysV init                | systemd                            |


[root@rhel ~]# chronyc tracking
Reference ID    : 00000000 ()
Stratum         : 0
Ref time (UTC)  : Thu Jan 01 00:00:00 1970
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 1.000000000 seconds
Root dispersion : 1.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Not synchronised
[root@rhel ~]# 

Common Health Indicators

**Healthy Sync:** Leap status: Normal, System time within milliseconds, and Reference ID set to a valid host.
**Unsynchronized:** Reference ID: 00000000 () or Leap status: Not synchronized indicates chronyd is unreachable, rejected all sources, or still accumulating baseline metrics.

redirections
####
| Command | What It Does |
|---|---|
| `command > file` | Redirect stdout to file |
| `command 2> file` | Redirect stderr to file |
| `command 2>/dev/null` | Discard all errors |
| `command &> file` | Redirect both stdout + stderr to file |
| `command > file 2>&1` | Same as above (older syntax) |
| `command >> file` | Append stdout to file |
| `command 2>> file` | Append stderr to file |
| `command > out.txt 2> err.txt` | Separate stdout and stderr into different files |


FIND Command
####
#find / -name "Foo.txt" 2>/dev/null
#find / -iname "*foo*txt" 2>/dev/null

| Option | Meaning | Case Sensitive? |
|---|---|---|
| `-name` | Match filename pattern | ✅ **Yes** (case-sensitive) |
| `-iname` | Match filename pattern | ❌ **No** (case-**i**nsensitive) |

3. Find everything
The ls -R command lists the contents of a directory recursively, meaning that it doesn't just list the target you provide for it, but also descends into every subdirectory within that target (and every subdirectory in each subdirectory, and so on.) The find command has that function too, by way of the -ls option:
#find ~/Documents -ls

4. Find by content
A find command doesn't have to perform just one task. In fact, one of the options in find enables you to execute a different command on whatever results find returns. This can be especially useful when you need to search for a file by content rather than by name, or you need to search by both.

#find ~/Documents/ -name "*txt" -exec grep -Hi penguin {} \;
/home/seth/Documents/Foo.txt:I like penguins.
/home/seth/Documents/foo.txt:Penguins are fun.
{} — gets substituted with each matched file path.
\; — signals the end of the -exec expression.

root@rhel ~]# find /root/ -name "*txt" -exec ls -l {} \;
-rw-r--r--. 1 root root 0 Sep  3 18:56 /root/foo.txt
-rw-r--r--. 1 root root 60 Sep  3 19:05 /root/errors.txt
[root@rhel ~]# 

root@rhel ~]# find /root/ -name "*txt" -exec ls -l {} \;
-rw-r--r--. 1 root root 0 Sep  3 18:56 /root/foo.txt
-rw-r--r--. 1 root root 60 Sep  3 19:05 /root/errors.txt
[root@rhel ~]# 
#find ~/Public/ -maxdepth 1 -type d

SUID (Set User ID)
SUID is a special permission that allows a user to execute a file with the permissions of the file's owner, rather than the user running it.

Numeric value: 4000
Symbol: s in the owner's execute position
Common example: /usr/bin/passwd — regular users can change their own password because passwd runs as root (the file owner)

# Using symbolic mode
chmod u+s filename

# Using numeric mode
chmod 4755 filename

How to identify:
ls -l /usr/bin/passwd
# Output: -rwsr-xr-x 1 root root ... /usr/bin/passwd
#            ^ 's' here means SUID is set
Security Warning: SUID on shell scripts or untrusted binaries is a major security risk. An attacker could exploit it to gain root access.

--
SGID (Set Group ID)
SGID works similarly to SUID but applies to the group level. It behaves differently on files vs. directories.

Numeric value: 2000
Symbol: s in the group's execute position
On a file: The file executes with the group permissions of the file's group owner.

On a directory (more common): Any new files/subdirectories created inside will inherit the group of the parent directory, not the creating user's primary group. This is very useful for shared project folders.

How to set SGID:

# On a file
chmod g+s filename

# On a directory
chmod 2775 /shared/project

# Numeric mode
chmod 2755 filename

How to set SGID:
# On a file
chmod g+s filename

# On a directory
chmod 2775 /shared/project

# Numeric mode
chmod 2755 filename

---
Sticky Bit
The Sticky Bit is a permission set on directories that ensures only the file owner, directory owner, or root can delete or rename files within it — even if others have write permission.

Numeric value: 1000
Symbol: t in the others' execute position
Classic example: /tmp directory
***How to set Sticky Bit:**

# Symbolic mode
chmod +t /shared/directory

# Numeric mode
chmod 1777 /shared/directory

Why it matters: Without the sticky bit on /tmp, any user could delete any other user's temporary files since /tmp is world-writable.
----

ACL (Access Control Lists)
ACLs provide fine-grained permissions beyond the traditional owner/group/others model. They let you grant specific permissions to individual users or groups.

Prerequisites:

The filesystem must be mounted with acl option (most modern distros enable this by default)
Packages: acl (provides getfacl and setfacl)

| Command | Purpose |
|---------|---------|
| `getfacl` | View ACLs on a file/directory |
| `setfacl` | Set or modify ACLs |

# Grant read+write to a specific user
setfacl -m u:john:rw filename

# Grant read to a specific group
setfacl -m g:developers:r filename

# Set default ACL on a directory (inherited by new files)
setfacl -d -m u:john:rwx /shared/project

# Remove ACL for a specific user
setfacl -x u:john filename

# Remove ALL ACLs
setfacl -b filename




