pipeline{   
    agent any 
    environment {
	    
	    COSIGN_PRIVATE_KEY =credentials('cosign-private-key')
	    COSIGN_PASSWORD=credentials('cosign-password')
    }    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/dmancloud/demo-counter-app.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonarqube-api') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-api'
                    }
                }
            }
            stage('Docker Build & Push') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'dockerhub') {
                            sh "docker build -t dmancloud/demo-counter-app:1.0.0-$BUILD_ID ."
                            sh "docker push dmancloud/demo-counter-app:1.0.0-$BUILD_ID"
                    }
                 }
             }
            }
            stage('sign the container image') {
      		steps {
		    script {
			withDockerRegistry(credentialsId: 'dockerhub') {
        		sh "cosign version"
        		sh "cosign sign --key $COSIGN_PRIVATE_KEY dmancloud/demo-counter-app:1.0.0-$BUILD_ID"
		    }
      }
    }
}	     


		stage("Setting Deepfactor RunToken") {
			steps {
				script {
					env.DF_RUN_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJiY2JlYzA5Yy03ZDhlLTQ4NjMtYjkyZS1mYTkxMjQzYjRhYjciLCJ0b2tlbmlkIjoiZWJiZjBjMDQtNmZlOS00MTE0LTg1ZTQtMjY1MjI2YzkzODM1Iiwic3ViZG9tYWluIjoiZGYiLCJjdXN0b21lcmlkIjoiOTExOWRjM2ItMDU2NC00OTQ5LWE0NGItYjQ4YTRmNWYzODNjIiwidXNlcmxldmVsIjoiQ1VTVE9NRVIiLCJyb2xlaWQiOiJmZDRkYTNhNi1hOTFhLTRmOTQtYjM2ZS1hNWU1ZDY3NThiZDEiLCJyb2xlbmFtZSI6IkNfQURNSU4iLCJ0b2tlbl90eXBlIjoiREZfUlVOX1RPS0VOIiwiZXhwIjoxNzA4OTY0MDAwLCJpYXQiOjE2Nzc0MjgwMDAsIm5iZiI6MTY3NzQyODAwMCwicG9ydGFsVVJMIjoiZGVlcGZhY3Rvci5kZXYuZG1hbi5jbG91ZCIsImN1c3RvbWVyUG9ydGFsVVJMIjoiZGYuZGVlcGZhY3Rvci5kZXYuZG1hbi5jbG91ZCIsInBvcnRhbENBIjoiTUlJRmF6Q0NBMU9nQXdJQkFnSVJBSUlRejdEU1FPTlpSR1BndTJPQ2l3QXdEUVlKS29aSWh2Y05BUUVMQlFBd1xuVHpFTE1Ba0dBMVVFQmhNQ1ZWTXhLVEFuQmdOVkJBb1RJRWx1ZEdWeWJtVjBJRk5sWTNWeWFYUjVJRkpsYzJWaFxuY21Ob0lFZHliM1Z3TVJVd0V3WURWUVFERXd4SlUxSkhJRkp2YjNRZ1dERXdIaGNOTVRVd05qQTBNVEV3TkRNNFxuV2hjTk16VXdOakEwTVRFd05ETTRXakJQTVFzd0NRWURWUVFHRXdKVlV6RXBNQ2NHQTFVRUNoTWdTVzUwWlhKdVxuWlhRZ1UyVmpkWEpwZEhrZ1VtVnpaV0Z5WTJnZ1IzSnZkWEF4RlRBVEJnTlZCQU1UREVsVFVrY2dVbTl2ZENCWVxuTVRDQ0FpSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnSVBBRENDQWdvQ2dnSUJBSzNvSkhQMEZEZnptNTRyVnlnY1xuaDc3Y3Q5ODRrSXh1UE9aWG9IajNkY0tpL3ZWcWJ2WUFUeWpiM21pR2JFU1R0ckZqL1JRU2E3OGYwdW94bXlGK1xuMFRNOHVrajEzWG5mczdqL0V2RWhta3ZCaW9aeGFVcG1abXlQZmp4d3Y2MHBJZ2J6NU1EbWdLN2lTNCszbVg2VVxuQTUvVFI1ZDhtVWdqVStnNHJrOEtiNE11MFVsWGpJQjB0dG92MERpTmV3TndJUnQxOGpBOCtvK3UzZHBqcStzV1xuVDhLT0VVdCt6d3ZvLzdWM0x2U3llMHJnVEJJbERIQ05BeW1nNFZNazdCUFo3aG0vRUxOS2pEK0pvMkZSM3F5SFxuQjVUMFkzSHNMdUp2VzVpQjRZbGNOSGxzZHU4N2tHSjU1dHVrbWk4bXhkQVE0UTdlMlJDT0Z2dTM5NmozeCtVQ1xuQjVpUE5naVY1K0kzbGcwMmRaNzdEbkt4SFp1OEEvbEpCZGlCM1FXMEt0WkI2YXdCZHBVS0Q5amYxYjBTSHpVdlxuS0JkczBwakJxQWxrZDI1SE43ck9yRmxlYUoxL2N0YUp4UVpCS1Q1WlB0MG05U1RKRWFkYW8weEFIMGFobWJXblxuT2xGdWhqdWVmWEtuRWdWNFdlMCtVWGdWQ3dPUGpkQXZCYkkrZTBvY1MzTUZFdnpHNnVCUUUzeERrM1N6eW5Ublxuamg4QkNOQXcxRnR4TnJRSHVzRXdNRnhJdDRJN21LWjlZSXFpb3ltQ3pMcTlnd1Fib29NRFFhSFdCZkVid3Jid1xucUh5R08wYW9TQ3FJM0hhYWRyOGZhcVU5R1kvck9QTmszc2dyRFFvby8vZmI0aFZDMUNMUUoxM2hlZjRZNTNDSVxuclU3bTJZczZ4dDBuVVc3L3ZHVDFNME5QQWdNQkFBR2pRakJBTUE0R0ExVWREd0VCL3dRRUF3SUJCakFQQmdOVlxuSFJNQkFmOEVCVEFEQVFIL01CMEdBMVVkRGdRV0JCUjV0Rm5tZTdibDVBRnpnQWlJeUJwWTl1bWJiakFOQmdrcVxuaGtpRzl3MEJBUXNGQUFPQ0FnRUFWUjlZcWJ5eXFGRFFETEhZR21rZ0p5a0lyR0YxWElwdStJTGxhUy9WOWxaTFxudWJoekVGblRJWmQrNTB4eCs3TFNZSzA1cUF2cUZ5RldoZkZRRGxucnp1Qlo2YnJKRmUrR25ZK0VnUGJrNlpHUVxuM0JlYllodEY4R2FWMG54dnd1bzc3eC9QeTlhdUovR3BzTWl1L1gxK212b2lCT3YvMlgvcWtTc2lzUmNPai9LS1xuTkZ0WTJQd0J5VlM1dUNiTWlvZ3ppVXd0aER5QzMrNldWd1c2TEx2M3hMZkhUanVDdmpISUluTnprdEhDZ0tRNVxuT1JBekk0Sk1QSitHc2xXWUhiNHBob3dpbTU3aWF6dFhPb0p3VGR3Sng0bkxDZ2ROYk9oZGpzbnZ6cXZIdTdVclxuVGtYV1N0QW16T1Z5eWdocXBaWGpGYUgzcE8zSkxGK2wrLytzS0FJdXZ0ZDd1K054ZTVBVzB3ZGVSbE44TndkQ1xuak5QRWxwelZtYlVxNEpVYWdFaXVURGtIenN4SHBGS1ZLN3E0KzYzU00xTjk1UjFOYmRXaHNjZENiK1pBSnpWY1xub3lpM0I0M25qVE9RNXlPZisxQ2NlV3hHMWJRVnM1WnVmcHNNbGpxNFVpMC8xbHZoK3dqQ2hQNGtxS09KMnF4cVxuNFJncXNhaERZVnZUSDl3N2pYYnlMZWlOZGQ4WE0ydzlVL3Q3eTBGZi85eWkwR0U0NFphNHJGMkxOOWQxMVRQQVxubVJHdW5VSEJjbldFdmdKQlFsOW5KRWlVMFpzbnZnYy91YmhQZ1hSUjRYcTM3WjBqNHI3ZzFTZ0VFend4QTU3ZFxuZW15UHhnY1l4bi9lUjQ0L0tKNEVCcytsVkRSM3ZleUptK2tYUTk5YjIxLytqaDVYb3MxQW5YNWlJdHJlR0NjPSJ9.aZPIzvAwfVUWhPD3228vfqPSsRxt7pcGFZ_hnb4D8eU"
				}
			}
		}


        
        stage('Deepfactor Static Scan') {
            steps {
                sh "dfctl scan dmancloud/demo-counter-app:1.0.0-$BUILD_ID"
            }
        }   
        }
        
}
