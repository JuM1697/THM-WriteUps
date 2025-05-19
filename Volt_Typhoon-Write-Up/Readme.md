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
## Task 4 - Persistence
## Task 5 - Defense Evasion
## Task 6 - Credential Access
## Task 7 - Discovery & Lateral Movement
## Task 8 - Collection
## Task 9 - C2 & Cleanup
