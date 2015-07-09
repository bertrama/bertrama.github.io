---
layout: post
title:  "Configuring JGroups"
date:   2015-07-08 20:38:25
categories: tech jgroups
---
I was working on configuring [JGroups][jgroups].  My main interest was to get encrypt and auth working. I wasn't able to locate adequate documentation on what the JGroups protocol stack should look for these two. Nor did I find an example of how to create the keystore to work with both.  Here is what I came up with.

First off, creating a keystore with keytool:

{% highlight bash %}
keytool \
  -genseckey \
  -alias EncryptKey \
  -keypass changeit \
  -storepass changeit  \
  -keyalg Blowfish \
  -keysize 56 \
  -keystore defaultStore.keystore \
  -storetype  JCEKS

keytool \
  -genkeypair \
  -dname "cn=Name,ou=Unit,o=Organization,c=XX" \
  -alias AuthKey \
  -keyalg RSA \
  -keypass changeit \
  -storepass changeit \
  -keystore defaultStore.keystore \
  -storetype JCEKS
{% endhighlight %}

Then in the jgroups.xml:

{% highlight xml %}
<config xmlns="urn:org:jgroups"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/JGroups-3.4.xsd">
  <TCP bind_addr="${jgroups.tcp.bind_addr}"
       bind_port="${jgroups.tcp.bind_port}"
       />
  <TCPPING initial_hosts="${jgroups.tcpping.initial_hosts}"
           port_range="10"
           />
  <MERGE3/>
  <FD_SOCK/>
  <FD_ALL interval="2000"
          timeout="5000"
          />
  <VERIFY_SUSPECT timeout="500"  />
  <ENCRYPT key_store_name="defaultStore.keystore"
           store_password="changeit"
           alias="EncryptKey"
           />
  <BARRIER />
  <pbcast.NAKACK use_mcast_xmit="false"
                 retransmit_timeout="100,300,600,1200"
                 discard_delivered_msgs="true" />
  <UNICAST3 conn_expiry_timeout="0"/>
  <pbcast.STABLE stability_delay="1000" desired_avg_gossip="50000"
                 max_bytes="10m"/>
  <pbcast.GMS print_local_addr="true" join_timeout="5000"
              max_bundling_time="30"
              view_bundling="true"/>
  <MFC max_credits="2M"
       min_threshold="0.4"/>
  <FRAG2 frag_size="60000"  />
  <AUTH auth_class="org.jgroups.auth.X509Token"
        auth_value="${jgroups.auth.auth_value}"
        keystore_path="defaultStore.keystore"
        keystore_password="changeit"
        keystore_type="JCEKS"
        cert_password="changeit"
        cert_alias="AuthKey"
        cipher_type="RSA"/>
  <pbcast.STATE_TRANSFER  />
</config>
{% endhighlight %}

This configuration works for me when every member of the cluster has a copy of the same keystore.  I thought I might try distributing the keystore in my application's .war file.  ENCRYPT will find the keystore if you put it in the classpath. AUTH, however, does not.  I provided a patch to X509Token to make it behave like ENCRYPT. If it isn't accepted, it's still pretty easy to build a new class to use as your auth_class.

[jgroups]: http://www.jgroups.org
