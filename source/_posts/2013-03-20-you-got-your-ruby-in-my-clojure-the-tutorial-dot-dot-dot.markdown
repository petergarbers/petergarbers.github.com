---
layout: post
title: "You Got Your Ruby in My Clojure! The Tutorial..."
date: 2013-03-20 00:48
comments: true
categories: [jRuby, clojure, immutant, torquebox]
immutant-url: https://github.com/immutant/lein-immutant
immutant-tutorial: http://immutant.org/tutorials/overlay/index.html
---

In this tutorial I will run you the process of setting up your torquebox so that it is able to serve both jruby and clojure apps from the same app server!

First things first... You will need to install immutant. For those of you unfamiliar with this. Immutant is
an application server for Clojure. You can find their github project and installation instructions <a href={{ page.immutant-url}}>here</a>

Now the fun begins. I began by following these instructions <a href={{ page.immutant-tutorial}}>instructions</a>
but found a couple of gotchas that I hope will help you along with this process.

<a href={{ page.external-ur }}>{{ page.external-ur }}</a>

<!-- more-->

{% codeblock lang:bash%}
lein immutant overlay torquebox
{% endcodeblock %}

What this does is it downloads a standalone version of TorqueBox along with it's own version of jRuby.
As a result we need to set the TorqueBox home and tell our system to use the jRuby that comes with TorqueBox.

{% codeblock lang:bash%}
export TORQUEBOX_HOME=$HOME/.lein/immutant/current
export PATH=$TORQUEBOX_HOME/jruby/bin:$PATH
{% endcodeblock %}

Now you'll need to bundle your gems. I'm using the full path to be safe

{% codeblock lang:bash%}
cd /path/to/your/jruby/app
$TORQUEBOX_HOME/jruby/bin/jruby -S bundle install
{% endcodeblock %}

This is where the excitement starts. You will now tell TorqueBox to deploy your jRuby app.

{% codeblock lang:bash%}
torquebox deploy /path/to/your/jruby/app
{% endcodeblock %}

Now if you're lucky, from the root of your clojure app

{% codeblock lang:bash%}
lein immutant run
{% endcodeblock %}

I however wasn't so lucky and was greeted by a massive stack trace. Upon futher inspection the following was found:

{% codeblock lang:bash%}
22:06:15,000 ERROR [org.projectodd.polyglot.cache.as] (MSC service thread 1-1) Cannot get cache container. : javax.naming.NameNotFoundException: infinispan/container/polyglot -- service jboss.naming.context.java.jboss.infinispan.container.polyglot
  at org.jboss.as.naming.ServiceBasedNamingStore.lookup(ServiceBasedNamingStore.java:103)
	at org.jboss.as.naming.NamingContext.lookup(NamingContext.java:193)
	at org.jboss.as.naming.NamingContext.lookup(NamingContext.java:173)
	at org.jboss.as.naming.InitialContext.lookup(InitialContext.java:122)
	at org.jboss.as.naming.NamingContext.lookup(NamingContext.java:182)
	at org.jboss.as.naming.NamingContext.lookup(NamingContext.java:178)
    ...
{% endcodeblock %}

After chatting to the nice people on #immutant on freenode I was told to look for the <container-cache> in ~/.lein/immutant/current/jboss/standalone/configuration/standalone.xml and append the following:

{% codeblock ~/.lein/immutant/current/jboss/standalone/configuration/standalone.xml lang:xml%}
<cache-container name='polyglot' default-cache='sessions' aliases='torquebox'>
    <local-cache name='sessions' start='EAGER'>
        <eviction strategy='LRU' max-entries='10000'/>
        <expiration max-idle='100000'/>
        <transaction mode='FULL_XA'/>
    </local-cache>
</cache-container>
{% endcodeblock %}

Restarted the server once more and everything was just peachy :)
