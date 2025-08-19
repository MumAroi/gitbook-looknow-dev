# CMD

docker ps --format "{{.Names}}"

git branch | grep -v  "main" | grep -v "dev" | grep -v "^*" | xargs git branch -D

 git branch -vv | grep ': gone]' | grep -v '\*' | awk '{print $1}' | xargs -r git branch -D

brew update && brew upgrade && brew upgrade --cask --greedy && brew cleanup

git reset --hard HEAD

docker system prune --volumes --all

docker-compose down -v --rmi all --remove-orphans