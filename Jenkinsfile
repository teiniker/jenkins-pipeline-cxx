pipeline 
{
    agent 
    {
        docker { dockerfile true }
    } 
    
    stages 
    {
        stage('build') 
        {
            steps 
            {
                echo 'Build stage: compile all code and build an executable' 
                sh 'cd file_service'
                sh 'mkdir -p build && cd build && cmake .. && make'
            }
        }
        stage('test') 
        {
            steps 
            {
                echo 'Test stage: run the test cases' 
                sh 'cd file_service'
               	sh 'build/test/test'
            }
        }
    }
}
