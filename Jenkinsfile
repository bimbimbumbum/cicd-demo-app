pipeline {
    agent any
    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/bimbimbumbum/cicd-demo-app.git', description: 'Git repository URL')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run unit tests?')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "*/${params.GIT_BRANCH}"]], userRemoteConfigs: [[url: "${params.REPO_URL}"]]
                ])
                script {
                    def commitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    echo "Branch: ${params.GIT_BRANCH}"
                    echo "Commit SHA: ${commitSha}"
                }
            }
        }
        stage('Validate') {
            steps {
                script {
                    def missing = []
                    if (!fileExists('README.md')) missing.add('README.md')
                    if (!fileExists('requirements.txt')) missing.add('requirements.txt')
                    if (!fileExists('src')) missing.add('src/')
                    if (!fileExists('tests')) missing.add('tests/')           
                    if (missing.size() > 0) {
                        error "Отсутствуют обязательные файлы/папки: ${missing.join(', ')}"
                    }
                    echo "Структура проекта в порядке."
                }
            }
        }
        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }
        stage('Build') {
            steps {
                script {
                    def commitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    writeFile file: 'build-info.txt', text: "Build Number: ${env.BUILD_NUMBER}\nCommit SHA: ${commitSha}\n"
                }
                sh 'tar -czf app.tar.gz src/ tests/ README.md requirements.txt build-info.txt'
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }
    post {
        success {
            echo 'Сборка успешно завершена!'
        }
        failure {
            echo 'Сборка завершилась с ошибкой!'
        }
        always {
            echo 'Pipeline завершен.'
        }
    }
}
