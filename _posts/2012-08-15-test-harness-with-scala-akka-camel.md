---
layout: post
title: Implementing a simple test harness using Scala, Akka, Camel (and Maven)
---

{{ page.title }}
================
In my work I've for a long time been looking for a use case where I could use some of the technologies that I'm currently interested in, namly Scala and Akka. And the other day I think I found it. In my current project we have an integration point where we through a SOAP web service communicates with a back end system running on a mainframe. And of cource, in the middle there is an Enterprise servcie bus as a decoupler (?!?). Now, this integration point is somewhat instable. Beside that it is very complex setup, we have issues with test data and so forth. So, to be able to do testing as we want to (functional, performance, endurance, stability, etc), we can't run the tests against the real endpoint. Enter test harness.
So, the requirement from our part was that it should be:
<ol>
  <li>Simple</li>
  <li>Easy to build an maintain</li>
  <li>Easy to extend</li>
  <li>Support our test cases, both functional and non functional</li>
</ol>
How to do this? Well after reading this <a href="http://blog.scala4java.com/2012/04/akka-camel-21-consumerproducer-example.html">blog</a> post, I was on the way.

To implement a test harness that simulates a SOAP endpoint was very simple, see below:

<pre class="terminal">
<code>
package ak
import akka.camel.{Consumer, CamelMessage}
import akka.actor.{Props, ActorSystem, ActorLogging}
import akka.util.Timeout
import akka.util.duration._
import akka.actor.Actor

object TestHarnessApp extends App {
  val sys = ActorSystem(&quot;TestHarnessApp&quot;)
  sys.actorOf(Props[HttpConsumer], &quot;EndPoint-foo&quot;)
  sys.awaitTermination()
}
class HttpConsumer extends Consumer {
  def endpointUri = &quot;jetty://http://0.0.0.0:8888/foo&quot;
  implicit val timeout = Timeout(30 seconds)
  def receive = {
    case msg : CamelMessage =&gt; {
      sender ! soap(&quot;Foo&quot;)
    }
  }
  def soap(message: String) = {
    &lt;soap:Envelope 
    xmlns:soap=&quot;http://www.w3.org/2001/12/soap-envelope&quot; 
    soap:encodingStyle=&quot;http://www.w3.org/2001/12/soap-encoding&quot;&gt;
      &lt;soap:Body xmlns:m=&quot;http://www.foo.org/foo&quot;&gt;
        &lt;m:message&gt;{message}&lt;/m:message&gt;
      &lt;/soap:Body&gt;
    &lt;/soap:Envelope&gt;
  }
}
</code>
</pre>

