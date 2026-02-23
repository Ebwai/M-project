## 文件结构
- 原文件的地址是随项目一块下载下来的,原配置文件为docker-compose.yml和docker-compose-base.yml
- docker-compose-gpu.yml:为了启用GPU版本，我做了一些修改，并且设置了一些环境变量，使得它可以接入minerU.
    - 关键修改：
        - 把ragflow-gpu:中的    profiles: - gpu选项给取消了，因为假如不取消的话，他默认启动的是CPU版本，profiles是一个选择的意思，只有在构建docker-compose-gpu.yml的时候加上- gpu才会启用GPU版本。
        - 新增了minerU的服务：
             MINERU_APISERVER=${MINERU_APISERVER}
             MINERU_DELETE_OUTPUT=${MINERU_DELETE_OUTPUT}
- .env文件为环境变量文件,需要根据实际情况修改
   - 关键修改： 
        - #USE_MINERU=true：一定不要添加这个选项，添加了要把它给注释掉。因为假如启动这个选项为true，那么它就会自动去下载新的minerU，它就不会去调用，笔记本电脑里的minerU fastapi.
        - # MinerU Configuration
            #MINERU_APISERVER=http://127.0.0.1:8000
            #MINERU_APISERVER是非常重要的minerU配置环境全局变量。很多地方直接调用这个环境变量，我在这里配置的是docker去调用宿主机的地址的形式。
            MINERU_APISERVER=http://host.docker.internal:9987#
            #MINERU_BACKEND=pipeline#不要开启这个，不然的话他默认用CPU执行。
            MINERU_DELETE_OUTPUT=1#1表示生成完临时文件会自动删除。
