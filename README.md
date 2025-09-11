CI/CD pipeline using GitHub Actions and deploy a Python Flask application to Azure App Service. This procedure combines the best practices, end-to-end process. 🚀

Step 1: Project Setup and Code Push
Create Your Local Flask App: On your local machine, create a project folder with two files:

app.py: Your Flask application code.

Python

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, this is a Flask app deployed via GitHub Actions!"

if __name__ == '__main__':
    app.run()
requirements.txt: Lists the necessary Python packages.

flask
gunicorn
Initialize Git and Push to GitHub:

Create a new, empty repository on GitHub.

In your local project folder, open a terminal and run the following commands to connect your local code to the new GitHub repo:

Bash

git init
git add .
git commit -m "Initial commit of Flask app"
git remote add origin <your-github-repo-url>
git push -u origin main
Make sure your GitHub repository's default branch is main to match the push command.

Step 2: Azure App Service Configuration
Create the App Service:

Go to the Azure Portal and create a new App Service.

Choose Code for publishing, select Python 3.10 as the runtime stack, and set the operating system to Linux. Give it a unique name (e.g., my-flask-ci-cd-123).

Configure Startup Command:

Once the App Service is created, go to Configuration > General settings.

In the Startup Command box, enter:

gunicorn --bind 0.0.0.0:8000 app:app
Click Save. This command tells Azure how to run your app with the production server, Gunicorn.

Step 3: Securely Connect GitHub and Azure
Get the Publish Profile:

In the Azure Portal, go to your App Service's Overview page.

Click the Get publish profile button.

If you encounter the "Basic authentication is disabled" error, you must temporarily enable it. Go to Configuration > General settings and set SCM Basic Auth Publishing Credentials to On. Remember to turn this off in a production environment for security.

Create a GitHub Secret:

Go to your GitHub repository and navigate to Settings > Secrets and variables > Actions.

Click New repository secret.

Name: AZURE_WEBAPP_PUBLISH_PROFILE

Secret: Open the .publishsettings file you downloaded and copy/paste its entire content into this box.

Click Add secret.

Step 4: Build and Deploy with GitHub Actions
Create the Workflow File: In your local project, create the folder structure .github/workflows.

Add main.yml: Inside the workflows folder, create a file named main.yml and paste the following code.

YAML

name: Build and Deploy Python App to Azure Web App

on:
  push:
    branches:
      - main

env:
  AZURE_WEBAPP_NAME: 'your-app-service-name'  # Replace with your App Service name

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout the repository code
      - uses: actions/checkout@v4

      # Step 2: Set up Python
      - name: Set up Python 3.10
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      # Step 3: Install dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # Step 4: Deploy to Azure Web App
      - name: 'Deploy to Azure Web App'
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
Commit and Push:

Update the AZURE_WEBAPP_NAME in the YAML file with your App Service's name.

Save the file, then commit and push it to GitHub.

Bash

git add .github/workflows/main.yml
git commit -m "Add GitHub Actions workflow"
git push
Final Understanding
Once you push the main.yml file, the workflow will be automatically triggered. You can watch the progress under the Actions tab in your GitHub repo. The workflow will run the steps you defined, which include installing dependencies and then using the publish profile secret to securely deploy your code directly to Azure App Service. This seamless process demonstrates the power of CI/CD. ✨
