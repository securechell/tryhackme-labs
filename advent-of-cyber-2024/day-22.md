# Day 22: It's because I'm kubed, isn't it?

**Learning Objectives**
- Learn about Kubernetes, what it is and why it is used.
- Learn about DFIR, and the challenges that come with DFIR in an ephemeral environment.
- Learn how DFIR can be done in a Kubernetes environment using log analysis.

### Kubernetes Explained
Back in the day, it was very common for companies/organisations to use a "monolithic architecture" when building their applications. A **monolithic architecture** is an application built as a single unit, a single code base, and usually, a single executable deployed as a single component. For many companies, this worked and still does to this day; however, for some companies, this style of architecture was causing problems, especially when it came to scaling. The problem with monolithic applications is that if one single part of the application needs scaling, the whole application has to be scaled with it. It would make far more sense for companies with applications that receive fluctuating levels of demand across their parts to break the application down component by component and run them as their own microservices. That way, if one "microservice" starts to receive an increase in demand, it can be scaled up rather than the entire application.

#### The Great Microservice Adoption
**Microservices architecture** was adopted by companies like Netflix, which is a perfect example of the hypothetical company discussed above. Their need to scale up services dedicated to streaming when a new title is released (whilst services dedicated to user registration, billing, etc, won't need the same scaling level) made a microservices architecture a no-brainer. As time went by, companies similar to Netflix hopped aboard the Microservices Express, and it became very widely adopted. Now, as for the hosting of these microservices, containers were chosen due to their lightweight nature. Only, as you may imagine, an application of this scale can require hundreds, even thousands of containers. Suddenly, a tool was needed to organise and manage these containers.

#### Introducing Kubernetes
Well, you guessed it! That's exactly what Kubernetes was made for. Kubernetes is a container orchestration system. Imagine one of those microservices mentioned earlier is running in a container, and suddenly, there is an increase in traffic, and this one container can no longer handle all requests. The solution to this problem is to have another container spun up for this microservice and balance the traffic between the two. Kubernetes takes care of this solution for you, "orchestrating" those containers when needed.

That makes things a lot easier for everyone involved, and it's because of this (along with the widespread adoption of microservices architecture) that Kubernetes is so ubiquitous in the digital landscape today. This popularity means that it's **highly portable** as no matter what technology stack is being used, it's very likely a Kubernetes integration is available; this, along with the fact it can help make an application **highly available** and **scalable**, makes Kubernetes a no-brainer!

In Kubernetes, containers run in **pods**; these pods run on **nodes**, and a collection of nodes makes up a Kubernetes **cluster**. It is within a cluster that McSkidy and Co's investigation will occur today. 

### DFIR Basics
Every cyber security professional has stumbled—or will stumble—upon **DFIR (Digital Forensics and Incident Response)** at some point in their career. These two investigative branches of cyber security come into play during a cyber security incident. A DFIR expert will likely be called to action as soon as an incident is ascertained and will be expected to perform actions that fall into one or both of the two disciplines:

- **Digital Forensics**, like any other "forensics" discipline, aims to collect and analyse digital evidence of an incident. The artefacts collected from the affected systems are used to trace the chain of attack and uncover all facts that ultimately led to the incident. DFIR experts sometimes use the term "post-mortem" to indicate that their analysis starts _after_ the incident has occurred and is performed on already compromised systems and networks.
- **Incident Response**, while still relying on data analysis to investigate the incident, focuses on "responsive" actions such as threat containment and system recovery. The incident responder will isolate infected machines, use the data collected during the analysis to identify the "hole" in the infrastructure's security and close it, and then recover the affected systems to a clean, previous-to-compromise state.

Picture the **incident responder** as an emergency first responder whose aim is to contain the damage, extinguish the fire, and find and stabilise all the victims. On the other hand, the **digital forensics analyst** is the Crime Scene Investigator (CSI) or detective trying to recreate the crime scene and ultimately find evidence to identify and frame the criminal.

Both roles are expected to document all findings thoroughly. The incident responder will present them to explain how the incident happened and what can be learnt from it, ultimately proposing changes to improve the security stance of the entity affected by the incident. The digital forensics analyst will use the findings to demonstrate the attackers' actions and—eventually—testify against them in court.

In the task at hand, we will help McSkidy and the Glitch become digital forensics analysts and retrace the malicious actor's steps. We will especially focus on collecting evidence and artefacts to uncover the perpetrator and present our analysis to Wareville townspeople.

### Excruciatingly Ephemeral
DFIR can be a lot of fun. It's easy to feel like a digital detective, analysing the crime scene and connecting the dots to create a narrative string of events explaining what happened. What if the crime scene vanished into thin air moments after the crime was committed? That is a problem we face regularly when carrying out DFIR in a Kubernetes environment. This is because, as mentioned, Kubernetes workloads run in containers. It is **very** common that a container will have a very short lifespan (either spun up to run a job quickly or to handle increased load, etc, before being spun back down again).

So what can we do about it? Well not to worry, it just means we have to expand our digital detectives toolkit. The key to keeping track of the ongoings in your often ephemeral workloads within your Kubernetes environment is increasing **visibility**. There are a few ways we can do this. One way is by enabling Kubernetes audit logging, a function that Kubernetes provides, allowing for requests to the API to be captured at various stages. For example, if a user makes a request to delete a pod, this request can be captured, and while the pod will be deleted (and logs contained within it lost), the request made to delete it will be persisted in the audit logs. What requests/events are captured can be defined with an audit policy. We can use these audit logs to answer questions which help us in a security/DFIR context, such as:

- What happened?
- When did it happen?
- Who initiated it?
- To what did it happen?
- Where was it observed?
- From where was it initiated?
- To where was it going?


### Following the Cookie Crumbs
Let's start our investigation. As mentioned before, some of the log sources would disappear as their sources, like pods, are ephemeral. Let's see this in action first.

1. On the VM, open a terminal as start K8s using the following command: `minikube start`. It will take roughly three minutes for the cluster to configure itself and start.

2. Verify that the cluster is up and running using the following command: `kubectl get pods -n wareville`

![image](https://github.com/user-attachments/assets/2d89970e-a79f-471b-9741-c5e537374194)

3. If all of the pods are up and running (based on their status), you are ready to go. This will take another **2 minutes**. Since we know that the web application was compromised, let's connect to that pod and see if we can recover any logs. Connect to the pod using the following command: `kubectl exec -n wareville naughty-or-nice -it -- /bin/bash`

4. Once connected, let's review the Apache2 access log: `cat /var/log/apache2/access.log`

5. Sadly, we only see logs from the 28th of October when our attack occurred later on. Looking at the last log, however, we do see something interesting with a request being made to a `shelly.php` file. So, this tells us we are on the right track. Terminate your session to the pod using `exit`. 

6. Fortunately, McSkidy knew that the log source was ephemeral and decided to ensure that remote backups of the log source were made. Navigate to our backup directory using `cd /home/ubuntu/dfir_artefacts/` where you will find the access logs stored in `pod_apache2_access.log`. 

![image](https://github.com/user-attachments/assets/7e1b1c61-21ac-489b-bbe1-5b380a3584d2)

7. Review these logs to see what Mayor Malware was up to on the website and answer the first 3 questions at the bottom of the task!

Sadly, our investigation hits a bit of a brick wall here. Firstly, because the pod was configured using a port forward, we don't see the actual IP that was used to connect to the instance. Also, we still don't fully understand how the webshell found its way into the pod. However, we rebooted the cluster and the webshell was present, meaning it must live within the actual image of the pod itself! That means we need to investigate the docker image registry itself. 

8. View the registry container ID by running the following command: `docker ps`

![image](https://github.com/user-attachments/assets/4799de9e-b34c-465d-9a08-c72ae8645f08)

9. Now, let's connect to the instance to see if we have any logs: `docker exec f7e68e2f4e40 ls -al /var/log`

![image](https://github.com/user-attachments/assets/6f6770e2-390f-4dce-8ad6-2210e20ed598)

10. Again, we hit a wall since we don't have any registry logs. Luckily, docker itself would keep logs for us. Let's pull these logs using the following: `docker logs f7e68e2f4e40`

![image](https://github.com/user-attachments/assets/5d3b52d0-e1f7-42fb-9812-ce328cb87533)

11. Now we have something we can use! These logs have been pulled for you and are stored in the `/home/ubuntu/dfir_artefacts/docker-registry-logs.log` file. Let's start by seeing all the different connections that were made to the registry by searching for the HEAD HTTP request code and restricting it down to only the first item, which is the IP
	1. In the terminal `cd` to `/home/ubuntu/dfir_artefacts/`
	2. Then type `cat docker-registry-logs.log | grep "HEAD" | cut -d ' ' -f 1`

![image](https://github.com/user-attachments/assets/a6222dac-7004-47ea-af16-0dcdad5cda13)

12. Here we can see that most of the connections to our registry was made from the expected IP of 172.17.0.1, however, we can see that connections were also made by 10.10.130.253, which is not an IP known to us. Let's find all of the requests made by this IP:

![image](https://github.com/user-attachments/assets/23a6c6ca-4791-4290-ade7-9fcba3992c39)

13. Now, we are getting somewhere. If we review the first few requests, we can see that several authentication attempts were made. But, we can also see that the request to read the manifest for the wishlistweb image succeeded, as the HTTP status code of 200 is returned in this log entry: `10.10.130.253 - - [29/Oct/2024:12:26:40 +0000] "GET /v2/wishlistweb/manifests/latest HTTP/1.1" 200 6366 "" "docker/19.03.12 go/go1.13.10 git-commit/48a66213fe kernel/4.15.0-213-generic os/linux arch/amd64 UpstreamClient(Docker-Client/19.03.12 \\(linux\\))"`

What we also notice is the User Agent in the request is docker, meaning this was a request made through the docker CLI to pull the image. This is confirmed as we see several requests then to download the image. From this, we learn several things:

- The docker CLI application was used to connect to the registry.
- Connections came from 10.10.130.253, which is unexpected since we only upload images from 172.17.0.1.
- The client was authenticated, which allowed the image to be pulled. This means that whoever made the request had access to credentials.

If they had access to credentials to pull an image, the same credentials might have allowed them to also push a new image.  

14. We can verify this by narrowing our search to any PATCH HTTP methods. The PATCH method is used to update docker images in a registry:

![image](https://github.com/user-attachments/assets/dc5b2f87-63b5-4183-b771-e8bbf667587b)

This is not good! It means that Mayor Malware could push a new version of our image! This would explain how the webshell made its way into the image, since Mayor Malware pulled the image, made malicious updates, and then pushed this compromised image back to the registry! Use the information to answer questions 4 through 6 at the bottom of the task. 

Now that we know Mayor Malware had access to the credentials of the docker registry, we need to learn how he could have gained access to them. We use these credentials in our Kubernetes cluster to read the image from the registry, so let's see what could have happened to disclose them!

Okay, so it looks like the attack happened via an authenticated docker registry push. Now, it's time to return to our Kubernetes environment and determine how this was possible. 

McSkidy was made aware that Mayor Malware was given user access to the naughty or nice Kubernetes environment but was assured by the DevSecOps team that he wouldn't have sufficient permissions to view secrets, etc. 

15. The first thing we should do is make sure this is the case. To do this, McSkidy decides to check what role was assigned to the mayor. She first checks the rolebindings (binds a role to a user):

![image](https://github.com/user-attachments/assets/a910086f-d1ab-46b4-b226-cd6e65eed637)

16. McSkidy then sees a rolebinding named after Mayor Malware and decides to take a closer look:

![image](https://github.com/user-attachments/assets/4d534f53-47c0-4804-8581-dc336fac9f63)

17. From the output, she could see that there is a role "mayor-user" that is bound to the user "mayor-malware". McSkidy then checked this role to see what permissions it has (and therefore Mayor Malware had):

![image](https://github.com/user-attachments/assets/3c12266b-28bc-40ec-823d-1586233bfeaa)

The output here tells McSkidy something very important. A lot of the permissions listed here are as you would expect for a non-admin user in a Kubernetes environment, all of those except for the permissions associated with "pods/exec". Exec allows the user to shell into the containers running within a pod. This gives McSkidy an idea of what Mayor Malware might have done. 

18. To confirm her suspicious, McSkidy checks the audit logs for Mayor Malware's activity: `cat audit.log | grep --color=always '"user":{"username":"mayor-malware"' | grep --color=always '"resource"' | grep --color=always '"verb"'`

![image](https://github.com/user-attachments/assets/ccc2c5a8-6480-4936-a0c2-a7669d4e0750)

This returns a lot of logs, let's go through them as McSkidy starts to form the attack path taken by Mayor Malware.

**Get Secrets:**
This log snippet tells us that Mayor Malware attempted to get the secrets stored on the cluster but received a 403 response as he didn't have sufficient permissions to do so (Note: a plural get command runs a list on the backend, and is why it appears as so in the logs).

**Get Roles**:
After being denied secret access, Mayor Malware then started snooping to see what roles were present on the cluster.

**Describe Role**:
Whilst running the previous "get roles" command, Mayor Malware will have found a role named "job-runner". These logs tell us that Mayor Malware then described this role, which would have given him key pieces of information regarding the role. Most importantly for our investigation, it would have told him this role has secret read access.

**Get Rolebindings**:
Now, knowing this role can view secrets, Mayor Malware tried to find its role binding to see what was using this role.

**Describe Rolebinding**:
After seeing a role binding named "job-runner-binding", Mayor Malware described it and found out this role is bound to a service account named "job-runner-sa" (aka this service account has permission to view secrets)

**Get Pods**:
Here, we can see that Mayor Malware, now armed with the knowledge that a service account has the permissions he needs, lists all of the pods running in the Wareville namespace with a kubectl get pods command.

**Describe Pod**:
Mayor Malware describes the pod as a "morality-checker" he then would have found out that this pod runs with the job-runner-sa service account attached. Meaning that if he were able to gain access to this pod, he would be able to gain secret read access.

**Exec**:
As mentioned in the role discussion, exec is permission usually not included in a non-admin role. It is for this exact reason that this is the case; McSkidy feels confident that the DevSecOps team had overly permissive Role-Based Access Control (RBAC) in place in the Kubernetes environment, and it was this that allowed Mayor Malware to run an exec command (as captured by the logs above) and gain shell access into morality-checker.

Mayor Malware is able to now run "get" commands on secrets to list them, but most importantly, we can see he has indeed been able to escalate his privileges and gain access to the "pull-creds" secret using the job-runner-sa service account.

19. The final piece of the puzzle revolved around this secret. Finally, McSkidy runs the command, and the attack path is confirmed:

![image](https://github.com/user-attachments/assets/be48d07a-bf57-4bee-bf98-51563ef628d7)

Shaking her head, McSkidy then confirms that the docker registry pull password is the same as the push password. This means that after retrieving these credentials, Mayor Malware would have been able to make the docker registry push we saw earlier and ensure his malicious web shell was deployed into the Kubernetes environment and gain persistence. It is for this reason that push and pull credentials should always be different. With that, the investigation is all tied up, the conclusion being that Mayor Malware most certainly belongs on the naughty list this year!


### Answers
1. What is the name of the webshell that was used by Mayor Malware? **shelly.php**
	- We can see commands coming from shelly.php, such as `whoami` (shown below), `ls`, `cat`

![image](https://github.com/user-attachments/assets/c8b08191-3757-422e-871f-22a41c4e63b1)

2. What file did Mayor Malware read from the pod? **db.php**
	- We can see the command `cat+db.php`. He was `cat`-ing that file, so that's the file he wants to see the contents of.

![image](https://github.com/user-attachments/assets/e958b736-ed0c-41df-8aee-0cacb1ab55da)

3. What tool did Mayor Malware search for that could be used to create a remote connection from the pod? **nc**
	- He typed `which+nc` so he's looking for "nc"

![image](https://github.com/user-attachments/assets/5c42903e-2831-432d-8bb4-142df03f6971)

4. What IP connected to the docker registry that was unexpected? **10.10.130.253**

5. At what time is the first connection made from this IP to the docker registry? **29/Oct/2024:10:06:33 +0000**

![image](https://github.com/user-attachments/assets/d3155f01-e5d9-4392-bfc7-abae825ba9e2)

6. At what time is the updated malicious image pushed to the registry? **29/Oct/2024:12:34:28 +0000**

![image](https://github.com/user-attachments/assets/75648fbb-f5e5-4c86-96c4-8bbdf98b911a)

7. What is the value stored in the "pull-creds" secret?
`{"auths":{"http://docker-registry.nicetown.loc:5000":{"username":"mr.nice","password":"Mr.N4ughty","auth":"bXIubmljZTpNci5ONHVnaHR5"}}}`

![image](https://github.com/user-attachments/assets/da017c28-915b-49f5-bd61-2d281a6e77ee)
