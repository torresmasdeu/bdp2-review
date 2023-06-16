final run command:
```docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter```
