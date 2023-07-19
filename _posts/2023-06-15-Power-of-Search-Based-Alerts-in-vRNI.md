# Power of Search Based Alerts in vRNI - Network Insight - VMware Core Confluence

Spotting problem based on configuration or metrics and alerting the end
user is an important usecase in vRNI. Hence vRNI provides about 380 out
of the box problem alerts, 31 change alerts, several intent based
alerts, threshold & outlier analytics based alerts. And yet, it is
impossible to foresee every possible scenario for which an admin might
want to get alerted about (e.g.: "no VM should have more than 6 CPU
cores, if anyone creates such a VM, alert me!" OR "My core switches
should always have atleast 1 uplink port functional, let me know if that
is not the case!" OR "ESX Hosts A, B, C & D are very important, nothing
should change on them without me coming to know about it" )

Keeping this shortcoming in mind, vRNI has a feature that we call
"Search Based Alerts" (sometimes also called: "user defined alerts")

With this, if you can write a query in vRNI to express your usecase, you
can have alerts generated for that.

Lets see how we could do that for the examples we mentioned earlier.

1. No VM should have more than 6 CPU cores, if anyone creates such a
   VM, alert me
   
   1. Use the search query: "*vm where cpu cores \> 6*"  
   
   2. On the search results page, click on the "bell" icon  
      ![](/docs/assets/images/search_based_alerts/Screenshot%202023-03-24%20at%2019.15.20.png)
       
   
   3. This will take you to the popup for creating and customising the
      search based alert where you can name the alert and configure
      the various parameters depending on when you want the alerts to
      be generated, whether you want these alerts to be treated as a
      "problem" to be fixed or a "change" to be aware about, its
      severity and choose the notifications to be sent out when the
      alert is triggered.  
      ![](/docs/assets/images/search_based_alerts/Screenshot%202023-03-24%20at%2019.16.31.png) 
      
      1. For instance, in this scenario, we don't want any VMs with
         more than 6 CPU cores. So, we do an initial search for VMs
         more than 6 CPU cores and in the search based alert
         configuration, we ask vRNI to alert us if this initial
         search result changes any time in the future. That way from
         this point onwards if any VM is discovered to be having more
         than 6 CPUs, vRNI will raise an alert.
      2. Since having VMs more than 6 CPUs is a problem we want to
         resolve as soon as it occurs, we also set the "Definition
         Type" to "Problem", that way any alert from this
         configuration will show up in the list of "open problems"
         when searched in vRNI, and when those VMs are eliminated or
         modified to have the right number of CPU cores, the alert
         will close.  
   
   4. Once the condition in the search based alert is triggered, vRNI
      should generate an alert like this:  
      ![](/docs/assets/images/search_based_alerts/Screenshot%202023-03-24%20at%2020.01.23.png) 

2. <u>My core switches should always have atleast 1 uplink port
   functional, let me know if that is not the case</u>
   
   1. In this case, we can write the query like this for every core
      switch we want to monitor: "switch port where Operational Status
      = 'UP' and name like 'uplink' and device = 'core-switch-1'"
   2. This query will show you all the ports whose name has the word
      "uplink" in its name, belongs to the router named
      "core-switch-1" and are operational.
   3. Since we are interested in getting alerted when no such port
      exists, we cant use the "the search results change" criteria for
      the "Generate alert when" parameter of the Search-Based-Alert
      popup since that one would get triggered even if the port count
      decreases by just 1. So instead we will use the other option
      that is available for this parameter, as is seen here:

3. <u>ESX Hosts A, B, C & D are very important, nothing should change
   on them without me coming to know about it</u>
   
   1. This usecase combines the power of user defined alerts with
      vRNI's change alerts.
   
   2. vRNI generates a change alert whenever it notices any change in
      the configuration of any entity discovered by vRNI. So, if I
      wanted to see all the changes that happened for ESX Server A, I
      can write a query like:"change alert where Entity in
      ('host-ip-or-fqdn-1', 'host-ip-or-fqdn-2') and message = 'Entity
      type Host properties updated'"
   
   3. This will list all the change alerts generated till now. You are
      now interested in getting alerted when anything else changes
      from this point forward.
   
   4. Now, at the time of this writing (vRNI version 6.9 is about to
      go GA), vRNI doesnt allow you to create a Search Based Alerts
      from UI on alert object themselves since alerts on alerts, if
      created indiscriminately, could soon lead to alert storms. But,
      we do allow these to be created from our public API.
   
   5. So, you can use the POST call on public
      API: https://your-vrni-ip-or-fqdn/api/ni/settings/alerts/search-based-alerts with
      a request body like this to create a search based alert on
      alerts itself:
      
      <table data-border="0" data-cellpadding="0" data-cellspacing="0">
      <colgroup>
      <col style="width: 100%" />
      </colgroup>
      <tbody>
      <tr class="odd">
      <td class="code"><div class="container"
      title="Hint: double-click to select code">
      <div class="line number1 index0 alt2" data-bidi-marker="true">
      <code class="sourceCode java"><span class="op">{</span></code>
      </div>
      <div class="line number2 index1 alt1" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"alert_name"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"Crown Jewel ESX Servers should not change without me knowing about it"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number3 index2 alt2" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"search_criteria"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"change alert where Entity in (&#39;host-ip-or-fqdn-1&#39;, &#39;host-ip-or-fqdn-2&#39;) and message = &#39;Entity type Host properties updated&#39;"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number4 index3 alt1" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"generate_alert_criteria"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"SEARCH_RESULT_CHANGE"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number5 index4 alt2" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"alert_type"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"PROBLEM"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number6 index5 alt1" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"severity"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"Critical"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number7 index6 alt2" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="st">"notification_settings"</span></code><code
      class="sourceCode java"><span class="op">:</span> <span class="op">[</span></code>
      </div>
      <div class="line number8 index7 alt1" data-bidi-marker="true">
      <code class="sourceCode java">    </code><code
      class="sourceCode java"><span class="op">{</span></code>
      </div>
      <div class="line number9 index8 alt2" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="st">"type"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"EMAIL"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number10 index9 alt1" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="st">"frequency"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"IMMEDIATE"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number11 index10 alt2" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="st">"notification_time"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="st">"string"</span></code><code
      class="sourceCode java"><span class="op">,</span></code>
      </div>
      <div class="line number12 index11 alt1" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="st">"receivers"</span></code><code
      class="sourceCode java"><span class="op">:</span> <span class="op">[</span></code>
      </div>
      <div class="line number13 index12 alt2" data-bidi-marker="true">
      <code class="sourceCode java">        </code><code
      class="sourceCode java"><span class="st">"myemail@mycompany.com"</span></code>
      </div>
      <div class="line number14 index13 alt1" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="op">],</span></code>
      </div>
      <div class="line number15 index14 alt2" data-bidi-marker="true">
      <code class="sourceCode java">      </code><code
      class="sourceCode java"><span class="st">"enabled"</span></code><code
      class="sourceCode java"><span class="op">:</span> </code><code
      class="sourceCode java"><span class="kw">true</span></code>
      </div>
      <div class="line number16 index15 alt1" data-bidi-marker="true">
      <code class="sourceCode java">    </code><code
      class="sourceCode java"><span class="op">}</span></code>
      </div>
      <div class="line number17 index16 alt2" data-bidi-marker="true">
      <code class="sourceCode java">  </code><code
      class="sourceCode java"><span class="op">]</span></code>
      </div>
      <div class="line number18 index17 alt1" data-bidi-marker="true">
      <code class="sourceCode java"><span class="op">}</span></code>
      </div>
      </div></td>
      </tr>
      </tbody>
      </table>

Once created, Search Based Alert configurations show up in the "Alerts
and Notifications" part of the settings page where you can edit or
delete them.They will show up under the "Problems" tab if you had chosen
the "Definition Type" of "Problem" while creating the alert definition,
else they will show up under the "Changes" tab.Please note that while
you can edit the parameters like "Definition Type" etc., you can't
change the search query itself. If you need to edit the query, simply
delete the existing configuration and create a new one.  

While all the examples used in this blog leverage queries on
configuration, you can create search based alerts on metrics too.

For instance, if you want to ensure that no VM in your datacenter should
ever consume more than 60% CPU, simply fire a query like "vm where CPU
Usage Rate \> 60 %" in vRNI and create a Search Based alert on that with
the "Generate alert when" criteria of "the search result changes".

We hope that based on these examples, you will be able to extrapolate
your own use-cases and requirements and leverage the power of Search
Based Alerts from vRNI.
