# wiki.wicket
Jak si nastavit vývojové prostředí pro Java / Wicket vývoj.

viz https://github.com/vrozkovec/wiki.wicket/wiki

# Spuštění ukolovníku

## Předpoklady
- Docker
- Git
- IDE pro Javu
- Java 17 od JetBrains 17.0.9
- hotswap-agent.jar (viz níže)
- .m2 xml soubor pro připojení
- .editorconfig pro správné nastavení formátování kódu nebo jiné adekvátní nastavení!!
- vypnutí imports on the fly

## Postup

1. **Stáhněte komponenty pomocí následujících příkazů:**

    ```bash
    D:\J\Java>git clone git@github.com:vrozkovec/name.berries.base.git
    D:\J\Java>git clone git@github.com:vrozkovec/docker-local-env.git (plus přepnout se do větve minimal_template)
    D:\J\name.berries.base\_projects>git clone git@github.com:vrozkovec/cz.svetit.ukolovnik.git
    ```

2. **Vytvořte db v Dockeru pomocí příkazu:**

    ```bash
    D:\J\docker-local-env>docker-compose up
    ```

3. **Spusťte IDE s name.berries a nastavte verzi Java projektu na JetBrains 17.0.9.**

4. **Stáhněte [HOTSWAP](https://github.com/HotswapProjects/HotswapAgent/releases/download/1.4.2-SNAPSHOT/hotswap-agent-1.4.2-SNAPSHOT.jar), přejmenujte na hotswap-agent.jar** a uložte do složky `C:\Users\CoolMaster\.jdks\jbrsdk-17.0.9\lib\hotswap` (Windows).

5. **Vytvořte si lokální konfiguraci ukolovníku v souboru** `D:\J\Java\name.berries.base\_projects\cz.svetit.ukolovnik\src\main\resources\<appconfig-SVE_JMENO.yml>` **s následujícím obsahem:**

    ```yaml
    app:
       baseUrl: http://localhost:8080
       directories:
          common:
             temp: /tmp
          local:
             projectBase: /speedy/dev/name.berries/_projects/cz.svetit.ukolovnik (ZDE NASTAVIT CESTU K PROJEKTU)
    files:
       folder:
          public: /data/webapps/www/cz.svetit.ukolovnik
          private: /data/webapps/www_internal/cz.svetit.ukolovnik
    ```

6. **Rebuild projektu v Mavenu.**

7. **Spuštění přes:** `D:\J\Java\name.berries.base\_projects\cz.svetit.ukolovnik\src\main\java\cz\svetit\ukolovnik\Server.java`

8. **Nutné nastavit VM Option:**
   ```bash
   -XX:HotswapAgent=fatjar -XX:+AllowEnhancedClassRedefinition -Dport=8080 -DgitBranch=master -Dlocal=yeah -Dprofile=vas_profil -Ddatabase=cz_svetit_ukolovnik

9. **Nastavení code style**
    Nastavit CodeStyle dle návodu u name.berries.

