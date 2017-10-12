pipeline {
    agent any 

    stages {
        stage('Prepare') {
            steps {
                sh 'rm -rf app/build/api'
                sh 'rm -rf app/build/code-browser'
                sh 'rm -rf app/build/coverage'
                sh 'rm -rf app/build/logs'
                sh 'rm -rf app/build/pdepend'
                sh 'rm -rf app/build/phpdox'
                sh 'mkdir app/build/api'
                sh 'mkdir app/build/code-browser'
                sh 'mkdir app/build/coverage'
                sh 'mkdir app/build/logs'
                sh 'mkdir app/build/pdepend'
                sh 'mkdir app/build/phpdox'
                sh 'php composer.phar self-update'
                sh 'php composer.phar update'
                sh 'php composer.phar install --dev --prefer-dist --no-progress'
            }
        }
        stage('PHP Syntax check') {
            steps {
                sh 'app/bin/parallel-lint -exclude app --exclude vendor/ .'
            }
        }
        stage('Test'){
            steps {
                sh 'bin/phpunit -c app/phpunit.xml || exit 0'
                step([
                    $class: 'XUnitBuilder',
                    thresholds: [[$class: 'FailedThreshold', unstableThreshold: '1']],
                    tools: [[$class: 'JUnitType', pattern: 'build/logs/phpunit.xml']]
                ])
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'app/build/coverage', reportFiles: 'index.html', reportName: 'Coverage Report', reportTitles: ''])
                /* BROKEN step([$class: 'CloverPublisher', cloverReportDir: 'build/coverage', cloverReportFileName: 'build/logs/clover.xml']) */
                /* BROKEN step([$class: 'hudson.plugins.crap4j.Crap4JPublisher', reportPattern: 'build/logs/crap4j.xml', healthThreshold: '10']) */
            }
        }
        stage('Checkstyle') {
            steps {
                sh 'bin/phpcs --report=checkstyle --report-file=`pwd`/app/build/logs/checkstyle.xml --standard=PSR2 --extensions=php src/ || exit 0'
                checkstyle pattern: 'app/build/logs/checkstyle.xml'
            }
        }
        stage('Lines of Code') {
            steps { sh 'bin/phploc --count-tests --log-csv app/build/logs/phploc.csv --log-xml app/build/logs/phploc.xml src/' }
        }
        stage('Copy paste detection') {
            steps {
                sh 'bin/phpcpd --log-pmd app/build/logs/pmd-cpd.xml src/ || exit 0'
                dry canRunOnFailed: true, pattern: 'build/logs/pmd-cpd.xml'
            }
        }
        /* -- SLOW */
        stage('Mess detection') {
            steps {
                sh 'bin/phpmd src/ xml app/build/phpmd.xml --reportfile app/build/logs/pmd.xml src/ || exit 0'
                pmd canRunOnFailed: true, pattern: 'app/build/logs/pmd.xml'
            }
        }
        /* -- SLOW */
        stage('Software metrics') {
            steps { sh 'bin/pdepend --jdepend-xml=app/build/logs/jdepend.xml --jdepend-chart=app/build/pdepend/dependencies.svg --overview-pyramid=app/build/pdepend/overview-pyramid.svg src/'
            }
        }
        stage('Generate documentation') {
            steps { sh 'bin/phpdox -f build/phpdox.xml'
            }
        }
    }
}
