1. Ajoutez les lignes suivantes dans le Dockerfile de votre application :

   ```go
   ARG DD_GIT_REPOSITORY_URL
   ARG DD_GIT_COMMIT_SHA
   ENV DD_TAGS="git.repository_url:${DD_GIT_REPOSITORY_URL},git.commit.sha:${DD_GIT_COMMIT_SHA}"
   ```

1. Ajoutez les arguments suivants à votre commande build Docker :

   ```go
   docker build . \
    -t my-application \
    --build-arg DD_GIT_REPOSITORY_URL=<git-provider.example/me/my-repo> \
    --build-arg DD_GIT_COMMIT_SHA=$(git rev-parse HEAD)
   ```