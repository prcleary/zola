+++
title = "Backing up servers and desktops with borgmatic"
+++

To have a resilient backup strategy, you need:

- (at least) three copies of every file, database, VM/filesystem snapshot or whatever you don't want to lose - this allows for failure of systems
- two of those copies should be on different media (being on different computers seems to count) - this allows for failure of media
- one of those copies should be off-site (e.g. in the cloud) - this allows for catastrophes affecting your home/data centre

I maintain a mail server for a local football club. It runs on a cloud VM (one of Hetzner's cheapest tier offerings) and automatically makes backups of the emails locally. I've been using `rsync` to keep an up-to-date copy of the backups on my desktop machine. The alias in my `.bashrc` that I use is (sensitive details obscured):

```bash
alias getboxbackups='rsync -avzP --delete --rsh='\''ssh -p 9999'\'' box.example.com:/home/user-data/backup/encrypted/ /home/paul/Backups/box.example.com'
```

I have another alias which runs several other aliases for various system administration tasks such as updates and backups. I run this daily from my home Linux box, or via my phone if I am away. 

When I got my new home server I decided to try out Nextcloud All-In-One and this comes with [a backup process](https://github.com/nextcloud/all-in-one/discussions/4391) based on [BorgBase - Simple and Secure Offsite Backups](https://www.borgbase.com/). As I now had an account with them, I decided to back up my football emails there too. You get a terabyte for about $24 a year so it's probably cheaper than doing it yourself. 

It is not that complicated at the end of the day, but I did make some mistakes in the process, for example starting with the [comprehensive documentation on the borgmatic site](https://torsion.org/borgmatic/docs/how-to/set-up-backups/) and getting diverted into some similar instructions on the BorgBase site. The instructions below are mostly taken from the borgmatic site, which it is best to adhere to. I also had some problems with SSH, which seemed to work in some ways but not in others. After several rounds with asking for help from DeepSeek and searching the Web, I eventually solved the problem myself by realising something quite simple, which I mention below.  

I'm glad I worked it out, because this could be useful for all sorts of server backups. 

## Instructions

Install the software and check it can be run: this installs it so you can run it as root or as an ordinary user - I might want to do either. 

Note that after some steps you should exit your shell and open another. I also had to add a local executable path into the `sudo` configuration for this to work.

```bash
sudo apt install borgbackup pipx
sudo pipx ensurepath  # exit shell and open another
sudo pipx install borgmatic
sudo borgmatic --version  # check
pipx ensurepath  # exit shell and open another
pipx install borgmatic
sudo EDITOR=vim visudo # add `:/home/paul/.local/bin` to end of `secure_path`
borgmatic --version  # check
```

If you don't already have SSH keys then you should create them. *If you want to run borgmatic both as root and as an ordinary user, you should make sure that the SSH keys are in both `~/.ssh` and `/root/.ssh`, and that permissions and ownership are correct.*

Then you will need to copy the public key you want to use to borgbase.com.

```bash
cat ~/.ssh/id_ed25519.pub  # copy the output
```
Paste the output to the SSH keys section on borgbase.com and save it there.

Again on the borgbase.com site, create a new repository, specifying:

- name of repo
- storage in EU
- type: Borg
- select key (append-only access - stops an intruder from permanently deleting your backups)
- alerting if no backup after 1 day
- enable compaction
- finally add the repo

You should then get a URL for the remote repo, in the format: `ssh://xxxxxxxx@xxxxxxxx.repo.borgbase.com/./repo`

I wanted to create a local backup as well, so I created a folder for that outside my home directory:

```bash
sudo mkdir -p /mnt/backup
sudo chown -R $USER:$USER /mnt/backup  
```

Next you want to create a template configuration file for borgmatic:

```bash
sudo borgmatic config generate
```

You can edit the configuration file you just created with:

```bash
sudo vim /etc/borgmatic/config.yaml
```

My configuration file looks like this (there are lots of other options I haven't explored):

```
source_directories:
  - /home/paul/Backups
  - /home/paul/.ssh
  - /home/paul/ansible
  - /home/paul/.bashrc
repositories:
  - path: "ssh://xxxxxxxx@xxxxxxxx.repo.borgbase.com/./repo"
    label: pop-os-Backups
  - path: /mnt/backup
    label: local
keep_daily: 7
keep_weekly: 4
keep_monthly: 6
keep_yearly: 1
checks:
  - name: repository
  - name: archives
    frequency: 2 weeks
encryption_passphrase: "flashing-unsuited-trench-shape-plexiglas-swizzle"  # not real!
```

What this does is (in a single configuration file - cool):

- Specify multiple files or folders for backup
- Specify where the backups are to go
- Retention periods for backups
- Checks to be done on the backups
- The passphrase I want to use for encryption

It is easy to end up with invalid YAML in your configuration file, so make sure to check it with:

```bash
sudo borgmatic config validate
```

Next you need to "initiate" your repositories:

```bash
sudo borgmatic init --encryption=repokey 
```

Then you should obtain the Borg encryption keys, which are different to your SSH key and different for each location you are backing up to (you will need these and the passphrase for recovery):

```bash
sudo borg key export ssh://xxxxxxxx@xxxxxxxx.repo.borgbase.com/./repo  
sudo borg key export /mnt/backup
```

Save each key shown, which each looks like this:

```
BORG_KEY aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa...
...
```

Now your repositories are set up, you can do a test backup:

```bash
sudo borgmatic create --verbosity 2 --dry-run
```

If that works without errors (I had to go back and try to fix SSH several times, before realising that both my user and root needed the keys), you can now run a backup:

```bash
sudo borgmatic create --verbosity 2 --list --stats
```

This may take a while if it is a big backup. 

The command which you will then run routinely is the following, which does the backup as well as other routine maintenance:

```bash
sudo borgmatic --verbosity 2 --list --stats
```

You can run the backup manually whenever you like, or (better) create a CRON job to run the backup/maintenance at 3am each day, in which case you need to create a file containing this line, with the commands below:

```
0 3 * * * root PATH=$PATH:/usr/bin:/usr/local/bin /root/.local/bin/borgmatic --verbosity 2 --syslog-verbosity 2
```

```bash
vim borgmatic  # add line above - can reduce verbosity if preferred
sudo mv borgmatic /etc/cron.d/borgmatic
sudo chown root:root /etc/cron.d/borgmatic
sudo chmod 644 /etc/cron.d/borgmatic
```

Check the CRON command can run with:
```bash
PATH=$PATH:/usr/bin:/usr/local/bin /root/.local/bin/borgmatic --verbosity 2 --syslog-verbosity 2
```

You now have a 3-2-1 backup strategy, with various checks.

If you lose a file, you can search for versions of it in your backups with:

```bash
sudo borgmatic list --find myfile.docx
```

I haven't tried this yet, but borgmatic has `extract` and `restore` commands for restoring from backups - I intend to try these out, as a resilient backup strategy should also check that the backups can be restored. 



