# Word Search
## Introduction
Word Search application has 2 components:
1. `word_search_system`: Responsible for managing the word list and facilitating queries to the it
2. `word_search_api`: Exposes a REST API for clients to use. Talks via gRPC to the `word_search_system`

#### Prerequisites:
1. Docker OR Docker-toolbox
2. Node.js (tested on 10.15.0 but above 8 should work)
3. Golang 1.11 (not necessary unless you want to dev the code)

#### Contents:
1. [Build](#build)
2. [Deploy](#deploy)
3. [Source](#source)
4. [Design Notes](#design-notes)

## Build
#### Install CLI
```sh
git clone https://github.com/chrisjpalmer/word_search_cli
cd word_search_cli
npm install
npm link

cd /my/blank/proj/dir #specify a blank project directory
word_search_cli init #initializes a new word_search_proj in your current directory
```

#### Point shell to docker instance
You can skip this step if your shell automatically points to the docker daemon.
```sh
eval $(docker-machine env default)
```

#### Build and Test
```sh
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.2.0 word_search_api
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.0.0 word_search_system
```

Sample output:
```sh
...
Step 7/9 : RUN go test -v
 ---> Running in fd62fcb1d4aa
=== RUN   TestWordSearchService
--- PASS: TestWordSearchService (0.00s)
=== RUN   TestWordSearchService_SearchWord
--- PASS: TestWordSearchService_SearchWord (0.00s)
=== RUN   TestWordSearchService_AddWord
=== RUN   TestWordSearchService_AddWord/basic_test
...
Successfully tagged chrisjpalmer/word_search_system:1.0.0
```

#### Versions
The source lives here:
1. https://github.com/chrisjpalmer/word_search_system
2. https://github.com/chrisjpalmer/word_search_api

For each source there is a builder repo which packs the source into a docker container:
1. https://github.com/chrisjpalmer/word_search_system_builder
2. https://github.com/chrisjpalmer/word_search_api_builder

Each repo is versioned and certain versions of builders wont work with certain versions of source. See table below:

`word_search_system`

| builder version | requires source version |
| :-------------- | :---------------------- |
| 1.0.0 | 1.0.0 |

`word_search_api`

| builder version | requires source version |
| :-- | :-- |
| 1.0.0 | 1.0.0, 1.1.0, 1.2.0 |

#### Uninstall CLI
If you ever want to uninstall the CLI, just `cd` into the location where you originally cloned it, and run `npm unlink`
```sh
cd word_search_cli
npm unlink
```

## Deploy
#### Clone Production Deploy
```sh
git clone https://github.com/chrisjpalmer/word_search_deploy_production --branch wsa-1.2.0-wss-1.0.0
cd word_search_deploy_production
```

#### Point shell to docker instance
You can skip this step if your shell automatically points to the docker daemon.
```sh
eval $(docker-machine env default)
```

#### Deploy Stack
```sh
docker swarm init # if you don't already have a swarm
docker stack deploy -c word_search.yaml ws
```

#### Sample Commands
```sh
#Add Words
curl -H "Content-Type: application/json" -X POST --data '{"words":["cool"]}' http://localhost/words
{}

#Search Words
curl -H "Content-Type: application/json" -X GET "http://localhost/words?keyword=co"
{"matches":["cool"]}

#Top 5 Searched Words
curl -H "Content-Type: application/json" -X GET http://localhost/keywords
{"keywords":["a","co","cool"]}

```

#### Versions
The Production Deploy repo has the following tags:

| production deploy version | word_search_api version | word_search_system version |
| :-- | :-- | :-- |
| wsa-1.0.0-wss-1.0.0 | 1.0.0 | 1.0.0 |
| wsa-1.1.0-wss-1.0.0 | 1.1.0 | 1.0.0 |
| wsa-1.2.0-wss-1.0.0 | 1.2.0 | 1.0.0 |


I guess in real situations, you would use a repo like this in conjunction with AWS Secrets manager for handling keys.

## Source
There are 3 source repositories:
1. https://github.com/chrisjpalmer/word_search_system : the `word_search_system` source
2. https://github.com/chrisjpalmer/word_search_api : the `word_search_api` source
3. https://github.com/chrisjpalmer/word_search_system_grpc : the protocol buffer definitions + transpiled go files for grpc communication

#### Clone
To play with them, use `go get`.
```sh
go get -u https://github.com/chrisjpalmer/word_search_system
go get -u https://github.com/chrisjpalmer/word_search_api
go get -u https://github.com/chrisjpalmer/word_search_system_grpc
```

To see specific versions checkout to the tag:
```sh
cd $GOPATH/src/github.com/chrisjpalmer/word_search_system
git checkout 1.0.0
```

#### Versions
You cannot use some versions of the `word_search_system` with the `word_search_api` because they have incompatible gRPC implementation.
This table shows the gRPC versions and which repos support them.

| word_search_system_grpc | word_search_api version | word_search_system version |
| :-- | :-- | :-- |
| 1.0.0 | supported by: 1.0.0, 1.1.0, 1.2.0 | supported by: 1.0.0 |

In reality, some scheme should be decided about when grpc versions are incompatible due to major change to the implementation.

#### Dependency management
I use `dep` to manage dependencies. `dep` allows version constraints to be placed on dependencies. `dep ensure` command makes the dependency tree sync with the contraints and code. I used this to make binding constraints on the grpc version for each repo.

## Design Notes

1. [How the build system works](#how-the-build-system-works)
2. [Application structure](#application-structure)
3. [Configuration](#configuration)
4. [All Repositories](#all-repositories)

#### How the build system works
I wanted to keep the build system as simple as possible whilst allowing flexiblity for the developer in the future.

Often changes to source can affect the way source is built, therefore I made the build process seperate.

*Builder Repos*

1. Each source has a builder repository which contains a Dockerfile.
2. The Dockerfile clones the source from github. 
3. The Dockerfile builds the source and runs the tests. 
4. If the tests fail, the image cannot be built.

The source tag can be injected into the Dockerfile build using the `--build-arg` flag
```sh
docker build -t name-of-image:X.X.X --build-arg SRC_TAG=X.X.X .
```
*CLI Builds*

The CLI does this automatically for you.
The CLI accepts the argument `--build-repo-tag` which specifies which version of the builder repo to use.
Compatibility between builders and source versions are listed above.

When you invoke this command:
```sh
word_search_cli build --build-repo-tag=1.0.0 --source-repo-tag=1.0.0 word_search_system
```
The CLI does the following:
1. Clones the builder repo in $CWD/temp
2. Checks it out to a specific tag
3. Runs `docker build -t name-of-image:X.X.X --build-arg SRC_TAG=X.X.X .`

#### Application Structure

*word_search_system*

This component manages the words list and exposes 3 methods via gRPC for manipulating and quering this words list:
1. SearchWord - takes a keyword as a parameter and searches through the words list, returning possible matches.
2. AddWords - takes a list of words as a parameter and adds them to the words list.
3. GetTop5KeyWords - returns the top 5 most searched keywords

This component defines a service object called `WordSearchService` which is the domain for all word searching logic.
Unit tests exist for this object.
The component recieves gRPC function calls `SearchWord`, `AddWords`, `GetTop5KeyWords` and executes logic on `WordSearchService`.

*word_search_api*

This component exposes a REST API which allows an http client to query the words list. It interfaces with the `word_search_system` component via gRPC.

The REST API exposes the following methods:
1. GET /words - takes a keyword as a parameter and searches through the words list, returning possible matches.
2. POST /words - takes a list of words as a parameter and adds them to the words list.
3. GET /keywords - returns the top 5 most searched keywords

#### Configuration
The `word_search_system` needs to know what port it should listen on. Likewise the `word_search_api` needs to know the address of the `word_search_system` to invoke its methods via gRPC. This is set dynamically by configuration files. The configuration files are passed into the docker swarm as instructed by the docker stack file. Then they are mounted in the container. When each component boots, it reads and parses this config information.

The config file templates are available in the source of each component as `config.template.json`. This helps other developers understand what the config file should look like.

Also during development, each application will run successfully if the developer simply performs:
```sh
cp config.template.json config.json
cd $GOPATH/src/github.com/chrisjpalmer/word_search_api
go run github.com/chrisjpalmer/word_search_api
```
`.gitignore` is set to ignore `config.json` so the developer is free to mess around here.


#### All Repositories:
1. https://github.com/chrisjpalmer/word_search : README
2. https://github.com/chrisjpalmer/word_search_system : the `word_search_system` source
3. https://github.com/chrisjpalmer/word_search_api : the `word_search_api` source
4. https://github.com/chrisjpalmer/word_search_system_grpc : the protocol buffer definitions + transpiled go files for grpc
5. https://github.com/chrisjpalmer/word_search_system_builder : the `word_search_system` builder
6. https://github.com/chrisjpalmer/word_search_api_builder : the `word_search_api` builder
7. https://github.com/chrisjpalmer/word_search_cli : the cli for building the images quickly
8. https://github.com/chrisjpalmer/word_search_deploy_production : already assembled docker stack files ready for quick deployment
