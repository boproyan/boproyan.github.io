+++
title = 'LUA'
date = 2019-09-20 00:00:00 +0100
categories = divers
+++
## LUA

![LUA](Lua-logo-nolabel.svg){:width="150"}    
*Nginx est un serveur HTTP et reverse proxy utilisé par de nombreux sites. OpenResty est une surcouche construite avec de nombreux modules par défaut, ils permettent par exemple la personnalisation via des <u>scripts Lua</u> ou des accès simplifiés à des bases de données.  
Installation nginx+lua avec openresty : [Debian , compilation et installation nginx OU openresty (nginx + lua + openssl TLSv1.3 + modules dynamiques) + PHP7.3 + MariaDb](/posts/Compilation-Nginx(avec-modules-dynamiques)+TLSv1.3+PHP7.3+MariaDB-sur-DebianBuster/)*

### Liens

* [Nginx et Lua, découverte d’OpenResty](https://jolicode.com/blog/nginx-et-lua-decouverte-d-openresty)
* [Supercharging NGINX with Lua (Part 1)](https://blog.cloud66.com/supercharging-nginx-with-lua/)
* [Supercharging NGINX with Lua (Part 2)](https://blog.cloud66.com/supercharging-nginx-with-lua-part-2/)
* [Supercharging NGINX with Lua (Part 3)](https://blog.cloud66.com/supercharging-nginx-with-lua-part-3/)
* [StephenPCG/nginx-lua-simpleauth-module](https://github.com/StephenPCG/nginx-lua-simpleauth-module/blob/master/README.md)
* [Simple authentication on NGINX using LUA / Habrahabr](http://www.techort.com/simple-authentication-on-nginx-using-lua-habrahabr/)

### Tests

Créer un dossier pour les tests lua et un dossier pour les fichiers de configuration

    sudo mkdir /etc/lua # pour le code lua
	#sudo mkdir -p /usr/local/openresty/nginx/sites/cinay.eu.d # sans les liens
	sudo mkdir -p /etc/nginx/conf.d/cinay.eu.d # avec les liens

Le fichier de configuration

	#sudo nano /usr/local/openresty/nginx/sites/cinay.eu.d/lua.conf # sans les liens
	sudo nano /etc/nginx/conf.d/cinay.eu.d/lua.conf # avec les liens

```
    location /example {
         default_type 'text/plain';

         content_by_lua_block {
             ngx.say('Hello, Sammy!')
         }
    }

location /file {
    default_type text/;
    lua_code_cache off; #enables livereload for development
    content_by_lua_file /etc/lua/hello_world.lua;
}
```

Vérification et  rechargement

    sudo openresty -t
    sudo systemctl reload openresty

Tester les liens https://cinay.eu/example et https://cinay.eu/file

### opm

Pour ajouter le module jwt

    sudo -s
    opm get cdbattags/lua-resty-jwt
    opm list

```
cdbattags/lua-resty-jwt                                      0.2.0
jkeys089/lua-resty-hmac                                      0.03
openresty/lua-resty-string                                   0.11
```

Les 3 modules sont utilisés...

### jwt

* [Création token](https://postgrest.org/en/v4.1/tutorials/tut1/)
* https://jwt.io/




Fichier de configuration pour tester jwt

    nano /etc/nginx/conf.d/auth.conf 

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name cinay.eu;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name auth.cinay.eu;

        default_type text/plain;
        location = /verify {
            content_by_lua '
                local cjson = require "cjson"
                local jwt = require "resty.jwt"

                local jwt_token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9" ..
                    ".eyJmb28iOiJiYXIifQ" ..
                    ".VAoRL1IU0nOguxURF2ZcKR0SGKE1gCbqwyh8u2MLAyY"
                local jwt_obj = jwt:verify("lua-resty-jwt", jwt_token)
                ngx.say(cjson.encode(jwt_obj))
            ';
        }
        location = /sign {
            content_by_lua '
                local cjson = require "cjson"
                local jwt = require "resty.jwt"

                local jwt_token = jwt:sign(
                    "lua-resty-jwt",
                    {
                        header={typ="JWT", alg="HS256"},
                        payload={foo="bar"}
                    }
                )
                ngx.say(jwt_token)
            ';
         }

    location = /check {
            content_by_lua '
            local jwt = require "resty.jwt"
            local authentication_token = ngx.var.http_authorization
            -- local secret = require “jwt-secret"
            local secret = "/x6fgevouTqHaNB3GLiTR8ZpIZiKB3yIOia3ZYhW7Kk="
		if jwt:verify(secret, authentication_token).valid ~= true then
		    #return ngx.exit(ngx.HTTP_FORBIDDEN)
		end
            ';
    }
    
    location /example {
         content_by_lua_block {
             ngx.say('Hello, Sammy!')
         }
    }

    location = /authenticate {
        internal;
        # proxy_pass http://flask-app:5000/login;
        proxy_intercept_errors on;
    }

    location = /login {
        # proxy_pass http://flask-app:5000/login;
        rewrite_by_lua_file /etc/nginx/lua/authenticator.lua;
    }

    include ssl_params;
    include header_params;
    # Diffie-Hellmann
    # Uncomment the following directive after DH generation
    # > openssl dhparam -out /etc/ssl/private/dh2048.pem -outform PEM -2 2048
    ssl_dhparam /etc/ssl/private/dh2048.pem;


    access_log /var/log/openresty/auth.cinay.eu-access.log;
    error_log /var/log/openresty/auth.cinay.eu-error.log;

}
```