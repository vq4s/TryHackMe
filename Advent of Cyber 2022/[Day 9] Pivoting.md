# Day 9: Dock the halls 
### Learning Objectives

*   Using Metasploit modules and Meterpreter to compromise systems
*   Network Pivoting
*   Post exploitation

## What is Docker?

Docker is a way to package applications, and the associated dependencies into a single unit called an image. This image can then be shared and run as a container, either locally as a developer or remotely on a production server. Santa’s web application and database are running in Docker containers, but only the web application is directly available via an exposed port. A common way to tell if a compromised application is running in a Docker container is to verify the existence of a ` /.dockerenv` file at the root directory of the filesystem.

## What is Metasploit?

Metasploit is a powerful penetration testing tool for gaining initial access to systems, performing post-exploitation, and pivoting to other applications and systems. Metasploit is free, open-source software owned by the US-based cybersecurity firm Rapid7.

## What is a Metasploit session?

After successfully exploiting a remote target with a Metasploit module, a session is often opened by default. These sessions are often Command Shells or Meterpreter sessions, which allow for executing commands against the target. It’s also possible to open up other session types in Metasploit, such as SSH or WinRM - which do not require payloads.
        
## What is Meterpreter?

Meterpreter is an advanced payload that provides interactive access to a compromised system. Meterpreter supports running commands on a remote target, including uploading/downloading files and pivoting.
     
Note that normal command shells do not support complex operations such as pivoting. In Metasploit’s console, you can upgrade the last opened Metasploit session to a Meterpreter session with `sessions -u -1`.

You can identify the opened session types with the `sessions` command. If you are currently interacting with a Meterpreter session, you must first `background` it. In the below example, the two session types are `shell cmd/unix` and `meterpreter x86/linux`:
       
## What is Pivoting?

Once an attacker gains initial entry into a system, the compromised machine can be used to send additional web traffic through - allowing previously inaccessible machines to be reached.

For example - an initial foothold could be gained through a web application running in a docker container or through an exposed port on a Windows machine. This system will become the attack launchpad for other systems in the network.

