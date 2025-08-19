# CMD

docker ps --format "{{.Names}}"

git branch | grep -v  "main" | grep -v "dev" | grep -v "^*" | xargs git branch -D;

brew update && brew upgrade && brew upgrade --cask --greedy && brew cleanup

git reset --hard HEAD

docker system prune --volumes --all

docker-compose down -v --rmi all --remove-orphans