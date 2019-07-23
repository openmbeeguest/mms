### Details on building, deploying the mms-repo Docker image


#### assets/

assets/update_properties.sh is used to define at container startup various

The update_properties.sh script assumes (by design) that there are a number
of environment variables set by the calling environment. Could be Docker Compose files,
could be docker run -e values, or some other mechanism. Flexibility is the
name of the game.

NOTE: mms-repo/assets/mms.properties uses the ACTIVE_WS_PORT when defining the 
jms.uri property.  As the jms.uri hostname (via the ACTIVEMQ_EXTERNAL_FQDN env var) isn't
known until startup time, this same env var mechanism is used.

This is needed because the internal Docker network hostname can't be used by
MMS as the activemq endpoint because that hostname won't resolve in the browsers
running View Editor. So we use the host machine's fqdn value to define this 
jms.uri property. At container startup time.