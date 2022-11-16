# phpVirtualBox - Multiple Server Config

```php
// default config.php
/*
var $servers = array(
    array(
        'name' => 'London',
        'username' => 'user',
        'password' => 'pass',
        'location' => 'http://192.168.1.1:18083/'
    ),
    array(
        'name' => 'New York',
        'username' => 'user2',
        'password' => 'pass2',
        'location' => 'http://192.168.1.2:18083/'
    ),
);
*/
```

example multiple config.php

```php
var $servers = array(
    array(
        'name' => 'London',
        'username' => 'vbox',
        'password' => 'secret1',
        'location' => 'http://10.32.32.4:18083/'
    ),
    array(
        'name' => 'New York',
        'username' => 'vboxadmin',
        'password' => 'secret2',
        'location' => 'http://10.32.45.2:18083/'
    ),
);
```

![image](https://sourceforge.net/p/phpvirtualbox/wiki/Multiple%20Server%20Configuration/attachment/phpvbmulti.png)

multiple server with advanced configuration

```php
var $servers = array(
    array(
        'name' => 'London',
        'username' => 'vbox',
        'password' => 'secret1',
        'location' => 'http://10.32.32.4:18083/',
                'consoleHost' => '10.32.32.4',
                'noPreview' => true,
                'browserRestrictFolders' => array('/home/vbox','/mnt/media')

    ),
    array(
        'name' => 'New York',
        'username' => 'vboxadmin',
        'password' => 'secret2',
        'location' => 'http://10.32.45.2:18083/',
                'consoleHost' => '10.32.45.2',
                'browserRestrictFolders' => array('/home/vboxadmin','/var/isos')
    ),
);
```

# Disable Auth

```php
// config.php
var $noAuth = true;
```

# Password Recovery

rename: `recovery.php-disabled` ke `recovery.php`

[http://host-ip/phpvirtualbox/recovery.php](http://host-ip/phpvirtualbox/recovery.php)

source:
- [https://sourceforge.net/p/phpvirtualbox/wiki/Multiple%20Server%20Configuration/](https://sourceforge.net/p/phpvirtualbox/wiki/Multiple%20Server%20Configuration/)
- [https://sourceforge.net/p/phpvirtualbox/wiki/Authentication%20in%20phpVirtualBox/](https://sourceforge.net/p/phpvirtualbox/wiki/Authentication%20in%20phpVirtualBox/)
