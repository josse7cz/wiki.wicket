# wiki.wicket
Jak si nastavit vývojové prostředí pro Java / Wicket vývoj.

viz https://github.com/vrozkovec/wiki.wicket/wiki

# Spuštění ukolovníku

## Předpoklady
- Docker
- Git
- IDE pro Javu (IntelliJ IDEA)
- Java 17 od JetBrains 17.0.9
- hotswap-agent.jar (viz níže)
- .m2 xml soubor pro připojení
- .editorconfig pro správné nastavení formátování kódu nebo jiné adekvátní nastavení!!
- vypnutí imports on the fly

## Postup

1. **Stáhněte komponenty pomocí následujících příkazů:**

    ```bash
    
    D:\J\Java>git clone git@github.com:vrozkovec/name.berries.base.git
    cd D:\J\Java\name.berries.base\_projects
    D:\J\Java\name.berries.base\_projects>git clone git@github.com:vrozkovec/cz.svetit.ukolovnik.git
    D:\J\Java\name.berries.base\_projects>git clone git@github.com:vrozkovec/cz.svetit.jaclean.git
    
    D:\J>git clone git@github.com:vrozkovec/docker-local-env.git
    cd D:\J>docker-local-env
    D:\J\docker-local-env>git checkout minimal_template
    
    ```

2. **Nastavte a vytvořte kontejnery s databází v Dockeru**

    
docker-compose.yml, přenastavit mapování do místního úložiště. např. v linux:

```
version: '3.8'

services:
    web:
        container_name: dev_nginx
        image: nginx:alpine
        restart: always
        ports:
            - "80:80"
            - "443:443"
        environment:
            - PHPMYADMIN_DOMAIN=phpmyadmin.local
        volumes:
         - ./nginx/sites-enabled:/etc/nginx/templates
         - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
         - ./nginx/dhparam.pem:/etc/nginx/dhparam.pem:ro
         - phpmyadmin_data:/var/www/html/:ro
         - /home/josefjanda/J/Java/name.berries.base/_projects:/projects #nastavit na absolutni cestu k adresáři name.berries/_projects
         - /home/josefjanda/J/temp/cache:/cache #nastavit na absolutni cestu k jakémukoliv adresáří, který bude slouit jako cache
         - /data/webapps/www/cz.svetit.ukolovnik:/webapps #nastavit na absolutni cestu adresáří, který obsahuje soubory nahrané aplikací (/data/webapps na produkci)
        extra_hosts:
            - "host.docker.internal:host-gateway"
        depends_on:
          - db
          - phpmyadmin

    db:
        container_name: dev_mariadb
        image: mariadb:11.2
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: YourPasswd
        ports:
            - 3306:3306
        volumes:
            - ./database-datadir:/var/lib/mysql
            - "./mariadb/conf.d:/etc/mysql/conf.d/:ro"
            - ./mariadb/root-defaults:/home/root/mysql-root-defaults
            - ./mariadb/docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d

    phpmyadmin:
        container_name: dev_phpmyadmin
        image: phpmyadmin:fpm-alpine
        restart: always
        environment:
            PMA_PMADB: phpmyadmin
            UPLOAD_LIMIT: 256M
        volumes:
            - /sessions
            - ./phpmyadmin/config.user.inc.php:/etc/phpmyadmin/config.user.inc.php
            - phpmyadmin_data:/var/www/html/
        ports:
        - 8081:80
        depends_on:
          - db

    redis:
        container_name: dev_redis
        image: redis:alpine
        restart: always
        ports:
            - 6379:6379
        volumes:
            - /var/lib/redis:/data

configs:
    redis_config:
        file: ./redis_config.txt

volumes:
    web_temp:
    phpmyadmin_data:
```

3. **Spustit IDE s name.berries a nastavit verzi Java projektu na jtbr 17.0.9.**

4. **Stáhněte [HOTSWAP](https://github.com/HotswapProjects/HotswapAgent/releases/download/1.4.2-SNAPSHOT/hotswap-agent-1.4.2-SNAPSHOT.jar), přejmenujte na hotswap-agent.jar** a ulote do sloky `~\.jdks\jbrsdk-17.0.9\lib\hotswap` (pro Windows).

5. **Vytvořte si lokální konfiguraci ukolovníku v souboru** 

`~\name.berries.base\_projects\cz.svetit.ukolovnik\src\main\resources\<appconfig-SVE_JMENO.yml>` **s následujícím obsahem:**

```yaml
    app:
       baseUrl: http://localhost:8080
       directories:
          common:
             temp: /tmp
          local:
             projectBase: ~\name.berries\_projects\cz.svetit.ukolovnik (ZDE NASTAVIT CESTU K PROJEKTU [Linux])
    files:
       folder:
          public: \data\webapps\www\cz.svetit.ukolovnik
          private: \data\webapps\www_internal\cz.svetit.ukolovnik
```

```bash
D:\J\docker-local-env>docker-compose up
```
6. **Main class Úkolovník a JaClean:**

`~\name.berries.base\_projects\cz.svetit.ukolovnik\src\main\java\cz\svetit\ukolovnik\Server.java`

`Main class: cz.jaclean.office.Server; Project: cz.svetit.jaclean.office / jaclean-office`


7. **Nutné nastavit VM Option v edit configuration pro každý projekt:**
   ```bash
   -XX:HotswapAgent=fatjar -XX:+AllowEnhancedClassRedefinition -Dport=8080 -DgitBranch=master -Dlocal=yeah -Dprofile=vas_profil -Ddatabase=cz_svetit_ukolovnik
   ```
   ```bash
   -XX:HotswapAgent=fatjar -XX:+AllowEnhancedClassRedefinition -Dport=8080 -DgitBranch=master -Dlocal=yeah -Dprofile=localdev -Ddatabase=cz_jaclean
   ```
8. **Nastavení localhost**

   nginx/phpmyadmin poslouchá na doméně http://phpmyadmin.local/ - nasměrujte si ji na localhost přidáním řádku
   127.0.0.1   phpmyadmin.local do  /etc/hosts (linux, mac) nebo C:\Windows\System32\drivers\etc\hosts

```
#==== phpMyAdmin
127.0.0.1   phpmyadmin.local
#====

#==== Ukolovnik
   127.0.0.1       local.ukol.svetit.cz
   127.0.0.1       local.files.ukol.svetit.cz
   127.0.0.1       local.images.ukol.svetit.cz
#====

#==== JaClean
127.0.0.1       local.jaclean.svetit.eu
127.0.0.1       local.muj.jaclean.svetit.eu
127.0.0.1       local.files.jaclean.svetit.eu
127.0.0.1       local.images.jaclean.svetit.eu
#====
```

9. **Nastavení code style**
   Nastavit CodeStyle dle návodu u repository name.berries.