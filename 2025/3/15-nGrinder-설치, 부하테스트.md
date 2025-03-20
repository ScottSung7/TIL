- [nGrinder 적용](https://notspoon.tistory.com/48)
- [설치와 사용](https://velog.io/@gjwjdghk123/nGrinder-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-%EB%B0%8F-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%B4%EB%B3%B4%EA%B8%B0)
- [예산](https://velog.io/@sontulip/web-performance-budget)
- [PageSpeedInsight](https://pagespeed.web.dev/)
- [클라우드](https://velog.io/@dydgjs2016/nGrinder-%EC%84%A4%EC%B9%98-%EB%B0%8F-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%B4%EB%B3%B4%EA%B8%B0)
```
./run_agent_bg.sh 백그라운드 에이전트 실행

pgrep -fl run_agent.sh 에이전트 실행 확인

java -Djava.io.tmpdir=/home/rocky/ngrinder/lib -jar ngrinder-controller-3.5.9.war --port 7070 엔그라인더 컨트롤러 실행
nohup java -Djava.io.tmpdir=/home/rocky/ngrinder/lib -jar ngrinder-controller-3.5.9.war --port 7070 > output.log 2>&1 & 백그라운드 실행 & 에러시 끄지않고 로그로 출력.

jar -xvf controller.war war 압축 풀기
tar -svf agent.tar tar 압축 풀기

sudo vim agent.conf 초기 설정(같은 서버라면 localhost, 아니라면 security group에서 풀어주어야 함)

common.start_mode=agent
agent.controller_host=아이피
agent.controller_port=16001
agent.resion = NONE
agent.owner= admin

```
<br>

- [시나리오](https://kirinman.tistory.com/102)
- [비교](https://baeji-develop.tistory.com/118)
- [내용 1](https://giron.tistory.com/83)
- [대용량 트래픽 테스트 1](https://velog.io/@zo_meong/Project-ngrinder-%EB%8C%80%EC%9A%A9%EB%9F%89-%ED%8A%B8%EB%9E%98%ED%94%BD-%ED%85%8C%EC%8A%A4%ED%8A%B8-3-time-out)
- [대용량 트래픽 테스트 2](https://afuew.tistory.com/20)
- [대용량 트래픽 테스트 3](https://2021-pick-git.github.io/practice-nGrinder/)
- [대용량 트래픽 테스트 - 튜닝](https://velog.io/@nick9999/Outstagram-nGrinder%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%B6%80%ED%95%98-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%9B%84-%EC%84%B1%EB%8A%A5-%ED%8A%9C%EB%8B%9D)
- [부하테스트 JSCode](https://www.youtube.com/playlist?list=PLtUgHNmvcs6qAqWz-UhH-_ploSbK2eHwG)
- [스프링 캠프](https://www.youtube.com/watch?v=jWxUMtum-H0)
- [+ Pinpoint](https://velog.io/@max9106/nGrinderPinpoint-test1)
