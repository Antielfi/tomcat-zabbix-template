About:

You can monitor your tomcat with custom settings by using the template below. 

Which template to be used depends on the ObjectName of your Mbean. You can get Mbean data through jconsole of JDK or cmdline-jmxclient-0.10.3.jar(not recommended) after setting up the zabbix-agent has finished.
ObjectName:Catalina:type=ThreadPool,name=ajp-bio-8080 (non-double-quotations) matches zbx_JMX-tomcat-old-version-with-macros_template.xml 
ObjectName:Catalina:type=ThreadPool,name="ajp-bio-8080"(double-quotations) matches zbx_JMX-tomcat-with-macros_template.xml

There are 7 macros defined on the templates. You can add Macros on the hosts(higher priority than the templates) to overwrite it.

{$AJP_CONNECTOR_DESC}:jk(default on tomcat 6?not formirmed)|ajp-bio(default on tomcat 7)|ajp-nio(default on tomcat 8)
#{$AJP_CONNECTOR_PORT}:8009 (default)
{$HTTP_CONNECTOR_DESC}:http-bio|http-nio|http
{$HTTP_CONNECTOR_PORT}:80|8080(default)
{$SSL_CONNECTOR_DESC}:http-bio(default on tomcat 7)
{$SSL_CONNECTOR_PORT}:8443(default)
{$SERVICE_HOST_NAME}:localhost (default)
{$SERVICE_HOST_PATH}:/ (default)

Requirement:
1. Check the Tomcat version:
sh /path/to/your/tomcat/bin/catalina.sh version
2. Get $URL of catalina-jmx-remote.jar that matches tomcat version via http://mvnrepository.com/artifact/org.apache.tomcat/tomcat-catalina-jmx-remote
3. Confirm the IP address of your hosts.(eg:172.27.4.105)
4. Confirm the ProtocolHander of connector (bio|nio|apr)

Getting started:
Setting up the zabbix-agent.

step 1: vim /path/to/your/tomcat/bin/catalina.sh
Replace 172.27.4.105 by your IP and insert these lines before the first non-commented lines of the catalina.sh.
CATALINA_OPTS="-Dcom.sun.management.jmxremote \
               -Dcom.sun.management.jmxremote.authenticate=false \
               -Dcom.sun.management.jmxremote.ssl=false \
               -Djava.rmi.server.hostname=172.27.4.105"

step 2:vim /path/to/your/tomcat/conf/server.xml
Add this line next to the last line of <Listener className=""/>
<Listener className="org.apache.catalina.mbeans.JmxRemoteLifecycleListener" rmiServerPortPlatform="12346" rmiRegistryPortPlatform="12345"/>

step 3 :
Download the matching catalina-jmx-remote.jar $URL for your /path/to/your/tomcat/lib/
wget $URL -O /path/to/your/tomcat/lib/catalina-jmx-remote.jar

step 4 :
Allow access to port 10050,12345,12346 if the firewall is started.

Setting up the zabbix-server.

step 1 : Import the right Template for your zabbix-server web interface. 
step 2 : Add JMX interface:$IP:12345
step 3 : Link the template to the host.
setp 4 : Add Macros on the host if your setting of tomcat does not match the default Macros on the templates.

Reference Documents:
https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/jmx_monitoring
https://www.zabbix.com/documentation/2.4/manual/concepts/java
Monitoring and Management Using JMX:http://docs.oracle.com/javase/1.5.0/docs/guide/management/agent.html
JMX cannot connect through firewalls:https://support.zabbix.com/browse/ZBX-5326