![note1](https://user-images.githubusercontent.com/53142039/208082285-c2c496b1-7c4c-45e8-8f44-3256ecbad0a3.png)

We can route network traffic through this compromised machine to run network scanning tools such as `nmap` or `arp` to find additional machines and services which were previously inaccessible to the pentester. This concept is called network pivoting.

![note2](https://user-images.githubusercontent.com/53142039/208082287-1bbf5558-e934-4a67-bc6e-bf231f4829bd.png)

## Using Meterpreter to pivot

Metasploit has an internal routing table that can be modified with the `route` command. This routing table determines where to send network traffic through, for instance, through a Meterpreter session. This way, we are using Meterpreter to pivot: sending traffic through to other machines on the network.

Note that Meterpreter has a separate route command, which is not the same as the top-level Metasploit prompt's route command described below. If you are currently interacting with a Meterpreter session, you must first `background` it.

## Socks Proxy

A socks proxy is an intermediate server that supports relaying networking traffic between two machines. This tool allows you to implement the technique of pivoting. You can run a socks proxy either locally on a pentester’s machine via Metasploit, or directly on the compromised server.

## Challenge Walkthrough
After deploying the attached VM, run Nmap against the target: 

![1](https://user-images.githubusercontent.com/53142039/208078879-06a29885-8b2a-42ed-8e74-7ff6a7d4019f.png) \
After loading the web application in our browser at http://MACHINE_IP:80 (use Firefox on the Kali web-Machine) and inspecting the Network tab, we can see that the server responds with an HTTP Set-Cookie header indicating that the server is running Laravel - a common web application development framework: 

![2](https://user-images.githubusercontent.com/53142039/208078881-db7472b7-59fb-449b-9aea-7216b6bc544b.png)

The application may be vulnerable to a remote code execution exploit which impacts Laravel applications using debug mode with Laravel versions before 8.4.2, which use ignite as a developer dependency.

We can use Metasploit to verify if the application is vulnerable to this exploit.

Note: be sure to set the HttpClientTimeout=20, or the check may fail. In extreme situations where your connection is really slow/unstable, you may need a value higher than 20 seconds.

![3](https://user-images.githubusercontent.com/53142039/208078888-7a0014a9-06f8-4074-9527-985160758905.png)

To find out what IP address you need to use, you can open up a new terminal and enter `ip addr`. The IP address you need will start with 10.x.x.x. Remember, you will either need to use eth0 or tun0, depending on whether or not you are using the TryHackMe Kali Web-Machine.

![4](https://user-images.githubusercontent.com/53142039/208078890-05e5091b-7f80-4a6c-8bd5-fd2e86f291e6.png)

Now that we’ve confirmed the vulnerability, let’s run the module to open a new session:

![5](https://user-images.githubusercontent.com/53142039/208078892-ccca3b90-638c-4f14-b135-08a5b28edb6d.png)

The opened shell will be a basic `cmd/unix/reverse_bash` shell. We can see this by running the background command and viewing the currently active sessions:

![6](https://user-images.githubusercontent.com/53142039/208078895-4b573e4c-4e09-4ee1-a3e0-f9f4f57bff0f.png)

If you are currently in a session - you can run the background command to go back to the top-level Metasploit prompt. To upgrade the most recently opened session to Meterpreter, use the `sessions -u -1` command. Metasploit will now show two sessions opened - one for the original shell session and another for the new Meterpreter session:

![7](https://user-images.githubusercontent.com/53142039/208078896-78ca2fb9-efc7-4452-a696-2b436f3bc828.png)

After interacting with the Meterpreter session with `sessions -i -1` and exploring the application, we can see there are database credentials available:

![8](https://user-images.githubusercontent.com/53142039/208078901-dc7796c5-68b5-4fd6-9818-c4a2e08510e1.png)

We can use Meterpreter to resolve this remote hostname to an IP address that we can use for attacking purposes:

![GIT](https://user-images.githubusercontent.com/53142039/208080582-b8b1ef7b-f810-4ddc-b39e-696a7da839e6.png)

As this is an internal IP address, it won’t be possible to send traffic to it directly. We can instead leverage the network pivoting support within msfconsole to reach the inaccessible host. To configure the global routing table in msfconsole, ensure you have run the background command from within a Meterpreter session:

![9](https://user-images.githubusercontent.com/53142039/208078902-9cffeeff-3a5e-42c3-ac43-430cf206c222.png)

We can also see, due to the presence of the /.dockerenv file, that we are in a docker container. By default, Docker chooses a hard-coded IP to represent the host machine. We will also add that to our routing table for later scanning:

![10](https://user-images.githubusercontent.com/53142039/208078904-6e0b18a1-f617-4951-8e87-ccf8370951ba.png)

We can print the routing table to verify the configuration settings:

![11](https://user-images.githubusercontent.com/53142039/208078906-87ca1fce-9b7e-4265-a9b4-6f7003e7f6f7.png)

With the previously discovered database credentials and the routing table configured, we can start to run Metasploit modules that target Postgres. Starting with a schema dump, followed by running queries to select information out of the database:

![12](https://user-images.githubusercontent.com/53142039/208078908-c08440f3-0343-471c-b454-8e3a9b3e8136.png)

To further pivot through the private network, we can create a socks proxy within Metasploit:

![13](https://user-images.githubusercontent.com/53142039/208078912-c6333d9a-66bd-424a-a36b-42c023f17a64.png)

This will expose a port on the attacker machine that can be used to run other network tools through, such as `curl` or `proxychains`

![14](https://user-images.githubusercontent.com/53142039/208078914-54ad1032-8010-486e-9633-2cd63faae4a8.png)

With the host scanned, we can see that port 22 is open on the host machine. It also is possible that Santa has re-used his password, and it’s possible to SSH into the host machine from the Docker container to grab the flag:

![15](https://user-images.githubusercontent.com/53142039/208078918-3629a1ef-9419-4e02-a5ef-d020a3b4aaa1.png)
From TryHackMe

## Answers
1. Deploy the attached VM, and wait a few minutes. What ports are open: **`80`**
2. What framework is the web application developed with: **`laravel`**
3. What CVE is the application vulnerable to: **`CVE-2021-3129`**
4. What command can be used to upgrade the last opened session to a Meterpreter session: **`sessions -u -1`**
5. What file indicates a session has been opened within a Docker container: **`/.dockerenv`**
6. What file often contains useful credentials for web applications: **`.env`**
7. What database table contains useful credentials: **`users`**
8. What is Santa's password: **`p4$$w0rd`**
9. What ports are open on the host machine: **`22,80`**
10. What is the root flag: **`THM{47C61A0FA8738BA77308A8A600F88E4B}`**
