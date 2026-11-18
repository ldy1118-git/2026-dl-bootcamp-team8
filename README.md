# Visual Task 개발과정 · Team 8 토큰부족

2026 AICOSS 산학협력 실습중심 딥러닝 여름부트캠프 & 경진대회(서울시립대학교) 8팀 **토큰부족**의 결과물입니다.

**프로젝트 페이지:** https://ldy1118-git.github.io/2026-dl-bootcamp-team8/

| 과제 | 내용 | 점수 |
|---|---|---|
| Task 1 · 상권분석 VQA | 매장 이미지 한 장으로 8개 업종(한식·양식·일식·중식·카페·베이커리·주점·동남아) 중 하나를 분류 | Accuracy **0.780** |
| Task 2 · Reasoning Segmentation | 이미지와 간접적인 질문을 받아, 질문이 가리키는 대상을 추론해 마스크로 분할 | Dice **0.513** |

두 과제 모두 추가 학습 없이 **Qwen3-VL-8B-Instruct**를 중심으로 추론 절차와 프롬프트를 다듬어 풀었습니다.

## Task 2 파이프라인

1. **쿼리 유형 분류.** Qwen3-VL이 질문을 A~F(직접 지칭, 간접 단서, 상황·기능, 비교·순위, 계산·논리, 사실 지식) 중 하나로 분류합니다.
2. **대상 찾기.** 유형별 추론 지시를 붙인 프롬프트로 대상마다 `bbox_2d`와 `point_2d`를 JSON으로 받습니다.
3. **마스크 생성.** bbox를 1.03배 넓혀 point와 함께 SAM 2.1(hiera large)에 넣고, 대상별 마스크를 합칩니다.
4. **제출.** 마스크를 RLE로 인코딩해 CSV로 저장합니다.

분류나 JSON 파싱이 실패하면 범용 프롬프트로 돌아가는 fallback이 있습니다.

점수 변화: 0.4724(베이스라인) → 0.474(bbox 1.03배) → 0.492(라우터 + 출력 규칙) → 0.505 → 0.5105(요구 조건 추출) → **0.513**(최종 확인 단계)

## 폴더 구조

```
index.html                      프로젝트 페이지 (GitHub Pages)
assets/examples/                페이지에 쓰인 결과 예시 이미지 6장
slides/team8_발표.pptx           발표자료 (Task 1, Task 2)
task2/
  ch2_team8.ipynb               Task 2 최종 제출 코드 (0.513)
  real_submission8.csv          최종 제출 파일
  sample_submission2.csv        제출 양식 (노트북 실행에 필요)
  previous_51.0/
    task2_51.0.ipynb            최종 확인 단계를 넣기 전 버전, 200개 실행 로그 포함
    submission_51.0.csv         위 버전의 제출 파일
```

## 참고

- 노트북의 모델·데이터 경로는 부트캠프 서버 기준입니다. 다른 환경에서 실행하려면 `MODEL_PATH`, `SAM2_1_MODEL_PATH`, 데이터 경로를 바꿔야 합니다.
- 필요한 것: `transformers`(Qwen3-VL 지원 버전), `torch`, `flash-attn`, [SAM 2](https://github.com/facebookresearch/sam2), Qwen3-VL-8B-Instruct와 SAM 2.1 hiera large 가중치.
- 대회 데이터셋(이미지·쿼리)은 주최 측 자료라 넣지 않았습니다. 페이지의 예시 이미지 6장만 결과를 보여주려고 테스트셋에서 골랐습니다.
- `previous_51.0/task2_51.0.ipynb`는 원본 용량이 65MB라서 결과 그림 400장을 빼고 텍스트 실행 로그만 남겼습니다.
- Task 1 코드는 아직 포함되어 있지 않습니다.
