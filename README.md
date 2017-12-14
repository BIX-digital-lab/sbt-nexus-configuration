# sbt-nexus-configuration
using sbt with nexus as single build source

We are pretty heavy users of scala - and hence wanted to use a single artifact repository that does all the proxy work behind the scenes. In our case that's Nexus

After countless attempts to get this working (Nexus with secured access) - here is the steps that made it fly for us.
(the work is largely based on https://stackoverflow.com/questions/40224921/sbt-is-unable-to-find-credentials-when-attempting-to-download-from-an-artifactor, https://stackoverflow.com/questions/4348805/how-to-access-a-secured-nexus-with-sbt  and http://www.scala-sbt.org/1.x/docs/Proxy-Repositories.html)

1. download & extract latest sbt (sbt-1.0.3.zip) to $SBT_HOME
2. in $USER_HOME/.sbt - create a new file called **repositories** (in my case: c:\Users\utschig\.sbt\repositories)
```
[repositories]
  local
  my-ivy-proxy-releases: <IVY proxy group>, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/[revision]/[type]s/[artifact](-[classifier]).[ext]
  my-maven-proxy-releases: <MAVEN Public releases proxy group>
```
3. in $USER_HOME/.sbt - create a new file called **.credentials** with the following content
```
realm=Sonatype Nexus Repository Manager
host=<NEXUS HOST, w/o port, and w/o protocol, e.g. x.y.z.com)
user=<NEXUS username with rights to access (view!) the repo groups)
password=<NEXUS pw>
```
4. in $SBT_HOME/conf/**sbtconfig.txt** add system properties to use your custom repository configuration & credentials. 
```
-Dsbt.override.build.repos=true
-Dsbt.boot.credentials=c:\users\utschig\.sbt\.credentials 
```
5. when you run sbt now .... it will update itself download all sorts of libs - 
```
C:\Users\utschig>sbt
"C:\Users\utschig\.sbt\preloaded\org.scala-sbt\sbt\"1.0.3"\jars\sbt.jar" 'about to robocopy'

-- omitted --

  Beendet: Thu Dec 14 09:56:18 2017
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256m; support was removed in 8.0
Getting org.fusesource.jansi jansi 1.11 ...
downloading <NEXUS_HOST>/maven-public/org/fusesource/jansi/jansi/1.11/jansi-1.11.jar ...
        [SUCCESSFUL ] org.fusesource.jansi#jansi;1.11!jansi.jar (346ms)
:: retrieving :: org.scala-sbt#boot-jansi
        confs: [default]
        1 artifacts copied, 0 already retrieved (111kB/30ms)
Getting org.scala-sbt sbt 1.0.3 ...
downloading <NEXUS_HOST>/repository/maven-public/org/scala-sbt/sbt/1.0.3/sbt-1.0.3.jar ...
        [SUCCESSFUL ] org.scala-sbt#sbt;1.0.3!sbt.jar (342ms)
downloading <NEXUS_HOST>/repository/maven-public/org/scala-lang/scala-library/2.12.4/scala-library-2.12.4.jar ...
        [SUCCESSFUL ] org.scala-lang#scala-library;2.12.4!scala-library.jar (7323ms)
downloading h<NEXUS_HOST>/repository/maven-public/org/scala-sbt/main_2.12/1.0.3/main_2.12-1.0.3.jar ...

-- omitted --

[info] Done updating.
[info] Set current project to utschig (in build file:/C:/Users/utschig/)
[info] sbt server started at 127.0.0.1:4289

```
6. Now create hello world .. (http://www.scala-sbt.org/0.12.4/docs/Getting-Started/Hello.html) and run it .. 
```
sbt:scala> run
[info] Running Hi
[debug] Waiting for threads to exit or System.exit to be called.
[debug] Waiting for thread run-main-0 to terminate.
[debug]   Classpath:
[debug]         C:\Users\utschig\AppData\Local\Temp\sbt_869beb5a\job-1\target\e5eec10b\scala_2.12-0.1-SNAPSHOT.jar
[debug]         C:\Users\utschig\AppData\Local\Temp\sbt_869beb5a\target\7663f74e\scala-library.jar
Hi!
[debug]         Thread run-main-0 exited.
[debug] Interrupting remaining threads (should be all daemons).
[debug] Sandboxed run complete..
[debug] Exited with code 0
[success] Total time: 2 s, completed 14.12.2017 10:10:25
```
Voila ... SBT running against nexus
