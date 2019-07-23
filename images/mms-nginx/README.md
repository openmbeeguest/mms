Using the Docker Hub official nginx image from https://hub.docker.com/_/nginx

### Purpose
We are using this nginx image as a SSL/TLS termination reverse proxy - handling secured
TCP communications between an OpenMBEE View Editor session and at least 1 MMS service 
endpoint. More service endpoints can be added as needed. It will also redirect
port 80 connections to https on port 443 using the SSL assets described below.
If you have your mms-repo service using a port other than 80, you'll have to 
chance 1 of the new services such that the http port for mms-repo matches
what the nginx.conf is listening on(80). Or you can certainly provide your
own nginx.conf per the notes on the nginx Docker Hub page(it shows a bind
mount providing a host-provided nginx.conf and a derived image providing
a custom nginx.conf).

### Custom deployment data

Currently, we require that the container is started using 4 bind mounts to 
manage what SSL assets are used by nginx (as we are using it to serve as
a SSL/TLS termination reverse proxy for our internal services. While the 
MMS View Editor uses secure websocket (wss) connections, by using this setup
we avoid having to use SSL in each service. Only nginx has to be so 
configured. The current nginx.conf only deals with the ActiveMQ service
(not the # of ActiveMQ ports) that are proxied by nginx)

the 4 bind mounts are:  (these are the docker run options). Comments below 
each -v line.

  -v ${BASE}/private/server.key:/etc/nginx/cert/localhost_openssl_key.pem:ro \
      the host's private RSA key - don't share, nor expose.
      
  -v ${BASE}/server2-Bundled.crt:/etc/nginx/cert/localhost_openssl_cert.crt:ro \
    the host certificate + any intermediate CA certificate + Root CA certificate
    (if & only if you have created your own testing-only Root CA). Don't forget
    to import the Root CA certificate for testing purposes so the server provided
    certifiate can be trusted.
    
  -v ${BASE}/my_cacerts.pem:/etc/nginx/cert/localhost_openssl_cacerts.pem:ro \
    The testing-only Root CA certificate. If this file is empty, the nginx OS 
    Root CA trust store is left as is. Else, this certificate is added to it.
    
  -v ${BASE}/private/my_key_passphrases.txt:/etc/nginx/cert/localhost_openssl_key_passphrases.txt:ro
    if the host's private key was generated with a pass phrase, add that phrase to
    the my_key_passphrases.txt. If not, create that file but leave it empty.

#### Startup order:
  Have ActiveMQ start prior (our docker-compose file reflects this sequencing). Just eliminates some
error messages if the broker is ready when the View Editor wants to connect.

#### nginx.conf:
  The config file has a number of tokens (@@ delimiter) that need to be substituted
at container startup time. That is what the etc/nginx/nginx-entrypoint.sh
script does. That & update the OS Root CA trust store if needed.
Check the nginx.conf - ensure that the various tokens have environment variables set, 
either in the docker-compose file or via the docker run -e option.

required ENV variables:
ACTIVEMQ_NGINX_URL: full http:host-name:port for the VE wss channel 
(eg. http://activemq:61614 - note: use a hostname the Docker container
can use to access the port exposed by the ActiveMQ container), 
ACTIVEMQ_STOMP_PORT, ACTIVEMQ_OPENWIRE_PORT, 
ACTIVEMQ_AMQP_PORT, ACTIVEMQ_MQTT_PORT

#### Note: 
because it appears that the javascript for View Editor has at least 1 place where the
STOMP channel port is hardcoded, it was decided to leave the public facing ports as is, 
changing the ActiveMQ transportConnection URLs to use private ports and having the nginx
proxy_pass directives for each channel use these same private ports. The ActiveMQ container
updates its activemq.conf with these private ports as part of its startup processing.
