---
layout: post
title: Http status code in akka-camel
---

{{ page.title }}
================
After a couple of weeks of using the test harness I presented in my previous post I came across a test case the other day where the test harness had to simulate various error cases. And since we are using http as transport protocol, error situations are (mainly) represented through http status codes.
Now, in a servlet environment (or other framework) where one have direct access to the HttpServletResponse object, this is straight forward. But since I now is in a somewhat new environment with scala, akka and camel, the task demanded quite some time to figure out. As usual, in the end it was very simple. As the akka-camel integration is rather thin, the answer was; "standard camel". So, to spare some of you the time, here is the solution:
<pre class="terminal">
<code>
class HttpConsumer extends Consumer {
  def endpointUri = &quot;jetty://http://0.0.0.0:8888/bad-request&quot;
  implicit val timeout = Timeout(30 seconds)
  def receive = {
    case msg : CamelMessage =&gt; {
      sender ! CamelMessage(&quot;Be nice&quot;, 
          Map(org.apache.camel.Exchange.HTTP_RESPONSE_CODE -> "400"))
    }
  }
}

</code>
</pre>
