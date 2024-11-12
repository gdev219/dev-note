# dev-note


# NextJS


- 재밌는 점을 발견했음
NextJS App 을 dockerize 해서 google cloud run 에 deploy 하는 과정중 보안 상의 이슈로 cloud run app 에 static out bound ip 를 설정해 주어야 했음
내 Ip를 확인하기 위해 Custom 으로 /api/ip 라는 route를 생성 해 테스트 했지만 계속 처음보는 IP를 return하고 있었음
debugging 하기 위해 python 으로 간단하게 내 public ip를 호출하는 app 을 만들어 deploy 결과 정상적으로 nat에 설정된 고정 IP가 보였으나
여전히 next app에서는 문제가 있었음

문제
gitlab ci runner에서 build 과정에 모든 api가 한번씩 호출되는데 이 과정에서 memozation을 하는데 memozation 된 api가 return 되는 것 이었음

export const dynamic = 'force-dynamic'; 추가로 해결

# cloud run on NextJS

```ascii
                                              +----------------+
                                              | External DNS   |
                                              | Provider       |
                                              +--------+-------+
                                                       |
                                                       v
            +----------------+            +-----------+-----------+
            |                |            |      Google Cloud     |
            |  Private       |            |                       |
            |  Network       |            |  +------------------+ |
            |                |            |  |   Cloud Armor    | |
            | +------------+ |            |  +--------+---------+ |
            | | GitLab     | |            |           |           |
            | | CI/CD      | |            |  +--------+---------+ |
            | +-----+------+ |            |  |   Load Balancer  | |
            |       |        |            |  +--------+---------+ |
            | +-----+------+ |            |           |           |
            | | GitLab     | |    +-------+--+     +--+--------+  |
            | | Runner     +----->+ Google   |     | Cloud Run |  |
            | | (Kaniko)   | |    | Artifact +---->|           |  |
            | +------------+ |    | Registry |     | +--------+|  |
            |                |    +----------+     | |Frontend||<-|
            +----------------+                     | +--------+|  |
                                                   | |Backend ||  |
                                                   | +--------+|  |
                                                   +-----+-----+  |
                                                         |        |
                                       +----------+      |        |
                                       | Cloud    |      |        |
                                       | Storage  <------+        |
                                       +----------+               |
                                                                  |
                                              +------------------+|
                                              | VPC Access       ||
                                              | Connector        ||
                                              +--------+---------+|
                                                       |          |
                                                +------+------+   |
                                                | VPC Network |   |
                                                +------+------+   |
                                                       |          |
                                                +------+------+   |
                                                |  Cloud NAT  |   |
                                                +------+------+   |
                                                       |          |
                                           +-----------+----------+
                                           | external service     |
                                           +----------------------+


https://www.reddit.com/r/java/comments/17fztht/lightweight_but_powerful_java_stack/

```
