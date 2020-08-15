---
title: Fat tails and parachutes
date: "2020-08-15T00:00:00.000Z"
---
This week I finally ran out of luck at work and cleaned up a database unwittingly. I meant not cleaned up in the sense removed unused data, I removed all data. I accidentally deleted a database, lucky for me it was not in a critical path. As luck would have it, this db did have it backups but they are useless because nobody had to use it. All I could do is (internally):

![https://media.giphy.com/media/3orif5NRSEKimzwcg0/giphy.gif](https://media.giphy.com/media/3orif5NRSEKimzwcg0/giphy.gif)

And we end up spending rest of the evening and night recovering the database by piecing data from various sources. The point of this monlogue is not about how we ended up recovering it, which was another mini epic in itself. People would have told this umpteen times, when you have a DB always exercise your muscle on recovery path. But I want to go little deeper and say, place enough guard rails around your most critical systems so that people cant nuke it off easily. It's not about having documentation, its about making to do mistakes insanely hard. Unless someone tries really hard to delete them intentionally, you should not be able to. A famous telugu movie dialog goes like "If we miss taking down protagonist at point 1, we will have more people at point 2 to take him down and if that misses..", you get the point to put simply you need belts and rubbers. Some options you have are at:

- Cloud layer (picking up AWS here, I bet other clouds have similar feature-set)
    - On AWS you can protect your instances in two ways. By either adding scale in protection via ASG stating what the minimum number of hosts to be in the group.
    - Adding termination protection on each individual host that is worth protecting(either manually r through code).
- Terraform/Cloud formation - Infrastructure as code layer
    - If you use something like terraform, you can use [prevent_destroy](https://www.terraform.io/docs/configuration/resources.html#prevent_destroy) flag which will make sure you dont actually delete important nodes on accidental terraform applies.
- Separate storage from compute and have better durability guarantees. If you can tolerate extra latency by going down the route of networked storage like EBS, you should. This not only gives you better disaster recovery story, it helps you in scaling storage and compute with different characteristics. Once you separate your storage, you can have protections on storage itself similar to point 1 and 2 above.
- Snapshots
    - Snapshot your DB every single day if possible more frequent that.
    - Have analytics or observer nodes that boot out of this snapshot every single week or something like that.
- When all else fails, because you need to be prepared for eventuality.

    [https://twitter.com/wolfejosh/status/827966114365861891?s=20](https://twitter.com/wolfejosh/status/827966114365861891?s=20)

    - Make sure you have runbooks in place describing who the impacted teams would be and how to work through recovering a semblance of original data.

In the end hoping I will learn enough from this mistake of mine in designing and coding better systems to help team move fast.