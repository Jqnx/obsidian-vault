# **COMPUTER BESTURINGSYSTEMEN 1**

## Shared Folders

Users moetten permissions hebben voor beidde **lokale** en de **shared** folders.  

Gebruik hiervoor **net share** en **icacls**.  
***

## Net share

### Gewone share

```cmd
net share SHARENAME=DRIVE:PATH
```

### Share met user permissions

```cmd
net share SHARENAME=DRIVE:PATH [/grant:{user},{READ | CHANGE | FULL}]
```

### Share met aangepaste hoeveelheid users

```cmd
net share SHARENAME=DRIVE:PATH [/users:{users} | /unlimited]
```  

> Source: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh750728(v=ws.11)
***

## ICACLS

### Geef permission aan een user of groep voor een specifieke map

```cmd
icacls "{filename} | {PATH}" [/grant{:r} <user | group>:<permission>]
```

### The perm option is a permission mask that can be specified in one of the following forms:

#### A sequence of simple rights (basic permissions):

| Permission | Description |
| ---------- | ----------- | 
| F          | Full access |
| M          | Modify access |
| RX         | Read and execute access |
| R          | Read-only access |
| W          | Write-only access |

#### A comma-separated list in parenthesis of specific rights (advanced permissions):

| Permission | Description |
| ---------- | ----------- |
| D | Delete |
| RC | Read control (read permissions) |
| WDAC | Write DAC (change permissions) |
| WO | Write owner (take ownership) |
| S | Synchronize |
| AS | Access system security |
| MA | Maximum allowed |
| GR | Generic read |
| GW | Generic write |
| GE | Generic execute |
| GA | Generic all |
| RD | Read data/list directory |
| WD | Write data/add file |
| AD | Append data/add subdirectory |
| REA | Read extended attributes |
| WEA | Write extended attributes |
| X | Execute/traverse |
| DC | Delete child |
| RA | Read attributes |
| WA | Write attributes |

#### Inheritance rights may precede either perm form:

| Permission | Description |
| ---------- | ----------- |
| (I) |  Inherit. ACE inherited from the parent container.| 
| (OI) | Object inherit. Objects in this container will inherit this ACE. Applies only to directories. |
| (CI) | Container inherit. Containers in this parent container will inherit this ACE. Applies only to directories. |
| (IO) | Inherit only. ACE inherited from the parent container, but does not apply to the object itself. Applies only to directories. |
| (NP) | Do not propagate inherit. ACE inherited by containers and objects from the parent container, but does not propagate to nested containers. Applies only to directories. |

> Source: https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls