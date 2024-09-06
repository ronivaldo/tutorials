# GitFlow Tutorial

Written based on [GitFlow](https://nvie.com/posts/a-successful-git-branching-model/).
```
1. Create project on Gitlab
private and enable Readme.md

2. Clone the project to computer
git clone https://gitlab.com/ronivaldo/scale-server-3.git

3. Init Git Flow
cd scale-server-3
# make default directories
mkdir dist docs src test tools
# init flow
git flow init
# push new branch develop to origin
git push origin develop

4. Initial Version
git checkout master
# copy initial source files
cp old/scale-server-3 ./src/
# bump version to readme
echo "Scale Server 3.0.0" >> README.md
# copy zip source file to dist
cp /old/dist/scale_server_3.0_source_code.zip ./dist/scale_server_3.0_source_code.zip
# copy gitignore
cp /old/.gitignore ./.gitignore
# add all files to stage
git add .
git commit -am "initial 3.0.0"
git push origin master
# create tag for initial
git tag "3.0.0"
# push tag to origin
git push --tags
# merge master into develop and push
git checkout develop
git merge master
git push origin develop

5. Hotfix
git status
git checkout master
git pull
# start hotfix
git flow hotfix start 3.0.1
# publish to origin
git flow hotfix publish 3.0.1
# update Readme.md
nano Readme.md
# update the files and check status
git status
# add all files to stage
git add .
git commit -am "3.0.1"
# push to origin the updated hotfix branch
git flow hotfix publish 3.0.1
# finish the hotfix
# use the 3.0.1 for commit messages
# automatically merge into master and develop
git flow hotfix finish 3.0.1
# push master to origin
git checkout master
git push origin master
# push develop to origin
git checkout develop
git push origin develop
# push tag to origin
git push --tags

5. New Feature
git status
git checkout develop
git pull origin develop
# start feature
git flow feature start auto-configuration-service-agent
# publish to origin
git flow feature publish auto-configuration-service-agent
# update Readme.md
nano Readme.md
# update the files and check status
git status
# add all files to stage
git add .
git commit -am "Auto-configuration mode and Service Agent"
# push to origin the updated hotfix branch
git flow feature publish auto-configuration-service-agent
# finish the feature
# use the auto-configuration-service-agent for commit messages
# automatically merge into develop
git flow feature finish auto-configuration-service-agent
# push to origin
git push origin develop

6. New Release
git status
git checkout develop
# start release
git flow release start 3.1.1
# publish to origin
git flow release publish 3.1.1
# update Readme.md
nano Readme.md
# update the files and check status
git status
# add all files to stage
git add .
git commit -am "3.1.1-rc.1"
# finish the release
# use the 3.1.1 for commit messages
# automatically merge into master and develop
git flow release finish 3.1.1
# push develop and master to origin
git checkout develop
git push origin develop
git checkout master
git push origin master
# push tag 3.1.1 to origin
git push --tags

7. Check Updates with Origin
# update with the changes in origin
git fecth origin
# list the changes
git diff master origin/master
# list only the file names
git diff --name-only master origin/master 
# get the changes
git pull origin master

8. Work on existing feature branch
# init git flow
git flow init
# track remote feature
git flow feature track this-is-my-feature

9. Work on existing hotfix branch
# init git flow
git flow init
# track remote hotfix
git pull
git checkout hotfix/<name>
```

## Authors

-   **Ronivaldo Sampaio**
