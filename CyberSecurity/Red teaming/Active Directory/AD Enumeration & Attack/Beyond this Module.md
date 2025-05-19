
---

## Status Update

By the end of the skills assessments, we provided enough access and enumeration results to our senior pentesters to complete their follow-on actions and successfully meet all assessment objectives. Demonstrating our skills has shown the team lead that we are now capable of performing actions for more upcoming assessments dealing with Active Directory environments. He will be providing us with more tasks soon.

---

## Real World

As a Penetration Tester, one could expect the tasks provided in this module to a part of our day-to-day duties. Having a deep understanding of AD and what we can glean from it (access and enumeration-wise) is essential to fulfill the duties of the role. Our actions may often influence the actions of our teammates and senior testers if we are working on an assessment as a team. Those actions could include:

- Taking advantage of cross-domain trusts to infiltrate other domains
- Persistence methods
- Command and Control within the domain for assessments that have longer windows,

With the modern enterprise moving toward hybrid and cloud environments, understanding the foundations within AD and how to abuse them will be extremely helpful when attempting to pivot to these new types of networks. If any of the concepts, terminology, or actions discussed in this module were a bit challenging or confusing, consider going back and checking out the [Introduction To Active Directory](https://academy.hackthebox.com/course/preview/introduction-to-active-directory) module. It contains a deep dive into all things AD and helps lay a foundation of knowledge needed to understand Active Directory.

---

## What's Next?

Check out the [Active Directory BloodHound](https://academy.hackthebox.com/course/preview/active-directory-bloodhound) module to better understand how BloodHound works. Also, check out the [Active Directory LDAP](https://academy.hackthebox.com/course/preview/active-directory-ldap) and [Active Directory PowerView](https://academy.hackthebox.com/course/preview/active-directory-powerview) modules. The [Cracking Passwords with Hashcat](https://academy.hackthebox.com/course/preview/cracking-passwords-with-hashcat) module can also help improve our understanding of the actions we took in the Kerberoasting and Password Spraying sections.

---

## More AD Learning Opportunities

The Hack The Box main platform has many targets for learning and practicing AD enumeration and attacks. The [Intro to Zephyr](https://app.hackthebox.com/tracks/Intro-to-Zephyr) on the main HTB platform is an excellent resource for practice. `Tracks` are curated lists of machines and challenges for users to work through and master a particular topic. The Intro to Zephyr contains boxes of varying difficulties with various attack vectors. Even if you cannot solve these boxes on your own, it is still worth working with them with a walkthrough or video or just watching the video on the box by Ippsec. The more you expose yourself to these topics, the more comfortable and second nature enumeration and many attacks will become. The boxes below are great to practice the skills learned in this module.

#### Boxes To Pwn

- [Forest](https://www.youtube.com/watch?v=H9FcE_FMZio)
- [Active](https://www.youtube.com/watch?v=jUc1J31DNdw)
- [Reel](https://youtu.be/ob9SgtFm6_g)
- [Mantis](https://youtu.be/VVZZgqIyD0Q)
- [Blackfield](https://youtu.be/IfCysW0Od8w)
- [Monteverde](https://youtu.be/HTJjPZvOtJ4)

Ippsec has recorded videos explaining the paths through many of these boxes and more. As a resource, [Ippsec's site](https://ippsec.rocks/?#) is a great resource to search for videos and write-ups pertaining to many different subjects. Check out his videos and write-ups if you get stuck or want a great primer dealing with Active Directory and wish to see how some of the tools work.

---

#### ProLabs

`Pro Labs` are large simulated corporate networks that teach skills applicable to real-life penetration testing engagements. The `Dante` Pro Lab is an excellent place to start with varying vectors and some AD exposure. The `Offshore` Pro Lab is an advanced-level lab that contains a wealth of opportunities for practicing AD enumeration and attacks.

- [Dante](https://app.hackthebox.com/prolabs/overview/dante) Pro Lab
- [Offshore](https://app.hackthebox.com/prolabs/overview/offshore) Pro Lab

Head [HERE](https://app.hackthebox.com/prolabs) to look at all the Pro Labs that HTB has to offer.

#### Endgames

For an extreme challenge that may take you a while to get through, check out the [Ascension](https://app.hackthebox.com/endgames/ascension) Endgames. This endgame features two different AD domains and has plenty of chances to practice our AD enumeration and attacking skills.

![Hack The Box Ascension page showing entry point IP, progress tracker, introduction, and list of machines.](https://academy.hackthebox.com/storage/modules/143/endgame.png)

#### Great Videos to Check Out

[Six Degrees of Domain Admin](https://youtu.be/wP8ZCczC1OU) from `DEFCON 24` is a great watch for an introduction to BloodHound.  
[Designing AD DACL Backdoors](https://youtu.be/_nGpZ1ydzS8) by Will Schroeder and Andy Robbins is a gem if you haven't seen it.[Kicking The Guard Dog of Hades](https://www.youtube.com/watch?v=PUyhlN-E5MU) is one of the original releases for Kerberoasting and is a great watch. In [Kerberoasting 101](https://youtu.be/Jaa2LmZaNeU), Tim Medin does an excellent job dissecting the Kerberoasting attack and how to perform them.

There are so many more, but building a list here would take a whole other section. The videos above are a great start to advancing your AD knowledge.

#### Writers and Blogs To Follow

Between the HTB `Discord`, Forums, and `blogs`, there are plenty of outstanding write-ups to help advance your skills along the way. One to pay attention to would be [0xdf's walkthroughs](https://0xdf.gitlab.io/tags.html#active-directory). These are also a great resource to understand how an Active Directory `attack path` may look in the real world. `0xdf` writes about much more, and his blog is an excellent resource. The list below contains links to other authors and blogs we feel do a great job discussing AD security topics and much more.

[SpecterOps](https://posts.specterops.io/) has an interesting blog where they talk about AD, `BloodHound`, Command and Control, and so much more.  
[Harmj0y](https://blog.harmj0y.net/category/activedirectory/) writes quite a bit about AD, among other things as well. He is someone you should be following if you are looking to work in this industry.  
[AD Security Blog](https://adsecurity.org/?author=2) by Sean Metcalf is a treasure box full of awesome content, all AD and security related. It is a must-read if you are focused on Active Directory.  
[Shenaniganslabs](https://shenaniganslabs.io/) is a great group of security researchers discussing many different topics in the security realm. These can include new vulnerabilities to Threat Actor TTPs.  
[Dirk-jan Mollema](https://dirkjanm.io/) also has a great blog documenting his adventures with AD security, Azure, protocols, vulnerabilities, Python, etc.  
[The DFIR Report](https://thedfirreport.com/) is maintained by a talented team of Blue Teamers/Infosec Content creators that share their findings from recent intrusion incidents in incredible detail. Many of their posts showcase AD attacks and the artifacts that attackers leave behind.

---

## Closing Thoughts

Absorbing everything we can about Active Directory security and becoming familiar with the TTPs utilized by different teams and threat actors will take us a long way. [MITRE's Enterprise Attack Matrix](https://attack.mitre.org/matrices/enterprise/windows/) is a great place to research attacks and their corresponding tools and defenses. AD is a vast topic and will take time to master. New vulnerability vectors and PoC attacks are being released frequently. This topic isn't going anywhere, so use the resources available to stay ahead of the curve and keep networks actively secure. A fundamental understanding of AD and the tools surrounding the field, both as a penetration tester or defender, will keep us up to date. The more we understand the bigger picture, the more powerful we will become as attackers and defenders, and the more value we can provide to our clients and the companies we work for. Improving security is our focus, but nothing says we can't have fun while doing so.