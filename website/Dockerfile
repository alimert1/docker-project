# Use the latest version of the Amazon Linux base image
FROM amazonlinux:2

# Update all installed packages to their latest versions
RUN yum update -y

# Install necessary packages
RUN yum install -y unzip wget httpd git

# Enable PHP 7.4 and install PHP with necessary extensions
RUN amazon-linux-extras enable php7.4 && \
    yum clean metadata && \
    yum install -y \
    php \
    php-common \
    php-pear \
    php-cgi \
    php-curl \
    php-mbstring \
    php-gd \
    php-mysqlnd \
    php-gettext \
    php-json \
    php-xml \
    php-fpm \
    php-intl \
    php-zip

# Install necessary tools and clear YUM cache
RUN yum install -y yum-utils && \
    yum clean all && \
    rm -rf /var/cache/yum

# Download and install the MySQL community server repository
RUN wget https://repo.mysql.com/mysql80-community-release-el7-11.noarch.rpm && \
    rpm -ivh mysql80-community-release-el7-11.noarch.rpm && \
    rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

# Enable the MySQL 8.0 repository explicitly and install MySQL community server
RUN yum-config-manager --enable mysql80-community && \
    yum install -y mysql-community-server


# Change directory to the html directory
WORKDIR /var/www/html

# Set and use environment variables for subsequent commands
ARG PERSONAL_ACCESS_TOKEN=your_token \
    GITHUB_USERNAME=your_username \
    REPOSITORY_NAME=your_repo \
    WEB_FILE_ZIP=your_zip_file \
    WEB_FILE_UNZIP=your_unzip_folder \
    DOMAIN_NAME=your_domain \
    RDS_ENDPOINT=your_rds_endpoint \
    RDS_DB_NAME=your_db_name \
    RDS_MASTER_USERNAME=your_db_username \
    RDS_DB_PASSWORD=your_db_password



# SSH anahtarı kullanarak GitHub deposunu klonlama
RUN mkdir -p /root/.ssh && \
    ssh-keyscan github.com >> /root/.ssh/known_hosts && \
    echo aZOiPZ2nbpdh7mMAyioOdCs96Ta9uegTJbd+AITJFXE > /root/.ssh/id_rsa && \
    chmod 600 /root/.ssh/id_rsa 



# Clone the GitHub repository, unzip web files, and setup the web directory
RUN git clone https://$PERSONAL_ACCESS_TOKEN@github.com/$GITHUB_USERNAME/$REPOSITORY_NAME.git 
RUN unzip $REPOSITORY_NAME/$WEB_FILE_ZIP -d $REPOSITORY_NAME/ 
RUN cp -av $REPOSITORY_NAME/$WEB_FILE_UNZIP/. /var/www/html
RUN rm -rf $REPOSITORY_NAME

# Adjust httpd.conf for mod_rewrite
RUN sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Set permissions on the html directory and storage
RUN chmod -R 777 /var/www/html && \
    chmod -R 777 storage/

# Configure environment variables in .env file
RUN sed -i \
    -e "/^APP_ENV=/ s/=.*$/=production/" \
    -e "/^APP_URL=/ s/=.*$/=https:\/\/$DOMAIN_NAME\//" \
    -e "/^DB_HOST=/ s/=.*$/=$RDS_ENDPOINT/" \
    -e "/^DB_DATABASE=/ s/=.*$/=$RDS_DB_NAME/" \
    -e "/^DB_USERNAME=/ s/=.*$/=$RDS_MASTER_USERNAME/" \
    -e "/^DB_PASSWORD=/ s/=.*$/=$RDS_DB_PASSWORD/" .env

# Copy AppServiceProvider.php into the container
COPY AppServiceProvider.php app/Providers/AppServiceProvider.php

# Expose the default Apache and MySQL ports
EXPOSE 80 3306

# Start Apache in the foreground
ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
