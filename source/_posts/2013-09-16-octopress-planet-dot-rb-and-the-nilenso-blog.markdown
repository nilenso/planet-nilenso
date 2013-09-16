---
layout: post
title: "Octopress, Planet.rb and the Nilenso Blog"
date: 2013-09-16 16:26
comments: true
categories: 
published: false
---

We use [Octopress](http://octopress.com) with [planet.rb](http://github/pote/planet.rb) for this blog. It aggregates our personal blogs and also lets us post on the company's behalf. It works pretty well.

We started off with [planetplanet](http://), tried [planetvenus](http://) and then settled on planet.rb. The plusses are:

-  Octopress is great for us ruby devs. We needed partials, sass, themeability and a static site
-  Planet.rb works as a plugin for Octopress. So everything in Octopress is as-is. We can now use the blog as a company blog and an aggregator at the same time
-  Planet.rb generates markdown files for all the feeds it parses. This keeps it consistent with the rest of the blog, and we love writing markdown

###The improvements we needed to make
 However, planet.rb is still in development, and there were a couple of things that needed to change before we could release. One such feature was the ability to filter posts that are not suitable for the company blog. We implemented this as a whitelist of tags. Only posts that have any tags in the whitelist will be imported.
 Here is our whitelist:
 ```yaml
 whitelisted_tags: [dev, software, ruby, nilenso]
 ```

Another issue with planet.rb was that it quit abruptly when it failed to parse a blog because the blog was unreacheable. We fixed that and then it was good to go. Here's how it looks now when we run `planet generate`:

Taking the final step in automating this, we set up a simple cronjob to aggregate posts everyday:
```bash crontab -e
@daily cd /path/to/blog && planet generate && rake generate
```
