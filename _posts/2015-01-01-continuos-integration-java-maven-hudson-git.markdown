---
layout: post
title:  "Continuous Integration: Java, Hudson, Maven and Git [english]"
date:   2014-03-30 13:00
categories: tdd test java maven git hudson jenkins
---

<center>*Hello. In this post we'll explore how to create a enviroment in your local machine to check how Continuous Integration (CI) workflow works. I'll assume that you already have Java, Maven and Git installed in your system.*</center>

#### Continuous Integration?

<div class="citacao">
  Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day. Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible. Many teams find that this approach leads to significantly reduced integration problems and allows a team to develop cohesive software more rapidly. This article is a quick overview of Continuous Integration summarizing the technique and its current usage.
</div>

<div class="citacao-autor-right">
  <a href="http://martinfowler.com/articles/continuousIntegration.html">Martin Fowler</a>
</div>

<br/>

Continuous Integration (CI) is very important. If your work with other people, you'll need CI. Well, in a general way, the CI starts when you commit your code to a remote repository. The CI machine is polling this repository and waiting for changes in the code. When a change is detected, the build starts using the last repository copy. Generally, the build consist of many phases, such as "compile the code", "make the database integration", "run the tests", "analyze the code quality", "generate a report", etc.

![]({{ site.url }}/assets/ci-maven-git/hudson0.jpg)

