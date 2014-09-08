<figure>
<img title="hilbert map of hashing algorithms" src="https://konklone.com/assets/images/blog/sha-1/hashing-650.png" />
<figcaption>Hilbert map of hashing algorithms, <a href="http://programmers.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed">by Ian Boyd</a></figcaption>
</figure>

Most of the secure web is using an insecure algorithm, and Google's just declared it to be a slow-motion emergency.

Something like [90% of websites](http://news.netcraft.com/archives/2014/05/05/sha-2-very-cryptographic-so-secure-such-growth-wow.html) that use SSL encryption — <img style="margin-bottom: -3px" src="https://konklone.com/assets/images/blog/sha-1/green-lock.png" title="green lock" /> — use an algorithm called [SHA-1](https://en.wikipedia.org/wiki/SHA-1) to protect themselves from being impersonated. This guarantees that when you go to <img style="margin-bottom: -6px" src="https://konklone.com/assets/images/blog/sha-1/facebook-green.png" title="green lock for facebook.com" />, you're visiting the real Facebook and not giving your password to an attacker.

Unfortunately, [**SHA-1 is dangerously weak**](http://arstechnica.com/security/2012/10/sha1-crypto-algorithm-could-fall-by-2018/), and has been for a [long](https://www.schneier.com/blog/archives/2005/02/sha1_broken.html) [time](https://www.schneier.com/blog/archives/2005/02/cryptanalysis_o.html). It gets weaker every year, but remains widely used on the internet. Its replacement, [SHA-2](https://en.wikipedia.org/wiki/SHA-2), is strong and supported just about everywhere.

Google [recently announced](http://googleonlinesecurity.blogspot.com/2014/09/gradually-sunsetting-sha-1.html) that if you use Chrome, then you're about to start seeing a progression of warnings for many secure websites:

<figure>
<a href="http://googleonlinesecurity.blogspot.com/2014/09/gradually-sunsetting-sha-1.html" target="_blank"><img class="border" title="browser warning progression" src="https://konklone.com/assets/images/blog/sha-1/chrome-warnings.png"></a>
<figcaption>
What's <a href="http://googleonlinesecurity.blogspot.com/2014/09/gradually-sunsetting-sha-1.html">about to befall websites</a> with SHA-1 certificates that expire in 2017, in Chrome.
</figcaption>
</figure>

The first set of warnings will hit before Christmas, and will keep getting more stern over the next 6 months. Eventually, even sites with SHA-1 certificates expiring in 2016 will be given yellow warnings.

By rolling out a staged set of warnings, Google is declaring a slow-motion emergency, and hurrying people to update their websites before things get worse. That's a good thing, because **SHA-1 has got to go**, and no one else is taking it as seriously as it deserves.

If you run a website that uses SSL, you can test your website using a [small SHA-1 testing tool](https://shaaaaaaaaaaaaa.com) I built that will tell you [what you need to do](#what-you-can-do).

Even if you don't, I encourage you to read on. In the rest of this post, I'll cover [**how SSL and SHA-1 work together**](#sha-what?) on the web, [**why it's as urgent**](#an-attack-on-sha-1-feels-plenty-viable-to-me) as Google says it is, and [**what web browsers are doing**](#what-browsers-are-doing).

As importantly, the security community needs to [**make changing certificates a lot less painful**](#changing-certificates-shouldn't-be-this-hard), because security upgrades to the web shouldn't have to feel like an emergency.

## SHA-what?

To understand why replacing SHA-1 is so important, you have to put yourself in a browser's shoes.

When you show up to a website using <img style="margin-bottom: -3px" src="https://konklone.com/assets/images/blog/sha-1/green-lock.png" title="green lock" />, the website presents a file — an SSL "certificate" — to your browser. This certificate is used to do two things: encrypt your connection to the website, and verify that you've connected to the real website.

Any certificate can be used to encrypt your connection. But to verify that you're at the real Facebook, your browser has to have a way to decide whether to trust that certificate, and show you a green lock.

Your browser does this by figuring out whether the website's certificate file has been issued by a "Certificate Authority" (CA). CAs generally charge money to give website owners this file. Your browser trusts over 50 Certificate Authorities to create and vouch for certificates, ranging from Verisign to GoDaddy to the US Department of Defense — and the [many hundreds of intermediary CAs](https://www.eff.org/files/colour_map_of_CAs.pdf) to whom those 50+ have delegated trust. As you might guess, this is [a **very** flawed system](https://konklone.com/post/certificate-authorities-are-actually-a-tremendous-problem), but it's the system that we have right now.

This website's Certificate Authority, for the time being, is [Comodo](http://www.comodo.com), purchased through [Namecheap](https://www.namecheap.com/security/ssl-certificates/domain-validation.aspx).

<figure>
<img title="screengrab of certificate information for konklone.com" class="border block" src="https://konklone.com/assets/images/blog/sha-1/certificate-connection.png" />
<figcaption>What you see in Chrome if you click the green lock for this site.</figcaption>
</figure>

So another crucial test your browser demands, when a website's SSL certificate claims to be issued for that website by a particular Certificate Authority, is: **can the certificate prove it?**

Wherever practical, the internet proves things through math. When a certificate is issued, the Certificate Authority includes this proof by cryptographically "signing" the certificate using a private key, in a way only the real CA could do and that browsers can verify.

But the CA doesn't actually sign the raw certificate: it first condenses the certificate into a unique slug by running it through a "one-way hash" algorithm, like MD5, SHA-1, or SHA-256.

<figure>
<img title="screengrab of more certificate information for konklone.com" class="border block" src="https://konklone.com/assets/images/blog/sha-1/certificate-detail.png" />
<figcaption>An excerpt of what you see in Chrome if you ask for more certificate information.</figcaption>
</figure>

Using a one-way hash keeps things small: for example, running [this exact 3.2MB version of War And Peace](/assets/images/blog/sha-1/war-and-peace.txt) through SHA-1 gives you `baeb2c3a70c85d44947c1b92b448655273ce22bb`.

<figure>
<a href="https://md5file.com/calculator" target="_blank"><img title="War and Peace run through several one-way hash algorithms" class="border" src="https://konklone.com/assets/images/blog/sha-1/war-and-peace-results.png" /></a>
<figcaption>Calculated using <a href="https://md5file.com/calculator">MD5file.com</a>, a pleasant online hash calculator.</figcaption>
</figure>

One-way hash algorithms like SHA-1 are designed to produce unique, irreversible slugs. You should not be able to take `baeb2c3a70c85d44947c1b92b448655273ce22bb` and work backwards and produce War And Peace. As importantly, **no other file should produce that same slug** — even changing one little period should cause the SHA-1 result to change so drastically as to be unrelated. 

If two files are discovered to produce the same slug when run through a one-way hash like SHA-1, that's called a "collision". Collisions are always theoretically possible, but they're supposed to be so rare as to be practically impossible. 

When your browser sees a certificate, it can calculate the SHA-1 for that certificate's information itself, and then compare it to the signed SHA-1 that the certificate offered as proof. Because SHA-1 promises unique slugs, the browser trusts that if they match, the certificate on offer is the same one the Certificate Authority signed.

If you could engineer a certificate that "collides" with a target certificate, and coax a Certificate Authority to issue you that certificate, then you would successfully forge a certificate that a browser would find indistinguishable from the target.

<div class="callout">
<strong>Gritty details:</strong> If you'd like to get into the details of signature algorithms and SSL certificates, Joshua Davies has an <a href="http://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art012">extremely detailed technical explainer</a>.
</div>

## An attack on SHA-1 feels plenty viable to me

In 2005, cryptographers proved that [SHA-1 could be cracked 2,000 times faster than predicted](https://www.schneier.com/blog/archives/2005/02/cryptanalysis_o.html). It would still be hard and expensive — but since computers always get faster and cheaper, it was time for the internet to stop using SHA-1.

Then the internet just kept using SHA-1. In 2012, Jesse Walker wrote an estimate, reprinted by Bruce Schneier, of [the cost to forge a SHA-1 certificate](https://www.schneier.com/blog/archives/2012/10/when_will_we_se.html). The estimate uses Amazon Web Services pricing and Moore's Law as a baseline.

Walker's estimate suggested then that a SHA-1 collision would cost **$2M** in 2012, **$700K** in 2015, **$173K** in 2018, and **$43K** in 2021. Based on these numbers, Schneier suggested that an "organized crime syndicate" would be able to forge a certificate in 2018, and that a university could do it in 2021.

Walker's estimates and Schneier's characterization have become [widely cited](http://arstechnica.com/security/2012/10/sha1-crypto-algorithm-could-fall-by-2018/) in the planning and debate over transitioning from SHA-1. A group of leading Certificate Authorities, the [CA Security Council](https://en.wikipedia.org/wiki/Certificate_Authority_Security_Council), cited them recently to [complain about Google's schedule](https://casecurity.org/2014/08/28/google-plans-to-deprecate-sha-1-certificates/). In that complaint, the CAs use those estimates to suggest "the lack of a practical attack until 2018".

I find this to be an absolutely cartoonish stance for the CA Security Council to take in 2014. They are clearly rationalizing the issue, based on how inconvenient they think a more rapid transition would be.

Walker and Schneier's estimates were pre-Snowden, before the public properly understood that [governments are adversaries too](http://www.nytimes.com/interactive/2013/09/05/us/documents-reveal-nsa-campaign-against-encryption.html?_r=0). By their estimates, a forged certificate in 2014 would cost less than $2 million, a sum that many determined actors in this world would pay.

How do we know that they would? **Because they have.**

<figure>
<a href="http://securelist.com/blog/incidents/34344/the-flame-questions-and-answers-51/" target="_blank"><img title="Flame-infected computers, by Kaspersky Labs" src="https://konklone.com/assets/images/blog/sha-1/kaspersky-flame.png" /></a>
<figcaption>Computers infected with Flame, <a href="http://securelist.com/blog/incidents/34344/the-flame-questions-and-answers-51/">as measured by Kaspersky Labs</a></figcaption>
</figure>

In 2012, researchers uncovered a malware known as [Flame](https://en.wikipedia.org/wiki/Flame_(malware)). The [Washington Post reported](http://www.washingtonpost.com/world/national-security/us-israel-developed-computer-virus-to-slow-iranian-nuclear-efforts-officials-say/2012/06/19/gJQA6xBPoV_story.html) that this was a US-Israel collaboration to obtain intelligence from Iran to stall their nuclear weapons program, and a [leaked NSA document](https://s3.amazonaws.com/s3.documentcloud.org/documents/1150433/lobban-nsa-visit-precis.pdf#page=3) seems to confirm this.

Flame relied on an SSL certificate forged by engineering a collision with SHA-1's predecessor, MD5. Disturbingly, it used a method that was not publicly known at the time, despite years of concerted research on MD5. This was a reminder that we should assume that the worst vulnerabilities go undisclosed.

And it's a funny story about MD5, because, like SHA-1, it was discovered to be breakably weak a very long time ago, and then, like SHA-1, it took a horrifying number of years to rid the internet of it.

MD5 was first shown to be theoretically weak in 1995, and over time was shown to be even weaker. But MD5 was still used by some Certificate Authorities until 2008, when researchers [successfully engineered a collision](http://www.win.tue.nl/hashclash/rogue-ca/) and got a forged certificate issued. That certificate, issued to "MD5 Collisions, Inc." has been [immortalized inside your browser](http://www.csoonline.com/article/2136863/core-javasl-hackers-immortalized-by-firefox/core-java/ssl-hackers-immortalized-by-firefox.html) so that it can be specifically blacklisted.

<figure>
<img title="screengrab of MD5 Collisions Inc. certificate in Chrome" class="border" src="https://konklone.com/assets/images/blog/sha-1/md5-collisions-inc.png" />
<figcaption>In Chrome, you can see these by visiting <a href="chrome://settings/certificates">chrome://settings/certificates</a>.</figcaption>
</figure>

That was an emergency, and yet Chrome wasn't able to remove support for MD5 [until 2011](https://code.google.com/p/chromium/issues/detail?id=101123) — **16 years** after MD5 was first shown to be untrustworthy.

That's because there's a particular challenge with updating signature algorithms on the internet today: as long as browsers need to support SHA-1 for someone, [anyone's certificate can be forged with it](https://github.com/konklone/shaaaaaaaaaaaaa/issues/25). In other words, it's not enough for "lots" of sites to upgrade: like a tumor, **you have to get rid of it all**, so that support for the algorithm can be removed entirely.

## What browsers are doing

Microsoft was the first to announce a [deprecation plan for SHA-1](http://blogs.technet.com/b/pki/archive/2013/11/12/sha1-deprecation-policy.aspx), where Windows and Internet Explorer will distrust SHA-1 certificates after 2016. Mozilla has [decided on the same thing](https://wiki.mozilla.org/CA:Problematic_Practices#SHA-1_Certificates). Neither Microsoft nor Mozilla have indicated they plan to change their user interface in the interim to suggest to the user that there's a problem.

Google, on the other hand, recently dropped a truth bomb by [announcing that Chrome would show warnings to the user right away](http://googleonlinesecurity.blogspot.com/2014/09/gradually-sunsetting-sha-1.html), because SHA-1 is just too weak:

> We plan to surface, in the HTTPS security indicator in Chrome, the fact that SHA-1 does not meet its design guarantee. We are taking a measured approach, gradually ratcheting down the security indicator and gradually moving the timetable up.

Google's Ryan Sleevi had first announced Chrome's intended policy a couple weeks before, and I strongly recommend [reading the full discussion](https://groups.google.com/a/chromium.org/d/msg/security-dev/2-R4XziFc7A/NDI8cOwMGRQJ) — you will get to see many Certificate Authorities and large site operators show up and nervously attempt to reason with this wild-eyed man who appears to be telling them that they should stop issuing weak certificates not late next year, but **right now**.

This is a gutsy move by Google, and represents substantial risk. One major reason why it's been so hard for browsers to move away from signature algorithms is that when browsers tell a user an important site is broken, the user believes the browser is broken and switches browsers. Google seems to be betting that Chrome is trusted enough for its security and liked enough by its users that they can withstand the first mover disadvantage.

Opera has also [backed Google's plan](https://groups.google.com/a/chromium.org/d/msg/security-dev/2-R4XziFc7A/-DyHwJRR7-AJ). The Safari team is [watching developments](https://cabforum.org/pipermail/public/2014-August/003769.html) and hasn't announced anything.

## What you can do

To help with the transition, I've built a small website at [**shaaaaaaaaaaaaa.com**](https://shaaaaaaaaaaaaa.com) that checks whether your site is using SHA-1 and needs to be updated:

<figure>
<a href="https://shaaaaaaaaaaaaa.com" target="_blank"><img title="SHAAAAAAAAAAAAA" class="border" src="https://konklone.com/assets/images/blog/sha-1/shaaaaa-site.png" /></a>
<figcaption>It's thirteen A's.</figcaption>
</figure>

Requesting a new certificate is usually very simple. You'll need to [generate a new certificate request](https://konklone.com/post/switch-to-https-now-for-free#generating-the-certificate) that asks your CA to use SHA-2, using the `-sha256` flag.

```bash
openssl req -new -sha256 -key your-private.key -out your-domain.csr
```

I'm [keeping track of issues and workarounds](https://shaaaaaaaaaaaaa.com/#sha2-certificate) for getting SHA-2 certificates from various CAs. If you run into a problem not mentioned there, [please ring in here](https://github.com/konklone/shaaaaaaaaaaaaa/issues/24) and I'll update the site.

You may also need to update any SHA-1 intermediate certificate(s), as they're also verified using a digital signature. This means tracking down whether and where your CA has published their SHA-2 intermediates. I'm [keeping track of SHA-2 intermediate locations](https://shaaaaaaaaaaaaa.com/#sha2-intermediate) for various CAs. If you find some not mentioned, or your CA doesn't have them yet, [please ring in](https://github.com/konklone/shaaaaaaaaaaaaa/issues/36).

If you have a site but some other company controls the certs, email their customer support and tweet at them. Link to [Google's announcement](http://googleonlinesecurity.blogspot.com/2014/09/gradually-sunsetting-sha-1.html), and ask for their timeline.

I could also use some help on [shaaaaaaaaaaaaa.com](https://shaaaaaaaaaaaaa.com) — if you feel like chipping in, check out the [open issues](https://github.com/konklone/shaaaaaaaaaaaaa/issues) and lend a hand.

<div class="callout">
<strong>SHA-1 roots</strong>: You <em>don't</em> need to worry about SHA-1 root certificates that ship with browsers, because their integrity is verified without using a digital signature.
</div>

## Changing certificates shouldn't be this hard

There is a larger issue here: there's no reason that fixing security issues should be so aggravating. A big reason why websites and Certificate Authorities are dragging their feet on updating to SHA-2 is because it means reissuing certificates, and _everyone_ hates replacing SSL certificates.

For individuals, the certificate process [can be strange](https://konklone.com/post/switch-to-https-now-for-free#register-with-startssl), filled with arcane terminology and confusing user interfaces.

For companies, certificates are generally obtained for as long as possible, stuck wherever, and forgotten about until it's time to panic. Certificate management is often outsourced or out of reach.

In discussing Chrome's new policy, Google's Ryan Sleevi makes this exact point — that security today means that [changing your certificate can't be such a tremendous operational hassle](https://groups.google.com/a/chromium.org/d/msg/security-dev/2-R4XziFc7A/ef18BfCysgkJ):

> The age of having a secure UI remain static for five years is no longer tenable. It is expected that sites be able to respond to and adapting to security threats. This applies whether you're talking about TLS protocol level attacks like BEAST, cryptographic attacks like those involved in certificate issuance, or application-level attacks such as mixed content.
>
> For the vast majority of our users, security is a one-or-two bit state, not the gradiations that CAs may see. In this model, we need to accurately reflect to the user whether or not we believe they're secure.
>
> ...If a site is not capable of changing its cert within 12 weeks, then I think we have a far more serious security problem - as Heartbleed [has] taught us.

If you poke around [Google's SSL configuration](https://www.ssllabs.com/ssltest/analyze.html?d=google.com), you'll see that (!) they use certificates signed with SHA-1. But each certificate expires in 3 months, a short-lived window that reduces the chances that a certificate could be forged, while they [migrate to SHA-2 in 2015](https://twitter.com/agl__/status/503694839481761792).

As importantly, a 3-month window **forces Google to make cert rotation operationally simple**. This is the equivalent of [Netflix's Chaos Monkey](http://blog.codinghorror.com/working-with-the-chaos-monkey/) — forcing yourself to take entropy and change seriously by turning it from an emergency into the routine.

Google has made it somewhat easier for themselves to do this, by setting up a [private intermediate certificate authority for themselves](https://pki.google.com/), signed by GeoTrust, that can reissue certificates on demand. But you don't need to own an intermediate CA to make frequent certificate rotation a part of your business processes — you need to prioritize it. That's what Google's doing, and that's what they're arguing we should all do.

<div class="callout">
<strong>Further reading:</strong> Another related idea, under <a href="https://groups.google.com/forum/#!topic/mozilla.dev.security.policy/T11up58JkFc">active discussion by Mozilla, Google, and others</a>, is to allow for extremely short-lived certs, on the order of a few days. This would improve performance, as browsers wouldn't need to bother with <a href="https://www.imperialviolet.org/2011/03/18/revocation.html">revocation</a> <a href="https://www.imperialviolet.org/2014/04/19/revchecking.html">checking</a> for those certs. It would also bring along the general security and operational benefits of frequent cert rotation, as described above.
</div>

## In conclusion

A SHA-1 push like this should have started years ago. Any annoyance at Google for amping up the pressure should be channeled towards the Certificate Authorities instead, for allowing nothing to happen for so long.

For individuals, obtaining a certificate needs to be as easy as buying a domain name, installing it needs to be easy as turning on a website, and replacing it needs to be automatic. There are some pretty clear business opportunities here.

For organizations, frequent certificate rotation needs to be prioritized in their infrastructure design and update processes. Organizations that have already done a good job of this should talk about it, and open source their work.

In the meantime, site operators should update their certificates and use the lack of Heartbleed-level urgency as an opportunity to revisit their SSL configuration and turn on things like [forward secrecy](https://community.qualys.com/blogs/securitylabs/2013/08/05/configuring-apache-nginx-and-openssl-for-forward-secrecy).

I'll be updating [shaaaaaaaaaaaaa.com](https://shaaaaaaaaaaaaa.com) throughout the transition, so please [open some discussions and report some bugs](https://github.com/konklone/shaaaaaaaaaaaaa/issues).

<div class="footer-text">
<strong>Related Stuff</strong>
<li>The 2008 <a href="http://www.win.tue.nl/hashclash/rogue-ca/">MD5 Collisions, Inc. attack</a> is really a terrific read.</li>
<li>Yes, there is a <a href="https://en.wikipedia.org/wiki/SHA-3">SHA-3</a>, finalized in 2012. We'll probably need to upgrade to it someday.</li>
<li>Heroku has an interesting <a href="https://addons.heroku.com/expeditedssl">all-in-one SSL management service</a> for apps hosted there.</li>
</div>

<section class="discuss footer">Thanks to Jacob Hoffman-Andrews, Tom Macwright, and Lindsay Young for reading drafts of this post.</p>