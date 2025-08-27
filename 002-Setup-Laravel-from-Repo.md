1) _Prepare the Git & the Git identity_  

        sudo apt update  
        sudo apt install -y git openssh-client  
        git config --global user.name "Noor Amalina"  
        git config --global user.email "youremail@example.com"  
        git config --global init.defaultBranch main  

2) _Create SSH key & link to GitHub_  
    _(recommended to use SSH so not need to re-enter)_
  
        ssh-keygen -t ed25519 -C "youremail@example.com"  


     _(press tekan Enter for default location)_   
     _(set passphrase when requested)_  
  
         eval "$(ssh-agent -s)"
         ssh-add ~/.ssh/id_ed25519

3) _Copy public key_  

          cat ~/.ssh/id_ed25519.pub

     _(Go to  GitHub → Settings → SSH and GPG keys → New SSH key → paste the key from ssh-add ~/.ssh/id_ed25519.)_
     _(Testing the connection:)_
     
           ssh -T git@github.com
  
     _( should get message “Hi <username>! You've successfully authenticated…” )_
4) _Clone Repo_

    _(Select the working folder, then:)_

            mkdir -p ~/work && cd ~/work
            git clone git@github.com:tenexdigital/tariq-core.git
            cd tariq-core

    _(if the SSH access has not been given, the owner of the repo need to set you as a collaborator, or as temporary, clone via HTTPS and login PAT)_


5) _Check the repo & prepare the environment_

            git status
            git remote -v
            ls -la

==================================================================================

            _(from root repo tariq-core)_
            cd ~/work/tariq-core

# 1) Install 'vendor' in container (ignore the Ubuntu platform)

        docker run --rm -u "$(id -u):$(id -g)" -v "$PWD":/app -w /app \
            laravelsail/php84-composer:latest \
            composer install --no-interaction --prefer-dist

# 2) Add Sail

        docker run --rm -u "$(id -u):$(id -g)" -v "$PWD":/app -w /app \
            laravelsail/php84-composer:latest \
            composer require laravel/sail --dev --no-interaction

# 3) Generate Docker Sail file
        docker run --rm -u "$(id -u):$(id -g)" -v "$PWD":/app -w /app \
            laravelsail/php84-composer:latest \
            php artisan sail:install --with=mariadb,redis,phpmyadmin

# 4) Prepare the .env

        [ -f .env ] || cp .env.example .env

# 5) Up the container
        ./vendor/bin/sail up -d –build

_(Open the vcode or any editor and open file docker-compose.yml. Then add below:)_


    # docker-compose.yml
    services:
    # ... servis already in the file (laravel.test, mariadb, redis, dll)

    phpmyadmin:
        image: phpmyadmin/phpmyadmin:latest
        depends_on:
            - mariadb          # guna "mysql" kalau kau guna MySQL
        environment:
            PMA_HOST: mariadb  # tukar ke "mysql" jika perlu
            PMA_USER: "${DB_USERNAME}"
            PMA_PASSWORD: "${DB_PASSWORD}"
            UPLOAD_LIMIT: 128M
        ports:
            - "8081:80"        # akses di http://localhost:8081
        networks:
            - sail


_(Then, again run this:)_

     ./vendor/bin/sail up -d


# 6) Provision dalam Sail

    ./vendor/bin/sail artisan key:generate
    ./vendor/bin/sail artisan migrate
    ./vendor/bin/sail artisan storage:link

# 7) Frontend
    ./vendor/bin/sail npm install  #dia run exactly ikut package.json     OR     ./vendor/bin/sail npm ci  #dia run exactly ikut package-lock.json
    ./vendor/bin/sail npm run dev


Note: 
- Laravel Sail is like a running server in container. So all the command like docker compoase, composer, npm is run in Laravel Sail
