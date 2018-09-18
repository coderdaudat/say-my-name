node {

    withMaven(maven:'maven') {

        stage('Checkout') {
            git url: 'https://source.developers.google.com/p/beemob-test/r/micro-test.git', branch: 'staging'
        }

        stage('Build') {
            sh 'mvn clean install'

            def pom = readMavenPom file:'pom.xml'
            print pom.version
            env.version = pom.version
        }

        stage('Image') {
            dir ('customer-service') {
                def app = docker.build "localhost:5000/customer-service:${env.version}"
                app.push()
            }
        }

        stage ('Run') {
            docker.image("localhost:5000/customer-service:${env.version}").run('-p 3333:3333 -h customer --name customer --link discovery --link account')
        }

        stage ('Final') {
            build job: 'gateway-service-pipeline', wait: false
        }


    }

}



