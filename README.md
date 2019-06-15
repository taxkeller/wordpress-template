# wordpress-template

### How to setup (CentOS)

1. Install docker

```
$ sudo yum install docker
$ sudo curl -L https://github.com/docker/compose/releases/download/1.25.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo service docker start
```

Check the latest version (https://github.com/docker/compose/releases)

2. Modify group

```
$ sudo vim /etc/group
-----

dockerroot:x:994:(add username)
```

3. Modify permission

```
$ sudo chgrp dockerroot /var/run/docker.sock
```


4. Download wordpress

```
$ wget https://ja.wordpress.org/latest-ja.tar.gz
$ tar -zxvf latest-ja.tar.gz
$ mv wordpress site1
```

5. Modify UID and GID in docker-compose.yml

```
$ vim docker-compose.yml
-----

  wpsite1:
    image: wordpress:5.2.1-php7.1-fpm-alpine
    user: '{UID}:{GID}' # modify
    restart: always
    volumes:
      - ./site1:/var/www/html/site1:z
    environment:
      WORDPRESS_SMTP_HOST: smtp.sendgrid.net
      WORDPRESS_SMTP_PORT: 2525
      WORDPRESS_SMTP_USERNAME: '${SendGrid_User}'       # modify
      WORDPRESS_SMTP_PASSWORD: '${SendGrid_Pass}'       # modify
      WORDPRESS_SMTP_FROM: '${SendGrid_FromMail}'       # modify
      WORDPRESS_SMTP_FROM_NAME: '${SendGrid_FromName}'  # modify

```

6. Start docker-compsose

```
$ docker-compose up -d --build
```

7. After install wordpress

modify wp-config.php

```
# Add these lines
add_action( 'phpmailer_init', 'mail_smtp' );
function mail_smtp( $phpmailer ) {
  $phpmailer->isSMTP();
  $phpmailer->Host = getenv('WORDPRESS_SMTP_HOST');
  $phpmailer->SMTPAutoTLS = true;
  $phpmailer->SMTPAuth = true;
  $phpmailer->Port = getenv('WORDPRESS_SMTP_PORT');
  $phpmailer->Username = getenv('WORDPRESS_SMTP_USERNAME');
  $phpmailer->Password = getenv('WORDPRESS_SMTP_PASSWORD');

  // Additional settings
  $phpmailer->SMTPSecure = "tls";
  $phpmailer->From = getenv('WORDPRESS_SMTP_FROM');
  $phpmailer->FromName = getenv('WORDPRESS_SMTP_FROM_NAME');
}
```
