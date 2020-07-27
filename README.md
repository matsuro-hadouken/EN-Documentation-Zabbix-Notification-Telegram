## Zabbix-Notification-Telegram Documentation

#### Send alerts from Zabbix to Telegram

##### Source: https://github.com/xxsokolov/Zabbix-Notification-Telegram

[Installation](#Installation)

[Configuration](#Configuration)

#### Key Features
- [x] Send graphs and last values **in one message**
- [x] Flexable configuration of templates in message body
- [x] Transfer data from action in XML format
- [x] Links generation in message body
- [x] Tag generation in message body
- [x] Emoji mapping for status and importance of the event
- [x] Image watermarks
- [x] Generation and updates of cache file (privat, group -> supergroup)

### **Part 1, Backend**

<a name="Installation"><h3>Installation</h2></a>

Zabbix use user`zabbix`by default, this user doesn't have home folder and profile after default installation. For this reason we will setup everything from `root` and then apply required permissions. If you have different setup and and your user have full functional shell enveronment then login as this user `su - <zabbix_user>`

* cd to directory
```bash
cd /usr/lib/zabbix/alertscripts/
```

* Clone latest repository, pay attention on the dot in the end
```bash
git clone https://github.com/xxsokolov/Zabbix-Notification-Telegram.git .
```

* Install virtual enveronment
```bash
sudo apt install virtualenv
```

* Create virtual environment
```bash
virtualenv venv --python=python3
``` 

* Activate virtual environment
```bash
source venv/bin/activate
```

* Install requirements
```bash
pip install -r .requirements
```

* deactivate source
```bash
deactivate
```

* Copy configuration file
```bash
cp zbxTelegram_config.example.py zbxTelegram_config.py
```

* Set permissions ( default user is zabbix by default )
```bash
chown -R zabbix:zabbix /usr/lib/zabbix/alertscripts/.[^.]*
```

* Make main script executable
```bash
chmod +x zbxTelegram.py
```

Follow instruction how to create bot and give bot permissions to send private messages for your user or a group as prefared.

* Edit configuration file
```bash
nano zbxTelegram_config.py
```
 
 
<a name="Configuration"><h3>Configuration</h2></a>


**zbxTelegram_config.py configuration**


`tg_proxy` = Use proxy True/False; default true True

`tg_proxy_server`  = Proxy server address

`tg_token` = Telegram bot token for API access

`zabbix_api_url` = Path to zabbix API (backslash '/' in the end is mandatory) example `zabbix_api_url = http://127.0.0.1/`

`zabbix_api_login` = read-only user with read permissions to specific hosts, see blow in UI configuration section.

`zabbix_api_pass` = read-only user password

`watermark` = Send watermarks True/False; default True

**To test config and API availability**

From console:
```bash
./zbxTelegram.py @username test test
```

@username can be a group ID, like ‘-1092834323’

After testing from console and get 100% positive feedback we need to set permissions one more time:

```bash
chown -R zabbix:zabbix /usr/lib/zabbix/alertscripts/.[^.]*
```

If any errors present fix it, usually need to fix permissions for alertscripts folder.

check script log `/lib/zabbix/alertscripts/zbxTelegram_files/znt.log` and `var/log/zabbix/` for more details.

**Part 2, Zabbix configuration from UI**

**Configuration Media types**

Go to Zabbix front-end, `Administration > Media Type`

_Name_: ZNT

_Type_: Script

_Script name_: zbxTelegram.py

_Script parameters_:

`{ALERT.SENDTO}`

`{ALERT.SUBJECT}`

`{ALERT.MESSAGE}`

<img src="https://user-images.githubusercontent.com/50751381/88594182-4ca40480-d050-11ea-81e5-d1167dcdd982.png">

**Configure user media**

_For alerts we need to create user with read-only permissions. Zabbix allow permissions only trough groups. Create read-only group, then create user and add to read only group. Give read permissions only for required hosts._

Go to: `Administration > Users`

Choose your `read-only user` and go to `Media` tab

<img src="https://user-images.githubusercontent.com/50751381/88594219-5f1e3e00-d050-11ea-97cf-cdda33857af6.png">

Add Telegram identity, group for example, or user name.

<img src="https://user-images.githubusercontent.com/50751381/88594244-66dde280-d050-11ea-9850-9fdb9f881f5d.png">

Now test media type from `Zabbix UI`

<img src="https://user-images.githubusercontent.com/50751381/88594261-6cd3c380-d050-11ea-84af-f1ed664dd1a6.png">

If any errors present fix it, check script log `/lib/zabbix/alertscripts/zbxTelegram_files/znt.log` and `var/log/zabbix/` for more details.

**Setup Actions**

Go to: `Configurations > Action`

Name: Anything you want, “Critical Alerts” for example.

Add required trigger:

<img src="https://user-images.githubusercontent.com/50751381/88594289-75c49500-d050-11ea-9810-d96e697581d4.png">

It is possible to add as much triggers as we need and set the rules:

<img src="https://user-images.githubusercontent.com/50751381/88594306-7ceba300-d050-11ea-8f4c-bf36241cb7ba.png">

Activate Operations tab:

<img src="https://user-images.githubusercontent.com/50751381/88594330-86750b00-d050-11ea-9b56-99b69169ae2b.png">

Here we can setup different actions:

<img src="https://user-images.githubusercontent.com/50751381/88594350-90970980-d050-11ea-9eb1-b99226335dc7.png">

All as example:

<img src="https://user-images.githubusercontent.com/50751381/88594369-9856ae00-d050-11ea-8c20-c256ec65a1bf.png">

Send to User = `your read-only user` ( it is possible  to add more users  )

* Default subjects for `Operations/Recovery/Update`

`{Problem} {TRIGGER.SEVERITY} {{TRIGGER.SEVERITY}}: {EVENT.NAME}`

`{Resolved} {TRIGGER.SEVERITY} {{TRIGGER.SEVERITY}} {EVENT.NAME}`

`{Update} {TRIGGER.SEVERITY} {{TRIGGER.SEVERITY}} {EVENT.NAME}`

Strings structure:

`{Problem}` – mapping of values `(Problem\Resolved\Update)` check the end of `zbxTelegram_config.py` to get more ideas.

<img src="https://user-images.githubusercontent.com/50751381/88595732-11570500-d053-11ea-846f-efb0bcc5a25b.png">

`{{TRIGGER.SEVERITY}}` - mapping of values in Severity, check the end of `zbxTelegram_config.py` to get more ideas.

* Default message

Inside messages we use XML syntax, default example inside _actions.example_ file, please check.

The most simple working solution is just copy example content from _actions.example_ in to `Default message`.

_actions.example:_ https://raw.githubusercontent.com/xxsokolov/Zabbix-Notification-Telegram/master/actions.example

Default message structure contain two sections:

```xml
<body>
   <messages>
      Actual message
   </messages>
</body>
``` 

```xml
<settings> 
       Configuration
</settings>
``` 
<img src="https://user-images.githubusercontent.com/50751381/88594393-a1477f80-d050-11ea-8815-89334d84e682.png">


`<graphs></graphs>` - attach graph (True/False)

`<graphlinks>True</graphlinks>` - attach url to history (True/False)

`<triggerlinks>True</triggerlinks>` - attach link to trigger (True/False)

`<tag>True</tag>` - attach tags (True/False)

`<graphs_period></graphs_period>` graph time frame in seconds

`<itemid></itemid>` - send Item ID {ITEM.ID1}

`<triggerid></triggerid>` send triggers ID {TRIGGER.ID}

`<eventid></eventid>` send event ID  {EVENT.ID}

`<title></title>` graph header {HOST.HOST} - {EVENT.NAME}

`<triggerurl></triggerurl>` - send URL from trigger {TRIGGER.URL}

`<tags></tags>` send tags from trigger {EVENT.TAGS}

* Press update, then activate your action.

#### Alert example:

<img src="https://user-images.githubusercontent.com/50751381/88594444-b91f0380-d050-11ea-8413-f3b70ca3b88d.png">

###### **Telegram group with REAL support:**: https://t.me/ZbxNTg

###### **GitHub source by xxsokolov:** https://github.com/xxsokolov/Zabbix-Notification-Telegram

###### **Zabbix 5.0 triggers:** https://www.zabbix.com/documentation/current/manual/config/triggers/trigger
###### **Zabbix 5.0 actions:** https://www.zabbix.com/documentation/current/manual/config/notifications/action
