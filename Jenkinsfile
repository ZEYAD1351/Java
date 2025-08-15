@Library('iti-sharedlib')_

pipeline {
    agent any
    
    options {
        disableConcurrentBuilds()
    }

    environment {
        JAVA_HOME = tool name: 'java-11', type: 'jdk'
        MAVEN_HOME = tool name: 'mvn-3-5-4', type: 'maven'
        DOCKER_USER = credentials('docker-username')
        DOCKER_PASS = credentials('docker-password')
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage("Get code") {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']], 
                    extensions: [], 
                    userRemoteConfigs: [[url: 'https://github.com/ZEYAD1351/java.git']]
                )
            }
        }
        
        stage("Build app") {
            steps {
                sh 'java -version'
                sh 'mvn -version'
                script {
                    def mavenBuild = new org.iti.mvn()
                    mavenBuild.javaBuild("package install")
                }
            }
        }
        
        stage("Archive app") {
            steps {
                archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
            }
        }
        
        stage("Docker build") {
            steps {
                script {
                    def docker = new com.iti.docker()
                    docker.build("iti-java", "${BUILD_NUMBER}")
                }
            }
        }
        
        stage("Push java app image") {
            steps {
                script {
                    def docker = new com.iti.docker()
                    docker.login("${env.DOCKER_USER}", "${env.DOCKER_PASS}")
                    docker.push("iti-java", "${BUILD_NUMBER}")
                }
            }
        }
        
        stage("Update ArgoCD manifest") {
            steps {
                sh "mkdir -p argocd"
                dir('argocd') {
                    checkout scmGit(
                        branches: [[name: '*/main']], 
                        extensions: [], 
                        userRemoteConfigs: [[url: 'https://github.com/ZEYAD1351/argocd.git']]
                    )
                    sh "sed -i 's#        image: .*#        image: iti-java:${BUILD_NUMBER}#' iti-dev/deployment.yaml"
                }
            }
        }
    }
}
