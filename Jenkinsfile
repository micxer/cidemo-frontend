#!/usr/bin/env groovy

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    parameters {
        string( name: 'CF_API',
                defaultValue: 'https://api.local.pcfdev.io',
                description: 'Cloud Foundry API url')

        string( name: 'CF_BASE_HOST',
                defaultValue: 'local.pcfdev.io',
                description: 'Base host for CF apps')

        string( name: 'CF_ORG',
                defaultValue: 'pcfdev-org',
                description: 'Cloud Foundry Org')

        string( name: 'CF_SPACE',
                defaultValue: 'pcfdev-space',
                description: 'Cloud Foundry Space')

        string( name: 'SELENIUM_HUB_URL',
                defaultValue: 'http://192.168.2.100:4444/wd/hub',
                description: 'Selenium Hub Address')
    }

    tools {
        nodejs 'node-7'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('NPM Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Lint Code') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm run test:headless'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build:prod'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'pcf', usernameVariable: 'CF_USER', passwordVariable: 'CF_PASSWORD')]) {
                        sh "cf login -a ${params.CF_API} -u $CF_USER -p $CF_PASSWORD -o ${params.CF_ORG} -s ${params.CF_SPACE}  --skip-ssl-validation"
                        sh 'cf push'
                    }
                }
            }
        }

        stage('E2E Tests') {
            steps {
                script {
                    def appInfo = readYaml file: './manifest.yml'
                    def appName = appInfo.applications[0].name
                    def appUrl = "https://${appName}.${params.CF_BASE_HOST}/"
                    withEnv(["APP_BASE_URL=${appUrl}", "SELENIUM_HUB_URL=${params.SELENIUM_HUB_URL}"]) {
                        sh 'npm run e2e:prod'
                    }
                }
            }
        }
    }
}
