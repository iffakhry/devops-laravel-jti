## Notes

## untuk proses demo jenkins di local komputer (tanpa vps)

isi Jenkinsfile:

```jenkinsfile
node {
    checkout scm

    // ── STAGE BUILD (sama dengan modul Acara 12) ─────────────────
    docker.image('composer:latest').inside('-u root') {
            sh 'rm -f composer.lock'
            sh 'composer install --ignore-platform-reqs'
            // Note: --ignore-platform-reqs berguna jika container composer
            // punya ekstensi PHP yang berbeda dengan server tujuan.
        }

    // ── STAGE TEST (sama dengan modul) ───────────────────────────
    stage('Test') {
        docker.image('ubuntu').inside('-u root') {
            sh 'echo "Running tests..."'
            // Aktifkan jika sudah ada unit test:
            // sh 'php artisan test'
        }
    }

    // ── STAGE DEPLOY ke PROD ──────────────────────────────────────
    // Di modul  : rsync ke IP VPS (PROD_HOST = IP publik)
    // Di lokal  : rsync ke container prod-app (PROD_HOST = prod-app)
    stage('Deploy') {
        docker.image('agung3wi/alpine-rsync:1.1').inside('-u root') {
            sshagent(credentials: ['ssh-prod']) {
                sh 'mkdir -p ~/.ssh'

                // prod-app = nama Docker container (pengganti IP VPS di modul)
                sh 'ssh-keyscan -H "$PROD_HOST" > ~/.ssh/known_hosts'

                sh """rsync -rav --delete ./laravel/ \
                    ubuntu@\$PROD_HOST:/home/ubuntu/prod.kelasdevops.xyz/ \
                    --exclude=.env --exclude=storage --exclude=.git"""
            }
        }
    }
}
```

### untuk proses demo jenkins di vps

isi Jenkinsfile:

```jenkinsfile
node {
    checkout scm
    // deploy env dev
    stage("Build"){
        // docker.image('shippingdocker/php-composer:7.4').inside('-u root') {
        //     sh 'rm composer.lock'
        //     sh 'composer install'
        // }
        docker.image('composer:latest').inside('-u root') {
            sh 'rm -f composer.lock'
            sh 'composer install --ignore-platform-reqs'
            // Note: --ignore-platform-reqs berguna jika container composer
            // punya ekstensi PHP yang berbeda dengan server tujuan.
        }
    }
    stage("Prepare Laravel"){
        // Gunakan image PHP 8.4 untuk menjalankan artisan
        docker.image('php:8.4-cli').inside('-u root') {
            sh 'cp .env.example .env'
            sh 'php artisan key:generate'
            sh 'chmod -R 777 storage bootstrap/cache'
        }
    }
    // Testing
    stage("Testing"){
        docker.image('ubuntu').inside('-u root') {
            sh 'echo "Ini adalah test"'
        }
    }
    // deploy env prod
    stage("Deploy"){
        // Di sini Anda bisa menambahkan langkah untuk mengirim file ke server produksi
        // Misalnya menggunakan scp atau rsync
        sh 'echo "Deploying to production server..."'
        docker.image('agung3wi/alpine-rsync:1.1').inside('-u root') {
            sshagent (credentials: ['ssh-prod']) {
                sh 'mkdir -p ~/.ssh'
                sh 'ssh-keyscan -H "$PROD_HOST" > ~/.ssh/known_hosts'
                sh """
                    rsync -rav --delete ./ \
                    dosen1@$PROD_HOST:/home/dosen1/prod.kelasdevops.xyz/ \
                    --exclude=.env \
                    --exclude=storage \
                    --exclude=.git
                """
            }
        }
    }
}

```
