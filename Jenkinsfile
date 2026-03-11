node {
    checkout scm

    // ── STAGE BUILD (sama dengan modul Acara 12) ─────────────────
    stage('Build') {
        docker.image('composer:latest').inside('--network jenkins -u root') {
            sh 'rm -f composer.lock'
            sh 'composer install --ignore-platform-reqs'
            // Note: --ignore-platform-reqs berguna jika container composer
            // punya ekstensi PHP yang berbeda dengan server tujuan.
        }
    }

    // ── STAGE TEST (sama dengan modul) ───────────────────────────
    stage('Test') {
        docker.image('ubuntu').inside('--network jenkins -u root') {
            sh 'echo "Running tests..."'
            // Aktifkan jika sudah ada unit test:
            // sh 'php artisan test'
        }
    }

    // ── STAGE DEPLOY ke PROD ──────────────────────────────────────
    // Di modul  : rsync ke IP VPS (PROD_HOST = IP publik)
    // Di lokal  : rsync ke container prod-app (PROD_HOST = prod-app)
    stage('Deploy') {
        docker.image('agung3wi/alpine-rsync:1.1').inside('--network jenkins -u root') {
            sshagent(credentials: ['ssh-prod']) {
                sh 'mkdir -p ~/.ssh'

                // prod-app = nama Docker container (pengganti IP VPS di modul)
                sh 'ssh-keyscan -H "$PROD_HOST" > ~/.ssh/known_hosts'

                sh """rsync -rav --delete ./ \
                    ubuntu@\$PROD_HOST:/home/ubuntu/prod.kelasdevops.xyz/ \
                    --exclude=.env --exclude=storage --exclude=.git"""
            }
        }
    }
}
