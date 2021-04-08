## DV.Campus 2021 - Magento ##

### Commit message should include:
 - task number: `#2 `
 - short task name: `Add Websites: `
 - what was done: `updated Docker configuration for new domain <DOMAIN_NAME>

**Example:**
```text
#2 Add Websites: updated Docker configuration for new domain <DOMAIN_NAME>
```

### Git flow DV.Campus
![Git workflow DV.Campus](https://i.ibb.co/vXQcmP8/dv-campus-branching-diagram.png)
For more information read this article:
[Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

For all merges on the `master` branch recommended use flag `--no-ff`

**KEEP IN MIND:** All newly created repositories must be configured with following command:
```bash
git config core.fileMode false
```
This configuration **disable** tracking permission changes in your local filesystem and this changes not pushed to
remote repository. 

**Usage example:**
```bash
# Case: We have changes commited on branch `lesson-2-adding-multiple-websites`
git checkout master ; git merge lesson-2-adding-multiple-websites --no-ff
```

### What is Store View?
Website and Store view have own domain name but keep differences.

- Website it is a lot a global think
- Store view it is a most of low-level think

```text
Example: If we need create a different languages it is a good case for make a different store views.
Websites make if we need to create resource with different locations, countries, currencies, prices and more... 
```

### Configuration requirements
Go to **Stores** > **Configuration** > **Web** >> **Base URLs (Secure)** and set **Yes** for 
**Enable HTTP Strict Transport Security (HSTS)** and **Upgrade Insecure Requests**.
These settings most be enabled for all your sites in the future.

### Manual add new website

1. Edit **Base URLs** and **Base URLs (Secure)** in **Stores** > **Configuration** > **Web** of newly add website.
2. Add website hostnames to **/etc/hosts**
3. Edit **ServerAlias** section in your **<YOUR_PROJECT_ROOT_DIR>/docker/virtual-host.conf**
4. Increase certificate counts for given domains in **SSLCertificateFile**, **SSLCertificateKeyFile**
```apacheconf
#    ...
    <IfModule ssl_module>
        SSLEngine on
        SSLCertificateFile /certs/<BASE_DOMAIN_NAME>.local+2.pem
        SSLCertificateKeyFile /certs/<BASE_DOMAIN_NAME>.local+2-key.pem
        SetEnvIf User-Agent ".*MSIE.*" nokeepalive ssl-unclean-shutdown
    </IfModule>
#    ...
```
5. Run next command in your terminal
```bash
mkcert <BASE_DOMAIN_NAME>.local <BASE_DOMAIN_NAME>-<SUBNAME>.local www.<BASE_DOMAIN_NAME>-<SUBNAME>.local
```
6. Edit **extra_hosts** and Hosts in **labels** of Traefik.
7. Execute command from terminal in your project root directory.
```bash
docker-compose down && docker-compose up -d --force-recreate
```
8. Increase certificate counts for your **<BASE_DOMAIN_NAME>** in **.../misc/apps/docker_infrastructure/local_infrastructure/configuration/certificates.toml**
9. Add next configuration to **.htaccess** in your project root directory.
```apacheconf
SetEnvIf Host .*<BASE_DOMAIN_NAME>.local.* MAGE_RUN_CODE=base
SetEnvIf Host .*<BASE_DOMAIN_NAME>.local.* MAGE_RUN_TYPE=website

SetEnvIf Host .*<BASE_DOMAIN_NAME>-<SUBNAME>.local.* MAGE_RUN_CODE=<YOUR_WEBSITE_CODE>
SetEnvIf Host .*<BASE_DOMAIN_NAME>-<SUBNAME>.local.* MAGE_RUN_TYPE=website
```
10. Clear cache, reindex and refresh your site in browser.

### Add new websites automatically
For adding new websites automatically you can use alias ``DOCKERIZE`` in your terminal. 

### Magento deployment modes

 - developer
 - default
 - production
 - maintenance

For switching between modes we can use next command:

```bash
php bin/magento deploy:mode:set <MODE> <OPTIONS>
```

### DOCKERIZER way to adding new environments
**DOKERIZER** can add new environments automatically. Just execute following command for adding environment

```bash
# In this example ENV_NAME is `dev`
php ${PROJECTS_ROOT_DIR}dockerizer_for_php/bin/console env:add <ENV_NAME>
```

### Switch to `dev` environment
This section contains way for switching project environments.
 - After add new environment you must clone repository to new folder like `<YOUR_PROJECT_NAME>-dev.local`
 ```bash
 git clone <REPOSITORY> <FOLDER_NAME>
 ```
 - Change directory, switch to actual branch and run following command for starting Docker instance
 ```bash
 docker-compose -f docker-compose-dev.yml up -d
 ```
 - Check instance after starting
 ```bash
 docker-compose ps
 ```
 - Enter to container using alias `BASH` in your project root directory
 - Create or copy files `auth.json` and `../app/etc/config.php` with your credentials and run `composer install`

### Exclude directories in PhpStorm
You can exclude following directories for speedup of filesystem indexation in PhpStorm.
- `../dev/`
- `../pub/static/`
- `../var/`

### Useful links
 - [Development Infrastructure 4.1](https://docs.google.com/presentation/d/1KFgYiPe5-Acviy083IIA1G5202PkXwJjwCYJv1us7NE)
 - [Lesson 2: Setting up multiple websites within one backend](https://docs.google.com/presentation/d/1R8ZmyVCSiikAM21gEQfH1azMVBs2xwclMarWhKscUQo)
 - [Lesson 3: Application Modes and Pipeline Deployment](https://docs.google.com/presentation/d/1PmmaUUDgPsDGrkCnsjf8z1Ec5m5Iza_8wyMpgYy9NDA)
