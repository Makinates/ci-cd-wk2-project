# Wk2 Task 1 - CI/CD PIPILINE 

## Subtask 1 
A CI/CD (Continuous Integration/ Continuous Deployment) pipeline is a crucial part of modern software development, allowing teams to automate the process of testing, building, and deploying applications. 

Setting up a CI/CD pipeline using GitHub Actions involves creating workflows defined in YAML files within your GitHub repository. 

To configure a CI/CD pipeline for a web application using Github Actions. we are using the RealWorld example apps available on Github. we will use the [React + Node.js RealWorld example app](https://github.com/gothinkster/react-redux-realworld-example-app)
The steps will include setting up the environment, building, testing and deploying the application. 

### Step 1: Clone the Repository  

we will clone the [React + Node.js RealWorld example app](https://github.com/gothinkster/react-redux-realworld-example-app) to our remote machine. 

![Clone the Repo](<Images/SC 04 - Git clone the example app repo.PNG>)

### Step 2: Set up Github Actions Workflow
We will create a GitHub Actions workflow file in our cloned repository. Create the directory 
`.github/workflows` and add a file named `sample-ci-cd-pipeline.yml` 

![Yaml File created](<Images/SC 05 - Created the Yaml File.PNG>)

### Step 3: Define the Workflow 
we will write a CI/CD pipeline configuration for the RealWorld app: 

```yaml 
name: CI/CD Pipeline
'on':
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
      - name: Build application
        run: npm run build
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Deploy to Production
        env:
          SSH_PRIVATE_KEY: '${{ secrets.SSH_PRIVATE_KEY }}'
          HOST: '${{ secrets.HOST }}'
          USERNAME: '${{ secrets.USERNAME }}'
          APP_DIR: '${{ secrets.APP_DIR }}'
        run: >
          echo "$SSH_PRIVATE_KEY" > private_key

          chmod 600 private_key

          rsync -avz -e "ssh -i private_key" --exclude 'node_modules' --exclude
          '.git' . $USERNAME@$HOST:$APP_DIR

          ssh -i private_key $USERNAME@$HOST <<EOF
            cd $APP_DIR
            npm install
            npm run build
            pm2 restart app
          EOF
```
**Workflow Explained**
- **name:** The name of the workflow. 
- **on:** Defines the events that triggers the workflow. In this case, it is triggered on push and pull request events to the `main` branch. 
- **jobs:** Defines the jobs to run as part of the workflow. 

Build Job
- **runs-on:** Specifies the type of runner to use. In this case, `ubuntu-latest`.
-**steps:** A list of steps to run in the job:
    - **Checkout code:** Uses the `actions/checkout` action to checkout the repository code.
    - **Set up Node.js:** Uses the `actions/setup-node` action to set up Node.js. 
    - **Install dependencies:** Runs `npm install` to install Node.js dependencies. 
    - **Run tests:** Runs `npm test` to execute tests. 
    - **Build application:** Runs `npm run build` to build the application. 

 