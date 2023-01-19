node {
    stage('Clone repository') {
    checkout scm
    }
    stage('OWASP Dependency-Check Vulnerabilities ') {
        dependencyCheck additionalArguments: '''
		-s "." 
		-f "ALL"
		-o "./report/"
		--prettyPrint
		--disableYarnAudit''', odcInstallation: 'OWASP Dependency-check'
		dependencyCheckPublisher pattern: 'report/dependency-check-report.xml'
    }
    stage('SonarQube analysis') {
        def scannerHome = tool 'sonarqube';
        withSonarQubeEnv('sonarserver'){
                sh "${scannerHome}/bin/sonar-scanner     -Dsonar.projectKey=helloworld     -Dsonar.host.url=http://192.168.160.244:9000     -Dsonar.login=8ad8f5347dd113510994c0c814120afff3aa12ab     -Dsonar.sources=.     -Dsonar.report.export.path=sonar-report.json     -Dsonar.exclusions=report/*     -Dsonar.dependencyCheck.jsonReportPath=./report/dependency-check-report.json     -Dsonar.dependencyCheck.xmlReportPath=./report/dependency-check-report.xml     -Dsonar.dependencyCheck.htmlReportPath=./report/dependency-check-report.html"
        }
    }
    stage('SonarQube Quality Gate'){
        timeout(time: 1, unit: 'HOURS') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        
        }
    }
    stage('Build image') {
        app = docker.build("sjin1105/helloworld", "--network host .")
    }
    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker') {
        app.push("$BUILD_NUMBER")
        app.push('latest')
        }
    }
}
        