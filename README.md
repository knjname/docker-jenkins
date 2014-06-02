
# Dockerfile of knjname/jenkins

## How to use simply

```sh
docker run -d -p 8080:8080 knjname/jenkins

# Jenkins will be available at:
#   http://localhost:8080
```

## Detailed usage

### Directory layout

/jenkins/bin
: Contains jenkins.war and run.sh script.
/jenkins/logs
: Contains logs. To persist, you can use like ``` -v /var/log/my_jenkins:/jenkins/logs ```.
/jenkins/home
: This is the ${JENKINS_HOME}, containing configurations, project settings, build histories, plugins and so on. Obviously you're going to have this path persistent by specifying ``` -v /opt/your_host_jenkins_home:/jenkins/home  ```.

### Parameters

Various parameters are available by specifying ```-e ENVNAME=VALUE```.

This functionality is provided by run.sh. run.sh runs Jenkins with various options given from ENVs.

Heres are some of parameters you can specify.

JENKINS_JAVA_OPTIONS
: JVM options which will be passed to JVM on which Jenkins runs. The default value is ```-Djava.awt.headless=true```. If you need to specify memory options such as ```-Xmx1g```, you may set this env as ``` -e JENKINS_JAVA_OPTIONS='-Djava.awt.headless=true -Xmx1g' ```. Many users probably have to specify this.
JENKINS_URL_PREFIX
: Jenkins URL prefix Jetty accepts. For example, if you want to put Jenkins behind a reverse proxy, and the Jenkins has got to accept requests to http://hostname:8080/your-jenkins, you may specify this option as ``` -e JENKINS_URL_PREFIX='/your-jenkins' ```. Please note that in many cases, you have to configure same url prefix additionally on your Jenkins configuration page, which is what Jenkins uses.

To learn more parameters, see [run.sh](run.sh).

For instance, to satisfy following requirements:

 * Need 2GB heap.
 * Nginx redirects /my-jenkins to http://localhost:10080/my-jenkins.
 * Persist log and other data on /opt/my-jenkins.

This ```docker run``` command helps you.

```sh
docker run -d --name my-jenkins \
  -p 10080:8080 \
  -e JENKINS_JAVA_OPTIONS='-Djava.awt.headless=true -Xmx2g' \
  -e JENKINS_URL_PREFIX=/my-jenkins \
  -v /opt/my-jenkins/logs:/jenkins/logs \
  -v /opt/my-jenkins/home:/jenkins/home \
  knjname/jenkins
```


## Recommended Jenkins configuration

Isolate the roles of master-node and slave-nodes explicitly.

 * Do not allow master-node to build. Make master-node unable to build. Master-node should concentrate on keeping build histories, artifacts, and other things master-node only can do.
 * Insteadly, slave-nodes should build everytime. Prepare and connect slave-nodes having plentiful tools to build. Of course when I do that, I setup a SSHD Docker container.

While you keep this rule, you can use simple Jenkins container such like what this repository provides.

