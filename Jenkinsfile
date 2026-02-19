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
