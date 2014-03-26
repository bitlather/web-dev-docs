XHPROF
======

https://stojg.se/blog/2011-08-27-install-xhprof-for-php5-on-centos-ubuntu-and-debian

Install graphviz:

    # for centos (WORK)
    sudo yum install graphviz
    # for ubuntu and debian
    sudo apt-get install graphviz

Continue:

    sudo pecl config-set preferred_state beta
    sudo pecl install xhprof

See error:

    running: phpize
    Cannot find config.m4.
    Make sure that you run '/usr/bin/phpize' in the top level source directory of the module
    
    ERROR: `phpize' failed

To fix bug:

    # NOTE using 0.9.4 at work
    wget http://pecl.php.net/get/xhprof-0.9.2.tgz
    tar zxf xhprof-0.9.2.tgz
    cd xhprof-0.9.2/extension
    phpize
    ./configure
    make
    sudo make install

Enable it:

    # for centos
    sudo touch /etc/php.d/xhprof.ini
    # for ubuntu and debian
    sudo touch /etc/php5/conf.d/xhprof.ini

Edit config file and write this:

    [xhprof]
    extension=xhprof.so
    xhprof.output_dir="/tmp/xhprof"

Restart apache:

    # for centos
    sudo /etc/init.d/httpd restart
    # for ubuntu and debian
    sudo /etc/init.d/apache2 restart

Clone xhprof repo where you keep all your code:

    git clone https://github.com/facebook/xhprof.git

Put this sample code in your app's index.php; be careful to point $XHPROF_ROOT to the repo you clones:

    <?php
    xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
    for ($i = 0; $i <= 1000; $i++) {
        $a = $i * $i;
    }

    $xhprof_data = xhprof_disable();
    $XHPROF_ROOT = "/data/xhprof/";
    include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_lib.php";
    include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_runs.php";

    $xhprof_runs = new XHProfRuns_Default();
    $run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_testing");
    echo "http://localhost/xhprof/xhprof_html/index.php?run={$run_id}&source=xhprof_testing\n";
    exit();
    ?>

Create a symbolic link in your app code so that xhprof is accessible (alternatively, you could do a vhost, etc):

    cd /webroot/public/yada-yada/
    ln -s /path/to/xhprof/repo xhprof

Now hit the app to generate an xhprof report and follow the link.