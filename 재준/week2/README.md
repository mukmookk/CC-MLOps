# K8s 설치

1. Docker 설치
   - 공식 레퍼런스 기준 ( https://docs.docker.com/engine/install/ubuntu/ )
   - Swap Memory 설정 변경
2. k8s API 요청을 위한 kubectl 설치
3. K3s 세팅
   - 오류
     - The connection to the server 127.0.0.1:6443 was refused - did you specify the right host or port?
   - kubectl cluster-info
     - 명령어를 사용하면 어디서 쿠버네티스가 실행되고 있는지 알 수 있다.
     - 127.0.0.1과 포트가 제대로 설정되어있지 않거나 Docker가 정상적으로 설치되어있지 않거나 작동하지 않으면 발생하는 오류
     - Docker 부터 다시 설치
4. Helm 설치
5. Kustomize 설치
6. CSI Plugin : Local Path Provisioner 설치

# Kubeflow 설치

1. Kubeflow 설치 파일 준비
2. Cert-manager 설치
3. Istio 설치
   - Istio는 마이크로서비스 간 데이터 공유를 제어하는 기반을 제공하는 오픈소스 서비스 메쉬 플랫폼입니다.
   - Istio is an open source service mesh that layers transparently onto existing distributed applications. It provides a uniform and more efficient way to secure, connect, and monitor services.
   - It is the path to load balancing, service-to-service authentication, and monitoring — with few or no service code changes.
   - Istio addresses the challenges developers and operators face with a distributed or microservices architecture.
4. Dex 설치
   - Dex는 서드 파티로부터 OAuth 인증 토큰을 가져와 관리하는 인증 도구이다. OAuth를 사용하기 위해 반드시 Dex를 써야 하는 것은 아니지만, Dex는 OAuth 서드 파티와의 중간 매개체 역할을 해주기 때문에 OAuth의 인증 토큰의 발급, 저장 및 관리를 좀 더 수월하게 해결할 수 있다. Dex는 2019년 7월 기준으로 LDAP, Github, SAML 세 가지 종류의 서드 파티를 Stable로서 지원하고 있다.
5. OIDC AuthService 설치
6. Kubeflow Namespace 생성
7. Kubeflow Roles 설치
8. Kubeflow Istio Resources 설치
9. Kubeflow Pipelines 설치 및 UI 반영
10. Katib 설치
    - 하이퍼파라미터 최적화 오픈소스를 쿠버네티스 위에서 서비스

# 결정적 오류

```
Unable to connect to the server: net/http: TLS handshake timeout
```

[Compact] 1vCPU, 2GB Mem, 50GB Disk [g1] 인스턴스에서는 돌아가지 않음.

# 레퍼런스

- https://www.redhat.com/ko/topics/microservices/what-is-istio
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=alice_k106&logNo=221598325656
