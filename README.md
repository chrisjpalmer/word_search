# Word Search
## Introduction

## Getting Started
You can deploy the entire stack very quickly if you have git and docker installed on your machine.

### Clone Production Stack
```sh
git clone https://github.com/chrisjpalmer/word_search_deploy_production --branch wsa-1.0.0-wss-1.0.0 # see https://github.com/chrisjpalmer/word_search_deploy_production readme for more tags
cd word_search_deploy_production
```

### Prepare Docker Machine
```sh
eval $(docker-machine env default)
docker swarm init
```

### Deploy Stack
```sh
docker stack deploy -c word_search.yaml ws
```

### Test
```sh

```

## Build from the source
You can build from the source easily using the cli which ships with word_search:
### Clone CLI Tool and Install it
```sh
git clone https://github.com/chrisjpalmer/word_search_cli && cd word_search_cli && npm link
cd /my/blank/proj/dir #specify a blank project directory
word_search_cli init #initializes a new word_search_proj in your current directory
```

### Prepare Docker Machine
```sh
eval $(docker-machine env default)
```

### Build source via the CLI
```sh
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.0.0 word_search_api #see https://github.com/chrisjpalmer/word_search_api for more tags
word_search_cli build --build-repo-tag=1.0.0 --src-repo-tag=1.0.0 word_search_system #see https://github.com/chrisjpalmer/word_search_system for more tags
```

Whenever you build, tests are run at the same time. On your machine the build repository which contains the docker file for building the target is checked out. Then the CLI runs the `docker build` command on the repository. The dockerfile contains commands to pull the specific source tag from git hub, build the source and run the tests.
