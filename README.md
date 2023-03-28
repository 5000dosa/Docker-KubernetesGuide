# Docker-KubernetesGuide

# 2023-03-22 수요일
도커와 쿠버네티스를 공식사이트 문서를 보고 만들어 볼려고 했지만 
처음부터 뭐 부터 해야 할지 막막해서 학원에서 준 '컨테이너 인프라 환격 구축을 위한 쿠버네티스/도커'를
처음부터 끝까지 해보고 그 기록을 남기고자 Repositories를 만들었다.
p125 3.2.5까지 따라 해보면서 느낀점은 저자가 처음부터 이 책은 나 같이 공식사이트를 보고 못따라하는 사람들을 위해
혹은 처음 컨테이너를 접하는 사람들을 위해 만들었으면 Cloud Native가 아니고 그 전까지의 과정이 담겨져 있다.
처음에는 이 책을 보고 코드를 따라 쳣지만 3장부터는 아예 만들어진 컨테이너를 가져오기때문에 따라칠수는 없지만 
그대신 책에 나와있는 모든 코드를 따라쳐보기로 했다.

3.3.4 온프레미스에서 로드밸런서를 제공하는 MetalLB 과정에서 168p 9번에서 EXTRNAL-IP가 책이랑 다르게
pending이 떠 로드밸런서 부분 진행 불가...
https://github.com/Azure/AKS/issues/1392 여기서 보고 고쳐볼려고했지만 포기하고 
할수 있는 부분만 진행하고 다음단계로 진입

# 2023-03-23 목요일
로드밸런스 IP에 부여에 대해서 알아봤지만 결국에 pending은 해결하지 못한 상태에서 진행하기로 했다.
173P kubectl edit deployment hpa-hname-pods 에서 변경이 되지 않는데 구글에서 찾아보니 
MetalLB가 재대로 되어있지 않은걸 확인했다.
kubectl get pods -n metallb-system -o wide로 확인 해보면 STATUS가 책에서는 Running이 되어 있지만
ImagePullBackOff가 뜨고 심지어 
kubectl apply -f ~/_Book_k8sInfra/ch3/3.3.4/metallb.yaml 입력하면
모든파일들이 unchanged가 되었다는걸 이제야 확인했고 이걸 해결하기 위해서  chat GPT에게 물어보았지만
읽어도 내가 해결을 할수는 없고 MetalLB를 사용하는 실습예제는 모두 넘어가서 쿠버네티스를 끝내기로 한다.
이미지 파일 3장 README에 chat GPT에게 물어본 내용을 정리했다.

오늘까지 쿠베네티스를 끝내고 도커를 시작했다. 쿠버네티스에서 너무 아쉬운건 로드밸런서에 IP가 보류가 뜬다는 점이다.
만약 보류가 안떳으면 완벽하게 모든 예제를 따라했을텐데.. 아쉬움을 뒤로하고 도커를 들어가기로 했다.

# 2023-03-24 금요일
도커는 263p 9단계에서 에러가 났다. https://raw.githubusercontent.com/sysent4admin/_Book_k8sInfra/main/ch4/4.3.4/Dockerfile
필자가 깃허브에 올려둔 Dockerfile을 받아 와 테스트를 위한 컨테이너 이미지를 받아야 하는데
다운로드가 되지 않는다.
10단계에서도 docker build로 컨테이너 이미지 multistage-img를 워커 노드 3번에 빌드하고 결과가 성공적으로
이루어졌는지 확인해야 하지만
docker build -t multistage-img를 입력하면
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /root/Dockerfile: no such file or directory
(컨텍스트를 준비할 수 없음: 도커 파일 경로: lstat /root/Docker 파일: 해당 파일 또는 디렉토리가 없음)라는 문구가 나온다.
당연히 구글에서 찾아보니 stackoverflow 에서 도커를 다시 설치하라는 답변이 있지만. 계속해서 에러가 나오면 그때 고쳐보자
하는 식으로 계속진행했고 다행인 263p 9단계에서만 에러가 났고 나머지는 정상적으로 다 실습을 진행을 했다.

오늘까지 도커를 끝내고 그다음 개발자들이 사랑하는 지속적통합과 배포자동화, 젠킨스를 진행하기로 한다.
젠킨스 290p에서 LoadBalancer만 나오면 EXTERNAL-IP가 pending이 된다. 서버쪽에 문제라고 하지만 지금은 내가 해결할수 없으니 
로드밸런스부분만 빼고 그대로 진행하기로 한다.

