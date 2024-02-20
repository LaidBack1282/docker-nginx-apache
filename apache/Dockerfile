FROM php:apache
RUN echo "Listen 8083" > /etc/apache2/ports.conf
RUN echo "<?php phpinfo(); ?>" > /var/www/html/index.php
CMD ["apache2-foreground"]