# The use of the penultimate stable Debian version is required by the project
# Debian over Alpine because it has better MariaDB compatibilty
FROM debian:bullseye-slim

# Maintainer lable is a good practice
LABEL maintainer="arturcarvalho0507@gmail.com"

# Install required packets
# - mariadb-server
# - dumb-init is a lightweight init system designed specifically for containers that:
#   Run as PID 1 and properly handles signals
#   Forwards signals to your application process
#   Reaps zombie processes automatically
#   Is extremely lightweight (minimal overhead)

# We use only one run to save up layers and optimize our image
RUN	apt-get update && \
	apt-get install -y \
	mariadb-server \
	dumb-init \
	rm -rf /var/lib/apt/lists/*	# to clean the image packet list

# Create configuration directory
RUN	mkdir -p /etc/mysql/conf.d

# Copy our custom MariaDB configuration
COPY	./conf/my.conf /ect/mysql/conf.d

# Copy initialization script aka entrypoint script
COPY	./tools/init-db.sh /docker-entrypoint-initdb.d/init-db.sh
RUN	chmod +x /docker-entrypoint-initdb.d/init-db.sh

# Runtime scripts to handle secrets
RUN	mkdir -p /scripts
COPY	./tools/handle-secrets.sh /scritps/
RUN	chmod +x /scripts/handle-secrets.sh

# Create directory for socket and ensure proper permissions
RUN	mkdir -p /run/mysql && \
	chown -R mysql:mysql /run/mysqld

# Set the working directory for MariaDB
WORKDIR	/var/lib/mysql

# Expose MariaDB default port
EXPOSE	3306

# Using dumb-init as and entry point to handle signals 
# Basically tells that the next command will be the main proccess
ENTRYPOINT ["/usr/bin/dumb-init", "--"]

# Start the MariaDB server through our secrets handling script
# The workflow will be:
# 1. dumb-init launches /scripts/handle-secrets.sh
# 2. The script reads Docker secrets from /run/secrets/
# 3. Sets up environment variables and validates configuration
# 4. Finally executes mysqld_safe which:
#    - Starts the actual MariaDB server (mysqld)
#    - Automatically loads configuration from:
#      * /etc/mysql/my.cnf (base config)
#      * /etc/mysql/conf.d/ (your custom configs)
#    - Handles restarts if mysqld crashes
# 5. mysqld_safe replaces the script process (due to 'exec') maintaining proper process hierarchy
CMD ["/scripts/handle-secrets.sh"]
	
