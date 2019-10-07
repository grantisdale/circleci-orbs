# CircleCI Orbs

### Set up packagecloud repositories to be used in CircleCI jobs
* `grantisdale/packagecloud` 
    : https://circleci.com/orbs/registry/orb/grantisdale/packagecloud


### Usage

#### Npm repositories

The `packagecloud/create` command will create a `.npmrc` in the home directory `(/home/circleci)` of the image/machine. In the `.npmrc` it will set the `registry` as your desired packagecloud URL and set the auth token for that URL as your newly created Read Token.

This will allow you to run `npm install` and `yarn install` and install packages from your packagecloud npm repository.

##### Example .yml
```
version: 2.1
      orbs:
        packagecloud: grantisdale/packagecloud@x.y.z
      jobs:
        build:
          docker:
          - image: circleci/node:10.16.3
          steps:
            - checkout

            - packagecloud/create:
                npm-repo: true
                username: packagecloud-username
                reponame: packagecloud-npm-reponame
                mastertoken: "$MY-NPM-MASTER_TOKEN"
                packagecloudtoken: "$MY-PACKAGECLOUD_API_TOKEN"

            - run:
                name: Install and test
                command: |
                  yarn install
                  yarn test

            - run:
                name: Install packagecloud cli
                command: |
                  if [[ $EUID == 0 ]]; then export SUDO=""; else export SUDO="sudo"; fi
                  $SUDO apt-get install ruby-full
                  $SUDO gem install rake
                  $SUDO gem install package_cloud

            - packagecloud/revoke:
                npm-repo: true
                username: packagecloud-username
                reponame: packagecloud-npm-reponame
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"
```

#### Maven repositories

The `packagecloud/create` is designed to be used with gradle. The command will create and set up a ` ~/.gradle/gradle.properties` file with the following lines:

`mavenPassword` = packagecloud API token

`<repo-name>.readtoken` = packagecloud read token for the repository

This command can be run more than once if you have more than one maven repository e.g. a releases and a snapshot repository. In your repositories `build.gradle` you can specify the the packagecloud maven repoistory to be used like so:

```
maven { url "https://packagecloud.io/priv/<read_token>/<username>/<repo1-name>/maven2" }
maven { url "https://packagecloud.io/priv/<read_token>/<username>/<repo2-name>/maven2" }
```

##### Example .yml

```
version: 2.1
      orbs:
        packagecloud: grantisdale/packagecloud@x.y.z
      jobs:
        build:
          machine:
            image: ubuntu-1604:201903-01
          steps:
            - checkout
            
            - packagecloud/create:
                maven-gradle-repo: true
                username: packagecloud-username
                reponame: packgecloud-maven-releases-reponame
                mastertoken: "$MY_MAVEN_RELEASES_REPO_MASTER_TOKEN"
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"

            - packagecloud/create:
                maven-gradle-repo: true
                username: packagecloud-username
                reponame: packgecloud-maven-snapshots-reponame
                mastertoken: "$MY_MAVEN_SNAPSHOTS_REPO_MASTER_TOKEN"
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"

            - run:
                name: Build
                command: |
                  ./gradlew

            - packagecloud/revoke:
                maven-gradle-repo: true
                username: packagecloud-username
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"
```

#### Pypi repositories

The `packagecloud/create` command will create an environment variable `READ_TOKEN`. This can be used as an `--index-url` or `--extra-index-url` in the command line or in your requirements.txt itself. Examples of both below:

On the command line:
 ```
 pip install -U -r requirements.txt --extra-index-url=https://${READ_TOKEN}:@packagecloud.io/<username>/<repo-name>/pypi/simple
 ```

In `requirements.txt`:`

```
--index-url https://${READ_TOKEN}:@packagecloud.io/<username>/<repo-name>/pypi/simple

docopt == 0.6.1
keyring >= 4.1.1
coverage != 3.5
Mopidy-Dirble ~= 1.1
```


##### Example .yml

```
version: 2.1
      orbs:
        packagecloud: grantisdale/packagecloud@x.y.z
      jobs:
        build:
          machine:
            image: ubuntu-1604:201903-01
          steps:
            - checkout

            - packagecloud/create:
                pypi-repo: true
                username: packagecloud-username
                reponame: packagecloud-pypi-reponame
                mastertoken: "$MY_PYPI_REPO_MASTER_TOKEN"
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"

            - run:
                name: Install dependencies
                command: |
                  pip3 install -U -r requirements.txt --extra-index-url=https://${READ_TOKEN}:@packagecloud.io/packagecloud-username/packagecloud-pypi-reponame/pypi/simple

            - run:
                name: Install packagecloud cli
                command: |
                  gem install package_cloud

            - packagecloud/revoke:
                pypi-repo: true
                username: packagecloud-username
                reponame: packagecloud-pypi-reponame
                packagecloudtoken: "$MY_PACKAGECLOUD_API_TOKEN"        
```