In our case, everything will run in our local machine, but the concept is the same. Our goal here is to be practical, so if you need deep and detail information about this topic, please read the [Martin's article][martin-fowler-article] and the [classical book][ci-classic-book].

In summary, we'll:

1. Create a basic Maven project
2. Create a Git local repository to this project
3. Set up the Hudson to use our repository and run a new build when new code be pushed to the server.

#### Creating a basic Maven project

Open the terminal, and tell to Maven generate your project with this:

{% highlight java %}
mvn archetype:generate -DgroupId=br.com.tutorial -DartifactId=tutorial
-DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
{% endhighlight %}

This could take a while on the first time. Wait until you see the nice success message.

![]({{ site.url }}/assets/ci-maven-git/hudson1.png)

The default project structure of a Maven project is this.

![]({{ site.url }}/assets/ci-maven-git/hudson1-1.png)

Now, open that on your IDE. Then let's create a simple test class. In my case, I had to change the `pom.xml` file to use JUnit 4.

{% highlight java %}
<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>4.10</version>
<scope>test</scope>
</dependency>
{% endhighlight %}

Now, let's create a test method, like this:

{% highlight java %}
@Test
public void sumTwoPositiveNumbers() {
int total = 2 + 2;
org.junit.Assert.assertEquals(4, total);
}
{% endhighlight %}

So, until now we have a project with a success unit test. Later, our CI server will run these tests every time that a new commit is done.

![]({{ site.url }}/assets/ci-maven-git/hudson2.png)

#### Creating a Git local repository

Now, enter in the project directory and initialize your GIT repository running "git init" command.

{% highlight java %}

$ git init
Initialized empty Git repository in /Users/acneto/projecXCI/tutorial/.git/

{% endhighlight %}

Now, let's add all project files to the repository and do our first commit.

{% highlight java %}
$ git add .
$ git commit -m "initial commit"
[master (root-commit) be72345] initial commit
17 files changed, 1028 insertions(+), 0 deletions(-)
create mode 100644 .idea/.name
.
.
.
create mode 100644 pom.xml
create mode 100644 src/main/java/br/com/tutorial/App.java
create mode 100644 src/test/java/br/com/tutorial/AppTest.java
create mode 100644 target/classes/br/com/tutorial/App.class
create mode 100644 tutorial.iml
{% endhighlight %}

In a real life project, you must use a [.gitignore][git-ignore-file] file to exclude many useless things, like IDE folders and temporary files (but let's skip this for now). If you run a "git log", you can confirm you last action.

{% highlight java %}
$ git log
commit be7234524ba25e43e852a196e9b3b463786e2da1
Author: Agostinho Neto <acneto@abc.com>
Date:   Wed Sep 19 17:14:38 2012 -0300
initial commit
{% endhighlight %}

#### Setting up Hudson

Now, let's setup up the Hudson. First, download the [.war file from the web][hudson-download]. Enter in the directory run (for example: "java -jar hudson-2.2.1.war").

{% highlight java %}
macbookpro:hudson-2.2.1 acneto$ java -jar hudson-2.2.1.war
{% endhighlight %}

Now, type http://localhost:8080 and you'll see the main page. Here, we have to do two things: first, install the GIT plugin at Hudson and create a new "job" to our project. A "job" is like a task and each project can have several jobs.

![]({{ site.url }}/assets/ci-maven-git/hudson3.png)

To install the plugin, click on Manage Hudson link on the left side and in the next page, click on Manage Plugins. Now, select the GIT plugin from the available list and install it.

![]({{ site.url }}/assets/ci-maven-git/hudson3-3.png)

With the plugin installed, click on New Job link and give a name to your project Select “Build a free-style software project” and click OK.

![]({{ site.url }}/assets/ci-maven-git/hudson4.png)

In this screen you have to configure your path to the Git repository (generally is a remote address, but here is a local path):

![]({{ site.url }}/assets/ci-maven-git/hudson5.png)

Now, put the time that you want the Hudson check your repository automatically and run the build. We put "* * * * *" to say “hey, run every minute".

![]({{ site.url }}/assets/ci-maven-git/hudson5-5.png)

Finally, put the Maven goals to be executed. Here we put "clean install". This command clean the project and executes the Maven install phase. The Maven Build Lifecycle will execute the test phase too. Check the [Maven lifecycle here][maven-lifecycle].

![]({{ site.url }}/assets/ci-maven-git/hudson6.png)

Now, we’ve finished. You can click “Build Now” on the left side and check out the output. If have to see something like this.

{% highlight java %}
-----T E S T S------
Running br.com.tutorial.AppTest?Tests run: 1, Failures: 0, Errors: 0, Skipped: 0,
Time elapsed: 0.069 sec?Results :?Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
{% endhighlight %}

Now, let’s see the magic happening, from now on, every commit in the repository will be built automatically. Let's change our code to fail the test. Like the Radiohead song, put: 2+2=5 in the test code.

{% highlight java %}
@Test
public void sumTwoPositiveNumbers() {
int total = 2 + 2;
org.junit.Assert.assertEquals(5, total);
}
{% endhighlight %}

Commit this change:

{% highlight java %}
$ git commit src/test/java/br/com/tutorial/AppTest.java -m "bug inserted in
the code here."
{% endhighlight %}

Now, wait a few seconds and see what happens. The Hudson build starts automatically, gets the last code from the repository and runs the Maven phase that you had configured before. As expected, this build will fail and you can check out the output console.

{% highlight java %}
-----?T E S T S?----
Running br.com.tutorial.AppTest?Tests run: 1, Failures: 1, Errors: 0, Skipped: 0,
Time elapsed: 0.089 sec <<< FAILURE!?Results :?Failed tests:?
sumTwoPositiveNumbers(br.com.tutorial.AppTest):
expected:<5> but was:<4>?Tests run: 1, Failures: 1, Errors: 0, Skipped: 0
{% endhighlight %}

You can get the [project scaffold in my Github.][maven-hudson-tutorial]

[maven-hudson-tutorial]: https://github.com/acneto/scaffold-java-maven-tutorial
[ci-classic-book]: http://www.amazon.com/Continuous-Integration-Improving-Software-Reducing/dp/0321336380
[maven-lifecycle]:https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
[git-ignore-file]:https://help.github.com/articles/ignoring-files
[martin-fowler-article]:http://martinfowler.com/articles/continuousIntegration.html
[hudson-download]:http://hudson-ci.org
