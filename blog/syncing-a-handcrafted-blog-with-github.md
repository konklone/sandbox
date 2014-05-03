For a while now, I've admired my friend [Ben Balter](https://twitter.com/benbalter) for writing out in the open. His blog is built on [GitHub Pages](https://pages.github.com/), which means that for every [piece](http://ben.balter.com/2014/03/21/want-to-innovate-in-government-focus-on-culture/) on his blog, you can see [each draft](https://github.com/benbalter/benbalter.github.com/commits/master/_posts/2014-03-21-want-to-innovate-in-government-focus-on-culture.md) and [revision](https://github.com/benbalter/benbalter.github.com/commit/d39bfec676299afed1c1018cd83a15f31a81ad8e) along the way. 

As importantly, Ben takes advantage of GitHub to truly embrace collaboration — whether that's [detailed debate on a complex topic](https://github.com/benbalter/benbalter.github.com/pull/98) as he works through issues, or just graciously [fielding](https://github.com/benbalter/benbalter.github.com/pull/108) 
[my](https://github.com/benbalter/benbalter.github.com/pull/105) [constant](https://github.com/benbalter/benbalter.github.com/pull/91) [typo](https://github.com/benbalter/benbalter.github.com/pull/77) [fixes](https://github.com/benbalter/benbalter.github.com/pull/99). The end product is better for it, and everyone feels engaged and invested.

It's such a healthy, beautiful approach to public writing. Why doesn't every newspaper in the world work this way?

### Making this work for my blog

And why not my blog? Well, Ben has the advantage of using GitHub Pages, which weaves his blog directly into [GitHub](https://github.com), one of the greatest systems for collaboration that mankind has yet crafted. And GitHub Pages, while [incredibly powerful](https://konklone.com/post/the-power-and-potential-of-github-pages), isn't for everyone. For me, I like having a database that can hold private content, and a comment system that belongs to me and not to Disqus. 

Fortunately, GitHub is much more than a nice website and host -- it's also home to an [extraordinarily rich and well-documented API](https://developer.github.com/v3/), through which you can order GitHub to do just about anything you need. Using it, I've neatly integrated my published blog posts with a new GitHub repository, [konklone/writing](https://github.com/konklone/writing). There's now an "[edit this post on GitHub](#)" link on the sidebar, to make the connection obvious.

I don't have to change my workflow or give up my database, but I still get a [public revision history](https://github.com/konklone/writing/commits/master/blog/switch-to-https-now-for-free.md) and the ability to accept pull requests that automatically update my blog as soon as I press the `Merge` button.

### How to sync with GitHub

Any changes to my database I make from inside my blog's internal CMS get automatically synced to a corresponding Markdown file on GitHub, and any changes from GitHub get automatically synced to my blog's database.

If you're curious about the technical mechanics, I created two pull requests that explain it:

* [Syncing TO GitHub](https://github.com/konklone/konklone/pull/125): Any published post with an assigned URL to a file on GitHub will, upon save, use the GitHub [Repo Contents API](https://developer.github.com/v3/repos/contents/) to update that file with the post's contents. A GitHub URL is assigned to a post upon first publish, which then automatically creates a file in [konklone/writing](https://github.com/konklone/writing).
* [Syncing FROM GitHub](https://github.com/konklone/konklone/pull/126): The `writing` repo has a [webhook](https://github.com/blog/1778-webhooks-level-up) enabled that sends a POST to a special endpoint in my blog upon any content changes. My blog accepts those changes from GitHub, while using a log of previously synced commits to avoid infinite sync loops.

Now, my blog is built on [Sinatra](http://www.sinatrarb.com/) and [MongoDB](https://www.mongodb.org/), and is hosted on [Amazon EC2](https://aws.amazon.com/ec2/), but you could take this approach with just about everything. It's just HTTP, the *lingua franca* of the Internet, along with a couple extra fields on posts to keep everything straight. It's not too bad, and if you try it, I encourage you to refer to my approach above.

One thing that keeps the system simple: posts are linked to files on GitHub through one field, a URL, e.g.: 

<blockquote><p>
<a style="font-size: 10pt" href="https://github.com/konklone/writing/blob/master/blog/open-data-day-dc-2014.md">https://github.com/konklone/writing/blob/master/blog/open-data-day-dc-2014.md</a>
</p></blockquote>

That URL contains all the data you need to establish the owner, repository, branch, path, and filename of a file on GitHub. It can be easily split apart into (or reassembled from) pieces on demand, and using an alternate repository as an exception (e.g. a guest post) just means changing the URL.

## 

I'd like to keep using this blog as a place to create [useful resources](https://konklone.com/post/switch-to-https-now-for-free) people return to, and [living guides](https://konklone.com/post/take-control-of-your-email-address) that stay up-to-date as times change. I also just want to write good stuff that reads clearly. It's a lot easier to make all that happen with an obvious contribution path, and a clear revision history.

I also don't see any reason why a larger site like a newspaper can't do the same thing. Why do we accept an unaccountable sentence of text at the bottom of a New York Times article?  It doesn't matter whether or not the solution uses GitHub: 

todo:
* header image
* NYT/newspaper correction example
* more detail?
