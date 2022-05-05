# tutorial-baeldung-spring-security-cas-sso
Tutorial: https://www.baeldung.com/spring-security-cas-sso

## Configuration
You can move the configurations with this command (by default it gets it from `cas-server/etc/cas/config`): 
```bash
./gradlew copyCasConfiguration
```
> Note: The specifics of the build are controlled using the `gradle.properties` file.

This is the code that is run (`cas-server/gradle/tasks.gradle`): 
```groovy
task copyCasConfiguration(type: Copy, group: "CAS",
        description: "Copy the CAS configuration from this project to /etc/cas/config") {
    from "etc/cas/config"
    into new File('/etc/cas/config').absolutePath
    doFirst {
        new File('/etc/cas/config').mkdirs()
    }
}
```

## Tutorial
### Basic setup
Add the CAS server: 
```bash
git submodule add https://github.com/apereo/cas-overlay-template.git cas-server
```
In `build.gradle` add in *dependencies*: 
```groovy
implementation "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"
implementation "org.apereo.cas:cas-server-support-jdbc:${project.'cas.version'}"
```
In `cas-server/src/main/resources` add: 
```properties
server.port=8443
spring.main.allow-bean-definition-overriding=true
server.ssl.key-store=classpath:/etc/cas/thekeystore
server.ssl.key-store-password=changeit
```
In `cas-server/src/main/resources`:
```bash
mkdir -p /etc/cas/config
```
In this new created directory `cas-server/src/main/resources/etc/cas/config` (for each question put in **localhost** to avoid SSL handshake error,): 
```bash
keytool -genkey -keyalg RSA -alias thekeystore -keystore thekeystore -storepass changeit -validity 360 -keysize 2048
```
Make sure `echo $JAVA11_HOME` is not empty if it is then to this: 
```bash
export JAVA11_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
```
The next step here is to import `thekeystore` that was just generated (in the tutorial the destination is `$JAVA11_HOME/jre/lib/security/cacerts` but for me there is no jre... you might need to verify. The `security/cacerts` directories should already exist) (FYI: the password it asking is: **changeit**): 
```bash
keytool -importkeystore -srckeystore thekeystore -destkeystore $JAVA11_HOME/lib/security/cacerts
```
Then you can run it: 
```bash
./gradlew run -Dorg.gradle.java.home=$JAVA11_HOME
```
Once it says **READY** you can then go to: https://localhost:8443/
WARNING! By default they are no username or password. You will need to continue to the next section to set that up.
### CAS Server User Configuration
In `cas-server/etc/cas/` add the directory `config`.
```bash
mkdir -p cas-server/etc/cas/config
```
Inside that new directory create: `cas.property` file
In `cas.property` add (this will be a username and password because by default they are none): 
```properties
cas.authn.accept.users=casuser::Mellon
```
```bash
./gradlew run -Dorg.gradle.java.home=$JAVA11_HOME -Pargs="-Dcas.standalone.configurationDirectory=/cas-server/src/main/resources/etc/cas/config"
```
Once **READY** go to : https://localhost:8443

And login with:
- username: casuser
- password: Mellon
### CAS Client Setup

## Troubleshoot
> Note: You can find some interesting details about CAS settings in the `config-metadata.properties` that you can generate: `./gradlew exportConfigMetadata` (Warning! The file is pretty big).
### `cas.authn.pm.json.location:` does not work
You may need to adjust the total number of {@code inotify} instances. On Linux, you may need to add the
following line to `/etc/sysctl.conf`: `fs.inotify.max_user_instances = 256`.

You can check the current value via 
- `cat /proc/sys/fs/inotify/max_user_instances`
- `sudo sysctl -n fs.inotify.max_user_instances`

Change it: 
- `vim /etc/sysctl.conf`
- `sudo sysctl -w fs.inotify.max_user_instances=256`

To reload after you modified in the config file:
- `sudo sysctl -p`

## Sources
- https://www.atlassian.com/git/tutorials/git-submodule
- https://www.baeldung.com/spring-security-cas-sso