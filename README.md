Fever RSS Reader tweaked for Heroku/AWS.
========================================

###If you want to run yourself, you'll need to buy a license from Shaun Inman (http://www.feedafever.com/).

This was actually sort of a bitch to set up, so for others looking to jump from Google Reader, here's how I got it working. Documenting for others as well as for myself if I ever need to move/redeploy.

1. Step one is, really decide if you wanna buy this. Read here: (http://feedafever.com).

2. Initialize a Heroku app (http://heroku.com).

    heroku create

3. Add a throwaway php file so that Heroku recognizes it as a PHP app when you push.
    touch index.php

    git add index.php

    git commit -am "Adding index.php"

    git push heroku master


4. Using heroku bash, update the PHP build so it includes Zlib and MBstring.
   heroku run bash
 
    mkdir tmp
    cd tmp
 
    git clone https://github.com/php/php-src.git -b PHP-5.3
 
    cd php-src
    cd ext/zlib
 
    mv config0.m4 config.m4

    /app/php/bin/phpize
 
    ./configure –with-php-config=/app/php/bin/php-config
 
    make
 
    cd ..

    cd mbstring/

    mv config0.m4 config.m4

    /app/php/bin/phpize/

    ./configure -with-php-config=/app/php/bin/php-config
 
    exit

Source: http://hakre.wordpress.com/2012/05/20/php-on-heroku-again/

5. Re-push to Heroku, making a small change on git if needed. You can now navigate to ```boot.php``` and confirm you have all of Fever's requirements set up.

6. Now it's time to set up the database fever will need. Download the RDS command line interface using Mac Homebrew (http://mxcl.github.com/homebrew/). 
    brew install rds-command-line-tools

7. Update your .bashrc or .zshrc with the environment variables Homebrew suggests. Re-source the files with ```source ~.{bash|zsh}rc``` to get them working in your session.

8. Grab your security credentials from AWS (https://portal.aws.amazon.com/gp/aws/securityCredentials); an X.509 certificate and private key. Both have a .pem extension.

9. After downloading, move the key/cert to a .ec2/ folder that you will create within your root directory.

    mkdir ~/.ec2/

    mv ~/{path-to-download}/*.pem ~/.ec2

10. Confirm these credentials are working by running:
    rds-describe-db-instances

This will run silently and complete with no messages assuming you have no databases on AWS RDS yet. Fooled me for quite a while thinking something was wrong.

11. Now, create your AWS database. This can be done via the web interface, but was not updating in real-time with the CLI for me, so I had to start over with the below:
    rds-authorize-db-security-group-ingress default --cidr-ip {your IP address}

    rds-create-db-instance --db-instance-identifier {name}\
      --allocated-storage 5 \
      --db-instance-class db.m1.small  \
      --engine MySQL5.1 \
      --master-username {user} \
      --master-user-password {pw} \
      --db-name {name} \
      --headers

Wait a few minutes for this to create your database (you can check by running ```rds-describe-db-instances``` again).

Then authorize your IP address:
    rds-authorize-db-security-group-ingress default --cidr-ip {your IP address}

Source: http://tjstein.com/2011/09/running-wordpress-on-heroku-and-amazon-rds/

12. Finally, we're ready to get Heroku running on this database. Install the Heroku RDS addon and give it the credentials it will need.

    rds-authorize-db-security-group-ingress default \
      --ec2-security-group-name default \
      --ec2-security-group-owner-id 098166147350

    heroku addons:add amazon_rds --url=mysql2://{db user, from last step}:{db pw, from last step}@{rdshostname}.amazonaws.com/3306:{databasename, from last step}

Source: https://devcenter.heroku.com/articles/amazon_rds

13. After all that, revisit your Heroku app and add the credentials you just added here to the boot.php form.
You'll need:
- Server: the database URL (just the URL, nothing before the @ in the last step, i.e. no user / pw or mysql2://).
- DB name: from your rds-create-db-instance {name}
- Username: same as above, {user}
- Password: same as above, {pw}

And, after all that, you can get your reader back.

C'mon now, Google. That was mean.
