# Word Search
## Introduction
Word Search application has 2 components:
1. `word_search_system`: Responsible for managing the word list and facilitating queries to the it
2. `word_search_api`: Exposes a REST API for clients to use. Talks via gRPC to the `word_search_system`

Contents:
1. [Build](#build)
2. [Deploy](#deploy)
3. [Source](#source)
4. [Notes](#notes)
   
## Build
#### Install CLI
```sh
git clone https://github.com/chrisjpalmer/word_search_cli
cd word_search_cli
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
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.0.0 word_search_api
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.0.0 word_search_system
```

Sample output:
```sh
TODO
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
| 1.0.0 | 1.0.0 |

## Deploy
#### Clone Production Deploy
```sh
git clone https://github.com/chrisjpalmer/word_search_deploy_production --branch wsa-1.0.0-wss-1.0.0
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
TODO
```

#### Versions
The Production Deploy repo has the following tags:

| production deploy version | word_search_api version | word_search_system version |
| :-- | :-- | :-- |
| wsa-1.0.0-wss-1.0.0 | 1.0.0 | 1.0.0 |


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
You cannot use some versions of the word_search_system with the word_search_api because they have incompatible gRPC implementation.
This table shows the gRPC versions and which repos support them.

| word_search_system_grpc | word_search_api version | word_search_system version |
| :-- | :-- | :-- |
| 1.0.0 | supported by: 1.0.0 | supported by: 1.0.0 |

In reality, some scheme should be decided about when grpc versions are incompatible due to major change to the implementation.

#### Dependency management
I use `dep` to manage dependencies. `dep` allows version constraints to be placed on dependencies. `dep ensure` command makes the dependency tree sync with the contraints and code. I used this to make binding constraints on the grpc version for each repo.

## Notes
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
