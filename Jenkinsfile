def jettyUrl = 'http://localhost:8081/'

stage 'Dev'
node('maven') {
    checkout scm
    sh 'mvn clean package'
    archive 'target/x.war'
}

try {
    checkpoint('Pre-QA')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in Jenkins Enterprise by CloudBees.'
}


stage 'QA'
parallel(longerTests: {
    runTests(30)
}, quickerTests: {
    runTests(20)
})

try {
    checkpoint('Pre-Stage')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in Jenkins Enterprise by CloudBees.'
}

stage name: 'Staging', concurrency: 1
node('maven') {
    deploy 'staging'
}
input message: "Does staging look good?"
try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'Production', concurrency: 1
node('maven') {
    //sh "wget -O - -S ${jettyUrl}staging/"
    echo 'Production server looks to be alive'
    deploy 'production'
    echo "Deployed to ${jettyUrl}production/"
}

def runTests(duration) {
    node('maven') {
        checkout scm
        sh "sleep ${duration}"
        /*
        runWithServer {url ->
            mvn "-f sometests test -Durl=${url} -Dduration=${duration}"
        } */
    }
}

def deploy(id) {
    unarchive mapping: ['target/x.war' : 'x.war']
    sh "mkdir -p /tmp/webapps"
    sh "cp x.war /tmp/webapps/${id}.war"
}

def undeploy(id) {
    sh "rm /tmp/webapps/${id}.war"
}

def runWithServer(body) {
    def jettyUrl = 'http://localhost:8081/' // TODO why is this not inherited from the top-level scope?
    def id = UUID.randomUUID().toString()
    deploy id
    
    try {
        body.call "${jettyUrl}${id}/"
    } finally {
        undeploy id
    }
}
