### Building the mms-activemq Docker image

#### assets/

As far as I can tell, the out-of-the-box configuration for 
activeMQ (conf/activemq.xml) is what is needed by the View Editor,
MMS services.
In our environment, we are using nginx as a SSL/TLS terminator 
reverse proxy gateway, sitting between a user's browser running the 
View Editor and the various backend docker services, including
activemq. Because the View Editor javascript expects various service
endpoints to use SSL connections, we will use nginx to handle the client
(eg. View Editor) SSL connections to the endpoints. In reality,
these SSL connections terminate with nginx, nginx will proxy the
connections to the non-SSL endpoints. This makes the total 
configuration of the services easier, also allowing testing of the
services to be less complicated.

As it appears that the VE javascript has hardcoded the activemq
endpoint ports (eg. 61614), it is easier to have nginx listening
to the default activemq ports, and change the activemq.xml config file 
to use different ports. This way, the clients appear to be using 
the defaults, while the 1 nginx.conf reflects the logical redirect
 (via the use of the ngx_http_proxy_module).
 As the ports can change per deployment (who knows what other 
 processes may be using the host ports?), the assets/update_properties.sh
 script is used at container startup to update the conf/activemq.xml 
 config file.
 
 The update_properties.sh script assumes (by design) that there are a number
 of environment variables set by the calling environment. Could be Docker Compose files,
 could be docker run -e values, or some other mechanism. Flexibility is the
 name of the game.

NOTE: make sure the ACTIVEMQ_WS_PORT matches the mms-repo's update_properties.sh
ACTIVEMQ_JMS_PORT - 2 (eg. if ACTIVE_WS_PORT == 62614 then ACTIVE_JMS_PORT := 62616)
mms-repo/assets/mms.properties uses the ACTIVE_WS_PORT when defining the 
jms.uri property.  As the hostname (via the ACTIVEMQ_EXTERNAL_FQDN env var) isn't
known until startup time, this same env var mechanism is used.