And since we use Maven to build our system I need to declare the dependencies and have maven do our Scala compilation:
<pre class="terminal">
<code>
&lt;project xmlns=&quot;http://maven.apache.org/POM/4.0.0&quot; 
  xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
  xsi:schemaLocation=&quot;
    http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd&quot;&gt;
  &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
  &lt;groupId&gt;ak&lt;/groupId&gt;
  &lt;artifactId&gt;test-harness&lt;/artifactId&gt;
  &lt;packaging&gt;jar&lt;/packaging&gt;
  &lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;
  &lt;name&gt;Test harness&lt;/name&gt;
  &lt;properties&gt;
    &lt;scala.version&gt;2.9.2&lt;/scala.version&gt;
  &lt;/properties&gt;
  &lt;dependencies&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;org.scala-lang&lt;/groupId&gt;
      &lt;artifactId&gt;scala-compiler&lt;/artifactId&gt;
      &lt;version&gt;${scala.version}&lt;/version&gt;
    &lt;/dependency&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;org.scala-lang&lt;/groupId&gt;
      &lt;artifactId&gt;scala-library&lt;/artifactId&gt;
      &lt;version&gt;${scala.version}&lt;/version&gt;
    &lt;/dependency&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;com.typesafe.akka&lt;/groupId&gt;
      &lt;artifactId&gt;akka-actor&lt;/artifactId&gt;
      &lt;version&gt;2.1-20120718-000833&lt;/version&gt;
    &lt;/dependency&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;com.typesafe.akka&lt;/groupId&gt;
      &lt;artifactId&gt;akka-camel&lt;/artifactId&gt;
      &lt;version&gt;2.1-20120718-000833&lt;/version&gt;
      &lt;exclusions&gt;
        &lt;exclusion&gt;
          &lt;groupId&gt;org.apache.camel&lt;/groupId&gt;
          &lt;artifactId&gt;camel-core&lt;/artifactId&gt;
        &lt;/exclusion&gt;
      &lt;/exclusions&gt;
    &lt;/dependency&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;org.apache.camel&lt;/groupId&gt;
      &lt;artifactId&gt;camel-jetty&lt;/artifactId&gt;
      &lt;version&gt;2.9.2&lt;/version&gt;
    &lt;/dependency&gt;
  &lt;/dependencies&gt;
  &lt;repositories&gt;
    &lt;repository&gt;
      &lt;id&gt;scala-tools.org&lt;/id&gt;
      &lt;name&gt;Scala-tools Maven2 Repository&lt;/name&gt;
      &lt;url&gt;http://scala-tools.org/repo-releases&lt;/url&gt;
    &lt;/repository&gt;
    &lt;repository&gt;
      &lt;snapshots&gt;
        &lt;enabled&gt;false&lt;/enabled&gt;
      &lt;/snapshots&gt;
      &lt;id&gt;typesafe-releases&lt;/id&gt;
      &lt;name&gt;releases&lt;/name&gt;
      &lt;url&gt;http://repo.typesafe.com/typesafe/releases&lt;/url&gt;
    &lt;/repository&gt;
    &lt;repository&gt;
      &lt;snapshots /&gt;
      &lt;id&gt;typesafe-snapshots&lt;/id&gt;
      &lt;name&gt;snapshots&lt;/name&gt;
      &lt;url&gt;http://repo.typesafe.com/typesafe/snapshots&lt;/url&gt;
    &lt;/repository&gt;
  &lt;/repositories&gt;
  &lt;pluginRepositories&gt;
    &lt;pluginRepository&gt;
      &lt;id&gt;scala-tools.org&lt;/id&gt;
      &lt;name&gt;Scala-tools Maven2 Repository&lt;/name&gt;
      &lt;url&gt;http://scala-tools.org/repo-releases&lt;/url&gt;
    &lt;/pluginRepository&gt;
  &lt;/pluginRepositories&gt;
  &lt;build&gt;
    &lt;sourceDirectory&gt;src/main/scala&lt;/sourceDirectory&gt;
    &lt;testSourceDirectory&gt;src/test/scala&lt;/testSourceDirectory&gt;
    &lt;plugins&gt;
      &lt;plugin&gt;
        &lt;groupId&gt;net.alchim31.maven&lt;/groupId&gt;
        &lt;artifactId&gt;scala-maven-plugin&lt;/artifactId&gt;
        &lt;version&gt;3.0.1&lt;/version&gt;
      &lt;/plugin&gt;
      &lt;plugin&gt;
            &lt;artifactId&gt;maven-assembly-plugin&lt;/artifactId&gt;
            &lt;version&gt;2.3&lt;/version&gt;
            &lt;configuration&gt;
              &lt;archive&gt;
                  &lt;manifest&gt;
                    &lt;mainClass&gt;ak.TestHarnessApp&lt;/mainClass&gt;
                  &lt;/manifest&gt;
              &lt;/archive&gt;
                &lt;descriptorRefs&gt;
                  &lt;descriptorRef&gt;jar-with-dependencies&lt;/descriptorRef&gt;
                &lt;/descriptorRefs&gt;
            &lt;/configuration&gt;
            &lt;executions&gt;
              &lt;execution&gt;
                &lt;id&gt;make-assembly&lt;/id&gt;
                &lt;phase&gt;package&lt;/phase&gt;
                &lt;goals&gt;
                  &lt;goal&gt;single&lt;/goal&gt;
                &lt;/goals&gt;
              &lt;/execution&gt;
            &lt;/executions&gt;
          &lt;/plugin&gt;
    &lt;/plugins&gt;
  &lt;/build&gt;
&lt;/project&gt;
</code>
</pre>
And as you see in the above code I also added a section with the maven-assembly-plugin to assemble an executable jar file. 
So, to build and run our test harness:

<pre>
mvn scala:compile install
</pre>

Put the resulting jar file on a server of your choice and fire it up with:
<pre>java -jar test-harness-1.0-SNAPSHOT-jar-with-dependencies</pre>

And then configure your program to access your test harness instead of the real endpoint.

The simplicity is remarkable, not many rows of code is necessary (actually, most of the code is maven conf and that could be fixed by gradle or sbt, but that is another story). No server is needed to be installed and configured. Most of the creds goes to Akka and Camel. Because of these to we have a simple, light, and performant solution that is easy to extend.

And of course, the test harness in the above condition doesn't support that many test cases, but it is very easy to see how it could be extended to support what ever test case you would like.

The complete source of above could be found here:
<a href="https://github.com/andreaskallberg/test-harness">https://github.com/andreaskallberg/test-harness</a>

cheers 