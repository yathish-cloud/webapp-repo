pipeline {
    agent none
    stages {      
        stage('FETCH-CODE') {
             agent { label 'master'}
             steps {                       
                        git branch: 'main', credentialsId: 'git_cred', url: 'https://github.com/yathish-cloud/tomcat-dep-java.git'
						sh "ls -l"
                   }
		}
		stage('BUILD') {
			agent { label 'master'}
            steps {
                sh'''
                mvn --version
                stat=$?
                if [ $stat -eq 0 ]
                then
                    echo " test = stat"                   
                    mvn clean install
                else
                    sudo yum maven install                    
                    mvn clean install
                fi
                sh'''
            }
        }
		stage('COPY-WAR-ARTIFACTORY') {	   
		agent { label 'master'}
            steps {
                    withCredentials([usernamePassword(credentialsId: 'git_cred', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')])
                     {
                        sh'''         
                        rm -rf webapp-repo
                        git clone https://github.com/yathish-cloud/webapp-repo.git
                       
                        cp  target/*.war  webapp-repo 
                       
                        cd  webapp-repo/ 
                        git add *.war .
                        git commit -m "addingWar"                       
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yathish-cloud/webapp-repo.git --all
                          
                         '''  
                      }                                              
                  }
        }
        stage('DEPLOY-IN-SLAVE') {
        agent {label 'slave1'}
            steps {
                    sh 'ls -la'
                    git branch: 'master', credentialsId: 'git_cred', url: 'https://github.com/yathish-cloud/webapp-repo.git'
					
                  sh'''
				     docker build -t sample_webapp .
				     docker run -it -d -p 8082:8080 sample_webapp
                   '''
                   }
            }           
    }
}
