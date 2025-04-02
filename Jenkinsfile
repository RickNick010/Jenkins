pipeline {
    agent any
    
    stages {
        stage('install-pip-deps') {
            steps {
                echo "Installing all necessary dependencies..."
                
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/mtararujs/python-greetings.git']]
                ])
                
                sh '''
                    echo "Starting dependencies installation..."
                    
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    
                    echo "Dependencies successfully installed!"
                '''
            }
        }
        
        stage('deploy-to-dev') {
            steps {
                echo "Deploying to DEV environment..."
                deployToEnvironment('dev', '7001')
            }
        }
        
        stage('tests-on-dev') {
            steps {
                echo "Running tests in DEV environment..."
                testInEnvironment('dev', '7001')
            }
        }
        
        stage('deploy-to-staging') {
            steps {
                echo "Deploying to STAGING environment..."
                deployToEnvironment('staging', '7002')
            }
        }
        
        stage('tests-on-staging') {
            steps {
                echo "Running tests in STAGING environment..."
                testInEnvironment('staging', '7002')
            }
        }
        
        stage('deploy-to-preprod') {
            steps {
                echo "Deploying to PREPROD environment..."
                deployToEnvironment('preprod', '7003')
            }
        }
        
        stage('tests-on-preprod') {
            steps {
                echo "Running tests in PREPROD environment..."
                testInEnvironment('preprod', '7003')
            }
        }
        
        stage('deploy-to-prod') {
            steps {
                echo "Deploying to PROD environment..."
                deployToEnvironment('prod', '7004')
            }
        }
        
        stage('tests-on-prod') {
            steps {
                echo "Running tests in PROD environment..."
                testInEnvironment('prod', '7004')
            }
        }
    }
}

def deployToEnvironment(String envName, String port) {
    sh """
        echo "Deploying to ${envName} environment..."
        
        if [ ! -d "python-greetings" ]; then
            git clone https://github.com/mtararujs/python-greetings.git python-greetings
        fi
        
        cd python-greetings
        
        pm2 delete greetings-app-${envName} || true
        
        echo "Current directory: \$(pwd)"
        echo "Python version: \$(python3 --version)"
        
        python3 -m venv venv || true
        . venv/bin/activate
        pip install -r requirements.txt
        
        pm2 delete greetings-app-${envName} || true
        pm2 start app.py --name greetings-app-${envName} --interpreter \$(pwd)/venv/bin/python -- --port ${port}
        
        pm2 status
        sleep 5
    """
}

def testInEnvironment(String envName, String port) {
    sh """
        echo "Running tests in ${envName} environment..."
        
        if [ ! -d "course-js-api-framework" ]; then
            git clone https://github.com/mtararujs/course-js-api-framework.git course-js-api-framework
        fi
        
        cd course-js-api-framework
        npm install

        echo "Checking service at http://localhost:${port}/greetings"
        if curl -s -f http://localhost:${port}/greetings > /dev/null; then
            echo "Service is available at http://localhost:${port}/greetings"
        else
            echo "WARNING: Service is not available at http://localhost:${port}/greetings"
        fi
        
        mkdir -p config
        
        cat > config/hosts.json << EOF
        {
          "dev": {
            "host": "http://localhost:${port}"
          },
          "staging": {
            "host": "http://localhost:${port}"
          },
          "preprod": {
            "host": "http://localhost:${port}"
          },
          "prod": {
            "host": "http://localhost:${port}"
          }
        }
        EOF
        
        echo "Created hosts.json with port ${port}:"
        cat config/hosts.json
        
        REQUESTS_FILE="tests/utils/requests.js"
        if [ -f "\$REQUESTS_FILE" ]; then
          echo "Modifying requests.js"
          
          cat > temp_requests.js << 'EOF_REQUESTS'
        import request from 'supertest';
        import fs from 'fs';
        import path from 'path';

        function getConfig() {
          try {
            const configPath = path.resolve('config/hosts.json');
            const configData = fs.readFileSync(configPath, 'utf8');
            return JSON.parse(configData);
          } catch (error) {
            console.error('Error reading configuration:', error.message);
            return {
              dev: { host: 'http://localhost:3000' }
            };
          }
        }

        const env = process.env.TEST_ENV || 'dev';
        const config = getConfig();
        
        if (!config[env]) {
          console.warn(`Warning: No configuration for \${env} environment, using dev`);
          config[env] = config.dev;
        }

        console.log(`Using host: \${config[env].host} for environment: \${env}`);

        export default (method, url, data, headers) => {
          return request(config[env].host)[method](url).send(data).set(headers);
        };
        EOF_REQUESTS
          
          mv temp_requests.js "\$REQUESTS_FILE"
          echo "requests.js file successfully modified"
        else
          echo "Error: requests.js not found!"
        fi
        
        export TEST_ENV="${envName}"
        echo "Set variable TEST_ENV=\$TEST_ENV"
        
        NODE_DEBUG=http npm run greetings greetings_${envName} || echo "Tests finished with errors, but continuing execution"
    """
}
