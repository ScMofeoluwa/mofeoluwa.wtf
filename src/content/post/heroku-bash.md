---
title: "Heroku-Bash"
description: "Automating Heroku Deployment with Bash Scripts"
publishDate: "22 May 2023"
tags: ["bash", "heroku"]
---

At times, deploying to Heroku can be a tedious and error-prone process.
Manually typing commands and overlooking branch switching can lead to unnecessary hassle and mistakes.
To address these challenges, I decided to automate the deployment workflow using bash scripts.
I reached out to a friend who recommended a useful resource to learn bash scripting.
Armed with this newfound knowledge, I set out to create a bash script that simplifies and automates Heroku deployment.
Let's explore the script below.

## Before we begin

Before we dive into the script, make sure you have created a Heroku app and connected it to your repository before running the script.

Useful Links:

- Bash: [Learn X in Y minutes](https://learnxinyminutes.com/docs/bash/)
- Heroku: [Heroku CLI Commands](https://devcenter.heroku.com/articles/heroku-cli-commands)

## Automating your Heroku Deployment

Now, let's take a closer look at the steps involved in automating your Heroku deployment:

### Step 1: Checking Heroku CLI Installation

The first thing we need to do is check if the Heroku CLI (Command Line Interface) is installed on your system.
The CLI allows us to interact with Heroku from the command line. We can use the `which command` to verify if the Heroku CLI is installed:

```sh
which heroku >/dev/null
if [ $? -ne 0 ]; then
    echo "Heroku CLI is not installed. Please install it first."
    exit 1
fi
```

If the Heroku CLI is not installed, the script will display an error message and exit.

### Step 2: Checking Heroku Login

Next, we want to ensure that the user is logged in to Heroku.
This can be done by checking the output of the `heroku whoami` command:

```sh
heroku whoami >/dev/null
if [ $? -ne 0 ]; then
    echo "Not logged in to Heroku. Please log in."
    exit 1
fi
```

If the user is not logged in, the script will prompt them to log in and exit.

### Step 3: Switching to the Master Branch

To avoid any potential issues, we want to make sure that we are on the master branch of our repository.
We can use the `git checkout` command to switch to the master branch:

```sh
git checkout master
```

### Step 4: Creating and Checking Out the Deploy Branch

Now, we need to check if the deploy branch exists.
If it doesn't, we will create it using the `git checkout -b` command:

```sh
git rev-parse --verify deploy >/dev/null 2>&1
if [ $? -ne 0 ]; then
    git checkout -b deploy
fi
git checkout deploy
if [ $? -ne 0 ]; then
    echo "Failed to checkout to the 'deploy' branch. Please check your repository."
    exit 1
fi
```

These commands ensure that we are on the deploy branch for our Heroku deployment.

### Step 5: Merging from the Master Branch

Before deploying to Heroku, we want to merge any changes from the master branch to the deploy branch.
This can be done using the `git merge` command:

```sh
git merge master
if [ $? -ne 0 ]; then
    echo "Failed to merge from 'master'. Please check your repository."
    exit 1
fi
```

If the merge fails, the script will display an error message and exit.

### Step 6: Checking for Merge Conflicts

After the merge, we need to check if there are any merge conflicts.
We can accomplish this by using the `git ls-files -u` command, which lists all the unmerged files,
and then counting the number of lines with conflict issues using `wc -l`:

```sh
CONFLICTS=$(git ls-files -u | wc -l)
if [ "$CONFLICTS" -gt 0 ]; then
    echo "There are merge conflicts. Please resolve them manually."
    exit 1
fi
```

If there are any merge conflicts, the script will notify you to resolve them manually and exit.

### Step 7: Deploying to Heroku

Assuming there are no merge conflicts, we can proceed with deploying our project to Heroku.
This can be achieved using the `git push` command:

```sh
git push heroku deploy:master
if [ $? -ne 0 ]; then
    echo "Failed to push to Heroku. Please check your Heroku settings."
    exit 1
fi
```

If the push to Heroku fails, the script will display an error message and exit.

### Step 8: Switching Back to the Master Branch

Once the deployment is complete, we want to switch back to the master branch.
We can do this using the `git checkout` command:

```sh
git checkout master
```

If the switch back to the master branch fails, the script will display an error message and exit.

### Side Notes

In addition to the script itself, there are a few important notes worth considering:

1. **Data Streams in Unix**: Unix systems have three data streams: stdin, stdout, and stderr.
   In the script, `>/dev/null` is used to discard the standard output (stderr) to the `dev/null` file,
   which acts as a special file discarding all data written to it.
   Similarly, `>/dev/null 2>&1` is used to discard the standard error (2) to wherever the standard output (1) is currently going.

2. **Checking the Exit Status**: The script utilizes the `$?` variable to hold the exit status or status code of the previous command.
   A status code of 0 indicates success, while a non-zero value indicates an error. The script checks if the exit status is not equal to zero using the `-ne` operator.

## Conclusion

The final script can be found here: [Script 0](https://gist.github.com/ScMofeoluwa/01b736484b4c9cf1ad76ab8eb057d625) and a less verbose one here: [Script 1](https://gist.github.com/ScMofeoluwa/66c3019db70c8fdb6cee2b9c94e628ac)
