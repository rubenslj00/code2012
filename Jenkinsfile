pipeline {
    options {
      timeout(time: 1, unit: 'HOURS') 
  }
  agent {
    docker {
      image 'hashmapinc/sqitch:jenkins'
      args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
    }
  }
  stages {
    stage('Moving .snowsql to workspace and replacing snowsql in /bin') {
        steps {
            sh '''
            rm /bin/snowsql 
            mv /var/snowsql /bin/
            mv /var/.snowsql ./
            ''' 
        }
    }
    stage('Deploy changes') {
      steps {
        withCredentials(bindings: [usernamePassword(credentialsId: 'snowflake_creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh '''
              sqitch deploy "db:snowflake://$USERNAME:$PASSWORD@@moffitt.us-east-1.privatelink.snowflakecomputing.com/mcap_dev?Driver=SnowflakeDSIIDriver;warehouse=ETL_DEV"
              '''           
        }
      }
    }
      stage('Verify changes') {
      steps {
        withCredentials(bindings: [usernamePassword(credentialsId: 'snowflake_creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh '''
              sqitch verify "db:snowflake://$USERNAME:$PASSWORD@@moffitt.us-east-1.privatelink.snowflakecomputing.com/mcap_dev?Driver=SnowflakeDSIIDriver;warehouse=ETL_DEV"
              ''' 
        }
      }
    }
  }
  post {
    always {
      sh 'chmod -R 777 .'
    }
  }
}
