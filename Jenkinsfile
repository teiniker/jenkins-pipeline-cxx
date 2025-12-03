pipeline 
{
    agent { dockerfile true }
    
    stages 
    {
        stage('build') 
        {
            steps 
            {
                echo 'Build stage: compile all code and build an executable' 
                dir('file_service') 
                {
                    sh 'mkdir -p build && cd build && cmake .. && make'
                }
            }
        }
        stage('test') 
        {
            steps 
            {
                echo 'Test stage: run the test cases' 
                dir('file_service/build') 
                {
               	    sh './test/test'
                }
            }
        }
    }
    post 
    {
        success 
        {
            echo 'Archive build artifacts'
            sh '''
                mkdir -p artifacts
                cp file_service/build/test/test artifacts/ 
            '''
            archiveArtifacts artifacts: 'artifacts/*', fingerprint: true
        }
    }
}
