---
title: "Clojure Deploying Like a Boss"
date: 2014-10-19 12:00:00
layout: post
categories: devops,clojure
---

Clojure is cool, with the help of [JVM][jvm_hosted] and [lein][lein],
we can easily build and compile a web app [from scratch][compojure_usage].


And one of my favourite feature in clojure is compiling the source code
into a single [jar][clojure_compile] (or [war][jar_vs_war]). Unlike Ruby/Python/PHP,
all we have to do is:

1. upload the compiled (standalone) jar to server,
1. restart the service.


## Homebrew solution

After a few investigations, I developed this approach to deploy [my app][gists-as-tricks]:

1. make some changes to the source code,
2. commit the changeset,
3. push to the upstream,
4. use [fabric][sample_fabfile] to deploy the latest code (of course auto pull in server side is much better),
5. pull out the latest code in the target server,
6. compile the code into standalone jar,
7. rebuild the docker image (if necessary),
8. restart the docker container service.


### Hints

- Why use docker?

By employing docker, we can ensure the server environment and the develop
environment stay the same.

- Why compile in the target server?

Because the upload speed in China is painful slow. A better way is to setup a
build server and make it compile the latest codebase automatically.



I will continually update this post when hanging out with clojure. :D


[jvm_hosted]: http://clojure.org/jvm_hosted
[lein]: https://github.com/technomancy/leiningen
[compojure_usage]: https://github.com/weavejester/compojure#usage
[clojure_compile]: http://clojure.org/compilation
[jar_vs_war]: http://stackoverflow.com/questions/5871053/java-war-vs-jar-what-is-the-difference
[gists-as-tricks]: https://github.com/bcho/gists-as-tricks
[sample_fabfile]: https://github.com/bcho/gists-as-tricks/blob/master/fabfile.py
