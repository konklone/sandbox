Recently, GitHub Pages [began supporting HTTPS](https://twitter.com/benbalter/status/444555263195217920) for `*.github.io` domains, like [this one](https://cfpb.github.io/) and [this one](https://sunlightlabs.github.io/congress/). 

In my opinion, this is a **huge** deal - GitHub Pages is the only major, free, general-purpose web host which also offers a secure channel. Given [how many simple and powerful uses](https://konklone.com/post/the-power-and-potential-of-github-pages) GitHub Pages has, and how vital secure and confidential connections are to the future of the web, I see this as a big step forward for developers, GitHub, and everyday people.

There are two catches: you *can't* force your site to only run over HTTPS, and you *can't* use HTTPS for custom domains.

But there's a lot you **can** do! Let's work around the problems.

### Defaulting to HTTPS

If you have a GitHub Pages site, and you'd like it to run as close to HTTPS-only as possible:

* Always link to your own site using its `https://` URL. Go through any supporting sites or blog posts or material and update the link.
* If there are any external and prominent links to your site, ask the site owner to update their link to use `https://`.
* If you're a go-getter, do a GitHub search for [anyone linking to your domain](https://github.com/search?q=%22sunlightlabs.github.io%22&ref=cmdform&type=Code), and file some quick in-browser PRs to switch people's links.

But none of those are important as...

### Forcing a redirect

So, how do you force a redirect for a static site when you don't control the server? You can't use a `<meta refresh>` tag, because you can't change what HTML gets delivered per-protocol.

The only choice is to **do it in JavaScript**. Put the following at the top of your HTML, replacing `YOURDOMAIN` with your user or organization name:

```javascript
var host = "YOURDOMAIN.github.io";
if ((host == window.location.host) && (window.location.protocol != "https:"))
    window.location.protocol = "https";
```

The above hack will immediately redirect any users who visit your domain on an insecure protocol to a secure protocol. It won't affect visitors on any other domain (like `localhost`). This is what we're doing at the [Sunlight Foundation](https://sunlightfoundation.com) now for the documentation for our [Sunlight Congress API](https://sunlightlabs.github.io/congress/).

Make no mistake, client-side redirecting is a hack and an imperfect solution by any means. It's also not secure! But the whole point is to make sure that the links other people make to your website going *forward* are the secure kind. The more links that you and others make that go directly to `https://`, the less often the redirect will need to be triggered.

If you're using Jekyll, you may wish to [follow my example](https://github.com/sunlightlabs/congress/commit/6426761a671d46df6fc5d2526bdaf506c39d789c) of using a `site.enforce_ssl` parameter, or you can just hardcode it like above. It'd be nicer if this could get baked into a Jekyll plugin, but GitHub Pages only supports a couple of whitelisted plugins. 

### Using your custom domain

Well, there's no easy, free way to work around this one. _Yet_! 

[Cloudflare](https://www.cloudflare.com/) will happily sit in front of your domain and terminate SSL for you, but right now they charge $20 per site per month. Fortunately, Cloudflare is [on a mission to double SSL on the web in 2014](http://www.theverge.com/2013/12/17/5217800/cloudflare-pledges-to-double-ssl-usage-on-the-web-in-2014), and they plan to do it by [offering SSL termination for free](https://twitter.com/CloudFlare/status/450390445365800961). That will be a serious gamechanger, not just for GitHub Pages but for the whole Internet.

**Update**: Unfortunately, GitHub will **also** need to update their configuration. I've been testing out a paid $20/month CloudFlare plan against GitHub Pages' SSL configuration, and you can [see the results for yourself](https://theunitedstates.io/): `unknown domain: theunitedstates.io`. Until that's resolved, which requires action on GitHub's part, not even turning on CloudFlare will make this work.

### What GitHub can do

It's unlikely that GitHub is going to create the infrastructure needed to natively support custom domains, and Cloudflare's leadership will render that less of a problem. 

However, there's still a few things GitHub can do to lower the barrier to using HTTPS and to make people more aware of it.

* **Document the feature.** There's no formal announcement or description of SSL for GitHub Pages; it could disappear any time. It's worth a quick blog post and a help page, to cement GitHub's commitment and describe how to avoid common mixed content pitfalls.
* **Allow SSL for custom domains configured elsewhere.** As mentioned above, even if you configure your custom domain with a service like CloudFlare and point a CNAME at GitHub Pages, you'll get an [error like this one](https://theunitedstates.io). That requires action on GitHub's part to fix.
* **Shore up the SSL configuration.** `github.com`'s SSL is [world-class](https://www.ssllabs.com/ssltest/analyze.html?d=github.com&s=192.30.252.128&hideResults=on), but `github.io`'s could still [use some tweaks](https://www.ssllabs.com/ssltest/analyze.html?d=sunlightlabs.github.io) around forward secrecy and cipher choices.
* Most importantly, **let users turn HTTPS on by default**, with a checkbox in their settings that forces a redirect at the server level. That would render the entire hack above unnecessary, and lead a lot more people to Just Do It. In fact, turn on the checkbox by default for new users, and for existing users who don't yet have any GitHub Pages!

2014 may well be the Year of SSL, and GitHub is fertile ground for expanding the playing field. If you want to push this forward, email [support@github.com](mailto:support@github.com) and ask for formal HTTPS support in GitHub Pages. (Referencing this blog post can't hurt.)

In the meantime, I've been going around [filing PRs](https://github.com/project-open-data/project-open-data.github.io/pull/295) and [opening tickets](https://github.com/cfpb/cfpb.github.io/issues/22) with GitHub Pages sites I care about, to squash mixed-content warnings and ensure they're HTTPS-ready. Go adopt some sites you know on GitHub Pages and do the same!