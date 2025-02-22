## Value cannot be null. Parameter name: value Error
Few days ago I started to get a strange error on my local Sitecore XP. Sitecore website stopped loading. It just kept giving the following error.

```
[ArgumentNullException: Value cannot be null.
Parameter name: value]
System.Collections.CollectionBase.OnValidate(Object value) +14458590
System.Collections.CollectionBase.System.Collections.IList.Add(Object value) +43

```

![](/docs/assets/images/Value%20Cannot%20be%20null.PNG)

More strangely, I had two Sitecore instances locally and both of them started to show this error at the same time. 


I restarted the app pools, did not help. I assigned a new self-signed certificate to my site, it also did not help. Sitecore Identity server was running properly though.

I reached out to Sitecore Support for help. They provided an easy tweak of the IIS server and it resolved the issue. The suggestion was to set "Load User Profile" in the App pool Advanced settings to "True". Probably IIS flagged some of my experiments with the Sitecore Identity Server as suspicious and became cautious to not to load my websites. However, the above change fixed the issue. 

