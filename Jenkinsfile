@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {
    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-5-4', type: 'maven'

    stage("Get code") {
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ZEYAD1351/java.git']])
    }

    stage("build app") {
        env.JAVA_HOME = javaHome
        env.PATH = "${javaHome}/bin:${mavenHome}/bin:${env.PATH}"
        sh 'java -version'
        sh 'mvn -version'

        def mavenBuild = new org.iti.mvn()
        mavenBuild.javaBuild("package install")
    }

    stage("archive app") {
        archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
    }

    stage("docker build") {
        def docker = new com.iti.docker()
        docker.build("iti-java", "${BUILD_NUMBER}")
    }

    stage("push docker image") {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
                echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                docker push iti-java:${BUILD_NUMBER}
            """
        }
    }

    stage("update ArgoCD manifest") {
        dir('argocd') {
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ZEYAD1351/argocd.git']])
            sh "sed -i 's#image: .*#image: iti-java:${BUILD_NUMBER}#' iti-dev/deployment.yaml"
        }
    }
}
