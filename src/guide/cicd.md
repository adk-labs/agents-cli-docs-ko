# CI/CD 및 프로덕션

풀 리퀘스트(PR) 시 테스트를 실행하고, 머지(merge) 시 스테이징에 배포하며, 수동 승인을 거쳐 프로덕션으로 프로모션하는 CI/CD 파이프라인을 구축하세요.

![Prototype to Production](../assets/prototype_to_prod.png)

---

## 파이프라인 작동 방식

1. **CI 파이프라인** (풀 리퀘스트 시 트리거):
    - 단위 테스트 및 통합 테스트를 실행합니다.

2. **스테이징 CD 파이프라인** (`main` 브랜치 머지 시 트리거):
    - 컨테이너를 빌드하여 Artifact Registry에 푸시합니다.
    - **스테이징 환경**에 배포합니다.
    - 자동화된 부하 테스트를 실행합니다.

3. **프로덕션 배포** (스테이징 성공 후 트리거):
    - 진행하기 전에 **수동 승인**이 필요합니다.
    - 스테이징에서 검증된 동일한 컨테이너 이미지를 배포합니다.

---

## 파이프라인 설정하기

단 하나의 명령으로 인프라를 프로비저닝하고 CI/CD를 설정하세요:

```bash
agents-cli infra cicd \
  --staging-project my-staging-project \
  --prod-project my-prod-project
```

다음 항목들이 처리됩니다:

- Terraform을 통한 스테이징 및 프로덕션 **인프라 프로비저닝**
- 선택한 러너(Cloud Build 또는 GitHub Actions)를 사용한 **CI/CD 설정**
- GitHub과의 **리포지토리 연결**

### CI/CD 러너 감지

| 러너 | 감지 방식 |
|--------|-------------------|
| GitHub Actions | 프로젝트의 `wif.tf`에서 자동 감지됩니다. 키 없는(keyless) 인증을 위해 Workload Identity Federation을 사용합니다. |
| Google Cloud Build | Terraform 설정에서 자동 감지됩니다. GitHub과의 Cloud Build 연결을 설정합니다. |

### 옵션

| 플래그 | 설명 |
|------|-------------|
| `--staging-project` | 스테이징용 GCP 프로젝트 ID (필수) |
| `--prod-project` | 프로덕션용 GCP 프로젝트 ID (필수) |
| `--cicd-project` | CI/CD 리소스용 별도 프로젝트 (기본값: prod) |
| `--dev-project` | 개발 프로젝트 (선택 사항, 개발 인프라도 함께 프로비저닝) |
| `--repository-name` | GitHub 리포지토리 이름 |
| `--repository-owner` | GitHub 리포지토리 소유자 |
| `--local-state` | GCS 대신 로컬 Terraform 상태 사용 |
| `--create` | 새 GitHub 리포지토리 생성 (기존 리포지토리를 사용할 경우 생략) |

---

## Terraform 변수

파이프라인은 `deployment/terraform/variables.tf`에 정의된 Terraform 변수를 사용합니다:

| 변수 | 설명 |
|----------|-------------|
| `project_name` | 리소스 명명을 위한 기본 이름 |
| `prod_project_id` | 프로덕션용 Google Cloud 프로젝트 ID |
| `staging_project_id` | 스테이징용 Google Cloud 프로젝트 ID |
| `cicd_runner_project_id` | CI/CD 파이프라인이 실행되는 Google Cloud 프로젝트 ID |
| `region` | Google Cloud 리전 (기본값: `us-west1`) |
| `repository_name` | GitHub 리포지토리 이름 |
| `repository_owner` | GitHub 사용자 이름 또는 조직명 |
| `app_sa_roles` | 애플리케이션 서비스 계정 권한(역할) |
| `cicd_roles` | CI/CD 러너 서비스 계정 권한(역할) |

---

배포가 완료되면 `agents-cli publish gemini-enterprise`를 사용하여 Gemini Enterprise에 에이전트를 등록하세요. 모든 옵션을 확인하려면 `--help`와 함께 실행하세요.
