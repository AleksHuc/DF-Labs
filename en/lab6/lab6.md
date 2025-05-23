# 6. Lab: Finding usernames and passwords

## Instructions

1. Determine the usernames and passwords for the specified Windows XP virtual disk.
2. Reset Windows XP user password.
3. Find the usernames and passwords for the given Linux virtual machine.

## More information

## Detailed instructions

### 1. Cracking passwords in Windows XP

To our virtual computer with the Linux operating system, we attach the `truplo1.vmdk` virtual disk using the `mount` command to the `/mnt/truplo1` folder, if it does not already exist, create it using the `mkdir` command. If we don't have a virtual disk, download the [`truplo.zip'](https://polaris.fri.uni-lj.si/truplo.zip) archive that contains it from the website.

    lsblk

    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda      8:0    0   20G  0 disk 
    ├─sda1   8:1    0   19G  0 part /
    ├─sda2   8:2    0    1K  0 part 
    └─sda5   8:5    0  975M  0 part [SWAP]
    sdb      8:16   0   10G  0 disk 
    └─sdb1   8:17   0   10G  0 part 
    sr0     11:0    1 60.8M  0 rom  

    mkdir /mnt/truplo1
    mount /dev/sdb1 /mnt/truplo1

Windows 7 and older operating systems hide passwords in the file `C:\windows\system32\config\SAM`, you can read more about it [here](https://www.ijfcc.org/vol5/455-F005.pdf) and [here](https://learn.microsoft.com/en-us/previous-versions//cc723740(v=technet.10)?redirectedfrom=MSDN). Passwords are not stored in text format, but in values from hash function of given passwords:

$$Password \rightarrow HashFunction(Password) \rightarrow HashedPassword$$

With the `reglookup` command we can find the name of an individual user and their alias. Under their alias we have the hashed password stored.

    reglookup /mnt/truplo1/WINDOWS/system32/config/SAM | grep dobrota

    /SAM/Domains/Account/Users/Names/dobrota,KEY,,2014-03-25 11:42:34
    /SAM/Domains/Account/Users/Names/dobrota/,0x000003EC,(null),

    reglookup /mnt/truplo1/WINDOWS/system32/config/SAM | grep 000003EC

    /SAM/Domains/Account/Users/000003EC,KEY,,2014-03-25 12:54:35
    /SAM/Domains/Account/Users/000003EC/F,BINARY,%02%00%01%00%00%00%00%00%AA%AC%93_)H%CF%01%00%00%00%00%00%00%00%00%C8%BCfY%1FH%CF%01%FF%FF%FF%FF%FF%FF%FF%7F~%D3%F4])H%CF%01%EC%03%00%00%01%02%00%00%10%02%00%00%00%00%00%00%00%00%01%00%01%00%00%00%00%00%FF%FF%BB%01%91|,
    /SAM/Domains/Account/Users/000003EC/V,BINARY,%00%00%00%00%BC%00%00%00%02%00%01%00%BC%00%00%00%0E%00%00%00%00%00%00%00%CC%00%00%00%0E%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%00%00%00%00%00%00%00%00%DC%00%00%00%08%00%00%00%01%00%00%00%E4%00%00%00%14%00%00%00%00%00%00%00%F8%00%00%00%14%00%00%00%00%00%00%00%0C%01%00%00%04%00%00%00%00%00%00%00%10%01%00%00%04%00%00%00%00%00%00%00%01%00%14%80%9C%00%00%00%AC%00%00%00%14%00%00%00D%00%00%00%02%000%00%02%00%00%00%02%C0%14%00D%00%05%01%01%01%00%00%00%00%00%01%00%00%00%00%02%C0%14%00%FF%07%0F%00%01%01%00%00%00%00%00%05%07%00%00%00%02%00X%00%03%00%00%00%00%00$%00D%00%02%00%01%05%00%00%00%00%00%05%15%00%00%00%BE%04>2%FE&%C6H%07%E5;+%EC%03%00%00%00%00%18%00%FF%07%0F%00%01%02%00%00%00%00%00%05 %00%00%00 %02%00%00%00%00%14%00[%03%02%00%01%01%00%00%00%00%00%01%00%00%00%00%01%02%00%00%00%00%00%05 %00%00%00 %02%00%00%01%02%00%00%00%00%00%05 %00%00%00 %02%00%00d%00o%00b%00r%00o%00t%00a%00%00%00d%00o%00b%00r%00o%00t%00a%00%01%00%01%02%00%00%07%00%00%00%01%00%01%00%8E%EA%A4%DF%97%FCo%02y%DB%F4%9C~{<P%01%00%01%00?p%C6%93/%15%F1%DCU%904%E9%11v%90'%01%00%01%00%01%00%01%00,
    /SAM/Domains/Account/Users/Names/dobrota/,0x000003EC,(null),
    /SAM/Domains/Builtin/Aliases/Members/S-1-5-21-842925246-1220945662-725345543/000003EC,KEY,,2014-03-25 11:42:35
    /SAM/Domains/Builtin/Aliases/Members/S-1-5-21-842925246-1220945662-725345543/000003EC/,EXPAND_SZ,!%02%00%00 %02%00%00,

This approach is not secure, since two identical passwords always return the same hashed password. From Windows 8 onwards, passwords are saved with added public random salt, which is a result of [cryptographic hash functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function), and the password hash process itself is further slowed down:

$$Password + (CryptographicHashFunction \rightarrow Salt) \rightarrow HashFunction(Password + Salt) \rightarrow HashedPasswordSalt$$

Users can be extracted using the `samdump2` tool in the format `user_name:user_id:LM_password_hash:NT_password_hash:::`.

    apt update
    apt install samdump2

    cd /mnt/truplo1/WINDOWS/system32/config/

    samdump2 system SAM

    Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    *disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    *disabled* HelpAssistant:1000:5b3d0b9dc228f45d3f13a2fa950e2631:432641671724da41e2f20960a8507b3e:::
    *disabled* SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:46402f847d7ccacd8900efbe289fbfdc:::
    user:1003:bb144e649c912bb7aad3b435b51404ee:6dd6b52a708ce96932eca95bd7352dde:::
    dobrota:1004:fd6d85bd2d15715faad3b435b51404ee:f1436ba5ed97ba211565ebf6693bf117:::

We find and crack passwords with a dedicated tool that uses [rainbow tables](https://en.wikipedia.org/wiki/Rainbow_table) to find passwords. Rainbow tables contain already calculated values of hash functions for frequently used passwords. The hashing function assures us with a high probability that two different inputs almost never map to the same output value. We install the tool with the mentioned functionality [`ophcrack`](https://manpages.org/ophcrack) and [`ophcrack-cli`](https://linuxcommandlibrary.com/man/ophcrack-cli) and the tool for opening archives [`unzip`](https://linux.die.net/man/1/unzip) with the package manager on our operating system.

    apt update
    apt install ophcrack ophcrack-cli unzip

Now we need the rainbow tables, which we get [here](https://ophcrack.sourceforge.io/tables.php). For our example, it is sufficient to download the tables [`XP free small`](https://polaris.fri.uni-lj.si/tables_xp_free_small.zip) and [`XP free fast`](https://polaris.fri.uni-lj.si/tables_xp_free_fast.zip) and open them.

    cd /home/USER

    wget https://polaris.fri.uni-lj.si/tables_xp_free_small.zip
    wget https://polaris.fri.uni-lj.si/tables_xp_free_fast.zip

    unzip tables_xp_free_fast.zip -d fast
    unzip tables_xp_free_small.zip -d small

Then we run the `ophcrack` tool via the `Activities` button in the upper left corner. In the upper row of buttons, click on `Tables` and then select the tables that we downloaded from the web `XP free fast` and `XP free small`.

![Ophcrack initial screen.](images/lab6-oph1.png)

Click on the `Install` button at the bottom right of each one, and through the wizard, select the place where they are located and press the `Open` button.

![Interface for selecting rainbow tables.](images/lab6-oph2.png)

After we have successfully selected both tables, press the `OK` button.

![Interface for selecting rainbow tables, where we have already successfully added them.](images/lab6-oph3.png)

Now we click on the `Load` button, where we select `Encrypted SAM`.

![Select the file where the usernames and passwords are stored.](images/lab6-oph4.png)

Then, through the wizard, find the folder in which `SAM` is located, namely `/mnt/truplo1/WINDOWS/system32/config`, and press the `Open` button. Usernames, password values from [LM hashing function](https://en.wikipedia.org/wiki/LAN_Manager#Password_hashing_algorithm), and password values from[NT hashing function](https://en.wikipedia.org/wiki/NTLM). The comparison of both hashing procedures is presented in this [article](https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4).

![Properly configured user password retrieval tool.](images/lab6-oph5.png)

Now press the `Crack` button and wait a bit for the tool to crack the passwords for the users found.

![Passwords obtained after a successful procedure.](images/lab6-oph6.png)

So we found two users, namely `user`, with the password `rednose` and `dobrota`, with the password `sirota`. Now we restart the virtual computer with the Windows XP operating system that we set up in the previous exercises and try to log in with the user names and their passwords.

### 2. Password reset

If we are not interested in specific passwords, we can just reset them and thus access the individual user account. For this we can use the [`chntpw`](https://manpages.org/chntpw/8) tool, which is installed with our operating system's package manager. Now mount the virtual disk of our virtual computer with the Windows XP operating system. Then we move to the folder `/mnt/truplo1/WINDOWS/system32/config` and reset the passwords. First, select option `1` to edit user data and passwords. Then we select the user `dobrota` by specifying his `RID`, which is `03ec`. Now we select the option `1` to reset the password to a blank password and press `q` twice to exit the program and `y` to save the changes.

    apt update
    apt install chntpw

    mount /dev/sdb1 /mnt/truplo1
    cd /mnt/truplo1/WINDOWS/system32/config
    
    chntpw -i SAM system SECURITY

    <>========<> chntpw Main Interactive Menu <>========<>

    Loaded hives: <SAM> <system> <SECURITY>

    1 - Edit user data and passwords
    2 - List groups
        - - -
    9 - Registry editor, now with full write support!
    q - Quit (you will be asked if there is something to save)


    What to do? [1] -> 1


    ===== chntpw Edit User Info & Passwords ====

    | RID -|---------- Username ------------| Admin? |- Lock? --|
    | 01f4 | Administrator                  | ADMIN  |          |
    | 03ec | dobrota                        | ADMIN  |          |
    | 01f5 | Guest                          |        | dis/lock |
    | 03e8 | HelpAssistant                  |        | dis/lock |
    | 03ea | SUPPORT_388945a0               |        | dis/lock |
    | 03eb | user                           | ADMIN  |          |

    Please enter user number (RID) or 0 to exit: [3eb] 03ec
    ================= USER EDIT ====================

    RID     : 1004 [03ec]
    Username: dobrota
    fullname: dobrota
    comment : 
    homedir : 

    00000221 = Users (which has 3 members)
    00000220 = Administrators (which has 3 members)

    Account bits: 0x0210 =
    [ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. | 
    [ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     | 
    [ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   | 
    [X] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  | 
    [ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  | 

    Failed login count: 0, while max tries is: 0
    Total  login count: 2

    - - - - User Edit Menu:
    1 - Clear (blank) user password
    (2 - Unlock and enable user account) [seems unlocked already]
    3 - Promote user (make user an administrator)
    4 - Add user to a group
    5 - Remove user from a group
    q - Quit editing user, back to user select
    Select: [q] > 1
    Password cleared!
    ================= USER EDIT ====================

    RID     : 1004 [03ec]
    Username: dobrota
    fullname: dobrota
    comment : 
    homedir : 

    00000221 = Users (which has 3 members)
    00000220 = Administrators (which has 3 members)

    Account bits: 0x0210 =
    [ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. | 
    [ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     | 
    [ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   | 
    [X] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  | 
    [ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  | 

    Failed login count: 0, while max tries is: 0
    Total  login count: 2
    ** No NT MD4 hash found. This user probably has a BLANK password!
    ** No LANMAN hash found either. Try login with no password!

    - - - - User Edit Menu:
    1 - Clear (blank) user password
    (2 - Unlock and enable user account) [seems unlocked already]
    3 - Promote user (make user an administrator)
    4 - Add user to a group
    5 - Remove user from a group
    q - Quit editing user, back to user select
    Select: [q] > q
        

    <>========<> chntpw Main Interactive Menu <>========<>

    Loaded hives: <SAM> <system> <SECURITY>

    1 - Edit user data and passwords
    2 - List groups
        - - -
    9 - Registry editor, now with full write support!
    q - Quit (you will be asked if there is something to save)


    What to do? [1] -> q

    Hives that have changed:
    #  Name
    0  <SAM>
    Write hive files? (y/n) [n] : y
    0  <SAM> - OK

Now let's try to log in with the `dobrota` user and an empty password.

### 3. Cracking Linux passwords

On Linux operating systems, user information is located in the file [`/etc/shadow`](https://linuxize.com/post/etc-shadow-file/) and only those can be cracked using the [John the Ripper](https://www.openwall.com/john/) tool. Each line specifies one user, namely in the format `UserName:EncryptedPassword:LastPasswordChange:MinimumPasswordAge:MaximumPasswordAge:WarningPeriod:InactivityPeriod:ExpiryPeriod:UnusedField`.

    cat /etc/shadow

    root:$y$j9T$bZLSEsD4PebYHL2ytt2eT.$/VJ8dekN/eEK/RhgNpFCK1tSJGg6USIJtHSQSNT2LI1:19424:0:99999:7:::
    daemon:*:19424:0:99999:7:::
    bin:*:19424:0:99999:7:::
    sys:*:19424:0:99999:7:::
    sync:*:19424:0:99999:7:::
    games:*:19424:0:99999:7:::
    man:*:19424:0:99999:7:::
    lp:*:19424:0:99999:7:::
    mail:*:19424:0:99999:7:::
    news:*:19424:0:99999:7:::
    uucp:*:19424:0:99999:7:::
    proxy:*:19424:0:99999:7:::
    www-data:*:19424:0:99999:7:::
    backup:*:19424:0:99999:7:::
    list:*:19424:0:99999:7:::
    irc:*:19424:0:99999:7:::
    gnats:*:19424:0:99999:7:::
    nobody:*:19424:0:99999:7:::
    _apt:*:19424:0:99999:7:::
    systemd-network:*:19424:0:99999:7:::
    systemd-resolve:*:19424:0:99999:7:::
    tss:*:19424:0:99999:7:::
    messagebus:*:19424:0:99999:7:::
    systemd-timesync:*:19424:0:99999:7:::
    avahi-autoipd:*:19424:0:99999:7:::
    usbmux:*:19424:0:99999:7:::
    rtkit:*:19424:0:99999:7:::
    dnsmasq:*:19424:0:99999:7:::
    avahi:*:19424:0:99999:7:::
    speech-dispatcher:!:19424:0:99999:7:::
    pulse:*:19424:0:99999:7:::
    saned:*:19424:0:99999:7:::
    colord:*:19424:0:99999:7:::
    geoclue:*:19424:0:99999:7:::
    Debian-gdm:*:19424:0:99999:7:::
    aleks:$y$j9T$DJnDTsvzSqd2GDXdvDf/e/$Xvs8zv2WqExAH4W.cedeGXY3f5tJQrsEk99ozl/de94:19424:0:99999:7:::
    systemd-coredump:!*:19424::::::
    vboxadd:!:19424::::::

The password is in the format `$HashingFunctionIdentifier$Salt$HashingFunctionValue`, where the identifier specifies the hashing function that our operating system uses to calculate the hashes for our passwords. All the hash functions we can use are listed [here](https://en.wikipedia.org/wiki/Crypt_(C)#Key_derivation_functions_supported_by_crypt).

Let's install the Linux password cracking tool `john` via the package manager. Now let's save the following examples of users and passwords to a file and try to crack the passwords.

    apt update
    apt install john

    nano unix_passwords

    student:$6$B18NDQmp$0c21HLddW7CxXh0W1S24TN4L.wfVkhqaXtf489ai/mT6n0Ov7osEb5.aKVxKyQ6j.38OMrKXv7AQ9jiUcVimA.:17948:0:99999:7:::
    test:$6$VdV5y2gl$YxpYuwcVZHSXiv0N4yzmF8PspBeIK8QLdGJZzYFuKRjkfc82DhaS5fQeuOt0q9APDPLeSMTzt8BtxI2Bwo/hH.:17948:0:99999:7:::
    john:$6$7CxX6vB2$BS5J1Ch4zwiIG2w/DJ48vwNm7V8iv/PGHBhfZoX9dYX8FqkX2WcbIuGy7W0AXhDme46jpOrM5QI/hFdby3ji40:17982:0:99999:7:::
    sam:$6$kawol/fc$b9zvgpBGB46Ju9YZlvYKbfsOBSzT/4xH.hZUbDRQTnHSn4BBqg1XNDlHmNyfYUPLqlGNF7uceviGc7hOznRCF0:17982:0:99999:7:::
    mike:$6$k51atM5t$drdibMMoXmGGQZuqoC/iXQFHdg6L8oNfGXwsXkVj0.r7RHBgTt0GZBvrQyfvlMqZOBrsUC43v8usx1tRiyfoF.:17982:0:99999:7:::
    bob:$6$ttp5oC25$mVVqVDAxa5ef0yqOcm1Gzp/1VNADPAgLSKhHsATHZI76j.kV/uXxBvxOjxmU3VRcvdPgAwkHY.aAfAJaweKKD/:17982:0:99999:7:::

    john unix_passwords

    Created directory: /root/.john
    Loaded 6 password hashes with 6 different salts (crypt, generic crypt(3) [?/64])
    Will run 2 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    test             (test)
    mike123          (mike)
    jennifer         (bob)
