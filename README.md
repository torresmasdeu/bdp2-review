final run command:
```docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter```

  341  mkdir review
  342  cd review/
  343  docker run -d --rm --name my_redis -v ~/review:/data --network bdp2-net --user 1000 redis redis-server
  344  docker network create bdp2-net
  345  docker run -d --rm --name my_redis -v ~/review:/data --network bdp2-net --user 1000 redis redis-server
  346  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" jupyter/minimal-notebook
  347  docker ps
  348  docker stop my_jupyter 
  349  git clone https://github.com/torresmasdeu/bdp2-review.git
  350  mkdir docker
  351  mkdir work
  352  ls
  353  rmdir docker
  354  rmdir work/
  355  ls
  356  cd bdp2-review/
  357  mkdir work
  358  mkdir docker
  359  ls
  360  git commit
  361  git push
  362  docker ps
  363  cd docker/
  364  vi Dockerfile
  365  git status
  366  cd ..
  367  git status
  368  ls
  369  git add docker/
  370  git status
  371  git commit -m "Commit Dockerfile"
  372  git status
  373  git push -u origin main
  374  git pull
  375  git config pull.rebase false
  376  git push -u origin main
  377  git pull
  378  git status
  379  git pull
  380  git config pull.rebase false
  381  git pull
  382  vi docker/Dockerfile 
  383  cd ../
  384  ls
  385  rm -rf bdp2-review/
  386  git clone https://github.com/torresmasdeu/bdp2-review.git
  387  cd bdp2-review/
  388  mkdir docker
  389  mkdir work
  390  cd docker/
  391  vi Dockerfile
  392  cd ..
  393  git pull
  394  git status
  395  ls
  396  git add docker/
  397  git commit -m "Commit Dockerfile"
  398  cat docker/Dockerfile 
  399  git push -u origin main
  400  docker images
  401  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  402  docker images
  403  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  404  docker login
  405  docker push torresmasdeu/ltm_jupyter:tagname
  406  docker ps
  407  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  408  git pull
  409  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  410  git status
  411  exit
  412  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  413  docker login
  414  docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" ltm_jupyter
  415  docker images
  416  docker run -d --rm --name my_redis_jupyter -v ~/review/:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter
  417  docker ps
  418  docker stop
  419  docker stop my_jupyter 
  420  ls
  421  cd review/
  422  cd 
  423  cd review/bdp2-review/
  424  ls
  425  docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter
  426  ls work/
  427  vi README.md
  428  ls
  429  vi .gitignore
  430  git status
  431  git add * 
  432  git status
  433  git add .gitignore 
  434  git status
  435  git commit -a -m "Commit after docker run successful command"
  436  git push
  437  docker stop my_redis
  438  docker stop my_redis_jupyter
  439  docker ps
