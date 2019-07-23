### Purpose of the deployment directory

scripts, etc. needed to facilitate deployment or run-time functionality.

For example, the mms-repo container has to modify the jms.uri property in the
mms.properties file based on run-time (startup is enough) environment. So we 
use the Docker mechanisms involving environment variables. To reduce the chance of
a manual start forgetting to set the needed variable (or just a mistake in doing so),
we've automated this by using a profile script and the .env file used by Docker to bootstrap
such environment variables. Whenever an interactive login session is initiated and the 
composer-vars-profile.sh is deployed to the /etc/profile.d/ directory, that session
will have the needed variable correctly set. No need to remember to do it.

For automated invocations (eg. the deployment/systemd/mms.service definition) the 
updateDockerEnv.sh script is used by the mms.service to add/update the .env's needed
ACTIVEMQ_EXTERNAL_FQDN variable so the docker-compose process will have it defined, hence
propogating that variable per the compose file's ENVIRONMENT directive.

### deployment targets

assets/deployment/composer-vars-profile.sh -> /etc/profile.d/  with read permissions enabled.
assets/deployment/systemd/updateDockerEnv.sh > /usr/local/docker-dir/  with execute permission enable
assets/deployment/system/mms.service  ->  /etc/systemd/system/ with read permission set