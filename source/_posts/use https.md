---
title: https配置
date: 2023-07-04 15:48:46
tags:
---

## apache启用以下模块
```bash
$  apachectl -M

Loaded Modules:
 core_module (static)
 so_module (static)
 watchdog_module (static)
 http_module (static)
 log_config_module (static)
 logio_module (static)
 version_module (static)
 unixd_module (static)
 access_compat_module (shared)
 alias_module (shared)
 auth_basic_module (shared)
 authn_core_module (shared)
 authn_file_module (shared)
 authz_core_module (shared)
 authz_host_module (shared)
 authz_user_module (shared)
 autoindex_module (shared)
 deflate_module (shared)
 dir_module (shared)
 env_module (shared)
 filter_module (shared)
 headers_module (shared)
 mime_module (shared)
 mpm_prefork_module (shared)
 negotiation_module (shared)
 php7_module (shared)
 proxy_module (shared)
 proxy_balancer_module (shared)
 proxy_http_module (shared)
 proxy_wstunnel_module (shared)
 reqtimeout_module (shared)
 rewrite_module (shared)
 setenvif_module (shared)
 slotmem_shm_module (shared)
 socache_shmcb_module (shared)
 ssl_module (shared)
 status_module (shared)
 wsgi_module (shared)
 ```

## 配置默认80端口强制重定向到443

```apache
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /home/ehigh/work/html
	RewriteEngine on
	RewriteCond   %{HTTPS} !=on
	RewriteRule   ^(.*)  https://%{SERVER_NAME}$1 [L,R]
	<Directory /home/ehigh/work/html>
                Options FollowSymLinks MultiViews
                AllowOverride All
		Require all granted
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
RewriteEngine, RewriteCond, RewriteRule


## 配置默认的default-ssl.conf
```apache
<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerAdmin webmaster@localhost

		DocumentRoot /home/ehigh/work/html
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		SSLEngine on
		SSLCertificateFile	/home/ehigh/Downloads/uwb-certs/uwb_cert.pem
		SSLCertificateKeyFile /home/ehigh/Downloads/uwb-certs/uwb_cert.key
		<Directory /home/ehigh/work/html>
			Options FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
	    </Directory>

        # 代理websocket9001,9100,8892,9802, 前端访问wss:127.0.0.1/wss_9001
		SSLProxyEngine on
		ProxyRequests Off
		ProxyPass /wss_9001 ws://127.0.0.1:9001
		ProxyPassReverse /wss_9001 ws://127.0.0.1:9001


		ProxyPass /wss_9100 ws://127.0.0.1:9100
		ProxyPassReverse /wss_9100 ws://127.0.0.1:9100

		ProxyPass /wss_8892 ws://127.0.0.1:8892
		ProxyPassReverse /wss_8892 ws://127.0.0.1:8892

		ProxyPass /wss_9802 ws://127.0.0.1:9802
		ProxyPassReverse /wss_9802 ws://127.0.0.1:9802

		SSLProtocol all -SSLv2 -SSLv3
		SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
		<FilesMatch "\.(cgi|shtml|phtml|php)$">
				SSLOptions +StdEnvVars
		</FilesMatch>
		<Directory /usr/lib/cgi-bin>
				SSLOptions +StdEnvVars
		</Directory>
	</VirtualHost>
</IfModule>
```


## python工程调用接口
跳过验证
```python
import requests
requests.get("https://127.0.0.1/EHCommon/admin/user/login", verify=False)
```
证书验证
```python
import requests
requests.get("https://127.0.0.1/EHCommon/admin/user/login", cert=('/path/server.crt', '/path/server.key')
```

## php工程调用接口

```bash
$ vi work/html/EHCommon/vendor/guzzlehttp/guzzle/src/Client.php
```
跳过验证
```php
    public function __construct(array $config = [])
    {
        if (!isset($config['handler'])) {
            $config['handler'] = HandlerStack::create();
        } elseif (!\is_callable($config['handler'])) {
            throw new InvalidArgumentException('handler must be a callable');
        }

        // Convert the base_uri to a UriInterface
        if (isset($config['base_uri'])) {
            $config['base_uri'] = Psr7\Utils::uriFor($config['base_uri']);
        }
        $config['verify'] = false;

        $this->configureDefaults($config);
    }
```

证书验证
```php
    public function __construct(array $config = [])
    {
        if (!isset($config['handler'])) {
            $config['handler'] = HandlerStack::create();
        } elseif (!\is_callable($config['handler'])) {
            throw new InvalidArgumentException('handler must be a callable');
        }

        // Convert the base_uri to a UriInterface
        if (isset($config['base_uri'])) {
            $config['base_uri'] = Psr7\Utils::uriFor($config['base_uri']);
        }
        $config['verify'] = '/path/to/cert.pem';
        $this->configureDefaults($config);
    }
```