300p 8단계에서 인자받는 명령줄인데 책에서 했던것처럼 바로 복사하면 안돼고 
helm install metallb edu/metallb
--namespace=metallb-system
--create-namespace
--set controller.tag=v0.8.3
--set speaker.tag=v0.8.3
--set configmap.ipRange=192.168.1.11-192.168.1.29

이 명령줄들은 한줄로 만들어서 넣어야 한다.
helm install metallb edu/metallb --namespace=metallb-system --create-namespace --set controller.tag=v0.8.3 --set speaker.tag=v0.8.3 --set configmap.ipRange=192.168.1.11-192.168.1.29 <-- 이런식으로 한다.

302p 11단계에서 헬름을 통해서 MetalLB를 생성해서 LoadBalancer 타입으로 노출했을 때 드디어 IP가 정상적으로 할당댔다
304P 2단계 미리 정의된 nfs-exporter.sh jenkins를 실행하는 부분까지 하고 오늘은 마무리 합니다.

# 2023-03-25 토요일
5.3 젠킨스 설치 및 설정하기 챕터를 통해 처음으로 젠킨스를 설치해보고 접속해 봤다.
처음 로그인 화면을 보자 드디어 뭔가 있어보이는걸 하는구나라는 걸 체감을 했다.
336p까지 젠킨스 설정과 메뉴 내용을 보고 마지막으로 jenkins 서비스 어카운트를 한다음 쿠버네티스 역할 부여 구조를 확인했다.

# 2023-03-26 일요일
휴식

# 2023-03-27 월요일
5.5.1 쿠버네티스 환경에 적합한 선언적인 배포 환경 챕터에서 30단계까지 왔는데 처음부터 다시 해야겠다.
github 랑 연동하는 건데 나는 기존에 만들어두었던 깃저장소로 했지만 Error fetching remote repo 'origin' 여기서 막혔다
stackoverflow에서 
git config --global --unset credential.helper
git config --system --unset credential.helper
명령으로 자격증명을 비활성하라고 하지만 다시 처음부터 새로운 저장소를 만들어서 해봐야겠다.
다행이도 정상적으로 10분마다 배포가 되는걸 확인했다.
406p 5.5.2 19단계까지 하고 오늘은 아쉽지만 마무리 하기로 했다. github랑 연동만 잘되었으면 오늘안에 젠킨스를 끝냈을텐데.
아쉬움은 뒤로하고 내일 마무리하겠다.

# 2023-03-28 화요일
Slack과 gituhb를 책에 나온대로 에러없이 그대로 진행이 되었다. 수퍼푸티창에서 저장되어있는 파일을 깃허브저장소로 복사하고
푸시를 해 깃허브저장소에서 푸시가 되었는지 확인
dpy-pl-gitops에서 3차 배포를 하면 Slack에서 젠킨스가 보낸 메시지를 확인할 수 있다.
이렇게 변경된 Jenkinsfile를 한번 더 배포해서 4차까지 완료가 되면 Check Last changed까지 화인해 코드가 변경 사항을 확인할 수 있는 페이지까지
확인하였다. 이제부터 6장 안정적인 운영을 완서앟는 모니터링, 프로메테우스와 그라파나 장에 들어간다.
6장에서는 컨테이너 인프라 환경에 주요 요소를 수집하고, 수집된 내용을 한눈에 파악할 수 있도록 대시보드에 작성해 효과적인 모니터링 시스템을
구축해 보겠습니다. 그리고 이상을 감지하면 알려주는 서비스를 추가해 컨테이너 인프라 환경을 완성하겠습니다.
이 채은 자유도가 높은 오픈소스를 활용해 컨테이너 인프라 환경을 구현하고 실습하는 것이 목적이므로 기능이 부족하지 않고, 확장성도 뛰어난 
프로메테우스와 그라파나 조합으로 모니터링 시스템을 구축합니다.

498P 5단계까지로 오늘 마무리를 합니다.
처음 그라파나를 했을 때 당황했던게 6.4.3 노드 메트릭 데이터 시각화하기에서
메트릭(Metrics) 부분에 쿼리 입력부분에서 처음에 타자를 치다가 입력도 되지않고 버벅이면서 뭔가 실행되는 것처럼 렉 현상이 나와서
당황했다. 텍스트창 하나 열어서 먼저 쓴다음 쿼리부분에 옮겨 넣었던것 빼고는 에러없이 무사히 넘어갔다. 
내일은 6장을 마무리하고 최종결과와 그 과정을 PPT로 만들어서 업로드 할 예정이다.
내일까지 딱 작업하면 일주일만에 책 한권을 독학하고 결과물이 나온다.