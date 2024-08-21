---
title: Borg backup server with docker
tags:
  - docker
  - backup
  - self-host
  - cloud
  - network
  - security
date: 2024-08-21
description: How to setup BorgBackup server to remotely backup your server
icon: 💾
---
> [!warning] Work in progress

---
## Intro

When you have a home server with a lot of data and services, it's really important to back up your disks correctly in case of disk failure or accidental mistakes. The 3-2-1 rule is a common guideline that suggests you should have:

- 3 copies of your data
- 2 different places
- 1 copy kept off-site

One way to create an offline backup is by using BorgBackup as a server. It lets you back up a folder remotely with encryption. Borg uses a differential system, which means it only updates the files that have changed between the old and new versions, to improve the speed and efficiency of the backup process.

In this guide, I'll show you how to configure a Borg server using Docker, enabling you to securely back up any folders from your home server disks.

---
## Prerequisite

Before we start, ensure you have the following prerequisites:

- A server or machine with Docker installed.
- Basic understanding of Docker concepts such as containers, images, and volumes.
- Docker Compose.
- A remote machine where the backup will be stored.

---
## Configuration

Log in to your remote machine and begin configuring your Borg server.

Create a `borg-server` folder and navigate into it:
```bash
mkdir borg-server && cd borg-server
```

Create a `docker-compose.yml` file with the content below:
```yml
services:
  borg:
    image: horaceworblehat/borg-server
    restart: unless-stopped
    environment:
      BORG_UID: "1000" # optional: your user id (run id in bash)
      BORG_GID: "1000" # optional: your group id (run id in bash)
    volumes:
      - ./backups:/home/borg/backups
      # persist the hosts ssh-keys across updates
      - ./server_keys:/var/lib/docker-borg
      - ./authorized_keys:/home/borg/.ssh/authorized_keys
    ports:
      - "8022:22"
```

> [!note] Note
> Borg uses the SSH protocol in server mode, which is why port 22 is the default. In our setup, we'll map it to 8022 to avoid conflicts with the host SSH server.

Create the `authorized_keys` file and copy your public SSH key from your home server. You can display the key using the command:
```bash
cat ~/.ssh/id_rsa.pub
```

Start your Borg server using the command:
```bash
docker compose up -d
```

Now test the connection using the command from your home server:
```bash
ssh borg@<ip-remote-machine> -p 8022
```

You should be able to connect without a password if you have installed the public key correctly.

## Backup

Now the borg server of the remote machine is running, you can test it using borg client.

Firstly, install borg client on your home server:
```bash
sudo apt install borgbackup
```

Now run the command to create a new `test` backup repository:
```bash
borg init -e repokey ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test
```

Enter a passphrase to encrypt the folder.

Now verify the backup folder using the command `info`:
```bash
borg info ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test
```

It should display something like:
```text
Enter passphrase for key ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test: 
Repository ID: e1bc9e10ac83535459d72126094701811d89b5250b96d65d6da0fc699eb09fb1
Location: ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test
Encrypted: Yes (repokey)
Cache: /root/.cache/borg/e1bc9e10ac83535459d72126094701811d89b5250b96d65d6da0fc699eb09fb1
Security dir: /root/.config/borg/security/e1bc9e10ac83535459d72126094701811d89b5250b96d65d6da0fc699eb09fb1
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                    0 B                  0 B                  0 B

                       Unique chunks         Total chunks
Chunk index:                       0                    0
```

You can now create your first backup of `/etc` using the command:
```bash
borg create --verbose --stats --compression lz4 ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test::test-{now:%Y-%m-%d_%H:%M:%S} /etc
```

```text
Enter passphrase for key ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test: 
Creating archive at "ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test::test-2024-08-21_22:07:36"
------------------------------------------------------------------------------
Repository: ssh://borg@<ip-remote-machine>:8022/home/borg/backups/test
Archive name: test-2024-08-21_22:07:36
Archive fingerprint: 777ec39e292c5ef616c4e5ca75e14eae0383564401d32993b6cc82bca8561eaa
Time (start): Wed, 2024-08-21 22:07:39
Time (end):   Wed, 2024-08-21 22:07:40
Duration: 0.52 seconds
Number of files: 136
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:              453.43 kB            312.77 kB            312.71 kB
All archives:              452.77 kB            312.11 kB            333.84 kB

                       Unique chunks         Total chunks
Chunk index:                     133                  134
------------------------------------------------------------------------------
```


---
## Conclusion

With the Borg server configured using Docker, you can now easily back up all your sensitive folders to a remote machine. This setup provides encryption and differential backups to minimize duplication and optimize transfer speed and time.

---
## Resources

- Photo by [Behnam Norouzi](https://unsplash.com/@behy_studio?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-white-and-blue-computer-chip-8FsybY-URs0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
- BorgBackup website: https://www.borgbackup.org/
- Docker BorgBackup image github: https://github.com/AnotherStranger/docker-borg-backup/
