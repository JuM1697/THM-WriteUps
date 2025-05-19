# Volt Typhoon
This Write-up is for the "Volt Typhoon" room of TryHackMe. You can find the room here: https://tryhackme.com/room/volttyphoon  
This is my very first own Write-Up for a THM room, so bear with me.  
I hope it will help you solving this room while also learning and tackling these challenges.  
As a side-note: I have zero to none experience with Splunk. I used mostly super basic string searches which are AND connected with each other and it lead me always to the result.

## Task 1 - IR Scenario
Task 1 does not ask you to do anything besides starting the attached machine and connecting to the Splunk instance that is hosted on it (http://MACHINE_IP:8000). I used the THM Attack-Box to connect to Splunk, however it can also be done via your own system and OpenVPN.  
There was no need for me to login to Splunk, I was already logged in apparently.

## Task 2 - Initial Access
Volt Typhoon often gains initial access to target networks by exploiting vulnerabilities in enterprise software. In recent incidents, Volt Typhoon has been observed leveraging vulnerabilities in Zoho ManageEngine ADSelfService Plus, a popular self-service password management solution used by organizations.

### Question 1
**Comb through the ADSelfService Plus logs to begin retracing the attacker’s steps. At what time (ISO 8601 format) was Dean's password changed and their account taken over by the attacker?**

To answer that question I paid attention to the user that was mentioned ("Dean"). I was hoping that his username was somehow connected to the real one and could be found in the logs.  
I simply queried the logs by searching for "dean".  
And luckily we got quite a few results (note: remember to first search through "All time" data!).
![alt text](image-1.png)
While scrolling through the results, you can quickly learn two thins:
1. the username is not "Dean" it is "dean-admin"
2. There seems to be a specific log source for the "ADSelfService Plus" logs which we can filter for.

With these two things in mind, we build a new query, looking something like this:
```
username="dean-admin" service_name=ADSelfServicePlus
```

This query narrows down all events connected to our target user "Dean" and possible actions connected to the "ADSelfService Plus" service. Scrolling through the results we should spot quickly events that somehow relate to "Password Change" activites. So lets add that to our query and check out the result to answer the first question.

```
username="dean-admin" service_name=ADSelfServicePlus action_name="Password Change"
```

Now our list of results is very short, and our answer easy to spot. There is only one successful "Password Change" event. This is our first answer.
![alt text](image.png)

### Question 2
**Shortly after Dean's account was compromised, the attacker created a new administrator account. What is the name of the new account that was created?**

Now that we know the time of compromise, we should write that down and add it to our time-filter to focus on the events post-compromise.
Once that is done, we will now try to find an event that tells us, that a new user has been created. And since we previously found out, that Volt Typhoon compromised an account, it's safe to say the compromised account "dean-admin" was probably issuing this command.  
So we start with a super simple and basic first query:
```
username="dean-admin" create
```
The good thing: we do find our answer.  
The bad thing: we need to dig for it.  
I did not come up with the idea to additionally add the "useraccount" filter to the query. If I would have though of it, it would have been way easier since only one result will show up.

```
username="dean-admin" create useraccount
```
![alt text](image-2.png)

## Task 3 - Execution
Volt Typhoon is known to exploit Windows Management Instrumentation Command-line (WMIC) for a range of execution techniques. They leverage WMIC for tasks such as gathering information and dumping valuable databases, allowing them to infiltrate and exploit target networks. By using "living off the land" binaries (LOLBins), they blend in with legitimate system activity, making detection more challenging.

### Question 3
**In an information gathering attempt, what command does the attacker run to find information about local drives on server01 & server02?**  
This is where I actually struggled for the first time with my basic searches. When I first used the following query:
```
username="dean-admin" wmic
```
I received 19 pages of results and knew I was a bit far off of finding the answer. So lets check the question again. The adversary wants to find out about local drives on the systems "server01" and "server02". Lets try to add that to our query and see how it affects the result:
```
username="dean-admin" wmic disk server01
```
"No results".  
Bummer.  
What if we remove the "disk" keyword, since I dont know how the event will actually look like until I've seen it, and maybe it's not called disk but something else. And from our previous 19 pages results we can confirm that there are definetely events with "server01" in the so chances are good that this keyword is actually right. So let's try:
```
username="dean-admin" wmic server01
```

And great success! We receive only one result, we can see why our previous query didn't work and we also have our answer.
![alt text](image-3.png)

### Question 4
**The attacker uses ntdsutil to create a copy of the AD database. After moving the file to a web server, the attacker compresses the database. What password does the attacker set on the archive?**  
To get to the answer of this question we actually have to perform at least two queries. First of all: we learned that the attacker uses "ntdsutil" to creat AD database copies. So let's try that first and check-out the results.

```
username="dean-admin" ntdsutil
```
![alt text](image-4.png)
We actually receive only one event, which tells us the name of the AD database copy file. With that knowledge, we create a new query, which includes the previous found-out filename.

```
username="dean-admin" temp.dit
```

Using this query, we receive three matching events. We see the creation event (which we saw earlier already) and two others. If we examine the two other events we notice that one of them seems to copy the file to a public webserver location on server01 using xcopy, and the other one does indeed perform a 7z operation on the file, including a password in the command-line-arguments.
![alt text](image-5.png)

## Task 4 - Persistence
## Task 5 - Defense Evasion
## Task 6 - Credential Access
## Task 7 - Discovery & Lateral Movement
## Task 8 - Collection
## Task 9 - C2 & Cleanup
