# Self-Host Bitwarden on Fly.io almost for free!

This README serves as a rough guide to self-host Bitwarden on fly.io, using the official unified Docker image, which is although not as resource efficient as Vaultwarden, but it sure has more feature parity of course, but also more correctness (sometimes Vaultwarden is gimmicky and nobody has the time to find out why), and most importantly, more security over the code safety, since Bitwarden has security audit team for their apps. The unified image usually took 3-4x resources than Vaultwarden, but that's still plenty of resources for fly.io hobby plan and free allowances (3xCPU and 768MB RAM in total).

The goal for this guide is to have a reliably self-hosted Bitwarden, and the total monthly cost should be less than $5 (less than your daily lunch!).

## Why though? Aren't you aware they have a managed cloud version that you don't need to hassle much with? 

Controls! My data, my rule. My mistakes, my responsibility.

Of course I do know Bitwarden offers subscription service, and in fact I do pay them for that and didn't use their service purposefully as a gratitude of appreciation for their works in open source and C#, but I simply feel unsafe storing data over their managed services, so I reinterpreted this to serve as a (yearly) donation. 

I know they are doing well financially and they don't need to accept any donations, but well, this is a great open source software and seeing the great LastPass leak and 1Password edging on the verge, and Bitwarden (as a whole, the managed service, can be another story) still has yet to be breached, that just shows how open source projects can be better than proprietary while also being safer in general. 

Open-sourcing your app is like making your house an exhibition: we have all eyes on it. Everyone can look at the source code, so maybe you can easily spot some critical bugs, not telling anybody, sell it to Zerodium, but they usually are also found by the others around the same time, likely independently. 

For security exploits, regardless of open source or proprietary project or not, in the end, it still depends on your goodwill to submit the exploit, __but we just need one person to stand out for that__. And if there are so many eyes for open source, chances are there are at least one person to stand out _eventually_. 

So in an essence, open source security is inadvertently an instance of [Kerckhoffs's principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle), and it is indeed very contrary to the public's reception on security, almost close to being an antithesis, whom will likely perceive security as a "don't know my private stuffs" kind of philosophy, and naturally incline to support _security through obscurity_. That's written in the DNA of all humans, including myself sometimes.

I have been self hosting open source offline key managers myself for this specific reason, until I found Bitwarden, and while I fully support them, I don't want to use their managed service, because it can introduce side channels that bad actors can indirectly get the Keys to the Kingdom, for example through social engineering their cloud management employees and gave them administrator access covertly if they are successful, and instill as APT and eventually a moment and got the unencrypted data through a fat finger misinput and leaked the transit through logs or internal APIs. 

There are simply too many factors to count in for their managed service, and Bitwarden, as a business, obviously wouldn't tell how their Azure services are set up. With fly.io, at least I get to deploy it myself and I take full responsibility for all the mistakes.

Technically speaking, this guide can be made platform-agonistic. You can simply swap fly.io with Google Cloud Run, Azure App Services, or your own Heroku alternatives based on Kubernetes. But we use fly.io here because 1. it's a splitting image of Heroku which I missed for a long time since my student days and 2. offers a pretty nice free allowance, albeit not that generous consider that they have to secure their business goal first (compare it with Oracle Cloud free tier, and you will clearly see the differences, but it makes sense for a startup like fly.io not to offer so many goodies), and 3. it uses OCI images (colloquially understood as Docker images) which integrates with the unified image well.

Anyway, we assume fly.io is _good enough_ for us here.

## 1. App setup

Just call `fly launch --no-deploy` on this repo and you will be guided for app creation.

## 2. Secrets setup

Go to https://bitwarden.com/host/ for a pair of installation ID and key. 
Those are going to be BW_INSTALLATION_ID and BW_INSTALLATION_KEY in the environment variables lately as a secret

(Optional) Prepare the following secrets too for the email feature
1. globalSettings__mail__replyToEmail -> The email to reply to Bitwarden, so this is usually "noreply@..." some sort
2. globalSettings__mail__smtp__host -> SMTP server hostname/IP, I use my personal Gmail for instance 
3. globalSettings__mail__smtp__password -> SMTP password, for Gmail it is app specific password
4. globalSettings__mail__smtp__port -> SMTP port, 465 for SSL and 587 for TLS
5. globalSettings__mail__smtp__ssl -> This one determines whether to connect the email service using SSL or TLS, should be derived from the port number but I like to explicitly set false 
6. globalSettings__mail__smtp__username -> SMTP username, here it is my Gmail account 
7. adminSettings__admins -> The list of emails that are hardcoded to be considered side-wide administrator, separated by a comma

Use `fly secrets set BW_INSTALLATION_ID=...` and `fly secrets set BW_INSTALLATION_KEY=...` for the bare minimum

## 3. Volume setup

Technically this step is not needed for a brand new app launch since this should be automatically provisioned, but as a precaution we should ensure the existence of the volume itself.
Run `fly volumes create bitwarden_data -r <app region>` (any name you wanted, and the region must match the app). If you want it free, have at most 3GB storage. 
Bitwarden shouldn't use that much storage anyway with sqlite.

After that use `fly volumes list` the confirm the volume is created.

## 4. Deploy

Run `fly deploy` and you will see the magic! After a while, go to your app hostname and get started with Bitwarden. 

## (Optional) 5. App scaling

The bare minimum VM setup probably won't serve Bitwarden well in the long run, so we need to scale it. We assumed sqlite to be the data backend, so we are limited to vertical scaling. 
You can just run `fly scale vm shared-cpu-2x` for a good experience with Bitwarden already.
You can further scale the RAM size up for a better experience by specifying the additional memory size, for example `fly scale vm shared-cpu-2x --vm-memory=768`.
Keep in mind this is the absolute number that must be greater than the preset default, as it will be later calculated as a relative additional memory over the preset.
(Example: I used `shared-cpu-2x` for my app and it has 512MB RAM bare minimum, so if I set `--vm-memory=768`, I would have an additional 768-512=256MB RAM billed over the preset `shared-cpu-2x`)

Recommendation for personal use: Just `shared-cpu-2x` and 768MB RAM. It should technically fit in the monthly free allowance and so that this should theoretically be free forever, until fly.io decided to cut free allowances like Heroku or worse, it went under as a business (which likely won't right now considering they have quite a good sum of VC money but on a long term, not so sure).

Recommendation for multiple share such as family members or company service: Use `shared-cpu-4x` with at least 2GB of RAM since there will be a lot more connections from the mobile app and the web extensions. That could cost a dime but at this point if you are still being greedy, you're _pathetic_.

## 6. Finals

Keep daily backups. Have a contingency plan when shit hits the fan.

## TODO

1. LiteFS seems to be supporting sqlite as a geo-replicated and distributed service, but it is too betaish and error prone to setup correctly