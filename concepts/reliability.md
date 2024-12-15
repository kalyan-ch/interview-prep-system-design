# Reliability

For a software application, the expectations are:

* It performs the function that the users expect.
* It can tolerate users' mistakes or unexpected uses of it
* Its performance is good enough under the expected load and data volume.
* It prevents any unauthorized access and other security risks.

For an app to be reliable, all of these conditions should be met even when "things go wrong" or faults.

* Such apps or systems are called fault-tolerant.
* A fault is different from failure as any component can be faulty if it deviates from its spec.
* It will help to introduce faults deliberately as this will ensure the efficient design of a fault-tolerant system. For example, [Netflix Chaos Monkey](https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116?gi=8ab06b5f6b85).
* Sometimes, prevention of faults is better than tolerating them like in cases of security faults.
* Hardware faults, like RAM faults, Hard Disk crashes, power outages etc., can be handled with redundancy. Hard Disks may be setup in a RAID system, systems may have mutliple power supplies or backup generators.
* However, redundancy isn't alone enough to prevent all faults. Therefore, systems are being designed to tolerate loss of entire machines by using software tolerant mechanisms, like using entire alternate data centers or rolling upgrades.
* Software faults, like bugs are much harder to anticipate and takes longer to fix, like [leap second bug in Linux Kernel](https://www.somebits.com/weblog/tech/bad/leap-second-2012.html) or deadlock bugs with locks on shared resources.
* Thorough testing, process isolation, measuring and monitoring system behavior in production etc. can help fix these bugs.
* Human errors like bad design of systems, configuration errors etc. also cause faults. They can be fixed by designing efficient systems, isolating failure parts of the system, thorough testing in all levels of the system, thorough monitoring, good management practices and training etc.
* Reliability is very important to preserve user data as poorly managed faults may result in huge losses of revenue to a company as well.
