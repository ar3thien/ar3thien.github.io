---
layout: post
title:  "Install Omada Controller on Ubuntu 18.04"
date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---

In this tutorial we will see how to install TP-Link Omada Controller on a Ubuntu 18.04 server.

My environment is running on a proxmox hypervisor and will be running on a LXC container.

The LXC container will be running using the following resources:
1 vCPU
512 MB RAM
4 GB disk

First step is to update the repo and upgrade to get the latest packages
```
> sudo apt-get update && apt-get -y upgrade
```

Install dependencies
```
> sudo apt-get -y install openjdk-8-jre-headless jsvc curl gnupg
```

Install MongoDB v3.6
```
> wget -qO - https://www.mongodb.org/static/pgp/server-3.6.asc | sudo apt-key add -
> echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
> sudo apt-get update && apt-get install -y mongodb-org
> sudo systemctl start mongod
> sudo systemctl enable mongod
```

Obtain Omada Controller v4 binary from [TP-Link official website](https://www.tp-link.com/en/support/download/omada-software-controller/#Controller_Software) and install it
```
> wget https://static.tp-link.com/2020/202012/20201211/omada_v4.2.8_linux_x64.deb
> dpkg -i omada_v4.2.8_linux_x64.deb
```

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
